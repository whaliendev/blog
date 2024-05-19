---
author: Hwa
date: 2023-08-28
title: 从一篇博客开始的GoLang Context
slug: golang-context
tags:
    - GoLang
    - Concurrency
summary: "golang context from golang official blog"
---

## 从一篇博客开始的GoLang Context

我们首先从一篇博客开始聊GoLang的Context包。博客来源是：https://go.dev/blog/context

首先我们来翻译一下该博客：

### GoLang并发模式：Context

#### 简介

在Go写的Server里，通常进来的request都是单独开一个goroutine来处理。Request Handdler程序也经常起额外的go routine来处理后端逻辑，比如访问数据库和RPC服务。这些处理同一个request的goroutine经常需要访问一些request特定的值，像用户的身份，，鉴权的token和request的deadline。为了系统能够更好的回收资源，当一个request被取消或者超时的时候，所有处理该request的goroutine都应该迅速的退出。

于是，Google内部就开发了一个叫context的包。这个包使得在request域内传递值，传播取消信号，处理一个request内所有goroutine的deadline超时非常藏方便。没错，这个包就是我们要从源码角度说明的[`context`包](https://pkg.go.dev/context)。这篇博客就讲解了怎么去使用这个包，并且提供了一个完整的可以工作的例子。

#### Context

context包的核心是`Context`类型：

```go
// A Context carries a deadline, cancellation signal, and request-scoped values
// across API boundaries. Its methods are safe for simultaneous use by multiple
// goroutines.
type Context interface {
    // Done returns a channel that is closed when this Context is canceled
    // or times out.
    Done() <-chan struct{}

    // Err indicates why this context was canceled, after the Done channel
    // is closed.
    Err() error

    // Deadline returns the time when this Context will be canceled, if any.
    Deadline() (deadline time.Time, ok bool)

    // Value returns the value associated with key or nil if none.
    Value(key interface{}) interface{}
}
```

(上面的注释精简了下，详细的可以看源码)

对于那些代表`Context`运行的函数，Done方法返回的chan相当于是一个取消信号：当通道关闭的时候，该函数应该立即放弃他的工作并且返回。`Err`方法返回一个错误，代表`Context`被取消的原因。[Pipelines and Cancellation](https://go.dev/blog/pipelines)一文详细讨论了`Done`通道的用法。

因为`Done`chan是receive-only的，所以`Context`类型没有`Cancel`函数：接受Cancel Signal的函数通常不是发送该信号的函数。尤其是，当一个赴操作为了一些子操作起了一些goroutine的时候，这些自操作不应该能够取消父操作中的Context。相反，`WithCancel`函数提供了一种方式来取消一个新的Context值。

`Context`能够同时被多个goroutine并发访问。我们可以在代码中将一个`Context`传递给很多goroutine并且取消该Context来通知所有的这些goroutine。

`Deadline`方法允许函数决定他们是不是该开始他们的工作。如果剩的时间太少了，可能并不值得启动一个goroutine去执行。当然，我们在代码中也可以为IO操作设立一个超时。

`Value`方法允许`Context`携带request级的数据。这些数据必须能够被多个goroutine安全的并发访问。

#### 导出Context

如果context包只包含上述一个类型当然还很不完整。让Context包完整的是Context包提供了函数来从现存Context中导出新的Context。这些值组成了一个Context树。当一个Context被取消的时候，所有从该Context导出的Context都会被取消。

`Background`是Context树的根，他永远不会被取消：

```go
// Background returns an empty Context. It is never canceled, has no deadline,
// and has no values. Background is typically used in main, init, and tests,
// and as the top-level Context for incoming requests.
func Background() Context
```

`WithCancel`和`WithTimeout`返回一个新的Context值，该值可以比父Context更快的被取消。`WithCancel`函数还可以用于取消冗余的请求。`WithTimeout`函数可以用于对后端服务器的请求设置deadline。

```go
// WithCancel returns a copy of parent whose Done channel is closed as soon as
// parent.Done is closed or cancel is called.
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)

// A CancelFunc cancels a Context.
type CancelFunc func()

// WithTimeout returns a copy of parent whose Done channel is closed as soon as
// parent.Done is closed, cancel is called, or timeout elapses. The new
// Context's Deadline is the sooner of now+timeout and the parent's deadline, if
// any. If the timer is still running, the cancel function releases its
// resources.
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```

`WithValue`函数给request级的数据传递提供了一种方式：

```go
// WithValue returns a copy of parent whose Value method returns val for key.
func WithValue(parent Context, key interface{}, val interface{}) Context
```

使用Context包最好的方式是通过一个可以运行的例子。

#### 例子: Google网络搜索

我们的例子是一个HTTP处理这样URL的`/search?q=golang&timeout=1s`的HTTP服务器。这个服务器通过将golang查询转发给[Google Web Search API](https://developers.google.com/custom-search)并且渲染结果。`timeout`参数告诉服务器在指定的时间过去后取消请求。

这些代码分布在三个包里：

+ server包提供了main函数和/search的handler
+ userip包提供了从用户请求中抽取IP地址的函数，并且将其与一个Context关联
+ google包提供了Search函数来向谷歌发起一个query

##### server程序

服务器程序处理像 `/search?q=golang` 这样的请求，通过提供前几个 `golang` 的谷歌搜索结果来处理。它注册 `handleSearch` 来处理 `/search` endpoint。处理程序创建一个初始的 `Context` ，名为 `ctx` ，并安排在处理程序返回时取消它。如果请求包括 `timeout` 的URL参数，当超时时间到达时， `Context` 会自动取消。

```go
func handleSearch(w http.ResponseWriter, req *http.Request) {
    // ctx is the Context for this handler. Calling cancel closes the
    // ctx.Done channel, which is the cancellation signal for requests
    // started by this handler.
    var (
        ctx    context.Context
        cancel context.CancelFunc
    )
    timeout, err := time.ParseDuration(req.FormValue("timeout"))
    if err == nil {
        // The request has a timeout, so create a context that is
        // canceled automatically when the timeout expires.
        ctx, cancel = context.WithTimeout(context.Background(), timeout)
    } else {
        ctx, cancel = context.WithCancel(context.Background())
    }
    defer cancel() // Cancel ctx as soon as handleSearch returns.
```

处理程序从请求中提取query，并通过调用 `userip` 包提取客户端的IP地址。后端请求需要客户端的IP地址，因此 `handleSearch` 将其附加到 `ctx` 中。

```go
    // Check the search query.
    query := req.FormValue("q")
    if query == "" {
        http.Error(w, "no query", http.StatusBadRequest)
        return
    }

    // Store the user IP in ctx for use by code in other packages.
    userIP, err := userip.FromRequest(req)
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    ctx = userip.NewContext(ctx, userIP)
```

处理程序使用`ctx`和`query`来调用`google.Search` ：

```go
    // Run the Google search and print the results.
    start := time.Now()
    results, err := google.Search(ctx, query)
    elapsed := time.Since(start)
```

如果搜索成功：处理程序将会呈现处理结果：

```go
    if err := resultsTemplate.Execute(w, struct {
        Results          google.Results
        Timeout, Elapsed time.Duration
    }{
        Results: results,
        Timeout: timeout,
        Elapsed: elapsed,
    }); err != nil {
        log.Print(err)
        return
    }
```

##### userip包

userip包提供了从请求中提取用户IP地址并将其与 `Context` 关联的功能。一个 `Context` 提供了键值映射，其中键和值都是 `interface{}` 类型。键类型必须支持相等性，值必须对多个goroutine的同时使用是安全的。像 `userip` 这样的包隐藏了这个映射的细节，并提供了对特定 `Context` 值的强类型访问。

为了避免键冲突，userip定义了一个未导出的类型key，并且使用该类型的值作为Context的键：

```go
// The key type is unexported to prevent collisions with context keys defined in
// other packages.
type key int

// userIPkey is the context key for the user IP address.  Its value of zero is
// arbitrary.  If this package defined other context keys, they would have
// different integer values.
const userIPKey key = 0
```

`FromRequest`从http.Request中抽取一个userIP出来：

```go
func FromRequest(req *http.Request) (net.IP, error) {
    ip, _, err := net.SplitHostPort(req.RemoteAddr)
    if err != nil {
        return nil, fmt.Errorf("userip: %q is not IP:port", req.RemoteAddr)
    }
```

`NewContext`返回一个新的`Context`，他携带了一个提供userIP值。

```go
func NewContext(ctx context.Context, userIP net.IP) context.Context {
    return context.WithValue(ctx, userIPKey, userIP)
}
```

`FromContext`从一个`Context`中抽取出一个userIP：

```go
func FromContext(ctx context.Context) (net.IP, bool) {
    // ctx.Value returns nil if ctx has no value for the key;
    // the net.IP type assertion returns ok=false for nil.
    userIP, ok := ctx.Value(userIPKey).(net.IP)
    return userIP, ok
}
```

##### google包

`google.Search`函数向Google Web Search API发出HTTP请求并解析JSON编码的结果。它会接受一个Context类型的参数ctx，并在ctx.Done关闭时立刻返回。

Google Web Search API request包括搜索query和user IP作为查询参数：

```go
func Search(ctx context.Context, query string) (Results, error) {
    // Prepare the Google Search API request.
    req, err := http.NewRequest("GET", "https://ajax.googleapis.com/ajax/services/search/web?v=1.0", nil)
    if err != nil {
        return nil, err
    }
    q := req.URL.Query()
    q.Set("q", query)

    // If ctx is carrying the user IP address, forward it to the server.
    // Google APIs use the user IP to distinguish server-initiated requests
    // from end-user requests.
    if userIP, ok := userip.FromContext(ctx); ok {
        q.Set("userip", userIP.String())
    }
    req.URL.RawQuery = q.Encode()
```

`Search`函数使用一个辅助函数`httpDo`来发出HTTP请求，并在请求或相应处理过程中，如果ctx.Done关闭则取消它。`Search`向`httpDo`传递一个闭包来处理HTTP响应。

```go
    var results Results
    err = httpDo(ctx, req, func(resp *http.Response, err error) error {
        if err != nil {
            return err
        }
        defer resp.Body.Close()

        // Parse the JSON search result.
        // https://developers.google.com/web-search/docs/#fonje
        var data struct {
            ResponseData struct {
                Results []struct {
                    TitleNoFormatting string
                    URL               string
                }
            }
        }
        if err := json.NewDecoder(resp.Body).Decode(&data); err != nil {
            return err
        }
        for _, res := range data.ResponseData.Results {
            results = append(results, Result{Title: res.TitleNoFormatting, URL: res.URL})
        }
        return nil
    })
    // httpDo waits for the closure we provided to return, so it's safe to
    // read results here.
    return results, err
```

`httpDo`函数在一个新的goroutine中发出http请求并且处理响应。如果`ctx.Done`在goroutine退出前关闭了，，那么它会取消该请求。

#### 为Context适配

许多服务器框架提供了用于传递请求范围值的包和类型。我们可以定义新的 `Context` 接口的实现，以在使用现有框架的代码和期望 `Context` 参数的代码之间建立桥梁。

例如，Gorilla的github.com/gorilla/context包允许处理程序通过将HTTP请求映射到键值对来关联数据与传入请求。在gorilla.go中，我们提供了一个实现，其 `Context` 方法返回与Gorilla包中特定HTTP请求关联的值。

其他软件包提供了类似于 `Context` 的取消支持。例如，Tomb提供了一个 `Kill` 方法，通过关闭 `Dying` 通道来发出取消信号。 `Tomb` 还提供了类似于 `sync.WaitGroup` 的等待这些goroutine退出的方法。在tomb.go中，我们提供了一个 `Context` 的实现，当其父 `Context` 被取消或提供的 `Tomb` 被终止时，该实现将被取消。

#### 结论

在Google，我们要求Go程序员将 `Context` 参数作为每个函数的第一个参数传递给进入和离开请求之间的调用路径上的每个函数。这样可以确保由许多不同团队开发的Go代码能够良好地互操作。它提供了对超时和取消的简单控制，并确保关键值（如安全凭据）在golang程序中正确的传递。

希望在 `Context` 上构建的服务器框架应该提供 `Context` 的实现，以在其包和期望 `Context` 参数的包之间建立桥梁。然后，它们的客户端库将接受来自调用代码的 `Context` 。通过为请求范围数据和取消操作建立一个共同的接口， `Context` 使包开发人员更容易共享和创建可扩展服务的代码。



### Context源码分析




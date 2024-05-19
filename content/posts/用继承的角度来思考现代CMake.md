---
author: Kuba Sejdak
title: 用继承的角度来思考现代CMake
date: 2023-02-14
summary: "When we link a target, we immediately inherit (get) its `INTERFACE` and `PUBLIC` properties, and make them our own with the access level specified in the linking command. This mechanism is similar to inheritance in C++, so it should be easy to understand."
slug: modern-cmake-targets-properties
tags:
    - CMake
    - DevOps
---
CMake是由Kitware的Bill Hoffman于2000年创建的。在过去的20年中，截至本文发布时，它不断发展，增加了新功能，并扩展了对第三方库的支持。但最重要的增加是在3.0版本中发布的，通常被称为“现代CMake”。尽管已经有5年多的时间了，但仍有许多程序员没有使用它们！

今天我将向您展示“现代CMake”中最伟大的功能之一，它几乎像C++继承一样运作。但在我们深入了解之前，让我简要解释一些基本概念。

文章目录：
- [[#Modern CMake = targets + properties|Modern CMake = targets + properties]]
- [[#设置目标的属性|设置目标的属性]]
- [[#使用和链接库的行为像继承一样|使用和链接库的行为像继承一样]]
- [[#例子1：避免头文件依赖|例子1：避免头文件依赖]]
- [[#Example 2: 定义只有头文件的库|Example 2: 定义只有头文件的库]]
- [[#总结|总结]]

### 现代 CMake = 目标（targets）+ 属性（properties）
Target是CMake中的基础概念，他和其他构建系统中job的概念类似。我们在日常开发中用到的最多的targets是可执行文件和库。下面是创建和使用他们的语法：
```cmake
add_executable(myExecutable
    main.cpp
)

add_library(libA
    sourceA.cpp
)

add_library(libB
    sourceB.cpp
    sourceB_impl.cpp
)

add_library(libC
    sourceC.cpp
)

target_link_libraries(myExecutable libA)
target_link_libraries(libA libB)
target_link_libraries(libB libC)
```
这是创建目标的经典方式。这里有一个名为 `myExecutable` 的可执行文件和三个库 `libA` 、 `libB` 和 `libC` 。目前我们知道，要构建 `myExecutable` ，我们必须链接 `libA` ，要构建 `libA` ，我们必须链接 `libB` ，要构建 `libB` ，我们必须链接 `libC` 。最后的`libC`不依赖其他库。

除了可执行文件和链接库，在CMake中还有其他可能的目标可以创建。比如我们也可以自定义target来仅仅执行一些命令：
```cmake
add_custom_target(firmware.bin
    COMMAND             ${CMAKE_OBJCOPY} -O binary firmware firmware.bin
    DEPENDS             firmware
    WORKING_DIRECTORY   "${CMAKE_BINARY_DIR}/bin"
)
```
在上面的代码片段中，我们定义了一个名为 `firmware.bin` 的自定义目标，其唯一目标是使用GNU objcopy从原始可执行文件创建一个BIN文件（如果您对objcopy不熟悉，可以将其视为对二进制文件的某种转换）。在这里，我们明确表示 `firmware.bin` 依赖于 `firmware` 。这是很自然的，因为要转换firmware文件，它必须已经构建完成！
>我们也可以省略DEPENDS行然后将`add_dependencies(firmware.bin firmware)`添加为单独的一个语句。但是看起来没有上面这么优雅。

每个目标都可以定义一些在合适的场景中会用到的属性。下面是最常见的一些：
+ 编译选项
+ 链接选项
+ 预处理选项
+ C/C++标志
+ include的目录

所有上述属性都存储在特殊的CMake变量中，并在给定目标出现在特定上下文中时自动被构建系统使用。例如，当目标被编译时，包含目录属性会自动添加到编译标志中。链接标志会在链接此目标时自动传递给链接器。你可能会说，所有这些属性要么是 `CFLAGS` 或 `LDFLAGS` ，所以没有必要将它们提取为单独的实体。但是我们很快就会看到，在现代CMake项目中，这种分离非常方便。


### 设置目标的属性
目标属性可以以至少几种方式设置。在CMake 3.x之前，我们可以设置原始的CMake标志（例如 `CMAKE_C_FLAGS` ， `CMAKE_LINKER_FLAGS` ）或使用面向目录的命令，在当前目录及其子目录中为所有目标设置给定属性。现代CMake引入了一组新的面向目标的命令，允许我们为各个目标设置属性。

下面是老版本面向目录的命令和新的面向target命令的一个对比：
| **DIRECTORY-ORIENTED OLD COMMANDS**        | **TARGET-ORIENTED NEW COMMANDS**                             |
| ------------------------------------------ | ------------------------------------------------------------ |
| `include_directories(<include_path>)`      | `target_include_directories(<target> [VISIBILITY] <include_path>)` |
| `add_definitions(<preprocessor_flags>)`    | `target_compile_definitions(<target> [VISIBILITY] <preprocessor_flags>)` |
| `set(CMAKE_CXX_FLAGS <compilation_flags>)` | `target_compile_options(<target> [VISIBILITY] <compilation_flags>)` |
| `set(CMAKE_LINKER_FLAGS <linker_flags>)`   | `target_link_options(<target> [VISIBILITY] <linker_flags>)`  |

现代CMake也引入了心得关键字来说明给定目标属性的可见性：`PRIVATE`, `PUBLIC`, `INTERFACE`。他们的含义如下：

+ PRIVATE属性只被属性的所有者所有，为内部用
+ PUBLIC属性可以被属性的所有者使用，也可以给链接了属性所有者的其他目标使用
+ INTERFACE属性只被其他的库使用

这与C++类中的访问修饰符非常相似！每个新命令都允许指定可见性。如果没有提供，则默认为 `PUBLIC` 。请注意，每个命令可以在不同的可见性级别上设置多个属性。

```cmake
target_include_directories(<target>
    INTERFACE <include_path_1> <include_path_2> <include_path_3>
    PUBLIC <include_path_4> <include_path_5> <include_path_6>
    PRIVATE <include_path_7> <include_path_8> <include_path_9>
)
```

> 不生成任何二进制文件的目标（例如仅包含头文件的库）只能具有 `INTERFACE` 属性，并且只能使用 `INTERFACE` 链接。这是可以理解的，因为这类目标中没有“内部”部分，所以 `PRIVATE` 关键字没有任何意义。

### 使用和链接库的行为像继承一样

为了链接库，我们使用如下的表达式：
```cmake
target_link_libraries(<TARGET_A> <TARGETS...>)
```

现代CMake为这个表达式扩充了可见性修饰符：

```cmake
target_link_libraries(<TARGET_A> [VISIBILITY] <TARGETS...>)
```

再次，可见性可以是 `PRIVATE`、`PUBLIC`和`INTERFACE` 之一。如果没有提供任何选项，则默认使用 `PUBLIC` 。但是如果是链接，这些可见性修饰符又意味着什么呢？

CMake 3.x引入了一个非常重要的“副作用”，即与目标链接的链接目标将其所有的 `PUBLIC` 和 `INTERFACE` 属性传递给所链接的库。因此，例如，如果 `libA` 与 `libB` 链接，那么 `libA` 将获得 `libB` 的所有 `PUBLIC` 和 `INTERFACE` 属性。但 `PRIVATE` 属性仍然无法访问。另一个问题是：通过 `libA` 获得的属性集的可见性是什么？答案很简单：它与在 `target_link_libraries()` 中为该目标使用的说明符相同。因此，如果 `libA` 以 `PRIVATE` 的方式与 `libB` 链接，那么 `libB` 的所有 `PUBLIC` 和 `INTERFACE` 属性将成为 `libA` 的 `PRIVATE` 属性。同样地，如果以 `PUBLIC` 的方式链接，那么 `libB` 的所有 `PUBLIC` 和 `INTERFACE` 属性将在 `libA` 中成为 `PUBLIC` 。对于 `INTERFACE` 的链接也是如此。

你现在能看到了吗，这几乎和C++中的继承一模一样吧？私有继承会将派生类中的所有公有和保护成员变为私有，而公有继承则保持可见性不变。

为了更好地理解，让我们看一些使用案例。



### 例子1：避免暴露私有实现

我们假设有如下的的目录结构：
```shell
libA/
    - include/
        - libA/
            - sourceA.h
    - privateHeaderA1.h
    - privateHeaderA2.h
    - sourceA.cpp
libB/
    - include/
        - libB/
            - sourceB.h
    - submodule/
        - submodule.h
        - submodule.cpp
    - privateHeaderB1.h
    - privateHeaderB2.h
    - sourceB.cpp
    - sourceB_impl.h
    - sourceB_impl.cpp
libC/
    - include/
        - libC/
            - sourceC.h
    - privateHeaderC1.h
    - privateHeaderC2.h
    - sourceC.cpp
main.cpp
```

并且在代码中有如下依赖关系：
```cpp
// sourceB.cpp

#include "libC/sourceC.h"
#include "submodule.h"

// ...
```

```cpp
// sourceA.h

#include "libB/sourceB.h”

// ...
```

```cpp
// main.cpp

#include "libA/sourceA.h"
#include "libC/sourceC.h"

// ...
```

还有，让我们定义一个规则，我们不希望任何库能够使用其他库的“私有”头文件：例如，下面的代码不应该通过编译：

```cpp
// main.cpp

#include "privateHeaderC2.h” // should fail as "no such file or directory"
```

这个限制是一种良好的架构实践，可以使代码免受不必要的依赖关系的干扰。如何为这个项目架构写一个干净的cmake配置呢？

首先，我们需要检查每个目标并确定它所“创建”的包含路径。我的意思是，哪些包含路径属于这个特定的目标。然后，对于给定目标中的每个路径，我们需要决定它是否应该对其他人可访问（ `PUBLIC` ）或不可访问（ `PRIVATE` ）。最后，我们将使用新的面向目标的命令来设置每个库的包含属性。

```cmake
add_executable(myExecutable
    main.cpp
)
```

```cmake
add_library(libA
    sourceA.cpp
)
target_include_directories(libA
    PUBLIC include
    PRIVATE .                 # "dot" is redundant, because local headers are always available in C/C++.
)
```

```cmake
add_library(libB
    sourceB.cpp
    submodule/submodule.cpp
)

target_include_directories(libB
    PUBLIC include
    PRIVATE . submodule/      # "dot" is redundant, because local headers are always available in C/C++.
)
```

```cmake
add_library(libC
    sourceC.cpp
)

target_include_directories(libC
    PUBLIC include
    PRIVATE .                 # "dot" is redundant, because local headers are always available in C/C++.
)
```

所有这些目标都有一个共同点：唯一的 `PUBLIC` 包含路径是 `include` 目录。这意味着，如果其他库只调用 `target_link_libraries()` 来获取包含路径并链接库，那么私有头文件将永远不会意外地泄露到包含库之外。

现在，是时候来链接这些库文件了：
```cmake
target_link_libraries(myExecutable
    PRIVATE libA libC
)

target_link_libraries(libA
    PUBLIC libB
)

target_link_libraries(libB
    PRIVATE libC
)
```

有一下内容需要注意：

1. 可执行文件不需要指定链接类型（因为没有任何东西可以与可执行文件链接），但我们为了一致性而定义它。
2. `libA` 与 `libB` PUBLIC链接，因为它在自己的公共头文件中使用了 `libB` 的头文件。因此，它必须向其客户提供此路径。
3. `libB`与`libC`PRIVATE链接，因为它只在内部实现中使用了`libC`的头文件，而且其用户不应该知道这个实现细节。
4. 在现代CMake中， `target_link_libraries()` 有两个含义：在编译阶段使用库（获取其属性），并在链接阶段与之链接。因此，也许给它一个更好的名称是 `target_use_libraries()` ，但这将破坏向后兼容性。



### Example 2: 定义只有头文件的库

有时候我们不得不处理一些不生成任何二进制文件的库。例如，它们只是一组只需要包含在你应用程序中的头文件。在这种情况下，它们被称为"header-only libraries"。

一个很好的例子就是Catch2库，它是C++流行的测试框架。它只包含一个头文件 `catch.hpp` ，存储在 `catch2` 目录中。首先，对于我们来说，仍然有一个CMaketarget需要提供该文件的路径，以便当有人链接它时可以方便使用。其次，Catch2允许通过定义指令进行一些行为定制。例如，我们可以禁用POSIX信号和异常的使用，而使用 `std::terminate()` 。这在嵌入式系统中尤为重要，因为我们无法使用异常和POSIX信号。因此，我们的目标还需要能够检测环境并相应地提供适当的定义。

在现代CMake中，可以如下表达：
```cmake
add_library(catch2 INTERFACE)

target_include_directories(catch2
    INTERFACE catch2
)

if (<some_condition_to_detect_embedded_platform>)
    target_compile_definitions(catch2
        INTERFACE CATCH_CONFIG_NO_POSIX_SIGNALS CATCH_CONFIG_DISABLE_EXCEPTIONS
    )
endif ()
```

注意 `INTERFACE` 关键字的使用。当 `add_library()` 包含 `INTERFACE` 说明符时，它告诉CMake，该目标不会生成任何二进制文件。在这种情况下，它不包含任何源文件。

如前所述， `INTERFACE` target的所有属性也必须标记为 `INTERFACE` 。这很好理解：因为header-only库没有任何私有实现。一切都始终对使用者可访问。如果这对您来说仍然令人困惑，那么请记住， `INTERFACE` 目标强制执行 `INTERFACE` 属性。但稍后与此类目标的链接可以是任何类型：

```cmake
add_library(myTestingModule source.cpp)

target_link_libraries(myTestingModule
    PRIVATE catch2
)
```



### 总结

CMake 提供了一种新的以target为导向的方式来指定各种编译器选项和其他属性。一旦你与一个目标进行链接，你立即继承（获取）它的 `INTERFACE` 和 `PUBLIC` 属性，并通过链接命令中指定的访问级别使其成为你自己的。这个机制类似于 C++ 的继承，因此应该很容易理解。

> **如果您正在使用CMake 3.x及以上版本，请按照以下规则创建目标：**
>
> 现代CMake目标的设计应该考虑到为其使用者提供一切所需的内容，但是使用者不能知道其内部实现细节。

如果你发现你在访问某个库中没有提供的包含路径，则意味着有什么错误发生了：

+ 要么是那个库的CMake配置有问题
+ 要么是你试图访问故意向你隐藏的头文件


+++
author = "Hwa"
title = "Python 代码风格指南"
tags = [
    "Python",
    "PEP8",
    "Style Guide"
]
date = "2023-04-14"
summary = "style guide excerpt from PEP8 and PEP257 for Python"
+++

## Python 代码风格指南

### 1 PEP8(Python代码风格指南)

#### 1.1 愚蠢的坚持一致性是可笑的妖怪

我们需要代码风格一致，但是始终要记住："Readability Counts"。为了可读性适当的破坏一致性是合适的。老代码也无需为了符合PEP8做重构。



#### 1.2 代码布局

##### 1.2.1 缩进

使用4个空格做缩进。

两种风格，第一种，使用竖直对齐：

```python
# Aligned with opening delimiter.
foo = long_function_name(var_one, var_two,
                         var_three, var_four)
```

第二种，使用悬挂缩进：

```python
# Add 4 spaces (an extra level of indentation) to distinguish arguments from the rest.
def long_function_name(
        var_one, var_two, var_three,
        var_four):
    print(var_one)
```

但是需要注意，悬挂缩进需要增加一级缩进（空格数目是可选的）来区分。

另外需要注意的是：对于if语句这种两字符的关键字，加一个空格，加一个括号，会形成一个自然的4空格缩进。对于（需要分行的）长条件语句来说，这样会很难区分条件语句和代码，本规范对如何处理这种代码没有明确的规定。可以接受的有：

```python
# No extra indentation.
if (this_is_one_thing and
    that_is_another_thing):
    do_something()

# Add a comment, which will provide some distinction in editors
# supporting syntax highlighting.
if (this_is_one_thing and
    that_is_another_thing):
    # Since both conditions are true, we can frobnicate.
    do_something()

# Add some extra indentation on the conditional continuation line.
if (this_is_one_thing
        and that_is_another_thing):
    do_something()
```

对于多行构造语句怎么放置最后一个括号，以下两种风格都是可以接受的：

```python
# 和多行构造的元素对齐
my_list = [
    1, 2, 3,
    4, 5, 6,
    ]
result = some_function_that_takes_arguments(
    'a', 'b', 'c',
    'd', 'e', 'f',
    )
```

```python
# 放在在变量的同一个缩进级别
my_list = [
    1, 2, 3,
    4, 5, 6,
]
result = some_function_that_takes_arguments(
    'a', 'b', 'c',
    'd', 'e', 'f',
)
```

##### 1.2.2 使用Tab还是空格键缩进？

使用空格。如果代码已经是使用Tab缩进的，那么使用Tab缩进。不允许混合使用Tab和空格。

##### 1.2.3 最大行长度

限制最大行长度到79个字符，限制文档注释的最大长度为72个字符。

如果团队内能够就最大行长度达成一致，那么使用79～99（包含）个字符都是合适的。但是Python标准库限制为79个字符。

此外，优先使用括号来隐式的连接多行，非必要不使用反斜线（"\\"）。除非在Python 3.10之前的版本的with-resources语句：

```python
with open('/path/to/some/file/you/want/to/read') as file_1, \
     open('/path/to/some/file/being/written', 'w') as file_2:
    file_2.write(file_1.read())
```

##### 1.2.4 在双目运算符前面还是后面换行？

Donald Knuth说过：“虽然段落中的公式总是在双目运算符后面换行，但是显示的公式应该在双目运算符前面换行”。也就是说，为了可读性更好，推荐在双目运算符前面换行：

```python
# Correct:
# easy to match operators with operands
income = (gross_wages
          + taxable_interest
          + (dividends - qualified_dividends)
          - ira_deduction
          - student_loan_interest)
```

风格规范没有明确规定是在前面还是后面，但是新代码推荐在前面换行。

##### 1.2.5 空行

顶层的行数和类间隔两行。

类中的方法间隔一行。

使用尽可能少的多余空行来分隔一组相关的行数。

对于一些相关的一行实现，可以省略空行。

在函数中使用空行来分割逻辑上相关的代码。

可以使用`Ctrl + L`来提示一些工具进行文件内的分页。（不是必须，因为有些基于Web的编辑器可能会不支持这种方式，进而显示一些其他的字形）

##### 1.2.6 源代码编码方式

Python核心库应该永远使用UTF-8编码，而且不需要写编码声明。在标准库中，是允许在测试的时候使用非UTF-8编码。尽可能少的使用非ASCII码，除了标识地名或者人名。避免使用一些很乱的Unicode编码，如：z̯̯͡a̧͎̺l̡͓̫g̹̲o̡̼̘ 和字节序标记。

Python标准库中所有的标识符都应该只使用ASCII编码，而且尽可能的用英语单词而不是一些缩写和技术词汇中的非英语单词音译。

开源项目也需要尽可能的遵循这些编码规范。

##### 1.2.7 import语句

+ 分行导入

```python
# Correct:
import os
import sys

# Wrong:
import sys, os
```

但是这样是没问题的：

```python
# Correct:
from subprocess import Popen, PIPE
```

+ import语句应该放在文件的头部，在模块级注释和文档字符串后，在全局变量和常量之前，而且应该以如下的顺序组织：

	1. 标准库导入
	1. 相关三方库导入
	1. 本地库导入

并且应该使用空行分开这三组导入。

+ 推荐使用绝对导入，因为他们可读性更好，而且报错信息也更友好。

  ```python
  import mypkg.sibling
  from mypkg import sibling
  from mypkg.sibling import example
  ```

  然而，相对导入也是可以的，如果项目结构比较复杂：

  ```python
  from . import sibling
  from .sibling import example
  ```

  但是标准库也应该尽可能的避免复杂的包文件结构。

+ 当从一个包含类的模块导入一个类的时候，可以这个样子：

  ```python
  from myclass import MyClass
  from foo.bar.yourclass import YourClass
  ```

  如果可能会命名冲突，那么使用全称导入：

  ```python
  import myclass
  import foo.bar.yourclass
  ```

  然后使用全称限定名去访问。

+ 避免使用通配符。除非是将一组内部API作为公共API重新发布，并且具体哪些API会被覆盖掉不明确的情况下。

##### 1.2.8 模块级Dunder Name

模块级的dunder应该始终放在模块级注释或文档字符串的后面，import之前（除了`from __future__` import，因为Python强制`from __future__` import应该在任何代码之前）。

```python
"""This is the example module.

This module does stuff.
"""

from __future__ import barry_as_FLUFL

__all__ = ['a', 'b', 'c']
__version__ = '0.1'
__author__ = 'Cardinal Biggles'

import os
import sys
```

#### 1.3 字符串引号

在Python中，单引号和双引号字符串没有任何差别。坚持使用一种，然后用另一种来比避免字符串里的backslashes就行。

对于三引号，在里面应该使用双引号来和PEP257保持一致。

#### 1.4 表达式和语句中的空格

##### 1.4.1 忌讳

+ 在括号里直接相邻接的地方不应该有空格

```python
# Correct:
spam(ham[1], {eggs: 2})

# Wrong:
spam( ham[ 1 ], { eggs: 2 } )
```

+ 在结尾的逗号和一个反括号间也不要空格

```python
# Correct:
foo = (0,)

# Wrong:
bar = (0, )
```

+ 在逗号，分号和冒号前都不要空格

```python
# Correct:
if x == 4: print(x, y); x, y = y, x

# Wrong:
if x == 4 : print(x , y) ; x , y = y , x
```

+ 针对上一条有个例外，对于slice中的冒号，我们应该将他当作一个双目运算符，所以应该在两边空同样多的空格。如果我们省略了一个操作数，那么我们也可以省略一边的空格。

```python
# Correct:
ham[1:9], ham[1:9:3], ham[:9:3], ham[1::3], ham[1:9:]
ham[lower:upper], ham[lower:upper:], ham[lower::step]
ham[lower+offset : upper+offset]
ham[: upper_fn(x) : step_fn(x)], ham[:: step_fn(x)]
ham[lower + offset : upper + offset]

# Wrong:
ham[lower + offset:upper + offset]
ham[1: 9], ham[1 :9], ham[1:9 :3]
ham[lower : : upper]
ham[ : upper]
```

+ 函数调用的参数列表的其实括号前不应该空格

```python
# Correct:
spam(1)

# Wrong:
spam (1)
```

+ index和slice的中括号前都不应该空格
+ 不要为了对齐赋值运算符加很多空格

```python
# Correct:
x = 1
y = 2
long_variable = 3

# Wrong:
x             = 1
y             = 2
long_variable = 3
```

##### 1.4.2 其他推荐

+ 不要到处加结尾的空格，因为它是不可见的，所以很confusing。比如说如果一个backslash后面加了一个空格就不算换行标记了。而且有些项目的pre-commit hooks会明确拒绝这种编码风格。
+ 对于下面的这些双目运算符，应该尽可能的在其左右两边加一个单空格：赋值（=），比较（==, <, >, !=, <>, <=, >=, in, not in, is, is not），布尔运算（and, or, not）。
+ 在使用不同优先级的operator时，应该在优先级最低的operator两边加上等量单个空格。
+ 函数注解的冒号应该使用一般的规则，而且总是应该在`->`箭头的两端放上空格。

```python
# Correct:
def munge(input: AnyStr): ...
def munge() -> PosInt: ...

# Wrong:
def munge(input:AnyStr): ...
def munge()->PosInt: ...
```

+ 当赋值符号表示一个关键字参数或者未注解的函数默认值的时候，不要在赋值运算符两边加空格。

然而，当我们使用注解标注函数参数类型的时候，应该在赋值运算符两端加空格：

```python
# Correct:
def munge(input: AnyStr): ...
def munge() -> PosInt: ...
# Correct:
def munge(sep: AnyStr = None): ...
def munge(input: AnyStr, sep: AnyStr = None, limit=1000): ...


# Wrong:
def complex(real, imag = 0.0):
    return magic(r = real, i = imag)
# Wrong:
def munge(input: AnyStr=None): ...
def munge(input: AnyStr, limit = 1000): ...
```

+ 不要将复合语句放在同一行上：

```python
# Correct:
if foo == 'blah':
    do_blah_thing()
do_one()
do_two()
do_three()

# Wrong:
if foo == 'blah': do_blah_thing()
do_one(); do_two(); do_three()
```

+ 有时候将if/for/while语句只有一行时放在同一行上是ok的，但是对于多个自居的statement，绝对不要这么做：

```python
# Wrong: 有时候是ok的，但是不要这么密集
if foo == 'blah': do_blah_thing()
for x in lst: total += x
while t < 10: t = delay()

# Wrong:
if foo == 'blah': do_blah_thing()
else: do_non_blah_thing()

try: something()
finally: cleanup()

do_one(); do_two(); do_three(long, argument,
                             list, like, this)

if foo == 'blah': one(); two(); three()
```

#### 1.5 什么时候使用行尾的逗号

一般是不需要行尾的逗号的，除非我们需要表示一个元组，而这种情况下，推荐把括号加上。

当我们把一个列表或者集合里的元素一个写一行时，推荐在行尾加上逗号，因为这样在版本控制系统上好看一些。而单行的集合定义就没必要了。

```python
# Correct:
FILES = [
    'setup.cfg',
    'tox.ini',
    ]
initialize(FILES,
           error=True,
           )


# Wrong:
FILES = ['setup.cfg', 'tox.ini',]
initialize(FILES, error=True,)
```



#### 1.6 注释

和代码冲突的注释比没有注释更差，应该永远保证优先更新注释。注释应该是完整的句子，且第一个字母大写（除了专有名词）。块级注释应该包含一个或多个段的完整句子，并且每个句子以一个句号结尾。在一个段落里的非最后一个句子的句号后都应该加两个空格。保证你写的注释对阅读的人来说是清晰的和简单的。非英语国家的Python coder应该尽可能的写英语注释，除非代码永远不可能被英语读者阅读到。

##### 1.6.1 块级注释

块级注释以`#`开始且仅跟一个空格，段落间以一个`#`开始的空行分隔开。

##### 1.6.2 行内注释

少用行内注释，如果代码很简明。

##### 1.6.3 文档注释

见PEP257。

+ 应当始终为公有模块，函数，类和方法写注释。非公有的不强制，但也应该有依据注释解释干了啥。
+ 多行注释最后的三引号应该单独占一行。
+ 单行文档注释不需要最后的三引号单独占一行。



#### 1.7 命名习惯

众所周知，Python的很多库代码命名习惯都不一样，而且也永远不可能一样。但是新代码推荐遵守下面的命名习惯。（虽然这样说，内部一致性还是更重要）

##### 1.7.1 最最最重要的原则

公开接口的命名应该展示用途而不是展示实现。

##### 1.7.2 一些命名风格

+ `b` 单字母命名
+ `B` 单字母但是大写
+ `lowercase`
+ `lower_case_with_underscores`
+ `UPPERCASE`
+ `CapitalizedWords`
+ `mixedCase`(camel case with initial lowercase character)
+ `Never_Name_Like_This`(as it's ugly)

除此之外，还有些命名风格给变量名加前缀来将一些变量名组合在一起。比如`os.stat()`返回的`st_mode, st_size, st_mtime`前的`st`表示返回的是符合POSIX规范的struct。

+ `_single_leading_underscore`: 表示很弱的internal use
+ `single_trailing_underscore_`: 为了避免和python的内置关键字冲突

```python
tkinter.Toplevel(master, class_='ClassName')
```

+ `__double_leading_underscore`: 当表示一个类的属性时，会触发name mangling。
+ `__double_leading_and_trailing_underscore__`：表示魔法objects。

##### 1.7.3 指导性的命名习惯

+ 绝对不要用单字母'l', 'O', 'I'来命名变量。
+ 标识符只用ASCII字符。
+ 模块名和包名应该用短的全小写的字母，尽量不要用下划线（只在确实能够提高可读性的时候用）。如果有C/C++写的底层库，那么应该给C/C++模块用前导下划线标识。
+ 类名一般用`CapitalizedWords`，除非我们写的类是个callable。那么我们则用函数的命名习惯（见后文）。
+ 类型变量名（PEP484）应该尽可能的用`CapitalizedWords`。如果有逆变和协变变量，则可以在类型名后加`_co`和`contra`后缀。

```python
from typing import TypeVar

VT_co = TypeVar('VT_co', covariant=True)
KT_contra = TypeVar('KT_contra', contravariant=True)
```

+ 异常名。因为异常一般来说是个类，所以应该用类的命名习惯。并且如果是个Error就加一个`Error`后缀。
+ 全局变量名。如果全局变量只是module内使用，那么应该用`__all__`变量来防止意外导出。或者使用`_leading_underscore`来表示internal use。
+ 函数名和变量名。使用`lowercase_with_underscore`来提高可读性。
+ 函数和方法参数。对于实例方法，使用`self`作为第一个参数；对于类方法，使用`cls`作为第一个参数；对于与关键词冲突的函数名，最好在变量名后面加个`_`。
+ 方法名和实例变量名称。对于private的变量，使用`_leading_underscore`，对于protected的变量，使用`__double_leading_underscore`。
+ 常量应该在模块级别书写，并且遵从`ALL_CAPITAL_WORDS_WITH_UNDERSCORE`。
+ 面向对象的设计。

1. 对于private的变量，使用`_leading_underscore`，对于protected的变量，使用`__double_leading_underscore`。
2. 尽可能的缩小权限。例如如果不确定是公有还是私有，最好弄成私有。
3. 公有属性前面不要加下划线。
4. 如果属性名可能会和保留字冲突，那么后面追加一个下划线。
5. 对于简单的属性访问，不要用accessor或者mutator，Python有很方便的方式（properties）来让我们增强一个属性的行为。但是也不要在properties里进行复杂的计算，或者造成一些side effect。因为人们期望属性访问是cheap的。

##### 1.7.4 公有的和私有的接口

1. 保证公开接口的反向兼容性
2. 凡是写文档的接口都应该认为是公开的。除非文档显式说明了API是可能变动的。所有没写文档的都应该认为是非公开的API。
3. 应该使用设置`__all__`来告知公开的API。。
4. 即使使用`__all__`设置了哪些变量是公开的，我们的包名、模块名、类名、函数名、属性名或者其他的名字都应该以单下划线开头。
5. 如果外部结构是internal use的，那应该认为里面所有的API都是internal use的。
6. 不要依赖间接引入的东西，因为这些是和实现强相关的。除非我们使用`__init__` modulelai 显式的从子module暴露了一些API。

#### 1.8 一些推荐的编程习惯

+ 代码不应该依赖于其他的Python实现，比如CPython提供了效率很好的字符串拼接，但是其他的实现如PyPy，Jython, IronPython等就不一定了。对于比较多的字符串拼接，可以使用`''.join`来保证线性效率。

+ 与单例的比较使用`is`和`is not`。

+ 使用`is not`而不是`not ... is`。

+ 当实现ordering operation时，最好六个运算符都实现一遍，而不是依赖于Python的hooks来完成。

  为了减少工作量，也可以使用`functools.total_ordering()`装饰器来提供缺失的比较方法。

  PEP207阐述了Python假设的自反性。

+ 尽量用def声明而不是lambda赋值。因为debug的时候信息会更好一些。如果使用lambda赋值的话就丧失了lambda的唯一好处（可以应用在一个更大的表达式中）。

+ 从`Exception`而不是`BaseException`继承异常。因为从`BaseException`继承的异常一般认为是不可捕获的。给异常取名的时候，要体现出是什么出了问题，而不是告诉使用者出了一个问题。

+ 使用`raise X from Y`来显式的进行异常替换。这样能够保存堆栈信息和异常信息。

+ 当我们进行异常捕捉的时候，尽可能指出具体的异常是什么。而不仅仅是一个`exceptr: `语句。后一种情况下在两种情况下可以采用：

  1. 如果我们进行全局异常捕捉，并且会log出所有异常。至少用户知道发生了一个异常，而不应该静默处理。
  2. 如果我们需要做一些cleanup work。那么可以使用该种风格，并且将捕捉到的异常通过`raise`给propagate出去。

+ 当捕捉系统异常的时候，选择Python 3.3版本引入的异常继承体系。而不是`errno`。

+ try-catch语句应该包含尽可能少的语句。

+ 使用with-resources来确保资源被合理的清理掉了。

+ 上下文管理器如果做的不仅仅是资源的获取及释放，那么应该将他做的其他的事通过一个函数表示出来。比如：

```python
# Correct:
with conn.begin_transaction():
    do_stuff_in_transaction(conn)
    
# Wrong:
with conn:
    do_stuff_in_transaction(conn)
```

+ 要保证return的一致性。也就是说不能有分支不返回任何东西。
+ 使用`''.startswith()`和`''.endswith()`来检查prefix和suffix，而不要用切片。
+ 使用`isinstance`而不是`type(...) is type(...)`。
+ 对于序列，在布尔环境中空序列代表false。
+ 在`finally`块中不要用`return/break/continue`这样的流程控制语句。因为他们会隐式的丢掉异常信息。

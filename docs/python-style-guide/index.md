# Google Python 风格指南

## 背景

Python 是 Google 主要使用的动态语言。这个风格指南是 Python 程序的*做*和*不做*的列表。

为了帮助你正确格式化代码，我们创建了一个 [Vim 的设置文件](google_python_style.vim)。对于 Emacs，使用默认设置即可。

许多团队使用 [Black](https://github.com/psf/black) 或 [Pyink](https://github.com/google/pyink) 自动格式化程序，以避免关于格式的争论。

## Python 语言规则

### 代码审查

对你的代码运行 `pylint`，使用这个 [pylintrc](https://google.github.io/styleguide/pylintrc)。

#### 定义

`pylint` 是一个用于在 Python 源代码中查找错误和样式问题的工具。它发现通常只有像 C 和 C++ 这样的不太动态的语言编译器才能发现的问题。由于 Python 的动态特性，一些警告可能是不正确的；然而，虚假的警告应该相当少见。

#### 优点

捕获易于忽视的错误，如拼写错误、在赋值之前使用变量等。

#### 缺点

`pylint` 并不完美。为了充分利用它，有时我们需要绕过它、抑制其警告或修复它。

#### 决策

确保对你的代码运行 `pylint`。

如果警告不合适，则抑制它们以避免隐藏其他问题。要抑制警告，你可以设置一行级别的注释：

```python
def do_PUT(self):  # WSGI 名称，因此禁用 pylint: invalid-name
  ...
```

每个 `pylint` 警告都用符号名称（`empty-docstring`）进行标识。以 `g-` 开头的警告是 Google 特定的警告。

如果不清楚抑制的原因，请添加解释。

以这种方式进行抑制的优点是我们可以轻松搜索抑制并重新访问它们。

你可以通过执行以下命令获取 `pylint` 警告列表：

```shell
pylint --list-msgs
```

要获取有关特定消息的更多信息，请使用：

```shell
pylint --help-msg=invalid-name
```

请优先使用 `pylint: disable`，而不是不推荐的旧形式 `pylint: disable-msg`。

未使用的参数警告可以通过删除函数开头的变量来抑制。请始终包含解释为什么要删除它的注释。“Unused” 就足够了。例如：

```python
def viking_cafe_order(spam: str, beans: str, eggs: str | None = None) -> str:
    del beans, eggs  # Unused by vikings.
    return spam + spam + spam
```

抑制此警告的其他常见形式包括使用 `'_'` 作为未使用参数的标识符，或者将参数名称前缀设为 `'unused_'`，或者将它们赋值给 `'_'`。这些形式是允许的，但不再鼓励使用。这些会破坏通过名称传递参数并且不强制执行参数实际上未使用的调用方。

### 导入

只为包和模块使用 `import` 语句，而不是为单个类型、类或函数使用。

#### 定义

从一个模块中将代码共享到另一个模块中的可重用机制。

#### 优点

命名空间管理约定简单。每个标识符的源都以一致的方式指示；`x.Obj` 表示对象 `Obj` 定义在模块 `x` 中。

#### 缺点

模块名仍然可能冲突。一些模块名过长。

#### 决策

- 使用 `import x` 导入包和模块。
- 使用 `from x import y` 其中 `x` 是包前缀，`y` 是无前缀的模块名。
- 在以下任何情况下使用 `from x import y as z`：
  - 两个名为 `y` 的模块需要导入。
  - `y` 与当前模块中定义的顶级名称冲突。
  - `y` 与公共 API 的常见参数名称冲突（例如 `features`）。
  - `y` 是一个不方便的长名称。
  - 在你的代码上下文中，`y` 太泛化（例如，`from storage.file_system import options as fs_options`）。
- 仅在 `z` 是标准缩写时使用 `import y as z`（例如，`import numpy as np`）。

例如，模块 `sound.effects.echo` 可以这样导入：

```python
from sound.effects import echo
...
echo.EchoFilter(input, output, delay=0.7, atten=4)
```

不要在导入中使用相对名称。即使模块位于同一个包中，也要使用完整的包名称。这有助于防止意外导入包两次。

#### 例外

- 以下模块中的符号用于支持静态分析和类型检查：
  - [`typing` 模块](#typing-imports)
  - [`collections.abc` 模块](#typing-imports)
  - [`typing_extensions` 模块](https://github.com/python/typing_extensions/blob/main/README.md)
- 来自 [six.moves 模块](https://six.readthedocs.io/#module-six.moves) 的重定向。

### 包

使用模块的完整路径名来导入每个模块。

#### 优点

避免模块名冲突或由于模块搜索路径不是作者预期的那样而导致的不正确导入。使查找模块更容易。

#### 缺点

使代码部署更困难，因为你必须复制包层次结构。这在现代部署机制中不是真正的问题。

#### 决策

所有新代码应该通过完整的包名称导入每个模块。

导入应该如下所示：

```python
正确:
  # 在代码中使用完整名称（冗长）引用 absl.flags。
  import absl.flags
  from doctor.who import jodie

  _FOO = absl.flags.DEFINE_string(...)
```

```python
正确:
  # 在代码中只使用模块名（常见）引用 flags。
  from absl import flags
  from doctor.who import jodie

  _FOO = flags.DEFINE_string(...)
```

*(假设此文件位于 `doctor/who/`，其中也存在 `jodie.py`)*

```python
错误:
  # 不清楚作者想要什么模块以及将要导入什么模块。实际导入行为取决于控制 sys.path 的外部因素。
  # 作者打算导入哪个可能的 jodie 模块？
  import jodie
```

不能假定主二进制文件所在的目录在 `sys.path` 中，尽管在某些环境中确实会发生这种情况。这种情况下，代码应该假设 `import jodie` 指的是名为 `jodie` 的第三方或顶级包，而不是本地的 `jodie.py`。

### 异常

异常是允许的，但必须小心使用。

#### 定义

异常是一种打破正常控制流以处理错误或其他异常情况的方法。

#### 优点

正常操作代码的控制流不会被错误处理代码混淆。它还允许在发生某些特定条件时跳过多个框架，例如，在一步中从 N 个嵌套函数返回而不是通过错误代码传递。

#### 缺点

可能会导致控制流变得混乱。在调用库时容易忽略错误情况。

#### 决策

异常必须遵循一定的条件：

- 在合适的情况下使用内置异常类。例如，当发生编程错误（如违反先决条件）时，抛出 `ValueError`。

- 不要将 `assert` 语句用于条件或验证前提条件。它们不应该对应用程序逻辑至关重要。一个检验条件的试金石是可以删除 `assert` 而不会破坏代码。`assert` 条件[不能保证](https://docs.python.org/3/reference/simple_stmts.html#the-assert-statement)被评估。对于基于 [pytest](https://pytest.org) 的测试，`assert` 是可以的，并且期望验证期望。例如：

  
  ```python
  正确:
    def connect_to_next_port(self, minimum: int) -> int:
      """Connects to the next available port.

      Args:
        minimum: A port value greater or equal to 1024.

      Returns:
        The new minimum port.

      Raises:
        ConnectionError: If no available port is found.
      """
      if minimum < 1024:
        # 请注意，此处引发 ValueError 没有在文档字符串的 "Raises:" 部分中提到，因为保证了对 API 滥用的具体行为反应不适当。
        raise ValueError(f'Min. port must be at least 1024, not {minimum}.')
      port = self._find_next_open_port(minimum)
      if port is None:
        raise ConnectionError(
            f'Could not connect to service on port {minimum} or higher.')
      # 代码不依赖于此断言的结果。
      assert port >= minimum, (
          f'Unexpected port {port} when minimum was {minimum}.')
      return port
  ```

  ```python
  错误:
    def connect_to_next_port(self, minimum: int) -> int:
      """Connects to the next available port.

      Args:
        minimum: A port value greater or equal to 1024.

      Returns:
        The new minimum port.
      """
      assert minimum >= 1024, 'Minimum port must be at least 1024.'
      # 以下代码依赖于前面的断言。
      port = self._find_next_open_port(minimum)
      assert port is not None
      # 返回语句的类型检查依赖于断言。
      return port
  ```


- 库或包可以定义自己的异常。这样做时，它们必须继承自现有的异常类。异常名称应以 `Error` 结尾，不应引入重复（`foo.FooError`）。

- 永远不要使用捕获所有 `except:` 语句，或捕获 `Exception` 或 `StandardError`，除非你是：

  - 重新引发异常，或
  - 在程序中创建了隔离点，异常在此处不会传播，而是被记录和抑制，例如通过保护线程免于崩溃的最外层块。

  在这方面，Python 是非常宽容的，`except:` 将真正捕获一切，包括拼写错误的名称、sys.exit() 调用、Ctrl+C 中断、unittest 失败以及所有其他你根本不想捕获的异常。

- 最小化 `try` 或 `except` 块中的代码量。`try` 的主体越大，越可能由你没料到会引发异常的代码行引发异常。在这些情况下，`try` 或 `except` 块会隐藏真正的错误。

- 使用 `finally` 子句执行代码，无论在 `try` 块中是否引发异常。这通常对于清理很有用，例如关闭文件。

### 可变全局状态

避免可变全局状态。

#### 定义

程序执行期间可能会发生变化的模块级值或类属性。

#### 优点

偶尔会很有用。

#### 缺点

- 打破封装：这种设计可能会使实现有效目标变得困难。例如，如果全局状态用于管理数据库连接，则同时连接到两个不同的数据库（例如，在迁移期间计算差异时）变得困难。类似的问题很容易在全局注册表中出现。

- 由于在模块被第一次导入时进行全局变量的赋值，可能会改变模块的行为，因此可能性。

#### 决策

避免可变全局状态。

在极少数情况下，如果确实需要使用全局状态，则应将可变全局实体声明为模块级或类属性，并通过在名称前添加 `_` 将其内部化。如果必要，对可变全局状态的外部访问必须通过公共函数或类方法进行。请通过注释或链接到注释的文档来解释为什么需要使用可变全局状态的设计原因。

模块级常量是允许的并且是鼓励的。例如：
`_MAX_HOLY_HANDGRENADE_COUNT = 3` 用于内部使用常量或
`SIR_LANCELOTS_FAVORITE_COLOR = "blue"` 用于公共 API 常量。常量必须使用大写字母和下划线命名。参见下面的 [命名](#s3.16-naming)。

### 嵌套/本地/内部类和函数

当用于封闭局部变量时，嵌套的本地函数或类是可以接受的。内部类是可以的。

#### 定义

类可以在方法、函数或类内部定义。函数可以在方法或函数内部定义。嵌套函数具有对封闭作用域中定义的变量的只读访问权限。

#### 优点

允许定义仅在非常有限范围内使用的实用类和函数。非常类似 [ADT](https://en.wikipedia.org/wiki/Abstract_data_type)（抽象数据类型）。常用于实现装饰器。

#### 缺点

嵌套函数和类无法直接进行测试。嵌套可能会使外部函数变得更长，难以阅读。

#### 决策

它们是可以接受的，但有一些注意事项。除了在封闭局部值（而不是 `self` 或 `cls`）时，避免嵌套函数或类。不要仅仅为了将其隐藏在模块的用户中而嵌套函数。而是，在模块级别的名称前面加上 `_`，这样它仍然可以被测试时访问。

### 理解与生成器表达式

简单情况下使用理解和生成器表达式是可以的。

#### 定义 

列表、字典和集合推导式以及生成器表达式提供了一种简洁高效的方式来创建容器类型和迭代器，而无需使用传统的循环、`map()`、`filter()`或`lambda`。

#### 优点 

简单的推导式可能比其他字典、列表或集合创建技术更清晰简单。生成器表达式可能非常高效，因为它们完全避免了列表的创建。

#### 缺点 

复杂的推导式或生成器表达式可能难以阅读。

#### 决策

允许使用理解，但不允许使用多个 `for` 子句或过滤器表达式。优化可读性，而不是简洁性。

```python
正确:
  result = [mapping_expr for value in iterable if filter_expr]

  result = [
      is_valid(metric={'key': value})
      for value in interesting_iterable
      if a_longer_filter_expression(value)
  ]

  descriptive_name = [
      transform({'key': key, 'value': value}, color='black')
      for key, value in generate_iterable(some_input)
      if complicated_condition_is_met(key, value)
  ]

  result = []
  for x in range(10):
    for y in range(5):
      if x * y > 10:
        result.append((x, y))

  return {
      x: complicated_transform(x)
      for x in long_generator_function(parameter)
      if x is not None
  }

  return (x**2 for x in range(10))

  unique_names = {user.name for user in users if user is not None}
```

```python
错误:
  result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]

  return (
      (x, y, z)
      for x in range(5)
      for y in range(5)
      if x != y
      for z in range(5)
      if y != z
  )
```

### 默认迭代器和运算符

对于支持的类型，请使用默认迭代器和运算符，如列表、字典和文件。

#### 定义 

容器类型，如字典和列表，定义了默认迭代器和成员关系测试运算符（“in”和“not in”）。

#### 优点 

默认的迭代器和运算符简单高效。它们直接表达了操作，无需额外的方法调用。使用默认运算符的函数是通用的。它可以与支持该操作的任何类型一起使用。

#### 缺点 

通过阅读方法名称无法确定对象的类型（除非变量具有类型注释）。这也是一个优点。

#### 决策

对于支持的类型，请使用默认迭代器和运算符，如列表、字典和文件。内置类型也定义了迭代器方法。优先使用这些方法而不是返回列表的方法，除了在迭代时不要改变容器。

```python
正确:  for key in adict: ...
      if obj in alist: ...
      for line in afile: ...
      for k, v in adict.items(): ...
```

```python
错误:   for key in adict.keys(): ...
      for line in afile.readlines(): ...
```

### 生成器

根据需要使用生成器。

#### 定义 

生成器函数返回一个迭代器，每次执行 yield 语句时都会产生一个值。在产生一个值后，生成器函数的运行状态会被暂停，直到下一个值被需要。

#### 优点 

代码更简单，因为每次调用时局部变量的状态和控制流都会被保留。生成器使用的内存比一次创建整个值列表的函数要少。

#### 缺点 

生成器中的局部变量在生成器被耗尽使用或被垃圾回收之前不会被回收。

#### 决策

可以。对于生成器函数，在文档字符串中使用“Yields:”而不是“Returns:”。

如果生成器管理昂贵的资源，请确保强制清理。

清理的一种好方法是使用上下文管理器来包装生成器 [PEP-0533](https://peps.python.org/pep-0533/)。

### Lambda 函数

可以用于单行代码。优先使用生成器表达式而不是带有 `lambda` 的 `map()` 或 `filter()`。

#### 定义

Lambda 在表达式中定义匿名函数，而不是语句。

#### 优点

方便。

#### 缺点

比本地函数更难阅读和调试。缺乏名称意味着堆栈跟踪更难理解。表达能力有限，因为函数可能只包含一个表达式。

#### 决策

允许使用 Lambda。如果 Lambda 函数内部的代码跨越多行或超过 60-80 个字符，最好将其定义为常规的 [嵌套函数](#lexical-scoping)。

对于像乘法这样的常见操作，使用 `operator` 模块中的函数，而不是 Lambda 函数。例如，更倾向于使用 `operator.mul` 而不是 `lambda x, y: x * y`。

### 条件表达式

在简单情况下可以使用。

#### 定义

条件表达式（有时称为“三元运算符”）是提供 if 语句的更短语法的机制。例如：`x = 1 if cond else 2`。

#### 优点

比 if 语句更短更方便。

#### 缺点

可能比 if 语句更难阅读。如果表达式很长，条件可能很难定位。

#### 决策

在简单情况下可以使用。每个部分都必须放在一行上：`true-expression`、`if-expression`、`else-expression`。当情况变得更加复杂时，请使用完整的 if 语句。

```python
正确:
    one_line = 'yes' if predicate(value) else 'no'
    slightly_split = ('yes' if predicate(value)
                      else 'no, nein, nyet')
    the_longest_ternary_style_that_can_be_done = (
        'yes, true, affirmative, confirmed, correct'
        if predicate(value)
        else 'no, false, negative, nay')
```

```python
错误:
    bad_line_breaking = ('yes' if predicate(value) else
                         'no')
    portion_too_long = ('yes'
                        if some_long_module.some_long_predicate_function(
                            really_long_variable_name)
                        else 'no, false, negative, nay')
```

### 默认参数值

在大多数情况下都可以使用。

#### 定义

你可以在函数参数列表的末尾指定变量的值，例如 `def foo(a, b=0):`。如果 `foo` 只使用一个参数调用，`b` 将被设置为 0。如果使用两个参数调用，`b` 的值将是第二个参数的值。

#### 优点

通常，你有一个函数使用了很多默认值，但在偶尔情况下你想要覆盖这些默认值。默认参数值提供了一种简单的方法来实现这一点，而不必为罕见的异常情况定义大量的函数。由于 Python 不支持方法/函数的重载，因此默认参数是一种“伪造”重载行为的简单方法。

#### 缺点

默认参数在模块加载时只会被计算一次。如果参数是可变对象，比如列表或字典，这可能会导致问题。如果函数修改了对象（例如，向列表添加一个项目），则默认值也会被修改。

#### 决策

在使用时要注意以下事项：

不要在函数或方法定义中使用可变对象作为默认值。

```python
正确: def foo(a, b=None):
         if b is None:
             b = []
正确: def foo(a, b: Sequence | None = None):
         if b is None:
             b = []
正确: def foo(a, b: Sequence = ()):  # 空元组可以，因为元组是不可变的。
         ...
```

```python
from absl import flags
_FOO = flags.DEFINE_string(...)

错误:  def foo(a, b=[]):
         ...
错误:  def foo(a, b=time.time()):  # `b` 应该表示模块加载的时间吗？
         ...
错误:  def foo(a, b=_FOO.value):  # 还未解析 sys.argv...
         ...
错误:  def foo(a, b: Mapping = {}):  # 仍然可能被传递给未检查的代码。
         ...
```

### 属性

属性可用于控制需要进行轻量级计算或逻辑处理的属性的获取或设置。属性的实现必须符合普通属性访问的一般期望：它们应该廉价、简单直接，并且不出乎意料。

#### 定义

一种包装方法调用以获取和设置属性的标准属性访问的方式。

#### 优点

- 允许使用属性访问和赋值 API，而不是 [getter 和 setter](#getters-and-setters) 方法调用。
- 可以用于创建只读属性。
- 允许计算延迟。
- 在内部发生更改时，提供了一种保持类的公共接口的方式，而类的内部发展与类的用户是独立的。

#### 缺点

- 类似于运算符重载，可能会隐藏副作用。
- 对子类而言可能会令人困惑。

#### 决策

允许使用属性，但是像运算符重载一样，应该只在必要时使用，并且要符合典型属性访问的预期；否则遵循 [getter 和 setter](#getters-and-setters) 规则。

例如，不允许使用属性简单地获取和设置内部属性：因为没有进行计算，所以属性是不必要的（[请将属性公开](#getters-and-setters)）。相比之下，允许使用属性来控制属性访问或计算*简单*派生值：逻辑简单且不出乎意料。

属性应该使用 `@property` [装饰器](#s2.17-function-and-method-decorators)创建。手动实现属性描述符被视为 [强大的特性](#power-features)。

属性的继承可能不太明显。不要使用属性来实现子类可能想要覆盖和扩展的计算。

### 真/假评估

尽可能使用“隐式”假（有一些注意事项）。

#### 定义

在 Python 中，当处于布尔上下文中时，会将某些值评估为 `False`。一个快速的“经验法则”是所有“空”值都被视为假，因此 `0、None、[]、{}、''` 在布尔上下文中都被视为假。

#### 优点

使用 Python 布尔值的条件更容易阅读，更少出错。在大多数情况下，它们也更快。

#### 缺点

可能会对 C/C++ 开发人员看起来很奇怪。

#### 决策

尽可能使用“隐式”假，例如 `if foo:` 而不是 `if foo != []:`。但你应该牢记以下几点：

- 总是使用 `if foo is None:`（或 `is not None`）来检查 `None` 值。例如，在测试默认为 `None` 的变量或参数是否设置为其他值时。其他值可能在布尔上下文中是假的！

- 永远不要使用 `==` 将布尔变量与 `False` 进行比较。应该使用 `if not x:`。如果需要区分 `False` 和 `None`，则链式表达式，例如 `if not x and x is not None:`。

- 对于序列（字符串、列表、元组），利用空序列为假的事实，因此 `if seq:` 和 `if not seq:` 优于 `if len(seq):` 和 `if not len(seq):`。

- 在处理整数时，隐式假可能带来的风险比好处更大（即，意外地将 `None` 处理为 0）。你可以将已知是整数的值（并且不是 `len()` 的结果）与整数 0 进行比较。

```python
正确: if not users:
         print('没有用户')

     if i % 10 == 0:
         self.handle_multiple_of_ten()

     def f(x=None):
         if x is None:
             x = []
```

```python
错误: if len(users) == 0:
         print('没有用户')

     if not i % 10:
         self.handle_multiple_of_ten()

     def f(x=None):
         x = x or []
```

- 注意 `'0'`（即，字符串 `0`）被视为真。

- 注意在隐式布尔上下文中，Numpy 数组可能会引发异常。当测试 `np.array` 的空值时，最好使用 `.size` 属性（例如 `if not users.size`）。

#### 词法作用域

可以使用。

#### 定义

Python 中的嵌套函数可以引用封闭函数中定义的变量，但不能对其进行赋值。变量绑定使用词法作用域来解析，即基于静态程序文本。对块中名称的任何赋值都会导致 Python 将对该名称的所有引用视为局部变量，即使使用在赋值之前。如果存在全局声明，则该名称将被视为全局变量。

以下是使用此特性的示例：

```python
def get_adder(summand1: float) -> Callable[[float], float]:
    """返回一个将数字添加到给定数字的函数。"""
    def adder(summand2: float) -> float:
        return summand1 + summand2

    return adder
```

#### 优点

通常会产生更清晰、更优雅的代码。对经验丰富的 Lisp 和 Scheme（以及 Haskell 和 ML 等）程序员来说，这尤其令人欣慰。

#### 缺点

可能会导致混乱的错误，例如基于 [PEP-0227](https://peps.python.org/pep-0227/) 的以下示例：

```python
i = 4
def foo(x: Iterable[int]):
    def bar():
        print(i, end='')
    # ...
    # 一堆代码在这里
    # ...
    for i in x:  # 啊，i 确实是 foo 的局部变量，所以这就是 bar 看到的情况
        print(i, end='')
    bar()
```

因此，`foo([1, 2, 3])` 将打印 `1 2 3 3`，而不是 `1 2 3 4`。

#### 决策

可以使用。


#### 函数和方法装饰器

在有明显优势时审慎使用装饰器。避免使用 `staticmethod`，限制使用 `classmethod`。

#### 定义

[函数和方法的装饰器](https://docs.python.org/3/glossary.html#term-decorator)（也称为“`@`符号”）。一个常见的装饰器是 `@property`，用于将普通方法转换为动态计算的属性。然而，装饰器语法也允许用户定义装饰器。具体来说，对于某个函数 `my_decorator`，这样：

```python
class C:
    @my_decorator
    def method(self):
        # method body...
```

等同于：

```python
class C:
    def method(self):
        # method body...
    method = my_decorator(method)
```

#### 优点

优雅地指定方法的某些转换；这种转换可能消除一些重复的代码，强制执行不变量等。

#### 缺点

装饰器可以对函数的参数或返回值执行任意操作，导致意外的隐式行为。此外，装饰器在对象定义时执行。对于模块级对象（类、模块函数等），这是在导入时发生的。装饰器代码中的失败几乎不可能恢复。

#### 决策

在有明显优势时审慎使用装饰器。装饰器应遵循与函数相同的导入和命名指南。装饰器的 pydoc 应明确说明函数是装饰器。为装饰器编写单元测试。

在装饰器本身避免使用外部依赖（例如，不依赖文件、套接字、数据库连接等），因为它们在装饰器运行时可能不可用（在导入时，可能来自 `pydoc` 或其他工具）。对于调用带有有效参数的装饰器，应尽可能保证它在所有情况下都能成功执行。

装饰器是“顶层代码”的特殊情况——有关更多讨论，请参阅 [main](#s3.17-main)。

除非强制与现有库中定义的 API 集成，否则永远不要使用 `staticmethod`。而是编写一个模块级函数。

仅在编写了一个具有命名构造函数或修改必要全局状态的类特定例程时使用 `classmethod`，例如进程范围的缓存。

### 线程

不要依赖内置类型的原子性。

虽然 Python 的内置数据类型如字典看起来具有原子操作，但在某些情况下它们并不是原子的（例如，如果 `__hash__` 或 `__eq__` 实现为 Python 方法），不应依赖其原子性。也不应依赖原子变量赋值（因为这又依赖于字典）。

使用 `queue` 模块的 `Queue` 数据类型作为线程间通信的首选方式。否则，使用 `threading` 模块及其锁原语。优先使用条件变量和 `threading.Condition`，而不是使用底层锁。

### 强大的特性

避免使用这些特性。

#### 定义

Python 是一种极其灵活的语言，为你提供了许多高级特性，如自定义元类、访问字节码、即时编译、动态继承、对象重新父级、导入黑客、反射（例如某些情况下的 `getattr()` 使用）、修改系统内部、实现自定义清理的 `__del__` 方法等。

#### 优点

这些是强大的语言特性。它们可以使你的代码更简洁。

#### 缺点

当不绝对必要时，使用这些“酷炫”的特性非常诱人。使用底层的不寻常特性编写的代码更难阅读、理解和调试。这似乎不是原始作者最初的看法，但当重新访问代码时，它往往比更长但简单直接的代码更难。

#### 决策

在你的代码中避免使用这些特性。

内置模块和类内部使用这些特性的标准库模块和类是可以使用的（例如 `abc.ABCMeta`、`dataclasses` 和 `enum`）。

### 现代 Python：`from __future__` 导入

可以通过特殊的 `from __future__` 导入来激活新语言版本的语义更改，以便在较早的运行时中在每个文件中启用它们。

#### 定义

通过 `from __future__ import` 语句能够在预期未来的 Python 版本中提前使用一些更现代的特性。

#### 2.20.2 优点

这已经被证明可以使运行时版本升级更加平滑，因为可以在声明兼容性和防止这些文件中的回归的同时，基于每个文件进行更改。现代化的代码更易于维护，因为它不太可能在将来的运行时升级过程中积累技术债务。

#### 缺点

这样的代码可能无法在非常旧的解释器版本上运行，这些版本在引入所需的未来语句之前。在支持各种环境的项目中，这种需要更加普遍。

#### 决策

##### `from __future__` 导入

鼓励使用 `from __future__ import` 语句。它允许给定的源文件从今天开始使用更现代的 Python 语法特性。一旦你不再需要在隐藏在 `__future__` 导入后面的版本上运行功能，可以随时删除这些行。

在可能执行的版本为 3.5 而不是 >= 3.7 的代码中，导入：

```python
from __future__ import generator_stop
```

有关更多信息，请阅读 [Python 未来语句定义](https://docs.python.org/3/library/__future__.html) 文档。

请在确保代码仅在足够现代的环境中使用时删除这些导入。即使你今天的代码中当前没有使用特定的未来导入启用的功能，保持它们在文件中的位置也可以防止以后对代码的修改不小心依赖于更旧的行为。

根据需要使用其他 `from __future__` 导入语句。

### 类型注释代码

根据 [PEP-484](https://peps.python.org/pep-0484/)，你可以对 Python 代码进行类型提示，并使用类似 [pytype](https://github.com/google/pytype) 的类型检查工具在构建时检查代码。

类型注释可以在源文件中或在 [stub pyi 文件](https://peps.python.org/pep-0484/#stub-files) 中。尽可能在源文件中添加注释。对于第三方或扩展模块，请使用 pyi 文件。

#### 定义

类型注释（或“类型提示”）用于函数或方法的参数和返回值：

```python
def func(a: int) -> list[int]:
```

你还可以使用类似 [PEP-526](https://peps.python.org/pep-0526/) 的语法声明变量的类型：

```python
a: SomeType = some_func()
```

#### 优点

类型注释提高了代码的可读性和可维护性。类型检查器将许多运行时错误转换为构建时错误，并减少你使用[强大的特性](#power-features) 的能力。

#### 缺点

你必须保持类型声明的最新状态。
你可能会看到你认为是有效代码的类型错误。使用[类型检查器](https://github.com/google/pytype) 可能会降低你使用[强大的特性](#power-features) 的能力。

#### 决策

在更新代码时，强烈建议启用 Python 类型分析。在添加或修改公共 API 时，请包含类型注释，并通过构建系统启用 pytype 检查。由于静态分析对于 Python 相对较新，我们承认意外副作用（如错误推断的类型）可能阻止某些项目采用。在这种情况下，鼓励作者在 BUILD 文件或代码本身中添加一个带有 TODO 或链接到描述当前阻止类型注释采用的问题的错误的注释。

### 分号

不要在行尾使用分号，并且不要使用分号将两个语句放在同一行上。

### 行长度

最大行长度为 *80 个字符*。

80 个字符限制的明确例外情况包括：

- 长的导入语句。
- 在注释中的 URL、路径名或长标志。
- 不包含不方便分割的空格的长字符串模块级常量，如 URL 或路径名。
  - Pylint 禁用注释（例如：`# pylint: disable=invalid-name`）。

不要使用反斜杠进行[显式行连接](https://docs.python.org/3/reference/lexical_analysis.html#explicit-line-joining)。

相反，利用 Python 在括号、方括号和大括号内部的[隐式行连接](http://docs.python.org/reference/lexical_analysis.html#implicit-line-joining)。
如果必要，可以在表达式周围添加额外的括号。

请注意，此规则不禁止字符串内的反斜杠换行（请参阅[下面](#strings)）。

```python
正确: foo_bar(self, width, height, color='black', design=None, x='foo',
             emphasis=None, highlight=0)
```

```python

正确: if (width == 0 and height == 0 and
         color == 'red' and emphasis == 'strong'):

     (bridge_questions.clarification_on
      .average_airspeed_of.unladen_swallow) = 'African or European?'

     with (
         very_long_first_expression_function() as spam,
         very_long_second_expression_function() as beans,
         third_thing() as eggs,
     ):
       place_order(eggs, beans, spam, beans)
```

```python

错误:  if width == 0 and height == 0 and \
         color == 'red' and emphasis == 'strong':

     bridge_questions.clarification_on \
         .average_airspeed_of.unladen_swallow = 'African or European?'

     with very_long_first_expression_function() as spam, \
           very_long_second_expression_function() as beans, \
           third_thing() as eggs:
       place_order(eggs, beans, spam, beans)
```

当一个字面字符串不能放在一行上时，请使用括号进行隐式行连接。

```python
x = ('This will build a very long long '
     'long long long long long long string')
```

尽量在最高可能的语法级别断开行。如果必须将一行断开两次，请同时在相同的语法级别断开。

```python
正确: bridgekeeper.answer(
         name="Arthur", quest=questlib.find(owner="Arthur", perilous=True))

     answer = (a_long_line().of_chained_methods()
               .that_eventually_provides().an_answer())

     if (
         config is None
         or 'editor.language' not in config
         or config['editor.language'].use_spaces is False
     ):
       use_tabs()
```

```python
错误: bridgekeeper.answer(name="Arthur", quest=questlib.find(
        owner="Arthur", perilous=True))

    answer = a_long_line().of_chained_methods().that_eventually_provides(
        ).an_answer()

    if (config is None or 'editor.language' not in config or config[
        'editor.language'].use_spaces is False):
      use_tabs()

```

在注释中，如有必要，将长 URL 放在自己的一行上。

```python
正确:  # See details at
      # http://www.example.com/us/developer/documentation/api/content/v2.0/csv_file_name_extension_full_specification.html
```

```python
错误:  # See details at
     # http://www.example.com/us/developer/documentation/api/content/\
     # v2.0/csv_file_name_extension_full_specification.html
```

请注意上面的行继续示例中元素的缩进；请参阅[缩进](#s3.4-indentation) 部分以获取解释。

在所有其他情况下，如果 [Black](https://github.com/psf/black) 或 [Pyink](https://github.com/google/pyink) 自动格式化程序无法帮助将行缩短到限制以下，则允许行超过此最大值。鼓励作者在合适的情况下按照上述说明手动断开行。

### 括号

谨慎使用括号。

在元组周围使用括号是可以的，但不是必须的。除非在条件语句中使用括号来暗示行继续或表示元组，否则不要在返回语句或条件语句中使用括号。

```python
正确: if foo:
         bar()
     while x:
         x = bar()
     if x and y:
         bar()
     if not x:
         bar()
     # 对于一个项的元组，() 比逗号更加明显。
     onesie = (foo,)
     return foo
     return spam, beans
     return (spam, beans)
     for (x, y) in dict.items(): ...
```

```python
错误:  if (x):
         bar()
     if not(x):
         bar()
     return (foo)
```

### 缩进

使用 *4 个空格* 缩进代码块。

永远不要使用制表符。隐式行续应将包装的元素垂直对齐（参见[行长度示例](#s3.2-line-length)），或使用悬挂的 4 个空格缩进。右括号（圆括号、方括号或大括号）可以放在表达式的末尾，也可以放在单独的行上，但是与相应的开括号行缩进相同。

```python
正确:   # 与开放分隔符对齐。
       foo = long_function_name(var_one, var_two,
                                var_three, var_four)
       meal = (spam,
               beans)

       # 在字典中与开放分隔符对齐。
       foo = {
           'long_dictionary_key': value1 +
                                  value2,
           ...
       }

       # 4 个空格的悬挂缩进；第一行没有任何内容。
       foo = long_function_name(
           var_one, var_two, var_three,
           var_four)
       meal = (
           spam,
           beans)

       # 4 个空格的悬挂缩进；第一行没有任何内容，
       # 将右括号放在新的一行上。
       foo = long_function_name(
           var_one, var_two, var_three,
           var_four
       )
       meal = (
           spam,
           beans,
       )

       # 字典中的 4 个空格的悬挂缩进。
       foo = {
           'long_dictionary_key':
               long_dictionary_value,
           ...
       }
```

```python
错误:    # 第一行上的内容禁止。
       foo = long_function_name(var_one, var_two,
           var_three, var_four)
       meal = (spam,
           beans)

       # 2 个空格的悬挂缩进禁止。
       foo = long_function_name(
         var_one, var_two, var_three,
         var_four)

       # 在字典中禁止悬挂缩进。
       foo = {
           'long_dictionary_key':
           long_dictionary_value,
           ...
       }
```

#### 序列中的尾随逗号？

推荐在项的序列中仅在最后一个元素之后不出现关闭容器标记 `]`、`)` 或 `}` 时以及具有单个元素的元组时使用尾随逗号。尾随逗号的存在也被用作我们的 Python 代码自动格式化器 [Black](https://github.com/psf/black) 或 [Pyink](https://github.com/google/pyink) 的提示，指示它将包含项目的容器自动格式化为每行一个项目，当最后一个元素后的 `,` 存在时。

```python
正确:   golomb3 = [0, 1, 3]
       golomb4 = [
           0,
           1,
           4,
           6,
       ]
```

```python
错误:    golomb4 = [
           0,
           1,
           4,
           6,]
```

### 空行

顶级定义（无论是函数还是类定义）之间有两个空行。方法定义之间以及 `class` 的文档字符串与第一个方法之间有一个空行。`def` 行后面没有空行。根据需要在函数或方法内使用单个空行。

空行不一定要与定义对齐。例如，在函数、类和方法定义之前的相关注释可能是有意义的。考虑一下，你的注释是否作为文档字符串的一部分会更有用。

### 空格

遵循标准的排版规则，使用标点符号周围的空格。

括号、方括号或大括号内部不要有空格。

```python
正确: spam(ham[1], {'eggs': 2}, [])
```

```python
错误:  spam( ham[ 1 ], { 'eggs': 2 }, [ ] )
```

逗号、分号或冒号前不要有空格。在逗号、分号或冒号后使用空格，除非在行末。

```python
正确: if x == 4:
         print(x, y)
     x, y = y, x
```

```python
错误:  if x == 4 :
         print(x , y)
     x , y = y , x
```

不要在开始参数列表、索引或切片的开放括号/方括号之前添加空格。

```python
正确: spam(1)
```

```python
错误:  spam (1)
```

```python
正确: dict['key'] = list[index]
```

```python
错误:  dict ['key'] = list [index]
```

不要留有尾随空格。

将赋值（`=`）、比较（`==, <, >, !=, <>, <=, >=, in, not in, is, is not`）和布尔运算符（`and, or, not`）周围各添加一个空格。对于算术运算符（`+`, `-`, `*`, `/`, `//`, `%`, `**`, `@`），在你认为需要的地方插入空格。

```python
正确: x == 1
```

```python
错误:  x<1
```

在传递关键字参数或定义默认参数值时，不要在 `=` 周围使用空格，除了一种情况：[当存在类型注释时](#typing-default-values)，为默认参数值*要*在 `=` 周围使用空格。

```python
正确: def complex(real, imag=0.0): return Magic(r=real, i=imag)
正确: def complex(real, imag: float = 0.0): return Magic(r=real, i=imag)
```

```python
错误:  def complex(real, imag = 0.0): return Magic(r = real, i = imag)
错误:  def complex(real, imag: float=0.0): return Magic(r = real, i = imag)
```

不要在连续行上垂直对齐标记周围使用空格，因为这会增加维护负担（适用于 `:`, `#`, `=` 等）：

```python
正确:
  foo = 1000  # comment
  long_name = 2  # comment that should not be aligned

  dictionary = {
      'foo': 1,
      'long_name': 2,
  }
```

```python
错误:
  foo       = 1000  # comment
  long_name = 2     # comment that should not be aligned

  dictionary = {
      'foo'      : 1,
      'long_name': 2,
  }
```


### Shebang 行

大多数 `.py` 文件不需要以 `#!` 行开始。在程序的主文件中使用 `#!/usr/bin/env python3`（以支持虚拟环境）或 `#!/usr/bin/python3`，根据 [PEP-394](https://peps.python.org/pep-0394/)。

这行是由内核用来找到 Python 解释器的，但在导入模块时 Python 会忽略它。它只在意图直接执行的文件上需要。

### 注释和文档字符串

确保在模块、函数、方法文档字符串和内联注释中使用正确的风格。

#### 文档字符串

Python 使用*文档字符串*来记录代码。文档字符串是包、模块、类或函数中的第一个语句。这些字符串可以通过对象的 `__doc__` 成员自动提取，并被 `pydoc` 使用。（尝试在你的模块上运行 `pydoc`，看看它是什么样子的。）始终使用三重双引号 `"""` 格式来编写文档字符串（参考 [PEP 257](https://peps.python.org/pep-0257/)）。文档字符串应当组织成一行摘要（一行实际内容不超过 80 个字符），以句号、问号或感叹号结尾。在编写更多内容（鼓励的情况下），摘要后面必须跟着一个空行，然后是文档字符串的其余部分，从与第一行的第一个引号的光标位置相同的位置开始。文档字符串的更多格式指南如下所示。

#### 模块

每个文件都应包含许可证样板。选择适用于项目使用的许可证的样板（例如 Apache 2.0、BSD、LGPL、GPL）。

文件应以描述模块内容和用法的文档字符串开始。
```python
"""一个模块或程序的一行摘要，以句号结束。

留出一个空行。本文档字符串的其余部分应包含对模块或程序的整体描述。可选地，它还可以包含对导出的类和函数的简要描述和/或用法示例。

典型的使用示例：

  foo = ClassFoo()
  bar = foo.FunctionBar()
"""
```

##### 测试模块

测试文件的模块级别文档字符串不是必需的。仅当可以提供其他信息时才应包含它们。

示例包括有关测试应如何运行的特定信息、关于不寻常设置模式的说明、对外部环境的依赖等。

```python
"""此 Blaze 测试使用黄金文件。

你可以通过从 `google3` 目录运行 `blaze run //foo/bar:foo_test -- --update_golden_files` 来更新这些文件。
"""
```

不提供任何新信息的文档字符串不应该被使用。

```python
"""foo.bar 的测试。"""
```

#### 函数和方法

在本节中，“函数”指的是方法、函数、生成器或属性。

对于具有以下一个或多个特性的每个函数，都必须有文档字符串：

-   是公共 API 的一部分
-   非平凡大小
-   逻辑不明显

文档字符串应提供足够的信息，以便在不阅读函数代码的情况下编写对该函数的调用。文档字符串应描述函数的调用语法和语义，但通常不应描述其实现细节，除非这些细节与如何使用该函数相关。例如，一个作为副作用改变其一个参数的函数应在其文档字符串中说明这一点。否则，对调用者不相关的函数实现的微妙但重要的细节最好作为注释与代码一起而不是函数的文档字符串内。

文档字符串可以是描述性样式（`"""Fetches rows from a Bigtable."""`）或命令式样式（`"""Fetch rows from a Bigtable."""`），但在文件中样式应保持一致。`@property` 数据描述符的文档字符串应使用与属性或[函数参数](#doc-function-args)的文档字符串相同的样式（`"""The Bigtable path."""`，而不是 `"""Returns the Bigtable path."""`）。

函数的某些方面应在特殊部分中记录，列在下面。每个部分都以一个标题行开始，以冒号结尾。除标题外，所有部分都应保持 2 或 4 个空格的悬挂缩进（在文件中保持一致）。在函数名和签名足够明确的情况下，这些部分可以省略，因为它们可以用一行文档字符串来恰当地描述。

```python
def fetch_smalltable_rows(
    table_handle: smalltable.Table,
    keys: Sequence[bytes | str],
    require_all_keys: bool = False,
) -> Mapping[bytes, tuple[str, ...]]:
    """从 Smalltable 中获取行。

    从由 table_handle 表示的 Table 实例中检索与给定键相关的行。字符串键将被编码为 UTF-8。

    Args:
        table_handle: 一个打开的 smalltable.Table 实例。
        keys: 表示要获取的每个表行的键的字符串序列。字符串键将被编码为 UTF-8。
        require_all_keys: 如果为 True，则只返回所有键都设置了值的行。

    Returns:
        一个将键映射到检索到的相应表行数据的字典。每行表示为字符串元组。例如：

        {b'Serak': ('Rigel VII', 'Preparer'),
         b'Zim': ('Irk', 'Invader'),
         b'Lrrr': ('Omicron Persei 8', 'Emperor')}

        返回的键始终为字节。如果 keys 参数中的键在字典中不存在，则表示该行未在表中找到（并且 require_all_keys 必须为 False）。

    Raises:
        IOError: 访问 smalltable 时发生错误。
    """
```

同样，也允许使用带有换行的 `Args:` 变体：

```python
def fetch_smalltable_rows(
    table_handle: smalltable.Table,
    keys: Sequence[bytes | str],
    require_all_keys: bool = False,
) -> Mapping[bytes, tuple[str, ...]]:
    """从 Smalltable 中获取行。

    从由 table_handle 表示的 Table 实例中检索与给定键相关的行。字符串键将被编码为 UTF-8。

    Args:
      table_handle:
        一个打开的 smalltable.Table 实例。
      keys:
        表示要获取的每个表行的键的字符串序列。字符串键将被编码为 UTF-8。
      require_all_keys:
        如果为 True，则只返回所有键都设置了值的行。

    Returns:
      一个将键映射到检索到的相应表行数据的字典。每行表示为字符串元组。例如：

      {b'Serak': ('Rigel VII', 'Preparer'),
       b'Zim': ('Irk', 'Invader'),
       b'Lrrr': ('Omicron Persei 8', 'Emperor')}

      返回的键始终为字节。如果 keys 参数中的键在字典中不存在，则表示该行未在表中找到（并且 require_all_keys 必须为 False）。

    Raises:
      IOError: 访问 smalltable 时发生错误。
    """
```

##### 覆盖方法

如果方法覆盖了基类的方法，则不需要添加文档字符串，除非明确使用了 `@override` 装饰器（来自 `typing_extensions` 或 `typing` 模块），除非覆盖方法的行为实质上完善了基本方法的约定，或者需要提供详细信息（例如，记录了额外的副作用），在这种情况下，覆盖方法上至少需要添加具有这些差异的文档字符串。

```python
from typing_extensions import override

class Parent:
  def do_something(self):
    """父类方法，包含文档字符串。"""

# 子类，方法使用了 override 注释。
class Child(Parent):
  @override
  def do_something(self):
    pass
```

```python
# 子类，但没有使用 @override 装饰器，需要添加文档字符串。
class Child(Parent):
  def do_something(self):
    pass

# 文档字符串内容较简单，@override 装饰器足以表示文档可以在基类中找到。
class Child(Parent):
  @override
  def do_something(self):
    """见基类。"""
```

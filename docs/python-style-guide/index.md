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

#### 3.8.4 类

类应该在类定义下方有一个文档字符串，描述该类。公共属性（不包括[属性](#properties)）应该在这里文档化，遵循与[函数的`Args`](#doc-function-args)部分相同的格式。

```python
class SampleClass:
    """类的摘要。

    更长的类信息...
    更长的类信息...

    属性:
        likes_spam: 一个布尔值，指示我们是否喜欢SPAM。
        eggs: 我们产下的鸡蛋的整数计数。
    """

    def __init__(self, likes_spam: bool = False):
        """根据SPAM的偏好初始化实例。

        参数:
          likes_spam: 定义实例是否具有此偏好。
        """
        self.likes_spam = likes_spam
        self.eggs = 0

    @property
    def butter_sticks(self) -> int:
        """我们拥有的黄油条数。"""
```

所有类文档字符串应该以一个一行摘要开始，描述类实例代表什么。这意味着 `Exception` 的子类也应该描述异常代表什么，而不是它可能发生的上下文。类文档字符串不应重复不必要的信息，比如这是一个类。

```python
# 正确:
class CheeseShopAddress:
  """奶酪店的地址。

  ...
  """

class OutOfCheeseError(Exception):
  """没有更多的奶酪可用。"""
```

```python
# 错误:
class CheeseShopAddress:
  """描述奶酪店地址的类。

  ...
  """

class OutOfCheeseError(Exception):
  """在没有更多奶酪可用时引发。"""
```

#### 块和行内注释

在代码的复杂部分最终需要有注释。如果你将不得不在下一次[代码审查](http://en.wikipedia.org/wiki/Code_review)中解释它，那么现在就应该注释。复杂的操作在操作开始前会有几行注释。不明显的操作在行尾会有注释。

```python
# 我们使用加权字典搜索来找出 i 在数组中的位置。我们根据数组中的最大数和数组大小来推断位置，然后做二进制搜索以获取精确的数字。

if i & (i-1) == 0:  # 如果 i 是 0 或 2 的幂，则为 True。
```

为了提高可读性，这些注释应该至少与代码相隔 2 个空格，注释字符 `#` 开始，注释文本本身之前至少有一个空格。

另一方面，不要描述代码。假设阅读代码的人比你更了解 Python（尽管不知道你要做什么）。

```python
# 坏注释：现在遍历 b 数组，并确保每当 i 出现时，下一个元素是 i+1
```

<!-- 下一节是从 C++ 风格指南复制的。-->

#### 标点符号、拼写和语法

注意标点符号、拼写和语法；写好的注释比写得不好的注释更容易阅读。

注释应该像叙述性文本一样易读，具有适当的大写和标点符号。在许多情况下，完整的句子比句子片段更易读。较短的注释，如代码行尾的注释，有时可能不太正式，但你应该保持一致的风格。

尽管代码审查者指出你在应该使用分号而不是逗号时可能会令人沮丧，但源代码保持高水平的清晰度和可读性非常重要。正确的标点符号、拼写和语法有助于实现这个目标。

### 字符串

使用 [f-string](https://docs.python.org/3/reference/lexical_analysis.html#f-strings)、`%` 运算符或 `format` 方法来格式化字符串，即使参数都是字符串。在选择字符串格式化选项时，请根据自己的判断来决定。使用 `+` 进行单个连接是可以的，但不要用 `+` 格式化。

```python
正确: x = f'name: {name}; score: {n}'
     x = '%s, %s!' % (imperative, expletive)
     x = '{}, {}'.

format(first, second)
     x = 'name: %s; score: %d' % (name, n)
     x = 'name: %(name)s; score: %(score)d' % {'name':name, 'score':n}
     x = 'name: {}; score: {}'.format(name, n)
     x = a + b
```

```python
错误: x = first + ', ' + second
    x = 'name: ' + name + '; score: ' + str(n)
```

避免在循环中使用 `+` 和 `+=` 运算符来累积字符串。在某些情况下，使用加法累积字符串可能导致二次方而不是线性的运行时间。尽管这种常见的累积可能在 CPython 上得到优化，但这是一个实现细节。优化适用的条件不易预测，并且可能会发生变化。相反，将每个子字符串添加到列表中，然后在循环终止后使用 `''.join` 连接列表，或者将每个子字符串写入 `io.StringIO` 缓冲区。这些技术始终具有摊销的线性运行时间复杂度。

```python
正确: items = ['<table>']
     for last_name, first_name in employee_list:
         items.append('<tr><td>%s, %s</td></tr>' % (last_name, first_name))
     items.append('</table>')
     employee_table = ''.join(items)
```

```python
错误: employee_table = '<table>'
    for last_name, first_name in employee_list:
        employee_table += '<tr><td>%s, %s</td></tr>' % (last_name, first_name)
    employee_table += '</table>'
```

在文件中一致地选择字符串引号字符。选择 `'` 或 `"` 并坚持使用它。在字符串中使用另一个引号字符是可以的，以避免在字符串内需要反斜杠转义引号字符。

```python
正确:
  Python('Why are you hiding your eyes?')
  Gollum("I'm scared of lint errors.")
  Narrator('"Good!" thought a happy Python reviewer.')
```

```python
错误:
  Python("Why are you hiding your eyes?")
  Gollum('The lint. It burns. It burns us.')
  Gollum("Always the great lint. Watching. Watching.")
```

在多行字符串中，更倾向于使用 `"""` 而不是 `'''`。如果项目选择对所有非文档字符串的多行字符串使用 `'''`，则必须同时使用 `'` 来表示常规字符串。文档字符串必须始终使用 `"""`。

多行字符串不会随着程序的其他部分的缩进而流动。如果需要避免在字符串中嵌入额外的空格，请使用连接的单行字符串或使用 [`textwrap.dedent()`](https://docs.python.org/3/library/textwrap.html#textwrap.dedent) 从每行中删除初始空格：

```python
  错误:
  long_string = """This is pretty ugly.
Don't do this.
"""
```

```python
  正确:
  long_string = """This is fine if your use case can accept
      extraneous leading spaces."""
```

```python
  正确:
  long_string = ("And this is fine if you cannot accept\n" +
                 "extraneous leading spaces.")
```

```python
  正确:
  long_string = ("And this too is fine if you cannot accept\n"
                 "extraneous leading spaces.")
```

```python
  正确:
  import textwrap

  long_string = textwrap.dedent("""\
      This is also fine, because textwrap.dedent()
      will collapse common leading spaces in each line.""")
```

请注意，在这里使用反斜杠不违反[明确的行连续性](#line-length)的禁令；在这种情况下，反斜杠是在字符串文字中[转义换行符](https://docs.python.org/3/reference/lexical_analysis.html#string-and-bytes-literals)。

#### 日志

对于期望一个模式字符串（带有 %-placeholders）作为第一个参数的日志函数：始终使用字符串文字（而不是 f-string!）作为它们的第一个参数，并使用模式参数作为后续参数。一些日志实现会收集未展开的模式字符串作为可查询的字段。这也可以防止花费时间渲染没有配置日志记录器输出的消息。

```python
正确:
import tensorflow as tf
logger = tf.get_logger()
logger.info('TensorFlow Version is: %s', tf.__version__)
```

```python
正确:
import os
from absl import logging

logging.info('Current $PAGER is: %s', os.getenv('PAGER', default=''))

homedir = os.getenv('HOME')
if homedir is None or not os.access(homedir, os.W_OK):
  logging.error('Cannot write to home directory, $HOME=%r', homedir)
```

```python
错误:
import os
from absl import logging

logging.info('Current $PAGER is:')
logging.info(os.getenv('PAGER', default=''))

homedir = os.getenv('HOME')
if homedir is None or not os.access(homedir, os.W_OK):
  logging.error(f'Cannot write to home directory, $HOME={homedir!r}')
```

#### 错误消息

错误消息（例如：`ValueError` 异常上的消息字符串，或向用户显示的消息）应遵循三条准则：

1. 消息需要精确匹配实际的错误条件。
2. 插值部分需要始终清晰地识别为这样的。
3. 它们应该允许简单的自动化处理（例如 grepping）。

```python
正确:
if not 0 <= p <= 1:
  raise ValueError(f'Not a probability: {p=}')

try:
  os.rmdir(workdir)
except OSError as error:
  logging.warning('Could not remove directory (reason: %r): %r',
                  error, workdir)
```

```python
错误:
if p < 0 or p > 1:  # PROBLEM: also false for float('nan')!
  raise ValueError(f'Not a probability: {p=}')

try:
  os.rmdir(workdir)
except OSError:
  # PROBLEM: Message makes an assumption that might not be true:
  # Deletion might have failed for some other reason, misleading
  # whoever has to debug this.
  logging.warning('Directory already was deleted: %s', workdir)

try:
  os.rmdir(workdir)
except OSError:
  # PROBLEM: The message is harder to grep for than necessary, and
  # not universally non-confusing for all possible values of `workdir`.
  # Imagine someone calling a library function with such code
  # using a name such as workdir = 'deleted'. The warning would read:
  # "The deleted directory could not be deleted."
  logging.warning('The %s directory could not be deleted.', workdir)
```

### 文件、套接字和类似的有状态资源

在使用完文件和套接字后明确关闭它们。这个规则自然地延伸到内部使用套接字的可关闭资源，例如数据库连接，以及需要以类似方式关闭的其他资源。仅举几个例子，这也包括 [mmap 映射](https://docs.python.org/3/library/mmap.html)、[h5py File 对象](https://docs.h5py.org/en/stable/high/file.html)和 [matplotlib.pyplot 图形窗口](https://matplotlib.org/2.1.0/api/_as_gen/matplotlib.pyplot.close.html)。

将文件、套接字或其他类似的有状态对象保持不必要地打开有很多弊端：

- 它们可能会消耗有限的系统资源，例如文件描述符。如果代码处理许多这样的对象，那么如果它们在使用后不立即返回到系统中，则可能不必要地耗尽这些资源。
- 保持文件打开可能会阻止其他操作，如移动或删除它们，或卸载文件系统。
- 在整个程序中共享的文件和套接字可能会在逻辑上关闭后无意中被读取或写入。如果它们实际上被关闭，尝试从中读取或写入将引发异常，从而更早地发现问题。

此外，虽然文件和套接字（以及一些行为类似的资源）在对象被销毁时会自动关闭，但将对象的生命周期与资源的状态耦合是不良的做法：

- 没有保证运行时实际上何时调用`__del__`方法。不同的Python实现使用不同的内存管理技术，例如延迟垃圾收集，这可能会将对象的生命周期增加到任意和无限的时间。
- 对文件的意外引用，例如在全局变量或异常回溯中，可能会使文件保留更长的时间。

依赖于最终器来执行具有可观察副作用的自动清理已被一次又一次地重新发现会导致重大问题，跨越了许多年和多种语言（例如，查看 Java 的[这篇文章](https://wiki.sei.cmu.edu/confluence/display/java/MET12-J.+Do+not+use+finalizers)）。

管理文件和类似资源的首选方法是使用 [`with` 语句](http://docs.python.org/reference/compound_stmts.html#the-with-statement)：

```python
with open("hello.txt") as hello_file:
    for line in hello_file:
        print(line)
```

对于不支持 `with` 语句的类似文件的对象，使用 `contextlib.closing()`：

```python
import contextlib

with contextlib.closing(urllib.urlopen("http://www.python.org/")) as front_page:
    for line in front_page:
        print(line)
```

在极少数情况下，无法使用基于上下文的资源管理，代码文档必须清楚地解释资源的生命周期如何管理。

### TODO 注释

对于临时的、短期的解决方案，或者足够好但不完美的代码，请使用 `TODO` 注释。

一个 `TODO` 注释以全大写的单词 `TODO` 开始，后跟一个冒号，接着是一个链接到包含上下文的资源的链接，理想情况下是一个错误引用。错误引用是首选的，因为错误是被跟踪的，并且有跟进注释。在这个上下文之后，跟着一个用连字符 `-` 引入的解释字符串。目的是有一个一致的 `TODO` 格式，可以通过搜索找到更多细节。

```python
# TODO: crbug.com/192795 - Investigate cpufreq optimizations.
```

老式风格，以前推荐使用，但不鼓励在新代码中使用：

```python
# TODO(crbug.com/192795): Investigate cpufreq optimizations.
# TODO(yourusername): Use a "\*" here for concatenation operator.
```

避免添加引用个人或团队的 TODO，作为上下文：

```python
# TODO: @yourusername - File an issue and use a '*' for repetition.
```

如果你的 `TODO` 是这种形式的：“在将来的某个日期做某事”，确保你要么包含一个非常具体的日期（“在2009年11月之前修复”），要么包含一个非常具体的事件（“当所有客户端都能处理XML响应时删除此代码。”），未来的代码维护者将理解。问题是追踪这个的理想选择。

### 导入格式化

导入应该在单独的行上；[typing 和 collections.abc 导入有例外](#typing-imports)。

例如：

```python
正确: from collections.abc import Mapping, Sequence
     import os
     import sys
     from typing import Any, NewType
```

```python
错误:  import os, sys
```

### Statements

通常每行只有一个语句。

但是，如果整个语句都适合在同一行上，则可以将测试的结果放在与测试相同的行上。特别是，不能使用 `try`/`except`，因为 `try` 和 `except` 不能同时放在同一行上，如果没有 `else`，则只能使用 `if`。

```python
正确:

  if foo: bar(foo)
```

```python
错误:

  if foo: bar(foo)
  else:   baz(foo)

  try:               bar(foo)
  except ValueError: baz(foo)

  try:
      bar(foo)
  except ValueError: baz(foo)
```

### Getters 和 Setters

当获取或设置变量的值具有有意义的角色或行为时，应使用 getter 和 setter 函数（也称为访问器和修改器）。

特别是，当获取或设置变量是复杂的或成本显着的时候，无论当前还是在合理的未来，都应该使用它们。

例如，如果一对 getter/setter 仅仅读取和写入一个内部属性，那么内部属性应该被公开。相比之下，如果设置变量意味着某些状态被无效或重建，那么它应该是一个setter函数。函数调用暗示了正在发生的可能是非平凡的操作。另外，当需要简单的逻辑时，[属性](#properties)可能是一个选项，或者重构以不再需要 getter 和 setter。

Getter 和 setter应遵循[命名](#s3.16-naming)指南，例如`get_foo()`和`set_foo()`。

如果过去的行为允许通过属性访问，请不要将新的 getter/setter 函数绑定到属性。任何仍然尝试通过旧方法访问变量的代码应该明显中断，以便他们意识到复杂性的变化。

### 3.16 Naming

`module_name`、`package_name`、`ClassName`、`method_name`、`ExceptionName`、`function_name`、`GLOBAL_CONSTANT_NAME`、`global_var_name`、`instance_var_name`、`function_parameter_name`、`local_var_name`、`query_proper_noun_for_thing`、`send_acronym_via_https`。

函数名、变量名和文件名应具有描述性；避免使用缩写。特别是，不要使用对读者在项目外不熟悉的模糊或不熟悉的缩写，并且不要通过删除单词内的字母缩写。

始终使用 `.py` 文件扩展名。不要使用破折号。

#### 应避免的命名

- 除了特定情况下允许的情况之外，应避免使用单个字符的名称：

  - 计数器或迭代器（例如 `i`、`j`、`k`、`v` 等）
  - 在 `try/except` 语句中作为异常标识符的 `e`
  - 在 `with` 语句中作为文件句柄的 `f`
  - 没有约束条件的私有[类型变量](#typing-type-var)（例如 `_T = TypeVar("_T")`、`_P = ParamSpec("_P")`）

  请注意，不要滥用单个字符命名。通常来说，名称的描述性应与其可见范围成正比。例如，在 5 行代码块中，`i` 可能是一个不错的名称，但在多个嵌套作用域中，它可能过于模糊。

- 任何包/模块名称中都不应包含破折号（`-`）

- `__double_leading_and_trailing_underscore__` 格式的名称（Python 保留）

- 含有冒犯性的术语的名称

- 不必要地包含变量类型的名称（例如：`id_to_name_dict`）

#### 命名约定

- “Internal”表示模块内部或类内部的受保护或私有成员。

- 在模块变量和函数上添加单个下划线 (`_`) 有一定程度的保护作用（静态检查器会标记受保护的成员访问）。请注意，单元测试可以访问受保护的常量，不会有问题。

- 在实例变量或方法前面加上双下划线 (`__`，也称为 “dunder”)，实际上使得该变量或方法对其类私有（使用名称改写）；我们不鼓励这样做，因为它会影响可读性和可测试性，并且实际上并不是私有的。应该优先使用单个下划线。

- 将相关的类和顶级函数放在同一个模块中。与 Java 不同，不需要将一个模块限制为一个类。

- 类名使用 CapWords，但模块名使用 lower\_with\_under.py。尽管有一些旧模块的名称是 CapWords.py，但这已经不推荐使用了，因为当模块碰巧以类命名时，这会导致混淆。（"等等——我写了 `import StringIO` 还是 `from StringIO import StringIO`？"）

- 新的单元测试文件应遵循 PEP 8 兼容的 lower\_with\_under 方法命名，例如 `test_<method_under_test>_<state>`。为了与遵循 CapWords 函数名称的旧模块保持一致，在方法名称以 `test` 开头时，下划线可能会出现在方法名称中，以分隔名称的逻辑组件。一个可能的模式是 `test<MethodUnderTest>_<state>`。

#### 文件命名

Python 文件名必须具有 `.py` 扩展名，不能包含破折号 (`-`)。这样可以使它们能够被导入和进行单元测试。如果希望可执行文件在没有扩展名的情况下也可以访问，则可以使用符号链接或一个包含 `exec "$0.py" "$@"` 的简单 bash 包装器。

#### 源自[Guido](https://en.wikipedia.org/wiki/Guido_van_Rossum)建议的指南

<table rules="all" border="1" summary="Guidelines from Guido's Recommendations"
       cellspacing="2" cellpadding="2">

  <tr>
    <th>类型</th>
    <th>公共</th>
    <th>内部</th>
  </tr>

  <tr>
    <td>包</td>
    <td><code>lower_with_under</code></td>
    <td></td>
  </tr>

  <tr>
    <td>模块</td>
    <td><code>lower_with_under</code></td>
    <td><code>_lower_with_under</code></td>
  </tr>

  <tr>
    <td>类</td>
    <td><code>CapWords</code></td>
    <td><code>_CapWords</code></td>
  </tr>

  <tr>
    <td>异常</td>
    <td><code

>CapWords</code></td>
    <td></td>
  </tr>

  <tr>
    <td>函数</td>
    <td><code>lower_with_under()</code></td>
    <td><code>_lower_with_under()</code></td>
  </tr>

  <tr>
    <td>全局/类常量</td>
    <td><code>CAPS_WITH_UNDER</code></td>
    <td><code>_CAPS_WITH_UNDER</code></td>
  </tr>

  <tr>
    <td>全局/类变量</td>
    <td><code>lower_with_under</code></td>
    <td><code>_lower_with_under</code></td>
  </tr>

  <tr>
    <td>实例变量</td>
    <td><code>lower_with_under</code></td>
    <td><code>_lower_with_under</code> (protected)</td>
  </tr>

  <tr>
    <td>方法名</td>
    <td><code>lower_with_under()</code></td>
    <td><code>_lower_with_under()</code> (protected)</td>
  </tr>

  <tr>
    <td>函数/方法参数</td>
    <td><code>lower_with_under</code></td>
    <td></td>
  </tr>

  <tr>
    <td>局部变量</td>
    <td><code>lower_with_under</code></td>
    <td></td>
  </tr>

</table>

#### 数学符号

对于数学密集型代码，当短变量名称违反风格指南时，最好使用与参考论文或算法中已建立的符号相匹配的名称。在这样做时，在注释或文档字符串中引用所有命名约定的来源，或者如果无法访问源，则清楚地记录命名约定。对于公共 API，最好使用符合 PEP8 的 `descriptive_names`，因为这些名称更有可能在上下文之外被遇到。

### main

在 Python 中，`pydoc` 和单元测试要求模块可以被导入。如果一个文件被用作可执行文件，则其主要功能应该在一个 `main()` 函数中，您的代码在执行主程序之前应该始终检查 `if __name__ == '__main__'`，以便在模块被导入时不会执行。

当使用 [absl](https://github.com/abseil/abseil-py) 时，使用 `app.run`：

```python
from absl import app
...

def main(argv: Sequence[str]):
    # 处理非标志参数
    ...

if __name__ == '__main__':
    app.run(main)
```

否则，使用：

```python
def main():
    ...

if __name__ == '__main__':
    main()
```

所有顶级代码都将在导入模块时执行。在执行 `pydoc` 时，要小心不要调用函数、创建对象或执行其他不应该执行的操作。

### 函数长度

首选小而功能聚焦的函数。

我们认识到长函数有时是合适的，因此并未对函数长度设置硬性限制。如果一个函数超过了大约 40 行，请考虑是否可以将其拆分而不损害程序的结构。

即使您的长函数现在工作得很完美，几个月后修改它的人可能会添加新的行为。这可能导致难以发现的错误。保持函数简短和简单可以使其他人更容易阅读和修改您的代码。

在处理一些代码时，您可能会遇到长而复杂的函数。不要被修改现有代码吓倒：如果与这样的函数一起工作证明很困难，您发现错误很难调试，或者您希望在多个不同的上下文中使用其中的一部分，请考虑将函数拆分为更小和更易管理的部分。

### 类型注释

#### 通用规则

- 熟悉 [PEP-484](https://peps.python.org/pep-0484/)。

- 通常不需要对 `self` 或 `cls` 进行注释。
    - 如果对于正确的类型信息而言是必要的，可以使用 [`Self`](https://docs.python.org/3/library/typing.html#typing.Self)，例如：

    ```python
    from typing import Self

    class BaseClass:
      @classmethod
      def create(cls) -> Self:
        ...

      def difference(self, other: Self) -> float:
        ...
    ```

- 同样，不必强制注释 `__init__` 的返回值（其中 `None` 是唯一有效的选项）。

- 如果任何其他变量或返回的类型不应该被表达，使用 `Any`。

- 您不需要对模块中的所有函数进行注释。

  - 至少注释您的公共 API。
  - 在安全性和清晰性之间、灵活性之间取得良好平衡的判断。
  - 对容易出现与类型相关的错误的代码进行注释（以前的错误或复杂性）。
  - 对难以理解的代码进行注释。
  - 当代码从类型的角度变得稳定时进行注释。在许多情况下，您可以在成熟的代码中注释所有函数，而不会失去太多的灵活性。

#### 换行

尝试遵循现有的[缩进](#indentation) 规则。

在注释后，许多函数签名将变为“每行一个参数”。为了确保返回类型也有自己的行，可以在最后一个参数后面放置逗号。

```python
def my_method(
    self,
    first_var: int,
    second_var: Foo,
    third_var: Bar | None,
) -> int:
  ...
```

始终优先在变量之间断行，而不是在变量名和类型注释之间断行。然而，如果一切都适合同一行，就继续进行。

```python
def my_method(self, first_var: int) -> int:
  ...
```

如果函数名、最后一个参数和返回类型的组合过长，可以在新的一行缩进 4 个字符。在使用换行时，最好将每个参数和返回类型放在自己的行上，并将闭合括号与 `def` 对齐：

```python
正确:
def my_method(
    self,
    other_arg: MyLongType | None,
) -> tuple[MyLongType1, MyLongType1]:
  ...
```

可选地，返回类型可以放在与最后一个参数相同的行上：

```python
正确:
def my_method(
    self,
    first_var: int,
    second_var: int) -> dict[OtherLongType, MyLongType]:
  ...
```

`pylint`
允许将闭合括号移动到新的一行，并与开头对齐，但这样不够可读。

```python
错误:
def my_method(self,
              other_arg: MyLongType | None,
             ) -> dict[OtherLongType, MyLongType]:
  ...
```

如上例所示，最好不要在类型之间断行。然而，有时它们太长了，无法放在一行上（尽量保持子类型不被断行）。

```python
def my_method(
    self,
    first_var: tuple[list[MyLongType1],
                     list[MyLongType2]],
    second_var: list[dict[
        MyLongType3, MyLongType4]],
) -> None:
  ...
```

如果单个名称和类型过长，请考虑使用[别名](#typing-aliases)。最后的选择是在冒号后面断行并缩进 4 个字符。

```python
正确:
def my_function(
    long_variable_name:
        long_module_name.LongTypeName,
) -> None:
  ...
```

```python
错误:
def my_function(
    long_variable_name: long_module_name.
        LongTypeName,
) -> None:
  ...
```

#### 前置声明

如果你需要使用一个尚未定义的类名（来自同一模块）——例如，如果你需要在类的声明中使用类名，或者如果你在代码后面定义了类并在前面使用了它——可以使用 `from __future__ import annotations` 或者在类名周围使用字符串。

```python
正确:
from __future__ import annotations

class MyClass:
  def __init__(self, stack: Sequence[MyClass], item: OtherClass) -> None:

class OtherClass:
  ...
```

```python
正确:
class MyClass:
  def __init__(self, stack: Sequence['MyClass'], item: 'OtherClass') -> None:

class OtherClass:
  ...
```

#### 3.19.4 默认值

根据 [PEP-008](https://peps.python.org/pep-0008/#other-recommendations) ，*只有*在参数具有类型注释和默认值的情况下，才在 `=` 周围使用空格。

```python
正确:
def func(a: int = 0) -> int:
  ...
```

```python
错误:
def func(a:int=0) -> int:
  ...
```

#### NoneType

在 Python 类型系统中，`NoneType` 是一种“一等公民”的类型，出于类型化目的，`None` 是对 `NoneType` 的别名。如果参数可以是 `None`，那么它必须声明！你可以使用 `|` 联合类型表达式（在新的 Python 3.10+ 代码中推荐使用），或者旧的 `Optional` 和 `Union` 语法。

使用显式的 `X | None`，而不是隐式的。PEP 484 的早期版本允许将 `a: str = None` 解释为 `a: str | None = None`，但这不再是首选行为。

```python
正确:
def modern_or_union(a: str | int | None, b: str | None = None) -> str:
  ...
def union_optional(a: Union[str, int, None], b: Optional[str] = None) -> str:
  ...
```

```python
错误:
def nullable_union(a: Union[None, str]) -> str:
  ...
def implicit_optional(a: str = None) -> str:
  ...
```

#### 类型别名

你可以声明复杂类型的别名。别名的名称应该采用大驼峰形式。如果别名只在本模块中使用，则应该以 `_Private` 开头。

需要注意的是，`: TypeAlias` 注释只在 3.10以上的版本中受支持。

```python
from typing import TypeAlias

_LossAndGradient: TypeAlias = tuple[tf.Tensor, tf.Tensor]
ComplexTFMap: TypeAlias = Mapping[str, _LossAndGradient]
```

#### 忽略类型

你可以使用特殊注释 `# type: ignore` 在一行上禁用类型检查。

`pytype` 有一个用于禁用特定错误的选项（类似于 lint）：

```python
# pytype: disable=attribute-error
```

#### 类型变量

[*注释的赋值*](#annotated-assignments)：如果内部变量的类型难以或无法推断，请使用注释的赋值来指定其类型 - 在变量名和值之间使用冒号和类型（与具有默认值的函数参数相同）：

  ```python
  a: Foo = SomeUndecoratedFunction()
  ```

[*类型注释*](#type-comments)：虽然你可能在代码库中看到它们（它们在 Python 3.6 之前是必需的），但请不要在行尾添加任何 `# type: <type name>` 注释：

    ```python
    a = SomeUndecoratedFunction()  # type: Foo
    ```

#### 元组 vs 列表

类型化列表只能包含单一类型的对象。类型化元组可以具有单一重复的类型或具有不同类型的固定数量的元素。后者通常用作函数的返回类型。

```python
a: list[int] = [1, 2, 3]
b: tuple[int, ...] = (1, 2, 3)
c: tuple[int, str, float] = (1, "2", 3.5)
```

#### 类型变量

Python 类型系统具有泛型功能。类型变量，例如 `TypeVar` 和 `ParamSpec`，是使用泛型的常见方式。

例如：

```python
from collections.abc import Callable
from typing import ParamSpec, TypeVar
_P = ParamSpec("_P")
_T = TypeVar("_T")
...
def next(l: list[_T]) -> _T:
  return l.pop()

def print_when_called(f: Callable[_P, _T]) -> Callable[_P, _T]:
  def inner(*args: _P.args, **kwargs: _P.kwargs) -> _T:
    print("Function was called")
    return f(*args, **kwargs)
  return inner
```

`TypeVar` 可以被约束：

```python
AddableType = TypeVar("AddableType", int, float, str)
def add(a: AddableType, b: AddableType) -> AddableType:
  return a + b
```

`typing` 模块中常见的预定义类型变量是 `AnyStr`。它用于多个注释，可以是 `bytes` 或 `str`，并且必须都是相同类型的。

```python
from typing import AnyStr
def check_length(x: AnyStr) -> AnyStr:
  if len(x) <= 42:
    return x
  raise ValueError()
```

类型变量必须有描述性的名称，除非满足以下所有条件：

* 不是外部可见的
* 没有约束

```python
正确:
  _T = TypeVar("_T")
  _P = ParamSpec("_P")
  AddableType = TypeVar("AddableType", int, float, str)
  AnyFunction = TypeVar("AnyFunction", bound=Callable)
```

```python
错误:
  T = TypeVar("T")
  P = ParamSpec("P")
  _T = TypeVar("_T", int, float, str)
  _F = TypeVar("_F", bound=Callable)
```

#### 字符串类型 

> 不要在新代码中使用 `typing.Text`。它只用于 Python 2/3 兼容性。

对于字符串/文本数据，请使用 `str`。对于处理二进制数据的代码，请使用 `bytes`。

```python
def deals_with_text_data(x: str) -> str:
  ...
def deals_with_binary_data(x: bytes) -> bytes:
  ...
```

如果函数的所有字符串类型始终相同，例如上面的代码中返回类型与参数类型相同，则使用 [AnyStr](#typing-type-var)。


#### 用于类型注释的导入 

对于用于支持静态分析和类型检查的 `typing` 或 `collections.abc` 模块中的符号（包括类型、函数和常量），始终导入符号本身。这样做可以使常见的注释更简洁，并符合全球使用的类型化实践。你可以明确允许一次从 `typing` 和 `collections.abc` 模块中导入多个特定符号。例如：

```python
from collections.abc import Mapping, Sequence
from typing import Any, Generic, cast, TYPE_CHECKING
```

由于这种导入方式会将项目添加到局部命名空间中，因此在 `typing` 或 `collections.abc` 中的名称应该与关键字类似，不应在你的 Python 代码中定义，无论是类型化的还是非类型化的。如果在模块中的名称与类型发生冲突，请使用 `import x as y` 导入它。

```python
from typing import Any as AnyType
```

在可用的情况下，优先使用内置类型作为注释。Python 支持使用参数化容器类型进行类型注释，通过 [PEP-585](https://peps.python.org/pep-0585/) 在 Python 3.9 中引入。

```python
def generate_foo_scores(foo: set[str]) -> list[float]:
  ...
```

#### 条件导入 

只有在需要避免运行时的额外导入以进行类型检查时才使用条件导入。这种模式不建议使用；应优先选择替代方案，例如重构代码以允许顶级导入。

仅用于类型注释的导入可以放在一个 `if TYPE_CHECKING:` 块中。

- 条件导入的类型需要被引用为字符串，以便与 Python 3.6 兼容，因为注释表达式实际上是在 Python 3.6 中执行的。
- 只有仅用于类型化的实体应该在此处定义；这包括别名。否则，它将是运行时错误，因为模块将不会在运行时导入。
- 此块应该位于所有常规导入之后。
- 在类型化导入列表中不应该有空行。
- 将此列表排序，就像它是一个常规导入列表一样。

```python
import typing
if typing.TYPE_CHECKING:
  import sketch
def f(x: "sketch.Sketch"): ...
```

#### 循环依赖 

由类型化引起的循环依赖是代码异味。这样的代码是重构的好候选者。虽然从技术上讲可以保持循环依赖，但是各种构建系统不会让你这样做，因为每个模块都必须依赖于另一个模块。

用 `Any` 替换创建循环依赖导入的模块。为其设置一个有意义的名称的 [别名](#typing-aliases)，并在此模块中使用真实类型名称（`Any` 的任何属性都是 `Any`）。别名定义应该与最后一个导入之间空一行。

```python
from typing import Any

some_mod = Any  # some_mod.py imports this module.
...

def my_method(self, var: "some_mod.SomeType") -> None:
  ...
```

<a id="typing-generics"></a>
<a id="s3.19.15-generics"></a>
<a id="31915-generics"></a>

<a id="generics"></a>
#### 3.19.15 泛型 

在注释时，优先为泛型类型指定类型参数；否则，[泛型的参数将被假定为 `Any`](https://peps.python.org/pep-0484/#the-any-type)。

```python
# 正确:
def get_names(employee_ids: Sequence[int]) -> Mapping[int, str]:
  ...
```

```python
# 错误:
# This is interpreted as get_names(employee_ids: Sequence[Any]) -> Mapping[Any, Any]
def get_names(employee_ids: Sequence) -> Mapping:
  ...
```

如果泛型的最佳类型参数是 `Any`，则明确指定它，但请记住，在许多情况下 [`TypeVar`](#typing-type-var) 可能更合适：

```python
# 错误:
def get_names(employee_ids: Sequence[Any]) -> Mapping[Any, str]:
  """Returns a mapping from employee ID to employee name for given IDs."""
```

```python
# 正确:
_T = TypeVar('_T')
def get_names(employee_ids: Sequence[_T]) -> Mapping[_T, str]:
  """Returns a mapping from employee ID to employee name for given IDs."""
```

## 4 分享最后的话 

*保持一致性*。

如果你在编辑代码，请花几分钟时间查看周围的代码并确定其风格。如果它们在索引变量名中使用 `_idx` 后缀，你也应该这样做。如果它们的注释周围有小框框的井号，你的注释也应该有小框框的井号。

拥有代码风格指南的目的是有一个共同的编码词汇，这样人们就可以专注于你的意思而不是你的表达方式。我们在这里提供全局风格规则，这样人们就知道词汇，但本地风格也很重要。如果你添加到文件中的代码看起来与周围的现有代码差异很大，那么当读者阅读它时，他们会失去节奏感。

但是，一致性也有限度。它在本地和全局风格未指定的选择上更为重要。通常不应该使用一致性作为在不考虑新样式的优点或代码库随时间趋于收敛于新样式的倾向的情况下做旧样式的理由。
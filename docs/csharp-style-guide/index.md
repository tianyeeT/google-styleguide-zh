# Google Csharp 风格指南

这个风格指南适用于 Google 内部开发的 C# 代码，并且是 Google 的默认 C# 代码风格。它做出了符合 Google 其他语言风格的风格选择，比如 Google C++ 风格和 Google Java 风格。

## 格式指南

### 命名规则

命名规则遵循[微软的 C# 命名指南](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/naming-guidelines)。在微软的命名指南未指定的情况下（例如私有和局部变量），规则取自 [CoreFX C# 编码指南](https://github.com/dotnet/runtime/blob/master/docs/coding-guidelines/coding-style.md)

规则概述：

#### 代码

* 类、方法、枚举、公共字段、公共属性、命名空间的名称：`PascalCase`。
* 局部变量、参数的名称：`camelCase`。
* 私有、受保护、内部和受保护的内部字段和属性的名称：`_camelCase`。
* 命名约定不受 `const`、`static`、`readonly` 等修饰符的影响。
* 对于大小写，一个“单词”是指没有内部空格的任何内容，包括缩写。例如，使用 `MyRpc` 而不是 ~~`MyRPC`~~。
* 接口的名称以 `I` 开头，例如 `IInterface`。

#### 文件

* 文件名和目录名是 `PascalCase`，例如 `MyFile.cs`。
* 在可能的情况下，文件名应该与文件中主要类的名称相同，例如 `MyClass.cs`。
* 一般来说，每个文件优先使用一个核心类。

### 组织

* 修饰符按照以下顺序出现：`public protected internal private new abstract virtual override sealed static readonly extern unsafe volatile async`。
* `using` 命名空间导入放在顶部，在任何命名空间之前。 `using` 导入的顺序是按字母顺序排列的，除了 `System` 导入总是首先出现。
* 类成员排序：
  * 按以下顺序对类成员进行分组：
    * 嵌套类、枚举、委托和事件。
    * 静态、const和readonly字段。
    * 字段和属性。
    * 构造函数和析构函数。
    * 方法。
  * 在每个组内，元素应按以下顺序排列：
    * 公共。
    * 内部。
    * 受保护内部。
    * 受保护的。
    * 私有。
  * 在可能的情况下，将接口实现分组在一起。

### 空白规则

从 Google Java 风格发展而来。

* 每行最多一个语句。
* 每个语句最多一个赋值。
* 缩进为2个空格，不使用制表符。
* 列限制：100。
* 在打开大括号之前不换行。
* 在大括号和 `else` 之间不换行。
* 即使可选，也使用大括号。
* 在 `if`/`for`/`while` 等之后有空格，逗号之后也有空格。
* 在开括号之后和闭括号之前没有空格。
* 在一元操作符和其操作数之间没有空格。在其他操作符的操作符和每个操作数之间有一个空格。
* 换行风格从 Google C++ 风格指南发展而来，稍作修改以与微软的 C# 格式化工具兼容：
    * 一般来说，行继续缩进 4 个空格。
    * 带大括号的换行（例如列表初始化器、lambda、对象初始化器等）不算作继续。
    * 对于函数定义和调用，如果参数不都适合一行，则应将其拆分为多行，其中每个后续行与第一个参数对齐。如果没有足够的空间，参数可以放在后续行上，并缩进四个空格。下面的代码示例说明了这一点。

### 示例

```c#
using System;                                       // `using`放在顶部，不在命名空间内。

namespace MyNamespace {                             // 命名空间是PascalCase。
                                                    // 在命名空间后缩进。
  public interface IMyInterface {                   // 接口以'I'开头
    public int Calculate(float value, float exp);   // 方法是PascalCase
                                                    // ...和逗号后有空格。
  }

  public enum MyEnum {                              // 枚举是PascalCase。
    Yes,                                            // 枚举器是PascalCase。
    No,
  }

  public class MyClass {                            // 类是PascalCase。
    public int Foo = 0;                             // 公共成员变量是PascalCase。
    public bool NoCounting = false;                 // 鼓励字段初始化程序。
    private class Results {
      public int NumNegativeResults = 0;
      public int NumPositiveResults = 0;
    }
    private Results _results;                       // 私有成员变量是_camelCase。
    public static int NumTimesCalled = 0;
    private const int _bar = 100;                   // const不影响命名约定。
    private int[] _someTable = {                    // 容器初始化器使用2个空格的缩进。
      2, 3, 4,                                      // 
    }

    public MyClass() {
      _results = new Results {
        NumNegativeResults = 1,                     // 对象初始化器使用2个空格的缩进。
        NumPositiveResults = 1,                     // 
      };
    }

    public int CalculateValue(int mulNumber) {      // 开括号之前不换行。
      var resultValue = Foo * mulNumber;            // 局部变量是camelCase。
      NumTimesCalled++;
      Foo += _bar;

      if (!NoCounting) {                            // 一元运算符之后没有空格，if

之后有空格。
        if (resultValue < 0) {                      // 即使是可选的也使用大括号，比较运算符周围有空格。
          _results.NumNegativeResults++;
        } else if (resultValue > 0) {               // 大括号和else之间没有换行。
          _results.NumPositiveResults++;
        }
      }

      return resultValue;
    }

    public void ExpressionBodies() {
      // 对于简单的lambda表达式，如果可能，尽量放在一行上，不需要括号或大括号。
      Func<int, int> increment = x => x + 1;

      // 结尾的大括号与包含开括号的行上的第一个字符对齐。
      Func<int, int, long> difference1 = (x, y) => {
        long diff = (long)x - y;
        return diff >= 0 ? diff : -diff;
      };

      // 如果在一个继续的换行之后定义，则将整个主体缩进。
      Func<int, int, long> difference2 =
          (x, y) => {
            long diff = (long)x - y;
            return diff >= 0 ? diff : -diff;
          };

      // 内联lambda参数也遵循这些规则。如果它们包括lambda，则最好在参数组之前放置一个前导换行。
      CallWithDelegate(
          (x, y) => {
            long diff = (long)x - y;
            return diff >= 0 ? diff : -diff;
          });
    }

    void DoNothing() {}                             // 空块可以简洁。

    // 如果可能，通过将换行与第一个参数对齐来包装参数。
    void AVeryLongFunctionNameThatCausesLineWrappingProblems(int longArgumentName,
                                                             int p1, int p2) {}

    // 如果将参数行与第一个参数对齐不适合，或者很难阅读，那么在新行上用4个空格缩进所有参数。
    void AnotherLongFunctionNameThatCausesLineWrappingProblems(
        int longArgumentName, int longArgumentName2, int longArgumentName3) {}

    void CallingLongFunctionName() {
      int veryLongArgumentName = 1234;
      int shortArg = 1;
      // 如果可能，通过将换行与第一个参数对齐来包装参数。
      AnotherLongFunctionNameThatCausesLineWrappingProblems(shortArg, shortArg,
                                                            veryLongArgumentName);
      // 如果将参数行与第一个参数对齐不适合，或者很难阅读，那么在新行上用4个空格缩进所有参数。
      AnotherLongFunctionNameThatCausesLineWrappingProblems(
          veryLongArgumentName, veryLongArgumentName, veryLongArgumentName);
    }
  }
}
```

## C# 编码指南

### 常量

* 可以声明为 `const` 的变量和字段应始终声明为 `const`。
* 如果无法使用 `const`，则 `readonly` 可以是一个合适的替代方案。
* 优先选择命名常量而不是魔术数字。

### IEnumerable vs IList vs IReadOnlyList

* 对于输入，使用尽可能严格的集合类型，例如 `IReadOnlyCollection` / `IReadOnlyList` / `IEnumerable` 作为方法的输入，当输入应该是不可变的时。
* 对于输出，如果将返回的容器所有权传递给所有者，请优先使用 `IList` 而不是 `IEnumerable`。如果不传递所有权，则使用最严格的选项。

### 生成器 vs 容器

* 根据情况进行判断，注意：
  * 生成器代码通常比填充容器的代码更难读。
  * 如果结果将被惰性处理，例如当不需要所有结果时，则生成器代码可能性能更好。
  * 直接通过 `ToList()` 将生成器代码转换为容器的代码将比直接填充容器的代码性能差。
  * 多次调用的生成器代码将比多次迭代容器慢得多。

### 属性样式

* 对于单行只读属性，尽可能使用表达式体属性（`=>`）。
* 对于其他所有情况，请使用旧的`{ get; set; }`语法。

### 表达式体语法

例如：

```c#
int SomeProperty => _someProperty
```

* 谨慎使用表达式体语法在 lambda 和属性中。
* 不要在方法定义中使用。这将在 C# 7 启用时进行审查，该语法将被广泛使用。
* 与方法和其他代码作用域块一样，将关闭的大括号与包含开括号的行上的第一个字符对齐。请参阅示例代码以了解示例。

### 结构体和类：

* 结构体与类非常不同：

  * 结构体始终通过值传递。
  * 将值分配给返回的结构体的成员不会修改原始结构体 - 例如，`transform.position.x = 10` 不会将 transform 的 position.x 设置为 10；在这里，position 是一个按值返回 `Vector3` 的属性，因此这只是设置原始副本的 x 参数。

* 几乎总是使用类。

* 当类型可以像其他值类型一样处理时考虑使用结构体。例如，如果类型的实例很小且通常生命周期很短，或者通常嵌入在其他对象中。良好的示例包括 Vector3、Quaternion 和 Bounds。

* 请注意，这些指导方针在团队之间可能会有所不同，例如，性能问题可能会强制使用结构体。

### Lambda vs 命名方法

* 如果lambda不是微不足道的（例如，多于几个语句，不包括声明），或者在多个地方重复使用，则它可能应该是一个命名方法。

### 字段初始化程序

* 通常鼓励使用字段初始化程序。

### 扩展方法

* 仅在原始类的源不可用时，或者在更改源不可行时使用扩展方法。
* 仅在要添加的功能是“核心”通用功能且适合添加到原始类的源代码时使用扩展方法。
  * 注意 - 如果我们拥有被扩展的类的源代码，并且原始类的维护者不想添加该功能，则最好不要使用扩展方法。
* 仅将扩展方法放入到处都可用的核心库中 - 仅在某些代码中可用的扩展将成为可读性问题。
* 请注意，使用扩展方法总是会使代码模糊不清，因此更倾向于不要添加它们。

### ref 和 out

* 对于不是输入的返回，请使用 `out`。
* 在方法定义中，将 `out` 参数放在所有其他参数之后。
* 应该很少使用 `ref`，仅在必要时才对输入进行修改。
* 不要将 `ref` 用作传递结构体的优化。只有在提供的容器需要被完全替换为另一个容器实例时才需要 `ref`。

### LINQ

* 通常情况下，优先使用单行 LINQ 调用和命令式代码，而不是长链的 LINQ。混合命令式代码和大量链接的 LINQ 通常难以阅读。
* 优先使用成员扩展方法而不是 SQL 风格的 LINQ 关键字 - 例如，优先使用 `myList.Where(x)` 而不是 `myList where x`。
* 避免对任何长于单个语句的内容使用 `Container.ForEach(...)`。

### 数组 vs List

* 通常情况下，对于公共变量、属性和返回类型，优先使用 `List<>` 而不是数组（请注意上面有关 `IList` / `IEnumerable` / `IReadOnlyList` 的指导）。
* 当容器的大小可以改变时，请使用 `List<>`。
* 在容器的大小是固定的并且在构造时已知的情况下，请使用数组。
* 对于多维数组，请使用数组。
* 注意：
  * 数组和 `List<>` 都表示线性、连续的容器。
  * 与C++数组相似，数组具有固定的容量，而 `List<>` 可以添加。
  * 在某些情况下，数组的性能更好，但一般来说 `List<>` 更灵活。

### 文件夹和文件位置

* 与项目保持一致。
* 在可能的情况下，优先使用平面结构。

### 使用元组作为返回类型

* 通常情况下，优先使用命名类类型而不是 `Tuple<>`，特别是在返回复杂类型时。

### 字符串内插 vs `String.Format()` vs `String.Concat` vs `operator+`

* 通常情况下，使用最容易阅读的方法，特别是用于日志记录和断言消息。
* 请注意，链式 `operator+` 连接会更慢，并且会导致大量内存消耗。
* 如果性能是一个问题，对于多个字符串连接，`StringBuilder`会更快。

### `using`

*   通常情况下，不要使用 `using` 别名长类型名称。通常这是一个 `Tuple<>` 需要转换为类的迹象。
  * 例如，`using RecordList = List<Tuple<int, float>>` 可能应该是一个命名类。
* 请注意，`using` 语句只能在文件范围内使用，因此使用范围有限。类型别名对外部用户不可用。

### 对象初始化程序语法

例如：

```c#
var x = new SomeClass {
  Property1 = value1,
  Property2 = value2,
};
```

* 对于“普通的旧数据”类型，对象初始化程序语法是可以接受的。
* 避免在具有构造函数的类或结构中使用此语法。
* 如果跨多行拆分，请将缩进设置为一个块级别。

### 命名空间命名

* 通常情况下，命名空间应不超过2个级别。
* 不要强制文件/文件夹布局与命名空间匹配。
* 对于共享的库/模块代码，请使用命名空间。对于子“应用程序”代码，例如 `unity_app`，命名空间不是必需的。
* 新的顶级命名空间名称必须是全局唯一且可识别的。

### 结构的默认值/空返回

* 优先返回“成功”布尔值和结构体 `out` 值。
* 在性能不是问题，结果代码明显更易读的情况下（例如，链式空条件运算符与深层嵌套的if语句相比），可接受使用可空结构。
* 注意：

  * 可空结构很方便，但强调了谷歌偏好避免的一般“null 是失败”的模式。如果有足够的需求，我们将来可能会研究 `StatusOr` 等价物。

### 在迭代时从容器中移除

C#（像许多其他语言一样）没有为在迭代时从容器中移除项提供明显的机制。有几种选择：

* 如果只需删除满足某些条件的项，请使用 `somelist.RemoveAll(somePredicate)`。
* 如果在迭代中需要进行其他工作，则 `RemoveAll` 可能不够。一个常见的替代模式是在循环外创建一个新容器，将要保留的项插入新容器，并在迭代结束时将原始容器与新容器交换。

### 调用委托

* 调用委托时，请使用 `Invoke()` 并使用空条件运算符 - 例如 `SomeDelegate?.Invoke()`。这清楚地标记了调用点的调用为“正在调用的委托”。空检查简洁且对线程竞态条件具有稳健性。

### `var`关键字

* 如果使用`var`有助于提高可读性，避免嘈杂、明显或不重要的类型名称，那么鼓励使用它。
* 鼓励：

    * 当类型是显而易见的时候 - 例如，`var apple = new Apple();`，或者`var request = Factory.Create<HttpRequest>();`
    * 对于仅直接传递给其他方法的临时变量 - 例如，`var item = GetItem(); ProcessItem(item);`

* 不鼓励：

  * 在处理基本类型时 - 例如，`var success = true;`
  * 在处理编译器解析的内置数字类型时 - 例如，`var number = 12 * ReturnsFloat();`
  * 当用户明显受益于知道类型时 - 例如，`var listOfItems = GetList();`

### 属性

* 属性应出现在与其相关联的字段、属性或方法的上一行，并与成员之间由一个换行符分隔。
* 多个属性应由换行符分隔。这样做可以更轻松地添加和删除属性，并确保每个属性都易于搜索。

### 参数命名

源自 Google C++ 风格指南。

当函数参数的含义不明显时，考虑以下解决方案之一：

* 如果参数是文字常量，并且相同的常量在多个函数调用中以一种暗示它们是相同的方式使用，则使用命名常量使该约束明确，并保证其保持不变。
* 考虑更改函数签名以用 `enum` 参数替换 `bool` 参数。这将使参数值自描述。
* 使用命名变量替换大型或复杂的嵌套表达式。
* 考虑使用[命名参数](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments)在调用站点清晰地说明参数的含义。
* 对于具有多个配置选项的函数，请考虑定义一个单独的类或结构来保存所有选项，并在调用时传递该类的实例。这种方法有几个优点。在调用站点上，选项通过名称引用，这清楚地说明了它们的含义。它还减少了函数参数数量，使函数调用更容易阅读和编写。作为额外的好处，当添加另一个选项时，不需要更改调用站点。

考虑以下示例：

```c#
// 错误 - 这些参数是什么？
DecimalNumber product = CalculateProduct(values, 7, false, null);
```

对比：

```c#
// 正确
ProductOptions options = new ProductOptions();
options.PrecisionDecimals = 7;
options.UseCache = CacheUsage.DontUseCache;
DecimalNumber product = CalculateProduct(values, options, completionDelegate: null);
```
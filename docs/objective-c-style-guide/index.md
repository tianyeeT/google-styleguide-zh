# Google Objective-C 风格指南

> Objective-C 是 C 语言的动态、面向对象的扩展。它旨在易于使用和阅读，同时支持复杂的面向对象设计。它是 OS X 和 iOS 应用程序的主要开发语言。
> 
> 苹果已经撰写了一份非常好的、广为接受的[Cocoa 编码指南](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)，用于 Objective-C。请在阅读本指南的同时参考该指南。
> 
> 本文档的目的是描述应用于 iOS 和 OS X 代码的 Objective-C（以及 Objective-C++）编码指南和实践。这些指南在其他项目和团队中经过了时间的演变和验证。Google 开发的开源项目符合本指南的要求。
> 
> 请注意，本指南不是 Objective-C 教程。我们假设读者已经熟悉了这种语言。如果你对 Objective-C 不熟悉或需要复习，请阅读[《使用 Objective-C 编程》](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html)。

## 原则

### 为读者而优化，而不是为写者

代码库通常存在很长时间，花在阅读代码上的时间比编写代码要多。我们明确选择优化我们代码库中的典型软件工程师的体验，使其能够更好地阅读、维护和调试代码，而不是为了编写代码的便利。例如，在代码片段中出现令人惊讶或不寻常的情况时，为读者留下文本提示是有价值的。

### 保持一致性

当样式指南允许多种选择时，最好选择一种选项，而不是混合使用多种选项。在整个代码库中保持一种样式的一致性，可以让工程师专注于其他（更重要的）问题。一致性还能更好地实现自动化，因为一致的代码可以更高效地开发和操作格式化或重构代码的工具。在许多情况下，归因于“保持一致性”的规则归结为“只选择一种，不要担心其他方式”；允许在这些点上灵活性的潜在价值被人们争论的成本所超过。

### 与 Apple SDK 保持一致

与苹果 SDK 使用 Objective-C 的方式保持一致，具有与我们代码库内部一致性相同的价值。如果 Objective-C 功能解决了一个问题，那么使用它是合理的。然而，有时语言特性和习惯用法存在缺陷，或者只是基于不普遍的假设而设计。在这些情况下，限制或禁止语言特性或习惯用法是合适的。

### 样式规则应该起到作用

样式规则的好处必须足够大，以至于能够正当地要求工程师记住它们。该好处是相对于没有该规则的代码库而言进行衡量的，因此，即使针对一个非常有害的实践制定了规则，如果人们不太可能使用该实践，该规则仍可能具有很小的好处。这个原则主要解释了我们没有的规则，而不是我们有的规则：例如，goto 违反了许多以下原则，但由于极其罕见，所以没有讨论它。

## 示例

有人说，一个示例相当于一千字，因此让我们从一个示例开始，它应该让你对样式、间距、命名等有所了解。

以下是一个示例头文件，演示了 `@interface` 声明的正确注释和间距。

```objectivec 
// 正确:

#import <Foundation/Foundation.h>

@class Bar;

/**
 * 展示良好 Objective-C 风格的示例类。所有接口、类别和协议（即头文件中的所有非平凡顶级声明）必须进行注释。注释必须与所述对象相邻。
 */
@interface Foo : NSObject

/** 已保留的 Bar。 */
@property(nonatomic) Bar *bar;

/** 当前的绘图属性。 */
@property(nonatomic, copy) NSDictionary<NSString *, NSNumber *> *attributes;

/**
 * 方便创建方法。
 * 有关 @c bar 的详细信息，请参见 -initWithBar:。
 *
 * @param bar 用于 fooing 的字符串。
 * @return Foo 的一个实例。
 */
+ (instancetype)fooWithBar:(Bar *)bar;

/**
 * 使用提供的 Bar 实例初始化并返回一个 Foo 对象。
 *
 * @param bar 表示执行某项操作的事物的字符串。
 */
- (instancetype)initWithBar:(Bar *)bar NS_DESIGNATED_INITIALIZER;

/**
 * 使用 @c blah 执行一些工作。
 *
 * @param blah
 * @return 如果工作已完成，则为 YES；否则为 NO。
 */
- (BOOL)doWorkWithBlah:(NSString *)blah;

@end
```

一个源文件的示例，演示了 `@implementation` 的正确注释和间距。

```objectivec 
// 正确:

#import "Shared/Util/Foo.h"

@implementation Foo {
  /** 用于显示“hi”的字符串。 */
  NSString *_string;
}

+ (instancetype)fooWithBar:(Bar *)bar {
  return [[self alloc] initWithBar:bar];
}

- (instancetype)init {
  // 带有自定义指定初始化方法的类应始终覆盖超类的指定初始化方法。
  return [self initWithBar:nil];
}

- (instancetype)initWithBar:(Bar *)bar {
  self = [super init];
  if (self) {
    _bar = [bar copy];
    _string = [[NSString alloc] initWithFormat:@"hi %d", 3];
    _attributes = @{
      @"

color" : [UIColor blueColor],
      @"hidden" : @NO
    };
  }
  return self;
}

- (BOOL)doWorkWithBlah:(NSString *)blah {
  // 在此处进行工作。
  return NO;
}

@end
```

## 命名

名称应尽可能描述性，但要在合理范围内。遵循标准的 [Objective-C 命名规则](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)。

避免使用非标准缩写（包括非标准首字母缩写和首字母缩写）。不要担心节省水平空间，因为使代码对新读者立即可理解更为重要。例如：

```objectivec 
// 正确:

// 正确的命名。
int numberOfErrors = 0;
int completedConnectionsCount = 0;
tickets = [[NSMutableArray alloc] init];
userInfo = [someObject object];
port = [network port];
NSDate *gAppLaunchDate;
```

```objectivec 
// 避免：

// 需要避免的名称。
int w;
int nerr;
int nCompConns;
tix = [[NSMutableArray alloc] init];
obj = [someObject object];
p = [network port];
```

任何类、类别、方法、函数或变量名称都应在名称中使用大写字母缩写和[首字母缩写](https://en.wikipedia.org/wiki/Initialism)。这遵循了苹果的标准，即在名称中使用大写字母缩写的缩写，如 URL、ID、TIFF 和 EXIF。

C 函数和 typedef 的名称应根据周围代码的情况使用大写字母和驼峰命名法。

### 文件名

文件名应反映它们所包含的类实现的名称，包括大小写。

遵循项目使用的约定。文件扩展名应如下：

扩展名     | 类型
--------- | ---------------------------------
.h        | C/C++/Objective-C 头文件
.m        | Objective-C 实现文件
.mm       | Objective-C++ 实现文件
.cc       | 纯 C++ 实现文件
.c        | C 实现文件

包含可能在多个项目之间共享或在大型项目中使用的代码的文件应具有清晰的唯一名称，通常包括项目或类[前缀](#前缀)。

类别的文件名应包括被扩展的类的名称，例如 GTMNSString+Utils.h 或 NSTextView+GTMAutocomplete.h。

### 前缀

在 Objective-C 中，通常需要使用前缀来避免在全局命名空间中出现命名冲突。类、协议、全局函数和全局常量通常应该使用以大写字母开头的前缀，后跟一个或多个大写字母或数字。

警告：苹果保留了两个字母的前缀，请参阅[使用 Objective-C 编程中的约定](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html)。因此，被认为最佳实践的前缀至少应该有三个字符。

```objectivec 
// 正确:

/** 一个示例错误域。 */
extern NSString *GTMExampleErrorDomain;

/** 获取默认时区。 */
extern NSTimeZone *GTMGetDefaultTimeZone(void);

/** 一个示例委托。 */
@protocol GTMExampleDelegate <NSObject>
@end

/** 一个示例类。 */
@interface GTMExample : NSObject
@end

```

### 类名

类名（以及类别和协议名）应以大写字母开头，并使用混合大小写来分隔单词。

在跨多个应用程序共享的代码中的类和协议必须具有适当的[前缀](#前缀)（例如 GTMSendMessage）。其他类和协议建议使用前缀，但不是必须的。

### 类别命名

类别名称应以适当的[前缀](#前缀)开头，以标识类别作为项目的一部分或开放供通用使用。

类别源文件名应以扩展的类名开头，后跟加号和类别名称，例如 `NSString+GTMParsing.h`。

类别中的方法应该使用类别名称的小写版本加下划线作为前缀（例如 `gtm_myCategoryMethodOnAString:`），以避免在 Objective-C 的全局命名空间中发生冲突。

类名与类别的开头括号之间应有一个空格。

```objectivec 
// 正确:

// UIViewController+GTMCrashReporting.h

/** UIViewController 添加到崩溃报告中的元数据的类别。 */
@interface UIViewController (GTMCrashReporting)

/** 在崩溃报告中表示视图控制器的唯一标识符。 */
@property(nonatomic, setter=gtm_setUniqueIdentifier:) int gtm_uniqueIdentifier;

/** 返回视图控制器当前状态的编码表示形式。 */
- (nullable NSData *)gtm_encodedState;

@end
```

如果一个类没有与其他项目共享，则扩展它的类别可以省略名称前缀和方法名称前缀。

```objectivec 
// 正确:

/** 这个类别扩展了一个不与其他项目共享的类。 */
@interface XYZDataObject (Storage)
- (NSString *)storageIdentifier;
@end
```

### Objective-C 方法名

方法和参数名通常以小写字母开头，然后使用混合大小写。

应尊重适当的大小写，包括在名称开头处。

```objectivec 
// 正确:

+ (NSURL *)URLWithString:(NSString *)URLString;
```

方法名应尽可能像句子一样阅读，这意味着应选择与方法名流畅配合的参数名。Objective-C 方法名通常会很长，但这有一个好处，那就是代码块几乎可以像散文一样阅读，从而使许多实现注释变得不必要。

在第二个及以后的参数名称中，仅在必要时使用诸如“with”、“from”和“to”等介词和连词，以澄清方法的含义或行为。

```objectivec 
// 正确:

- (void)addTarget:(id)target action:(SEL)action;                          // 正确; 不需要连词
- (CGPoint)convertPoint:(CGPoint)point fromView:(UIView *)view;           // 正确; 连词澄清参数
- (void)replaceCharactersInRange:(NSRange)aRange
            withAttributedString:(NSAttributedString *)attributedString;  // 正确.
```

返回对象的方法应该以标识所返回对象的名词开头：

```objectivec 
// 正确:

- (Sandwich *)sandwich;      // 正确.
```

```objectivec 
// 错误:

- (Sandwich *)makeSandwich;  // 错误.
```

访问器方法应该与其获取的对象的名称相同，但不应以“get”开头。例如：

```objectivec 
// 正确:

- (id)delegate;     // 正确.
```

```objectivec 
// 错误:

- (id)getDelegate;  // 错误.
```

返回布尔形容词值的访问器方法的方法名以“is”开头，但这些方法的属性名省略了“is”。

只有属性名称可以使用点表示法，方法名称不能使用。

```objectivec 
// 正确:

@property(nonatomic, getter=isGlorious) BOOL glorious;
- (BOOL)isGlorious;

BOOL isGood = object.glorious;      // 正确.
BOOL isGood = [object isGlorious];  // 正确.
```

```objectivec 
// 错误:

BOOL isGood = object.isGlorious;    // 错误.
```

```objectivec 
// 正确:

NSArray<Frog *> *frogs = [NSArray<Frog *> arrayWithObject:f

rog];
NSEnumerator *enumerator = [frogs reverseObjectEnumerator];  // 正确.
```

```objectivec 
// 错误:

NSEnumerator *enumerator = frogs.reverseObjectEnumerator;    // 错误.
```

有关 Objective-C 命名的更多详细信息，请参阅[苹果命名方法指南](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingMethods.html#//apple_ref/doc/uid/20001282-BCIGIJJF)。

这些准则仅适用于 Objective-C 方法。C++ 方法名称继续遵循 C++ 样式指南中设定的规则。

### 函数名

函数名应以大写字母开头，并且每个新单词都要有一个大写字母（即所谓的“[驼峰命名法](https://en.wikipedia.org/wiki/Camel_case)”或“Pascal case”）。

```objectivec 
// 正确:

static void AddTableEntry(NSString *tableEntry);
static BOOL DeleteFile(const char *filename);
```

因为 Objective-C 不提供命名空间，所以非静态函数应该具有一个[前缀](#前缀)，以最小化名称冲突的可能性。

```objectivec 
// 正确:

extern NSTimeZone *GTMGetDefaultTimeZone(void);
extern NSString *GTMGetURLScheme(NSURL *URL);
```

### 变量名

变量名通常以小写字母开头，并使用混合大小写来分隔单词。

实例变量以下划线开头。文件范围或全局变量以 `g` 为前缀。例如：`myLocalVariable`、`_myInstanceVariable`、`gMyGlobalVariable`。

#### 常见的变量名

读者应该能够从名称中推断出变量的类型，但不要使用匈牙利命名法来表示语法属性，比如变量的静态类型（int 或指针）。

文件范围或全局变量（与常量相对）声明在方法或函数的范围之外应该很少见，并且应该有前缀 g。

```objectivec 
// 正确:

static int gGlobalCounter;
```

#### 实例变量

实例变量名称使用混合大小写，并应以下划线为前缀，例如 `_usernameTextField`。

注意：Google 以前对于 Objective-C 实例变量的惯例是使用尾部下划线。现有项目可以选择在新代码中继续使用尾部下划线，以保持项目代码库中的一致性。在每个类内部应保持前缀或后缀下划线的一致性。

#### 常量

常量符号（使用 `#define` 创建的 const 全局和静态变量以及常量）应该使用混合大小写来分隔单词。

全局和文件范围的常量应具有适当的[前缀](#前缀)。

```objectivec 
// GOOD:

extern NSString *const GTLServiceErrorDomain;

typedef NS_ENUM(NSInteger, GTLServiceError) {
  GTLServiceErrorQueryResultMissing = -3000,
  GTLServiceErrorWaitTimedOut       = -3001,
};
```

因为 Objective-C 不提供命名空间，具有外部链接的常量应该具有最小化名称冲突的前缀，通常像 `ClassNameConstantName` 或 `ClassNameEnumName`。

为了与 Swift 代码互操作，枚举值应该具有扩展 typedef 名称的名称：

```objectivec 
// 正确:

typedef NS_ENUM(NSInteger, DisplayTinge) {
  DisplayTingeGreen = 1,
  DisplayTingeBlue = 2,
};
```

小写的 k 可以作为静态存储期常量的前缀，声明在实现文件中：

```objectivec 
// 正确:

static const int kFileCount = 12;
static NSString *const kUserKey = @"kUserKey";
```

注意：以前的惯例是让公共常量名称以小写 k 开头，后跟项目特定的[前缀](#前缀)。不再推荐这种做法。

## 类型和声明

### 方法声明

如[示例](#示例)所示，在 `@interface` 声明中的推荐顺序为：属性、类方法、初始化方法，最后是实例方法。类方法部分应该以任何便利构造器开始。

### 局部变量

在尽可能窄的作用域内声明变量，并在使用附近进行初始化。

```objectivec 
// 正确:

CLLocation *location = [self lastKnownLocation];
for (int meters = 1; meters < 10; meters++) {
  reportFrogsWithinRadius(location, meters);
}
```

偶尔，为了效率，将变量声明在其使用范围之外会更合适。以下示例将 meters 声明为单独的变量，并在每次循环中不必要地发送 lastKnownLocation 消息：

```objectivec 
// 错误:

int meters;                                         // 错误.
for (meters = 1; meters < 10; meters++) {
  CLLocation *location = [self lastKnownLocation];  // 错误.
  reportFrogsWithinRadius(location, meters);
}
```

在自动引用计数（ARC）下，对 Objective-C 对象的 strong 和 weak 指针会自动初始化为 `nil`，因此对于这些常见情况，不需要显式初始化为 `nil`。然而，对于许多 Objective-C 指针类型，包括使用 `__unsafe_unretained` 所有权修饰符声明的对象指针和 CoreFoundation 对象指针类型，*不*会发生自动初始化。如果有疑问，最好初始化所有 Objective-C 局部变量。

### 无符号整数

避免使用无符号整数，除非匹配系统接口中使用的类型。

使用无符号整数进行数学运算或计数至零时会出现微妙的错误。在数学表达式中只依赖于有符号整数，除非与系统接口中的 NSUInteger 匹配。

```objectivec 
// 正确:

NSUInteger numberOfObjects = array.count;
for (NSInteger counter = numberOfObjects - 1; counter > 0; --counter)
```

```objectivec 
// 错误:

for (NSUInteger counter = numberOfObjects - 1; counter > 0; --counter)  // AVOID.
```

无符号整数可用于标志和位掩码，但通常 NS_OPTIONS 或 NS_ENUM 更合适。

### 大小不一致的类型

由于在 32 位和 64 位构建中大小不同，除了匹配系统接口之外，应避免使用 long、NSInteger、NSUInteger 和 CGFloat 类型。

类型 long、NSInteger、NSUInteger 和 CGFloat 在 32 位和 64 位构建之间大小不同。在处理系统接口公开的值时，使用这些类型是合适的，但在大多数其他计算中应避免使用它们。

```objectivec 
// 正确:

int32_t scalar1 = proto.intValue;

int64_t scalar2 = proto.longValue;

NSUInteger numberOfObjects = array.count;

CGFloat offset = view.bounds.origin.x;
```

```objectivec 
// 错误:

NSInteger scalar2 = proto.longValue;  // 错误.
```

文件和缓冲区大小通常超过 32 位限制，因此应该使用 `int64_t` 声明，而不是使用 `long`、`NSInteger` 或 `NSUInteger`。

## 注释

注释对于保持我们的代码可读性至关重要。以下规则描述了你应该在何处以及如何进行注释。但请记住：尽管注释很重要，但最好的代码是自解释的。为类型和变量赋予合理的名称比使用晦涩的名称然后尝试通过注释来解释它们要好得多。

注意标点符号、拼写和语法；读起来容易的注释比读起来困难的注释更容易。

注释应该与叙述性文本一样可读，具有适当的大写和标点符号。在许多情况下，完整的句子比句子片段更易读。较短的注释，例如在代码行末尾的注释，有时可能不太正式，但要使用一致的风格。
在编写注释时，请为你的受众编写：下一个需要理解你的代码的贡献者。要慷慨一点——下一个可能就是你！

### 文件注释

文件可以选择以其内容的描述开始。
每个文件可以包含以下项目，按顺序排列：
  * 如果需要，许可证模板。选择适用于项目所使用的许可证的适当模板。
  * 如果需要，对文件内容的基本描述。

如果对带有作者行的文件进行了重大更改，请考虑删除作者行，因为修订历史已经提供了更详细和准确的作者记录。

### 声明注释

每个非平凡的接口，无论是公共的还是私有的，都应该有一个相应的注释，描述其目的及其如何与整体架构配合。

应该使用注释来记录类、属性、实例变量、函数、类别、协议声明和枚举。

```objectivec 
// 正确:

/**
 * NSApplication 的委托，用于处理有关应用程序启动和关闭的通知。
 * 由主应用程序控制器拥有。
 */
@interface MyAppDelegate : NSObject {
  /**
   * 正在进行的后台任务（如果有的话）。初始化为 UIBackgroundTaskInvalid。
   */
  UIBackgroundTaskIdentifier _backgroundTaskID;
}

/** 用于创建和管理应用程序的 fetcher 的工厂。 */
@property(nonatomic) GTMSessionFetcherService *fetcherService;

@end
```

对于接口，建议使用 Doxygen 风格的注释，因为 Xcode 解析这些注释以显示格式化的文档。有各种各样的 Doxygen 命令；在项目中要一致使用它们。

如果你已经在文件顶部的注释中详细描述了一个接口，请随意简单地说明：“有关完整描述，请参见文件顶部的注释”，但一定要有某种形式的注释。

此外，每个方法应该有一个注释，解释其功能、参数、返回值、线程或队列假设以及任何副作用。对于公共方法，文档注释应放在头部；对于非平凡的私有方法，文档注释应放在方法之前。

对于方法和函数注释，使用描述形式（“打开文件”）而不是命令形式（“打开文件”）。注释描述了函数的功能；它并不告诉函数该做什么。

如果类、属性或方法对线程使用有假设，请记录这些假设。如果一个类的实例可以被多个线程访问，请特别小心地记录多线程使用的规则和不变式。

任何属性和实例变量的标志值，如 `NULL` 或 `-1`，都应在注释中进行记录。

声明注释解释了如何使用方法或函数。解释方法或函数的实现方式的注释应与声明分开。

### 实现注释

提供解释复杂、微妙或复杂代码部分的注释。

```objectivec 
// 正确:

// 在调用完成处理程序之前将属性设置为 nil，以避免再次进入导致回调再次被调用的风险。
CompletionHandler handler = self.completionHandler;
self.completionHandler = nil;
handler();
```

如果有必要，还可以提供有关考虑或放弃的实现方法的注释。

行尾注释应该与代码之间至少有 2 个空格的分隔。如果你有几个连续的注释行，通常更易读地将它们排成一行。

```objectivec 
// 正确:

[self doSomethingWithALongName];  // 注释之前有两个空格。
[self doSomethingShort];          // 更多的间距以对齐注释。
```

### 消歧符号

为了避免歧义，注释中引用变量名和符号时应优先使用反引号或竖线来引用，而不是内联命名符号的引号。

在 Doxygen 风格的注释中，最好使用等宽文本命令来标识符号，例如 `@c`。

在涉及常见词汇可能使句子看起来构造不佳时，采用标识有助于提供清晰度。常见示例是符号 `count`：

```objectivec 
// 正确的例子:

// 有时 `count` 可能小于零。
```

或者在引用已包含引号的内容时：

```objectivec 
// 正确的例子:

// 记得调用 `StringWithoutSpaces("foo bar baz")`
```

当符号是不言自明时，不需要使用反引号或竖线。

```objectivec 
// 正确的例子:

// 此类用作 GTMDepthCharge 的委托。
```

Doxygen 格式也适用于标识符号。

```objectivec 
// 正确的例子:

/** @param maximum 最大值为 @c count。 */
```

### 对象所有权 

对于不由 ARC 管理的对象，在超出最常见的 Objective-C 用法习惯范围之外时，尽可能使指针所有权模型明确。

#### 手动引用计数 

假定 NSObject 派生对象的实例变量是被保留的；如果它们没有被保留，它们应该被注释为弱引用，或者用 `__weak` 生命周期限定符声明。

例外情况是在 Mac 软件中标记为 `@IBOutlets` 的实例变量，假定它们不被保留。

当实例变量是指向 Core Foundation、C++ 和其他非 Objective-C 对象的指针时，它们应始终被声明为强引用和弱引用，以指示哪些指针是被保留的，哪些不是。Core Foundation 和其他非 Objective-C 对象指针需要明确的内存管理，即使构建为自动引用计数也是如此。

强引用和弱引用声明的示例：

```objectivec 
// 正确的例子:

@interface MyDelegate : NSObject

@property(nonatomic) NSString *doohickey;
@property(nonatomic, weak) NSString *parent;

@end


@implementation MyDelegate {
  IBOutlet NSButton *_okButton;  // 普通的 NSControl；在 Mac 上隐式弱引用

  AnObjcObject *_doohickey;  // 我的 doohickey
  __weak MyObjcParent *_parent;  // 用于发送消息回去（拥有此实例）

  // 非 NSObject 指针...
  CWackyCPPClass *_wacky;  // 强引用，一些跨平台对象
  CFDictionaryRef *_dict;  // 强引用
}
@end
```

#### 自动引用计数 

使用 ARC 时，对象所有权和生命周期是明确的，因此不需要为自动保留的对象添加额外的注释。

## C 语言特性 

### 宏 

避免使用宏，特别是在可以使用 `const` 变量、枚举、XCode 片段或 C 函数的情况下。

宏使你看到的代码与编译器看到的代码不同。现代 C 使得传统用于常量和实用函数的宏的使用变得不必要。只有在没有其他解决方案可用时才应使用宏。

如果需要宏，请使用唯一的名称以避免在编译单元中出现符号冲突的风险。如果可行，请在使用后限制作用域，使用 `#undefining` 宏。

宏名称应使用 `SHOUTY_SNAKE_CASE`，即所有大写字母之间用下划线分隔。类似函数的宏可能使用 C 函数命名惯例。不要定义看起来像 C 或 Objective-C 关键字的宏。

```objectivec 
// 正确的例子:

#define GTM_EXPERIMENTAL_BUILD ...      // 正确的例子

// 除非 X > Y，否则断言
#define GTM_ASSERT_GT(X, Y) ...         // 正确的例子，宏风格。

// 除非 X > Y，否则断言
#define GTMAssertGreaterThan(X, Y) ...  // 正确的例子，函数风格。
```

```objectivec 
// 错误的例子:

#define kIsExperimentalBuild ...        // 错误的例子

#define unless(X) if(!(X))              // 错误的例子
```

避免扩展到不平衡的 C 或 Objective-C 结构的宏。避免引入作用域的宏，或者可能在块中捕获值的宏。

避免生成用作公共 API 的头文件中的类、属性或方法定义的宏。这只会使代码难以理解，而语言本身已经有更好的方法来做到这一点。

避免生成方法实现的宏，或者生成稍后在宏外部使用的变量声明的宏。宏不应该使代码难以理解，因为它们隐藏了变量的声明位置和方式。

```objectivec 
// 错误的例子:

#define ARRAY_ADDER(CLASS) \
  -(void)add ## CLASS ## :(CLASS *)obj toArray:(NSMutableArray *)array

ARRAY_ADDER(NSString) {
  if (array.count > 5) {              // 错误——'array' 定义在哪里？
    ...
  }
}
```

可接受宏用法的示例包括基于构建设置有条件地编译的断言和调试日志宏——通常情况下，这些宏不会编译到发布版本中。

### 非标准扩展

除非另有规定，否则不得使用 C/Objective-C 的非标准扩展。

编译器支持各种不属于标准 C 的扩展。例如，复合语句表达式（例如 `foo = ({ int x; Bar(&x); x })`）。

`__attribute__` 是一个例外，因为它在 Objective-C API 规范中使用。

条件运算符的二进制形式 `A ?: B` 也是一个例外。

## Cocoa 和 Objective-C 功能

### 标识指定的初始化器

清晰地标识出你的指定初始化器。

对于可能会子类化你的类的人来说，清晰地标识指定的初始化器很重要。这样，他们只需要重写单个初始化器（可能是多个中的一个）来确保调用子类的初始化器。这也有助于以后调试你的类的人理解初始化代码的流程，如果需要的话，他们可以逐步进行调试。使用注释或 `NS_DESIGNATED_INITIALIZER` 宏来标识指定的初始化器。如果使用 `NS_DESIGNATED_INITIALIZER`，请使用 `NS_UNAVAILABLE` 标记不支持的初始化器。

### 覆盖指定的初始化器

当编写需要 `init...` 方法的子类时，请确保覆盖父类的指定初始化器。

如果未覆盖父类的指定初始化器，你的初始化器可能不会在所有情况下被调用，导致难以发现且非常困难的 bug。

### 覆盖的 NSObject 方法放置

将 NSObject 的覆盖方法放置在 `@implementation` 的顶部。

这通常适用于（但不限于）`init...`、`copyWithZone:` 和 `dealloc` 方法。`init...` 方法应该分组在一起，然后是其他典型的 `NSObject` 方法，如 `description`、`isEqual:` 和 `hash`。

用于创建实例的方便类工厂方法可以位于 `NSObject` 方法之前。

### 初始化

不要在 `init` 方法中将实例变量初始化为 `0` 或 `nil`；这样做是多余的。

新分配对象的所有实例变量都会被[初始化为](https://developer.apple.com/library/mac/documentation/General/Conceptual/CocoaEncyclopedia/ObjectAllocation/ObjectAllocation.html) `0`（除了 isa），因此不要通过将变量重新初始化为 `0` 或 `nil` 来使 init 方法变得混乱。

### 头文件中的实例变量应该是 @protected 或 @private

实例变量通常应该在实现文件中声明，或者由属性自动合成。当实例变量在头文件中声明时，它们应该被标记为 `@protected` 或 `@private`。

```objectivec 
// 正确的例子:

@interface MyClass : NSObject {
 @protected
  id _myInstanceVariable;
}
@end
```

### 不要使用 +new

不要调用 `NSObject` 类方法 `new`，也不要在子类中覆盖它。`+new` 很少被使用，并且与初始化器的用法形成鲜明对比。而是使用 `+alloc` 和 `-init` 方法来实例化保留的对象。

### 保持公共 API 简单

保持你的类简单；避免“厨房水槽”API。如果一个方法不需要是公共的，请将其保持在公共接口之外。

与 C++ 不同，Objective-C 不区分公共方法和私有方法；任何消息都可以发送给对象。因此，避免将方法放在公共 API 中，除非实际上期望被类的消费者使用。这有助于减少它们在不期望时被调用的可能性。这包括从父类中覆盖的方法。

由于内部方法实际上并不私有，所以很容易意外地覆盖父类的“私有”方法，从而导致一个非常难以排查的 bug。通常，私有方法应该具有相当独特的名称，以防止子类意外地覆盖它们。

### #import 和 #include

`#import` Objective-C 和 Objective-C++ 头文件，以及 `#include` C/C++ 头文件。

C/C++ 头文件使用 `#include` 包含其他 C/C++ 头文件。在 C/C++ 头文件中使用 `#import` 可以防止未来使用 `#include` 包含，并可能导致意外的编译行为。C/C++ 头文件应该提供自己的 `#define` 保护。

### include的顺序

头文件包含的标准顺序是相关的头文件、操作系统头文件、语言库头文件，最后是其他依赖项的头文件组。

相关的头文件位于其他头文件之前，以确保它没有隐藏的依赖关系。对于实现文件，相关的头文件是头文件。对于测试文件，相关的头文件是包含被测试接口的头文件。

空行可以分隔逻辑上不同的包含头文件组。

在每个组内，包含应按字母顺序排列。

使用相对于项目源目录的路径导入头文件。

```objectivec 
// 正确的例子:

#import "ProjectX/BazViewController.h"

#import <Foundation/Foundation.h>

#include <unistd.h>
#include <vector>

#include "base/basictypes.h"
#include "base/integral_types.h"
#include "util/math/mathutil.h"

#import "ProjectX/BazModel.h"
#import "Shared/Util/Foo.h"
```

### 使用系统框架的整体头文件

对于系统框架和系统库，应该导入整体头文件，而不是包含单独的文件。

虽然从 Cocoa 或 Foundation 等框架中包含单独的系统头文件似乎很诱人，但实际上，如果包含顶级根框架，则编译器的工作量较小。顶级根框架通常是预编译的，可以更快地加载。此外，请记住，对于 Objective-C 框架，应该使用 `@import` 或 `#import`，而不是 `#include`。

```objectivec 
// 正确：

@import UIKit;     // 正确。
#import <Foundation/Foundation.h>     // 正确。
```

```objectivec 
// 错误：

#import <Foundation/NSArray.h>        // 错误。
#import <Foundation/NSString.h>
...
```

### 避免在初始化方法和 `-dealloc` 中发送当前对象的消息

初始化方法和 `-dealloc` 中的代码应避免调用实例方法。

在子类初始化之前，父类初始化完成。在所有类都有机会初始化其实例状态之前，对 self 的任何方法调用可能导致子类在未初始化的实例状态上操作。

`-dealloc` 中也存在类似的问题，方法调用可能导致一个类在已释放的状态上操作。

一个不太明显的情况是属性访问器。这些可以像任何其他选择器一样被覆盖。在初始化方法和 `-dealloc` 中，直接分配和释放 ivars，而不是依赖访问器，是可行的。

```objectivec 
// 正确：

- (instancetype)init {
  self = [super init];
  if (self) {
    _bar = 23;  // 好。
  }
  return self;
}
```

注意将通用初始化代码合并到辅助方法中的情况：

- 方法可以在子类中被覆盖，不管是有意还是由于命名冲突。
- 在编辑辅助方法时，可能不明显代码是从初始化方法中运行的。

```objectivec 
// 错误：

- (instancetype)init {
  self = [super init];
  if (self) {
    self.bar = 23;  // 避免。
    [self sharedMethod];  // 避免。对子类化或未来扩展很脆弱。
  }
  return self;
}
```

```objectivec 
// 正确：

- (void)dealloc {
  [_notifier removeObserver:self];  // 正确。
}
```

```objectivec 
// 避免：

- (void)dealloc {
  [self removeNotifications];  // 错误。
}
```

### 设置器应复制 NSString

接受 `NSString` 的设置器应始终复制它接受的字符串。对于诸如 `NSArray` 和 `NSDictionary` 这样的集合，通常也是如此。

永远不要只保留字符串，因为它可能是 `NSMutableString`。这可以避免在你不知情的情况下，调用者对其进行更改。

接收和持有集合对象的代码也应该考虑到传递的集合可能是可变的，因此集合更安全地被持有为原始的副本或可变副本。

```objectivec 
// 正确：

@property(nonatomic, copy) NSString *name;

- (void)setZigfoos:(NSArray<Zigfoo *> *)zigfoos {
  // 确保我们持有一个不可变集合。
  _zigfoos = [zigfoos copy];
}
```

### 使用轻量级泛型来记录包含的类型

在 Xcode 7 或更新版本上编译的所有项目应使用 Objective-C 的轻量级泛型表示法来为包含的对象类型进行类型标注。

每个 `NSArray`、`NSDictionary` 或 `NSSet` 引用应使用轻量级泛型进行声明，以提高类型安全性，并明确记录用法。

```objectivec 
// 正确：

@property(nonatomic, copy) NSArray<Location *> *locations;
@property(nonatomic, copy, readonly) NSSet<NSString *> *identifiers;

NSMutableArray<MyLocation *> *mutableLocations = [otherObject.locations mutableCopy];
```

如果全面标注的类型变得复杂，考虑使用 typedef 来保持可读性。

```objectivec 
// 正确:

typedef NSSet<NSDictionary<NSString *, NSDate *> *> TimeZoneMappingSet;
TimeZoneMappingSet *timeZoneMappings = [TimeZoneMappingSet setWithObjects:...];
```

使用最具描述性的公共超类或协议。在最通用的情况下，当没有其他信息可用时，使用 id 显式地声明集合为异构集合。

```objectivec 
// 正确:

@property(nonatomic, copy) NSArray<id> *unknowns;
```

### 避免抛出异常 

不要使用 `@throw` 抛出 Objective-C 异常，但应该准备从第三方或操作系统调用中捕获它们。

这遵循了在[苹果的 Cocoa 异常编程主题简介](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Exceptions/Exceptions.html)中使用错误对象进行错误传递的建议。

我们使用 `-fobjc-exceptions` 编译（主要是为了获得 `@synchronized`），但我们不抛出异常。在需要正确使用第三方代码或库时，允许使用 `@try`、`@catch` 和 `@finally`。如果你使用它们，请确切地记录哪些方法你希望抛出异常。

### `nil` 检查 

避免仅为防止向 `nil` 发送消息而进行 `nil` 指针检查。向 `nil` 发送消息[可靠地返回](http://www.sealiesoftware.com/blog/archive/2012/2/29/objc_explain_return_value_of_message_to_nil.html)`nil` 作为指针，作为整数或浮点值为零，结构体初始化为 `0`，_Complex 值等于 `{0, 0}`。

```objectivec 
// 错误:

if (dataSource) {  // 错误。
  [dataSource moveItemAtIndex:1 toIndex:0];
}
```

```objectivec 
// 正确:

[dataSource moveItemAtIndex:1 toIndex:0];  // 正确。
```

请注意，这适用于 `nil` 作为消息目标，而不是作为参数值。单个方法可能会或可能不会安全地处理 `nil` 参数值。

还要注意，这与检查 C/C++ 指针和块指针是否为 `NULL` 不同，运行时不会处理并将导致应用程序崩溃。你仍然需要确保不解引用 `NULL` 指针。

### 可空性

可以使用可空性注释来描述接口的使用方式和行为。可以接受使用可空性区域（例如，`NS_ASSUME_NONNULL_BEGIN` 和 `NS_ASSUME_NONNULL_END`）和显式的可空性注释。优先使用 `_Nullable` 和 `_Nonnull` 关键字，而不是 `__nullable` 和 `__nonnull` 关键字。对于 Objective-C 方法和属性，优先使用上下文敏感、非下划线的关键字，例如 `nonnull` 和 `nullable`。

```objectivec 
// 正确:

/** 代表一个拥有的书籍的类。 */
@interface GTMBook : NSObject

/** 书籍的标题。 */
@property(readonly, copy, nonnull) NSString *title;

/** 书籍的作者，如果存在的话。 */
@property(readonly, copy, nullable) NSString *author;

/** 书籍的拥有者。将 nil 设置为默认拥有者。 */
@property(copy, null_resettable) NSString *owner;

/** 使用标题和可选作者初始化一本书。 */
- (nonnull instancetype)initWithTitle:(nonnull NSString *)title
                               author:(nullable NSString *)author
    NS_DESIGNATED_INITIALIZER;

/** 返回 nil，因为书籍预计有一个标题。 */
- (nullable instancetype)init;

@end

/** 从指定路径的文件加载书籍。 */
NSArray<GTMBook *> *_Nullable GTMLoadBooksFromFile(NSString *_Nonnull path);
```

```objectivec 
// 错误:

NSArray<GTMBook *> *__nullable GTMLoadBooksFromTitle(NSString *__nonnull path);
```

小心假设指针不为 null，因为编译器可能不能保证指针不为 null。

### BOOL 陷阱 

在将一般整数值转换为 `BOOL` 时要小心。避免直接与 `YES` 进行比较。

在 OS X 和 32 位 iOS 构建中，`BOOL` 被定义为有符号 `char`，因此它可能具有除 `YES`（`1`）和 `NO`（`0`）之外的值。不要直接将一般整数值转换为 `BOOL`。

常见错误包括将数组大小、指针值或按位逻辑操作的结果转换为 `BOOL`，这可能根据整数值的最后一个字节的值而导致 `NO` 值。当将通用整数值转换为 `BOOL` 时，应使用三元运算符返回 `YES` 或 `NO` 值。

你可以安全地交换和转换 `BOOL`、`_Bool` 和 `bool`（参见 C++ Std 4.7.4、4.12 和 C99 Std 6.3.1.2）。在 Objective-C 方法签名中使用 `BOOL`。

对于 `BOOL`，使用逻辑运算符（`&&`、`||` 和 `!`）也是有效的，并且将返回可以安全地转换为 `BOOL` 的值，而不需要使用三元运算符。

```objectivec 
// 错误:

- (BOOL)isBold {
  return [self fontTraits] & NSFontBoldTrait;  // 错误。
}
- (BOOL)isValid {
  return [self stringValue];  // 错误。
}
```

```objectivec 
// 正确:

- (BOOL)isBold {
  return ([self fontTraits] & NSFontBoldTrait) ? YES : NO;
}
- (BOOL)isValid {
  return [self stringValue] != nil;
}
- (BOOL)isEnabled {
  return [self isValid] && [self isBold];
}
```

此外，不要直接将 `BOOL` 变量与 `YES` 直接比较。不仅对于精通 C 的人来说更难阅读，而且上面的第一个要点说明返回值可能并不总是你所期望的。

```objectivec 
// 错误:

BOOL great = [foo isGreat];
if (great == YES) {  // 错误。
  // ...be great!
}
```

```objectivec 
// 正确:

BOOL great = [foo isGreat];
if (great) {         // 正确。
  // ...be great!
}
```

### 没有实例变量的接口 

省略没有声明任何实例变量的接口的空大括号。

```objectivec 
// 正确:

@interface MyClass : NSObject
// 做了很多事情。
- (void)fooBarBam;
@end
```

```objectivec 
// 错误:

@interface MyClass : NSObject {
}
// 做了很多事情。
- (void)fooBarBam;
@end
```

## Cocoa 模式 

### 委托模式 

委托、目标对象和块指针在会导致循环引用时不应该被保留。

为了避免产生循环引用，委托或目标指针应在确定不再需要向对象发送消息时立即释放。

如果没有明确的时间点指示委托或目标指针不再需要，那么指针应该仅以弱引用的方式被保留。

块指针不能被弱引用保留。为了避免在客户端代码中产生循环引用，块指针应仅在被调用后或不再需要时明确释放。否则，应通过弱委托或目标指针进行回调。

## Objective-C++ 

### 风格与语言匹配 

在 Objective-C++ 源文件中，遵循正在实现的函数或方法的语言的风格。为了在混合使用 Cocoa/Objective-C 和 C++ 时最小化不同命名风格之间的冲突，遵循正在实现的方法的风格。

对于 `@implementation` 块中的代码，请使用 Objective-C 的命名规则。对于 C++ 类的方法中的代码，请使用 C++ 的命名规则。

对于 Objective-C++ 文件中类实现之外的代码，请在文件内保持一致。

```objectivec++ 
// 正确:

// 文件: cross_platform_header.h

class CrossPlatformAPI {
 public:
  ...
  int DoSomethingPlatformSpecific();  // 在每个平台上实现
 private:
  int an_instance_var_;
};

// 文件: mac_implementation.mm
#include "cross_platform_header.h"

/** 典型的 Objective-C 类，使用 Objective-C 的命名。 */
@interface MyDelegate : NSObject {
 @private
  int _instanceVar;
  CrossPlatformAPI* _backEndObject;
}

- (void)respondToSomething:(id)something;

@end

@implementation MyDelegate

- (void)respondToSomething:(id)something {
  // 通过我们的 C++ 后端桥接 Cocoa
  _instanceVar = _backEndObject->DoSomethingPlatformSpecific();
  NSString* tempString = [NSString stringWithFormat:@"%d", _instanceVar];
  NSLog(@"%@", tempString);
}

@end

/** C++ 类的平台特定实现，使用 C++ 的命名。 */
int CrossPlatformAPI::DoSomethingPlatformSpecific() {
  NSString* temp_string = [NSString stringWithFormat:@"%d", an_instance_var_];
  NSLog(@"%@", temp_string);
  return [temp_string intValue];
}
```

项目可以选择使用 80 列的行长度限制，以保持与 Google 的 C++ 风格指南的一致性。

## 空格和格式 

### 空格与制表符 

只使用空格，每次缩进 2 个空格。我们使用空格进行缩进。在你的代码中不要使用制表符。

你应该设置你的编辑器在按下制表键时输出空格，并在行尾删除多余的空格。

### 行长度 

Objective-C 文件的最大行长度为 100 列。

你可以通过在 Xcode 中启用 *首选项 > 文本编辑 > 在列处显示页面指南: 100* 来更容易地发现违规。

### 方法声明和定义 

`-` 或 `+` 与返回类型之间应使用一个空格，并且在参数列表中不使用空格，除了参数之间。

方法应该像这样：

```objectivec 
// 正确:

- (void)doSomethingWithString:(NSString *)theString {
  ...
}
```

在星号之前的空格是可选的。当添加新代码时，请与周围文件的风格保持一致。

如果一个方法声明不能在单行上完全显示，则将每个参数放在单独的行上。除第一行外的所有行都应至少缩进四个空格。在参数之前的冒号应在所有行上对齐。如果方法声明中的第一行上的参数之前的冒号位置使得在后续行上的缩进小于四个空格，则只需要在除第一行外的所有行上进行冒号对齐。

```objectivec 
// 正确:

- (void)doSomethingWithFoo:(GTMFoo *)theFoo
                      rect:(NSRect)theRect
                  interval:(float)theInterval {
  ...
}

- (void)shortKeyword:(GTMFoo *)theFoo
            longerKeyword:(NSRect)theRect
    someEvenLongerKeyword:(float)theInterval
                    error:(NSError **)theError {
  ...
}

- (id<UIAdaptivePresentationControllerDelegate>)
    adaptivePresentationControllerDelegateForViewController:(UIViewController *)viewController;

- (void)presentWithAdaptivePresentationControllerDelegate:
    (id<UIAdaptivePresentationControllerDelegate>)delegate;
```

### 函数声明和定义

如果可能的话，倾向于将返回类型放在与函数名相同的行上，并且如果它们能够容纳，则将所有参数放在同一行上。如果参数列表不能放在单行上，请按照你在[函数调用](#Function_Calls)中换行参数的方式进行换行。

```objectivec 
// 正确:

NSString *GTMVersionString(int majorVersion, minorVersion) {
  ...
}

void GTMSerializeDictionaryToFileOnDispatchQueue(
    NSDictionary<NSString *, NSString *> *dictionary,
    NSString *filename,
    dispatch_queue_t queue) {
  ...
}
```

函数的声明和定义还应满足以下条件：

* 开括号必须始终与函数名在同一行。
* 如果无法将返回类型和函数名放在单行上，请在它们之间换行，并且不要缩进函数名。
* 在开括号之前绝不应该有空格。
* 在函数括号和参数之间绝不应该有空格。
* 开花括号始终在函数声明的最后一行末尾，而不是下一行的开始。
* 闭花括号要么单独放在最后一行，要么与开花括号放在同一行。
* 在闭括号和开花括号之间应该有一个空格。
* 所有参数应尽可能对齐。
* 函数范围应缩进 2 个空格。
* 换行的参数应该有 4 个空格的缩进。

### 条件语句

在 `if`、`while`、`for` 和 `switch` 后面加上一个空格，并在比较运算符周围加上空格。

```objectivec 
// 正确:

for (int i = 0; i < 5; ++i) {
}

while (test) {};
```

如果循环体或条件语句适合放在一行上，则可以省略大括号。

```objectivec 
// 正确:

if (hasSillyName) LaughOutLoud();

for (int i = 0; i < 10; i++) {
  BlowTheHorn();
}
```

```objectivec 
// 错误:

if (hasSillyName)
  LaughOutLoud();               // 错误.

for (int i = 0; i < 10; i++)
  BlowTheHorn();                // 错误.
```

如果 `if` 语句有 `else` 语句，则两个语句块都应该使用大括号。

```objectivec 
// 正确:

if (hasBaz) {
  foo();
} else {  // else 和右括号在同一行上。
  bar();
}
```

```objectivec 
// 错误:

if (hasBaz) foo();
else bar();        // 错误.

if (hasBaz) {
  foo();
} else bar();      // 错误.
```

有意的穿透到下一个 case 应该通过注释进行说明，除非在下一个 case 前没有其他代码。

```objectivec 
// 正确:

switch (i) {
  case 1:
    ...
    break;
  case 2:
    j++;
    // Falls through.
  case 3: {
    int k;
    ...
    break;
  }
  case 4:
  case 5:
  case 6: break;
}
```

### 表达式

在二元操作符和赋值操作符周围使用空格。对于一元操作符，不需要添加空格。不要在括号内部添加空格。

```objectivec 
// 正确:

x = 0;
v = w * x + y / z;
v = -y * (x + z);
```

表达式中的因子可以省略空格。

```objectivec 
// 正确:

v = w*x + y/z;
```

### 方法调用

方法调用应该与方法声明的格式相似。

在选择格式样式时，应该遵循源文件中已使用的惯例。方法调用应该将所有参数放在一行上：

```objectivec 
// 正确:

[myObject doFooWith:arg1 name:arg2 error:arg3];
```

或者每个参数一行，冒号对齐：

```objectivec 
// 正确:

[myObject doFooWith:arg1
               name:arg2
              error:arg3];
```

不要使用以下任何样式：

```objectivec 
// 错误:

[myObject doFooWith:arg1 name:arg2  // 一行上有多个参数
              error:arg3];

[myObject doFooWith:arg1
               name:arg2 error:arg3];

[myObject doFooWith:arg1
          name:arg2  // 对齐关键字而不是冒号
          error:arg3];
```

与声明和定义一样，如果第一个关键字比其他关键字短，请至少缩进后面的行，保持冒号对齐：

```objectivec 
// 正确:

[myObj short:arg1
          longKeyword:arg2
    evenLongerKeyword:arg3
                error:arg4];
```

包含多个内联块的调用可以在四个空格缩进的情况下将参数名称左对齐。

### 函数调用

函数调用应该包括尽可能多的参数，每行一个参数，除非需要为参数的清晰性或文档化而使用较短的行。

函数参数的续行可以缩进以与开括号对齐，也可以缩进四个空格。

```objectivec 
// 正确:

CFArrayRef array = CFArrayCreate(kCFAllocatorDefault, objects, numberOfObjects,
                                 &kCFTypeArrayCallBacks);

NSString *string = NSLocalizedStringWithDefaultValue(@"FEET", @"DistanceTable",
    resourceBundle,  @"%@ feet", @"Distance for multiple feet");

UpdateTally(scores[x] * y + bases[x],  // 评分启发式。
            x, y, z);

TransformImage(image,
               x1, x2, x3,
               y1, y2, y3,
               z1, z2, z3);
```

使用具有描述性名称的局部变量缩短函数调用，并减少调用的嵌套。

```objectivec 
// 正确:

double scoreHeuristic = scores[x] * y + bases[x];
UpdateTally(scoreHeuristic, x, y, z);
```

### 异常

使用 `@catch` 和 `@finally` 标签格式化异常，它们应该与前面的 `}` 放在同一行上。在 `@` 标签和开括号 (`{`) 之间添加一个空格，以及在 `@catch` 和被捕获对象声明之间也要添加一个空格。如果必须使用 Objective-C 异常，请按以下格式进行格式化。然而，参见 [Avoid Throwing Exceptions](#Avoid_Throwing_Exceptions) 了解为什么你不应该使用异常。

```objectivec 
// 正确:

@try {
  foo();
} @catch (NSException *ex) {
  bar(ex);
} @finally {
  baz();
}
```

### 函数长度

更倾向于编写小而专注的函数。

长函数和方法偶尔是适当的，因此对函数长度没有硬性限制。如果函数超过约 40 行，请考虑它是否可以被拆分而不影响程序的结构。

即使你的长函数现在完美运行，几个月后修改它的人可能会添加新的行为。这可能导致难以找到的错误。保持函数简短和简单，可以让其他人更容易阅读和修改你的代码。

当更新旧代码时，还考虑将长函数分解为更小且更易管理的片段。

### 垂直空白

谨慎使用垂直空白。

为了让更多的代码能够在屏幕上轻松查看，请避免将空行放在函数的括号内。

在函数之间和代码的逻辑组之间，将空行限制为一到两行。

## Objective-C 风格异常

### 指示风格异常

不符合这些风格建议的代码行在行尾需要添加 `// NOLINT`，或者在前一行末尾添加 `// NOLINTNEXTLINE`。有时需要忽略部分 Objective-C 代码必须忽略这些风格建议（例如，代码可能是机器生成的，或者代码结构使得无法正确地进行格式化）。

可以使用该行的 `// NOLINT` 注释或前一行的 `// NOLINTNEXTLINE` 来指示读者代码是有意忽略风格指南的。此外，这些注释还可以被诸如代码检查器等自动化工具捕获并处理代码。请注意 `//` 和 `NOLINT*` 之间有一个空格。
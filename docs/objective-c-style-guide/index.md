# Google Objective-C 风格指南

> Objective-C 是 C 语言的动态、面向对象的扩展。它旨在易于使用和阅读，同时支持复杂的面向对象设计。它是 OS X 和 iOS 应用程序的主要开发语言。
> 
> 苹果已经撰写了一份非常好的、广为接受的[Cocoa 编码指南](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)，用于 Objective-C。请在阅读本指南的同时参考该指南。
> 
> 本文档的目的是描述应用于 iOS 和 OS X 代码的 Objective-C（以及 Objective-C++）编码指南和实践。这些指南在其他项目和团队中经过了时间的演变和验证。Google 开发的开源项目符合本指南的要求。
> 
> 请注意，本指南不是 Objective-C 教程。我们假设读者已经熟悉了这种语言。如果您对 Objective-C 不熟悉或需要复习，请阅读[《使用 Objective-C 编程》](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html)。

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

注释对于保持我们的代码可读性至关重要。以下规则描述了您应该在何处以及如何进行注释。但请记住：尽管注释很重要，但最好的代码是自解释的。为类型和变量赋予合理的名称比使用晦涩的名称然后尝试通过注释来解释它们要好得多。

注意标点符号、拼写和语法；读起来容易的注释比读起来困难的注释更容易。

注释应该与叙述性文本一样可读，具有适当的大写和标点符号。在许多情况下，完整的句子比句子片段更易读。较短的注释，例如在代码行末尾的注释，有时可能不太正式，但要使用一致的风格。
在编写注释时，请为您的受众编写：下一个需要理解您的代码的贡献者。要慷慨一点——下一个可能就是你！

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

如果您已经在文件顶部的注释中详细描述了一个接口，请随意简单地说明：“有关完整描述，请参见文件顶部的注释”，但一定要有某种形式的注释。

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

行尾注释应该与代码之间至少有 2 个空格的分隔。如果您有几个连续的注释行，通常更易读地将它们排成一行。

```objectivec 
// 正确:

[self doSomethingWithALongName];  // 注释之前有两个空格。
[self doSomethingShort];          // 更多的间距以对齐注释。
```

### 消歧符号

为了避免歧义，注释中引用变量名和符号时应优先使用反引号或竖线来引用，而不是内联命名符号的引号。

在 Doxygen 风格的注释中，最好使用等宽文本命令来标识符号，例如 `@c`。


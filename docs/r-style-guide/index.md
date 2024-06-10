# Google R 风格指南

R 是一种高级编程语言，主要用于统计计算和绘图。R编程风格指南的目标是使我们的 R 代码更易于阅读、共享和验证。

Google R 风格指南是 Hadley Wickham [许可](https://creativecommons.org/licenses/by-sa/2.0/)下的 [Tidyverse 风格指南](https://style.tidyverse.org/)的一个分支。Google 的修改是与内部 R 用户社区合作开发的。本文档的其余部分解释了 Google 与 Tidyverse 指南之间的主要区别，以及这些区别存在的原因。

## 语法

### 命名约定

Google 更喜欢使用 `BigCamelCase` 来标识函数，以清晰区分它们和其他对象。

```r
# 好的
DoNothing <- function() {
return(invisible(NULL))
}
```

私有函数的名称应以点号开头。这有助于传达函数的来源和预期用途。

```r
# 好的
.DoNothingPrivately <- function() {
return(invisible(NULL))
}
```

我们以前建议使用 `dot.case` 来命名对象。我们正在摒弃这一做法，因为它会与 S3 方法造成混淆。

### 不要使用attach()

使用 `attach()` 时出错的可能性很多。

## 管道操作符

### 右手赋值

我们不支持使用右手赋值。

```r
# 不好的
iris %>%
dplyr::summarize(max_petal = max(Petal.Width)) -> results
```

这一惯例与其他语言的做法大不相同，并且使得在代码中很难看出对象的定义位置。例如，搜索 `foo <-` 比搜索 `foo <-` 和 `-> foo`（可能分行显示）更容易。

### 使用显式返回语句

不要依赖于 R 的隐式返回特性。最好明确表达你意图 `return()` 一个对象。

```r
# 好的
AddValues <- function(x, y) {
return(x + y)
}

# 不好的
AddValues <- function(x, y) {
x + y
}
```

### 限定命名空间

用户应该为所有外部函数显式限定命名空间。

```r
# 好的
purrr::map()
```

我们不建议使用 `@import` 这个 Roxygen 标签将所有函数引入到 NAMESPACE 中。Google 有一个非常庞大的 R 代码库，将所有函数导入会产生太多的命名冲突风险。

虽然使用 `::` 会带来一些小的性能损失，但它使代码中的依赖关系更容易理解。有一些例外情况。

- 中缀函数（`%name%`）始终需要导入。
- 某些 `rlang` 代词，特别是 `.data`，需要导入。
- 默认的R包函数，包括 `datasets`、`utils`、`grDevices`、`graphics`、`stats` 和 `methods`。如果需要，你可以 `@import` 整个包。

当导入函数时，请将 `@importFrom` 标签放在 Roxygen 头部，在使用外部依赖项的函数上方。

## 文档

### 包级别文档

所有包都应该有一个包文档文件，在一个 `packagename-package.R` 文件中。
# Google Vimscript 风格简要指南

## 背景

这是一个关于 Vim 脚本编写风格的简要指南，因为 Vim 脚本是一种随意的语言。在提交 Vim 插件代码时，你必须遵循这些规则。如果需要更多解释和细节，请参见详细指南。

## 可移植性

编写 Vim 脚本很难。许多命令取决于用户的设置。通过遵循这些准则，希望可以使你的脚本具有可移植性。

### 字符串

> 优先使用单引号字符串。

在 Vim 脚本中，双引号字符串在语义上有所不同，你可能不希望使用它们（因为它们会破坏正则表达式）。

当你需要转义序列（例如 `"\n"`）或者需要嵌入单引号并且知道不会有影响时，可以使用双引号字符串。

### 字符串匹配

> 使用 `=~#` 或 `=~?` 运算符族而不是 `=~` 运算符族。

匹配行为取决于用户的 ignorecase 和 smartcase 设置，以及你是否使用 `=~`、`=~#` 或 `=~?` 运算符族进行比较。除非你要遵循用户的大小写敏感设置，否则请使用 `=~#`  和 `=~?` 运算符族进行字符串比较。

### 正则表达式

> 所有正则表达式前缀加上 `\m\C`。

除了大小写敏感设置外，正则表达式的行为还取决于用户的 nomagic 设置。为了使正则表达式的行为像 nomagic 和 noignorecase 已设置一样，请在所有正则表达式前加上 `\m\C`。

欢迎使用魔术级别（例如 `\v`）和大小写敏感性（`\c`），只要它们是目的明确的。

### 危险命令

> 避免使用具有意外副作用的命令。

避免使用 `:s[ubstitute]`，因为它会移动光标并打印错误消息。优先使用适合脚本的函数（如 `search()`）。

对于许多Vim命令，都存在可以以较少副作用执行相同操作的函数。请参阅 `:help functions()` 获取内置函数列表。

### 脆弱命令

> 避免依赖用户设置的命令。

始终使用 `normal!` 而不是 `normal`。后者依赖用户的键映射，可能会执行任意操作。

避免使用 `:s[ubstitute]`，因为其行为依赖于许多本地设置。

对于此处未列出的其他命令也适用相同的原则。

### 捕获异常

> 匹配错误代码，而不是错误文本。

错误文本可能依赖于地区设置。

## 一般准则

### 消息

> 避免经常向用户发送消息。

吵闹的脚本很烦人。只有在以下情况下向用户发送消息：

- 启动了长时间运行的进程。
- 发生了错误。

### 类型检查

> 尽可能使用严格和明确的检查。

处理某些类型时，Vim 脚本具有不安全、不直观的行为。例如，`0 == 'foo'` 会评估为 `true`。

尽可能使用严格的比较运算符。当与字符串字面量进行比较时，请使用 `is#` 运算符。否则，优先使用 `maktaba#value#IsEqual` 或者严格地检查 `type()`。

在使用之前明确检查变量类型。使用 `maktaba#ensure` 中的函数，或者检查 `maktaba#value` 或 `type()` 并抛出自己的错误。

对于可能更改类型的变量，特别是在循环内赋值的变量，请使用 `:unlet`。

### Python

> 谨慎使用。

只有在提供关键功能时才使用 Python，例如编写多线程代码。

### 其他语言

> 优先使用 Vim 脚本。

避免使用其他脚本语言，如 ruby 和 lua。我们无法保证最终用户的 Vim 是否已编译支持非 Vim 脚本语言。

### 样板代码

> 使用 [`maktaba`](https://github.com/google/maktaba)。

`maktaba` 可以去除样板文件，包括：

- 插件创建
- 错误处理
- 依赖检查

### 插件布局

> 将功能组织成模块化插件。

将功能作为一个插件组织起来，统一放在一个目录（或代码存储库）中，该目录使用插件名称命名（如果需要，可以使用“vim-”前缀或“.vim”后缀）。它应该根据需要分割成 `plugin/`、`autoload/` 等子目录，并且应该以 `addon-info.json` 格式声明元数据（有关详情，请参阅 [VAM 文档](https://github.com/MarcWeber/vim-addon-manager/blob/master/doc/vim-addon-manager-additional-documentation.txt)）。

### 函数

> 在 autoload/ 目录中定义，并使用 `[!]` 和 `[abort]`。

自动加载允许函数按需加载，这样可以加快启动时间并强制进行函数命名空间。

欢迎使用脚本局部函数，但也应放在 autoload/ 中，并由自动加载的函数调用。

非库插件应该暴露命令而不是函数。命令逻辑应该提取到函数中并进行自动加载。

`[!]` 允许开发者重新加载他们的函数而不受干扰。

`[abort]` 在遇到错误时强制函数停止。

### 命令

> 在 plugin/commands.vim 或 ftplugin/ 目录下定义命令，不使用 `[!]`。

通用命令放在 `plugin/commands.vim` 中，特定文件类型的命令放在 `ftplugin/` 中。

排除 `[!]` 可以防止你的插件悄悄地覆盖现有命令。命令冲突应由用户解决。

### 自动命令

> 将所有自动命令放在自动命令组（augroup） 中，并放在 plugin/autocmds.vim 中。

自动命令组（augroup） 名称应该是唯一的。它应该是插件名称或以插件名称为前缀。

在自动命令组（augroup） 中定义新的自动命令之前，使用 `autocmd!` 清除自动命令组（augroup）。这样可以使你的插件可重新进入。

### 映射

> 在 `plugin/mappings.vim` 中定义所有键映射，使用 `maktaba#plugin#MapPrefix` 来获取前缀。

部分映射（请参阅 `:help` 中关于 `<Plug>` 的内容）应该在 `plugin/plugs.vim` 中定义。

### 设置

> 本地更改设置。

使用 `:setlocal` 或 `&l:`。或者有足够的理由时使用 `:set` 和 `s`。

## 样式

遵循 Google 的风格规范。如果有疑问，将 Vim 脚本风格视为 Python 风格。

### 空白

> 类似于 Python。

- 缩进使用两个空格
- 不使用制表符
- 在运算符周围使用空格

这不适用于命令的参数。

```vim
let s:variable = "concatenated " . "strings"
command -range=% MyCommand
```

- 不要引入尾随空白

不需要刻意去除它。

在为用户输入准备命令的映射中允许尾随空白，例如 `"noremap <leader>gf
-f"`。

- 将行限制为 80 列宽
- 将续行缩进四个空格
- 不要对齐命令的参数

应避免这种：
```vim
command -bang MyCommand  call myplugin#foo()
command       MyCommand2 call myplugin#bar()
```

而应采用这种：

```vim
command -bang MyCommand call myplugin#foo()
command MyCommand2 call myplugin#bar()
```

### 命名

> 通常使用 `plugin-names-like-this`、`FunctionNamesLikeThis`、`CommandNamesLikeThis`、`augroup_names_like_this`、`variable_names_like_this`。始终为变量添加其范围前缀。

#### plugin-names-like-this

保持简洁。

#### FunctionNamesLikeThis

将脚本局部函数以 `s:` 前缀开头。

自动加载的函数可能没有范围前缀。

不要创建全局函数。使用自动加载的函数。

#### CommandNamesLikeThis

与常见命令前缀相比，更喜欢简洁的命令名称。

#### variable_names_like_this

自动命令组（augroup） 名称用于命名目的。

#### 为所有变量添加其范围前缀

- 全局变量用 `g:` 前缀

- 脚本局部变量用 `s:` 前缀

- 函数参数用 `a:` 前缀

- 函数局部变量用 `l:` 前缀

- Vim 预定义变量用 `v:` 前缀

- 缓冲区局部变量用 `b:` 前缀

必须始终使用 `g:`、`s:` 和 `a:`。

`b:` 更改变量语义；当你需要缓冲区局部语义时使用它。

`l:` 和 `v:` 应该用于一致性、未来的证明和避免微妙的错误。它们不是严格要求的。在新代码中添加它们，但不要刻意为其他地方添加它们。
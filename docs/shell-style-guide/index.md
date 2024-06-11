# Google Shell Style Guide

## 背景

### 选择使用哪种 Shell

Bash 是唯一允许用于可执行文件的 shell 脚本语言。

可执行文件必须以 `#!/bin/bash` 开头，并包含最少数量的标志。使用 `set` 命令设置 shell 选项，以便在将脚本作为 `bash script_name` 调用时不会破坏其功能。

将所有可执行的 shell 脚本限制为 *bash* 可以确保我们使用的是在所有计算机上都安装的一致的 shell 语言。

唯一的例外是在你被迫使用某种 shell 的情况下。一个例子是 Solaris SVR4 软件包，它要求所有脚本都必须使用纯 Bourne shell。

### 何时使用 Shell

Shell 只应用于小型实用程序或简单的包装脚本。

尽管 Shell 脚本不是一种开发语言，但它被用于在 Google 中编写各种实用程序脚本。本风格指南更多地是对其使用的认可，而不是建议广泛部署使用。

一些准则：

- 如果你主要调用其他实用程序，并且进行的数据操作相对较少，则 Shell 对于任务是可以接受的选择。
- 如果性能很重要，请使用其他语言而不是 Shell。
- 如果你正在编写的脚本超过 100 行，或者使用了非直接的控制流逻辑，你应该立即将其重写为更结构化的语言。要记住脚本会不断增长。及早重写你的脚本，以避免以后更耗时的重写。
- 在评估代码复杂性时（例如决定是否切换语言），考虑代码是否易于除作者外的其他人维护。

## Shell 文件和解释器调用

### 文件扩展名

可执行文件应该没有扩展名（强烈推荐）或者有一个 `.sh` 扩展名。库文件必须有一个 `.sh` 扩展名，并且不应该是可执行的。

当执行一个程序时，不需要知道它是用什么语言编写的，而且 shell 不需要扩展名，因此我们更喜欢不为可执行文件使用扩展名。

但是，对于库文件来说，知道它是什么语言很重要，有时需要在不同的语言中有相似的库。这允许具有相同目的但不同语言的库文件具有相同的名称，除了语言特定的后缀。

### SUID/SGID

在 shell 脚本上 *禁止* 使用 SUID 和 SGID。

由于 Shell 存在太多的安全问题，使其几乎不可能保护得足够好以允许 SUID/SGID。尽管 bash 确实使 SUID 运行变得困难，但在某些平台上仍然可能存在这种情况，这就是为什么我们明确禁止它的原因。

如果需要提供提升的访问权限，请使用 `sudo`。

## 环境

### STDOUT vs STDERR

所有错误消息都应该输出到 `STDERR`。

这样可以更容易地区分正常状态和实际问题。

建议使用一个打印错误消息以及其他状态信息的函数。

```shell
err() {
  echo "[$(date +'%Y-%m-%dT%H:%M:%S%z')]: $*" >&2
}

if ! do_something; then
  err "无法执行某些操作"
  exit 1
fi
```

## 注释

### 文件头

每个文件都应该以其内容的描述开头。

每个文件必须有一个顶层注释，包括其内容的简要概述。 版权声明和作者信息是可选的。

示例：

```shell
#!/bin/bash
#
# 对 Oracle 数据库执行热备份。
```

### 函数注释

任何既不明显又不短小的函数都必须进行注释。 不管长度或复杂性如何，库中的任何函数都必须进行注释。

其他人应该能够通过阅读注释（和自助信息，如果提供的话）而无需阅读代码来学习如何使用你的程序或库中的函数。

所有函数注释都应描述预期的 API 行为，包括：

- 函数的描述。
- 全局变量：使用和修改的全局变量列表。
- 参数：接受的参数。
- 输出：输出到 STDOUT 或 STDERR。
- 返回值：除了最后一个运行命令的默认退出状态外的返回值。

示例：

```shell
#######################################
# 从备份目录中清理文件。
# 全局变量:
#   BACKUP_DIR
#   ORACLE_SID
# 参数:
#   无
#######################################
function cleanup() {
  …
}

#######################################
# 获取配置目录。
# 全局变量:
#   SOMEDIR
# 参数:
#   无
# 输出:
#   将位置写入 stdout
#######################################
function get_dir() {
  echo "${SOMEDIR}"
}

#######################################
# 以复杂的方式删除文件。
# 参数:
#   要删除的文件，即路径。
# 返回:
#   如果删除了内容，则返回0，否则返回非零值。
#######################################
function del_thing() {
  rm "$1"
}
```

### 实现注释

对代码中棘手、不明显、有趣或重要的部分进行注释。

这符合通用的 Google 编码注释实践。不要注释一切。如果存在复杂的算法或者你在做一些不寻常的事情，请添加简短的注释。

### TODO 注释

对于临时的、短期的解决方案或者尚不完美的代码，请使用 TODO 注释。

这与 [C++ 风格指南](https://google.github.io/styleguide/cppguide.html#TODO_Comments)中的约定相匹配。

`TODO` 应该包含全部大写的字符串 `TODO`，后跟最了解有关问题的人员的姓名、电子邮件地址或其他标识符。主要目的是拥有一个一致的 `TODO`，可以搜索以了解如何在请求时获得更多详细信息。 `TODO` 不是一个承诺，即被引用的人会修复问题。因此，当你创建一个 `TODO` 时，几乎总是给出你的姓名。

示例：

```shell
# TODO(mrmonkey): 处理不太可能的边缘情况（bug ####）
```

## 格式化

在修改文件时，应该遵循已有代码的样式，但以下内容适用于任何新代码。

### 缩进

缩进为 2 个空格，不使用制表符。

使用空行将各个块分隔开来以提高可读性。缩进为两个空格。无论如何，都不要使用制表符。对于现有文件，应保持现有的缩进不变。

### 行长度和长字符串

每行的最大长度为 80 个字符。

如果必须编写超过 80 个字符的字符串，则应使用文档中的字符串或嵌入换行符（如果可能）。如果需要的文本字符串超过80个字符且不能合理地分割，则是可以接受的，但强烈建议找到缩短字符串的方法。

```shell
# 使用文档中的字符串
cat <<END
我是一个异常长的
字符串。
END

# 嵌入换行符也是可以的
long_string="我是一个异常长的
字符串。"
```

### 管道命令

如果管道命令不适合一行，则应将其拆分为每行一个管道命令。

如果管道命令适合一行，则应在一行上写完。

如果不适合，则应在每个管道命令的换行处拆分，每行一个管道命令，并在新行上以两个空格的缩进开头。这适用于使用 `|` 结合的命令链，以及使用 `||` 和 `&&` 的逻辑组合。

```shell
# 适合一行
command1 | command2

# 较长的命令
command1 \
  | command2 \
  | command3 \
  | command4
```

### 循环

将 `; do` 和 `; then` 放在 `while`、`for` 或 `if` 的同一行。

Shell 中的循环有点不同，但我们遵循与声明函数时相同的原则。也就是说：`then` 和 `do` 应该与 if/for/while 在同一行上。`else` 应该单独一行，而关闭语句应该在自己的一行上，与开放语句在垂直方向上对齐。

示例：

```shell
# 如果在函数内部，考虑将循环变量声明为局部变量，以避免它泄漏到全局环境中：
# local dir
for dir in "${dirs_to_cleanup[@]}"; do
  if [[ -d "${dir}/${ORACLE_SID}" ]]; then
    log_date "Cleaning up old files in ${dir}/${ORACLE_SID}"
    rm "${dir}/${ORACLE_SID}/"*
    if (( $? != 0 )); then
      error_message
    fi
  else
    mkdir -p "${dir}/${ORACLE_SID}"
    if (( $? != 0 )); then
      error_message
    fi
  fi
done
```

### Case 语句

- 使用两个空格缩进备选项。
- 单行备选项在模式的右括号后和 `;;` 前需要有一个空格。
- 长或多命令备选项应在多行上拆分，模式、操作和 `;;` 分别位于单独的行上。

匹配表达式与 `case` 和 `esac` 同级缩进。多行操作再缩进一级。一般情况下，匹配表达式不需要用引号括起来。模式表达式不应该在开括号之前。避免使用 `;&` 和 `;;&` 表示法。

```shell
case "${expression}" in
  a)
    variable="…"
    some_command "${variable}" "${other_expr}" …
    ;;
  absolute)
    actions="relative"
    another_command "${actions}" "${other_expr}" …
    ;;
  *)
    error "Unexpected expression '${expression}'"
    ;;
esac
```

如果简单命令可以放在模式和 `;;` 的同一行上，并且表达式仍然可读，则可以这样做。这通常适用于单字母选项处理。当操作不适合一行时，将模式放在单独的一行上，然后是操作，然后是 `;;` 也在自己的一行上。在与操作相同的行上时，在模式的右括号后使用一个空格，`;;` 前再加一个空格。

```shell
verbose='false'
aflag=''
bflag=''
files=''
while getopts 'abf:v' flag; do
  case "${flag}" in
    a) aflag='true' ;;
    b) bflag='true' ;;
    f) files="${OPTARG}" ;;
    v) verbose='true' ;;
    *) error "Unexpected option ${flag}" ;;
  esac
done
```

### 变量扩展

按优先顺序排序：保持与现有代码一致；引用你的变量；使用 `"${var}"` 而不是 `"$var"`。

这些是强烈推荐的指南，但不是强制性的规定。尽管如此，这是一项建议而不是强制性的规定，并不意味着应该轻率对待或淡化它的重要性。

它们按优先级排序。

- 对于现有代码，保持一致性。
- 引用变量，请参见下面的 [引用部分](#quoting)。
- 除非绝对必要或避免深度混淆，否则不要使用大括号限定单个字符的shell特殊字符/位置参数。

  首选使用大括号限定所有其他变量。

  ```shell
  # 推荐使用示例

  # '特殊'变量的首选样式：
  echo "Positional: $1" "$5" "$3"
  echo "Specials: !=$!, -=$-, _=$_. ?=$?, #=$# *=$* @=$@ \$=$$ …"

  # 必须使用大括号：
  echo "many parameters: ${10}"

  # 避免混淆：
  # 输出是 "a0b0c0"
  set -- a b c
  echo "${1}0${2}0${3}0"

  # 其他变量的首选样式：
  echo "PATH=${PATH}, PWD=${PWD}, mine=${some_var}"
  while read -r f; do
    echo "file=${f}"
  done < <(find /tmp)
  ```

  ```shell
  # 不推荐使用示例

  # 未引用变量，未大括号限定变量，大括号限定单个字母的shell特殊字符。
  echo a=$avar "b=$bvar" "PID=${$}" "${1}"

  # 混淆使用：这将扩展为 "${1}0${2}0${3}0"，而不是 "${10}${20}${30}"
  set -- a b c
  echo "$10$20$30"
  ```

注意：在 `${var}` 中使用大括号不是引用的一种形式。仍然必须使用“双引号”。

### 引用

- 包含变量、命令替换、空格或 shell 元字符的字符串必须始终引用，除非需要小心地未引用扩展或它是 shell 内部整数（见下一点）。
- 对于列表元素，特别是命令行标志，使用数组进行安全引用。参见下面的[数组](#arrays) 部分。
- 可选地引用 shell 内部的、只读的特殊整数变量：`$?`、`$#`、`$$`、`$!`（参见 `man bash`）。为了保持一致性，更倾向于引用“命名”内部整数变量，例如 PPID 等。
- 更倾向于引用“单词”字符串（而不是命令选项或路径名）。
- 永远不要引用*字面整数*。
- 了解 `[[ … ]]` 中的模式匹配引用规则。请参见下面的 [测试，`[ … ]` 和 `[[ … ]]`](#tests) 部分。
- 使用 `"$@"`，除非你有特定的原因要使用 `$*`，例如在消息或日志中简单地将参数附加到字符串中。

```shell
# '单引号' 表示不希望进行替换。
# "双引号" 表示需要/可以进行替换。

# 简单示例

# "引用命令替换"
# 请注意，"$()" 内部嵌套的引号不需要转义。
flag="$(some_command and its args "$@" 'quoted separately')"

# "引用变量"
echo "${flag}"

# 对列表使用带引号的扩展的数组。
declare -a FLAGS
FLAGS=( --foo --bar='baz' )
readonly FLAGS
mybinary "${FLAGS[@]}"

# 不引用内部整数变量也是可以的。
if (( $# > 3 )); then
  echo "ppid=${PPID}"
fi

# "永远不要引用字面整数"
value=32
# "引用命令替换"，即使你期望是整数
number="$(generate_number)"

# "更倾向于引用单词"，不是强制性的
readonly USE_INTEGER='true'

# "引用shell元字符"
echo 'Hello stranger, and well met. Earn lots of $$$'
echo "Process $$: Done making \$\$\$."

# "命令选项或路径名"
# （此处假定 $1 包含一个值）
grep -li Hugo /dev/null "$1"

# 较复杂的示例
# "引用变量，除非证明为假"：ccs 可能为空
git send-email --to "${reviewers}" ${ccs:+"--cc" "${ccs}"}

# 位置参数预防措施：$1 可能未设置
# 单引号保留正则表达式原样。
grep -cP '([Ss]pecial|\|?characters*)$' ${1:+"$1"}

# 用于传递参数，
# "$@" 几乎每次都是正确的选择，而
# $* 几乎每次都是错误的选择：
#
# * $* 和 $@ 会根据空格拆分参数，
#   删除包含空格的参数并丢弃空字符串；
# * "$@" 将保留参数不变，因此不提供参数将导致不传递参数；
#   这在大多数情况下是传递参数的所需方式。
# * "$*" 扩展为一个参数，所有参数由（通常是）空格连接，
#   因此不提供参数将导致传递一个空字符串。
# （请参阅 `man bash` 以获取细节）

(set -- 1 "2 two" "3 three tres"; echo $#; set -- "$*"; echo "$#, $@")
(set -- 1 "2 two" "3 three tres"; echo $#; set -- "$@"; echo "$#, $@")
```

## 特性和缺陷

### ShellCheck

[ShellCheck 项目](https://www.shellcheck.net/) 会为你的 Shell 脚本识别常见的错误和警告。对于所有规模的脚本，无论大小，都建议使用它。

### 命令替换

使用 `$(command)` 而不是反引号。

嵌套的反引号需要使用 `\ ` 转义内部的反引号。`$(command)` 格式在嵌套时不变，并且更容易阅读。

示例：

```shell
# 这是首选方式：
var="$(command "$(command1)")"
```

```shell
# 这是不推荐的方式：
var="`command \`command1\``"
```

### 测试，`[ … ]` 和 `[[ … ]]` 

`[[ … ]]` 优先于 `[ … ]`、`test` 和 `/usr/bin/[`。

`[[ … ]]` 减少错误，因为在 `[[` 和 `]]` 之间不会进行路径名扩展或单词拆分。此外，`[[ … ]]` 允许正则表达式匹配，而 `[ … ]` 不允许。

```shell
# 这确保了左侧的字符串由字母数字字符类和字符串 name 组成。
# 请注意，这里的 RHS 不应引用。
if [[ "filename" =~ ^[[:alnum:]]+name ]]; then
  echo "Match"
fi

# 这匹配了精确的模式 "f*"（在这种情况下不匹配）
if [[ "filename" == "f*" ]]; then
  echo "Match"
fi
```

```shell
# 这会导致 "too many arguments" 错误，因为 f* 被扩展为当前目录的内容
if [ "filename" == f* ]; then
  echo "Match"
fi
```

有关详细信息，请参阅 http://tiswww.case.edu/php/chet/bash/FAQ 中的 E14。

### 测试字符串

尽可能使用引号而不是填充字符。

Bash 足够聪明，能够处理测试中的空字符串。因此，鉴于代码更易读，尽量使用测试来检查空/非空字符串或空字符串，而不是填充字符。

```shell
# 这样做：
if [[ "${my_var}" == "some_string" ]]; then
  do_something
fi

# -z（字符串长度为零）和 -n（字符串长度不为零）优于测试空字符串
if [[ -z "${my_var}" ]]; then
  do_something
fi

# 这样做是可以的（确保在空侧使用引号），但不是首选：
if [[ "${my_var}" == "" ]]; then
  do_something
fi
```

```shell
# 不要这样做：
if [[ "${my_var}X" == "some_stringX" ]]; then
  do_something
fi
```

为避免对测试内容产生疑惑，明确地使用 `-z` 或 `-n`。

```shell
# 使用这个
if [[ -n "${my_var}" ]]; then
  do_something
fi
```

```shell
# 而不是这个
if [[ "${my_var}" ]]; then
  do_something
fi
```

为了清晰起见，对于相等性，请使用 `==` 而不是 `=`，即使两者都可以。前者鼓励使用 `[[`，而后者可能会与赋值混淆。
然而，在 `[[ … ]]` 中使用 `<` 和 `>` 时要小心，它执行的是词典顺序比较。对于数值比较，请使用 `(( … ))` 或 `-lt` 和 `-gt`。

```shell
# 使用这个
if [[ "${my_var}" == "val" ]]; then
  do_something
fi

if (( my_var > 3 )); then
  do_something
fi

if [[ "${my_var}" -gt 3 ]]; then
  do_something
fi
```

```shell
# 而不是这个
if [[ "${my_var}" = "val" ]]; then
  do_something
fi

# 可能意外进行的词典顺序比较。
if [[ "${my_var}" > 3 ]]; then
  # 对于 4 返回 true，对于 22 返回 false。
  do_something
fi
```

### 文件名通配符扩展

在进行文件名的通配符扩展时，请使用显式路径。

由于文件名可以以 `-` 开头，因此最好使用 `./*` 而不是 `*` 来扩展通配符。

```shell
# 这是目录的内容：
# -f  -r  somedir  somefile

# 错误地强制删除了目录中的几乎所有内容
psa@bilby$ rm -v *
removed directory: `somedir'
removed `somefile'
```

```shell
# 与之相反：
psa@bilby$ rm -v ./*
removed `./-f'
removed `./-r'
rm: cannot remove `./somedir': Is a directory
removed `./somefile'
```

### Eval

应避免使用 `eval`。

当用于向变量赋值时，`eval` 会混淆输入，并且可以设置变量，而不需要检查这些变量是什么。

```shell
# 这个设置了什么？
# 它成功了吗？部分成功还是完全成功了？
eval $(set_my_variables)

# 如果返回的值中有一个带有空格，会发生什么？
variable="$(eval some_function)"
```

### 数组

Bash 数组应该用于存储元素列表，以避免引号的复杂性。这特别适用于参数列表。数组不应该用于更复杂的数据结构（请参阅上面的[何时使用 Shell](#when-to-use-shell)）。

数组存储有序的字符串集合，并且可以安全地展开为单独的元素用于命令或循环。

应避免将多个命令参数放在单个字符串中，因为这不可避免地会导致作者使用 `eval` 或尝试在字符串中嵌套引号，这不会产生可靠或可读的结果，并且会引起不必要的复杂性。

```shell
# 通过括号分配数组，并且可以使用+=( … )进行追加。
declare -a flags
flags=(--foo --bar='baz')
flags+=(--greeting="Hello ${name}")
mybinary "${flags[@]}"
```

```shell
# 不要使用字符串表示序列。
flags='--foo --bar=baz'
flags+=' --greeting="Hello world"'  # 这不会按预期工作。
mybinary ${flags}
```

```shell
# 命令扩展返回单个字符串，而不是数组。避免在数组赋值中使用未引用的扩展，因为如果命令输出包含特殊字符或空格，则不会正确工作。

# 这将列表输出展开为字符串，然后进行特殊关键字扩展，然后进行空格拆分。然后才将其转换为单词列表。ls命令还可能根据用户的活动环境而改变行为！
declare -a files=($(ls /directory))

# get_arguments将所有内容写入STDOUT，但然后会通过上面的扩展过程进行相同的扩展，然后转换为参数列表。
mybinary $(get_arguments)
```

#### 数组优点

- 使用数组允许列出事物而不会引起引号语义混淆。相反，不使用数组会导致尝试在字符串内部嵌套引号。
- 数组使得可以安全地存储任意字符串的序列/列表，包括包含空格的字符串。

#### 数组缺点

使用数组可能会增加脚本的复杂性。

#### 数组决策

应使用数组安全地创建和传递列表。特别是在构建一组命令参数时，应使用数组以避免引号引用问题。使用引号扩展 `${array[@]}` 来访问数组。然而，如果需要更复杂的数据操作，应完全避免使用 shell 脚本；参见[上文](#when-to-use-shell)。

### 管道到 While

在管道到 `while` 时，优先使用进程替换或 `readarray` 内置（bash4+）。管道创建子 shell，因此在管道内修改的任何变量都不会传播到父 shell。

管道到 `while` 的隐式子 shell 可能会引入难以跟踪的微妙错误。

```shell
last_line='NULL'
your_command | while read -r line; do
  if [[ -n "${line}" ]]; then
    last_line="${line}"
  fi
done

# 这将始终输出'NULL'！
echo "${last_line}"
```

使用进程替换也会创建子 shell。但是，它允许将子shell中的输出重定向到 `while`，而无需将 `while`（或任何其他命令）放在子 shell 中。

```shell
last_line='NULL'
while read line; do
  if [[ -n "${line}" ]]; then
    last_line="${line}"
  fi
done < <(your_command)

# 这将输出your_command的最后一个非空行
echo "${last_line}"
```

或者，使用 `readarray` 内置将文件读入数组，然后循环数组的内容。请注意（出于与上述相同的原因），你需要在使用 `readarray` 时使用进程替换而不是管道，但是这样做的好处是将循环的输入生成放在之前，而不是之后。

```shell
last_line='NULL'
readarray -t lines < <(your_command)
for line in "${lines[@]}"; do
  if [[ -n "${line}" ]]; then
    last_line="${line}"
  fi
done
echo "${last_line}"
```

> 注意：谨慎使用for循环来迭代输出，例如 `for var in $(...)`，因为输出是按空格分隔的，而不是按行分隔的。有时你会知道这是安全的，因为输出不会包含任何意外的空格，但是如果这不明显或者不会提高可读性（例如在 `$(...)` 中的长命令），那么使用 `while read` 循环或 `readarray` 通常更安全和更清晰。

### 算术运算

始终使用 `(( … ))` 或 `$(( … ))` 而不是 `let` 或 `$[ … ]` 或 `expr`。

永远不要使用 `$[ … ]` 语法、`expr` 命令或 `let` 内建命令。

`<` 和 `>` 在 `[[ … ]]` 表达式中不执行数值比较（它们执行词典排序比较，参见[Testing Strings](#testing-strings)）。最好根本不要在数值比较中使用 `[[ … ]]`，而是使用 `(( … ))`。

建议避免将 `(( … ))` 用作独立语句，否则要小心其表达式评估为零 - 特别是在启用了 `set -e` 的情况下。例如，`set -e; i=0; (( i++ ))` 将导致shell退出。

```shell
# 在字符串中执行简单的计算 - 注意在字符串内使用 $(( … ))
echo "$(( 2 + 2 )) is 4"

# 在测试中执行算术比较时
if (( a < b )); then
  …
fi

# 将一些计算结果分配给变量。
(( i = 10 * j + 400 ))
```

```shell
# 这种形式是不可移植的且已被弃用
i=$[2 * 10]

# 尽管外观上看起来像是声明性关键字之一，但'let'不是，所以未引用的赋值会受到通配符和单词分割的影响。
# 为了简单起见，避免使用'let'，使用(( … ))代替。
let i="2 + 2"

# expr实用程序是外部程序而不是shell内置命令。
i=$( expr 4 + 4 )

# 使用expr时，引号可能会出错。
i=$( expr 4 '*' 4 )
```

除了样式考虑之外，shell的内置算术比`expr`快得多。

在使用变量时，`${var}`和 `$var` 形式在 `$(( ... ))` 内部不是必需的。Shell 会为你查找 `var`，省略 `${…}` 会使代码更清晰。这与前面关于总是使用大括号的规则略有不同，因此这只是一个建议。

```shell
# 注意：请记得在可能的情况下将变量声明为整数，并优先使用局部变量而不是全局变量。
local -i hundred=$(( 10 * 10 ))
declare -i five=$(( 10 / 2 ))

# 将变量“i”增加三。
# 注意：
#  - 我们不写 ${i} 或 $i。
#  - 在((和)之间加上一个空格。
(( i += 3 ))

# 将变量“i”减少五。
(( i -= 5 ))

# 进行一些复杂的计算。
# 注意正常的算术运算符优先级。
hr=2
min=5
sec=30
echo $(( hr * 3600 + min * 60 + sec )) # 打印预期的7530
```

## 命名约定

### 函数名称

使用小写字母，并使用下划线来分隔单词。使用 `::` 来分隔库。在函数名称后面必须加上括号。关键字 `function` 是可选的，但在整个项目中必须一致使用。

如果你只写单个函数，使用小写字母，并用下划线分隔单词。如果你写一个包，使用 `::` 分隔包名。大括号必须与函数名在同一行（与 Google 的其他语言一样），函数名和括号之间不要空格。

```shell
# 单个函数
my_func() {
  …
}

# 包的一部分
mypackage::my_func() {
  …
}
```

当在函数名后面出现“()”时，关键字 `function` 是多余的，但它增强了对函数的快速识别。

### 变量名称

与函数名称相同。

循环中的变量名称应该对任何你正在遍历的变量进行相似命名。

```shell
for zone in "${zones[@]}"; do
  something_with "${zone}"
done
```

### 常量和环境变量名称

全部大写，用下划线分隔，声明在文件顶部。

常量和导出到环境的任何东西都应该是大写的。

```shell
# 常量
readonly PATH_TO_FILES='/some/path'

# 常量和环境变量
declare -xr ORACLE_SID='PROD'
```

有些东西在第一次设置时就变成了常量（例如，通过 getopts）。因此，在 getopts 或基于条件设置常量是可以的，但应立即将其设置为只读。为了清晰起见，推荐使用 `readonly` 或 `export` 而不是等效的 `declare` 命令。

```shell
VERBOSE='false'
while getopts 'v' flag; do
  case "${flag}" in
    v) VERBOSE='true' ;;
  esac
done
readonly VERBOSE
```

### 源文件名

小写，如果需要，用下划线分隔单词。

这是为了与Google中其他代码样式保持一致：`maketemplate` 或 `make_template`，但不是 `make-template`。

### 只读变量

使用 `readonly` 或 `declare -r` 确保它们是只读的。

由于全局变量在 shell 中被广泛使用，因此在处理它们时捕获错误很重要。当声明一个变量是只读的时候，要明确说明这一点。

```shell
zip_version="$(dpkg --status zip | grep Version: | cut -d ' ' -f 2)"
if [[ -z "${zip_version}" ]]; then
  error_message
else
  readonly zip_version
fi
```

### 使用局部变量

使用 `local` 声明特定于函数的变量。声明和赋值应该在不同的行上。

确保局部变量只在函数及其子函数中可见，方法是在声明时使用 `local`。这样可以避免污染全局命名空间，并意外设置可能在函数外部具有重要意义的变量。

当赋值值由命令替换提供时，声明和赋值必须是分开的语句；因为 `local` 内置不传播命令替换的退出代码。

```shell
my_func2() {
  local name="$1"

  # 声明和赋值应该在不同的行上：
  local my_var
  my_var="$(my_func)"
  (( $? == 0 )) || return

  …
}
```

```shell
my_func2() {
  # 不要这样做：
  # $? 将始终为零，因为它包含 'local' 的退出代码，而不是 my_func 的退出代码
  local my_var="$(my_func)"
  (( $? == 0 )) || return

  …
}
```

### 函数位置

将所有函数放在文件中常量的下方。不要在函数之间隐藏可执行代码。这样做会使代码难以跟踪，并在调试时导致不好的结果。

如果你有函数，把它们都放在文件的顶部附近。只有在声明函数之前才可以包含 include、`set` 语句和设置常量。

### 主函数

对于足够长以至少包含一个其他函数的脚本，需要一个名为 `main` 的函数。

为了方便找到程序的起始点，将主要代码放在一个名为 `main` 的函数中，作为最底部的函数。这样做既保持了与代码库中其余部分的一致性，也允许你将更多变量定义为 `local`（如果主要代码不是函数，则无法这样做）。文件中最后一个非注释行应该是对 `main` 的调用：

```shell
main "$@"
```

显然，对于只是线性流程的短脚本，使用 `main` 是过度的，因此不是必需的。

## 调用命令

### 检查返回值

始终检查返回值并提供有意义的返回值。

对于未使用管道的命令，使用 `$?` 或通过 `if` 语句直接检查以保持简单。

示例：

```shell
if ! mv "${file_list[@]}" "${dest_dir}/"; then
  echo "Unable to move ${file_list[*]} to ${dest_dir}" >&2
  exit 1
fi

# 或者
mv "${file_list[@]}" "${dest_dir}/"
if (( $? != 0 )); then
  echo "Unable to move ${file_list[*]} to ${dest_dir}" >&2
  exit 1
fi
```

Bash 还有 `PIPESTATUS` 变量，允许检查管道的所有部分的返回码。如果只需要检查整个管道的成功或失败，则可以接受以下内容：

```shell
tar -cf - ./* | ( cd "${dir}" && tar -xf - )
if (( PIPESTATUS[0] != 0 || PIPESTATUS[1] != 0 )); then
  echo "Unable to tar files to ${dir}" >&2
fi
```

但是，由于一旦执行任何其他命令，`PIPESTATUS` 就会被覆盖，如果根据管道中发生错误的位置以不同方式进行操作，则需要在运行命令后立即将 `PIPESTATUS` 赋值给另一个变量（不要忘记 `[` 是一个命令，并且会清除 `PIPESTATUS`）。

```shell
tar -cf - ./* | ( cd "${DIR}" && tar -xf - )
return_codes=( "${PIPESTATUS[@]}" )
if (( return_codes[0] != 0 )); then
  do_something
fi
if (( return_codes[1] != 0 )); then
  do_something_else
fi
```

### 内置命令与外部命令

在调用内置命令和调用单独进程之间进行选择时，选择内置命令。

我们更喜欢使用内置命令，例如 `bash(1)` 中的*参数扩展*函数，因为它更健壮和可移植（特别是与 `sed` 等工具相比）。

示例：

```shell
# 更喜欢这种方式：
addition=$(( X + Y ))
substitution="${string/#foo/bar}"
```

```shell
# 而不是这种方式：
addition="$(expr "${X}" + "${Y}")"
substitution="$(echo "${string}" | sed -e 's/^foo/bar/')"
```

## 结论

请理性思考，并*保持一致*。

请花几分钟阅读 [C++ 指南](https://google.github.io/styleguide/cppguide.html#Parting_Words)底部的告别语。
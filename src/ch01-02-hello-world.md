## Hello, World!

现在你已经安装了 Rust，是时候编写你的第一个 Rust 程序了。学习一门新语言时，传统上会编写一个小程序，将文本 `Hello, world!` 打印到屏幕上，所以我们在这里也这样做！

> 注意：本书假设你对命令行有基本的熟悉程度。Rust 对你的编辑器或工具以及代码存放的位置没有特定要求，所以如果你更喜欢使用集成开发环境（IDE）而不是命令行，可以随意使用你喜欢的 IDE。许多 IDE 现在都有一定程度的 Rust 支持；请查阅 IDE 的文档以获取详细信息。Rust 团队一直在专注于通过 `rust-analyzer` 提供出色的 IDE 支持。有关更多详细信息，请参阅 [附录 D][devtools]<!-- ignore -->。

### 创建项目目录

首先，你需要创建一个目录来存放你的 Rust 代码。Rust 并不关心你的代码存放在哪里，但为了本书中的练习和项目，我们建议在你的主目录下创建一个 _projects_ 目录，并将所有项目放在其中。

打开终端并输入以下命令来创建一个 _projects_ 目录，并在 _projects_ 目录内为 “Hello, world!” 项目创建一个目录。

对于 Linux、macOS 和 Windows 上的 PowerShell，输入以下命令：

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

对于 Windows CMD，输入以下命令：

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

### 编写和运行 Rust 程序

接下来，创建一个新的源文件并命名为 _main.rs_。Rust 文件总是以 _.rs_ 扩展名结尾。如果你的文件名包含多个单词，约定使用下划线分隔它们。例如，使用 _hello_world.rs_ 而不是 _helloworld.rs_。

现在打开你刚刚创建的 _main.rs_ 文件，并输入清单 1-1 中的代码。

<Listing number="1-1" file-name="main.rs" caption="一个打印 `Hello, world!` 的程序">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

保存文件并返回到你的终端窗口，确保你在 _~/projects/hello_world_ 目录中。在 Linux 或 macOS 上，输入以下命令来编译和运行该文件：

```console
$ rustc main.rs
$ ./main
Hello, world!
```

在 Windows 上，输入命令 `.\main.exe` 而不是 `./main`：

```powershell
> rustc main.rs
> .\main.exe
Hello, world!
```

无论你的操作系统是什么，字符串 `Hello, world!` 都应该打印到终端。如果你没有看到这个输出，请参考安装部分的 [“故障排除”][troubleshooting]<!-- ignore --> 部分以获取帮助。

如果 `Hello, world!` 确实打印出来了，恭喜你！你已经正式编写了一个 Rust 程序。这意味着你成为了一名 Rust 程序员——欢迎！

### Rust 程序的剖析

让我们详细回顾一下这个 “Hello, world!” 程序。首先是第一部分：

```rust
fn main() {

}
```

这些行定义了一个名为 `main` 的函数。`main` 函数很特殊：它是每个可执行的 Rust 程序中首先运行的代码。这里，第一行声明了一个名为 `main` 的函数，该函数没有参数并且不返回任何内容。如果有参数，它们会放在括号 `()` 内。

函数体被包裹在 `{}` 中。Rust 要求所有函数体周围都有大括号。良好的风格是将左大括号放在与函数声明相同的行上，并在两者之间添加一个空格。

> 注意：如果你想在 Rust 项目中坚持一种标准风格，可以使用一个名为 `rustfmt` 的自动格式化工具来以特定的风格格式化你的代码（有关 `rustfmt` 的更多信息，请参阅 [附录 D][devtools]<!-- ignore -->）。Rust 团队已将此工具包含在标准 Rust 发行版中，就像 `rustc` 一样，因此它应该已经安装在你的计算机上！

`main` 函数的主体包含以下代码：

```rust
println!("Hello, world!");
```

这一行完成了这个小程序的所有工作：它将文本打印到屏幕上。这里有三个重要的细节需要注意。

首先，`println!` 调用了 Rust 宏。如果它调用的是函数而不是宏，则会写成 `println`（没有 `!`）。我们将在第 20 章更详细地讨论 Rust 宏。目前，你只需要知道使用 `!` 表示你在调用宏而不是普通函数，并且宏并不总是遵循与函数相同的规则。

其次，你看到了 `"Hello, world!"` 字符串。我们将这个字符串作为参数传递给 `println!`，然后字符串被打印到屏幕上。

第三，我们在行尾加上了分号 (`;`)，这表示这个表达式结束了，下一个表达式即将开始。大多数 Rust 代码行都以分号结尾。

### 编译和运行是分开的步骤

你刚刚运行了一个新创建的程序，让我们来检查一下这个过程中的每个步骤。

在运行 Rust 程序之前，你必须使用 Rust 编译器编译它，方法是输入 `rustc` 命令并传递你的源文件名，如下所示：

```console
$ rustc main.rs
```

如果你有 C 或 C++ 背景，你会注意到这类似于 `gcc` 或 `clang`。成功编译后，Rust 会输出一个二进制可执行文件。

在 Linux、macOS 和 Windows 上的 PowerShell 中，你可以通过在 shell 中输入 `ls` 命令来查看可执行文件：

```console
$ ls
main  main.rs
```

在 Linux 和 macOS 上，你会看到两个文件。在 Windows 上使用 PowerShell 时，你会看到与使用 CMD 时相同的三个文件。在 Windows 上使用 CMD 时，你会输入以下命令：

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

这会显示带有 _.rs_ 扩展名的源代码文件、可执行文件（在 Windows 上是 _main.exe_，但在其他平台上都是 _main_），以及在使用 Windows 时，一个带有 _.pdb_ 扩展名的包含调试信息的文件。从这里开始，你可以运行 _main_ 或 _main.exe_ 文件，如下所示：

```console
$ ./main # 或者在 Windows 上使用 .\main.exe
```

如果你的 _main.rs_ 是你的 “Hello, world!” 程序，这一行会将 `Hello, world!` 打印到你的终端。

如果你更熟悉动态语言，如 Ruby、Python 或 JavaScript，你可能不习惯将编译和运行程序作为分开的步骤。Rust 是一种 _预编译_ 语言，这意味着你可以编译一个程序并将可执行文件交给其他人，他们甚至可以在没有安装 Rust 的情况下运行它。如果你给某人一个 _.rb_、_.py_ 或 _.js_ 文件，他们需要分别安装 Ruby、Python 或 JavaScript 实现。但在这些语言中，你只需要一个命令就可以编译和运行你的程序。语言设计中的一切都是权衡。

仅使用 `rustc` 编译对于简单的程序来说是可以的，但随着项目的增长，你会希望管理所有的选项并使分享代码变得容易。接下来，我们将向你介绍 Cargo 工具，它将帮助你编写现实世界中的 Rust 程序。

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
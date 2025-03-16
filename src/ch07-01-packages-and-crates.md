## 包和Crate

我们将首先介绍模块系统中的包（packages）和crate。

一个 _crate_ 是Rust编译器一次处理的最小代码单位。即使你运行的是 `rustc` 而不是 `cargo`，并且传递了一个单独的源代码文件（就像我们在第1章的“编写和运行Rust程序”中所做的那样），编译器也会将该文件视为一个crate。Crate可以包含模块，而这些模块可能定义在与crate一起编译的其他文件中，我们将在接下来的章节中看到这一点。

一个crate可以有两种形式之一：二进制crate或库crate。_二进制crate_ 是可以编译成可执行文件的程序，例如命令行程序或服务器。每个二进制crate必须有一个名为 `main` 的函数，用于定义可执行文件运行时的行为。我们到目前为止创建的所有crate都是二进制crate。

_库crate_ 没有 `main` 函数，也不会编译成可执行文件。相反，它们定义了旨在与多个项目共享的功能。例如，我们在[第2章][rand]<!-- ignore -->中使用的 `rand` crate提供了生成随机数的功能。大多数情况下，当Rustaceans提到“crate”时，他们指的是库crate，并且他们将“crate”与“库”这一通用编程概念互换使用。

_crate根_ 是Rust编译器开始处理的源文件，它构成了你的crate的根模块（我们将在[“定义模块以控制作用域和隐私”][modules]<!-- ignore -->中深入解释模块）。

一个 _包_ 是一个或多个crate的集合，提供一组功能。一个包包含一个 _Cargo.toml_ 文件，该文件描述了如何构建这些crate。Cargo实际上是一个包，其中包含了你一直用来构建代码的命令行工具的二进制crate。Cargo包还包含一个库crate，二进制crate依赖于这个库crate。其他项目可以依赖于Cargo库crate，以使用Cargo命令行工具所使用的相同逻辑。

一个包可以包含任意数量的二进制crate，但最多只能包含一个库crate。一个包必须至少包含一个crate，无论是库crate还是二进制crate。

让我们来看看当我们创建一个包时会发生什么。首先我们输入命令 `cargo new my-project`：

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

在我们运行 `cargo new my-project` 之后，我们使用 `ls` 查看Cargo创建的内容。在项目目录中，有一个 _Cargo.toml_ 文件，这表示我们有一个包。还有一个 _src_ 目录，其中包含 _main.rs_。在你的文本编辑器中打开 _Cargo.toml_，注意其中没有提到 _src/main.rs_。Cargo遵循一个约定，即 _src/main.rs_ 是与包同名的二进制crate的crate根。同样，Cargo知道如果包目录包含 _src/lib.rs_，则该包包含一个与包同名的库crate，并且 _src/lib.rs_ 是其crate根。Cargo将crate根文件传递给 `rustc` 以构建库或二进制文件。

在这里，我们有一个只包含 _src/main.rs_ 的包，这意味着它只包含一个名为 `my-project` 的二进制crate。如果一个包包含 _src/main.rs_ 和 _src/lib.rs_，那么它有两个crate：一个二进制crate和一个库crate，两者都与包同名。一个包可以通过将文件放在 _src/bin_ 目录中来拥有多个二进制crate：每个文件将是一个单独的二进制crate。

[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
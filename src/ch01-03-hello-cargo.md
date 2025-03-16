## Hello, Cargo!

Cargo 是 Rust 的构建系统和包管理器。大多数 Rust 开发者使用这个工具来管理他们的 Rust 项目，因为 Cargo 为你处理了许多任务，比如构建你的代码、下载你的代码所依赖的库，并构建这些库。（我们称你的代码所需的库为 _依赖项_。）

最简单的 Rust 程序，比如我们目前编写的这个，没有任何依赖项。如果我们使用 Cargo 构建“Hello, world!”项目，它只会使用 Cargo 处理代码构建的部分。随着你编写更复杂的 Rust 程序，你会添加依赖项，如果你使用 Cargo 启动项目，添加依赖项将会变得更加容易。

因为绝大多数 Rust 项目都使用 Cargo，本书的其余部分假设你也在使用 Cargo。如果你使用了[“安装”][installation]<!-- ignore -->部分讨论的官方安装程序，Cargo 会随 Rust 一起安装。如果你通过其他方式安装了 Rust，可以通过在终端中输入以下命令来检查是否安装了 Cargo：

```console
$ cargo --version
```

如果你看到一个版本号，说明你已经安装了 Cargo！如果你看到一个错误，比如 `command not found`，请查看你安装方法的文档，以确定如何单独安装 Cargo。

### 使用 Cargo 创建项目

让我们使用 Cargo 创建一个新项目，并看看它与我们最初的“Hello, world!”项目有何不同。导航回你的 _projects_ 目录（或你决定存储代码的任何地方）。然后，在任何操作系统上运行以下命令：

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

第一个命令创建了一个名为 _hello_cargo_ 的新目录和项目。我们将项目命名为 _hello_cargo_，Cargo 会在同名目录中创建其文件。

进入 _hello_cargo_ 目录并列出文件。你会看到 Cargo 为我们生成了两个文件和一个目录：一个 _Cargo.toml_ 文件和一个包含 _main.rs_ 文件的 _src_ 目录。

它还初始化了一个新的 Git 仓库，并附带了一个 _.gitignore_ 文件。如果你在现有的 Git 仓库中运行 `cargo new`，则不会生成 Git 文件；你可以通过使用 `cargo new --vcs=git` 来覆盖此行为。

> 注意：Git 是一个常见的版本控制系统。你可以通过使用 `--vcs` 标志来更改 `cargo new` 以使用不同的版本控制系统或不使用版本控制系统。运行 `cargo new --help` 查看可用的选项。

在你选择的文本编辑器中打开 _Cargo.toml_。它应该类似于 Listing 1-2 中的代码。

<Listing number="1-2" file-name="Cargo.toml" caption="由 `cargo new` 生成的 *Cargo.toml* 的内容">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

这个文件采用 [_TOML_][toml]<!-- ignore -->（_Tom’s Obvious, Minimal Language_）格式，这是 Cargo 的配置格式。

第一行 `[package]` 是一个部分标题，表示以下语句正在配置一个包。随着我们向这个文件添加更多信息，我们将添加其他部分。

接下来的三行设置了 Cargo 编译你的程序所需的配置信息：名称、版本和要使用的 Rust 版本。我们将在[附录 E][appendix-e]<!-- ignore -->中讨论 `edition` 键。

最后一行 `[dependencies]` 是一个部分的开始，用于列出你的项目的任何依赖项。在 Rust 中，代码包被称为 _crates_。我们在这个项目中不需要任何其他 crate，但在第二章的第一个项目中会需要，所以我们将在那时使用这个依赖项部分。

现在打开 _src/main.rs_ 并查看：

<span class="filename">文件名: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo 为你生成了一个“Hello, world!”程序，就像我们在 Listing 1-1 中编写的那个！到目前为止，我们的项目与 Cargo 生成的项目之间的区别在于，Cargo 将代码放在了 _src_ 目录中，并且我们在顶层目录中有一个 _Cargo.toml_ 配置文件。

Cargo 期望你的源文件位于 _src_ 目录中。顶层项目目录仅用于 README 文件、许可证信息、配置文件以及任何与代码无关的内容。使用 Cargo 有助于你组织项目。每个东西都有其位置，每个东西都在其位置上。

如果你启动了一个不使用 Cargo 的项目，就像我们使用“Hello, world!”项目一样，你可以将其转换为使用 Cargo 的项目。将项目代码移动到 _src_ 目录中，并创建一个适当的 _Cargo.toml_ 文件。获取该 _Cargo.toml_ 文件的一个简单方法是运行 `cargo init`，它会自动为你创建它。

### 构建和运行 Cargo 项目

现在让我们看看使用 Cargo 构建和运行“Hello, world!”程序时有什么不同！从你的 _hello_cargo_ 目录中，通过输入以下命令来构建你的项目：

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

这个命令在 _target/debug/hello_cargo_（或在 Windows 上是 _target\debug\hello_cargo.exe_）中创建一个可执行文件，而不是在你的当前目录中。因为默认构建是调试构建，Cargo 将二进制文件放在名为 _debug_ 的目录中。你可以使用以下命令运行可执行文件：

```console
$ ./target/debug/hello_cargo # 或者在 Windows 上是 .\target\debug\hello_cargo.exe
Hello, world!
```

如果一切顺利，`Hello, world!` 应该会打印到终端。第一次运行 `cargo build` 还会导致 Cargo 在顶层创建一个新文件：_Cargo.lock_。这个文件跟踪项目中依赖项的确切版本。这个项目没有依赖项，所以文件有点稀疏。你永远不需要手动更改这个文件；Cargo 会为你管理其内容。

我们刚刚使用 `cargo build` 构建了一个项目，并使用 `./target/debug/hello_cargo` 运行它，但我们也可以使用 `cargo run` 来编译代码，然后在一个命令中运行生成的可执行文件：

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

使用 `cargo run` 比记住运行 `cargo build` 然后使用二进制文件的完整路径更方便，因此大多数开发者使用 `cargo run`。

注意，这次我们没有看到表明 Cargo 正在编译 `hello_cargo` 的输出。Cargo 发现文件没有改变，所以它没有重新构建，而是直接运行了二进制文件。如果你修改了源代码，Cargo 会在运行之前重新构建项目，你会看到以下输出：

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo 还提供了一个名为 `cargo check` 的命令。这个命令快速检查你的代码以确保它可以编译，但不会生成可执行文件：

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

为什么你不需要可执行文件？通常，`cargo check` 比 `cargo build` 快得多，因为它跳过了生成可执行文件的步骤。如果你在编写代码时不断检查你的工作，使用 `cargo check` 将加快让你知道你的项目是否仍在编译的过程！因此，许多 Rust 开发者在编写程序时定期运行 `cargo check`，以确保它能够编译。然后，当他们准备好使用可执行文件时，他们会运行 `cargo build`。

让我们回顾一下我们到目前为止学到的关于 Cargo 的内容：

- 我们可以使用 `cargo new` 创建一个项目。
- 我们可以使用 `cargo build` 构建一个项目。
- 我们可以使用 `cargo run` 在一个步骤中构建并运行一个项目。
- 我们可以使用 `cargo check` 构建一个项目而不生成二进制文件以检查错误。
- Cargo 将构建结果存储在 _target/debug_ 目录中，而不是与我们的代码保存在同一目录中。

使用 Cargo 的另一个优点是，无论你在哪个操作系统上工作，命令都是相同的。因此，在这一点上，我们将不再提供针对 Linux 和 macOS 与 Windows 的具体说明。

### 发布构建

当你的项目最终准备好发布时，你可以使用 `cargo build --release` 来编译它并进行优化。这个命令将在 _target/release_ 而不是 _target/debug_ 中创建一个可执行文件。优化使你的 Rust 代码运行得更快，但启用它们会延长程序的编译时间。这就是为什么有两种不同的配置文件：一种用于开发，当你希望快速且频繁地重新构建时；另一种用于构建最终程序，你将把它交给用户，不会重复构建，并且会尽可能快地运行。如果你在基准测试代码的运行时间，请确保运行 `cargo build --release` 并使用 _target/release_ 中的可执行文件进行基准测试。

### Cargo 作为惯例

对于简单的项目，Cargo 并没有比直接使用 `rustc` 提供太多价值，但随着你的程序变得更加复杂，它将证明其价值。一旦程序增长到多个文件或需要依赖项，让 Cargo 协调构建会容易得多。

尽管 `hello_cargo` 项目很简单，但它现在使用了许多你在 Rust 职业生涯中会使用的真实工具。事实上，要处理任何现有项目，你可以使用以下命令通过 Git 检出代码，切换到该项目的目录，并进行构建：

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

有关 Cargo 的更多信息，请查看[其文档][cargo]。

## 总结

你已经开始了 Rust 之旅的精彩开端！在本章中，你学会了如何：

- 使用 `rustup` 安装最新的稳定版 Rust
- 更新到较新的 Rust 版本
- 打开本地安装的文档
- 直接使用 `rustc` 编写并运行“Hello, world!”程序
- 使用 Cargo 的惯例创建并运行一个新项目

现在是构建一个更实质性的程序以习惯阅读和编写 Rust 代码的好时机。因此，在第二章中，我们将构建一个猜数字游戏程序。如果你更愿意从学习 Rust 中常见的编程概念开始，请参见第三章，然后返回第二章。

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
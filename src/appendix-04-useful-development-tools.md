## 附录 D - 有用的开发工具

在本附录中，我们将讨论 Rust 项目提供的一些有用的开发工具。我们将介绍自动格式化、快速应用警告修复的方法、代码检查工具（linter）以及与 IDE 的集成。

### 使用 `rustfmt` 进行自动格式化

`rustfmt` 工具会根据社区代码风格重新格式化你的代码。许多协作项目使用 `rustfmt` 来避免在编写 Rust 代码时关于使用哪种风格的争论：每个人都使用该工具来格式化他们的代码。

Rust 安装默认包含 `rustfmt`，因此你的系统上应该已经安装了 `rustfmt` 和 `cargo-fmt` 这两个程序。这两个命令类似于 `rustc` 和 `cargo`，其中 `rustfmt` 允许更细粒度的控制，而 `cargo-fmt` 理解使用 Cargo 的项目的约定。要格式化任何 Cargo 项目，请输入以下命令：

```sh
$ cargo fmt
```

运行此命令会重新格式化当前 crate 中的所有 Rust 代码。这只会改变代码风格，而不会改变代码的语义。

这个命令为你提供了 `rustfmt` 和 `cargo-fmt`，类似于 Rust 为你提供了 `rustc` 和 `cargo`。要格式化任何 Cargo 项目，请输入以下命令：

```console
$ cargo fmt
```

运行此命令会重新格式化当前 crate 中的所有 Rust 代码。这只会改变代码风格，而不会改变代码的语义。有关 `rustfmt` 的更多信息，请参阅[其文档][rustfmt]。

[rustfmt]: https://github.com/rust-lang/rustfmt

### 使用 `rustfix` 修复代码

`rustfix` 工具包含在 Rust 安装中，可以自动修复那些有明确修复方法的编译器警告，这些修复方法很可能是你想要的。你可能之前已经见过编译器警告。例如，考虑以下代码：

<span class="filename">文件名: src/main.rs</span>

```rust
fn main() {
    let mut x = 42;
    println!("{x}");
}
```

在这里，我们将变量 `x` 定义为可变的，但实际上我们从未改变它。Rust 对此发出警告：

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
2 |     let mut x = 0;
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```

警告建议我们移除 `mut` 关键字。我们可以通过运行 `cargo fix` 命令自动应用该建议：

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

当我们再次查看 _src/main.rs_ 时，我们会发现 `cargo fix` 已经更改了代码：

<span class="filename">文件名: src/main.rs</span>

```rust
fn main() {
    let x = 42;
    println!("{x}");
}
```

现在 `x` 变量是不可变的，警告也不再出现。

你还可以使用 `cargo fix` 命令在不同 Rust 版本之间迁移代码。版本迁移在[附录 E][editions] 中有详细介绍。

### 使用 Clippy 进行更多代码检查

Clippy 工具是一个代码检查工具集合，用于分析你的代码，以便你可以捕获常见错误并改进你的 Rust 代码。Clippy 包含在标准的 Rust 安装中。

要在任何 Cargo 项目上运行 Clippy 的代码检查，请输入以下命令：

```console
$ cargo clippy
```

例如，假设你编写了一个使用数学常数（如 pi）近似值的程序，如下所示：

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

在这个项目上运行 `cargo clippy` 会导致以下错误：

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

这个错误告诉你 Rust 已经定义了一个更精确的 `PI` 常量，如果你的程序使用这个常量会更正确。然后你会将代码更改为使用 `PI` 常量。以下代码不会导致 Clippy 产生任何错误或警告：

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

有关 Clippy 的更多信息，请参阅[其文档][clippy]。

[clippy]: https://github.com/rust-lang/rust-clippy

### 使用 `rust-analyzer` 进行 IDE 集成

为了帮助 IDE 集成，Rust 社区推荐使用 [`rust-analyzer`][rust-analyzer]<!-- ignore -->。这个工具是一组以编译器为中心的实用程序，支持 [Language Server Protocol][lsp]<!-- ignore -->，这是一个用于 IDE 和编程语言之间通信的规范。不同的客户端可以使用 `rust-analyzer`，例如 [Visual Studio Code 的 Rust 分析器插件][vscode]。

[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer

访问 `rust-analyzer` 项目的[主页][rust-analyzer]<!-- ignore --> 获取安装说明，然后在你的特定 IDE 中安装语言服务器支持。你的 IDE 将获得自动补全、跳转到定义和内联错误等功能。

[rust-analyzer]: https://rust-analyzer.github.io
[editions]: appendix-05-editions.md
<!-- 旧链接，请勿删除 -->

<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## 使用 `cargo install` 安装二进制文件

`cargo install` 命令允许你在本地安装和使用二进制 crate。这并不是为了取代系统包管理器；它的目的是为 Rust 开发者提供一种便捷的方式来安装其他人在 [crates.io](https://crates.io/)<!-- ignore --> 上分享的工具。需要注意的是，你只能安装具有二进制目标的包。_二进制目标_ 是指如果 crate 包含 _src/main.rs_ 文件或指定为二进制的其他文件时创建的可运行程序，与之相对的是库目标，库目标本身不可运行，但适合包含在其他程序中。通常，crate 的 _README_ 文件中会包含有关 crate 是库、具有二进制目标还是两者兼有的信息。

所有通过 `cargo install` 安装的二进制文件都存储在安装根目录的 _bin_ 文件夹中。如果你使用 _rustup.rs_ 安装了 Rust 并且没有进行任何自定义配置，那么这个目录将是 *$HOME/.cargo/bin*。确保该目录在你的 `$PATH` 中，以便能够运行通过 `cargo install` 安装的程序。

例如，在第 12 章中我们提到有一个名为 `ripgrep` 的 Rust 实现的 `grep` 工具，用于搜索文件。要安装 `ripgrep`，我们可以运行以下命令：

<!-- 手动重新生成
cargo install 一些你没有的东西，复制相关输出
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

输出的倒数第二行显示了安装的二进制文件的位置和名称，对于 `ripgrep` 来说，这个名称是 `rg`。只要安装目录在你的 `$PATH` 中，如前所述，你就可以运行 `rg --help` 并开始使用一个更快、更 Rust 化的工具来搜索文件！
## 安装

第一步是安装 Rust。我们将通过 `rustup` 下载 Rust，这是一个用于管理 Rust 版本和相关工具的命令行工具。下载时需要互联网连接。

> 注意：如果由于某种原因你不想使用 `rustup`，请参阅 [其他 Rust 安装方法页面][otherinstall] 了解更多选项。

以下步骤将安装最新稳定版本的 Rust 编译器。Rust 的稳定性保证确保书中所有能够编译的示例将继续在更新的 Rust 版本中编译。由于 Rust 经常改进错误消息和警告，不同版本之间的输出可能会有细微差别。换句话说，使用这些步骤安装的任何更新的稳定版本 Rust 都应该能够按预期与本书内容一起工作。

> ### 命令行表示法
>
> 在本章和整本书中，我们将展示一些在终端中使用的命令。需要在终端中输入的行都以 `$` 开头。你不需要输入 `$` 字符；它是命令行提示符，表示每个命令的开始。不以 `$` 开头的行通常显示前一个命令的输出。此外，特定于 PowerShell 的示例将使用 `>` 而不是 `$`。

### 在 Linux 或 macOS 上安装 `rustup`

如果你使用的是 Linux 或 macOS，打开终端并输入以下命令：

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

该命令下载一个脚本并启动 `rustup` 工具的安装，该工具将安装最新稳定版本的 Rust。系统可能会提示你输入密码。如果安装成功，将出现以下行：

```text
Rust is installed now. Great!
```

你还需要一个 _链接器_，这是 Rust 用来将其编译输出合并到一个文件中的程序。很可能你已经有一个了。如果遇到链接器错误，你应该安装一个 C 编译器，它通常会包含一个链接器。C 编译器也很有用，因为一些常见的 Rust 包依赖于 C 代码，并且需要一个 C 编译器。

在 macOS 上，你可以通过运行以下命令来获取 C 编译器：

```console
$ xcode-select --install
```

Linux 用户应根据其发行版的文档安装 GCC 或 Clang。例如，如果你使用 Ubuntu，可以安装 `build-essential` 包。

### 在 Windows 上安装 `rustup`

在 Windows 上，访问 [https://www.rust-lang.org/tools/install][install] 并按照安装 Rust 的说明操作。在安装过程中的某个时刻，系统会提示你安装 Visual Studio。这提供了编译程序所需的链接器和本地库。如果需要更多帮助，请参阅 [https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]。

本书的其余部分使用在 _cmd.exe_ 和 PowerShell 中都能工作的命令。如果有特定差异，我们会解释使用哪一个。

### 故障排除

要检查 Rust 是否正确安装，打开一个 shell 并输入以下命令：

```console
$ rustc --version
```

你应该看到最新稳定版本的版本号、提交哈希和提交日期，格式如下：

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

如果你看到这些信息，说明 Rust 安装成功！如果没有看到这些信息，请按照以下方式检查 Rust 是否在你的 `%PATH%` 系统变量中。

在 Windows CMD 中，使用：

```console
> echo %PATH%
```

在 PowerShell 中，使用：

```powershell
> echo $env:Path
```

在 Linux 和 macOS 中，使用：

```console
$ echo $PATH
```

如果这些都没问题但 Rust 仍然无法工作，你可以从多个地方获得帮助。了解如何在 [社区页面][community] 上联系其他 Rustaceans（我们给自己的昵称）。

### 更新和卸载

通过 `rustup` 安装 Rust 后，更新到新发布的版本非常简单。在你的 shell 中运行以下更新脚本：

```console
$ rustup update
```

要从你的 shell 中卸载 Rust 和 `rustup`，运行以下卸载脚本：

```console
$ rustup self uninstall
```

### 本地文档

Rust 的安装还包括一份本地文档副本，这样你就可以离线阅读。运行 `rustup doc` 在浏览器中打开本地文档。

每当标准库提供了某个类型或函数而你不确定它的作用或如何使用时，使用应用程序编程接口（API）文档来查找相关信息！

### 文本编辑器和集成开发环境

本书对用于编写 Rust 代码的工具没有任何假设。几乎任何文本编辑器都能完成这项工作！然而，许多文本编辑器和集成开发环境（IDE）都有内置的 Rust 支持。你可以在 Rust 网站的 [工具页面][tools] 上随时找到一份相当最新的编辑器和 IDE 列表。

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
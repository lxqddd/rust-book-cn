# Rust 编程语言

![构建状态](https://github.com/rust-lang/book/workflows/CI/badge.svg)

本仓库包含《Rust 编程语言》书籍的源代码。

[该书可以通过 No Starch Press 购买纸质版][nostarch]。

[nostarch]: https://nostarch.com/rust-programming-language-2nd-edition

你也可以在线阅读这本书。请参阅与最新 [stable]、[beta] 或 [nightly] Rust 版本一起发布的书籍。请注意，这些版本中的问题可能已经在本仓库中修复，因为这些版本的更新频率较低。

[stable]: https://doc.rust-lang.org/stable/book/
[beta]: https://doc.rust-lang.org/beta/book/
[nightly]: https://doc.rust-lang.org/nightly/book/

请参阅 [releases] 以下载书中所有代码清单的代码。

[releases]: https://github.com/rust-lang/book/releases

## 要求

构建本书需要 [mdBook]，最好是 rust-lang/rust 在 [此文件][rust-mdbook] 中使用的版本。要获取它：

[mdBook]: https://github.com/rust-lang/mdBook
[rust-mdbook]: https://github.com/rust-lang/rust/blob/master/src/tools/rustbook/Cargo.toml

```bash
$ cargo install mdbook --locked --version <version_num>
```

本书还使用了本仓库中的两个 mdbook 插件。如果你不安装它们，构建时会看到警告，并且输出可能不会正确显示，但你仍然可以构建本书。要使用这些插件，你应该运行：

```bash
$ cargo install --locked --path packages/mdbook-trpl
```

## 构建

要构建本书，请输入：

```bash
$ mdbook build
```

输出将位于 `book` 子目录中。要查看它，请在浏览器中打开。

_Firefox:_

```bash
$ firefox book/index.html                       # Linux
$ open -a "Firefox" book/index.html             # OS X
$ Start-Process "firefox.exe" .\book\index.html # Windows (PowerShell)
$ start firefox.exe .\book\index.html           # Windows (Cmd)
```

_Chrome:_

```bash
$ google-chrome book/index.html                 # Linux
$ open -a "Google Chrome" book/index.html       # OS X
$ Start-Process "chrome.exe" .\book\index.html  # Windows (PowerShell)
$ start chrome.exe .\book\index.html            # Windows (Cmd)
```

要运行测试：

```bash
$ cd packages/trpl
$ mdbook test --library-path packages/trpl/target/debug/deps
```

## 贡献

我们非常欢迎你的帮助！请参阅 [CONTRIBUTING.md][contrib] 了解我们正在寻找的贡献类型。

[contrib]: https://github.com/rust-lang/book/blob/main/CONTRIBUTING.md

由于本书是 [印刷版][nostarch]，并且我们希望尽可能保持在线版本与印刷版一致，因此我们可能需要比你所习惯的更长的时间来处理你的问题或拉取请求。

到目前为止，我们一直在与 [Rust Editions](https://doc.rust-lang.org/edition-guide/) 同步进行较大的修订。在这些较大的修订之间，我们只会修正错误。如果你的问题或拉取请求不仅仅是修复错误，它可能会等到我们下一次进行较大修订时再处理：预计需要几个月或几年的时间。感谢你的耐心！

### 翻译

我们非常欢迎帮助翻译本书！请参阅 [Translations] 标签以加入当前正在进行的翻译工作。打开一个新问题以开始新的语言翻译！我们正在等待 [mdbook 支持] 多语言功能，然后才会合并任何翻译，但请随时开始！

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang/mdBook/issues/5

## 拼写检查

要扫描源文件中的拼写错误，你可以使用 `ci` 目录中的 `spellcheck.sh` 脚本。它需要一个有效单词的字典，该字典在 `ci/dictionary.txt` 中提供。如果脚本产生了误报（例如，你使用了 `BTreeMap` 这个词，但脚本认为它是无效的），你需要将该词添加到 `ci/dictionary.txt` 中（保持排序以保持一致性）。
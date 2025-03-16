## 发布 Crate 到 Crates.io

我们已经使用过 [crates.io](https://crates.io/)<!-- ignore --> 上的包作为项目的依赖，但你也可以通过发布自己的包来与他人分享代码。[crates.io](https://crates.io/)<!-- ignore --> 上的 crate 注册表会分发你包的源代码，因此它主要托管开源的代码。

Rust 和 Cargo 有一些功能可以让你的发布包更容易被他人找到和使用。接下来我们将讨论其中的一些功能，然后解释如何发布一个包。

### 编写有用的文档注释

准确地记录你的包将帮助其他用户了解如何以及何时使用它们，因此值得花时间编写文档。在第 3 章中，我们讨论了如何使用两个斜杠 `//` 来注释 Rust 代码。Rust 还有一种特殊的注释用于文档，称为**文档注释**，它会生成 HTML 文档。HTML 会显示文档注释的内容，这些内容是为那些想要了解如何使用你的 crate 而不是如何实现它的程序员准备的。

文档注释使用三个斜杠 `///` 而不是两个，并支持 Markdown 语法来格式化文本。将文档注释放在它们所描述的项之前。Listing 14-1 展示了一个名为 `my_crate` 的 crate 中 `add_one` 函数的文档注释。

<Listing number="14-1" file-name="src/lib.rs" caption="函数的文档注释">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

在这里，我们描述了 `add_one` 函数的功能，开始了一个标题为 `Examples` 的部分，然后提供了演示如何使用 `add_one` 函数的代码。我们可以通过运行 `cargo doc` 从这个文档注释生成 HTML 文档。这个命令会运行 Rust 自带的 `rustdoc` 工具，并将生成的 HTML 文档放在 _target/doc_ 目录中。

为了方便，运行 `cargo doc --open` 会为你当前的 crate 生成 HTML 文档（以及所有依赖项的文档），并在浏览器中打开结果。导航到 `add_one` 函数，你会看到文档注释中的文本是如何渲染的，如图 14-1 所示：

<img alt="Rendered HTML documentation for the `add_one` function of `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">图 14-1: `add_one` 函数的 HTML 文档</span>

#### 常用的部分

我们在 Listing 14-1 中使用了 `# Examples` Markdown 标题来创建一个标题为 “Examples” 的 HTML 部分。以下是一些 crate 作者在文档中常用的其他部分：

- **Panics**：函数可能会 panic 的场景。不希望程序 panic 的调用者应确保不会在这些情况下调用该函数。
- **Errors**：如果函数返回 `Result`，描述可能发生的错误类型以及可能导致这些错误返回的条件对调用者有帮助，以便他们可以编写代码以不同方式处理不同类型的错误。
- **Safety**：如果函数调用是 `unsafe` 的（我们将在第 20 章讨论不安全代码），应该有一个部分解释为什么函数是不安全的，并涵盖函数期望调用者维护的不变量。

大多数文档注释不需要所有这些部分，但这是一个很好的清单，提醒你用户会感兴趣的代码方面。

#### 文档注释作为测试

在文档注释中添加示例代码块可以帮助演示如何使用你的库，这样做还有一个额外的好处：运行 `cargo test` 会将文档中的代码示例作为测试运行！没有什么比带有示例的文档更好的了。但没有什么比文档中的示例因为代码更改而无法工作更糟糕的了。如果我们使用 Listing 14-1 中的 `add_one` 函数的文档运行 `cargo test`，我们将在测试结果中看到一个如下所示的部分：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

现在，如果我们更改函数或示例，使得示例中的 `assert_eq!` panic，然后再次运行 `cargo test`，我们会看到文档测试捕捉到了示例和代码不同步的情况！

#### 注释包含的项

文档注释 `//!` 的风格将文档添加到包含注释的项，而不是注释后面的项。我们通常将这些文档注释放在 crate 根文件（通常是 _src/lib.rs_）或模块内部，以记录 crate 或模块的整体。

例如，要为包含 `add_one` 函数的 `my_crate` crate 添加描述其用途的文档，我们在 _src/lib.rs_ 文件的开头添加以 `//!` 开头的文档注释，如 Listing 14-2 所示：

<Listing number="14-2" file-name="src/lib.rs" caption="`my_crate` crate 的整体文档">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

注意，在最后一行以 `//!` 开头的注释之后没有任何代码。因为我们以 `//!` 而不是 `///` 开始注释，所以我们是在记录包含此注释的项，而不是注释后面的项。在这种情况下，该项是 _src/lib.rs_ 文件，即 crate 根。这些注释描述了整个 crate。

当我们运行 `cargo doc --open` 时，这些注释将显示在 `my_crate` 文档的首页，位于 crate 中公共项的列表上方，如图 14-2 所示。

<img alt="Rendered HTML documentation with a comment for the crate as a whole" src="img/trpl14-02.png" class="center" />

<span class="caption">图 14-2: `my_crate` 的渲染文档，包括描述 crate 整体的注释</span>

项内的文档注释对于描述 crate 和模块特别有用。使用它们来解释容器的整体目的，以帮助用户理解 crate 的组织结构。

### 使用 `pub use` 导出方便的公共 API

发布 crate 时，公共 API 的结构是一个重要的考虑因素。使用你的 crate 的人对结构的熟悉程度不如你，如果你的 crate 有一个大的模块层次结构，他们可能会很难找到他们想要使用的部分。

在第 7 章中，我们介绍了如何使用 `pub` 关键字使项公开，并使用 `use` 关键字将项引入作用域。然而，你在开发 crate 时认为合理的结构可能对你的用户来说并不方便。你可能希望将结构组织成包含多个层次的层次结构，但想要使用你定义在层次结构深处的类型的人可能会很难发现该类型的存在。他们也可能对必须输入 `use my_crate::some_module::another_module::UsefulType;` 而不是 `use my_crate::UsefulType;` 感到烦恼。

好消息是，如果结构对其他人来说不方便使用，你不必重新安排内部组织：相反，你可以使用 `pub use` 重新导出项，以创建一个与私有结构不同的公共结构。**重新导出**将一个位置的公共项在另一个位置公开，就好像它是在另一个位置定义的一样。

例如，假设我们创建了一个名为 `art` 的库来建模艺术概念。在这个库中有两个模块：一个 `kinds` 模块包含两个枚举 `PrimaryColor` 和 `SecondaryColor`，一个 `utils` 模块包含一个名为 `mix` 的函数，如 Listing 14-3 所示：

<Listing number="14-3" file-name="src/lib.rs" caption="一个 `art` 库，项组织在 `kinds` 和 `utils` 模块中">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

图 14-3 显示了由 `cargo doc` 生成的此 crate 文档的首页：

<img alt="Rendered documentation for the `art` crate that lists the `kinds` and `utils` modules" src="img/trpl14-03.png" class="center" />

<span class="caption">图 14-3: `art` crate 文档的首页，列出了 `kinds` 和 `utils` 模块</span>

请注意，`PrimaryColor` 和 `SecondaryColor` 类型没有列在首页，`mix` 函数也没有。我们必须点击 `kinds` 和 `utils` 才能看到它们。

另一个依赖此库的 crate 需要使用 `use` 语句将 `art` 中的项引入作用域，指定当前定义的模块结构。Listing 14-4 展示了一个使用 `art` crate 中的 `PrimaryColor` 和 `mix` 项的 crate 示例：

<Listing number="14-4" file-name="src/main.rs" caption="一个使用 `art` crate 项的 crate，其内部结构已导出">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

</Listing>

Listing 14-4 中使用 `art` crate 的代码作者必须弄清楚 `PrimaryColor` 在 `kinds` 模块中，而 `mix` 在 `utils` 模块中。`art` crate 的模块结构对开发 `art` crate 的开发人员更相关，而不是使用它的人。内部结构不包含任何对试图理解如何使用 `art` crate 的人有用的信息，反而会引起混淆，因为使用它的开发人员必须弄清楚在哪里查找，并且必须在 `use` 语句中指定模块名称。

为了从公共 API 中移除内部组织，我们可以修改 Listing 14-3 中的 `art` crate 代码，添加 `pub use` 语句以在顶层重新导出项，如 Listing 14-5 所示：

<Listing number="14-5" file-name="src/lib.rs" caption="添加 `pub use` 语句以重新导出项">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

`cargo doc` 为此 crate 生成的 API 文档现在将在首页列出并链接重新导出的项，如图 14-4 所示，使得 `PrimaryColor` 和 `SecondaryColor` 类型以及 `mix` 函数更容易找到。

<img alt="Rendered documentation for the `art` crate with the re-exports on the front page" src="img/trpl14-04.png" class="center" />

<span class="caption">图 14-4: `art` crate 文档的首页，列出了重新导出的项</span>

`art` crate 的用户仍然可以看到并使用 Listing 14-3 中的内部结构，如 Listing 14-4 所示，或者他们可以使用 Listing 14-5 中更方便的结构，如 Listing 14-6 所示：

<Listing number="14-6" file-name="src/main.rs" caption="一个使用 `art` crate 重新导出项的程序">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

在有许多嵌套模块的情况下，使用 `pub use` 在顶层重新导出类型可以显著改善使用 crate 的人的体验。`pub use` 的另一个常见用途是重新导出当前 crate 中依赖项的定义，以使该 crate 的定义成为你的 crate 公共 API 的一部分。

创建一个有用的公共 API 结构更像是一门艺术而不是科学，你可以通过迭代找到最适合用户的 API。选择 `pub use` 使你在如何内部组织 crate 方面具有灵活性，并将内部结构与呈现给用户的内容解耦。查看你安装的一些 crate 的代码，看看它们的内部结构是否与公共 API 不同。

### 设置 Crates.io 账户

在发布任何 crate 之前，你需要在 [crates.io](https://crates.io/)<!-- ignore --> 上创建一个账户并获取一个 API 令牌。为此，访问 [crates.io](https://crates.io/)<!-- ignore --> 的主页并通过 GitHub 账户登录。（目前 GitHub 账户是必需的，但未来该网站可能会支持其他创建账户的方式。）登录后，访问你的账户设置页面 [https://crates.io/me/](https://crates.io/me/)<!-- ignore --> 并获取你的 API 密钥。然后运行 `cargo login` 命令并在提示时粘贴你的 API 密钥，如下所示：

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

此命令将通知 Cargo 你的 API 令牌并将其本地存储在 _~/.cargo/credentials_ 中。请注意，此令牌是**秘密**：不要与任何人分享。如果你因任何原因与任何人分享了它，你应该撤销它并在 [crates.io](https://crates.io/)<!-- ignore --> 上生成一个新的令牌。

### 为新 Crate 添加元数据

假设你有一个想要发布的 crate。在发布之前，你需要在 crate 的 _Cargo.toml_ 文件的 `[package]` 部分添加一些元数据。

你的 crate 需要一个唯一的名称。当你在本地开发 crate 时，你可以随意命名 crate。然而，[crates.io](https://crates.io/)<!-- ignore --> 上的 crate 名称是按先到先得的原则分配的。一旦一个 crate 名称被占用，其他人就不能再发布同名的 crate。在尝试发布 crate 之前，搜索你想要使用的名称。如果该名称已被使用，你需要找到另一个名称并编辑 _Cargo.toml_ 文件中 `[package]` 部分下的 `name` 字段以使用新名称进行发布，如下所示：

<span class="filename">文件名: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

即使你选择了一个唯一的名称，当你运行 `cargo publish` 发布 crate 时，你会收到一个警告，然后是一个错误：

<!-- manual-regeneration
创建一个具有未注册名称的新包，不对生成的包进行进一步修改，
  因此它缺少描述和许可证字段。
cargo publish
复制下面相关的行
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error (status 400 Bad Request): missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for more information on configuring these fields
```

这是因为你缺少一些关键信息：描述和许可证是必需的，以便人们知道你的 crate 是做什么的以及他们可以在什么条款下使用它。在 _Cargo.toml_ 中，添加一个简短的描述，因为它会出现在搜索结果中。对于 `license` 字段，你需要提供一个**许可证标识符值**。[Linux 基金会的软件包数据交换 (SPDX)][spdx] 列出了你可以用于此值的标识符。例如，要指定你使用 MIT 许可证发布你的 crate，添加 `MIT` 标识符：

<span class="filename">文件名: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

如果你想使用 SPDX 中未列出的许可证，你需要将该许可证的文本放在一个文件中，将该文件包含在你的项目中，然后使用 `license-file` 指定该文件的名称，而不是使用 `license` 键。

关于哪种许可证适合你的项目的指导超出了本书的范围。许多 Rust 社区成员通过使用 `MIT OR Apache-2.0` 的双重许可证来发布他们的项目，与 Rust 相同。这种做法表明你也可以通过用 `OR` 分隔多个许可证标识符来为你的项目指定多个许可证。

有了唯一的名称、版本、描述和许可证后，准备发布的项目的 _Cargo.toml_ 文件可能如下所示：

<span class="filename">文件名: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Cargo 的文档](https://doc.rust-lang.org/cargo/) 描述了你可以指定的其他元数据，以确保其他人可以更容易地发现和使用你的 crate。

### 发布到 Crates.io

现在你已经创建了账户，保存了 API 令牌，选择了 crate 的名称，并指定了所需的元数据，你已经准备好发布了！发布 crate 会将特定版本上传到 [crates.io](https://crates.io/)<!-- ignore --> 供他人使用。

要小心，因为发布是**永久**的。版本永远不能被覆盖，代码也不能被删除。[crates.io](https://crates.io/)<!-- ignore --> 的一个主要目标是作为代码的永久存档，以便所有依赖 [crates.io](https://crates.io/)<!-- ignore --> 上的 crate 的项目构建都能继续工作。允许删除版本将使实现这一目标变得不可能。然而，你可以发布的 crate 版本数量没有限制。

再次运行 `cargo publish` 命令。现在它应该会成功：

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

恭喜！你现在已经与 Rust 社区分享了你的代码，任何人都可以轻松地将你的 crate 添加为他们项目的依赖项。

### 发布现有 Crate 的新版本

当你对 crate 进行了更改并准备发布新版本时，你可以在 _Cargo.toml_ 文件中更改 `version` 值并重新发布。使用 [语义版本控制规则][semver] 根据你所做的更改类型决定下一个版本号。然后运行 `cargo publish` 上传新版本。

<!-- 旧链接，不要删除 -->

<a id="removing-versions-from-cratesio-with-cargo-yank"></a>

### 使用 `cargo yank` 从 Crates.io 弃用版本

虽然你不能删除以前版本的 crate，但你可以防止任何未来的项目将它们添加为新的依赖项。这在某个 crate 版本因某种原因损坏时非常有用。在这种情况下，Cargo 支持弃用 crate 版本。

**弃用**一个版本会阻止新项目依赖该版本，同时允许所有现有依赖它的项目继续使用。本质上，弃用意味着所有具有 _Cargo.lock_ 的项目不会中断，并且任何未来生成的 _Cargo.lock_ 文件都不会使用弃用的版本。

要弃用 crate 的一个版本，在你之前发布的 crate 的目录中运行 `cargo yank` 并指定你想要弃用的版本。例如，如果我们发布了一个名为 `guessing_game` 的 crate 版本 1.0.1 并想要弃用它，在 `guessing_game` 的项目目录中运行：

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

通过在命令中添加 `--undo`，你还可以取消弃用并允许项目再次依赖某个版本：

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

弃用**不会**删除任何代码。例如，它不能删除意外上传的密钥。如果发生这种情况，你必须立即重置这些密钥。

[spdx]: http://spdx.org/licenses/
[semver]: http://semver.org/
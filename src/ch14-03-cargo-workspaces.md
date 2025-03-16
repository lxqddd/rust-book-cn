## Cargo 工作空间

在第 12 章中，我们构建了一个包含二进制 crate 和库 crate 的包。随着项目的发展，你可能会发现库 crate 变得越来越大，并且你希望将包进一步拆分为多个库 crate。Cargo 提供了一个称为 **工作空间** 的功能，可以帮助管理多个相关的包，这些包是同时开发的。

### 创建工作空间

**工作空间** 是一组共享同一个 `Cargo.lock` 和输出目录的包。让我们使用工作空间来创建一个项目——我们将使用简单的代码，以便我们可以专注于工作空间的结构。有多种方式来构建工作空间，因此我们只展示一种常见的方式。我们将创建一个包含一个二进制 crate 和两个库 crate 的工作空间。二进制 crate 将提供主要功能，并依赖于这两个库 crate。一个库将提供一个 `add_one` 函数，另一个库将提供一个 `add_two` 函数。这三个 crate 将属于同一个工作空间。我们首先为工作空间创建一个新目录：

```console
$ mkdir add
$ cd add
```

接下来，在 `add` 目录中，我们创建 `Cargo.toml` 文件，该文件将配置整个工作空间。这个文件不会包含 `[package]` 部分。相反，它将从 `[workspace]` 部分开始，该部分允许我们向工作空间添加成员。我们还通过将 `resolver` 设置为 `"3"`，确保在工作空间中使用最新版本的 Cargo 解析器算法。

<span class="filename">文件名: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

接下来，我们将在 `add` 目录中通过运行 `cargo new` 来创建 `adder` 二进制 crate：

```console
$ cargo new adder
    Creating binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

在工作空间内运行 `cargo new` 还会自动将新创建的包添加到工作空间 `Cargo.toml` 中的 `[workspace]` 定义的 `members` 键中，如下所示：

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

此时，我们可以通过运行 `cargo build` 来构建工作空间。你的 `add` 目录中的文件应该如下所示：

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

工作空间在顶层有一个 `target` 目录，编译后的工件将放置在其中；`adder` 包没有自己的 `target` 目录。即使我们从 `adder` 目录内运行 `cargo build`，编译后的工件仍然会放在 `add/target` 中，而不是 `add/adder/target`。Cargo 这样构建工作空间的 `target` 目录是因为工作空间中的 crate 是相互依赖的。如果每个 crate 都有自己的 `target` 目录，每个 crate 将不得不重新编译工作空间中的其他 crate，以便将工件放在自己的 `target` 目录中。通过共享一个 `target` 目录，crate 可以避免不必要的重新构建。

### 在工作空间中创建第二个包

接下来，让我们在工作空间中创建另一个成员包，并将其命名为 `add_one`。生成一个名为 `add_one` 的新库 crate：

```console
$ cargo new add_one --lib
    Creating library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

顶层的 `Cargo.toml` 现在将在 `members` 列表中包含 `add_one` 路径：

<span class="filename">文件名: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

你的 `add` 目录现在应该包含以下目录和文件：

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

在 `add_one/src/lib.rs` 文件中，让我们添加一个 `add_one` 函数：

<span class="filename">文件名: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

现在我们可以让 `adder` 包中的二进制文件依赖于包含我们库的 `add_one` 包。首先，我们需要在 `adder/Cargo.toml` 中添加对 `add_one` 的路径依赖。

<span class="filename">文件名: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo 不会假设工作空间中的 crate 会相互依赖，因此我们需要明确依赖关系。

接下来，让我们在 `adder` crate 中使用 `add_one` 函数（来自 `add_one` crate）。打开 `adder/src/main.rs` 文件，并将 `main` 函数修改为调用 `add_one` 函数，如代码清单 14-7 所示。

<Listing number="14-7" file-name="adder/src/main.rs" caption="在 `adder` crate 中使用 `add_one` 库 crate">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

让我们通过在顶层的 `add` 目录中运行 `cargo build` 来构建工作空间！

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

要从 `add` 目录运行二进制 crate，我们可以使用 `-p` 参数和包名称来指定要运行的工作空间中的包：

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

这将运行 `adder/src/main.rs` 中的代码，该代码依赖于 `add_one` crate。

#### 在工作空间中依赖外部包

请注意，工作空间在顶层只有一个 `Cargo.lock` 文件，而不是在每个 crate 的目录中都有一个 `Cargo.lock`。这确保了所有 crate 都使用相同版本的所有依赖项。如果我们将 `rand` 包添加到 `adder/Cargo.toml` 和 `add_one/Cargo.toml` 文件中，Cargo 会将这两个文件解析为 `rand` 的一个版本，并将其记录在唯一的 `Cargo.lock` 中。使工作空间中的所有 crate 使用相同的依赖项意味着这些 crate 将始终相互兼容。让我们将 `rand` crate 添加到 `add_one/Cargo.toml` 文件的 `[dependencies]` 部分，以便我们可以在 `add_one` crate 中使用 `rand` crate：

<span class="filename">文件名: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

我们现在可以将 `use rand;` 添加到 `add_one/src/lib.rs` 文件中，并通过在 `add` 目录中运行 `cargo build` 来构建整个工作空间，这将引入并编译 `rand` crate。我们会收到一个警告，因为我们没有引用我们引入作用域的 `rand`：

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

顶层的 `Cargo.lock` 现在包含 `add_one` 对 `rand` 的依赖信息。然而，即使 `rand` 在工作空间的某个地方被使用，我们也不能在工作空间的其他 crate 中使用它，除非我们也将 `rand` 添加到它们的 `Cargo.toml` 文件中。例如，如果我们将 `use rand;` 添加到 `adder/src/main.rs` 文件中，我们会得到一个错误：

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

要解决这个问题，编辑 `adder` 包的 `Cargo.toml` 文件，并指示 `rand` 也是它的依赖项。构建 `adder` 包将把 `rand` 添加到 `Cargo.lock` 中 `adder` 的依赖项列表中，但不会下载额外的 `rand` 副本。Cargo 将确保工作空间中使用 `rand` 包的每个包中的每个 crate 都使用相同的版本，只要它们指定了兼容的 `rand` 版本，从而节省空间并确保工作空间中的 crate 相互兼容。

如果工作空间中的 crate 指定了相同依赖项的不兼容版本，Cargo 将解析每个版本，但仍会尝试解析尽可能少的版本。

#### 向工作空间添加测试

作为另一个增强功能，让我们在 `add_one` crate 中添加一个 `add_one::add_one` 函数的测试：

<span class="filename">文件名: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

现在在顶层的 `add` 目录中运行 `cargo test`。在像这样结构的工作空间中运行 `cargo test` 将运行工作空间中所有 crate 的测试：

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

输出的第一部分显示 `add_one` crate 中的 `it_works` 测试通过了。下一部分显示在 `adder` crate 中找到了零个测试，最后一部分显示在 `add_one` crate 中找到了零个文档测试。

我们还可以通过使用 `-p` 标志并指定我们要测试的 crate 名称，从顶层目录运行工作空间中特定 crate 的测试：

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

此输出显示 `cargo test` 只运行了 `add_one` crate 的测试，而没有运行 `adder` crate 的测试。

如果你将工作空间中的 crate 发布到 [crates.io](https://crates.io/)，工作空间中的每个 crate 都需要单独发布。与 `cargo test` 类似，我们可以通过使用 `-p` 标志并指定要发布的 crate 名称来发布工作空间中的特定 crate。

作为额外的练习，以类似于 `add_one` crate 的方式向此工作空间添加一个 `add_two` crate！

随着项目的增长，考虑使用工作空间：它使你能够处理更小、更易于理解的组件，而不是一大块代码。此外，如果 crate 经常同时更改，将它们保持在工作空间中可以使 crate 之间的协调更容易。
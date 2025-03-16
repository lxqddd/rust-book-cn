## 测试组织

正如本章开头提到的，测试是一个复杂的领域，不同的人使用不同的术语和组织方式。Rust 社区将测试分为两大类：单元测试和集成测试。**单元测试**通常较小且更集中，一次只测试一个模块，并且可以测试私有接口。**集成测试**则完全独立于你的库，它们像其他外部代码一样使用你的代码，只使用公共接口，并且可能在每个测试中涉及多个模块。

编写这两种测试对于确保库的各个部分单独和整体都能按预期工作非常重要。

### 单元测试

单元测试的目的是将代码的每个单元与其他代码隔离开来进行测试，以便快速定位代码是否按预期工作。你会将单元测试放在 `src` 目录中，与它们测试的代码放在同一个文件中。惯例是在每个文件中创建一个名为 `tests` 的模块来包含测试函数，并使用 `cfg(test)` 注解该模块。

#### 测试模块和 `#[cfg(test)]`

`tests` 模块上的 `#[cfg(test)]` 注解告诉 Rust 只在运行 `cargo test` 时编译和运行测试代码，而不是在运行 `cargo build` 时。这样当你只想构建库时可以节省编译时间，并且由于测试代码不包含在编译结果中，可以节省编译产物的空间。你会看到，由于集成测试放在不同的目录中，它们不需要 `#[cfg(test)]` 注解。然而，由于单元测试与代码放在同一个文件中，你会使用 `#[cfg(test)]` 来指定它们不应包含在编译结果中。

回想一下，当我们在本章的第一节生成新的 `adder` 项目时，Cargo 为我们生成了以下代码：

<span class="filename">文件名: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

在自动生成的 `tests` 模块上，属性 `cfg` 代表 **配置**，并告诉 Rust 只有在特定配置选项下才包含以下项。在这种情况下，配置选项是 `test`，这是 Rust 提供的用于编译和运行测试的选项。通过使用 `cfg` 属性，Cargo 只在我们主动运行 `cargo test` 时编译我们的测试代码。这包括可能在这个模块中的任何辅助函数，以及用 `#[test]` 注解的函数。

#### 测试私有函数

测试社区内部对于是否应该直接测试私有函数存在争议，而其他语言使得测试私有函数变得困难或不可能。无论你遵循哪种测试理念，Rust 的隐私规则确实允许你测试私有函数。考虑 Listing 11-12 中的代码，其中包含私有函数 `internal_adder`。

<Listing number="11-12" file-name="src/lib.rs" caption="测试私有函数">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

注意，`internal_adder` 函数没有标记为 `pub`。测试只是 Rust 代码，`tests` 模块只是另一个模块。正如我们在[“模块树中引用项的路径”][paths]<!-- ignore -->中讨论的那样，子模块中的项可以使用其祖先模块中的项。在这个测试中，我们使用 `use super::*` 将 `tests` 模块的父模块的所有项引入作用域，然后测试可以调用 `internal_adder`。如果你认为不应该测试私有函数，Rust 中没有任何东西会强迫你这样做。

### 集成测试

在 Rust 中，集成测试完全独立于你的库。它们像其他代码一样使用你的库，这意味着它们只能调用库的公共 API 中的函数。它们的目的是测试库的许多部分是否能正确协同工作。单独工作正常的代码单元在集成时可能会出现问题，因此集成代码的测试覆盖也很重要。要创建集成测试，首先需要一个 `tests` 目录。

#### `tests` 目录

我们在项目目录的顶层创建一个 `tests` 目录，与 `src` 目录并列。Cargo 知道要在这个目录中查找集成测试文件。然后我们可以创建任意数量的测试文件，Cargo 会将每个文件编译为一个独立的 crate。

让我们创建一个集成测试。在 `src/lib.rs` 文件中仍然包含 Listing 11-12 的代码，创建一个 `tests` 目录，并创建一个名为 `tests/integration_test.rs` 的新文件。你的目录结构应该如下所示：

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

将 Listing 11-13 中的代码输入到 `tests/integration_test.rs` 文件中。

<Listing number="11-13" file-name="tests/integration_test.rs" caption="`adder` crate 中函数的集成测试">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

`tests` 目录中的每个文件都是一个独立的 crate，因此我们需要将我们的库引入每个测试 crate 的作用域。为此，我们在代码顶部添加 `use adder::add_two;`，这在单元测试中是不需要的。

我们不需要在 `tests/integration_test.rs` 中使用 `#[cfg(test)]` 注解任何代码。Cargo 会特殊处理 `tests` 目录，并且只在我们运行 `cargo test` 时编译该目录中的文件。现在运行 `cargo test`：

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

输出的三个部分包括单元测试、集成测试和文档测试。请注意，如果某个部分的任何测试失败，后续部分将不会运行。例如，如果单元测试失败，将不会有集成测试和文档测试的输出，因为这些测试只有在所有单元测试都通过时才会运行。

单元测试的第一部分与我们之前看到的一样：每个单元测试一行（我们在 Listing 11-12 中添加了一个名为 `internal` 的测试），然后是单元测试的总结行。

集成测试部分以 `Running tests/integration_test.rs` 开始。接下来是该集成测试中每个测试函数的一行，以及集成测试结果的总结行，紧接着是 `Doc-tests adder` 部分。

每个集成测试文件都有自己的部分，因此如果我们在 `tests` 目录中添加更多文件，将会有更多的集成测试部分。

我们仍然可以通过将测试函数的名称作为参数传递给 `cargo test` 来运行特定的集成测试函数。要运行特定集成测试文件中的所有测试，请使用 `cargo test` 的 `--test` 参数，后跟文件名：

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

此命令仅运行 `tests/integration_test.rs` 文件中的测试。

#### 集成测试中的子模块

随着你添加更多的集成测试，你可能希望在 `tests` 目录中创建更多文件来帮助组织它们；例如，你可以按它们测试的功能对测试函数进行分组。如前所述，`tests` 目录中的每个文件都被编译为独立的 crate，这对于创建独立的作用域以更接近地模拟最终用户使用你的 crate 的方式非常有用。然而，这意味着 `tests` 目录中的文件与 `src` 目录中的文件行为不同，正如你在第 7 章中学到的关于如何将代码分离到模块和文件中的内容。

当你有一组辅助函数要在多个集成测试文件中使用，并尝试按照第 7 章中[“将模块分离到不同文件”][separating-modules-into-files]<!-- ignore -->部分的步骤将它们提取到一个公共模块时，`tests` 目录文件的不同行为最为明显。例如，如果我们创建 `tests/common.rs` 并在其中放置一个名为 `setup` 的函数，我们可以向 `setup` 添加一些代码，以便从多个测试文件中的多个测试函数调用：

<span class="filename">文件名: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

当我们再次运行测试时，我们会在测试输出中看到 `common.rs` 文件的新部分，即使该文件不包含任何测试函数，我们也没有从任何地方调用 `setup` 函数：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

在测试结果中看到 `common` 并显示 `running 0 tests` 并不是我们想要的。我们只是想与其他集成测试文件共享一些代码。为了避免 `common` 出现在测试输出中，我们不会创建 `tests/common.rs`，而是创建 `tests/common/mod.rs`。项目目录现在如下所示：

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

这是 Rust 也理解的旧命名约定，我们在第 7 章的[“替代文件路径”][alt-paths]<!-- ignore -->中提到过。以这种方式命名文件告诉 Rust 不要将 `common` 模块视为集成测试文件。当我们将 `setup` 函数代码移动到 `tests/common/mod.rs` 并删除 `tests/common.rs` 文件时，测试输出中的部分将不再出现。`tests` 目录的子目录中的文件不会被编译为独立的 crate，也不会在测试输出中有部分。

在我们创建 `tests/common/mod.rs` 之后，我们可以从任何集成测试文件中将其作为模块使用。以下是从 `tests/integration_test.rs` 中的 `it_adds_two` 测试调用 `setup` 函数的示例：

<span class="filename">文件名: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

注意，`mod common;` 声明与我们在 Listing 7-21 中演示的模块声明相同。然后，在测试函数中，我们可以调用 `common::setup()` 函数。

#### 二进制 crate 的集成测试

如果我们的项目是一个只包含 `src/main.rs` 文件而没有 `src/lib.rs` 文件的二进制 crate，我们无法在 `tests` 目录中创建集成测试，并使用 `use` 语句将 `src/main.rs` 文件中定义的函数引入作用域。只有库 crate 会暴露其他 crate 可以使用的函数；二进制 crate 旨在独立运行。

这是 Rust 项目提供二进制文件时通常有一个简单的 `src/main.rs` 文件调用 `src/lib.rs` 文件中的逻辑的原因之一。使用这种结构，集成测试**可以**通过 `use` 测试库 crate 以使重要功能可用。如果重要功能正常工作，`src/main.rs` 文件中的少量代码也会正常工作，而这少量代码不需要测试。

## 总结

Rust 的测试功能提供了一种指定代码应如何运行的方式，以确保即使在你进行更改时，它仍能按预期工作。单元测试分别测试库的不同部分，并且可以测试私有实现细节。集成测试检查库的许多部分是否能正确协同工作，并且它们使用库的公共 API 来测试代码，就像外部代码使用它一样。尽管 Rust 的类型系统和所有权规则有助于防止某些类型的错误，但测试对于减少与代码预期行为相关的逻辑错误仍然很重要。

让我们结合你在本章和之前章节中学到的知识来做一个项目吧！

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]: ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths
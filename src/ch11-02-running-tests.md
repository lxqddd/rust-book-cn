## 控制测试的运行方式

就像 `cargo run` 会编译代码并运行生成的二进制文件一样，`cargo test` 会在测试模式下编译代码并运行生成的测试二进制文件。`cargo test` 生成的二进制文件的默认行为是并行运行所有测试，并捕获测试运行期间生成的输出，防止输出显示在终端上，从而更容易阅读与测试结果相关的输出。然而，你可以通过指定命令行选项来更改这种默认行为。

一些命令行选项传递给 `cargo test`，而另一些则传递给生成的测试二进制文件。为了区分这两种类型的参数，你需要列出传递给 `cargo test` 的参数，然后是分隔符 `--`，接着是传递给测试二进制文件的参数。运行 `cargo test --help` 会显示可以与 `cargo test` 一起使用的选项，而运行 `cargo test -- --help` 会显示可以在分隔符后使用的选项。这些选项也在 [rustc 书][rustc] 的 [“Tests” 部分][tests] 中有详细说明。

[tests]: https://doc.rust-lang.org/rustc/tests/index.html
[rustc]: https://doc.rust-lang.org/rustc/index.html

### 并行或连续运行测试

当你运行多个测试时，默认情况下它们会使用线程并行运行，这意味着它们会更快地完成运行，并且你会更快地得到反馈。由于测试是同时运行的，你必须确保你的测试不相互依赖，也不依赖于任何共享状态，包括共享环境，例如当前工作目录或环境变量。

例如，假设每个测试都运行一些代码，这些代码会在磁盘上创建一个名为 _test-output.txt_ 的文件，并向该文件写入一些数据。然后每个测试读取该文件中的数据，并断言该文件包含一个特定值，这个值在每个测试中都是不同的。由于测试是同时运行的，一个测试可能会在另一个测试写入和读取文件之间的时间内覆盖该文件。然后第二个测试将失败，不是因为代码不正确，而是因为测试在并行运行时相互干扰。一个解决方案是确保每个测试写入不同的文件；另一个解决方案是一次只运行一个测试。

如果你不想并行运行测试，或者希望对使用的线程数进行更细粒度的控制，你可以向测试二进制文件传递 `--test-threads` 标志和你希望使用的线程数。看看以下示例：

```console
$ cargo test -- --test-threads=1
```

我们将测试线程数设置为 `1`，告诉程序不要使用任何并行性。使用一个线程运行测试会比并行运行它们花费更长的时间，但如果测试共享状态，它们不会相互干扰。

### 显示函数输出

默认情况下，如果测试通过，Rust 的测试库会捕获打印到标准输出的任何内容。例如，如果我们在测试中调用 `println!` 并且测试通过，我们不会在终端中看到 `println!` 的输出；我们只会看到指示测试通过的行。如果测试失败，我们会在失败消息的其余部分中看到打印到标准输出的内容。

例如，Listing 11-10 有一个愚蠢的函数，它会打印其参数的值并返回 10，还有一个通过的测试和一个失败的测试。

<Listing number="11-10" file-name="src/lib.rs" caption="Tests for a function that calls `println!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

当我们使用 `cargo test` 运行这些测试时，我们会看到以下输出：

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

请注意，在此输出中我们看不到 `I got the value 4`，这是在通过的测试运行时打印的。该输出已被捕获。失败的测试的输出 `I got the value 8` 出现在测试摘要输出的部分中，该部分还显示了测试失败的原因。

如果我们还想看到通过测试的打印值，我们可以告诉 Rust 使用 `--show-output` 标志来显示成功测试的输出：

```console
$ cargo test -- --show-output
```

当我们再次使用 `--show-output` 标志运行 Listing 11-10 中的测试时，我们会看到以下输出：

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### 通过名称运行测试子集

有时，运行完整的测试套件可能需要很长时间。如果你正在处理特定区域的代码，你可能只想运行与该代码相关的测试。你可以通过将你想要运行的测试的名称作为参数传递给 `cargo test` 来选择要运行的测试。

为了演示如何运行测试子集，我们首先为 `add_two` 函数创建三个测试，如 Listing 11-11 所示，并选择要运行的测试。

<Listing number="11-11" file-name="src/lib.rs" caption="Three tests with three different names">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

如果我们不传递任何参数运行测试，正如我们之前看到的，所有测试都会并行运行：

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### 运行单个测试

我们可以将任何测试函数的名称传递给 `cargo test` 以仅运行该测试：

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

只有名为 `one_hundred` 的测试运行了；其他两个测试不匹配该名称。测试输出通过在末尾显示 `2 filtered out` 让我们知道还有更多测试没有运行。

我们不能以这种方式指定多个测试的名称；只有传递给 `cargo test` 的第一个值会被使用。但是有一种方法可以运行多个测试。

#### 过滤运行多个测试

我们可以指定测试名称的一部分，任何名称匹配该值的测试都将运行。例如，因为我们的两个测试名称包含 `add`，我们可以通过运行 `cargo test add` 来运行这两个测试：

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

这个命令运行了所有名称中包含 `add` 的测试，并过滤掉了名为 `one_hundred` 的测试。还要注意，测试所在的模块也成为测试名称的一部分，因此我们可以通过过滤模块名称来运行模块中的所有测试。

### 忽略某些测试除非特别请求

有时，一些特定的测试可能非常耗时，因此你可能希望在大多数 `cargo test` 运行中排除它们。与其列出所有你想要运行的测试作为参数，你可以使用 `ignore` 属性来注释这些耗时的测试以排除它们，如下所示：

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

在 `#[test]` 之后，我们为要排除的测试添加 `#[ignore]` 行。现在当我们运行测试时，`it_works` 会运行，但 `expensive_test` 不会：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

`expensive_test` 函数被列为 `ignored`。如果我们只想运行被忽略的测试，我们可以使用 `cargo test -- --ignored`：

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

通过控制哪些测试运行，你可以确保 `cargo test` 的结果会快速返回。当你处于一个检查 `ignored` 测试结果有意义并且有时间等待结果的时候，你可以运行 `cargo test -- --ignored`。如果你想运行所有测试，无论它们是否被忽略，你可以运行 `cargo test -- --include-ignored`。
## 使用 `panic!` 处理不可恢复的错误

有时候，代码中会发生一些糟糕的事情，而你对此无能为力。在这种情况下，Rust 提供了 `panic!` 宏。实际上有两种方式可以引发 panic：通过执行某些操作导致代码 panic（例如访问数组超出其范围）或显式调用 `panic!` 宏。在这两种情况下，我们都会导致程序 panic。默认情况下，这些 panic 会打印失败消息，展开堆栈，清理堆栈并退出。通过环境变量，你还可以让 Rust 在 panic 发生时显示调用堆栈，以便更容易追踪 panic 的来源。

> ### 展开堆栈或在 panic 时中止
>
> 默认情况下，当 panic 发生时，程序会开始**展开**堆栈，这意味着 Rust 会回溯堆栈并清理它遇到的每个函数中的数据。然而，回溯和清理是一项繁重的工作。因此，Rust 允许你选择立即**中止**，这将结束程序而不进行清理。
>
> 程序使用的内存随后需要由操作系统清理。如果你的项目需要使生成的二进制文件尽可能小，你可以通过在 _Cargo.toml_ 文件的适当 `[profile]` 部分添加 `panic = 'abort'` 来将 panic 时的行为从展开切换为中止。例如，如果你希望在发布模式下 panic 时中止，可以添加以下内容：
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

让我们尝试在一个简单的程序中调用 `panic!`：

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

当你运行这个程序时，你会看到类似以下的输出：

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

调用 `panic!` 会导致包含在最后两行中的错误消息。第一行显示了我们的 panic 消息以及 panic 发生的源代码位置：_src/main.rs:2:5_ 表示它是 _src/main.rs_ 文件的第二行第五个字符。

在这个例子中，指示的行是我们代码的一部分，如果我们查看该行，我们会看到 `panic!` 宏的调用。在其他情况下，`panic!` 调用可能位于我们代码调用的代码中，错误消息报告的文件名和行号将是调用 `panic!` 宏的代码，而不是最终导致 `panic!` 调用的我们的代码行。

<!-- 旧标题。不要删除，否则链接可能会失效。 -->

<a id="using-a-panic-backtrace"></a>

我们可以使用 `panic!` 调用来源的函数回溯来找出导致问题的代码部分。为了理解如何使用 `panic!` 回溯，让我们看另一个例子，看看当 `panic!` 调用来自库时是什么样子，而不是我们的代码直接调用宏。Listing 9-1 中的代码尝试访问向量中超出有效索引范围的索引。

<Listing number="9-1" file-name="src/main.rs" caption="尝试访问向量末尾之外的元素，这将导致 `panic!` 调用">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

在这里，我们尝试访问向量的第 100 个元素（索引为 99，因为索引从零开始），但向量只有三个元素。在这种情况下，Rust 会 panic。使用 `[]` 应该返回一个元素，但如果你传递了一个无效的索引，Rust 在这里无法返回一个正确的元素。

在 C 语言中，尝试读取数据结构末尾之外的内容是未定义行为。你可能会得到内存中对应于该数据结构元素的位置上的任何内容，即使该内存不属于该结构。这被称为**缓冲区越界读取**，如果攻击者能够操纵索引以读取他们不应该被允许读取的数据结构之后存储的数据，可能会导致安全漏洞。

为了保护你的程序免受此类漏洞的影响，如果你尝试读取一个不存在的索引处的元素，Rust 会停止执行并拒绝继续。让我们试试看：

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

这个错误指向我们的 _main.rs_ 文件的第 4 行，我们尝试访问向量 `v` 的索引 `99`。

`note:` 行告诉我们，我们可以设置 `RUST_BACKTRACE` 环境变量来获取导致错误的确切回溯。**回溯**是到达这一点所调用的所有函数的列表。Rust 中的回溯与其他语言中的回溯一样：阅读回溯的关键是从顶部开始阅读，直到看到你编写的文件。那就是问题起源的地方。该点之前的行是你的代码调用的代码；该点之后的行是调用你的代码的代码。这些前后行可能包括 Rust 核心代码、标准库代码或你正在使用的 crate。让我们尝试通过将 `RUST_BACKTRACE` 环境变量设置为除 `0` 之外的任何值来获取回溯。Listing 9-2 显示了类似于你将看到的输出。

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="当设置环境变量 `RUST_BACKTRACE` 时，由 `panic!` 调用生成的回溯">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

输出很多！你看到的输出可能会因操作系统和 Rust 版本而有所不同。为了获取包含这些信息的回溯，必须启用调试符号。在使用 `cargo build` 或 `cargo run` 而不带 `--release` 标志时，调试符号默认是启用的，就像我们这里的情况一样。

在 Listing 9-2 的输出中，回溯的第 6 行指向我们项目中导致问题的行：_src/main.rs_ 的第 4 行。如果我们不希望程序 panic，我们应该从第一个提到我们编写的文件的行指向的位置开始调查。在 Listing 9-1 中，我们故意编写了会导致 panic 的代码，修复 panic 的方法是不要请求超出向量索引范围的元素。当你的代码在未来 panic 时，你需要弄清楚代码正在使用哪些值执行什么操作导致了 panic，以及代码应该做什么。

我们将在本章后面的 [“To `panic!` or Not to `panic!`”][to-panic-or-not-to-panic]<!-- ignore --> 部分再次讨论 `panic!` 以及何时应该和不应该使用 `panic!` 来处理错误条件。接下来，我们将看看如何使用 `Result` 从错误中恢复。

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
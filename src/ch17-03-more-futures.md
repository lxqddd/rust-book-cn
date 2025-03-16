## 处理任意数量的 Futures

在前一节中，当我们从使用两个 futures 切换到使用三个 futures 时，我们也必须从使用 `join` 切换到使用 `join3`。每次我们改变要连接的 futures 数量时，都必须调用不同的函数，这可能会很烦人。幸运的是，我们有一个宏形式的 `join`，可以向它传递任意数量的参数。它还会自己处理等待 futures 的过程。因此，我们可以重写代码，使用 `join!` 而不是 `join3`，如 Listing 17-14 所示。

<Listing number="17-14" caption="使用 `join!` 等待多个 futures" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:here}}
```

</Listing>

这绝对比在 `join`、`join3`、`join4` 等之间切换要好得多！然而，即使这种宏形式也仅在我们提前知道 futures 的数量时才有效。在现实世界的 Rust 中，将 futures 推入集合中，然后等待其中一些或所有 futures 完成是一种常见的模式。

要检查集合中的所有 futures，我们需要遍历并连接 _所有_ 的 futures。`trpl::join_all` 函数接受任何实现了 `Iterator` trait 的类型，你在 [The Iterator Trait and the `next` Method][iterator-trait]<!-- ignore --> 第 13 章中已经学过，所以它似乎正是我们需要的。让我们尝试将我们的 futures 放入一个向量中，并用 `join_all` 替换 `join!`，如 Listing 17-15 所示。

<Listing number="17-15" caption="将匿名 futures 存储在向量中并调用 `join_all`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:here}}
```

</Listing>

不幸的是，这段代码无法编译。相反，我们得到了以下错误：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo build
copy just the compiler error
-->

```text
error[E0308]: mismatched types
  --> src/main.rs:45:37
   |
10 |         let tx1_fut = async move {
   |                       ---------- the expected `async` block
...
24 |         let rx_fut = async {
   |                      ----- the found `async` block
...
45 |         let futures = vec![tx1_fut, rx_fut, tx_fut];
   |                                     ^^^^^^ expected `async` block, found a different `async` block
   |
   = note: expected `async` block `{async block@src/main.rs:10:23: 10:33}`
              found `async` block `{async block@src/main.rs:24:22: 24:27}`
   = note: no two async blocks, even if identical, have the same type
   = help: consider pinning your async block and casting it to a trait object
```

这可能会让人感到惊讶。毕竟，这些 async 块都没有返回任何内容，所以每个块都会生成一个 `Future<Output = ()>`。请记住，`Future` 是一个 trait，编译器会为每个 async 块创建一个唯一的枚举。你不能将两个不同的手写结构体放入 `Vec` 中，同样的规则也适用于编译器生成的不同枚举。

为了使这项工作正常进行，我们需要使用 _trait 对象_，就像我们在第 12 章的 [“Returning Errors from the run function”][dyn]<!-- ignore --> 中所做的那样。（我们将在第 18 章详细讨论 trait 对象。）使用 trait 对象可以让我们将这些类型生成的匿名 futures 视为相同类型，因为它们都实现了 `Future` trait。

> 注意：在第 8 章的 [Using an Enum to Store Multiple Values][enum-alt]<!-- ignore --> 部分中，我们讨论了另一种在 `Vec` 中包含多种类型的方法：使用枚举来表示向量中可能出现的每种类型。不过，我们不能在这里这样做。一方面，我们无法命名这些不同的类型，因为它们是匿名的。另一方面，我们首先使用向量和 `join_all` 的原因是为了能够处理一个动态的 futures 集合，我们只关心它们具有相同的输出类型。

我们首先将 `vec!` 中的每个 future 包装在 `Box::new` 中，如 Listing 17-16 所示。

<Listing number="17-16" caption="使用 `Box::new` 对齐 `Vec` 中 futures 的类型" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

不幸的是，这段代码仍然无法编译。事实上，我们在第二个和第三个 `Box::new` 调用中得到了与之前相同的基本错误，以及一些涉及 `Unpin` trait 的新错误。我们稍后再讨论 `Unpin` 错误。首先，让我们通过显式注释 `futures` 变量的类型来修复 `Box::new` 调用中的类型错误（见 Listing 17-17）。

<Listing number="17-17" caption="通过使用显式类型声明修复其余的类型不匹配错误" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:here}}
```

</Listing>

这个类型声明有点复杂，所以让我们逐步分析一下：

1. 最内层的类型是 future 本身。我们通过写 `Future<Output = ()>` 显式地指出 future 的输出是单元类型 `()`。
2. 然后我们用 `dyn` 注释 trait，将其标记为动态的。
3. 整个 trait 引用被包装在一个 `Box` 中。
4. 最后，我们显式声明 `futures` 是一个包含这些项的 `Vec`。

这已经带来了很大的不同。现在当我们运行编译器时，我们只得到了提到 `Unpin` 的错误。尽管有三个错误，但它们的内容非常相似。

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-17
cargo build
# copy *only* the errors
# fix the paths
-->

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
   --> src/main.rs:49:24
    |
49  |         trpl::join_all(futures).await;
    |         -------------- ^^^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
    |         |
    |         required by a bound introduced by this call
    |
    = note: consider using the `pin!` macro
            consider using `Box::pin` if you need to access the pinned value outside of the current scope
    = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `join_all`
   --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:105:14
    |
102 | pub fn join_all<I>(iter: I) -> JoinAll<I::Item>
    |        -------- required by a bound in this function
...
105 |     I::Item: Future,
    |              ^^^^^^ required by this bound in `join_all`

error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:49:9
   |
49 |         trpl::join_all(futures).await;
   |         ^^^^^^^^^^^^^^^^^^^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`

error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:49:33
   |
49 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `async_await` (bin "async_await") due to 3 previous errors
```

这有很多内容需要消化，所以让我们分解一下。消息的第一部分告诉我们，第一个 async 块（`src/main.rs:8:23: 20:10`）没有实现 `Unpin` trait，并建议使用 `pin!` 或 `Box::pin` 来解决它。在本章的后面，我们将深入探讨一些关于 `Pin` 和 `Unpin` 的更多细节。不过，现在我们可以按照编译器的建议来解决问题。在 Listing 17-18 中，我们首先更新 `futures` 的类型注释，用 `Pin` 包装每个 `Box`。其次，我们使用 `Box::pin` 来固定 futures 本身。

<Listing number="17-18" caption="使用 `Pin` 和 `Box::pin` 使 `Vec` 类型检查通过" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

如果我们编译并运行这段代码，最终会得到我们期望的输出：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'messages'
received 'the'
received 'for'
received 'future'
received 'you'
```

终于成功了！

这里还有一些值得探索的地方。一方面，使用 `Pin<Box<T>>` 会带来一些开销，因为我们将这些 futures 放在堆上使用 `Box`——而我们这样做只是为了对齐类型。毕竟，我们实际上并不 _需要_ 堆分配：这些 futures 是特定于这个函数的。如前所述，`Pin` 本身是一个包装类型，因此我们可以在 `Vec` 中获得单一类型的好处——这是我们最初使用 `Box` 的原因——而不需要进行堆分配。我们可以直接使用 `Pin` 与每个 future，使用 `std::pin::pin` 宏。

然而，我们仍然必须显式地声明固定引用的类型；否则，Rust 仍然不知道如何将这些解释为动态 trait 对象，而这正是我们在 `Vec` 中需要的。因此，我们在定义每个 future 时使用 `pin!`，并将 `futures` 定义为一个包含固定可变引用的动态 future 类型的 `Vec`，如 Listing 17-19 所示。

<Listing number="17-19" caption="直接使用 `Pin` 和 `pin!` 宏以避免不必要的堆分配" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:here}}
```

</Listing>

我们通过忽略我们可能有不同的 `Output` 类型的事实走到了这一步。例如，在 Listing 17-20 中，`a` 的匿名 future 实现了 `Future<Output = u32>`，`b` 的匿名 future 实现了 `Future<Output = &str>`，`c` 的匿名 future 实现了 `Future<Output = bool>`。

<Listing number="17-20" caption="三个具有不同类型的 futures" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:here}}
```

</Listing>

我们可以使用 `trpl::join!` 来等待它们，因为它允许我们传入多个 future 类型并生成这些类型的元组。我们 _不能_ 使用 `trpl::join_all`，因为它要求传入的所有 futures 具有相同的类型。记住，这个错误正是我们开始这段 `Pin` 冒险的原因！

这是一个根本性的权衡：我们可以使用 `join_all` 处理动态数量的 futures，只要它们都具有相同的类型，或者我们可以使用 `join` 函数或 `join!` 宏处理固定数量的 futures，即使它们具有不同的类型。这与我们在 Rust 中处理任何其他类型时面临的情况相同。Futures 并不特殊，尽管我们有一些很好的语法来处理它们，这是一件好事。

### 竞速 Futures

当我们使用 `join` 系列函数和宏“连接”futures 时，我们要求 _所有_ 的 futures 都完成后才能继续。然而，有时我们只需要 _一些_ futures 完成就可以继续——类似于将一个 future 与另一个 future 竞速。

在 Listing 17-21 中，我们再次使用 `trpl::race` 来运行两个 futures，`slow` 和 `fast`，相互竞速。

<Listing number="17-21" caption="使用 `race` 获取最先完成的 future 的结果" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:here}}
```

</Listing>

每个 future 在开始运行时打印一条消息，通过调用并等待 `sleep` 暂停一段时间，然后在完成时打印另一条消息。然后我们将 `slow` 和 `fast` 传递给 `trpl::race` 并等待其中一个完成。（这里的结果并不太令人惊讶：`fast` 赢了。）与我们在 [“Our First Async Program”][async-program]<!-- ignore --> 中使用 `race` 时不同，我们在这里忽略了它返回的 `Either` 实例，因为所有有趣的行为都发生在 async 块的主体中。

请注意，如果你翻转 `race` 参数的顺序，“started”消息的顺序会改变，即使 `fast` future 总是先完成。这是因为这个特定的 `race` 函数的实现是不公平的。它总是按照传递参数的顺序运行 futures。其他实现 _是_ 公平的，会随机选择先轮询哪个 future。无论我们使用的 race 实现是否公平，_一个_ future 都会在其主体中的第一个 `await` 点之前运行。

回想一下 [Our First Async Program][async-program]<!-- ignore -->，在每个 await 点，Rust 都会给运行时一个机会暂停任务并切换到另一个任务，如果被等待的 future 还没有准备好。反之亦然：Rust _只_ 在 await 点暂停 async 块并将控制权交还给运行时。await 点之间的所有内容都是同步的。

这意味着如果你在 async 块中做了一堆工作而没有 await 点，那么这个 future 将阻止任何其他 futures 取得进展。你有时可能会听到这种情况被称为一个 future _饿死_ 其他 futures。在某些情况下，这可能不是什么大问题。然而，如果你正在进行某种昂贵的设置或长时间运行的工作，或者如果你有一个 future 将无限期地继续执行某个特定任务，你需要考虑何时何地将控制权交还给运行时。

同样地，如果你有长时间运行的阻塞操作，async 可以是一个有用的工具，用于提供程序不同部分之间相互关联的方式。

但是 _如何_ 在这些情况下将控制权交还给运行时呢？

<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### 将控制权交还给运行时

让我们模拟一个长时间运行的操作。Listing 17-22 引入了一个 `slow` 函数。

<Listing number="17-22" caption="使用 `thread::sleep` 模拟慢操作" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:slow}}
```

</Listing>

这段代码使用 `std::thread::sleep` 而不是 `trpl::sleep`，因此调用 `slow` 会阻塞当前线程一段时间。我们可以使用 `slow` 来模拟现实世界中既长时间运行又阻塞的操作。

在 Listing 17-23 中，我们使用 `slow` 来模拟在一对 futures 中执行这种 CPU 密集型工作。

<Listing number="17-23" caption="使用 `thread::sleep` 模拟慢操作" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:slow-futures}}
```

</Listing>

开始时，每个 future 只有在执行了一堆慢操作后才会将控制权交还给运行时。如果你运行这段代码，你会看到以下输出：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-23/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

与我们之前的示例一样，`race` 仍然在 `a` 完成后立即完成。不过，这两个 futures 之间没有交错。`a` future 在 `trpl::sleep` 调用被等待之前完成了所有工作，然后 `b` future 在它自己的 `trpl::sleep` 调用被等待之前完成了所有工作，最后 `a` future 完成。为了让两个 futures 在它们的慢任务之间取得进展，我们需要 await 点，以便我们可以将控制权交还给运行时。这意味着我们需要一些可以 await 的东西！

我们已经在 Listing 17-23 中看到了这种控制权移交的发生：如果我们移除 `a` future 末尾的 `trpl::sleep`，它将在 `b` future 运行 _之前_ 完成。让我们尝试使用 `sleep` 函数作为让操作交替取得进展的起点，如 Listing 17-24 所示。

<Listing number="17-24" caption="使用 `sleep` 让操作交替取得进展" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

在 Listing 17-24 中，我们在每次调用 `slow` 之间添加了 `trpl::sleep` 调用和 await 点。现在两个 futures 的工作是交替进行的：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-24
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

`a` future 仍然在将控制权交给 `b` 之前运行了一段时间，因为它在调用 `trpl::sleep` 之前调用了 `slow`，但之后 futures 每次遇到 await 点时都会交替进行。在这种情况下，我们在每次调用 `slow` 之后都这样做，但我们可以以对我们最有意义的方式分解工作。

不过，我们并不真的想在这里 _睡眠_：我们想尽可能快地取得进展。我们只需要将控制权交还给运行时。我们可以直接使用 `yield_now` 函数来实现这一点。在 Listing 17-25 中，我们将所有这些 `sleep` 调用替换为 `yield_now`。

<Listing number="17-25" caption="使用 `yield_now` 让操作交替取得进展" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-25/src/main.rs:yields}}
```

</Listing>

这段代码既更清晰地表达了实际意图，又比使用 `sleep` 快得多，因为像 `sleep` 使用的计时器通常对它们的粒度有限制。例如，我们使用的 `sleep` 版本即使我们传递给它一个纳秒的 `Duration`，也总是会至少睡眠一毫秒。再次强调，现代计算机 _非常快_：它们在一毫秒内可以做很多事情！

你可以通过设置一个小型基准测试来亲自看到这一点，例如 Listing 17-26 中的基准测试。（这不是一个特别严格的性能测试方法，但它足以显示这里的差异。）

<Listing number="17-26" caption="比较 `sleep` 和 `yield_now` 的性能" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-26/src/main.rs:here}}
```

</Listing>

在这里，我们跳过了所有的状态打印，传递了一个一纳秒的 `Duration` 给 `trpl::sleep`，并让每个 future 自己运行，没有在 futures 之间切换。然后我们运行 1,000 次迭代，看看使用 `trpl::sleep` 的 future 与使用 `trpl::yield_now` 的 future 相比需要多长时间。

使用 `yield_now` 的版本 _快得多_！

这意味着 async 即使对于计算密集型任务也可能有用，具体取决于你的程序在做什么，因为它为构建程序不同部分之间的关系提供了一个有用的工具。这是一种 _协作式多任务处理_，其中每个 future 都有权通过 await 点决定何时移交控制权。因此，每个 future 也有责任避免阻塞太长时间。在一些基于 Rust 的嵌入式操作系统中，这是 _唯一_ 的多任务处理方式！

在现实世界的代码中，你通常不会在每一行上都交替函数调用和 await 点。虽然以这种方式移交控制权相对便宜，但它并不是免费的。在许多情况下，尝试分解计算密集型任务可能会使其显著变慢，因此有时为了 _整体_ 性能，最好让操作短暂阻塞。始终测量以查看代码的实际性能瓶颈是什么。不过，如果你 _确实_ 看到大量工作以串行方式发生，而你期望它们并发发生，那么记住这种底层动态是很重要的！

### 构建我们自己的 Async 抽象

我们还可以将 futures 组合在一起以创建新的模式。例如，我们可以使用我们已经拥有的 async 构建块来构建一个 `timeout` 函数。当我们完成后，结果将是另一个构建块，我们可以用它来创建更多的 async 抽象。

Listing 17-27 展示了我们期望这个 `timeout` 如何与一个慢 future 一起工作。

<Listing number="17-27" caption="使用我们想象的 `timeout` 来运行一个有时间限制的慢操作" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-27/src/main.rs:here}}
```

</Listing>

让我们实现这个功能！首先，让我们考虑一下 `timeout` 的 API：

- 它本身需要是一个 async 函数，以便我们可以 await 它。
- 它的第一个参数应该是一个要运行的 future。我们可以将其设为泛型，以允许它与任何 future 一起工作。
- 它的第二个参数将是最大等待时间。如果我们使用 `Duration`，这将使其易于传递给 `trpl::sleep`。
- 它应该返回一个 `Result`。如果 future 成功完成，`Result` 将是 `Ok`，包含 future 生成的值。如果超时先发生，`Result` 将是 `Err`，包含超时等待的时间。

Listing 17-28 展示了这个声明。

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-28" caption="定义 `timeout` 的签名" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-28/src/main.rs:declaration}}
```

</Listing>

这满足了我们对类型的目标。现在让我们考虑一下我们需要的 _行为_：我们希望将传入的 future 与持续时间竞速。我们可以使用 `trpl::sleep` 从持续时间中创建一个计时器 future，并使用 `trpl::race` 来运行这个计时器与调用者传递的 future。

我们还知道 `race` 是不公平的，按照传递参数的顺序轮询参数。因此，我们首先将 `future_to_try` 传递给 `race`，以便即使 `max_time` 是一个非常短的持续时间，它也有机会完成。如果 `future_to_try` 先完成，`race` 将返回 `Left`，包含 `future_to_try` 的输出。如果 `timer` 先完成，`race` 将返回 `Right`，包含计时器的输出 `()`。

在 Listing 17-29 中，我们匹配了 `trpl::race` 的 await 结果。

<Listing number="17-29" caption="使用 `race` 和 `sleep` 定义 `timeout`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-29/src/main.rs:implementation}}
```

</Listing>

如果 `future_to_try` 成功并且我们得到了 `Left(output)`，我们返回 `Ok(output)`。如果 sleep 计时器先超时并且我们得到了 `Right(())`，我们用 `_` 忽略 `()` 并返回 `Err(max_time)`。

有了这个，我们就有了一个由两个其他 async 助手构建的工作 `timeout`。如果我们运行我们的代码，它将在超时后打印失败模式：

```text
Failed after 2 seconds
```

因为 futures 可以与其他 futures 组合，你可以使用较小的 async 构建块构建非常强大的工具。例如，你可以使用相同的方法将超时与重试结合起来，然后将其与网络调用等操作一起使用（本章开头的示例之一）。

在实践中，你通常会直接使用 `async` 和 `await`，其次使用诸如 `join`、`join_all`、`race` 等函数和宏。你只需要偶尔使用 `pin` 来与这些 API 一起使用 futures。

我们现在已经看到了多种同时处理多个 futures 的方法。接下来，我们将看看如何通过 _streams_ 在一段时间内按顺序处理多个 futures。不过，这里还有一些你可能想先考虑的事情：

- 我们使用 `Vec` 和 `join_all` 来等待一组 futures 中的所有 futures 完成。你如何使用 `Vec` 来按顺序处理一组 futures？这样做的权衡是什么？

- 看看 `futures` crate 中的 `futures::stream::FuturesUnordered` 类型。使用它与使用 `Vec` 有什么不同？（不用担心它来自 crate 的 `stream` 部分；它与任何 futures 集合一起工作得很好。）

[dyn]: ch12-03-improving-error-handling-and-modularity.html
[enum-alt]: ch12-03-improving-error-handling-and-modularity.html#returning-errors-from-the-run-function
[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
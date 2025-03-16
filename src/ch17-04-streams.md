## Streams: 顺序中的 Futures

<!-- 旧标题。不要删除，否则链接可能会失效。 -->

<a id="streams"></a>

到目前为止，本章我们主要讨论的是单个的 future。唯一的例外是我们使用的异步通道。回想一下我们在本章的[“消息传递”][17-02-messages]<!-- ignore -->部分中如何使用异步通道的接收器。异步的 `recv` 方法会随着时间的推移产生一系列的项目。这是一个更通用的模式，称为 _stream_（流）。

我们在第 13 章中看到了一个项目序列，当时我们研究了 `Iterator` trait 在[The Iterator Trait and the `next` Method][iterator-trait]<!-- ignore -->部分中的内容，但迭代器和异步通道接收器之间有两个区别。第一个区别是时间：迭代器是同步的，而通道接收器是异步的。第二个区别是 API。当直接使用 `Iterator` 时，我们调用它的同步 `next` 方法。而对于 `trpl::Receiver` 流，我们调用的是异步的 `recv` 方法。除此之外，这些 API 感觉非常相似，而这种相似性并非巧合。流就像是异步形式的迭代。虽然 `trpl::Receiver` 专门等待接收消息，但通用的流 API 要广泛得多：它提供了像 `Iterator` 一样的下一个项目，但是是异步的。

Rust 中迭代器和流之间的相似性意味着我们实际上可以从任何迭代器创建一个流。与迭代器一样，我们可以通过调用流的 `next` 方法并等待输出来处理流，如 Listing 17-30 所示。

<Listing number="17-30" caption="从迭代器创建流并打印其值" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-30/src/main.rs:stream}}
```

</Listing>

我们从一个数字数组开始，将其转换为迭代器，然后调用 `map` 方法将所有值加倍。然后我们使用 `trpl::stream_from_iter` 函数将迭代器转换为流。接下来，我们使用 `while let` 循环遍历流中的项目。

不幸的是，当我们尝试运行代码时，它无法编译，而是报告没有可用的 `next` 方法：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-30
cargo build
copy only the error output
-->

```console
error[E0599]: no method named `next` found for struct `Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = note: the full type name has been written to 'file:///projects/async-await/target/debug/deps/async_await-575db3dd3197d257.long-type-14490787947592691573.txt'
   = note: consider using `--verbose` to print the full type name to the console
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

正如输出所解释的那样，编译器错误的原因是我们需要在作用域中有正确的 trait 才能使用 `next` 方法。根据我们目前的讨论，你可能会合理地期望这个 trait 是 `Stream`，但它实际上是 `StreamExt`。`Ext` 是 _extension_ 的缩写，是 Rust 社区中用于扩展一个 trait 的常见模式。

我们将在本章末尾更详细地解释 `Stream` 和 `StreamExt` trait，但现在你只需要知道 `Stream` trait 定义了一个低级别的接口，有效地结合了 `Iterator` 和 `Future` trait。`StreamExt` 在 `Stream` 之上提供了一组更高级别的 API，包括 `next` 方法以及其他类似于 `Iterator` trait 提供的实用方法。`Stream` 和 `StreamExt` 尚未成为 Rust 标准库的一部分，但大多数生态系统 crate 都使用相同的定义。

修复编译器错误的方法是添加一个 `use` 语句来引入 `trpl::StreamExt`，如 Listing 17-31 所示。

<Listing number="17-31" caption="成功使用迭代器作为流的基础" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-31/src/main.rs:all}}
```

</Listing>

将这些部分组合在一起后，这段代码按我们想要的方式工作！更重要的是，现在我们有了 `StreamExt` 在作用域中，我们可以使用它的所有实用方法，就像使用迭代器一样。例如，在 Listing 17-32 中，我们使用 `filter` 方法过滤掉所有不是三或五的倍数的项目。

<Listing number="17-32" caption="使用 `StreamExt::filter` 方法过滤流" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-32/src/main.rs:all}}
```

</Listing>

当然，这并不是很有趣，因为我们可以使用普通的迭代器来完成同样的操作，而不需要任何异步操作。让我们看看流独有的功能。

### 组合流

许多概念自然表示为流：队列中可用的项目、从文件系统中逐步拉取的数据块（当完整数据集太大而无法放入计算机内存时），或随着时间的推移通过网络到达的数据。因为流是 future，我们可以将它们与任何其他类型的 future 结合使用，并以有趣的方式组合它们。例如，我们可以批量处理事件以避免触发过多的网络调用，为长时间运行的操作序列设置超时，或限制用户界面事件以避免不必要的工作。

让我们首先构建一个小消息流，作为我们从 WebSocket 或其他实时通信协议中可能看到的数据流的替代品，如 Listing 17-33 所示。

<Listing number="17-33" caption="使用 `rx` 接收器作为 `ReceiverStream`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-33/src/main.rs:all}}
```

</Listing>

首先，我们创建一个名为 `get_messages` 的函数，返回 `impl Stream<Item = String>`。在实现中，我们创建一个异步通道，循环遍历英文字母表的前 10 个字母，并将它们发送到通道中。

我们还使用了一个新类型：`ReceiverStream`，它将 `trpl::channel` 的 `rx` 接收器转换为具有 `next` 方法的 `Stream`。回到 `main` 函数中，我们使用 `while let` 循环打印流中的所有消息。

当我们运行这段代码时，我们得到了预期的结果：

<!-- 不提取输出，因为输出的变化不显著；
这些变化可能是由于线程运行方式不同，而不是编译器的变化 -->

```text
Message: 'a'
Message: 'b'
Message: 'c'
Message: 'd'
Message: 'e'
Message: 'f'
Message: 'g'
Message: 'h'
Message: 'i'
Message: 'j'
```

同样，我们可以使用常规的 `Receiver` API 甚至常规的 `Iterator` API 来完成此操作，所以让我们添加一个需要流的功能：为流中的每个项目添加超时，并对我们发出的项目添加延迟，如 Listing 17-34 所示。

<Listing number="17-34" caption="使用 `StreamExt::timeout` 方法为流中的项目设置时间限制" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-34/src/main.rs:timeout}}
```

</Listing>

我们首先使用 `timeout` 方法为流添加超时，该方法来自 `StreamExt` trait。然后我们更新 `while let` 循环的主体，因为流现在返回一个 `Result`。`Ok` 变体表示消息按时到达；`Err` 变体表示在消息到达之前超时已过。我们对该结果进行 `match` 匹配，并在成功接收到消息时打印消息，或在超时时打印通知。最后，请注意我们在应用超时后将消息固定，因为超时助手生成的流需要固定才能被轮询。

然而，由于消息之间没有延迟，这个超时不会改变程序的行为。让我们为发送的消息添加可变延迟，如 Listing 17-35 所示。

<Listing number="17-35" caption="通过 `tx` 发送带有异步延迟的消息，而不使 `get_messages` 成为异步函数" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-35/src/main.rs:messages}}
```

</Listing>

在 `get_messages` 中，我们使用 `enumerate` 迭代器方法与 `messages` 数组一起使用，以便我们可以获取每个项目的索引以及项目本身。然后我们对偶数索引的项目应用 100 毫秒的延迟，对奇数索引的项目应用 300 毫秒的延迟，以模拟我们在现实世界中可能看到的消息流的不同延迟。因为我们的超时是 200 毫秒，这应该会影响一半的消息。

为了在 `get_messages` 函数中在消息之间睡眠而不阻塞，我们需要使用异步。然而，我们不能将 `get_messages` 本身变成异步函数，因为那样我们会返回一个 `Future<Output = Stream<Item = String>>` 而不是 `Stream<Item = String>>`。调用者将不得不等待 `get_messages` 本身才能访问流。但请记住：在给定的 future 中，所有事情都是线性发生的；并发发生在 _future 之间_。等待 `get_messages` 将要求它在返回接收器流之前发送所有消息，包括每个消息之间的睡眠延迟。结果，超时将毫无用处。流本身不会有延迟；它们都会在流可用之前发生。

相反，我们将 `get_messages` 保留为返回流的常规函数，并生成一个任务来处理异步的 `sleep` 调用。

> 注意：以这种方式调用 `spawn_task` 是有效的，因为我们已经设置了运行时；如果我们没有设置，它将导致 panic。其他实现选择了不同的权衡：它们可能会生成一个新的运行时并避免 panic，但最终会有一些额外的开销，或者它们可能根本不提供独立生成任务的方式而不引用运行时。确保你知道你的运行时选择了什么权衡，并相应地编写代码！

现在我们的代码有了更有趣的结果。在每对消息之间，会出现一个 `Problem: Elapsed(())` 错误。

<!-- 不提取输出，因为输出的变化不显著；
这些变化可能是由于线程运行方式不同，而不是编译器的变化 -->

```text
Message: 'a'
Problem: Elapsed(())
Message: 'b'
Message: 'c'
Problem: Elapsed(())
Message: 'd'
Message: 'e'
Problem: Elapsed(())
Message: 'f'
Message: 'g'
Problem: Elapsed(())
Message: 'h'
Message: 'i'
Problem: Elapsed(())
Message: 'j'
```

超时并不会阻止消息最终到达。我们仍然会收到所有原始消息，因为我们的通道是 _无界的_：它可以容纳尽可能多的消息，只要内存允许。如果消息在超时之前没有到达，我们的流处理程序将处理这种情况，但当它再次轮询流时，消息可能已经到达。

如果需要，你可以通过使用其他类型的通道或其他类型的流来获得不同的行为。让我们通过将时间间隔流与消息流结合起来，看看其中的一个实际应用。

### 合并流

首先，让我们创建另一个流，如果我们直接运行它，它将每毫秒发出一个项目。为了简单起见，我们可以使用 `sleep` 函数在延迟后发送消息，并将其与我们在 `get_messages` 中使用的从通道创建流的方法结合起来。不同的是，这次我们将发送经过的间隔计数，因此返回类型将是 `impl Stream<Item = u32>`，我们可以将该函数称为 `get_intervals`（见 Listing 17-36）。

<Listing number="17-36" caption="创建一个流，其中包含一个计数器，每毫秒发出一次" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-36/src/main.rs:intervals}}
```

</Listing>

我们首先在任务中定义一个 `count`。（我们也可以在任务外部定义它，但限制任何给定变量的范围更清晰。）然后我们创建一个无限循环。循环的每次迭代都会异步睡眠一毫秒，增加计数，然后将其发送到通道中。因为这一切都包装在 `spawn_task` 创建的任务中，所以所有内容——包括无限循环——都会随着运行时一起被清理。

这种无限循环在异步 Rust 中相当常见：许多程序需要无限期地运行。使用异步，只要每次循环迭代中至少有一个 await 点，这不会阻塞任何其他内容。

现在，回到我们的主函数的异步块中，我们可以尝试合并 `messages` 和 `intervals` 流，如 Listing 17-37 所示。

<Listing number="17-37" caption="尝试合并 `messages` 和 `intervals` 流" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-37/src/main.rs:main}}
```

</Listing>

我们首先调用 `get_intervals`。然后我们使用 `merge` 方法合并 `messages` 和 `intervals` 流，该方法将多个流组合成一个流，只要源流中的项目可用，就会生成项目，而不强加任何特定的顺序。最后，我们循环遍历该组合流，而不是 `messages`。

此时，`messages` 和 `intervals` 都不需要被固定或可变，因为两者都将被组合到单个 `merged` 流中。然而，这个 `merge` 调用无法编译！（`while let` 循环中的 `next` 调用也无法编译，但我们会回到这个问题。）这是因为两个流具有不同的类型。`messages` 流的类型是 `Timeout<impl Stream<Item = String>>`，其中 `Timeout` 是为 `timeout` 调用实现 `Stream` 的类型。`intervals` 流的类型是 `impl Stream<Item = u32>`。要合并这两个流，我们需要将其中一个转换为与另一个匹配。我们将重新处理 `intervals` 流，因为 `messages` 已经是我们想要的基本格式，并且必须处理超时错误（见 Listing 17-38）。

<!-- 我们无法直接测试这个，因为它永远不会停止。 -->

<Listing number="17-38" caption="将 `intervals` 流的类型与 `messages` 流的类型对齐" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-38/src/main.rs:main}}
```

</Listing>

首先，我们可以使用 `map` 辅助方法将 `intervals` 转换为字符串。其次，我们需要匹配 `messages` 的 `Timeout`。因为我们实际上并不希望 `intervals` 有超时，所以我们可以创建一个比其他持续时间更长的超时。在这里，我们使用 `Duration::from_secs(10)` 创建一个 10 秒的超时。最后，我们需要使 `stream` 可变，以便 `while let` 循环的 `next` 调用可以遍历流，并将其固定以便安全地执行此操作。这使我们几乎达到了我们需要的位置。一切类型检查都通过了。如果你运行这个，会有两个问题。首先，它永远不会停止！你需要使用 <span class="keystroke">ctrl-c</span> 来停止它。其次，来自英文字母表的消息将淹没在所有间隔计数器消息中：

<!-- 不提取输出，因为输出的变化不显著；
这些变化可能是由于任务运行方式不同，而不是编译器的变化 -->

```text
--snip--
Interval: 38
Interval: 39
Interval: 40
Message: 'a'
Interval: 41
Interval: 42
Interval: 43
--snip--
```

Listing 17-39 展示了一种解决最后两个问题的方法。

<Listing number="17-39" caption="使用 `throttle` 和 `take` 来管理合并的流" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-39/src/main.rs:throttle}}
```

</Listing>

首先，我们在 `intervals` 流上使用 `throttle` 方法，以免它压倒 `messages` 流。_Throttling_ 是一种限制函数调用速率的方式——或者在这种情况下，限制流被轮询的频率。每 100 毫秒一次应该足够了，因为我们的消息大约每 100 毫秒到达一次。

为了限制我们从流中接受的项目数量，我们将 `take` 方法应用于 `merged` 流，因为我们希望限制最终输出，而不仅仅是其中一个流。

现在当我们运行程序时，它在从流中拉取 20 个项目后停止，并且间隔不会压倒消息。我们也不会得到 `Interval: 100` 或 `Interval: 200` 等等，而是得到 `Interval: 1`、`Interval: 2` 等等——即使我们有一个可以每毫秒生成一个事件的源流。这是因为 `throttle` 调用生成了一个包装原始流的新流，以便原始流仅在节流速率下被轮询，而不是其“原生”速率。我们没有一堆未处理的间隔消息被选择忽略。相反，我们一开始就没有生成那些间隔消息！这是 Rust 的 future 固有的“惰性”再次发挥作用，允许我们选择性能特征。

<!-- 不提取输出，因为输出的变化不显著；
这些变化可能是由于线程运行方式不同，而不是编译器的变化 -->

```text
Interval: 1
Message: 'a'
Interval: 2
Interval: 3
Problem: Elapsed(())
Interval: 4
Message: 'b'
Interval: 5
Message: 'c'
Interval: 6
Interval: 7
Problem: Elapsed(())
Interval: 8
Message: 'd'
Interval: 9
Message: 'e'
Interval: 10
Interval: 11
Problem: Elapsed(())
Interval: 12
```

我们还需要处理最后一件事：错误！对于这两个基于通道的流，当通道的另一端关闭时，`send` 调用可能会失败——这只是运行时执行组成流的 future 的方式。到目前为止，我们通过调用 `unwrap` 忽略了这种可能性，但在一个行为良好的应用程序中，我们应该显式地处理错误，至少通过结束循环以便我们不再尝试发送任何消息。Listing 17-40 展示了一个简单的错误策略：打印问题然后从循环中 `break`。

<Listing number="17-40" caption="处理错误并关闭循环">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-40/src/main.rs:errors}}
```

</Listing>

通常，处理消息发送错误的正确方式会有所不同；只需确保你有一个策略。

现在我们已经看到了很多异步的实际应用，让我们退一步，深入了解 Rust 如何使异步工作的 `Future`、`Stream` 和其他关键 trait 的一些细节。

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
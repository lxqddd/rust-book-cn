## 使用 Async 实现并发

<!-- 旧标题。不要删除，否则链接可能会失效。 -->

<a id="concurrency-with-async"></a>

在本节中，我们将把 async 应用到一些与第 16 章中使用线程解决的并发挑战相同的问题上。因为我们已经在那里讨论了很多关键概念，所以本节我们将重点放在线程和 futures 之间的区别上。

在许多情况下，使用 async 进行并发操作的 API 与使用线程的 API 非常相似。在其他情况下，它们最终会有很大的不同。即使线程和 async 的 API **看起来**相似，它们的行为通常也不同——而且它们的性能特征几乎总是不同。

<!-- 旧标题。不要删除，否则链接可能会失效。 -->

<a id="counting"></a>

### 使用 `spawn_task` 创建新任务

在 [使用 Spawn 创建新线程][thread-spawn]<!-- ignore --> 中，我们解决的第一个操作是在两个独立的线程上进行计数。让我们使用 async 来做同样的事情。`trpl` crate 提供了一个 `spawn_task` 函数，看起来与 `thread::spawn` API 非常相似，以及一个 `sleep` 函数，它是 `thread::sleep` API 的 async 版本。我们可以一起使用这些函数来实现计数示例，如 Listing 17-6 所示。

<Listing number="17-6" caption="创建一个新任务来打印一些内容，同时主任务打印其他内容" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

作为我们的起点，我们使用 `trpl::run` 设置 `main` 函数，以便我们的顶层函数可以是 async 的。

> 注意：从本章的这一点开始，每个示例都将包含与 `main` 中的 `trpl::run` 完全相同的包装代码，因此我们通常会像处理 `main` 一样跳过它。不要忘记在你的代码中包含它！

然后我们在该块中编写两个循环，每个循环包含一个 `trpl::sleep` 调用，它在发送下一条消息之前等待半秒（500 毫秒）。我们将一个循环放在 `trpl::spawn_task` 的主体中，另一个放在顶层的 `for` 循环中。我们还在 `sleep` 调用之后添加了一个 `await`。

这段代码的行为与基于线程的实现类似——包括当你运行它时，可能会在终端中看到消息以不同的顺序出现：

<!-- 不提取输出，因为输出的变化不显著；变化可能是由于线程运行方式不同，而不是编译器的变化 -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

这个版本在主 async 块中的 `for` 循环完成后立即停止，因为由 `spawn_task` 生成的任务在 `main` 函数结束时被关闭。如果你希望它一直运行到任务完成，你需要使用一个 join 句柄来等待第一个任务完成。对于线程，我们使用 `join` 方法来“阻塞”直到线程运行完毕。在 Listing 17-7 中，我们可以使用 `await` 来做同样的事情，因为任务句柄本身就是一个 future。它的 `Output` 类型是一个 `Result`，所以我们在等待它之后还要解包它。

<Listing number="17-7" caption="使用 `await` 与 join 句柄一起运行任务直到完成" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

这个更新后的版本会一直运行，直到**两个**循环都完成。

<!-- 不提取输出，因为输出的变化不显著；变化可能是由于线程运行方式不同，而不是编译器的变化 -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

到目前为止，看起来 async 和线程给了我们相同的基本结果，只是语法不同：使用 `await` 而不是在 join 句柄上调用 `join`，并且等待 `sleep` 调用。

更大的区别是我们不需要生成另一个操作系统线程来执行此操作。事实上，我们甚至不需要在这里生成任务。因为 async 块编译为匿名的 futures，我们可以将每个循环放在一个 async 块中，并使用 `trpl::join` 函数让运行时将它们都运行到完成。

在 [使用 `join` 句柄等待所有线程完成][join-handles]<!-- ignore --> 一节中，我们展示了如何在调用 `std::thread::spawn` 时返回的 `JoinHandle` 类型上使用 `join` 方法。`trpl::join` 函数类似，但用于 futures。当你给它两个 futures 时，它会生成一个新的 future，其输出是一个元组，包含你传入的每个 future 的输出，一旦它们**都**完成。因此，在 Listing 17-8 中，我们使用 `trpl::join` 来等待 `fut1` 和 `fut2` 完成。我们**不**等待 `fut1` 和 `fut2`，而是等待由 `trpl::join` 生成的新 future。我们忽略输出，因为它只是一个包含两个单元值的元组。

<Listing number="17-8" caption="使用 `trpl::join` 等待两个匿名 futures" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

当我们运行这个时，我们看到两个 futures 都运行到完成：

<!-- 不提取输出，因为输出的变化不显著；变化可能是由于线程运行方式不同，而不是编译器的变化 -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

现在，你每次都会看到完全相同的顺序，这与我们在线程中看到的情况非常不同。这是因为 `trpl::join` 函数是**公平的**，意味着它平等地检查每个 future，交替进行，并且如果一个 future 准备好了，它不会让另一个 future 抢先。对于线程，操作系统决定检查哪个线程以及让它运行多长时间。对于 async Rust，运行时决定检查哪个任务。（实际上，细节会变得复杂，因为 async 运行时可能在底层使用操作系统线程作为其管理并发的一部分，因此保证公平性对运行时来说可能是更多的工作——但它仍然是可能的！）运行时不必保证任何给定操作的公平性，它们通常提供不同的 API 来让你选择是否需要公平性。

尝试一些这些变体来等待 futures，看看它们会做什么：

- 移除一个或两个循环周围的 async 块。
- 在定义每个 async 块后立即等待它。
- 只将第一个循环包装在 async 块中，并在第二个循环的主体之后等待生成的 future。

作为一个额外的挑战，看看你是否能在运行代码之前**预测**每种情况下的输出！

<!-- 旧标题。不要删除，否则链接可能会失效。 -->

<a id="message-passing"></a>

### 使用消息传递在两个任务上进行计数

在 futures 之间共享数据也会很熟悉：我们将再次使用消息传递，但这次使用 async 版本的类型和函数。我们将采取与 [使用消息传递在线程之间传输数据][message-passing-threads]<!-- ignore --> 中略有不同的路径，以说明基于线程和基于 futures 的并发之间的一些关键区别。在 Listing 17-9 中，我们将从一个单一的 async 块开始——**不**像我们生成一个单独的线程那样生成一个单独的任务。

<Listing number="17-9" caption="创建一个 async 通道并将两端分配给 `tx` 和 `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

在这里，我们使用 `trpl::channel`，这是一个 async 版本的多生产者、单消费者通道 API，我们在第 16 章中与线程一起使用过。async 版本的 API 与基于线程的版本只有一点不同：它使用一个可变的而不是不可变的接收器 `rx`，并且它的 `recv` 方法生成一个我们需要等待的 future，而不是直接生成值。现在我们可以从发送者向接收者发送消息。注意，我们不需要生成一个单独的线程甚至任务；我们只需要等待 `rx.recv` 调用。

`std::mpsc::channel` 中的同步 `Receiver::recv` 方法会阻塞，直到它收到消息。`trpl::Receiver::recv` 方法不会阻塞，因为它是 async 的。它不会阻塞，而是将控制权交还给运行时，直到收到消息或通道的发送端关闭。相比之下，我们不会等待 `send` 调用，因为它不会阻塞。它不需要阻塞，因为我们发送到的通道是无界的。

> 注意：因为所有这些 async 代码都在 `trpl::run` 调用中的 async 块中运行，所以其中的所有内容都可以避免阻塞。然而，**外部**的代码会在 `run` 函数返回时阻塞。这就是 `trpl::run` 函数的全部意义：它让你**选择**在哪里阻塞某些 async 代码，从而在哪里在同步和异步代码之间进行转换。在大多数 async 运行时中，`run` 实际上被称为 `block_on`，正是因为这个原因。

注意这个例子中的两件事。首先，消息会立即到达。其次，尽管我们在这里使用了一个 future，但还没有并发。列表中的所有内容都是按顺序发生的，就像没有 futures 参与一样。

让我们通过发送一系列消息并在它们之间休眠来解决第一部分，如 Listing 17-10 所示。

<!-- 我们无法测试这个，因为它永远不会停止！ -->

<Listing number="17-10" caption="通过 async 通道发送和接收多条消息，并在每条消息之间使用 `await` 休眠" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

除了发送消息外，我们还需要接收它们。在这种情况下，因为我们知道有多少消息会进来，我们可以手动调用 `rx.recv().await` 四次。然而，在现实世界中，我们通常会等待一些**未知**数量的消息，所以我们需要一直等待，直到我们确定没有更多的消息。

在 Listing 16-10 中，我们使用了一个 `for` 循环来处理从同步通道接收的所有项目。Rust 还没有办法编写一个 `for` 循环来遍历一个**异步**系列的项目，所以我们需要使用一个我们之前没有见过的循环：`while let` 条件循环。这是我们在 [使用 `if let` 和 `let else` 进行简洁控制流][if-let]<!-- ignore --> 一节中看到的 `if let` 构造的循环版本。只要它指定的模式继续匹配值，循环就会继续执行。

`rx.recv` 调用生成一个 future，我们等待它。运行时将暂停 future，直到它准备好。一旦消息到达，future 将解析为 `Some(message)`，每次消息到达时都会这样。当通道关闭时，无论**是否**有任何消息到达，future 将解析为 `None`，表示没有更多的值，因此我们应该停止轮询——即停止等待。

`while let` 循环将所有这些结合在一起。如果调用 `rx.recv().await` 的结果是 `Some(message)`，我们可以访问消息并在循环体中使用它，就像我们可以使用 `if let` 一样。如果结果是 `None`，循环结束。每次循环完成时，它都会再次到达等待点，因此运行时再次暂停它，直到另一条消息到达。

代码现在成功地发送和接收了所有消息。不幸的是，仍然有几个问题。首先，消息不会以半秒的间隔到达。它们会在我们启动程序 2 秒（2000 毫秒）后一次性到达。其次，这个程序永远不会退出！相反，它会永远等待新消息。你需要使用 <span class="keystroke">ctrl-c</span> 来关闭它。

让我们首先检查为什么消息会在完整延迟后一次性到达，而不是在每条消息之间延迟。在给定的 async 块中，代码中 `await` 关键字出现的顺序也是程序运行时它们执行的顺序。

Listing 17-10 中只有一个 async 块，所以其中的所有内容都是线性运行的。仍然没有并发。所有的 `tx.send` 调用都会发生，穿插着所有的 `trpl::sleep` 调用及其相关的等待点。然后 `while let` 循环才会通过 `recv` 调用的任何等待点。

为了获得我们想要的行为，即每条消息之间的睡眠延迟，我们需要将 `tx` 和 `rx` 操作放在它们自己的 async 块中，如 Listing 17-11 所示。然后运行时可以使用 `trpl::join` 分别执行它们，就像在计数示例中一样。再次强调，我们等待调用 `trpl::join` 的结果，而不是单独的 futures。如果我们按顺序等待单独的 futures，我们最终会回到顺序流程——这正是我们**不**想做的事情。

<!-- 我们无法测试这个，因为它永远不会停止！ -->

<Listing number="17-11" caption="将 `send` 和 `recv` 分离到它们自己的 `async` 块中，并等待这些块的 futures" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

在 Listing 17-11 的更新代码中，消息以 500 毫秒的间隔打印，而不是在 2 秒后一次性到达。

然而，程序仍然不会退出，因为 `while let` 循环与 `trpl::join` 的交互方式：

- 从 `trpl::join` 返回的 future 只有在传递给它的**两个** futures 都完成后才会完成。
- `tx` future 在发送完 `vals` 中的最后一条消息并完成休眠后完成。
- `rx` future 只有在 `while let` 循环结束后才会完成。
- `while let` 循环只有在等待 `rx.recv` 产生 `None` 时才会结束。
- 等待 `rx.recv` 只有在通道的另一端关闭时才会返回 `None`。
- 通道只有在调用 `rx.close` 或发送端 `tx` 被丢弃时才会关闭。
- 我们没有在任何地方调用 `rx.close`，并且 `tx` 只有在传递给 `trpl::run` 的最外层 async 块结束时才会被丢弃。
- 该块无法结束，因为它被阻塞在 `trpl::join` 完成上，这让我们回到了这个列表的顶部。

我们可以通过调用 `rx.close` 来手动关闭 `rx`，但这没有多大意义。在处理一些任意数量的消息后停止会使程序关闭，但我们可能会错过消息。我们需要其他方法来确保 `tx` 在函数结束之前被丢弃。

现在，我们发送消息的 async 块只借用 `tx`，因为发送消息不需要所有权，但如果我们可以将 `tx` 移动到该 async 块中，它将在该块结束时被丢弃。在第 13 章的 [捕获引用或移动所有权][capture-or-move]<!-- ignore --> 一节中，你学习了如何使用 `move` 关键字与闭包，并且如第 16 章的 [使用 `move` 闭包与线程][move-threads]<!-- ignore --> 一节中所讨论的，我们在使用线程时经常需要将数据移动到闭包中。同样的基本动态适用于 async 块，因此 `move` 关键字与 async 块一起使用，就像它与闭包一起使用一样。

在 Listing 17-12 中，我们将用于发送消息的块从 `async` 更改为 `async move`。当我们运行**这个**版本的代码时，它会在最后一条消息发送和接收后优雅地关闭。

<Listing number="17-12" caption="Listing 17-11 代码的修订版，完成后正确关闭" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

这个 async 通道也是一个多生产者通道，所以如果我们想从多个 futures 发送消息，我们可以调用 `clone` 来复制 `tx`，如 Listing 17-13 所示。

<Listing number="17-13" caption="使用 async 块的多生产者" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

首先，我们克隆 `tx`，在第一个 async 块之外创建 `tx1`。我们像之前对 `tx` 一样将 `tx1` 移动到该块中。然后，稍后，我们将原始的 `tx` 移动到一个**新的** async 块中，在那里我们以稍慢的延迟发送更多消息。我们碰巧将这个新的 async 块放在接收消息的 async 块之后，但它也可以放在前面。关键是 futures 被等待的顺序，而不是它们被创建的顺序。

发送消息的两个 async 块都需要是 `async move` 块，以便 `tx` 和 `tx1` 在这些块完成时被丢弃。否则，我们将回到我们开始的无限循环中。最后，我们从 `trpl::join` 切换到 `trpl::join3` 来处理额外的 future。

现在我们看到来自两个发送 futures 的所有消息，并且因为发送 futures 在发送后使用略有不同的延迟，消息也会以这些不同的间隔接收。

<!-- 不提取输出，因为输出的变化不显著；变化可能是由于线程运行方式不同，而不是编译器的变化 -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

这是一个好的开始，但它将我们限制在只有少数 futures：两个使用 `join`，或三个使用 `join3`。让我们看看我们如何可能处理更多的 futures。

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish-using-join-handles
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
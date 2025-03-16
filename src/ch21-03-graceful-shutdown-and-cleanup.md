## 优雅关闭与清理

在 Listing 21-20 中的代码通过使用线程池异步地响应请求，正如我们所期望的那样。我们收到了一些关于 `workers`、`id` 和 `thread` 字段的警告，这些字段我们没有直接使用，提醒我们没有清理任何东西。当我们使用不太优雅的 <kbd>ctrl</kbd>-<kbd>c</kbd> 方法来停止主线程时，所有其他线程也会立即停止，即使它们正在处理请求。

接下来，我们将实现 `Drop` trait 来在线程池中的每个线程上调用 `join`，以便它们可以在关闭之前完成正在处理的请求。然后我们将实现一种方法来告诉线程它们应该停止接受新请求并关闭。为了看到这段代码的实际效果，我们将修改我们的服务器，使其在优雅关闭线程池之前只接受两个请求。

在我们进行的过程中需要注意的一点是：这些都不会影响处理执行闭包的代码部分，因此如果我们为异步运行时使用线程池，这里的一切都将保持不变。

### 在 `ThreadPool` 上实现 `Drop` Trait

让我们从在线程池上实现 `Drop` 开始。当线程池被丢弃时，我们的线程应该全部 `join` 以确保它们完成工作。Listing 21-22 展示了 `Drop` 实现的第一次尝试；这段代码还不能完全工作。

<Listing number="21-22" file-name="src/lib.rs" caption="当线程池超出作用域时，对每个线程进行 join">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

首先，我们遍历线程池中的每个 `workers`。我们使用 `&mut` 因为 `self` 是一个可变引用，并且我们还需要能够改变 `worker`。对于每个 worker，我们打印一条消息，说明这个特定的 `Worker` 实例正在关闭，然后我们对该 `Worker` 实例的线程调用 `join`。如果调用 `join` 失败，我们使用 `unwrap` 使 Rust 恐慌并进入不优雅的关闭。

当我们编译这段代码时，会得到以下错误：

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

错误告诉我们不能调用 `join`，因为我们只有每个 `worker` 的可变借用，而 `join` 需要拥有其参数的所有权。为了解决这个问题，我们需要将线程从拥有 `thread` 的 `Worker` 实例中移出，以便 `join` 可以消费线程。一种方法是采用我们在 Listing 18-15 中使用的相同方法。如果 `Worker` 持有一个 `Option<thread::JoinHandle<()>>`，我们可以调用 `Option` 上的 `take` 方法将值从 `Some` 变体中移出，并在其位置留下一个 `None` 变体。换句话说，一个正在运行的 `Worker` 会在 `thread` 中有一个 `Some` 变体，当我们想要清理一个 `Worker` 时，我们会用 `None` 替换 `Some`，这样 `Worker` 就不会有线程可以运行。

然而，唯一会出现这种情况的时候是在丢弃 `Worker` 时。作为交换，我们必须在任何访问 `worker.thread` 的地方处理 `Option<thread::JoinHandle<()>>`。惯用的 Rust 会大量使用 `Option`，但当你发现自己将你知道总是存在的东西包装在 `Option` 中作为这样的变通方法时，最好寻找替代方法。它们可以使你的代码更简洁且更不容易出错。

在这种情况下，存在一个更好的替代方法：`Vec::drain` 方法。它接受一个范围参数来指定要从 `Vec` 中移除哪些项，并返回这些项的迭代器。传递 `..` 范围语法将移除 `Vec` 中的每个值。

所以我们需要像这样更新 `ThreadPool` 的 `drop` 实现：

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

这解决了编译器错误，并且不需要对我们的代码进行任何其他更改。

### 向线程发出信号以停止监听任务

随着我们进行的所有更改，我们的代码在没有任何警告的情况下编译。然而，坏消息是这段代码还没有按照我们想要的方式工作。关键在于 `Worker` 实例的线程运行的闭包中的逻辑：目前，我们调用 `join`，但这不会关闭线程，因为它们会永远循环寻找任务。如果我们尝试使用当前的 `drop` 实现来丢弃 `ThreadPool`，主线程将永远阻塞，等待第一个线程完成。

要解决这个问题，我们需要在 `ThreadPool` 的 `drop` 实现中进行更改，然后在 `Worker` 循环中进行更改。

首先，我们将更改 `ThreadPool` 的 `drop` 实现，在等待线程完成之前显式丢弃 `sender`。Listing 21-23 显示了对 `ThreadPool` 的更改，以显式丢弃 `sender`。与线程不同，这里我们确实需要使用 `Option` 来能够使用 `Option::take` 将 `sender` 从 `ThreadPool` 中移出。

<Listing number="21-23" file-name="src/lib.rs" caption="在 join `Worker` 线程之前显式丢弃 `sender`">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

丢弃 `sender` 会关闭通道，这表明不会再发送消息。当这种情况发生时，`Worker` 实例在无限循环中执行的所有 `recv` 调用都将返回一个错误。在 Listing 21-24 中，我们更改了 `Worker` 循环，以便在这种情况下优雅地退出循环，这意味着当 `ThreadPool` 的 `drop` 实现调用 `join` 时，线程将完成。

<Listing number="21-24" file-name="src/lib.rs" caption="当 `recv` 返回错误时显式跳出循环">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

为了看到这段代码的实际效果，让我们修改 `main`，使其在优雅关闭服务器之前只接受两个请求，如 Listing 21-25 所示。

<Listing number="21-25" file-name="src/main.rs" caption="通过退出循环在服务两个请求后关闭服务器">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

你不会希望一个真实世界的 Web 服务器在只服务两个请求后就关闭。这段代码只是演示了优雅关闭和清理功能正常工作。

`take` 方法定义在 `Iterator` trait 中，并将迭代限制为最多前两项。`ThreadPool` 将在 `main` 结束时超出作用域，并且 `drop` 实现将运行。

使用 `cargo run` 启动服务器，并发出三个请求。第三个请求应该会出错，并且在你的终端中你应该会看到类似于以下的输出：

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

你可能会看到不同的 `Worker` ID 和消息打印顺序。我们可以从消息中看到这段代码是如何工作的：`Worker` 实例 0 和 3 获得了前两个请求。服务器在第二个连接后停止接受连接，并且 `ThreadPool` 上的 `Drop` 实现甚至在 `Worker` 3 开始其工作之前就开始执行。丢弃 `sender` 会断开所有 `Worker` 实例的连接并告诉它们关闭。每个 `Worker` 实例在断开连接时都会打印一条消息，然后线程池调用 `join` 等待每个 `Worker` 线程完成。

注意这个特定执行的一个有趣方面：`ThreadPool` 丢弃了 `sender`，并且在任何 `Worker` 收到错误之前，我们尝试 join `Worker` 0。`Worker` 0 还没有从 `recv` 收到错误，所以主线程阻塞等待 `Worker` 0 完成。与此同时，`Worker` 3 收到了一个任务，然后所有线程都收到了一个错误。当 `Worker` 0 完成时，主线程等待其余的 `Worker` 实例完成。那时，它们都已经退出了循环并停止了。

恭喜！我们现在已经完成了我们的项目；我们有一个使用线程池异步响应的基本 Web 服务器。我们能够执行服务器的优雅关闭，这会清理池中的所有线程。

以下是完整代码供参考：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

我们还可以做更多的事情！如果你想继续增强这个项目，这里有一些想法：

- 为 `ThreadPool` 及其公共方法添加更多文档。
- 添加库功能的测试。
- 将 `unwrap` 的调用更改为更健壮的错误处理。
- 使用 `ThreadPool` 执行除服务 Web 请求之外的其他任务。
- 在 [crates.io](https://crates.io/) 上找到一个线程池 crate，并使用该 crate 实现一个类似的 Web 服务器。然后将其 API 和健壮性与我们实现的线程池进行比较。

## 总结

干得好！你已经完成了本书的学习！我们感谢你加入我们的 Rust 之旅。你现在已经准备好实现自己的 Rust 项目并帮助其他人的项目。请记住，有一个欢迎的 Rustaceans 社区，他们很乐意帮助你在 Rust 旅程中遇到的任何挑战。
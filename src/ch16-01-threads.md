## 使用线程同时运行代码

在大多数现代操作系统中，执行的程序代码运行在一个**进程**中，操作系统会同时管理多个进程。在一个程序中，你也可以有独立的部分同时运行。运行这些独立部分的功能称为**线程**。例如，一个 Web 服务器可以有多个线程，以便能够同时响应多个请求。

将程序中的计算拆分为多个线程以同时运行多个任务可以提高性能，但也会增加复杂性。由于线程可以同时运行，因此无法保证不同线程上的代码部分的执行顺序。这可能会导致以下问题：

- **竞态条件**：线程以不一致的顺序访问数据或资源。
- **死锁**：两个线程互相等待，导致两个线程都无法继续执行。
- **难以复现和修复的 bug**：某些情况下才会发生的 bug，难以可靠地复现和修复。

Rust 试图减轻使用线程的负面影响，但在多线程上下文中编程仍然需要仔细思考，并且需要一个与单线程程序中不同的代码结构。

编程语言以几种不同的方式实现线程，许多操作系统提供了语言可以调用的 API 来创建新线程。Rust 标准库使用**1:1**线程实现模型，即程序为每个语言线程使用一个操作系统线程。有一些 crate 实现了其他线程模型，这些模型在 1:1 模型的基础上做出了不同的权衡。（Rust 的异步系统，我们将在下一章中看到，也提供了另一种并发方法。）

### 使用 `spawn` 创建新线程

要创建一个新线程，我们调用 `thread::spawn` 函数并传递一个闭包（我们在第 13 章讨论过闭包），闭包中包含我们希望在新线程中运行的代码。Listing 16-1 中的示例在主线程中打印一些文本，并在新线程中打印其他文本：

<Listing number="16-1" file-name="src/main.rs" caption="创建一个新线程以打印一些内容，同时主线程打印其他内容">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

请注意，当 Rust 程序的主线程完成时，所有生成的线程都会被关闭，无论它们是否已完成运行。这个程序的输出可能每次都会有所不同，但看起来会类似于以下内容：

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

对 `thread::sleep` 的调用会强制线程停止执行一小段时间，从而允许其他线程运行。线程可能会轮流执行，但这并不能保证：这取决于操作系统如何调度线程。在这个运行中，主线程首先打印，尽管生成线程的打印语句在代码中首先出现。而且，尽管我们告诉生成线程打印直到 `i` 为 `9`，但它只打印到 `5`，主线程就关闭了。

如果你运行此代码并只看到主线程的输出，或者没有看到任何重叠，请尝试增加范围中的数字，以创建更多机会让操作系统在线程之间切换。

### 使用 `join` 句柄等待所有线程完成

Listing 16-1 中的代码不仅由于主线程结束而大多数情况下会提前停止生成线程，而且由于无法保证线程运行的顺序，我们也无法保证生成线程会运行！

我们可以通过将 `thread::spawn` 的返回值保存在一个变量中来修复生成线程不运行或提前结束的问题。`thread::spawn` 的返回类型是 `JoinHandle<T>`。`JoinHandle<T>` 是一个拥有值，当我们对其调用 `join` 方法时，它将等待其线程完成。Listing 16-2 展示了如何使用我们在 Listing 16-1 中创建的线程的 `JoinHandle<T>`，并调用 `join` 以确保生成线程在 `main` 退出之前完成。

<Listing number="16-2" file-name="src/main.rs" caption="保存 `thread::spawn` 的 `JoinHandle<T>` 以确保线程运行到完成">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

在句柄上调用 `join` 会阻塞当前运行的线程，直到句柄所代表的线程终止。**阻塞**一个线程意味着该线程被阻止执行工作或退出。因为我们将 `join` 的调用放在了主线程的 `for` 循环之后，运行 Listing 16-2 应该会产生类似于以下的输出：

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

两个线程继续交替执行，但主线程由于调用了 `handle.join()` 而等待，直到生成线程完成后才会结束。

但是，让我们看看如果我们将 `handle.join()` 移到 `main` 中的 `for` 循环之前会发生什么，如下所示：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

主线程将等待生成线程完成，然后运行其 `for` 循环，因此输出将不再交错，如下所示：

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

一些小细节，比如 `join` 的调用位置，可能会影响你的线程是否同时运行。

### 在线程中使用 `move` 闭包

我们经常将 `move` 关键字与传递给 `thread::spawn` 的闭包一起使用，因为闭包将获取其使用环境中的值的所有权，从而将这些值的所有权从一个线程转移到另一个线程。在第 13 章的[“使用闭包捕获环境”][capture]中，我们讨论了 `move` 在闭包上下文中的使用。现在，我们将更多地关注 `move` 和 `thread::spawn` 之间的交互。

请注意，在 Listing 16-1 中，我们传递给 `thread::spawn` 的闭包没有参数：我们没有在主线程中使用任何数据来生成线程的代码。要在生成线程中使用主线程中的数据，生成线程的闭包必须捕获它需要的值。Listing 16-3 展示了尝试在主线程中创建一个向量并在生成线程中使用它。然而，这还不会起作用，你马上就会看到。

<Listing number="16-3" file-name="src/main.rs" caption="尝试在主线程中创建的向量在另一个线程中使用">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

闭包使用了 `v`，因此它将捕获 `v` 并使其成为闭包环境的一部分。因为 `thread::spawn` 在新线程中运行此闭包，我们应该能够在该新线程中访问 `v`。但是当我们编译这个示例时，会得到以下错误：

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust **推断**如何捕获 `v`，因为 `println!` 只需要 `v` 的引用，闭包尝试借用 `v`。然而，有一个问题：Rust 无法知道生成线程将运行多长时间，因此它不知道 `v` 的引用是否始终有效。

Listing 16-4 提供了一个更有可能出现 `v` 的引用无效的场景：

<Listing number="16-4" file-name="src/main.rs" caption="一个线程的闭包尝试捕获主线程中 `v` 的引用，而主线程会丢弃 `v`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

如果 Rust 允许我们运行此代码，生成线程可能会立即被放到后台而不运行。生成线程内部有一个对 `v` 的引用，但主线程立即使用我们在第 15 章讨论的 `drop` 函数丢弃了 `v`。然后，当生成线程开始执行时，`v` 不再有效，因此对它的引用也无效。哦不！

要修复 Listing 16-3 中的编译器错误，我们可以使用错误消息的建议：

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

通过在闭包前添加 `move` 关键字，我们强制闭包获取它使用的值的所有权，而不是让 Rust 推断它应该借用这些值。Listing 16-5 中显示的 Listing 16-3 的修改将按我们的意图编译并运行。

<Listing number="16-5" file-name="src/main.rs" caption="使用 `move` 关键字强制闭包获取它使用的值的所有权">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

我们可能会尝试用同样的方法来修复 Listing 16-4 中的代码，其中主线程调用了 `drop`，使用 `move` 闭包。然而，这个修复方法不会奏效，因为 Listing 16-4 试图做的事情由于不同的原因而被禁止。如果我们在闭包中添加 `move`，我们将 `v` 移动到闭包的环境中，并且我们不能再在主线程中调用 `drop`。我们会得到以下编译器错误：

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Rust 的所有权规则再次拯救了我们！我们在 Listing 16-3 中得到的错误是因为 Rust 保守地只借用 `v` 给线程，这意味着主线程理论上可能会使生成线程的引用无效。通过告诉 Rust 将 `v` 的所有权移动到生成线程，我们向 Rust 保证主线程不会再使用 `v`。如果我们以同样的方式修改 Listing 16-4，当我们尝试在主线程中使用 `v` 时，我们就违反了所有权规则。`move` 关键字覆盖了 Rust 保守的默认借用行为；它不允许我们违反所有权规则。

现在我们已经介绍了线程是什么以及线程 API 提供的方法，让我们看看一些可以使用线程的场景。

[capture]: ch13-01-closures.html#capturing-the-environment-with-closures
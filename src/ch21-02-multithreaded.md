## 将我们的单线程服务器转变为多线程服务器

目前，服务器会依次处理每个请求，这意味着在处理完第一个连接之前，它不会处理第二个连接。如果服务器收到越来越多的请求，这种串行执行将变得越来越不理想。如果服务器收到一个需要很长时间处理的请求，即使新的请求可以快速处理，后续请求也必须等待长时间请求完成。我们需要解决这个问题，但首先我们来看一下实际的问题。

### 在当前服务器实现中模拟慢请求

我们将看看一个处理缓慢的请求如何影响我们当前服务器实现中的其他请求。Listing 21-10 实现了对 _/sleep_ 请求的处理，模拟了一个慢响应，导致服务器在响应前睡眠五秒钟。

<Listing number="21-10" file-name="src/main.rs" caption="通过睡眠5秒模拟慢请求">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

我们现在从 `if` 切换到了 `match`，因为我们有三个情况。我们需要显式地匹配 `request_line` 的一个切片，以便与字符串字面值进行模式匹配；`match` 不会像相等方法那样自动引用和解引用。

第一个分支与 Listing 21-9 中的 `if` 块相同。第二个分支匹配 _/sleep_ 请求。当收到该请求时，服务器将在渲染成功的 HTML 页面之前睡眠五秒钟。第三个分支与 Listing 21-9 中的 `else` 块相同。

你可以看到我们的服务器是多么原始：真正的库会以更简洁的方式处理多个请求的识别！

使用 `cargo run` 启动服务器。然后打开两个浏览器窗口：一个用于 _http://127.0.0.1:7878/_，另一个用于 _http://127.0.0.1:7878/sleep_。如果你像之前一样多次输入 _/_ URI，你会看到它快速响应。但如果你输入 _/sleep_ 然后加载 _/_，你会看到 _/_ 会等待 `sleep` 完成五秒的睡眠后再加载。

我们可以使用多种技术来避免请求在慢请求后堆积，包括像我们在第17章中那样使用异步；我们将实现的是线程池。

### 使用线程池提高吞吐量

_线程池_ 是一组生成的线程，它们等待并准备处理任务。当程序收到新任务时，它会将池中的一个线程分配给该任务，该线程将处理该任务。池中的其余线程可用于处理第一个线程处理任务时传入的其他任务。当第一个线程处理完其任务后，它会返回到空闲线程池中，准备处理新任务。线程池允许你并发处理连接，从而提高服务器的吞吐量。

我们将限制池中的线程数量，以防止 DoS 攻击；如果我们的程序为每个传入的请求创建一个新线程，那么有人向我们的服务器发出1000万次请求可能会通过耗尽我们服务器的所有资源并使请求处理停止来造成混乱。

因此，我们将让池中等待的线程数量固定。传入的请求被发送到池中进行处理。池将维护一个传入请求的队列。池中的每个线程将从该队列中弹出一个请求，处理该请求，然后向队列请求另一个请求。通过这种设计，我们可以并发处理最多 *`N`* 个请求，其中 *`N`* 是线程的数量。如果每个线程都在响应一个长时间运行的请求，后续请求仍然可能在队列中堆积，但我们已经增加了在达到该点之前可以处理的长时间运行请求的数量。

这种技术只是提高 Web 服务器吞吐量的众多方法之一。你可能探索的其他选项包括 fork/join 模型、单线程异步 I/O 模型和多线程异步 I/O 模型。如果你对这个主题感兴趣，可以阅读更多关于其他解决方案的内容，并尝试实现它们；对于像 Rust 这样的低级语言，所有这些选项都是可能的。

在我们开始实现线程池之前，让我们讨论一下使用池应该是什么样子。当你尝试设计代码时，首先编写客户端接口可以帮助指导你的设计。编写代码的 API，使其结构符合你希望调用的方式；然后在该结构中实现功能，而不是先实现功能再设计公共 API。

类似于我们在第12章的项目中使用测试驱动开发的方式，我们将在这里使用编译器驱动开发。我们将编写调用我们想要的函数的代码，然后查看编译器的错误，以确定接下来应该更改什么以使代码工作。然而，在此之前，我们将探讨我们不打算作为起点的技术。

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### 为每个请求生成一个线程

首先，让我们探讨一下如果我们的代码为每个连接创建一个新线程，它可能是什么样子。如前所述，由于可能生成无限数量的线程的问题，这不是我们的最终计划，但它是首先获得一个工作的多线程服务器的起点。然后我们将添加线程池作为改进，对比这两种解决方案会更容易。Listing 21-11 显示了在 `for` 循环中生成一个新线程以处理每个流的 `main` 的更改。

<Listing number="21-11" file-name="src/main.rs" caption="为每个流生成一个新线程">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

正如你在第16章中学到的，`thread::spawn` 将创建一个新线程，然后在新线程中运行闭包中的代码。如果你运行此代码并在浏览器中加载 _/sleep_，然后在另外两个浏览器标签中加载 _/_，你确实会看到 _/_ 的请求不必等待 _/sleep_ 完成。然而，正如我们提到的，这最终会压倒系统，因为你将无限制地创建新线程。

你可能还记得第17章，这正是 async 和 await 真正闪耀的地方！在我们构建线程池时，请记住这一点，并思考使用 async 时事情会有什么不同或相同。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### 创建有限数量的线程

我们希望我们的线程池以类似、熟悉的方式工作，以便从线程切换到线程池不需要对使用我们 API 的代码进行大的更改。Listing 21-12 显示了我们希望使用的 `ThreadPool` 结构的假设接口，而不是 `thread::spawn`。

<Listing number="21-12" file-name="src/main.rs" caption="我们理想的 `ThreadPool` 接口">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

我们使用 `ThreadPool::new` 创建一个具有可配置数量线程的新线程池，在本例中为四个。然后，在 `for` 循环中，`pool.execute` 的接口与 `thread::spawn` 类似，因为它接受一个闭包，池应该为每个流运行该闭包。我们需要实现 `pool.execute`，以便它接受闭包并将其交给池中的一个线程运行。这段代码还不能编译，但我们会尝试，以便编译器可以指导我们如何修复它。

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### 使用编译器驱动开发构建 `ThreadPool`

在 _src/main.rs_ 中进行 Listing 21-12 中的更改，然后让我们使用 `cargo check` 的编译器错误来驱动我们的开发。这是我们得到的第一个错误：

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

太好了！这个错误告诉我们我们需要一个 `ThreadPool` 类型或模块，所以我们现在就构建一个。我们的 `ThreadPool` 实现将独立于我们的 Web 服务器所做的工作类型。因此，让我们将 `hello` crate 从二进制 crate 切换为库 crate，以保存我们的 `ThreadPool` 实现。在更改为库 crate 后，我们还可以使用单独的线程池库来处理我们想要使用线程池完成的任何工作，而不仅仅是处理 Web 请求。

创建一个 _src/lib.rs_ 文件，其中包含以下内容，这是我们目前可以拥有的最简单的 `ThreadPool` 结构定义：

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>

然后编辑 _main.rs_ 文件，通过将以下代码添加到 _src/main.rs_ 的顶部，将 `ThreadPool` 从库 crate 引入作用域：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

这段代码仍然无法工作，但让我们再次检查它以获取我们需要解决的下一个错误：

```console
{{#include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

这个错误表明接下来我们需要为 `ThreadPool` 创建一个名为 `new` 的关联函数。我们还知道 `new` 需要有一个可以接受 `4` 作为参数的参数，并且应该返回一个 `ThreadPool` 实例。让我们实现最简单的 `new` 函数，该函数将具有这些特征：

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

我们选择 `usize` 作为 `size` 参数的类型，因为我们知道负数的线程没有意义。我们还知道我们将使用这个 `4` 作为线程集合中的元素数量，这正是 `usize` 类型的用途，正如第3章中的 [“整数类型”][integer-types]<!-- ignore --> 所讨论的那样。

让我们再次检查代码：

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

现在错误发生是因为我们没有 `ThreadPool` 上的 `execute` 方法。回想一下 [“创建有限数量的线程”](#creating-a-finite-number-of-threads)<!-- ignore -->，我们决定我们的线程池应该有一个类似于 `thread::spawn` 的接口。此外，我们将实现 `execute` 函数，以便它接受给定的闭包并将其交给池中的一个空闲线程运行。

我们将在 `ThreadPool` 上定义 `execute` 方法，以闭包作为参数。回想一下第13章中的 [“将捕获的值移出闭包和 `Fn` 特性”][fn-traits]<!-- ignore -->，我们可以使用三种不同的特性来接受闭包作为参数：`Fn`、`FnMut` 和 `FnOnce`。我们需要决定在这里使用哪种闭包。我们知道我们最终会做一些类似于标准库 `thread::spawn` 实现的事情，所以我们可以查看 `thread::spawn` 的签名对其参数的限制。文档向我们展示了以下内容：

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`F` 类型参数是我们在这里关心的；`T` 类型参数与返回值有关，我们不关心它。我们可以看到 `spawn` 使用 `FnOnce` 作为 `F` 的特性限制。这可能也是我们想要的，因为我们最终会将 `execute` 中得到的参数传递给 `spawn`。我们可以进一步确信 `FnOnce` 是我们想要使用的特性，因为运行请求的线程只会执行该请求的闭包一次，这与 `FnOnce` 中的 `Once` 相匹配。

`F` 类型参数还具有 `Send` 特性限制和生命周期限制 `'static`，这在我们的情况下很有用：我们需要 `Send` 将闭包从一个线程转移到另一个线程，并且需要 `'static` 因为我们不知道线程执行需要多长时间。让我们在 `ThreadPool` 上创建一个 `execute` 方法，该方法将接受一个类型为 `F` 的通用参数，并具有这些限制：

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

我们仍然在 `FnOnce` 之后使用 `()`，因为这个 `FnOnce` 表示一个不接受参数并返回单元类型 `()` 的闭包。就像函数定义一样，返回类型可以从签名中省略，但即使我们没有参数，我们仍然需要括号。

再次强调，这是 `execute` 方法的最简单实现：它什么都不做，但我们只是试图让我们的代码编译。让我们再次检查它：

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

它编译了！但请注意，如果你尝试 `cargo run` 并在浏览器中发出请求，你会看到我们在本章开头看到的浏览器中的错误。我们的库实际上还没有调用传递给 `execute` 的闭包！

> 注意：你可能会听到关于具有严格编译器的语言（如 Haskell 和 Rust）的说法：“如果代码编译，它就能工作。”但这种说法并不普遍正确。我们的项目编译了，但它绝对没有做任何事情！如果我们正在构建一个真正的、完整的项目，这将是一个开始编写单元测试以检查代码是否编译 _并且_ 具有我们想要的行为的好时机。

考虑：如果我们打算执行一个 _future_ 而不是闭包，这里会有什么不同？

#### 在 `new` 中验证线程数量

我们没有对 `new` 和 `execute` 的参数做任何事情。让我们用我们想要的行为实现这些函数的主体。首先，让我们考虑 `new`。之前我们为 `size` 参数选择了一个无符号类型，因为具有负数的线程池没有意义。然而，具有零个线程的池也没有意义，但零是一个完全有效的 `usize`。我们将在返回 `ThreadPool` 实例之前添加代码来检查 `size` 是否大于零，并通过使用 `assert!` 宏让程序在接收到零时 panic，如 Listing 21-13 所示。

<Listing number="21-13" file-name="src/lib.rs" caption="实现 `ThreadPool::new` 以在 `size` 为零时 panic">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

我们还为我们的 `ThreadPool` 添加了一些文档注释。请注意，我们遵循了良好的文档实践，添加了一个部分来指出我们的函数可能 panic 的情况，如第14章所讨论的。尝试运行 `cargo doc --open` 并点击 `ThreadPool` 结构以查看为 `new` 生成的文档！

与其像我们在这里所做的那样添加 `assert!` 宏，我们可以将 `new` 更改为 `build` 并返回一个 `Result`，就像我们在 I/O 项目中的 `Config::build` 中所做的那样（Listing 12-9）。但在这种情况下，我们决定尝试创建一个没有任何线程的线程池应该是一个不可恢复的错误。如果你有雄心壮志，尝试编写一个名为 `build` 的函数，其签名如下，以与 `new` 函数进行比较：

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### 创建存储线程的空间

现在我们有了一个方法来知道我们有一个有效的线程数量来存储在池中，我们可以在返回结构之前创建这些线程并将它们存储在 `ThreadPool` 结构中。但是我们如何“存储”一个线程？让我们再看一下 `thread::spawn` 的签名：

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`spawn` 函数返回一个 `JoinHandle<T>`，其中 `T` 是闭包返回的类型。让我们也尝试使用 `JoinHandle`，看看会发生什么。在我们的情况下，我们传递给线程池的闭包将处理连接并且不返回任何内容，因此 `T` 将是单元类型 `()`。

Listing 21-14 中的代码将编译但尚未创建任何线程。我们更改了 `ThreadPool` 的定义，以保存一个 `thread::JoinHandle<()>` 实例的向量，用 `size` 的容量初始化了向量，设置了一个 `for` 循环来运行一些代码以创建线程，并返回一个包含它们的 `ThreadPool` 实例。

<Listing number="21-14" file-name="src/lib.rs" caption="为 `ThreadPool` 创建一个向量以保存线程">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

我们将 `std::thread` 引入库 crate 的作用域，因为我们在 `ThreadPool` 中使用 `thread::JoinHandle` 作为向量中的项的类型。

一旦接收到有效的 `size`，我们的 `ThreadPool` 就会创建一个可以容纳 `size` 个项的新向量。`with_capacity` 函数执行与 `Vec::new` 相同的任务，但有一个重要的区别：它预先分配向量中的空间。因为我们知道我们需要在向量中存储 `size` 个元素，所以预先进行此分配比使用 `Vec::new` 稍微更高效，后者在插入元素时会调整自身大小。

当你再次运行 `cargo check` 时，它应该会成功。

#### 一个 `Worker` 结构负责将代码从 `ThreadPool` 发送到线程

我们在 Listing 21-14 的 `for` 循环中留下了一个关于创建线程的注释。在这里，我们将看看我们如何实际创建线程。标准库提供了 `thread::spawn` 作为创建线程的方式，`thread::spawn` 期望在创建线程时立即获得一些代码来运行。然而，在我们的情况下，我们想要创建线程并让它们 _等待_ 我们稍后发送的代码。标准库的线程实现不包括任何方法来做到这一点；我们必须手动实现它。

我们将通过引入一个新的数据结构来实现这种行为，该数据结构位于 `ThreadPool` 和线程之间，将管理这种新行为。我们将这个数据结构称为 _Worker_，这是池实现中的一个常见术语。`Worker` 会拾取需要运行的代码并在 Worker 的线程中运行该代码。

想象一下在餐厅厨房工作的人：工人们等待顾客的订单，然后他们负责接受这些订单并完成它们。

我们不会在线程池中存储 `JoinHandle<()>` 实例的向量，而是存储 `Worker` 结构的实例。每个 `Worker` 将存储一个 `JoinHandle<()>` 实例。然后我们将在 `Worker` 上实现一个方法，该方法将接受要运行的代码的闭包并将其发送到已经运行的线程以执行。我们还将为每个 `Worker` 提供一个 `id`，以便在记录或调试时区分池中的不同 `Worker` 实例。

以下是当我们创建 `ThreadPool` 时将发生的新过程。在我们以这种方式设置 `Worker` 之后，我们将实现将闭包发送到线程的代码：

1. 定义一个 `Worker` 结构，它包含一个 `id` 和一个 `JoinHandle<()>`。
2. 将 `ThreadPool` 更改为保存 `Worker` 实例的向量。
3. 定义一个 `Worker::new` 函数，它接受一个 `id` 数字并返回一个包含 `id` 和一个使用空闭包生成的线程的 `Worker` 实例。
4. 在 `ThreadPool::new` 中，使用 `for` 循环计数器生成一个 `id`，使用该 `id` 创建一个新的 `Worker`，并将该 worker 存储在向量中。

如果你愿意接受挑战，尝试在查看 Listing 21-15 中的代码之前自己实现这些更改。

准备好了吗？以下是 Listing 21-15，其中包含一种进行上述修改的方式。

<Listing number="21-15" file-name="src/lib.rs" caption="修改 `ThreadPool` 以保存 `Worker` 实例而不是直接保存线程">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

我们将 `ThreadPool` 上的字段名称从 `threads` 更改为 `workers`，因为它现在保存的是 `Worker` 实例而不是 `JoinHandle<()>` 实例。我们使用 `for` 循环中的计数器作为 `Worker::new` 的参数，并将每个新的 `Worker` 存储在名为 `workers` 的向量中。

外部代码（如我们在 _src/main.rs_ 中的服务器）不需要知道有关在 `ThreadPool` 中使用 `Worker` 结构的实现细节，因此我们将 `Worker` 结构及其 `new` 函数设为私有。`Worker::new` 函数使用我们给它的 `id` 并存储一个 `JoinHandle<()>` 实例，该实例是通过使用空闭包生成一个新线程创建的。

> 注意：如果操作系统无法创建线程，因为系统资源不足，`thread::spawn` 会 panic。这将导致我们的整个服务器 panic，即使某些线程的创建可能成功。为了简单起见，这种行为是可以的，但在生产线程池实现中，你可能希望使用 [`std::thread::Builder`][builder]<!-- ignore --> 及其 [`spawn`][builder-spawn]<!-- ignore --> 方法，该方法返回 `Result` 而不是 panic。

这段代码将编译并存储我们指定为 `ThreadPool::new` 参数的 `Worker` 实例数量。但我们 _仍然_ 没有处理我们在 `execute` 中得到的闭包。让我们看看接下来如何做到这一点。

#### 通过通道向线程发送请求

我们将解决的下一个问题是，传递给 `thread::spawn` 的闭包绝对什么都不做。目前，我们在 `execute` 方法中得到了我们想要执行的闭包。但我们需要在创建 `ThreadPool` 期间创建每个 `Worker` 时给 `thread::spawn` 一个闭包来运行。

我们希望我们刚刚创建的 `Worker` 结构从 `ThreadPool` 持有的队列中获取要运行的代码，并将该代码发送到其线程以运行。

我们在第16章中学到的通道——一种在两个线程之间通信的简单方式——将非常适合这个用例。我们将使用一个通道作为作业队列，`execute` 将从 `ThreadPool` 向 `Worker` 实例发送一个作业，该作业将发送作业到其线程。以下是计划：

1. `ThreadPool` 将创建一个通道并保留发送者。
2. 每个 `Worker` 将保留接收者。
3. 我们将创建一个新的 `Job` 结构，它将保存我们想要通过通道发送的闭包。
4. `execute` 方法将通过发送者发送它想要执行的作业。
5. 在其线程中，`Worker` 将循环遍历其接收者并执行它接收到的任何作业的闭包。

让我们从在 `ThreadPool::new` 中创建一个通道并让 `ThreadPool` 实例保留发送者开始，如 Listing 21-16 所示。`Job` 结构目前不保存任何内容，但将是我们通过通道发送的项的类型。

<Listing number="21-16" file-name="src/lib.rs" caption="修改 `ThreadPool` 以存储传输 `Job` 实例的通道的发送者">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

在 `ThreadPool::new` 中，我们创建了我们的新通道，并让池保留发送者。这将成功编译。

让我们尝试在创建通道时将通道的接收者传递给每个 `Worker`。我们知道我们希望在 `Worker` 实例生成的线程中使用接收者，因此我们将在闭包中引用 `receiver` 参数。Listing 21-17 中的代码还不能完全编译。

<Listing number="21-17" file-name="src/lib.rs" caption="将接收者传递给每个 `Worker`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

我们做了一些小而直接的更改：我们将接收者传递给 `Worker::new`，然后我们在闭包中使用它。

当我们尝试检查这段代码时，我们得到了这个错误：

```console
{{#include ../listings/ch21-web-server/listing-21-17/output.txt}}
```

代码试图将 `receiver` 传递给多个 `Worker` 实例。这不会起作用，正如你从第16章回忆的那样：Rust 提供的通道实现是多个 _生产者_，单个 _消费者_。这意味着我们不能仅仅克隆通道的消费端来修复这段代码。我们也不想多次向多个消费者发送消息；我们希望有一个消息列表，多个 `Worker` 实例，以便每个消息只被处理一次。

此外，从通道队列中取出一个作业涉及修改 `receiver`，因此线程需要一种安全的方式来共享和修改 `receiver`；否则，我们可能会遇到竞争条件（如第16章所述）。

回想一下第16章中讨论的线程安全智能指针：为了跨多个线程共享所有权并允许线程修改值，我们需要使用 `Arc<Mutex<T>>`。`Arc` 类型将允许多个 `Worker` 实例拥有接收者，而 `Mutex` 将确保一次只有一个 `Worker` 从接收者获取作业。Listing 21-18 显示了我们需要的更改。

<Listing number="21-18" file-name="src/lib.rs" caption="使用 `Arc` 和 `Mutex` 在 `Worker` 实例之间共享接收者">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

在 `ThreadPool::new` 中，我们将接收者放入 `Arc` 和 `Mutex` 中。对于每个新的 `Worker`，我们克隆 `Arc` 以增加引用计数，以便 `Worker` 实例可以共享接收者的所有权。

通过这些更改，代码编译了！我们快完成了！

#### 实现 `execute` 方法

让我们最终实现 `ThreadPool` 上的 `execute` 方法。我们还将把 `Job` 从结构更改为类型别名，用于保存 `execute` 接收到的闭包类型的特征对象。正如第20章中的 [“使用类型别名创建类型同义词”][creating-type-synonyms-with-type-aliases]<!-- ignore --> 所讨论的那样，类型别名允许我们缩短长类型以便于使用。请看 Listing 21-19。

<Listing number="21-19" file-name="src/lib.rs" caption="为保存每个闭包的 `Box` 创建一个 `Job` 类型别名，然后将作业发送到通道">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

在创建了一个新的 `Job` 实例后，使用我们在 `execute` 中得到的闭包，我们将该作业发送到通道的发送端。我们在 `send` 上调用 `unwrap` 以处理发送失败的情况。如果例如我们停止所有线程执行，这意味着接收端已停止接收新消息，则可能会发生这种情况。目前，我们无法停止线程执行：只要池存在，我们的线程就会继续执行。我们使用 `unwrap` 的原因是我们知道失败的情况不会发生，但编译器不知道。

但我们还没有完全完成！在 `Worker` 中，我们传递给 `thread::spawn` 的闭包仍然只 _引用_ 通道的接收端。相反，我们需要闭包永远循环，向通道的接收端请求作业并在获得作业时运行它。让我们对 `Worker::new` 进行 Listing 21-20 中所示的更改。

<Listing number="21-20" file-name="src/lib.rs" caption="在 `Worker` 实例的线程中接收并执行作业">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

在这里，我们首先调用 `receiver` 上的 `lock` 以获取互斥锁，然后我们调用 `unwrap` 以在出现任何错误时 panic。获取锁可能会失败，如果互斥锁处于 _中毒_ 状态，这可能发生在某个线程在持有锁时 panic 而不是释放锁的情况下。在这种情况下，调用 `unwrap` 让此线程 panic 是正确的操作。你可以将此 `unwrap` 更改为带有对你有意义的错误消息的 `expect`。

如果我们获得了互斥锁上的锁，我们调用 `recv` 从通道接收一个 `Job`。最后的 `unwrap` 也移过了这里的任何错误，如果持有发送者的线程已关闭，则可能会发生这种情况，类似于如果接收者关闭，`send` 方法返回 `Err` 的情况。

`recv` 的调用会阻塞，因此如果还没有作业，当前线程将等待直到有作业可用。`Mutex<T>` 确保一次只有一个 `Worker` 线程尝试请求作业。

我们的线程池现在处于工作状态！给它一个 `cargo run` 并发出一些请求：

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^

warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

成功！我们现在有一个异步执行连接的线程池。创建的线程永远不会超过四个，因此如果服务器收到大量请求，我们的系统不会过载。如果我们向 _/sleep_ 发出请求，服务器将能够通过让另一个线程运行它们来服务其他请求。

> 注意：如果你在多个浏览器窗口中同时打开 _/sleep_，它们可能会以五秒的间隔一次加载一个。一些 Web 浏览器会出于缓存原因顺序执行相同请求的多个实例。这个限制不是由我们的 Web 服务器引起的。

这是一个暂停并考虑如果我们使用 futures 而不是闭包来执行工作，Listing 21-18、21-19 和 21-20 中的代码会有什么不同的好时机。哪些类型会改变？方法签名会有什么不同，如果有的话？代码的哪些部分会保持不变？

在学习了第17章和第18章中的 `while let` 循环后，你可能想知道为什么我们没有将工作线程代码写成 Listing 21-21 中所示的样子。

<Listing number="21-21" file-name="src/lib.rs" caption="使用 `while let` 的 `Worker::new` 的替代实现">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-21/src/lib.rs:here}}
```

</Listing>

这段代码编译并运行，但不会导致所需的线程行为：慢请求仍然会导致其他请求等待处理。原因有些微妙：`Mutex` 结构没有公共的 `unlock` 方法，因为锁的所有权基于 `lock` 方法返回的 `LockResult<MutexGuard<T>>` 中的 `MutexGuard<T>` 的生命周期。在编译时，借用检查器可以强制执行规则，即除非我们持有锁，否则不能访问由 `Mutex` 保护的资源。然而，如果我们不注意 `MutexGuard<T>` 的生命周期，这种实现也可能导致锁被持有时间比预期长。

Listing 21-20 中使用 `let job = receiver.lock().unwrap().recv().unwrap();` 的代码有效，因为使用 `let`，等号右侧表达式中的任何临时值都会在 `let` 语句结束时立即丢弃。然而，`while let`（以及 `if let` 和 `match`）不会丢弃临时值，直到关联的块结束。在 Listing 21-21 中，锁在调用 `job()` 期间保持持有，这意味着其他 `Worker` 实例无法接收作业。

[creating-type-synonyms-with-type-aliases]: ch20-03-advanced-types.html#creating-type-synonyms-with-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[fn-traits]: ch13-01-closures.html#moving-captured-values-out-of-the-closure-and-the-fn-traits
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
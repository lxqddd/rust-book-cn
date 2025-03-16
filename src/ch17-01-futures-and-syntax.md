## Futures 和 Async 语法

Rust 中异步编程的关键元素是 _futures_ 和 Rust 的 `async` 和 `await` 关键字。

一个 _future_ 是一个可能现在还没有准备好，但在未来的某个时刻会准备好的值。（这个概念在许多语言中都有出现，有时使用其他名称，如 _task_ 或 _promise_。）Rust 提供了一个 `Future` trait 作为构建块，以便不同的异步操作可以使用不同的数据结构实现，但具有共同的接口。在 Rust 中，futures 是实现 `Future` trait 的类型。每个 future 都持有关于已经取得的进展以及“准备好”意味着什么的信息。

你可以将 `async` 关键字应用于代码块和函数，以指定它们可以被中断和恢复。在 async 块或 async 函数中，你可以使用 `await` 关键字来 _等待一个 future_（即等待它变为准备好）。在 async 块或函数中等待 future 的任何地方都是该 async 块或函数可以暂停和恢复的潜在点。检查 future 以查看其值是否可用的过程称为 _轮询_。

其他一些语言，如 C# 和 JavaScript，也使用 `async` 和 `await` 关键字进行异步编程。如果你熟悉这些语言，你可能会注意到 Rust 在处理方式上的一些显著差异，包括它如何处理语法。这是有充分理由的，我们稍后会看到！

在编写异步 Rust 时，我们大多数时候使用 `async` 和 `await` 关键字。Rust 将它们编译成使用 `Future` trait 的等效代码，就像它将 `for` 循环编译成使用 `Iterator` trait 的等效代码一样。由于 Rust 提供了 `Future` trait，你还可以在需要时为自己的数据类型实现它。我们将在本章中看到的许多函数返回具有自己 `Future` 实现的类型。我们将在本章末尾回到 trait 的定义，并深入探讨它的工作原理，但这些细节足以让我们继续前进。

这一切可能感觉有点抽象，所以让我们编写我们的第一个异步程序：一个小型网页抓取器。我们将从命令行传入两个 URL，并发地获取它们，并返回先完成的结果。这个例子会有一些新的语法，但不用担心——我们会逐步解释你需要知道的一切。

## 我们的第一个异步程序

为了将本章的重点放在学习异步而不是处理生态系统的各个部分上，我们创建了 `trpl` crate（`trpl` 是“The Rust Programming Language”的缩写）。它重新导出了你需要的所有类型、trait 和函数，主要来自 [`futures`][futures-crate]<!-- ignore --> 和 [`tokio`][tokio]<!-- ignore --> crate。`futures` crate 是 Rust 异步代码实验的官方场所，实际上 `Future` trait 最初就是在这里设计的。Tokio 是当今 Rust 中最广泛使用的异步运行时，尤其是对于 Web 应用程序。还有其他优秀的运行时，它们可能更适合你的需求。我们在 `trpl` 中使用 `tokio` crate，因为它经过了良好的测试并被广泛使用。

在某些情况下，`trpl` 还会重命名或包装原始 API，以让你专注于本章相关的细节。如果你想了解 crate 的功能，我们鼓励你查看 [其源代码][crate-source]<!-- ignore -->。你将能够看到每个重新导出的 crate 来自哪里，并且我们留下了大量注释来解释 crate 的功能。

创建一个名为 `hello-async` 的新二进制项目，并将 `trpl` crate 添加为依赖项：

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

现在我们可以使用 `trpl` 提供的各种组件来编写我们的第一个异步程序。我们将构建一个小型命令行工具，它获取两个网页，从每个网页中提取 `<title>` 元素，并打印出先完成整个过程的页面的标题。

### 定义 page_title 函数

让我们首先编写一个函数，它接受一个页面 URL 作为参数，向它发出请求，并返回标题元素的文本（见 Listing 17-1）。

<Listing number="17-1" file-name="src/main.rs" caption="定义一个异步函数以从 HTML 页面获取标题元素">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

首先，我们定义一个名为 `page_title` 的函数，并用 `async` 关键字标记它。然后我们使用 `trpl::get` 函数获取传入的 URL，并使用 `await` 关键字等待响应。为了获取响应的文本，我们调用它的 `text` 方法，并再次使用 `await` 关键字等待它。这两个步骤都是异步的。对于 `get` 函数，我们必须等待服务器发送回其响应的第一部分，其中包括 HTTP 头、cookie 等，并且可以与响应体分开传递。特别是如果响应体非常大，可能需要一些时间才能全部到达。因为我们必须等待 _整个_ 响应到达，所以 `text` 方法也是异步的。

我们必须显式地等待这两个 futures，因为 Rust 中的 futures 是 _惰性_ 的：它们在你使用 `await` 关键字要求它们之前不会做任何事情。（事实上，如果你不使用 future，Rust 会显示一个编译器警告。）这可能会让你想起第 13 章中关于迭代器的讨论，在 [使用迭代器处理一系列项目][iterators-lazy]<!-- ignore --> 部分。迭代器在你调用它们的 `next` 方法之前不会做任何事情——无论是直接调用还是通过使用 `for` 循环或 `map` 等方法间接调用。同样，futures 在你明确要求它们之前也不会做任何事情。这种惰性允许 Rust 避免在真正需要之前运行异步代码。

> 注意：这与我们在前一章中使用 `thread::spawn` 在 [使用 spawn 创建新线程][thread-spawn]<!--ignore--> 中看到的行为不同，在那里我们传递给另一个线程的闭包立即开始运行。这也与许多其他语言处理异步的方式不同。但对于 Rust 来说，能够提供其性能保证是很重要的，就像它对迭代器所做的那样。

一旦我们有了 `response_text`，我们就可以使用 `Html::parse` 将其解析为 `Html` 类型的实例。现在我们有了一个数据类型，可以用来将 HTML 作为更丰富的数据结构进行处理。特别是，我们可以使用 `select_first` 方法来查找给定 CSS 选择器的第一个实例。通过传递字符串 `"title"`，我们将获得文档中的第一个 `<title>` 元素（如果有的话）。因为可能没有任何匹配的元素，`select_first` 返回一个 `Option<ElementRef>`。最后，我们使用 `Option::map` 方法，它允许我们在 `Option` 中有项目时对其进行处理，如果没有则不做任何事情。（我们也可以在这里使用 `match` 表达式，但 `map` 更符合习惯。）在我们提供给 `map` 的函数体中，我们调用 `title_element` 的 `inner_html` 来获取其内容，这是一个 `String`。当一切完成后，我们得到了一个 `Option<String>`。

请注意，Rust 的 `await` 关键字位于你正在等待的表达式 _之后_，而不是之前。也就是说，它是一个 _后缀_ 关键字。如果你在其他语言中使用过 `async`，这可能会与你习惯的不同，但在 Rust 中，它使得方法链更加容易使用。因此，我们可以将 `page_url_for` 的函数体更改为将 `trpl::get` 和 `text` 函数调用与 `await` 链接在一起，如 Listing 17-2 所示。

<Listing number="17-2" file-name="src/main.rs" caption="使用 `await` 关键字进行链式调用">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

这样，我们就成功编写了我们的第一个异步函数！在我们添加一些代码到 `main` 中调用它之前，让我们再讨论一下我们编写的内容及其含义。

当 Rust 看到一个用 `async` 关键字标记的块时，它会将其编译成一个独特的、匿名的数据类型，该类型实现了 `Future` trait。当 Rust 看到一个用 `async` 标记的函数时，它会将其编译成一个非异步函数，其主体是一个异步块。异步函数的返回类型是编译器为该异步块创建的匿名数据类型的类型。

因此，编写 `async fn` 等同于编写一个返回 _future_ 的函数。对于编译器来说，像 Listing 17-1 中的 `async fn page_title` 这样的函数定义等同于一个非异步函数，定义如下：

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

让我们逐步了解转换后的版本的每个部分：

- 它使用了我们在第 10 章 [“Traits as Parameters”][impl-trait]<!-- ignore --> 部分讨论的 `impl Trait` 语法。
- 返回的 trait 是一个 `Future`，其关联类型为 `Output`。请注意，`Output` 类型是 `Option<String>`，这与 `async fn` 版本的 `page_title` 的原始返回类型相同。
- 原始函数体中调用的所有代码都包装在一个 `async move` 块中。记住，块是表达式。这个整个块是从函数返回的表达式。
- 这个异步块生成一个类型为 `Option<String>` 的值，如前所述。该值与返回类型中的 `Output` 类型匹配。这就像你见过的其他块一样。
- 新的函数体是一个 `async move` 块，因为它使用了 `url` 参数。（我们将在本章后面更多地讨论 `async` 与 `async move`。）

现在我们可以在 `main` 中调用 `page_title`。

## 确定单个页面的标题

首先，我们只获取一个页面的标题。在 Listing 17-3 中，我们遵循与第 12 章中相同的模式，在 [接受命令行参数][cli-args]<!-- ignore --> 部分获取命令行参数。然后我们将第一个 URL 传递给 `page_title` 并等待结果。因为 future 生成的值是一个 `Option<String>`，所以我们使用 `match` 表达式来打印不同的消息，以考虑页面是否有 `<title>`。

<Listing number="17-3" file-name="src/main.rs" caption="从 `main` 调用 `page_title` 函数，使用用户提供的参数">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

不幸的是，这段代码无法编译。我们唯一可以使用 `await` 关键字的地方是在异步函数或块中，而 Rust 不允许我们将特殊的 `main` 函数标记为 `async`。

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

`main` 不能标记为 `async` 的原因是异步代码需要一个 _运行时_：一个管理异步代码执行细节的 Rust crate。程序的 `main` 函数可以 _初始化_ 一个运行时，但它本身不是一个运行时。（我们稍后会看到更多关于为什么是这样的原因。）每个执行异步代码的 Rust 程序至少有一个地方设置运行时并执行 futures。

大多数支持异步的语言都捆绑了一个运行时，但 Rust 没有。相反，有许多不同的异步运行时可用，每个运行时都根据其目标用例做出不同的权衡。例如，具有许多 CPU 核心和大量 RAM 的高吞吐量 Web 服务器与具有单核、少量 RAM 且没有堆分配能力的微控制器有着非常不同的需求。提供这些运行时的 crate 通常还提供常见功能的异步版本，如文件或网络 I/O。

在这里，以及本章的其余部分，我们将使用 `trpl` crate 中的 `run` 函数，它接受一个 future 作为参数并运行它直到完成。在幕后，调用 `run` 会设置一个运行时，用于运行传入的 future。一旦 future 完成，`run` 返回 future 生成的任何值。

我们可以直接将 `page_title` 返回的 future 传递给 `run`，一旦它完成，我们就可以匹配生成的 `Option<String>`，就像我们在 Listing 17-3 中尝试做的那样。然而，对于本章中的大多数示例（以及现实世界中的大多数异步代码），我们将不仅仅做一个异步函数调用，所以我们将传递一个 `async` 块并显式地等待 `page_title` 调用的结果，如 Listing 17-4 所示。

<Listing number="17-4" caption="使用 `trpl::run` 等待异步块" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

当我们运行这段代码时，我们得到了最初预期的行为：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run https://www.rust-lang.org
# copy the output here
-->

```console
$ cargo run -- https://www.rust-lang.org
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

终于，我们有了一些可以工作的异步代码！但在我们添加代码来让两个站点竞争之前，让我们简要地回到 futures 的工作原理。

每个 _等待点_——即代码使用 `await` 关键字的地方——代表一个将控制权交还给运行时的位置。为了实现这一点，Rust 需要跟踪异步块中涉及的状态，以便运行时可以启动其他工作，然后在准备好时返回并尝试推进第一个工作。这是一个不可见的状态机，就像你编写了一个枚举来在每个等待点保存当前状态一样：

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

手动编写代码来在每种状态之间转换将是繁琐且容易出错的，特别是当你需要添加更多功能和更多状态时。幸运的是，Rust 编译器为异步代码自动创建并管理状态机数据结构。围绕数据结构的正常借用和所有权规则仍然适用，幸运的是，编译器还为我们处理这些检查并提供有用的错误消息。我们将在本章后面处理其中的一些。

最终，必须有东西执行这个状态机，而这个东西就是运行时。（这就是为什么你在查看运行时时可能会遇到 _执行器_ 的引用：执行器是运行时负责执行异步代码的部分。）

现在你可以看到为什么编译器在 Listing 17-3 中阻止我们将 `main` 本身变成一个异步函数。如果 `main` 是一个异步函数，那么其他东西需要管理 `main` 返回的 future 的状态机，但 `main` 是程序的起点！相反，我们在 `main` 中调用 `trpl::run` 函数来设置一个运行时，并运行 `async` 块返回的 future 直到它完成。

> 注意：一些运行时提供了宏，因此你可以 _编写_ 一个异步 `main` 函数。这些宏将 `async fn main() { ... }` 重写为一个普通的 `fn main`，它执行与我们在 Listing 17-5 中手动执行的相同操作：调用一个函数，以 `trpl::run` 的方式运行一个 future 直到完成。

现在让我们将这些部分放在一起，看看如何编写并发代码。

### 让我们的两个 URL 竞争

在 Listing 17-5 中，我们调用 `page_title` 并传入从命令行传入的两个不同 URL，让它们竞争。

<Listing number="17-5" caption="" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

我们首先为每个用户提供的 URL 调用 `page_title`。我们将生成的 futures 保存为 `title_fut_1` 和 `title_fut_2`。记住，这些 futures 现在还没有做任何事情，因为 futures 是惰性的，我们还没有等待它们。然后我们将 futures 传递给 `trpl::race`，它返回一个值来指示传递给它的 futures 中哪个先完成。

> 注意：在幕后，`race` 是建立在更通用的函数 `select` 之上的，你会在现实世界的 Rust 代码中更频繁地遇到它。`select` 函数可以做很多 `trpl::race` 函数无法做到的事情，但它也有一些我们目前可以跳过的额外复杂性。

任何一个 future 都可以合法地“获胜”，所以返回一个 `Result` 是没有意义的。相反，`race` 返回一个我们以前没有见过的类型，`trpl::Either`。`Either` 类型与 `Result` 有些相似，因为它有两个情况。与 `Result` 不同的是，`Either` 中没有成功或失败的概念。相反，它使用 `Left` 和 `Right` 来表示“一个或另一个”：

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

如果第一个参数获胜，`race` 函数返回 `Left` 并带有该 future 的输出；如果第二个 future 参数获胜，则返回 `Right` 并带有该 future 的输出。这与调用函数时参数的顺序相匹配：第一个参数在第二个参数的左边。

我们还更新了 `page_title` 以返回传入的相同 URL。这样，如果先返回的页面没有可解析的 `<title>`，我们仍然可以打印一个有意义的消息。有了这些信息，我们通过更新 `println!` 输出来指示哪个 URL 先完成以及该网页的 `<title>`（如果有的话）。

你现在已经构建了一个小型的工作网页抓取器！选择几个 URL 并运行命令行工具。你可能会发现某些站点始终比其他站点更快，而在其他情况下，较快的站点在每次运行时都会有所不同。更重要的是，你已经学会了使用 futures 的基础知识，所以现在我们可以更深入地探讨异步编程的更多内容。

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
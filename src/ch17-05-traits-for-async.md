## 深入探讨异步特性

<!-- 旧标题。不要删除，否则可能会破坏链接。 -->

<a id="digging-into-the-traits-for-async"></a>

在本章中，我们以各种方式使用了 `Future`、`Pin`、`Unpin`、`Stream` 和 `StreamExt` 特性。不过，到目前为止，我们避免深入探讨它们的工作原理或它们如何协同工作，这对于日常的 Rust 工作来说大多数情况下是没问题的。然而，有时你会遇到需要理解更多细节的情况。在本节中，我们将深入探讨这些细节，以帮助你在这些情况下解决问题，但仍然将 _真正_ 的深入探讨留给其他文档。

<!-- 旧标题。不要删除，否则可能会破坏链接。 -->

<a id="future"></a>

### `Future` 特性

让我们首先仔细看看 `Future` 特性的工作原理。以下是 Rust 对其的定义：

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

这个特性定义包含了一些新的类型以及一些我们之前没有见过的语法，所以让我们逐部分解析这个定义。

首先，`Future` 的关联类型 `Output` 表示 future 解析后的结果。这与 `Iterator` 特性的 `Item` 关联类型类似。其次，`Future` 还有一个 `poll` 方法，它为其 `self` 参数接受一个特殊的 `Pin` 引用，并接受一个对 `Context` 类型的可变引用，返回一个 `Poll<Self::Output>`。我们稍后会讨论 `Pin` 和 `Context`。现在，让我们关注这个方法返回的内容，即 `Poll` 类型：

```rust
enum Poll<T> {
    Ready(T),
    Pending,
}
```

这个 `Poll` 类型类似于 `Option`。它有一个带有值的变体 `Ready(T)`，以及一个没有值的变体 `Pending`。不过，`Poll` 的含义与 `Option` 大不相同！`Pending` 变体表示 future 仍有工作要做，因此调用者需要稍后再检查。`Ready` 变体表示 future 已完成其工作，并且 `T` 值可用。

> 注意：对于大多数 future，调用者不应在 future 返回 `Ready` 后再次调用 `poll`。许多 future 在准备就绪后再次轮询时会 panic。可以安全地再次轮询的 future 会在其文档中明确说明这一点。这与 `Iterator::next` 的行为类似。

当你看到使用 `await` 的代码时，Rust 在底层将其编译为调用 `poll` 的代码。如果你回顾一下 Listing 17-4，我们在其中打印出单个 URL 的页面标题，Rust 会将其编译为类似（尽管不完全相同）以下内容：

```rust,ignore
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    }
    Pending => {
        // 但这里应该放什么呢？
    }
}
```

当 future 仍然是 `Pending` 时，我们应该做什么？我们需要某种方式一次又一次地尝试，直到 future 最终准备就绪。换句话说，我们需要一个循环：

```rust,ignore
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match page_title {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        }
        Pending => {
            // 继续
        }
    }
}
```

如果 Rust 将其编译为完全相同的代码，那么每个 `await` 都会阻塞——这与我们的初衷完全相反！相反，Rust 确保循环可以将控制权交给可以暂停此 future 的工作以处理其他 future，然后再稍后检查此 future。正如我们所看到的，这个“东西”就是异步运行时，这种调度和协调工作是它的主要职责之一。

在本章的前面部分，我们描述了等待 `rx.recv` 的情况。`recv` 调用返回一个 future，而等待 future 会轮询它。我们注意到，运行时会在 future 准备好时暂停它，直到它准备好返回 `Some(message)` 或当通道关闭时返回 `None`。通过我们对 `Future` 特性（特别是 `Future::poll`）的深入理解，我们可以看到这是如何工作的。当 future 返回 `Poll::Pending` 时，运行时知道 future 尚未准备好。相反，当 `poll` 返回 `Poll::Ready(Some(message))` 或 `Poll::Ready(None)` 时，运行时知道 future _已经_ 准备好并推进它。

运行时的具体实现细节超出了本书的范围，但关键是要了解 future 的基本机制：运行时 _轮询_ 它负责的每个 future，当 future 尚未准备好时将其放回睡眠状态。

<!-- 旧标题。不要删除，否则可能会破坏链接。 -->

<a id="pinning-and-the-pin-and-unpin-traits"></a>

### `Pin` 和 `Unpin` 特性

当我们在 Listing 17-16 中引入 pinning 的概念时，我们遇到了一个非常棘手的错误消息。以下是该错误消息的相关部分：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo build
copy *only* the final `error` block from the errors
-->

```text
error[E0277]: `{async block@src/main.rs:10:23: 10:33}` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `{async block@src/main.rs:10:23: 10:33}`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<{async block@src/main.rs:10:23: 10:33}>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`
```

这个错误消息不仅告诉我们我们需要 pin 这些值，还解释了为什么需要 pinning。`trpl::join_all` 函数返回一个名为 `JoinAll` 的结构体。该结构体泛型化了一个类型 `F`，该类型被约束为实现 `Future` 特性。直接使用 `await` 等待 future 会隐式地 pin 该 future。这就是为什么我们不需要在我们想要等待 future 的地方到处使用 `pin!`。

然而，我们在这里并不是直接等待一个 future。相反，我们通过将一组 future 传递给 `join_all` 函数来构造一个新的 future `JoinAll`。`join_all` 的签名要求集合中的项的类型都实现 `Future` 特性，而 `Box<T>` 只有在它包装的 `T` 是实现 `Unpin` 特性的 future 时才实现 `Future`。

这需要消化很多内容！为了真正理解它，让我们进一步深入了解 `Future` 特性的实际工作原理，特别是围绕 _pinning_ 的部分。

再次查看 `Future` 特性的定义：

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    // 必需的方法
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

`cx` 参数及其 `Context` 类型是运行时实际知道何时检查任何给定 future 的关键，同时仍然保持惰性。同样，具体的工作原理超出了本章的范围，你通常只需要在编写自定义 `Future` 实现时考虑这一点。我们将重点关注 `self` 的类型，因为这是我们第一次看到 `self` 带有类型注解的方法。`self` 的类型注解与其他函数参数的类型注解类似，但有两个关键区别：

- 它告诉 Rust `self` 必须是什么类型才能调用该方法。

- 它不能是任何类型。它仅限于该方法实现的类型、对该类型的引用或智能指针，或者包装对该类型引用的 `Pin`。

我们将在 [第 18 章][ch-18]<!-- ignore --> 中看到更多关于这种语法的内容。现在，只需要知道如果我们想要轮询一个 future 以检查它是 `Pending` 还是 `Ready(Output)`，我们需要一个 `Pin` 包装的可变引用类型。

`Pin` 是指针类型（如 `&`、`&mut`、`Box` 和 `Rc`）的包装器。（技术上，`Pin` 适用于实现 `Deref` 或 `DerefMut` 特性的类型，但这实际上等同于仅适用于指针。）`Pin` 本身不是指针，也没有像 `Rc` 和 `Arc` 那样具有引用计数的行为；它纯粹是编译器用来强制执行指针使用约束的工具。

回想一下，`await` 是通过调用 `poll` 实现的，这开始解释了我们之前看到的错误消息，但那是关于 `Unpin` 的，而不是 `Pin`。那么 `Pin` 与 `Unpin` 究竟有什么关系，为什么 `Future` 需要 `self` 是 `Pin` 类型才能调用 `poll`？

记得本章前面提到的，future 中的一系列 await 点被编译成一个状态机，编译器确保该状态机遵循 Rust 的所有正常规则，包括借用和所有权。为了实现这一点，Rust 会查看从一个 await 点到下一个 await 点或 async 块结束之间需要哪些数据。然后它在编译后的状态机中创建相应的变体。每个变体都获得它需要的访问权限，以使用该部分源代码中的数据，无论是通过获取该数据的所有权还是通过获取对该数据的可变或不可变引用。

到目前为止，一切顺利：如果我们在给定的 async 块中关于所有权或引用有任何错误，借用检查器会告诉我们。当我们想要移动与该块对应的 future 时——比如将其移动到 `Vec` 中以传递给 `join_all`——事情就变得棘手了。

当我们移动一个 future 时——无论是通过将其推入数据结构以用作 `join_all` 的迭代器，还是通过从函数中返回它——这实际上意味着移动 Rust 为我们创建的状态机。与 Rust 中的大多数其他类型不同，Rust 为 async 块创建的 future 可能会在任何给定变体的字段中引用自身，如图 17-4 中的简化图示所示。

<figure>

<img alt="一个单列三行的表格，表示一个 future，fut1，前两行有数据值 0 和 1，第三行有一个箭头指向第二行，表示 future 内部的引用。" src="img/trpl17-04.svg" class="center" />

<figcaption>图 17-4：自引用数据类型。</figcaption>

</figure>

然而，默认情况下，任何具有对自身引用的对象在移动时都是不安全的，因为引用始终指向它们所引用的实际内存地址（见图 17-5）。如果你移动数据结构本身，这些内部引用将指向旧位置。然而，该内存位置现在无效。一方面，当你对数据结构进行更改时，它的值不会更新。另一方面——更重要的是——计算机现在可以自由地将该内存用于其他用途！你可能会在以后读取完全不相关的数据。

<figure>

<img alt="两个表格，描绘了两个 future，fut1 和 fut2，每个都有一个列和三行，表示将 future 从 fut1 移动到 fut2 的结果。第一个，fut1，被灰显，每个索引中有一个问号，表示未知的内存。第二个，fut2，第一行和第二行有 0 和 1，第三行有一个箭头指向 fut1 的第二行，表示一个指针，它引用了 future 移动前的旧内存位置。" src="img/trpl17-05.svg" class="center" />

<figcaption>图 17-5：移动自引用数据类型的不安全结果</figcaption>

</figure>

理论上，Rust 编译器可以尝试在每次移动对象时更新每个引用，但这可能会增加很多性能开销，特别是如果需要更新整个引用网络。如果我们能确保所讨论的数据结构 _不会在内存中移动_，我们就不必更新任何引用。这正是 Rust 的借用检查器所要求的：在安全代码中，它阻止你移动任何具有活动引用的项目。

`Pin` 在此基础上为我们提供了我们所需的保证。当我们通过将指向该值的指针包装在 `Pin` 中来 _pin_ 一个值时，它就不能再移动了。因此，如果你有 `Pin<Box<SomeType>>`，你实际上是在 pin `SomeType` 值，_而不是_ `Box` 指针。图 17-6 说明了这个过程。

<figure>

<img alt="三个并排排列的盒子。第一个标记为“Pin”，第二个标记为“b1”，第三个标记为“pinned”。在“pinned”中是一个标记为“fut”的表格，有一个列；它表示一个 future，每个数据结构部分都有一个单元格。它的第一个单元格有值“0”，第二个单元格有一个箭头指向第四个也是最后一个单元格，其中包含值“1”，第三个单元格有虚线和省略号，表示数据结构可能还有其他部分。总的来说，“fut”表格表示一个自引用的 future。一个箭头从标记为“Pin”的盒子出发，穿过标记为“b1”的盒子，终止在“pinned”盒子内的“fut”表格中。" src="img/trpl17-06.svg" class="center" />

<figcaption>图 17-6：Pin 一个指向自引用 future 类型的 `Box`。</figcaption>

</figure>

事实上，`Box` 指针仍然可以自由移动。记住：我们关心的是确保最终被引用的数据保持在原位。如果指针移动，_但它指向的数据在同一个位置_，如图 17-7 所示，就没有潜在的问题。作为一个独立的练习，查看这些类型的文档以及 `std::pin` 模块，并尝试弄清楚如何使用 `Pin` 包装 `Box` 来实现这一点。）关键是自引用类型本身不能移动，因为它仍然被 pin。

<figure>

<img alt="四个盒子大致排列成三列，与之前的图表相同，但第二列有变化。现在第二列有两个盒子，标记为“b1”和“b2”，“b1”被灰显，箭头从“Pin”穿过“b2”而不是“b1”，表示指针已从“b1”移动到“b2”，但“pinned”中的数据没有移动。" src="img/trpl17-07.svg" class="center" />

<figcaption>图 17-7：移动指向自引用 future 类型的 `Box`。</figcaption>

</figure>

然而，大多数类型在移动时是完全安全的，即使它们恰好位于 `Pin` 包装器后面。我们只需要在项目具有内部引用时考虑 pinning。原始值（如数字和布尔值）是安全的，因为它们显然没有任何内部引用。你在 Rust 中通常使用的大多数类型也是如此。例如，你可以移动 `Vec` 而不必担心。根据我们目前所看到的，如果你有一个 `Pin<Vec<String>>`，你必须通过 `Pin` 提供的安全但限制性的 API 来完成所有操作，即使 `Vec<String>` 在没有其他引用的情况下总是可以安全移动。我们需要一种方法来告诉编译器在这种情况下移动项目是没问题的——这就是 `Unpin` 的用武之地。

`Unpin` 是一个标记特性，类似于我们在第 16 章中看到的 `Send` 和 `Sync` 特性，因此它本身没有任何功能。标记特性仅用于告诉编译器在特定上下文中使用实现给定特性的类型是安全的。`Unpin` 通知编译器给定类型 _不需要_ 维护有关该值是否可以安全移动的任何保证。

<!--
  下一个块中的内联 `<code>` 是为了允许其中的内联 `<em>`，
  匹配 NoStarch 的风格，并在此处的文本中强调它与普通类型不同。
-->

与 `Send` 和 `Sync` 一样，编译器会自动为所有可以证明安全的类型实现 `Unpin`。一个特殊情况，再次类似于 `Send` 和 `Sync`，是 `Unpin` _不_ 为某个类型实现的情况。表示这种情况的符号是 <code>impl !Unpin for <em>SomeType</em></code>，其中 <code><em>SomeType</em></code> 是需要维护这些保证以在 `Pin` 中使用指向该类型的指针时保持安全的类型的名称。

换句话说，关于 `Pin` 和 `Unpin` 之间的关系，有两件事需要记住。首先，`Unpin` 是“正常”情况，而 `!Unpin` 是特殊情况。其次，类型是否实现 `Unpin` 或 `!Unpin` _仅_ 在你使用像 <code>Pin<&mut <em>SomeType</em>></code> 这样的 pinned 指针时才重要。

为了具体说明这一点，考虑一个 `String`：它有长度和组成它的 Unicode 字符。我们可以将 `String` 包装在 `Pin` 中，如图 17-8 所示。然而，`String` 会自动实现 `Unpin`，Rust 中的大多数其他类型也是如此。

<figure>

<img alt="并发工作流" src="img/trpl17-08.svg" class="center" />

<figcaption>图 17-8：Pin 一个 `String`；虚线表示 `String` 实现了 `Unpin` 特性，因此没有被 pin。</figcaption>

</figure>

因此，我们可以做一些如果 `String` 实现 `!Unpin` 而不是 `Unpin` 时是非法的操作，例如在内存中的完全相同的位置用一个字符串替换另一个字符串，如图 17-9 所示。这不会违反 `Pin` 的契约，因为 `String` 没有任何使其在移动时不安全的内部引用！这正是为什么它实现 `Unpin` 而不是 `!Unpin`。

<figure>

<img alt="并发工作流" src="img/trpl17-09.svg" class="center" />

<figcaption>图 17-9：在内存中用完全不同的 `String` 替换 `String`。</figcaption>

</figure>

现在我们已经了解了足够多的内容，可以理解 Listing 17-17 中 `join_all` 调用报告的错误。我们最初尝试将 async 块生成的 future 移动到 `Vec<Box<dyn Future<Output = ()>>>` 中，但正如我们所看到的，这些 future 可能具有内部引用，因此它们不实现 `Unpin`。它们需要被 pin，然后我们可以将 `Pin` 类型传递给 `Vec`，确信 future 中的底层数据 _不会_ 被移动。

`Pin` 和 `Unpin` 主要用于构建低级库，或者当你构建运行时本身时，而不是用于日常的 Rust 代码。然而，当你在错误消息中看到这些特性时，现在你将更好地了解如何修复你的代码！

> 注意：`Pin` 和 `Unpin` 的这种组合使得在 Rust 中安全地实现一类复杂的类型成为可能，否则这些类型将因为自引用而变得具有挑战性。需要 `Pin` 的类型在今天的异步 Rust 中最常见，但偶尔你可能也会在其他上下文中看到它们。
>
> `Pin` 和 `Unpin` 的具体工作原理以及它们需要遵守的规则在 `std::pin` 的 API 文档中有详细说明，所以如果你有兴趣了解更多，这是一个很好的起点。
>
> 如果你想更深入地了解底层工作原理，请参阅 [_Asynchronous Programming in Rust_][async-book] 的 [第 2 章][under-the-hood] 和 [第 4 章][pinning]。

### `Stream` 特性

现在你已经对 `Future`、`Pin` 和 `Unpin` 特性有了更深入的理解，我们可以将注意力转向 `Stream` 特性。正如你在本章前面学到的，stream 类似于异步迭代器。然而，与 `Iterator` 和 `Future` 不同，`Stream` 在撰写本文时还没有在标准库中定义，但 `futures` crate 中有一个非常常见的定义在整个生态系统中使用。

让我们在查看 `Stream` 特性如何将它们结合在一起之前，回顾一下 `Iterator` 和 `Future` 特性的定义。从 `Iterator` 中，我们有序列的概念：它的 `next` 方法提供了一个 `Option<Self::Item>`。从 `Future` 中，我们有随时间准备就绪的概念：它的 `poll` 方法提供了一个 `Poll<Self::Output>`。为了表示随时间准备就绪的项目序列，我们定义了一个 `Stream` 特性，将这些特性结合在一起：

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

`Stream` 特性定义了一个名为 `Item` 的关联类型，表示 stream 生成的项目的类型。这与 `Iterator` 类似，其中可能有零到多个项目，而与 `Future` 不同，后者总是有一个 `Output`，即使它是单元类型 `()`。

`Stream` 还定义了一个方法来获取这些项目。我们称之为 `poll_next`，以明确它像 `Future::poll` 一样进行轮询，并像 `Iterator::next` 一样生成一系列项目。它的返回类型结合了 `Poll` 和 `Option`。外部类型是 `Poll`，因为它必须像 future 一样检查是否准备就绪。内部类型是 `Option`，因为它需要像迭代器一样发出信号，表示是否有更多消息。

类似于这个定义的内容很可能会成为 Rust 标准库的一部分。与此同时，它是大多数运行时工具包的一部分，因此你可以依赖它，我们接下来介绍的所有内容通常都适用！

在我们看到的流式处理部分的示例中，我们没有使用 `poll_next` _或_ `Stream`，而是使用了 `next` 和 `StreamExt`。当然，我们可以通过手动编写自己的 `Stream` 状态机直接使用 `poll_next` API，就像我们可以通过 `poll` 方法直接使用 future 一样。不过，使用 `await` 要好得多，而 `StreamExt` 特性提供了 `next` 方法，因此我们可以这样做：

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-stream-ext/src/lib.rs:here}}
```

<!--
TODO: 如果/当 tokio/etc. 更新其 MSRV 并切换到在特性中使用 async 函数时，更新此内容，
因为缺乏这些是它们尚未拥有此功能的原因。
-->

> 注意：我们在本章前面使用的实际定义看起来与此略有不同，因为它支持尚未支持在特性中使用 async 函数的 Rust 版本。因此，它看起来像这样：
>
> ```rust,ignore
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> 这个 `Next` 类型是一个实现 `Future` 的 `struct`，并允许我们使用 `Next<'_, Self>` 命名对 `self` 的引用的生命周期，以便 `await` 可以使用此方法。

`StreamExt` 特性也是所有有趣方法的家园，这些方法可用于流。`StreamExt` 会自动为每个实现 `Stream` 的类型实现，但这些特性是分开定义的，以便社区可以在不影响基础特性的情况下迭代便利 API。

在 `trpl` crate 中使用的 `StreamExt` 版本中，该特性不仅定义了 `next` 方法，还提供了 `next` 的默认实现，该实现正确处理了调用 `Stream::poll_next` 的细节。这意味着即使你需要编写自己的流式数据类型，你 _只需要_ 实现 `Stream`，然后任何使用你的数据类型的人都可以自动使用 `StreamExt` 及其方法。

这就是我们将要介绍的这些特性的低级细节的全部内容。总结一下，让我们考虑一下 future（包括 stream）、任务和线程是如何协同工作的！

[ch-18]: ch18-00-oop.html
[async-book]: https://rust-lang.github.io/async-book/
[under-the-hood]: https://rust-lang.github.io/async-book/02_execution/01_chapter.html
[pinning]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
[first-async]: ch17-01-futures-and-syntax.html#our-first-async-program
[any-number-futures]: ch17-03-more-futures.html#working-with-any-number-of-futures
## 使用 `Send` 和 `Sync` Trait 实现可扩展的并发性

<!-- 旧链接，不要移除 -->

<a id="extensible-concurrency-with-the-sync-and-send-traits"></a>

有趣的是，到目前为止，我们在本章中讨论的几乎所有并发特性都是标准库的一部分，而不是语言本身。处理并发的选择不仅限于语言或标准库；你可以编写自己的并发特性，或者使用其他人编写的特性。

然而，嵌入在语言中而不是标准库中的关键并发概念是 `std::marker` trait `Send` 和 `Sync`。

### 使用 `Send` 允许在线程之间转移所有权

`Send` 标记 trait 表示实现了 `Send` 的类型的值的所有权可以在线程之间转移。几乎所有的 Rust 类型都是 `Send`，但有一些例外，包括 `Rc<T>`：它不能实现 `Send`，因为如果你克隆了一个 `Rc<T>` 值并尝试将克隆的所有权转移到另一个线程，两个线程可能会同时更新引用计数。因此，`Rc<T>` 被实现用于单线程场景，以避免线程安全的性能开销。

因此，Rust 的类型系统和 trait 约束确保你不会意外地不安全地将 `Rc<T>` 值跨线程发送。当我们在 Listing 16-14 中尝试这样做时，我们得到了错误 `the trait Send is not implemented for Rc<Mutex<i32>>`。当我们切换到实现了 `Send` 的 `Arc<T>` 时，代码编译通过。

任何完全由 `Send` 类型组成的类型也会自动标记为 `Send`。几乎所有原始类型都是 `Send`，除了裸指针，我们将在第 20 章讨论。

### 使用 `Sync` 允许多线程访问

`Sync` 标记 trait 表示实现了 `Sync` 的类型可以安全地从多个线程引用。换句话说，如果 `&T`（对 `T` 的不可变引用）实现了 `Send`，那么任何类型 `T` 都实现了 `Sync`，这意味着引用可以安全地发送到另一个线程。与 `Send` 类似，所有原始类型都实现了 `Sync`，完全由实现了 `Sync` 的类型组成的类型也实现了 `Sync`。

智能指针 `Rc<T>` 也不实现 `Sync`，原因与它不实现 `Send` 相同。`RefCell<T>` 类型（我们在第 15 章讨论过）及其相关的 `Cell<T>` 类型家族也不实现 `Sync`。`RefCell<T>` 在运行时进行的借用检查实现不是线程安全的。智能指针 `Mutex<T>` 实现了 `Sync`，可以用于在多个线程之间共享访问，正如你在 [“Sharing a `Mutex<T>` Between Multiple Threads”][sharing-a-mutext-between-multiple-threads]<!-- ignore --> 中看到的那样。

### 手动实现 `Send` 和 `Sync` 是不安全的

因为完全由实现了 `Send` 和 `Sync` trait 的其他类型组成的类型也会自动实现 `Send` 和 `Sync`，所以我们不必手动实现这些 trait。作为标记 trait，它们甚至没有任何方法需要实现。它们只是用于强制执行与并发相关的不变量。

手动实现这些 trait 涉及到实现不安全的 Rust 代码。我们将在第 20 章讨论使用不安全的 Rust 代码；现在，重要的信息是，构建不由 `Send` 和 `Sync` 部分组成的新的并发类型需要仔细思考以维护安全保证。[“The Rustonomicon”][nomicon] 提供了更多关于这些保证以及如何维护它们的信息。

## 总结

这并不是你在本书中最后一次看到并发：下一章将专注于异步编程，而第 21 章的项目将在一个比这里讨论的小例子更现实的情况下使用本章的概念。

如前所述，由于 Rust 处理并发的方式很少是语言的一部分，许多并发解决方案都是作为 crate 实现的。这些解决方案比标准库发展得更快，因此请务必在线搜索当前最先进的 crate 以用于多线程场景。

Rust 标准库提供了用于消息传递的通道和智能指针类型，如 `Mutex<T>` 和 `Arc<T>`，它们在并发上下文中使用是安全的。类型系统和借用检查器确保使用这些解决方案的代码不会出现数据竞争或无效引用。一旦你的代码编译通过，你可以放心它将在多个线程上愉快地运行，而不会出现其他语言中常见的难以追踪的错误。并发编程不再是一个令人恐惧的概念：勇敢地前进，让你的程序并发起来吧！

[sharing-a-mutext-between-multiple-threads]: ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads
[nomicon]: ../nomicon/index.html
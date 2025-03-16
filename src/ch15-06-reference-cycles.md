## 引用循环会导致内存泄漏

Rust 的内存安全保证使得意外创建永远不会被清理的内存（称为 _内存泄漏_）变得困难，但并非不可能。完全防止内存泄漏并不是 Rust 的保证之一，这意味着 Rust 中的内存泄漏是内存安全的。我们可以通过使用 `Rc<T>` 和 `RefCell<T>` 看到 Rust 允许内存泄漏：可以创建循环引用，其中项目相互引用。这会导致内存泄漏，因为循环中每个项目的引用计数永远不会达到 0，因此这些值永远不会被丢弃。

### 创建引用循环

让我们看看引用循环是如何发生的以及如何防止它，从 `List` 枚举的定义和 `tail` 方法开始，如 Listing 15-25 所示。

<Listing number="15-25" file-name="src/main.rs" caption="一个包含 `RefCell<T>` 的 cons list 定义，以便我们可以修改 `Cons` 变体所引用的内容">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs}}
```

</Listing>

我们使用了 Listing 15-5 中 `List` 定义的另一个变体。`Cons` 变体中的第二个元素现在是 `RefCell<Rc<List>>`，这意味着我们希望在 `Cons` 变体中修改 `List` 值所指向的内容，而不是像在 Listing 15-24 中那样修改 `i32` 值。我们还添加了一个 `tail` 方法，以便在拥有 `Cons` 变体时方便地访问第二个项目。

在 Listing 15-26 中，我们添加了一个 `main` 函数，它使用了 Listing 15-25 中的定义。这段代码在 `a` 中创建了一个列表，并在 `b` 中创建了一个指向 `a` 中列表的列表。然后它修改 `a` 中的列表以指向 `b`，从而创建了一个引用循环。在这个过程中，有 `println!` 语句显示各个点的引用计数。

<Listing number="15-26" file-name="src/main.rs" caption="创建两个 `List` 值相互指向的引用循环">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

</Listing>

我们创建了一个 `Rc<List>` 实例，其中包含一个 `List` 值，存储在变量 `a` 中，初始列表为 `5, Nil`。然后我们创建了另一个 `Rc<List>` 实例，其中包含另一个 `List` 值，存储在变量 `b` 中，该值包含 `10` 并指向 `a` 中的列表。

我们修改 `a` 使其指向 `b` 而不是 `Nil`，从而创建了一个循环。我们通过使用 `tail` 方法获取 `a` 中 `RefCell<Rc<List>>` 的引用，并将其存储在变量 `link` 中。然后我们使用 `RefCell<Rc<List>>` 上的 `borrow_mut` 方法将内部的值从包含 `Nil` 值的 `Rc<List>` 更改为 `b` 中的 `Rc<List>`。

当我们运行这段代码时，暂时将最后一个 `println!` 注释掉，我们将得到以下输出：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

在我们修改 `a` 中的列表以指向 `b` 之后，`a` 和 `b` 中的 `Rc<List>` 实例的引用计数都为 2。在 `main` 函数的末尾，Rust 丢弃了变量 `b`，这将 `b` 的 `Rc<List>` 实例的引用计数从 2 减少到 1。此时，`Rc<List>` 在堆上的内存不会被丢弃，因为它的引用计数是 1，而不是 0。然后 Rust 丢弃 `a`，这将 `a` 的 `Rc<List>` 实例的引用计数从 2 减少到 1。这个实例的内存也不能被丢弃，因为另一个 `Rc<List>` 实例仍然引用它。分配给列表的内存将永远无法被回收。为了可视化这个引用循环，我们创建了图 15-4 中的图表。

<img alt="Reference cycle of lists" src="img/trpl15-04.svg" class="center" />

<span class="caption">图 15-4: 列表 `a` 和 `b` 相互指向的引用循环</span>

如果你取消注释最后一个 `println!` 并运行程序，Rust 将尝试打印这个循环，`a` 指向 `b`，`b` 指向 `a`，依此类推，直到栈溢出。

与真实世界的程序相比，在这个示例中创建引用循环的后果并不是非常严重：在我们创建引用循环后，程序立即结束。然而，如果一个更复杂的程序在循环中分配了大量内存并长时间持有它，程序将使用比所需更多的内存，并可能使系统不堪重负，导致可用内存耗尽。

创建引用循环并不容易，但也不是不可能的。如果你有包含 `Rc<T>` 值的 `RefCell<T>` 值或类似的内部可变性和引用计数的嵌套组合类型，你必须确保不会创建循环；你不能依赖 Rust 来捕获它们。创建引用循环将是程序中的逻辑错误，你应该使用自动化测试、代码审查和其他软件开发实践来最小化这种错误。

另一种避免引用循环的解决方案是重新组织你的数据结构，使某些引用表示所有权，而某些引用不表示所有权。因此，你可以拥有由一些所有权关系和一些非所有权关系组成的循环，只有所有权关系会影响值是否可以被丢弃。在 Listing 15-25 中，我们总是希望 `Cons` 变体拥有它们的列表，因此重新组织数据结构是不可能的。让我们看一个使用由父节点和子节点组成的图的示例，看看非所有权关系何时是防止引用循环的适当方式。

<!-- Old link, do not remove -->

<a id="preventing-reference-cycles-turning-an-rct-into-a-weakt"></a>

### 使用 `Weak<T>` 防止引用循环

到目前为止，我们已经展示了调用 `Rc::clone` 会增加 `Rc<T>` 实例的 `strong_count`，并且只有当 `strong_count` 为 0 时，`Rc<T>` 实例才会被清理。你还可以通过调用 `Rc::downgrade` 并传递一个 `Rc<T>` 的引用来创建对 `Rc<T>` 实例中值的 _弱引用_。强引用是你共享 `Rc<T>` 实例所有权的方式。弱引用不表示所有权关系，它们的计数不会影响 `Rc<T>` 实例何时被清理。它们不会导致引用循环，因为一旦涉及的值的强引用计数为 0，任何涉及一些弱引用的循环都会被打破。

当你调用 `Rc::downgrade` 时，你会得到一个类型为 `Weak<T>` 的智能指针。调用 `Rc::downgrade` 不会将 `Rc<T>` 实例中的 `strong_count` 增加 1，而是将 `weak_count` 增加 1。`Rc<T>` 类型使用 `weak_count` 来跟踪有多少 `Weak<T>` 引用存在，类似于 `strong_count`。不同的是，`weak_count` 不需要为 0 才能清理 `Rc<T>` 实例。

因为 `Weak<T>` 引用的值可能已经被丢弃，所以要对 `Weak<T>` 指向的值做任何事情，你必须确保该值仍然存在。通过调用 `Weak<T>` 实例上的 `upgrade` 方法来实现这一点，该方法将返回一个 `Option<Rc<T>>`。如果 `Rc<T>` 值尚未被丢弃，你将得到一个 `Some` 结果；如果 `Rc<T>` 值已被丢弃，你将得到一个 `None` 结果。因为 `upgrade` 返回一个 `Option<Rc<T>>`，Rust 将确保处理 `Some` 和 `None` 的情况，并且不会出现无效指针。

作为一个示例，我们将创建一个树，其项目知道它们的子项目 _和_ 它们的父项目，而不是使用一个项目只知道下一个项目的列表。

#### 创建一个树数据结构：一个带有子节点的 `Node`

首先，我们将构建一个树，其中的节点知道它们的子节点。我们将创建一个名为 `Node` 的结构体，它持有自己的 `i32` 值以及对其子节点 `Node` 值的引用：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

我们希望 `Node` 拥有其子节点，并且我们希望与变量共享该所有权，以便我们可以直接访问树中的每个 `Node`。为此，我们将 `Vec<T>` 项定义为 `Rc<Node>` 类型的值。我们还希望修改哪些节点是另一个节点的子节点，因此我们在 `children` 中有一个 `RefCell<T>` 包裹着 `Vec<Rc<Node>>`。

接下来，我们将使用我们的结构体定义并创建一个名为 `leaf` 的 `Node` 实例，其值为 `3` 且没有子节点，以及另一个名为 `branch` 的实例，其值为 `5` 且 `leaf` 作为其子节点之一，如 Listing 15-27 所示。

<Listing number="15-27" file-name="src/main.rs" caption="创建一个没有子节点的 `leaf` 节点和一个以 `leaf` 作为其子节点之一的 `branch` 节点">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

</Listing>

我们克隆 `leaf` 中的 `Rc<Node>` 并将其存储在 `branch` 中，这意味着 `leaf` 中的 `Node` 现在有两个所有者：`leaf` 和 `branch`。我们可以通过 `branch.children` 从 `branch` 访问 `leaf`，但没有办法从 `leaf` 访问 `branch`。原因是 `leaf` 没有对 `branch` 的引用，也不知道它们有关系。我们希望 `leaf` 知道 `branch` 是它的父节点。我们接下来会这样做。

#### 从子节点添加对父节点的引用

为了使子节点知道其父节点，我们需要在我们的 `Node` 结构体定义中添加一个 `parent` 字段。问题在于决定 `parent` 的类型应该是什么。我们知道它不能包含 `Rc<T>`，因为这将创建一个引用循环，`leaf.parent` 指向 `branch`，`branch.children` 指向 `leaf`，这将导致它们的 `strong_count` 值永远不会为 0。

从另一个角度考虑关系，父节点应该拥有其子节点：如果父节点被丢弃，其子节点也应该被丢弃。然而，子节点不应该拥有其父节点：如果我们丢弃一个子节点，父节点应该仍然存在。这是一个弱引用的情况！

因此，我们将 `parent` 的类型改为使用 `Weak<T>`，具体来说是一个 `RefCell<Weak<Node>>`。现在我们的 `Node` 结构体定义如下：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

一个节点将能够引用其父节点，但不拥有其父节点。在 Listing 15-28 中，我们更新 `main` 以使用这个新定义，以便 `leaf` 节点将有一种方式引用其父节点 `branch`。

<Listing number="15-28" file-name="src/main.rs" caption="一个 `leaf` 节点，具有对其父节点 `branch` 的弱引用">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

</Listing>

创建 `leaf` 节点看起来与 Listing 15-27 类似，除了 `parent` 字段：`leaf` 开始时没有父节点，因此我们创建一个新的空的 `Weak<Node>` 引用实例。

此时，当我们尝试通过使用 `upgrade` 方法获取 `leaf` 的父节点的引用时，我们得到一个 `None` 值。我们在第一个 `println!` 语句的输出中看到这一点：

```text
leaf parent = None
```

当我们创建 `branch` 节点时，它也会在 `parent` 字段中有一个新的 `Weak<Node>` 引用，因为 `branch` 没有父节点。我们仍然将 `leaf` 作为 `branch` 的子节点之一。一旦我们有了 `branch` 中的 `Node` 实例，我们就可以修改 `leaf` 以赋予它一个 `Weak<Node>` 引用指向其父节点。我们使用 `leaf` 的 `parent` 字段中的 `RefCell<Weak<Node>>` 上的 `borrow_mut` 方法，然后我们使用 `Rc::downgrade` 函数从 `branch` 中的 `Rc<Node>` 创建一个 `Weak<Node>` 引用指向 `branch`。

当我们再次打印 `leaf` 的父节点时，这次我们将得到一个 `Some` 变体，其中包含 `branch`：现在 `leaf` 可以访问其父节点了！当我们打印 `leaf` 时，我们也避免了像在 Listing 15-26 中那样最终导致栈溢出的循环；`Weak<Node>` 引用被打印为 `(Weak)`：

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

没有无限输出表明这段代码没有创建引用循环。我们还可以通过查看调用 `Rc::strong_count` 和 `Rc::weak_count` 得到的值来判断这一点。

#### 可视化 `strong_count` 和 `weak_count` 的变化

让我们通过创建一个新的内部作用域并将 `branch` 的创建移到该作用域中，来看看 `Rc<Node>` 实例的 `strong_count` 和 `weak_count` 值是如何变化的。通过这样做，我们可以看到当 `branch` 被创建并在它超出作用域时被丢弃时会发生什么。修改如 Listing 15-29 所示。

<Listing number="15-29" file-name="src/main.rs" caption="在内部作用域中创建 `branch` 并检查强引用和弱引用计数">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

</Listing>

在 `leaf` 创建后，它的 `Rc<Node>` 的强引用计数为 1，弱引用计数为 0。在内部作用域中，我们创建 `branch` 并将其与 `leaf` 关联，此时当我们打印计数时，`branch` 中的 `Rc<Node>` 的强引用计数为 1，弱引用计数为 1（因为 `leaf.parent` 指向 `branch`，使用 `Weak<Node>`）。当我们打印 `leaf` 的计数时，我们将看到它的强引用计数为 2，因为 `branch` 现在有一个存储在 `branch.children` 中的 `leaf` 的 `Rc<Node>` 的克隆，但弱引用计数仍为 0。

当内部作用域结束时，`branch` 超出作用域，`Rc<Node>` 的强引用计数减少到 0，因此它的 `Node` 被丢弃。`leaf.parent` 的弱引用计数为 1 并不影响 `Node` 是否被丢弃，因此我们不会出现任何内存泄漏！

如果我们在作用域结束后尝试访问 `leaf` 的父节点，我们将再次得到 `None`。在程序结束时，`leaf` 中的 `Rc<Node>` 的强引用计数为 1，弱引用计数为 0，因为变量 `leaf` 现在再次是 `Rc<Node>` 的唯一引用。

所有管理计数和值丢弃的逻辑都内置在 `Rc<T>` 和 `Weak<T>` 以及它们的 `Drop` trait 实现中。通过在 `Node` 的定义中指定从子节点到父节点的关系应该是 `Weak<T>` 引用，你能够拥有父节点指向子节点和子节点指向父节点的关系，而不会创建引用循环和内存泄漏。

## 总结

本章介绍了如何使用智能指针来做出与 Rust 默认使用常规引用不同的保证和权衡。`Box<T>` 类型具有已知的大小并指向堆上分配的数据。`Rc<T>` 类型跟踪堆上数据的引用计数，以便数据可以有多个所有者。`RefCell<T>` 类型通过其内部可变性为我们提供了一个类型，当我们需要一个不可变类型但需要更改该类型的内部值时可以使用它；它还在运行时而不是编译时强制执行借用规则。

还讨论了 `Deref` 和 `Drop` trait，它们启用了智能指针的许多功能。我们探讨了可能导致内存泄漏的引用循环以及如何使用 `Weak<T>` 防止它们。

如果本章引起了你的兴趣，并且你想实现自己的智能指针，请查看 [“The Rustonomicon”][nomicon] 以获取更多有用的信息。

接下来，我们将讨论 Rust 中的并发性。你甚至会学到一些新的智能指针。

[nomicon]: ../nomicon/index.html
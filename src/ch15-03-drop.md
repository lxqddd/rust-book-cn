## 使用 `Drop` Trait 在清理时运行代码

智能指针模式的第二个重要 trait 是 `Drop`，它允许你自定义当一个值即将离开作用域时会发生什么。你可以为任何类型实现 `Drop` trait，并且该代码可以用于释放文件或网络连接等资源。

我们在智能指针的上下文中介绍 `Drop`，因为在实现智能指针时几乎总是会用到 `Drop` trait 的功能。例如，当 `Box<T>` 被丢弃时，它将释放该 box 指向的堆上的空间。

在某些语言中，对于某些类型，程序员必须在每次使用完这些类型的实例后调用代码来释放内存或资源。例如，文件句柄、套接字和锁。如果他们忘记了，系统可能会过载并崩溃。在 Rust 中，你可以指定每当一个值离开作用域时运行特定的代码，编译器会自动插入这段代码。因此，你不需要小心地在程序中每个特定类型实例使用完毕的地方放置清理代码——你仍然不会泄漏资源！

你通过实现 `Drop` trait 来指定当一个值离开作用域时要运行的代码。`Drop` trait 要求你实现一个名为 `drop` 的方法，该方法接受一个对 `self` 的可变引用。为了查看 Rust 何时调用 `drop`，我们现在用 `println!` 语句来实现 `drop`。

Listing 15-14 展示了一个 `CustomSmartPointer` 结构体，其唯一的自定义功能是当实例离开作用域时会打印 `Dropping CustomSmartPointer!`，以展示 Rust 何时运行 `drop` 方法。

<Listing number="15-14" file-name="src/main.rs" caption="一个实现了 `Drop` trait 的 `CustomSmartPointer` 结构体，我们将在其中放置清理代码">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

</Listing>

`Drop` trait 包含在预导入模块中，所以我们不需要手动引入作用域。我们在 `CustomSmartPointer` 上实现了 `Drop` trait，并为 `drop` 方法提供了一个调用 `println!` 的实现。`drop` 方法的主体是你希望在类型的实例离开作用域时运行的任何逻辑。我们在这里打印一些文本来直观地展示 Rust 何时调用 `drop`。

在 `main` 函数中，我们创建了两个 `CustomSmartPointer` 的实例，然后打印 `CustomSmartPointers created`。在 `main` 函数的末尾，我们的 `CustomSmartPointer` 实例将离开作用域，Rust 将调用我们放在 `drop` 方法中的代码，打印出最终的消息。注意，我们不需要显式调用 `drop` 方法。

当我们运行这个程序时，我们将看到以下输出：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

Rust 在我们的实例离开作用域时自动为我们调用了 `drop`，调用了我们指定的代码。变量以它们创建的顺序相反的顺序被丢弃，所以 `d` 在 `c` 之前被丢弃。这个例子的目的是给你一个关于 `drop` 方法如何工作的直观指南；通常你会指定你的类型需要运行的清理代码，而不是打印消息。

<!-- 旧链接，不要删除 -->

<a id="dropping-a-value-early-with-std-mem-drop"></a>

不幸的是，禁用自动 `drop` 功能并不简单。通常不需要禁用 `drop`；`Drop` trait 的整个意义在于它是自动处理的。然而，有时你可能希望提前清理一个值。一个例子是当你使用管理锁的智能指针时：你可能希望强制调用释放锁的 `drop` 方法，以便同一作用域中的其他代码可以获取锁。Rust 不允许你手动调用 `Drop` trait 的 `drop` 方法；相反，如果你想强制在作用域结束之前丢弃一个值，你必须调用标准库提供的 `std::mem::drop` 函数。

如果我们尝试通过修改 Listing 15-14 中的 `main` 函数来手动调用 `Drop` trait 的 `drop` 方法，如 Listing 15-15 所示，我们将得到一个编译器错误。

<Listing number="15-15" file-name="src/main.rs" caption="尝试手动调用 `Drop` trait 的 `drop` 方法以提前清理">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

</Listing>

当我们尝试编译这段代码时，我们将得到以下错误：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

这个错误消息指出我们不允许显式调用 `drop`。错误消息使用了术语 _destructor_，这是清理实例的函数的通用编程术语。_destructor_ 类似于创建实例的 _constructor_。Rust 中的 `drop` 函数是一个特定的 destructor。

Rust 不允许我们显式调用 `drop`，因为 Rust 仍然会在 `main` 结束时自动调用 `drop`。这将导致 _double free_ 错误，因为 Rust 将尝试两次清理同一个值。

我们不能禁用当一个值离开作用域时自动插入的 `drop`，也不能显式调用 `drop` 方法。因此，如果我们需要强制提前清理一个值，我们使用 `std::mem::drop` 函数。

`std::mem::drop` 函数与 `Drop` trait 中的 `drop` 方法不同。我们通过传递我们想要强制丢弃的值作为参数来调用它。该函数在预导入模块中，因此我们可以修改 Listing 15-15 中的 `main` 函数来调用 `drop` 函数，如 Listing 15-16 所示。

<Listing number="15-16" file-name="src/main.rs" caption="调用 `std::mem::drop` 以在值离开作用域之前显式丢弃它">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

</Listing>

运行这段代码将打印以下内容：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

文本 ``Dropping CustomSmartPointer with data `some data`!`` 在 `CustomSmartPointer created.` 和 `CustomSmartPointer dropped before the end of main.` 文本之间打印，表明 `drop` 方法代码在该点被调用来丢弃 `c`。

你可以以多种方式使用 `Drop` trait 实现中指定的代码来使清理变得方便和安全：例如，你可以使用它来创建自己的内存分配器！有了 `Drop` trait 和 Rust 的所有权系统，你不必记住清理，因为 Rust 会自动完成。

你也不必担心由于意外清理仍在使用的值而导致的问题：确保引用始终有效的所有权系统也确保 `drop` 只在值不再被使用时调用一次。

现在我们已经研究了 `Box<T>` 和一些智能指针的特性，让我们看看标准库中定义的其他一些智能指针。
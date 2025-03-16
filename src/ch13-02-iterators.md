## 使用迭代器处理一系列项目

迭代器模式允许你依次对一系列项目执行某些任务。迭代器负责遍历每个项目的逻辑，并确定序列何时结束。当你使用迭代器时，你不需要自己重新实现这些逻辑。

在 Rust 中，迭代器是**惰性**的，这意味着在你调用消耗迭代器的方法之前，它们不会产生任何效果。例如，Listing 13-10 中的代码通过调用 `Vec<T>` 上定义的 `iter` 方法，在向量 `v1` 的项目上创建了一个迭代器。这段代码本身并没有做任何有用的事情。

<Listing number="13-10" file-name="src/main.rs" caption="创建一个迭代器">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

迭代器存储在 `v1_iter` 变量中。一旦我们创建了一个迭代器，我们可以以多种方式使用它。在第 3 章的 Listing 3-5 中，我们使用 `for` 循环遍历数组，对其中的每个项目执行一些代码。在底层，这隐式地创建并消耗了一个迭代器，但直到现在我们才详细讨论它的工作原理。

在 Listing 13-11 的示例中，我们将迭代器的创建与 `for` 循环中的迭代器使用分离开来。当使用 `v1_iter` 中的迭代器调用 `for` 循环时，迭代器中的每个元素都会在循环的一次迭代中使用，并打印出每个值。

<Listing number="13-11" file-name="src/main.rs" caption="在 `for` 循环中使用迭代器">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

在没有标准库提供迭代器的语言中，你可能会通过从索引 0 开始一个变量，使用该变量索引向量以获取值，并在循环中递增变量值，直到达到向量中的项目总数来实现相同的功能。

迭代器为你处理所有这些逻辑，减少了你可能搞砸的重复代码。迭代器为你提供了更多的灵活性，可以将相同的逻辑用于许多不同类型的序列，而不仅仅是你可以索引的数据结构，比如向量。让我们看看迭代器是如何做到这一点的。

### `Iterator` Trait 和 `next` 方法

所有迭代器都实现了一个名为 `Iterator` 的 trait，该 trait 定义在标准库中。该 trait 的定义如下：

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // 省略了带有默认实现的方法
}
```

注意，这个定义使用了一些新的语法：`type Item` 和 `Self::Item`，它们定义了这个 trait 的**关联类型**。我们将在第 20 章详细讨论关联类型。现在，你只需要知道这段代码表示实现 `Iterator` trait 需要你同时定义一个 `Item` 类型，并且这个 `Item` 类型用于 `next` 方法的返回类型。换句话说，`Item` 类型将是迭代器返回的类型。

`Iterator` trait 只需要实现者定义一个方法：`next` 方法，它一次返回一个迭代器的项目，包装在 `Some` 中，当迭代结束时返回 `None`。

我们可以直接在迭代器上调用 `next` 方法；Listing 13-12 展示了从向量创建的迭代器上重复调用 `next` 方法返回的值。

<Listing number="13-12" file-name="src/lib.rs" caption="在迭代器上调用 `next` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

注意，我们需要将 `v1_iter` 设为可变：在迭代器上调用 `next` 方法会改变迭代器用于跟踪其在序列中位置的内部状态。换句话说，这段代码**消耗**或使用掉了迭代器。每次调用 `next` 都会消耗迭代器中的一个项目。当我们使用 `for` 循环时，不需要将 `v1_iter` 设为可变，因为循环获取了 `v1_iter` 的所有权并在幕后使其可变。

还要注意，我们从 `next` 调用中获取的值是对向量中值的不可变引用。`iter` 方法生成一个不可变引用的迭代器。如果我们想创建一个获取 `v1` 所有权并返回拥有值的迭代器，我们可以调用 `into_iter` 而不是 `iter`。类似地，如果我们想遍历可变引用，我们可以调用 `iter_mut` 而不是 `iter`。

### 消耗迭代器的方法

`Iterator` trait 有许多不同的方法，标准库提供了默认实现；你可以通过查看标准库 API 文档中的 `Iterator` trait 来了解这些方法。其中一些方法在其定义中调用了 `next` 方法，这就是为什么在实现 `Iterator` trait 时需要实现 `next` 方法。

调用 `next` 的方法被称为**消耗适配器**，因为调用它们会消耗迭代器。一个例子是 `sum` 方法，它获取迭代器的所有权并通过重复调用 `next` 来遍历项目，从而消耗迭代器。在遍历过程中，它将每个项目添加到一个运行总数中，并在迭代完成时返回总数。Listing 13-13 展示了一个使用 `sum` 方法的测试。

<Listing number="13-13" file-name="src/lib.rs" caption="调用 `sum` 方法以获取迭代器中所有项目的总和">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

在调用 `sum` 之后，我们不允许再使用 `v1_iter`，因为 `sum` 获取了我们调用它的迭代器的所有权。

### 生成其他迭代器的方法

**迭代器适配器**是定义在 `Iterator` trait 上的方法，它们不会消耗迭代器。相反，它们通过改变原始迭代器的某些方面来生成不同的迭代器。

Listing 13-14 展示了调用迭代器适配器方法 `map` 的示例，该方法接受一个闭包，在遍历每个项目时调用该闭包。`map` 方法返回一个新的迭代器，生成修改后的项目。这里的闭包创建了一个新的迭代器，其中向量中的每个项目都将增加 1：

<Listing number="13-14" file-name="src/main.rs" caption="调用迭代器适配器 `map` 以创建一个新的迭代器">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

然而，这段代码会产生一个警告：

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

Listing 13-14 中的代码没有做任何事情；我们指定的闭包从未被调用。警告提醒我们原因：迭代器适配器是惰性的，我们需要在这里消耗迭代器。

为了修复这个警告并消耗迭代器，我们将使用 `collect` 方法，我们在第 12 章的 Listing 12-1 中使用 `env::args` 时使用过这个方法。该方法消耗迭代器并将结果值收集到一个集合数据类型中。

在 Listing 13-15 中，我们将从 `map` 调用返回的迭代器遍历的结果收集到一个向量中。这个向量最终将包含原始向量中的每个项目，增加 1。

<Listing number="13-15" file-name="src/main.rs" caption="调用 `map` 方法以创建一个新的迭代器，然后调用 `collect` 方法以消耗新的迭代器并创建一个向量">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

因为 `map` 接受一个闭包，我们可以指定要对每个项目执行的任何操作。这是一个很好的例子，展示了闭包如何让你自定义某些行为，同时重用 `Iterator` trait 提供的迭代行为。

你可以将多个迭代器适配器调用链接在一起，以可读的方式执行复杂的操作。但由于所有迭代器都是惰性的，你必须调用其中一个消耗适配器方法才能从迭代器适配器调用中获得结果。

### 使用捕获环境的闭包

许多迭代器适配器接受闭包作为参数，通常我们指定为迭代器适配器参数的闭包将是捕获其环境的闭包。

在这个例子中，我们将使用接受闭包的 `filter` 方法。闭包从迭代器中获取一个项目并返回一个 `bool`。如果闭包返回 `true`，则该值将包含在 `filter` 生成的迭代中。如果闭包返回 `false`，则该值将不会被包含。

在 Listing 13-16 中，我们使用 `filter` 和一个捕获其环境中 `shoe_size` 变量的闭包来遍历 `Shoe` 结构实例的集合。它将只返回指定尺寸的鞋子。

<Listing number="13-16" file-name="src/lib.rs" caption="使用 `filter` 方法与一个捕获 `shoe_size` 的闭包">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

`shoes_in_size` 函数获取一个鞋子向量和一个鞋子尺寸作为参数。它返回一个只包含指定尺寸鞋子的向量。

在 `shoes_in_size` 的函数体中，我们调用 `into_iter` 来创建一个获取向量所有权的迭代器。然后我们调用 `filter` 来将该迭代器适配为一个新的迭代器，该迭代器只包含闭包返回 `true` 的元素。

闭包从环境中捕获 `shoe_size` 参数，并将其与每只鞋子的尺寸进行比较，只保留指定尺寸的鞋子。最后，调用 `collect` 将适配后的迭代器返回的值收集到一个向量中，该向量由函数返回。

测试显示，当我们调用 `shoes_in_size` 时，我们只得到与我们指定尺寸相同的鞋子。
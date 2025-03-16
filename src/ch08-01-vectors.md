## 使用向量存储值列表

我们将要看的第一个集合类型是 `Vec<T>`，也称为 _向量_。向量允许你在一个数据结构中存储多个值，这些值在内存中是相邻存放的。向量只能存储相同类型的值。当你有一个项目列表时，它们非常有用，例如文件中的文本行或购物车中商品的价格。

### 创建一个新的向量

要创建一个新的空向量，我们可以调用 `Vec::new` 函数，如 Listing 8-1 所示。

<Listing number="8-1" caption="创建一个新的空向量来保存 `i32` 类型的值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-01/src/main.rs:here}}
```

</Listing>

注意，我们在这里添加了类型注解。因为我们没有向这个向量中插入任何值，Rust 不知道我们打算存储什么类型的元素。这是一个重要的点。向量是使用泛型实现的；我们将在第 10 章中介绍如何在自己的类型中使用泛型。现在，只需知道标准库提供的 `Vec<T>` 类型可以保存任何类型。当我们创建一个向量来保存特定类型时，我们可以在尖括号中指定类型。在 Listing 8-1 中，我们告诉 Rust `v` 中的 `Vec<T>` 将保存 `i32` 类型的元素。

更常见的是，你会创建一个带有初始值的 `Vec<T>`，Rust 会推断出你想要存储的值的类型，因此你很少需要做这种类型注解。Rust 方便地提供了 `vec!` 宏，它将创建一个包含你提供的值的新向量。Listing 8-2 创建了一个新的 `Vec<i32>`，它保存了值 `1`、`2` 和 `3`。整数类型是 `i32`，因为这是默认的整数类型，正如我们在第 3 章的 [“数据类型”][data-types] 部分讨论的那样。

<Listing number="8-2" caption="创建一个包含值的新向量">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-02/src/main.rs:here}}
```

</Listing>

因为我们提供了初始的 `i32` 值，Rust 可以推断出 `v` 的类型是 `Vec<i32>`，因此类型注解是不必要的。接下来，我们将看看如何修改向量。

### 更新向量

要创建一个向量然后向其添加元素，我们可以使用 `push` 方法，如 Listing 8-3 所示。

<Listing number="8-3" caption="使用 `push` 方法向向量添加值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-03/src/main.rs:here}}
```

</Listing>

与任何变量一样，如果我们希望能够改变它的值，我们需要使用 `mut` 关键字使其可变，正如第 3 章讨论的那样。我们放入的数字都是 `i32` 类型，Rust 从数据中推断出这一点，因此我们不需要 `Vec<i32>` 注解。

### 读取向量中的元素

有两种方法可以引用存储在向量中的值：通过索引或使用 `get` 方法。在以下示例中，我们为这些函数返回的值的类型添加了注解，以便更清晰。

Listing 8-4 展示了访问向量中值的两种方法，使用索引语法和 `get` 方法。

<Listing number="8-4" caption="使用索引语法和使用 `get` 方法访问向量中的项">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-04/src/main.rs:here}}
```

</Listing>

这里有几个细节需要注意。我们使用索引值 `2` 来获取第三个元素，因为向量是按数字索引的，从零开始。使用 `&` 和 `[]` 会给我们一个对索引值处元素的引用。当我们使用 `get` 方法并将索引作为参数传递时，我们会得到一个 `Option<&T>`，我们可以将其与 `match` 一起使用。

Rust 提供了这两种引用元素的方式，以便你可以选择在尝试使用超出现有元素范围的索引值时程序的行为。例如，让我们看看当我们有一个包含五个元素的向量，然后我们尝试使用每种技术访问索引为 100 的元素时会发生什么，如 Listing 8-5 所示。

<Listing number="8-5" caption="尝试访问包含五个元素的向量中索引为 100 的元素">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-05/src/main.rs:here}}
```

</Listing>

当我们运行这段代码时，第一个 `[]` 方法将导致程序 panic，因为它引用了一个不存在的元素。这种方法最适合在你希望程序在尝试访问超出向量末尾的元素时崩溃的情况下使用。

当 `get` 方法传递一个超出向量范围的索引时，它会返回 `None` 而不会 panic。如果访问超出向量范围的元素在正常情况下偶尔会发生，你会使用这种方法。然后你的代码将具有处理 `Some(&element)` 或 `None` 的逻辑，正如第 6 章讨论的那样。例如，索引可能来自用户输入的数字。如果他们不小心输入了一个过大的数字，程序得到了一个 `None` 值，你可以告诉用户当前向量中有多少项，并给他们另一个机会输入一个有效的值。这比由于输入错误而崩溃程序更友好！

当程序有一个有效的引用时，借用检查器会强制执行所有权和借用规则（在第 4 章中介绍），以确保此引用和任何其他对向量内容的引用保持有效。回想一下，规则规定你不能在同一作用域中同时拥有可变和不可变引用。这条规则适用于 Listing 8-6，我们持有一个对向量中第一个元素的不可变引用，并尝试在末尾添加一个元素。如果我们稍后在函数中尝试引用该元素，这个程序将无法工作。

<Listing number="8-6" caption="尝试在持有对项的引用的同时向向量添加元素">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-06/src/main.rs:here}}
```

</Listing>

编译此代码将导致以下错误：

```console
{{#include ../listings/ch08-common-collections/listing-08-06/output.txt}}
```

Listing 8-6 中的代码看起来应该可以工作：为什么对第一个元素的引用会关心向量末尾的变化？这个错误是由于向量的工作方式：因为向量将值相邻地放在内存中，如果当前存储向量的位置没有足够的空间将所有元素相邻存放，那么在向量末尾添加一个新元素可能需要分配新的内存并将旧元素复制到新空间。在这种情况下，对第一个元素的引用将指向已释放的内存。借用规则防止程序陷入这种情况。

> 注意：有关 `Vec<T>` 类型的实现细节的更多信息，请参见 [“The Rustonomicon”][nomicon]。

### 遍历向量中的值

要依次访问向量中的每个元素，我们会遍历所有元素，而不是使用索引逐个访问。Listing 8-7 展示了如何使用 `for` 循环获取对 `i32` 值向量中每个元素的不可变引用并打印它们。

<Listing number="8-7" caption="通过使用 `for` 循环遍历元素来打印向量中的每个元素">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-07/src/main.rs:here}}
```

</Listing>

我们还可以遍历可变向量中每个元素的可变引用，以便对所有元素进行更改。Listing 8-8 中的 `for` 循环将为每个元素添加 `50`。

<Listing number="8-8" caption="遍历向量中元素的可变引用">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-08/src/main.rs:here}}
```

</Listing>

要更改可变引用所指向的值，我们必须使用 `*` 解引用运算符来获取 `i` 中的值，然后才能使用 `+=` 运算符。我们将在第 15 章的 [“跟随指针到值”][deref] 部分中更多地讨论解引用运算符。

遍历向量，无论是不可变的还是可变的，都是安全的，因为借用检查器的规则。如果我们尝试在 Listing 8-7 和 Listing 8-8 的 `for` 循环体中插入或删除项，我们将得到一个类似于 Listing 8-6 中代码的编译器错误。`for` 循环持有的对向量的引用防止了同时修改整个向量。

### 使用枚举存储多种类型

向量只能存储相同类型的值。这可能不方便；肯定有一些用例需要存储不同类型的项目列表。幸运的是，枚举的变体是在同一个枚举类型下定义的，所以当我们需要一个类型来表示不同类型的元素时，我们可以定义并使用一个枚举！

例如，假设我们想从电子表格的一行中获取值，其中该行的某些列包含整数，某些列包含浮点数，某些列包含字符串。我们可以定义一个枚举，其变体将保存不同的值类型，所有枚举变体将被视为同一类型：枚举的类型。然后我们可以创建一个向量来保存该枚举，从而最终保存不同的类型。我们在 Listing 8-9 中演示了这一点。

<Listing number="8-9" caption="定义一个 `enum` 以在一个向量中存储不同类型的值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-09/src/main.rs:here}}
```

</Listing>

Rust 需要在编译时知道向量中将包含哪些类型，以便确切知道堆上需要多少内存来存储每个元素。我们还必须明确允许在此向量中包含哪些类型。如果 Rust 允许向量保存任何类型，那么有一种或多种类型可能会导致对向量元素执行的操作出错。使用枚举加上 `match` 表达式意味着 Rust 将在编译时确保处理所有可能的情况，正如第 6 章讨论的那样。

如果你不知道程序在运行时将获得哪些类型的详尽集合来存储在向量中，枚举技术将不起作用。相反，你可以使用 trait 对象，我们将在第 18 章中介绍。

现在我们已经讨论了一些使用向量的最常见方法，请务必查看 [API 文档][vec-api]，了解标准库在 `Vec<T>` 上定义的所有许多有用方法。例如，除了 `push`，还有一个 `pop` 方法可以移除并返回最后一个元素。

### 删除向量会删除其元素

与任何其他 `struct` 一样，当向量超出作用域时，它会被释放，如 Listing 8-10 所示。

<Listing number="8-10" caption="显示向量及其元素被删除的位置">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-10/src/main.rs:here}}
```

</Listing>

当向量被删除时，它的所有内容也会被删除，这意味着它保存的整数将被清理。借用检查器确保对向量内容的任何引用仅在向量本身有效时使用。

让我们继续讨论下一个集合类型：`String`！

[data-types]: ch03-02-data-types.html#data-types
[nomicon]: ../nomicon/vec/vec.html
[vec-api]: ../std/vec/struct.Vec.html
[deref]: ch15-02-deref.html#following-the-pointer-to-the-value-with-the-dereference-operator
## 使用允许不同类型值的 Trait 对象

在第 8 章中，我们提到向量（vector）的一个限制是它们只能存储单一类型的元素。我们在 Listing 8-9 中创建了一个变通方案，定义了一个 `SpreadsheetCell` 枚举，它有变体来保存整数、浮点数和文本。这意味着我们可以在每个单元格中存储不同类型的数据，并且仍然有一个表示一行单元格的向量。当我们的可互换项是编译时已知的一组固定类型时，这是一个非常好的解决方案。

然而，有时我们希望库的用户能够扩展在特定情况下有效的类型集。为了展示如何实现这一点，我们将创建一个图形用户界面（GUI）工具示例，该工具会遍历一个项目列表，并在每个项目上调用 `draw` 方法将其绘制到屏幕上——这是 GUI 工具的常见技术。我们将创建一个名为 `gui` 的库 crate，其中包含 GUI 库的结构。这个 crate 可能包含一些供人们使用的类型，例如 `Button` 或 `TextField`。此外，`gui` 用户可能希望创建他们自己的可绘制类型：例如，一个程序员可能会添加一个 `Image`，而另一个程序员可能会添加一个 `SelectBox`。

我们不会为这个示例实现一个完整的 GUI 库，但会展示这些部分如何组合在一起。在编写库时，我们无法知道并定义所有其他程序员可能想要创建的类型。但我们确实知道 `gui` 需要跟踪许多不同类型的值，并且它需要在这些不同类型的值上调用 `draw` 方法。它不需要确切知道调用 `draw` 方法时会发生什么，只需要知道该值将具有我们可以调用的该方法。

在具有继承的语言中，我们可能会定义一个名为 `Component` 的类，该类具有一个名为 `draw` 的方法。其他类，如 `Button`、`Image` 和 `SelectBox`，将从 `Component` 继承，从而继承 `draw` 方法。它们可以各自覆盖 `draw` 方法以定义它们的自定义行为，但框架可以将所有这些类型视为 `Component` 实例并调用它们的 `draw` 方法。但由于 Rust 没有继承，我们需要另一种方式来构建 `gui` 库，以允许用户使用新类型扩展它。

### 为通用行为定义 Trait

为了实现我们希望 `gui` 具有的行为，我们将定义一个名为 `Draw` 的 trait，该 trait 将有一个名为 `draw` 的方法。然后我们可以定义一个接受 trait 对象的向量。_trait 对象_ 既指向实现我们指定 trait 的类型的实例，也指向在运行时用于查找该类型上的 trait 方法的表。我们通过指定某种指针（例如 `&` 引用或 `Box<T>` 智能指针）来创建 trait 对象，然后是 `dyn` 关键字，最后指定相关的 trait。（我们将在第 20 章的 [“动态大小类型和 `Sized` Trait”][dynamically-sized] 中讨论 trait 对象必须使用指针的原因。）我们可以使用 trait 对象来代替泛型或具体类型。无论我们在哪里使用 trait 对象，Rust 的类型系统都会在编译时确保在该上下文中使用的任何值都实现了 trait 对象的 trait。因此，我们不需要在编译时知道所有可能的类型。

我们提到过，在 Rust 中，我们避免将结构体和枚举称为“对象”，以将它们与其他语言的对象区分开来。在结构体或枚举中，结构体字段中的数据与 `impl` 块中的行为是分开的，而在其他语言中，数据和行为组合成一个概念通常被称为对象。然而，trait 对象 _确实_ 更类似于其他语言中的对象，因为它们结合了数据和行为。但 trait 对象与传统对象的不同之处在于，我们不能向 trait 对象添加数据。trait 对象不像其他语言中的对象那样普遍有用：它们的特定目的是允许跨通用行为进行抽象。

Listing 18-3 展示了如何定义一个名为 `Draw` 的 trait，该 trait 有一个名为 `draw` 的方法。

<Listing number="18-3" file-name="src/lib.rs" caption="`Draw` trait 的定义">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

这种语法应该从我们在第 10 章讨论如何定义 trait 时看起来很熟悉。接下来是一些新语法：Listing 18-4 定义了一个名为 `Screen` 的结构体，它包含一个名为 `components` 的向量。这个向量的类型是 `Box<dyn Draw>`，这是一个 trait 对象；它是 `Box` 中实现 `Draw` trait 的任何类型的替身。

<Listing number="18-4" file-name="src/lib.rs" caption="定义 `Screen` 结构体，其中 `components` 字段保存实现 `Draw` trait 的 trait 对象向量">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

在 `Screen` 结构体上，我们将定义一个名为 `run` 的方法，该方法将在其 `components` 中的每个组件上调用 `draw` 方法，如 Listing 18-5 所示。

<Listing number="18-5" file-name="src/lib.rs" caption="`Screen` 上的 `run` 方法，它在每个组件上调用 `draw` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

这与使用带有 trait 约束的泛型类型参数定义结构体不同。泛型类型参数一次只能替换为一个具体类型，而 trait 对象允许在运行时用多个具体类型填充 trait 对象。例如，我们可以使用泛型类型和 trait 约束来定义 `Screen` 结构体，如 Listing 18-6 所示：

<Listing number="18-6" file-name="src/lib.rs" caption="使用泛型和 trait 约束的 `Screen` 结构体及其 `run` 方法的替代实现">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

这将我们限制为具有所有组件类型为 `Button` 或所有组件类型为 `TextField` 的 `Screen` 实例。如果你只会有同质集合，使用泛型和 trait 约束是更可取的，因为这些定义将在编译时单态化为使用具体类型。

另一方面，使用 trait 对象的方法，一个 `Screen` 实例可以包含一个 `Vec<T>`，其中包含 `Box<Button>` 和 `Box<TextField>`。让我们看看这是如何工作的，然后我们将讨论运行时性能的影响。

### 实现 Trait

现在我们将添加一些实现 `Draw` trait 的类型。我们将提供 `Button` 类型。再次强调，实际实现 GUI 库超出了本书的范围，因此 `draw` 方法在其主体中不会有任何有用的实现。为了想象实现可能是什么样子，`Button` 结构体可能有 `width`、`height` 和 `label` 字段，如 Listing 18-7 所示：

<Listing number="18-7" file-name="src/lib.rs" caption="实现 `Draw` trait 的 `Button` 结构体">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

`Button` 上的 `width`、`height` 和 `label` 字段将与其他组件上的字段不同；例如，`TextField` 类型可能有这些相同的字段加上一个 `placeholder` 字段。我们想要在屏幕上绘制的每种类型都将实现 `Draw` trait，但将在 `draw` 方法中使用不同的代码来定义如何绘制该特定类型，如 `Button` 在这里所做的那样（没有实际的 GUI 代码，如前所述）。例如，`Button` 类型可能有一个额外的 `impl` 块，其中包含与用户点击按钮时发生的情况相关的方法。这些方法不适用于像 `TextField` 这样的类型。

如果使用我们库的人决定实现一个具有 `width`、`height` 和 `options` 字段的 `SelectBox` 结构体，他们将在 `SelectBox` 类型上实现 `Draw` trait，如 Listing 18-8 所示。

<Listing number="18-8" file-name="src/main.rs" caption="另一个使用 `gui` 并在 `SelectBox` 结构体上实现 `Draw` trait 的 crate">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

我们库的用户现在可以编写他们的 `main` 函数来创建一个 `Screen` 实例。他们可以向 `Screen` 实例添加一个 `SelectBox` 和一个 `Button`，方法是将每个放入 `Box<T>` 中以成为 trait 对象。然后他们可以调用 `Screen` 实例上的 `run` 方法，该方法将在每个组件上调用 `draw`。Listing 18-9 展示了这个实现：

<Listing number="18-9" file-name="src/main.rs" caption="使用 trait 对象存储实现相同 trait 的不同类型的值">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

当我们编写库时，我们不知道有人可能会添加 `SelectBox` 类型，但我们的 `Screen` 实现能够操作新类型并绘制它，因为 `SelectBox` 实现了 `Draw` trait，这意味着它实现了 `draw` 方法。

这个概念——只关心值响应的消息而不是值的具体类型——类似于动态类型语言中的 _鸭子类型_ 概念：如果它走起来像鸭子，叫起来像鸭子，那么它一定是鸭子！在 Listing 18-5 中 `Screen` 上的 `run` 实现中，`run` 不需要知道每个组件的具体类型是什么。它不检查组件是 `Button` 还是 `SelectBox` 的实例，它只是在组件上调用 `draw` 方法。通过指定 `Box<dyn Draw>` 作为 `components` 向量中值的类型，我们定义了 `Screen` 需要我们可以调用 `draw` 方法的值。

使用 trait 对象和 Rust 的类型系统编写类似于使用鸭子类型的代码的优势在于，我们不必在运行时检查值是否实现了特定方法，也不必担心如果值没有实现方法但我们仍然调用它时会出现错误。如果值没有实现 trait 对象所需的 trait，Rust 将不会编译我们的代码。

例如，Listing 18-10 展示了如果我们尝试创建一个以 `String` 作为组件的 `Screen` 会发生什么。

<Listing number="18-10" file-name="src/main.rs" caption="尝试使用未实现 trait 对象的 trait 的类型">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

我们会得到这个错误，因为 `String` 没有实现 `Draw` trait：

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

这个错误让我们知道，我们要么向 `Screen` 传递了我们不打算传递的东西，因此应该传递不同的类型，要么我们应该在 `String` 上实现 `Draw`，以便 `Screen` 能够在其上调用 `draw`。

### Trait 对象执行动态分派

回想一下第 10 章中 [“使用泛型的代码性能”][performance-of-code-using-generics] 中我们对编译器对泛型执行的单态化过程的讨论：编译器为我们使用的每个具体类型生成函数和方法的非泛型实现。单态化产生的代码执行的是 _静态分派_，这是编译器在编译时知道你在调用哪个方法的情况。这与 _动态分派_ 相反，动态分派是编译器在编译时无法确定你在调用哪个方法的情况。在动态分派的情况下，编译器会生成代码，在运行时确定要调用哪个方法。

当我们使用 trait 对象时，Rust 必须使用动态分派。编译器不知道可能与使用 trait 对象的代码一起使用的所有类型，因此它不知道要调用哪个类型上的哪个方法。相反，在运行时，Rust 使用 trait 对象内部的指针来确定要调用哪个方法。这种查找会产生运行时成本，而静态分派不会产生这种成本。动态分派还阻止编译器选择内联方法的代码，这反过来又阻止了一些优化，Rust 有一些关于在哪里可以使用和不能使用动态分派的规则，称为 _dyn 兼容性_。这些规则超出了本次讨论的范围，但你可以在 [参考][dyn-compatibility] 中阅读更多关于它们的信息。然而，我们在 Listing 18-5 中编写的代码确实获得了额外的灵活性，并且能够在 Listing 18-9 中支持，因此这是一个需要考虑的权衡。

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
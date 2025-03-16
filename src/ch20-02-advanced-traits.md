## 高级特性

我们首先在第 10 章的 [“Traits: 定义共享行为”][traits-defining-shared-behavior]<!-- ignore --> 中介绍了 traits，但我们没有讨论更高级的细节。现在你对 Rust 有了更多的了解，我们可以深入探讨这些细节。

<!-- 旧链接，不要删除 -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>

### 关联类型

_关联类型_ 将类型占位符与 trait 连接起来，使得 trait 方法定义可以在其签名中使用这些占位符类型。trait 的实现者将为特定实现指定要使用的具体类型，而不是占位符类型。这样，我们可以定义一个使用某些类型的 trait，而不需要在实现 trait 之前确切知道这些类型是什么。

我们在本章中描述的大多数高级特性都是很少需要的。关联类型介于两者之间：它们比本书其余部分解释的特性使用得更少，但比本章讨论的许多其他特性更常见。

一个带有关联类型的 trait 的例子是标准库提供的 `Iterator` trait。关联类型名为 `Item`，代表实现 `Iterator` trait 的类型正在迭代的值的类型。`Iterator` trait 的定义如 Listing 20-13 所示。

<Listing number="20-13" caption="带有关联类型 `Item` 的 `Iterator` trait 的定义">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

类型 `Item` 是一个占位符，`next` 方法的定义显示它将返回类型为 `Option<Self::Item>` 的值。`Iterator` trait 的实现者将为 `Item` 指定具体类型，`next` 方法将返回包含该具体类型值的 `Option`。

关联类型可能看起来与泛型类似，因为后者允许我们定义一个函数而不指定它可以处理的类型。为了检查这两个概念之间的区别，我们将查看一个名为 `Counter` 的类型上实现 `Iterator` trait 的例子，该实现指定 `Item` 类型为 `u32`：

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

这种语法似乎与泛型类似。那么为什么不直接使用泛型定义 `Iterator` trait，如 Listing 20-14 所示呢？

<Listing number="20-14" caption="使用泛型的 `Iterator` trait 的假设定义">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

区别在于，当使用泛型时，如 Listing 20-14 所示，我们必须在每个实现中注解类型；因为我们也可以为 `Counter` 实现 `Iterator<String>` 或任何其他类型，所以我们可以为 `Counter` 实现多个 `Iterator`。换句话说，当一个 trait 有一个泛型参数时，它可以为一个类型多次实现，每次改变泛型类型参数的具体类型。当我们在 `Counter` 上使用 `next` 方法时，我们必须提供类型注解来指示我们要使用哪个 `Iterator` 实现。

使用关联类型时，我们不需要注解类型，因为我们不能为一个类型多次实现一个 trait。在 Listing 20-13 中使用关联类型的定义中，我们只能选择一次 `Item` 的类型，因为只能有一个 `impl Iterator for Counter`。我们不必在每次调用 `Counter` 上的 `next` 时指定我们想要一个 `u32` 值的迭代器。

关联类型也成为 trait 契约的一部分：trait 的实现者必须提供一个类型来代替关联类型占位符。关联类型通常有一个描述类型将如何使用的名称，并且在 API 文档中记录关联类型是一个好的做法。

### 默认泛型类型参数和运算符重载

当我们使用泛型类型参数时，我们可以为泛型类型指定一个默认的具体类型。这消除了 trait 实现者指定具体类型的需要，如果默认类型适用的话。你可以在声明泛型类型时使用 `<PlaceholderType=ConcreteType>` 语法指定默认类型。

一个非常有用的技术是 _运算符重载_，在这种情况下，你可以自定义运算符（如 `+`）在特定情况下的行为。

Rust 不允许你创建自己的运算符或重载任意运算符。但你可以通过实现与运算符相关的 traits 来重载 `std::ops` 中列出的操作和相应的 traits。例如，在 Listing 20-15 中，我们重载了 `+` 运算符以将两个 `Point` 实例相加。我们通过在 `Point` 结构体上实现 `Add` trait 来实现这一点。

<Listing number="20-15" file-name="src/main.rs" caption="实现 `Add` trait 以重载 `Point` 实例的 `+` 运算符">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

`add` 方法将两个 `Point` 实例的 `x` 值和 `y` 值相加以创建一个新的 `Point`。`Add` trait 有一个名为 `Output` 的关联类型，它决定了 `add` 方法返回的类型。

此代码中的默认泛型类型在 `Add` trait 中。以下是它的定义：

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

这段代码应该看起来很熟悉：一个带有一个方法和一个关联类型的 trait。新部分是 `Rhs=Self`：这种语法称为 _默认类型参数_。`Rhs` 泛型类型参数（“right-hand side” 的缩写）定义了 `add` 方法中 `rhs` 参数的类型。如果我们在实现 `Add` trait 时不指定 `Rhs` 的具体类型，`Rhs` 的类型将默认为 `Self`，即我们正在实现 `Add` 的类型。

当我们为 `Point` 实现 `Add` 时，我们使用了 `Rhs` 的默认值，因为我们想要将两个 `Point` 实例相加。让我们看一个实现 `Add` trait 的例子，其中我们希望自定义 `Rhs` 类型而不是使用默认值。

我们有两个结构体，`Millimeters` 和 `Meters`，它们以不同的单位保存值。这种将现有类型包装在另一个结构体中的薄包装称为 _newtype 模式_，我们将在 [“使用 Newtype 模式在外部类型上实现外部 Traits”][newtype]<!-- ignore --> 部分中更详细地描述。我们希望将毫米值与米值相加，并让 `Add` 的实现正确地进行转换。我们可以为 `Millimeters` 实现 `Add`，并将 `Meters` 作为 `Rhs`，如 Listing 20-16 所示。

<Listing number="20-16" file-name="src/lib.rs" caption="在 `Millimeters` 上实现 `Add` trait 以将 `Millimeters` 与 `Meters` 相加">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

要将 `Millimeters` 和 `Meters` 相加，我们指定 `impl Add<Meters>` 来设置 `Rhs` 类型参数的值，而不是使用默认的 `Self`。

你将以两种主要方式使用默认类型参数：

1. 扩展类型而不破坏现有代码
2. 允许在大多数用户不需要的特定情况下进行自定义

标准库的 `Add` trait 是第二个目的的一个例子：通常，你会将两个相同类型的值相加，但 `Add` trait 提供了超越这一点的自定义能力。在 `Add` trait 定义中使用默认类型参数意味着大多数时候你不需要指定额外的参数。换句话说，不需要一些实现样板，使得使用 trait 更容易。

第一个目的与第二个目的类似，但方向相反：如果你想向现有 trait 添加类型参数，可以为其提供默认值，以允许扩展 trait 的功能而不破坏现有的实现代码。

<!-- 旧链接，不要删除 -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>

### 消除同名方法之间的歧义

Rust 中没有任何东西阻止一个 trait 拥有与另一个 trait 的方法同名的方法，Rust 也不阻止你在一个类型上同时实现这两个 trait。还可以直接在类型上实现与 trait 方法同名的方法。

当调用同名方法时，你需要告诉 Rust 你想要使用哪一个。考虑 Listing 20-17 中的代码，我们定义了两个 trait，`Pilot` 和 `Wizard`，它们都有一个名为 `fly` 的方法。然后我们在已经实现了 `fly` 方法的 `Human` 类型上实现了这两个 trait。每个 `fly` 方法都做不同的事情。

<Listing number="20-17" file-name="src/main.rs" caption="定义了两个具有 `fly` 方法的 trait，并在 `Human` 类型上实现了它们，同时在 `Human` 上直接实现了 `fly` 方法。">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

当我们在 `Human` 实例上调用 `fly` 时，编译器默认调用直接在该类型上实现的方法，如 Listing 20-18 所示。

<Listing number="20-18" file-name="src/main.rs" caption="在 `Human` 实例上调用 `fly`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

运行此代码将打印 `*waving arms furiously*`，显示 Rust 调用了直接在 `Human` 上实现的 `fly` 方法。

要调用 `Pilot` trait 或 `Wizard` trait 的 `fly` 方法，我们需要使用更明确的语法来指定我们指的是哪个 `fly` 方法。Listing 20-19 演示了这种语法。

<Listing number="20-19" file-name="src/main.rs" caption="指定我们要调用的 trait 的 `fly` 方法">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

在方法名称前指定 trait 名称向 Rust 澄清了我们想要调用哪个 `fly` 实现。我们也可以写 `Human::fly(&person)`，这与我们在 Listing 20-19 中使用的 `person.fly()` 等效，但如果不需要消除歧义，这样写会稍微长一些。

运行此代码将打印以下内容：

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

因为 `fly` 方法有一个 `self` 参数，如果我们有两个 _类型_ 都实现了一个 _trait_，Rust 可以根据 `self` 的类型推断出要使用哪个 trait 的实现。

然而，不是方法的关联函数没有 `self` 参数。当有多个类型或 trait 定义了具有相同函数名称的非方法函数时，Rust 并不总是知道你要指的是哪个类型，除非你使用 _完全限定语法_。例如，在 Listing 20-20 中，我们为一个动物收容所创建了一个 trait，该收容所希望将所有小狗命名为 _Spot_。我们创建了一个带有关联非方法函数 `baby_name` 的 `Animal` trait。`Animal` trait 为结构体 `Dog` 实现，我们在 `Dog` 上也直接提供了一个关联的非方法函数 `baby_name`。

<Listing number="20-20" file-name="src/main.rs" caption="一个带有关联函数的 trait 和一个具有相同名称的关联函数的类型，该类型也实现了该 trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

我们在 `Dog` 上定义的 `baby_name` 关联函数中实现了将所有小狗命名为 Spot 的代码。`Dog` 类型还实现了 `Animal` trait，该 trait 描述了所有动物的特征。小狗被称为 puppy，这在 `Dog` 上实现的 `Animal` trait 的 `baby_name` 关联函数中表达。

在 `main` 中，我们调用 `Dog::baby_name` 函数，该函数直接调用在 `Dog` 上定义的关联函数。此代码打印以下内容：

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

这个输出不是我们想要的。我们想要调用我们在 `Dog` 上实现的 `Animal` trait 的 `baby_name` 函数，以便代码打印 `A baby dog is called a puppy`。我们在 Listing 20-19 中使用的指定 trait 名称的技术在这里没有帮助；如果我们将 `main` 改为 Listing 20-21 中的代码，我们将得到一个编译错误。

<Listing number="20-21" file-name="src/main.rs" caption="尝试调用 `Animal` trait 的 `baby_name` 函数，但 Rust 不知道要使用哪个实现">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

因为 `Animal::baby_name` 没有 `self` 参数，并且可能有其他类型实现了 `Animal` trait，Rust 无法确定我们要使用哪个 `Animal::baby_name` 实现。我们将得到以下编译器错误：

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

为了消除歧义并告诉 Rust 我们要使用 `Dog` 上的 `Animal` 实现，而不是其他类型的 `Animal` 实现，我们需要使用完全限定语法。Listing 20-22 演示了如何使用完全限定语法。

<Listing number="20-22" file-name="src/main.rs" caption="使用完全限定语法指定我们要调用 `Dog` 上实现的 `Animal` trait 的 `baby_name` 函数">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

我们在尖括号内为 Rust 提供了一个类型注解，指示我们要通过将 `Dog` 类型视为 `Animal` 来调用 `Animal` trait 的 `baby_name` 方法。此代码现在将打印我们想要的内容：

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

一般来说，完全限定语法定义如下：

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

对于不是方法的关联函数，不会有 `receiver`：只有其他参数的列表。你可以在调用函数或方法时使用完全限定语法。然而，你可以省略 Rust 可以从程序中的其他信息推断出的任何部分。只有在有多个实现使用相同名称并且 Rust 需要帮助来识别你要调用哪个实现的情况下，你才需要使用这种更详细的语法。

<!-- 旧链接，不要删除 -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### 使用 Supertraits

有时你可能会编写一个依赖于另一个 trait 的 trait 定义：为了让一个类型实现第一个 trait，你希望要求该类型也实现第二个 trait。你会这样做，以便你的 trait 定义可以使用第二个 trait 的关联项。你的 trait 定义所依赖的 trait 称为你的 trait 的 _supertrait_。

例如，假设我们想创建一个 `OutlinePrint` trait，它有一个 `outline_print` 方法，该方法将打印一个给定的值，格式化为用星号框起来的形式。也就是说，给定一个实现了标准库 trait `Display` 的 `Point` 结构体，结果为 `(x, y)`，当我们调用 `outline_print` 时，对于一个 `x` 为 `1`，`y` 为 `3` 的 `Point` 实例，它应该打印以下内容：

```text
**********
*        *
* (1, 3) *
*        *
**********
```

在 `outline_print` 方法的实现中，我们希望使用 `Display` trait 的功能。因此，我们需要指定 `OutlinePrint` trait 仅适用于也实现 `Display` 并提供 `OutlinePrint` 所需功能的类型。我们可以在 trait 定义中通过指定 `OutlinePrint: Display` 来做到这一点。这种技术类似于向 trait 添加 trait 绑定。Listing 20-23 展示了 `OutlinePrint` trait 的实现。

<Listing number="20-23" file-name="src/main.rs" caption="实现需要 `Display` 功能的 `OutlinePrint` trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

因为我们指定了 `OutlinePrint` 需要 `Display` trait，所以我们可以使用为任何实现 `Display` 的类型自动实现的 `to_string` 函数。如果我们尝试在不添加冒号并在 trait 名称后指定 `Display` trait 的情况下使用 `to_string`，我们会得到一个错误，指出在当前作用域中没有为类型 `&Self` 找到名为 `to_string` 的方法。

让我们看看当我们尝试在一个没有实现 `Display` 的类型（如 `Point` 结构体）上实现 `OutlinePrint` 时会发生什么：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

我们得到一个错误，指出需要 `Display` 但未实现：

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

要修复此问题，我们在 `Point` 上实现 `Display` 并满足 `OutlinePrint` 所需的约束，如下所示：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

然后，在 `Point` 上实现 `OutlinePrint` trait 将成功编译，我们可以在 `Point` 实例上调用 `outline_print` 以在星号框中显示它。

### 使用 Newtype 模式在外部类型上实现外部 Traits

在第 10 章的 [“在类型上实现 Trait”][implementing-a-trait-on-a-type]<!-- ignore --> 中，我们提到了孤儿规则，该规则规定我们只能在类型上实现 trait，如果 trait 或类型，或两者都是我们 crate 本地的。可以使用 _newtype 模式_ 绕过此限制，该模式涉及在元组结构体中创建一个新类型。（我们在第 5 章的 [“使用没有命名字段的元组结构体创建不同类型”][tuple-structs]<!-- ignore --> 中介绍了元组结构体。）元组结构体将有一个字段，并且是我们想要实现 trait 的类型的薄包装。然后包装类型是我们 crate 本地的，我们可以在包装类型上实现 trait。_Newtype_ 是一个源自 Haskell 编程语言的术语。使用此模式没有运行时性能损失，并且包装类型在编译时被省略。

例如，假设我们想在 `Vec<T>` 上实现 `Display`，孤儿规则阻止我们直接这样做，因为 `Display` trait 和 `Vec<T>` 类型是在我们 crate 之外定义的。我们可以创建一个 `Wrapper` 结构体，它持有 `Vec<T>` 的实例；然后我们可以在 `Wrapper` 上实现 `Display` 并使用 `Vec<T>` 值，如 Listing 20-24 所示。

<Listing number="20-24" file-name="src/main.rs" caption="创建一个围绕 `Vec<String>` 的 `Wrapper` 类型以实现 `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

`Display` 的实现使用 `self.0` 访问内部的 `Vec<T>`，因为 `Wrapper` 是一个元组结构体，`Vec<T>` 是元组中索引为 0 的项。然后我们可以在 `Wrapper` 上使用 `Display` trait 的功能。

使用此技术的缺点是 `Wrapper` 是一个新类型，因此它没有它所持有的值的方法。我们必须直接在 `Wrapper` 上实现 `Vec<T>` 的所有方法，以便这些方法委托给 `self.0`，这将允许我们像对待 `Vec<T>` 一样对待 `Wrapper`。如果我们希望新类型具有内部类型的所有方法，实现 `Deref` trait 以返回内部类型将是一个解决方案（我们在第 15 章的 [“使用 `Deref` trait 将智能指针视为常规引用”][smart-pointer-deref]<!-- ignore --> 中讨论了实现 `Deref` trait）。如果我们不希望 `Wrapper` 类型具有内部类型的所有方法——例如，为了限制 `Wrapper` 类型的行为——我们必须手动实现我们确实想要的方法。

即使不涉及 traits，这种 newtype 模式也很有用。让我们转换焦点，看看一些与 Rust 类型系统交互的高级方法。

[newtype]: ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits-defining-shared-behavior]: ch10-02-traits.html#traits-defining-shared-behavior
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait
[tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
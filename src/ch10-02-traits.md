## Traits: 定义共享行为

一个 _trait_ 定义了特定类型所具有的功能，并且可以与其他类型共享。我们可以使用 traits 以抽象的方式定义共享行为。我们可以使用 _trait bounds_ 来指定泛型类型可以是具有特定行为的任何类型。

> 注意：Traits 类似于其他语言中通常称为 _接口_ 的特性，尽管有一些差异。

### 定义一个 Trait

类型的行为由我们可以在该类型上调用的方法组成。如果我们可以对所有类型调用相同的方法，那么不同的类型就共享相同的行为。Trait 定义是一种将方法签名组合在一起的方式，以定义实现某些目的所需的一组行为。

例如，假设我们有多个结构体，它们持有各种类型和数量的文本：一个 `NewsArticle` 结构体，它持有一个在特定位置提交的新闻故事，以及一个 `SocialPost`，它最多可以有 280 个字符，并带有指示它是新帖子、转发还是对另一个帖子的回复的元数据。

我们想要创建一个名为 `aggregator` 的媒体聚合器库 crate，它可以显示可能存储在 `NewsArticle` 或 `SocialPost` 实例中的数据摘要。为此，我们需要每种类型的摘要，并且我们将通过在实例上调用 `summarize` 方法来请求该摘要。Listing 10-12 展示了一个公共 `Summary` trait 的定义，该 trait 表达了这种行为。

<Listing number="10-12" file-name="src/lib.rs" caption="一个由 `summarize` 方法提供的行为组成的 `Summary` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

在这里，我们使用 `trait` 关键字声明一个 trait，然后是该 trait 的名称，在本例中为 `Summary`。我们还将该 trait 声明为 `pub`，以便依赖于该 crate 的其他 crate 也可以使用该 trait，正如我们将在几个示例中看到的那样。在大括号内，我们声明了描述实现该 trait 的类型的行为的方法签名，在本例中为 `fn summarize(&self) -> String`。

在方法签名之后，我们没有在大括号内提供实现，而是使用分号。实现该 trait 的每个类型都必须为方法的主体提供自己的自定义行为。编译器将强制要求任何具有 `Summary` trait 的类型都必须具有使用此签名定义的 `summarize` 方法。

一个 trait 的主体可以有多个方法：方法签名每行列出一个，每行以分号结尾。

### 在类型上实现 Trait

现在我们已经定义了 `Summary` trait 方法的期望签名，我们可以在媒体聚合器中的类型上实现它。Listing 10-13 展示了在 `NewsArticle` 结构体上实现 `Summary` trait 的代码，该实现使用标题、作者和位置来创建 `summarize` 的返回值。对于 `SocialPost` 结构体，我们将 `summarize` 定义为用户名后跟帖子的全部文本，假设帖子内容已经限制在 280 个字符以内。

<Listing number="10-13" file-name="src/lib.rs" caption="在 `NewsArticle` 和 `SocialPost` 类型上实现 `Summary` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

在类型上实现 trait 类似于实现常规方法。不同之处在于，在 `impl` 之后，我们放置要实现的 trait 名称，然后使用 `for` 关键字，然后指定我们要为其实现 trait 的类型的名称。在 `impl` 块内，我们放置 trait 定义所定义的方法签名。我们没有在每个签名后添加分号，而是使用大括号并填充方法主体，以指定我们希望该 trait 的方法在特定类型上具有的具体行为。

现在库已经在 `NewsArticle` 和 `SocialPost` 上实现了 `Summary` trait，crate 的用户可以像调用常规方法一样在 `NewsArticle` 和 `SocialPost` 实例上调用 trait 方法。唯一的区别是用户必须将 trait 和类型都引入作用域。以下是一个二进制 crate 如何使用我们的 `aggregator` 库 crate 的示例：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

此代码打印 `1 new post: horse_ebooks: of course, as you probably already know, people`。

依赖于 `aggregator` crate 的其他 crate 也可以将 `Summary` trait 引入作用域，以在其自己的类型上实现 `Summary`。需要注意的是，我们只能在 trait 或类型（或两者）是我们 crate 本地的情况下在类型上实现 trait。例如，我们可以将标准库中的 `Display` trait 实现为 `SocialPost` 自定义类型的一部分，作为我们 `aggregator` crate 功能的一部分，因为 `SocialPost` 类型是我们 `aggregator` crate 本地的。我们还可以在我们的 `aggregator` crate 中为 `Vec<T>` 实现 `Summary`，因为 `Summary` trait 是我们 `aggregator` crate 本地的。

但我们不能在外部类型上实现外部 trait。例如，我们不能在我们的 `aggregator` crate 中为 `Vec<T>` 实现 `Display` trait，因为 `Display` 和 `Vec<T>` 都是在标准库中定义的，而不是我们 `aggregator` crate 本地的。此限制是称为 _一致性_ 的属性的一部分，更具体地说是 _孤儿规则_，之所以这样命名是因为父类型不存在。此规则确保其他人的代码不会破坏你的代码，反之亦然。如果没有此规则，两个 crate 可以为同一类型实现相同的 trait，而 Rust 将不知道使用哪个实现。

### 默认实现

有时为 trait 中的部分或全部方法提供默认行为是有用的，而不是要求每个类型都实现所有方法。然后，当我们在特定类型上实现 trait 时，我们可以保留或覆盖每个方法的默认行为。

在 Listing 10-14 中，我们为 `Summary` trait 的 `summarize` 方法指定了一个默认字符串，而不是像在 Listing 10-12 中那样仅定义方法签名。

<Listing number="10-14" file-name="src/lib.rs" caption="定义一个带有 `summarize` 方法默认实现的 `Summary` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

要使用默认实现来总结 `NewsArticle` 实例，我们指定一个空的 `impl` 块，即 `impl Summary for NewsArticle {}`。

尽管我们不再直接在 `NewsArticle` 上定义 `summarize` 方法，但我们提供了默认实现并指定 `NewsArticle` 实现了 `Summary` trait。因此，我们仍然可以在 `NewsArticle` 实例上调用 `summarize` 方法，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

此代码打印 `New article available! (Read more...)`。

创建默认实现不需要我们更改 Listing 10-13 中 `SocialPost` 上 `Summary` 实现的任何内容。原因是覆盖默认实现的语法与实现没有默认实现的 trait 方法的语法相同。

默认实现可以调用同一 trait 中的其他方法，即使这些其他方法没有默认实现。通过这种方式，trait 可以提供大量有用的功能，而只需要实现者指定其中的一小部分。例如，我们可以定义 `Summary` trait 具有一个需要实现的 `summarize_author` 方法，然后定义一个具有默认实现的 `summarize` 方法，该方法调用 `summarize_author` 方法：

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

要使用此版本的 `Summary`，我们只需要在类型上实现 trait 时定义 `summarize_author`：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

在我们定义了 `summarize_author` 之后，我们可以在 `SocialPost` 结构体的实例上调用 `summarize`，`summarize` 的默认实现将调用我们提供的 `summarize_author` 定义。因为我们实现了 `summarize_author`，所以 `Summary` trait 已经为我们提供了 `summarize` 方法的行为，而无需我们编写更多代码。以下是它的样子：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

此代码打印 `1 new post: (Read more from @horse_ebooks...)`。

请注意，无法从同一方法的覆盖实现中调用默认实现。

### Traits 作为参数

现在你已经知道如何定义和实现 traits，我们可以探索如何使用 traits 来定义接受许多不同类型的函数。我们将使用在 Listing 10-13 中在 `NewsArticle` 和 `SocialPost` 类型上实现的 `Summary` trait 来定义一个 `notify` 函数，该函数在其 `item` 参数上调用 `summarize` 方法，该参数是实现了 `Summary` trait 的某种类型。为此，我们使用 `impl Trait` 语法，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

我们没有为 `item` 参数指定具体类型，而是指定了 `impl` 关键字和 trait 名称。此参数接受任何实现了指定 trait 的类型。在 `notify` 的主体中，我们可以调用 `item` 上来自 `Summary` trait 的任何方法，例如 `summarize`。我们可以调用 `notify` 并传入 `NewsArticle` 或 `SocialPost` 的任何实例。使用任何其他类型（例如 `String` 或 `i32`）调用该函数的代码将无法编译，因为这些类型没有实现 `Summary`。

<!-- 旧标题。不要删除或链接可能会中断。 -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Trait Bound 语法

`impl Trait` 语法适用于简单的情况，但它实际上是称为 _trait bound_ 的较长形式的语法糖；它看起来像这样：

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

这种较长形式与上一节中的示例等效，但更冗长。我们将 trait bounds 放在泛型类型参数声明之后，冒号后面和尖括号内。

`impl Trait` 语法很方便，并且在简单情况下可以使代码更简洁，而完整的 trait bound 语法可以在其他情况下表达更复杂的逻辑。例如，我们可以有两个实现了 `Summary` 的参数。使用 `impl Trait` 语法看起来像这样：

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

如果我们希望此函数允许 `item1` 和 `item2` 具有不同的类型（只要两种类型都实现了 `Summary`），那么使用 `impl Trait` 是合适的。但是，如果我们希望强制两个参数具有相同的类型，则必须使用 trait bound，如下所示：

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

指定为 `item1` 和 `item2` 参数类型的泛型类型 `T` 约束了该函数，使得传递给 `item1` 和 `item2` 参数的具体类型必须相同。

#### 使用 `+` 语法指定多个 Trait Bounds

我们还可以指定多个 trait bounds。假设我们希望 `notify` 在 `item` 上使用显示格式以及 `summarize`：我们在 `notify` 定义中指定 `item` 必须同时实现 `Display` 和 `Summary`。我们可以使用 `+` 语法来做到这一点：

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

`+` 语法也适用于泛型类型的 trait bounds：

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

指定了两个 trait bounds 后，`notify` 的主体可以调用 `summarize` 并使用 `{}` 格式化 `item`。

#### 使用 `where` 子句使 Trait Bounds 更清晰

使用过多的 trait bounds 有其缺点。每个泛型都有自己的 trait bounds，因此具有多个泛型类型参数的函数可能会在函数名称和其参数列表之间包含大量 trait bound 信息，使得函数签名难以阅读。出于这个原因，Rust 提供了在函数签名之后使用 `where` 子句指定 trait bounds 的替代语法。因此，与其这样写：

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

我们可以使用 `where` 子句，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

此函数的签名不那么杂乱：函数名称、参数列表和返回类型紧密地放在一起，类似于没有大量 trait bounds 的函数。

### 返回实现 Traits 的类型

我们还可以在返回位置使用 `impl Trait` 语法来返回实现了某个 trait 的某种类型的值，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

通过使用 `impl Summary` 作为返回类型，我们指定 `returns_summarizable` 函数返回某种实现了 `Summary` trait 的类型，而不命名具体类型。在这种情况下，`returns_summarizable` 返回一个 `SocialPost`，但调用此函数的代码不需要知道这一点。

仅通过它实现的 trait 指定返回类型的能力在闭包和迭代器的上下文中特别有用，我们将在第 13 章中介绍。闭包和迭代器创建的类型只有编译器知道或类型非常长。`impl Trait` 语法让你可以简洁地指定函数返回某种实现了 `Iterator` trait 的类型，而无需写出非常长的类型。

但是，只有在返回单一类型时才能使用 `impl Trait`。例如，此代码返回 `NewsArticle` 或 `SocialPost`，并将返回类型指定为 `impl Summary`，这是不允许的：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

由于编译器如何实现 `impl Trait` 语法的限制，返回 `NewsArticle` 或 `SocialPost` 是不允许的。我们将在第 18 章的 [“使用允许不同类型值的 Trait 对象”][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore --> 部分介绍如何编写具有此行为的函数。

### 使用 Trait Bounds 有条件地实现方法

通过使用带有泛型类型参数的 `impl` 块的 trait bound，我们可以有条件地为实现了指定 trait 的类型实现方法。例如，Listing 10-15 中的 `Pair<T>` 类型总是实现 `new` 函数以返回 `Pair<T>` 的新实例（回想一下第 5 章的 [“定义方法”][methods]<!-- ignore --> 部分，`Self` 是 `impl` 块的类型的别名，在本例中为 `Pair<T>`）。但在下一个 `impl` 块中，`Pair<T>` 仅在其内部类型 `T` 实现了允许比较的 `PartialOrd` trait 和允许打印的 `Display` trait 时才实现 `cmp_display` 方法。

<Listing number="10-15" file-name="src/lib.rs" caption="根据 trait bounds 有条件地在泛型类型上实现方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

我们还可以为任何实现了另一个 trait 的类型有条件地实现一个 trait。为满足 trait bounds 的任何类型实现 trait 的代码称为 _blanket implementations_，并且在 Rust 标准库中广泛使用。例如，标准库在任何实现了 `Display` trait 的类型上实现了 `ToString` trait。标准库中的 `impl` 块看起来类似于以下代码：

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

因为标准库有这个 blanket implementation，我们可以在任何实现了 `Display` trait 的类型上调用由 `ToString` trait 定义的 `to_string` 方法。例如，我们可以将整数转换为相应的 `String` 值，因为整数实现了 `Display`：

```rust
let s = 3.to_string();
```

Blanket implementations 出现在 trait 文档的“Implementors”部分。

Traits 和 trait bounds 让我们可以使用泛型类型参数来减少重复代码，同时也向编译器指定我们希望泛型类型具有特定行为。然后，编译器可以使用 trait bound 信息来检查与我们代码一起使用的所有具体类型是否提供了正确的行为。在动态类型语言中，如果我们在未定义方法的类型上调用方法，我们会在运行时收到错误。但 Rust 将这些错误移到编译时，因此我们被迫在代码甚至能够运行之前修复问题。此外，我们不必编写在运行时检查行为的代码，因为我们已经在编译时进行了检查。这样做可以提高性能，而不必放弃泛型的灵活性。

[using-trait-objects-that-allow-for-values-of-different-types]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[methods]: ch05-03-method-syntax.html#defining-methods
## 方法语法

_方法_ 与函数类似：我们使用 `fn` 关键字和一个名称来声明它们，它们可以有参数和返回值，并且它们包含一些代码，当方法从其他地方调用时，这些代码会被执行。与函数不同的是，方法是在结构体（或枚举或 trait 对象，我们分别在[第 6 章][enums]<!-- ignore -->和[第 18 章][trait-objects]<!-- ignore -->中介绍）的上下文中定义的，并且它们的第一个参数总是 `self`，它表示调用该方法的结构体实例。

### 定义方法

让我们将 `area` 函数改为一个 `Rectangle` 结构体上的 `area` 方法，如 Listing 5-13 所示。

<Listing number="5-13" file-name="src/main.rs" caption="在 `Rectangle` 结构体上定义一个 `area` 方法">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

为了在 `Rectangle` 的上下文中定义函数，我们为 `Rectangle` 启动了一个 `impl`（实现）块。这个 `impl` 块中的所有内容都将与 `Rectangle` 类型相关联。然后我们将 `area` 函数移动到 `impl` 的大括号内，并将第一个（在这种情况下，也是唯一的）参数在签名和函数体中的所有地方改为 `self`。在 `main` 中，我们调用 `area` 函数并传递 `rect1` 作为参数的地方，我们可以改为使用 _方法语法_ 来调用 `Rectangle` 实例上的 `area` 方法。方法语法跟在实例后面：我们添加一个点，后面跟着方法名、括号和任何参数。

在 `area` 的签名中，我们使用 `&self` 而不是 `rectangle: &Rectangle`。`&self` 实际上是 `self: &Self` 的缩写。在 `impl` 块中，`Self` 类型是 `impl` 块所针对的类型的别名。方法的第一个参数必须是一个名为 `self` 的 `Self` 类型的参数，因此 Rust 允许你在第一个参数位置只使用 `self` 来缩写这个参数。注意，我们仍然需要在 `self` 缩写前使用 `&` 来表示该方法借用 `Self` 实例，就像我们在 `rectangle: &Rectangle` 中所做的那样。方法可以获取 `self` 的所有权，不可变地借用 `self`，就像我们在这里所做的那样，或者可变地借用 `self`，就像它们可以处理任何其他参数一样。

我们在这里选择 `&self` 的原因与我们在函数版本中使用 `&Rectangle` 的原因相同：我们不想获取所有权，我们只想读取结构体中的数据，而不是写入数据。如果我们想要在方法执行过程中改变调用该方法的实例，我们会使用 `&mut self` 作为第一个参数。让一个方法通过仅使用 `self` 作为第一个参数来获取实例的所有权是罕见的；这种技术通常用于当方法将 `self` 转换为其他东西时，并且你希望在转换后阻止调用者使用原始实例。

使用方法而不是函数的主要原因，除了提供方法语法和不必在每个方法的签名中重复 `self` 的类型之外，是为了组织代码。我们将所有可以对类型的实例做的事情放在一个 `impl` 块中，而不是让未来的代码使用者在我们的库中到处搜索 `Rectangle` 的功能。

注意，我们可以选择给方法一个与结构体字段相同的名称。例如，我们可以在 `Rectangle` 上定义一个名为 `width` 的方法：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

在这里，我们选择让 `width` 方法在实例的 `width` 字段的值大于 `0` 时返回 `true`，如果值为 `0` 则返回 `false`：我们可以在同名方法中使用字段来实现任何目的。在 `main` 中，当我们在 `rect1.width` 后面加上括号时，Rust 知道我们指的是 `width` 方法。当我们不使用括号时，Rust 知道我们指的是 `width` 字段。

通常，但并非总是如此，当我们给方法一个与字段相同的名称时，我们希望它只返回字段中的值而不做其他事情。这样的方法被称为 _getters_，Rust 不会像其他一些语言那样自动为结构体字段实现它们。Getters 很有用，因为你可以将字段设为私有，但将方法设为公共，从而作为类型的公共 API 的一部分启用对该字段的只读访问。我们将在[第 7 章][public]<!-- ignore -->中讨论什么是公共和私有，以及如何将字段或方法指定为公共或私有。

> ### `->` 操作符在哪里？
>
> 在 C 和 C++ 中，使用两个不同的操作符来调用方法：如果你直接在对象上调用方法，则使用 `.`；如果你在指向对象的指针上调用方法并需要先解引用指针，则使用 `->`。换句话说，如果 `object` 是一个指针，`object->something()` 类似于 `(*object).something()`。
>
> Rust 没有与 `->` 操作符等价的操作符；相反，Rust 有一个称为 _自动引用和解引用_ 的功能。调用方法是 Rust 中少数几个具有这种行为的地方之一。
>
> 它的工作原理是：当你使用 `object.something()` 调用方法时，Rust 会自动添加 `&`、`&mut` 或 `*`，以便 `object` 与方法的签名匹配。换句话说，以下代码是相同的：
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> 第一个看起来更简洁。这种自动引用行为之所以有效，是因为方法有一个明确的接收者——`self` 的类型。给定方法的接收者和名称，Rust 可以明确地确定方法是读取（`&self`）、修改（`&mut self`）还是消耗（`self`）。Rust 使方法接收者的借用隐式化，这是使所有权在实践中变得符合人体工程学的一个重要部分。

### 带有更多参数的方法

让我们通过在 `Rectangle` 结构体上实现第二个方法来练习使用方法。这次我们希望 `Rectangle` 的一个实例接受另一个 `Rectangle` 实例，并返回 `true` 如果第二个 `Rectangle` 可以完全包含在 `self`（第一个 `Rectangle`）中；否则，它应该返回 `false`。也就是说，一旦我们定义了 `can_hold` 方法，我们希望能够编写如 Listing 5-14 所示的程序。

<Listing number="5-14" file-name="src/main.rs" caption="使用尚未编写的 `can_hold` 方法">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

预期的输出将如下所示，因为 `rect2` 的两个维度都小于 `rect1` 的维度，但 `rect3` 比 `rect1` 更宽：

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

我们知道我们想要定义一个方法，所以它将在 `impl Rectangle` 块内。方法名将是 `can_hold`，并且它将接受另一个 `Rectangle` 的不可变借用作为参数。我们可以通过查看调用方法的代码来告诉参数的类型：`rect1.can_hold(&rect2)` 传入 `&rect2`，这是 `rect2` 的不可变借用，`rect2` 是 `Rectangle` 的一个实例。这是有道理的，因为我们只需要读取 `rect2`（而不是写入，这意味着我们需要一个可变借用），并且我们希望 `main` 保留 `rect2` 的所有权，以便我们可以在调用 `can_hold` 方法后再次使用它。`can_hold` 的返回值将是一个布尔值，实现将检查 `self` 的宽度和高度是否分别大于另一个 `Rectangle` 的宽度和高度。让我们将新的 `can_hold` 方法添加到 Listing 5-13 的 `impl` 块中，如 Listing 5-15 所示。

<Listing number="5-15" file-name="src/main.rs" caption="在 `Rectangle` 上实现 `can_hold` 方法，该方法接受另一个 `Rectangle` 实例作为参数">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

当我们使用 Listing 5-14 中的 `main` 函数运行此代码时，我们将得到我们想要的输出。方法可以接受多个参数，我们在 `self` 参数之后将它们添加到签名中，这些参数的工作方式与函数中的参数相同。

### 关联函数

在 `impl` 块中定义的所有函数都称为 _关联函数_，因为它们与 `impl` 后面的类型相关联。我们可以定义没有 `self` 作为第一个参数的关联函数（因此它们不是方法），因为它们不需要类型的实例来工作。我们已经使用过一个这样的函数：`String::from` 函数，它定义在 `String` 类型上。

不是方法的关联函数通常用于构造函数，这些构造函数将返回结构体的新实例。这些函数通常被称为 `new`，但 `new` 不是一个特殊的名称，也不是语言内置的。例如，我们可以选择提供一个名为 `square` 的关联函数，它将有一个维度参数，并将其用作宽度和高度，从而更容易创建一个正方形的 `Rectangle`，而不必两次指定相同的值：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

函数返回类型和函数体中的 `Self` 关键字是 `impl` 关键字后面出现的类型的别名，在这种情况下是 `Rectangle`。

要调用这个关联函数，我们使用结构体名称和 `::` 语法；`let sq = Rectangle::square(3);` 就是一个例子。这个函数由结构体命名空间化：`::` 语法用于关联函数和模块创建的命名空间。我们将在[第 7 章][modules]<!-- ignore -->中讨论模块。

### 多个 `impl` 块

每个结构体允许有多个 `impl` 块。例如，Listing 5-15 等同于 Listing 5-16 所示的代码，其中每个方法都在自己的 `impl` 块中。

<Listing number="5-16" caption="使用多个 `impl` 块重写 Listing 5-15">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

在这里没有理由将这些方法分离到多个 `impl` 块中，但这是有效的语法。我们将在第 10 章中看到一个多个 `impl` 块有用的案例，在那里我们讨论泛型类型和 trait。

## 总结

结构体允许你创建对你的领域有意义的自定义类型。通过使用结构体，你可以将相关的数据片段保持在一起，并为每个片段命名以使你的代码清晰。在 `impl` 块中，你可以定义与你的类型相关联的函数，而方法是一种关联函数，它允许你指定你的结构体实例的行为。

但结构体并不是你创建自定义类型的唯一方式：让我们转向 Rust 的枚举功能，为你的工具箱添加另一个工具。

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
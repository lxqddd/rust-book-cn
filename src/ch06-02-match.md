<!-- 旧标题。请勿删除，否则链接可能会失效。 -->

<a id="the-match-control-flow-operator"></a>

## `match` 控制流结构

Rust 有一个非常强大的控制流结构，称为 `match`，它允许你将一个值与一系列模式进行比较，然后根据匹配的模式执行代码。模式可以由字面值、变量名、通配符和许多其他内容组成；[第 19 章][ch19-00-patterns]<!-- ignore --> 涵盖了所有不同类型的模式及其作用。`match` 的强大之处在于模式的表达能力以及编译器确保所有可能的情况都得到处理。

你可以将 `match` 表达式想象成一个硬币分拣机：硬币沿着轨道滑下，轨道上有各种大小的孔，每个硬币会从它遇到的第一个合适的孔中掉下去。同样地，值会依次通过 `match` 中的每个模式，并在第一个匹配的模式处“掉入”相关的代码块中执行。

说到硬币，让我们以它们为例使用 `match`！我们可以编写一个函数，该函数接受一个未知的美国硬币，并以类似于计数机的方式确定它是哪种硬币，并返回其价值（以美分为单位），如 Listing 6-3 所示。

<Listing number="6-3" caption="一个枚举和一个 `match` 表达式，该表达式以枚举的变体作为其模式">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

</Listing>

让我们分解一下 `value_in_cents` 函数中的 `match`。首先我们列出 `match` 关键字，后跟一个表达式，在这个例子中是值 `coin`。这看起来与 `if` 使用的条件表达式非常相似，但有一个很大的区别：对于 `if`，条件需要求值为布尔值，但在这里它可以是任何类型。在这个例子中，`coin` 的类型是我们在第一行定义的 `Coin` 枚举。

接下来是 `match` 的分支。一个分支有两个部分：一个模式和一些代码。这里的第一个分支有一个模式，即值 `Coin::Penny`，然后是 `=>` 运算符，它将模式与要运行的代码分开。在这个例子中，代码只是值 `1`。每个分支之间用逗号分隔。

当 `match` 表达式执行时，它会按顺序将结果值与每个分支的模式进行比较。如果模式与值匹配，则执行与该模式关联的代码。如果该模式不匹配值，则继续执行下一个分支，就像在硬币分拣机中一样。我们可以根据需要拥有任意数量的分支：在 Listing 6-3 中，我们的 `match` 有四个分支。

每个分支关联的代码是一个表达式，匹配分支中的表达式的结果值就是整个 `match` 表达式返回的值。

如果匹配分支的代码很短，我们通常不使用花括号，如 Listing 6-3 中每个分支只返回一个值的情况。如果你想在匹配分支中运行多行代码，则必须使用花括号，并且分支后的逗号是可选的。例如，以下代码在每次使用 `Coin::Penny` 调用方法时打印“Lucky penny!”，但仍然返回块的最后一个值 `1`：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### 绑定值的模式

匹配分支的另一个有用特性是它们可以绑定到与模式匹配的值的部分。这就是我们如何从枚举变体中提取值的方式。

举个例子，让我们更改一个枚举变体以在其中保存数据。从 1999 年到 2008 年，美国铸造了 25 美分硬币，每枚硬币的一面都有 50 个州的不同设计。没有其他硬币有州设计，所以只有 25 美分硬币有这个额外的价值。我们可以通过更改 `Quarter` 变体以包含一个 `UsState` 值来将此信息添加到我们的 `enum` 中，如 Listing 6-4 所示。

<Listing number="6-4" caption="一个 `Coin` 枚举，其中 `Quarter` 变体还包含一个 `UsState` 值">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

</Listing>

让我们想象一下，一个朋友正在尝试收集所有 50 个州的 25 美分硬币。当我们按硬币类型分类零钱时，我们还会喊出与每个 25 美分硬币相关的州名，这样如果我们的朋友没有这个州的硬币，他们可以将其添加到他们的收藏中。

在这段代码的 `match` 表达式中，我们向匹配 `Coin::Quarter` 变体值的模式添加了一个名为 `state` 的变量。当 `Coin::Quarter` 匹配时，`state` 变量将绑定到该 25 美分硬币的州值。然后我们可以在该分支的代码中使用 `state`，如下所示：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

如果我们调用 `value_in_cents(Coin::Quarter(UsState::Alaska))`，`coin` 将是 `Coin::Quarter(UsState::Alaska)`。当我们将该值与每个匹配分支进行比较时，直到我们到达 `Coin::Quarter(state)` 时才会匹配。此时，`state` 的绑定将是值 `UsState::Alaska`。然后我们可以在 `println!` 表达式中使用该绑定，从而从 `Quarter` 的 `Coin` 枚举变体中提取出内部的州值。

### 使用 `Option<T>` 进行匹配

在上一节中，我们希望在处理 `Option<T>` 时从 `Some` 情况中提取出内部的 `T` 值；我们也可以像处理 `Coin` 枚举一样使用 `match` 来处理 `Option<T>`！我们不再比较硬币，而是比较 `Option<T>` 的变体，但 `match` 表达式的工作方式保持不变。

假设我们想编写一个函数，该函数接受一个 `Option<i32>`，如果其中有值，则将该值加 1。如果其中没有值，函数应返回 `None` 值，并且不尝试执行任何操作。

这个函数非常容易编写，多亏了 `match`，它将如 Listing 6-5 所示。

<Listing number="6-5" caption="一个在 `Option<i32>` 上使用 `match` 表达式的函数">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

</Listing>

让我们更详细地检查 `plus_one` 的第一次执行。当我们调用 `plus_one(five)` 时，`plus_one` 函数体中的变量 `x` 将具有值 `Some(5)`。然后我们将其与每个匹配分支进行比较：

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

`Some(5)` 值与模式 `None` 不匹配，所以我们继续到下一个分支：

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

`Some(5)` 是否匹配 `Some(i)`？是的！我们有相同的变体。`i` 绑定到 `Some` 中包含的值，因此 `i` 取值为 `5`。然后执行匹配分支中的代码，因此我们将 `i` 的值加 1，并创建一个新的 `Some` 值，其中包含我们的总和 `6`。

现在让我们考虑 Listing 6-5 中 `plus_one` 的第二次调用，其中 `x` 是 `None`。我们进入 `match` 并与第一个分支进行比较：

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

它匹配！没有值可以加，因此程序停止并返回 `=>` 右侧的 `None` 值。因为第一个分支匹配，所以不再比较其他分支。

将 `match` 和枚举结合在许多情况下都很有用。你会在 Rust 代码中经常看到这种模式：对枚举进行 `match`，将变量绑定到内部数据，然后根据它执行代码。一开始可能有点棘手，但一旦你习惯了，你会希望所有语言都有这个功能。它一直是用户的最爱。

### 匹配是穷尽的

我们需要讨论 `match` 的另一个方面：分支的模式必须覆盖所有可能性。考虑这个版本的 `plus_one` 函数，它有一个错误并且不会编译：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

我们没有处理 `None` 情况，因此这段代码会导致错误。幸运的是，Rust 知道如何捕获这个错误。如果我们尝试编译这段代码，我们会得到这个错误：

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

Rust 知道我们没有覆盖所有可能的情况，甚至知道我们忘记了哪个模式！Rust 中的匹配是**穷尽的**：我们必须穷尽所有可能性才能使代码有效。特别是在 `Option<T>` 的情况下，当 Rust 阻止我们忘记显式处理 `None` 情况时，它保护我们免于假设我们有值而实际上可能为空，从而使之前讨论的十亿美元错误成为不可能。

### 通配模式和 `_` 占位符

使用枚举，我们还可以对几个特定值采取特殊操作，但对所有其他值采取一个默认操作。想象我们正在实现一个游戏，如果你掷出 3，你的玩家不会移动，而是获得一顶新帽子。如果你掷出 7，你的玩家会失去一顶帽子。对于所有其他值，你的玩家会在游戏板上移动相应数量的空格。以下是一个实现该逻辑的 `match`，其中骰子掷出的结果是硬编码的，而不是随机值，所有其他逻辑由没有函数体的函数表示，因为实际实现它们超出了本示例的范围：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

对于前两个分支，模式是字面值 `3` 和 `7`。对于覆盖所有其他可能值的最后一个分支，模式是我们选择命名为 `other` 的变量。为 `other` 分支运行的代码通过将其传递给 `move_player` 函数来使用该变量。

这段代码可以编译，即使我们没有列出 `u8` 的所有可能值，因为最后一个模式将匹配所有未明确列出的值。这个通配模式满足了 `match` 必须是穷尽的要求。请注意，我们必须将通配分支放在最后，因为模式是按顺序评估的。如果我们把通配分支放在前面，其他分支将永远不会运行，所以如果我们添加分支到通配分支之后，Rust 会警告我们！

Rust 还有一个模式，当我们想要一个通配但不希望在通配模式中**使用**该值时可以使用：`_` 是一个特殊的模式，它匹配任何值并且不绑定到该值。这告诉 Rust 我们不会使用该值，因此 Rust 不会警告我们有关未使用的变量。

让我们改变游戏的规则：现在，如果你掷出 3 或 7 以外的任何值，你必须重新掷骰子。我们不再需要使用通配值，因此我们可以将代码更改为使用 `_` 而不是名为 `other` 的变量：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

这个示例也满足了穷尽性要求，因为我们在最后一个分支中明确忽略了所有其他值；我们没有忘记任何东西。

最后，我们再次改变游戏的规则，使得如果你掷出 3 或 7 以外的任何值，你的回合不会发生任何事情。我们可以通过使用单元值（我们在[“元组类型”][tuples]<!-- ignore --> 部分提到的空元组类型）作为与 `_` 分支关联的代码来表达这一点：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

在这里，我们明确告诉 Rust，我们不会使用任何不匹配前面分支模式的值，并且在这种情况下我们不希望运行任何代码。

关于模式和匹配的更多内容，我们将在[第 19 章][ch19-00-patterns]<!-- ignore --> 中介绍。现在，我们将继续讨论 `if let` 语法，它在 `match` 表达式有点冗长的情况下非常有用。

[tuples]: ch03-02-data-types.html#the-tuple-type
[ch19-00-patterns]: ch19-00-patterns.html
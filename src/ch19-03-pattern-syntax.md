## 模式语法

在本节中，我们将收集所有在模式中有效的语法，并讨论为什么以及何时可能想要使用每种语法。

### 匹配字面量

正如你在第6章中看到的，你可以直接将模式与字面量匹配。以下代码给出了一些示例：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

这段代码会打印 `one`，因为 `x` 的值是 1。当你希望代码在获得特定具体值时采取行动时，这种语法非常有用。

### 匹配命名变量

命名变量是不可反驳的模式，它们可以匹配任何值，我们在本书中已经多次使用它们。然而，当你在 `match`、`if let` 或 `while let` 表达式中使用命名变量时，会出现一个复杂情况。因为这些表达式都会开启一个新的作用域，所以作为模式一部分声明的变量会遮蔽外部同名的变量，就像所有变量一样。在 Listing 19-11 中，我们声明了一个名为 `x` 的变量，其值为 `Some(5)`，以及一个名为 `y` 的变量，其值为 `10`。然后我们在 `x` 的值上创建了一个 `match` 表达式。查看匹配分支中的模式以及最后的 `println!`，尝试在运行此代码或继续阅读之前弄清楚代码将打印什么。

<Listing number="19-11" file-name="src/main.rs" caption="一个 `match` 表达式，其中一个分支引入了一个新变量，遮蔽了现有的变量 `y`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

让我们逐步分析 `match` 表达式运行时的过程。第一个匹配分支中的模式与 `x` 的定义值不匹配，因此代码继续执行。

第二个匹配分支中的模式引入了一个名为 `y` 的新变量，它将匹配 `Some` 值中的任何值。因为我们在 `match` 表达式内部的新作用域中，所以这是一个新的 `y` 变量，而不是我们在开始时声明的值为 `10` 的 `y`。这个新的 `y` 绑定将匹配 `Some` 中的任何值，这正是我们在 `x` 中拥有的。因此，这个新的 `y` 绑定到 `x` 中 `Some` 的内部值。该值是 `5`，所以该分支的表达式执行并打印 `Matched, y = 5`。

如果 `x` 是 `None` 而不是 `Some(5)`，前两个分支中的模式都不会匹配，因此值会匹配到下划线。我们没有在下划线分支的模式中引入 `x` 变量，所以表达式中的 `x` 仍然是未被遮蔽的外部 `x`。在这种假设情况下，`match` 会打印 `Default case, x = None`。

当 `match` 表达式结束时，其作用域结束，内部的 `y` 的作用域也随之结束。最后的 `println!` 产生 `at the end: x = Some(5), y = 10`。

要创建一个比较外部 `x` 和 `y` 值的 `match` 表达式，而不是引入一个遮蔽现有 `y` 变量的新变量，我们需要使用匹配守卫条件。我们将在稍后的 [“使用匹配守卫的额外条件”](#extra-conditionals-with-match-guards)<!-- ignore --> 中讨论匹配守卫。

### 多个模式

你可以使用 `|` 语法匹配多个模式，这是模式的 _或_ 运算符。例如，在以下代码中，我们将 `x` 的值与匹配分支进行匹配，第一个分支有一个 _或_ 选项，意味着如果 `x` 的值匹配该分支中的任何一个值，该分支的代码将运行：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

这段代码会打印 `one or two`。

### 使用 `..=` 匹配值范围

`..=` 语法允许我们匹配一个包含范围的值。在以下代码中，当模式匹配给定范围内的任何值时，该分支将执行：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

如果 `x` 是 `1`、`2`、`3`、`4` 或 `5`，第一个分支将匹配。这种语法比使用 `|` 运算符表达相同的意思更方便；如果我们使用 `|`，我们必须指定 `1 | 2 | 3 | 4 | 5`。指定一个范围要短得多，特别是如果我们想匹配，比如说，1 到 1,000 之间的任何数字！

编译器在编译时检查范围是否为空，因为 Rust 只能判断 `char` 和数值类型的范围是否为空，所以范围只允许用于数值或 `char` 值。

以下是一个使用 `char` 值范围的示例：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust 可以判断 `'c'` 在第一个模式的范围内，并打印 `early ASCII letter`。

### 解构以分解值

我们还可以使用模式来解构结构体、枚举和元组，以使用这些值的不同部分。让我们逐步分析每个值。

#### 解构结构体

Listing 19-12 展示了一个 `Point` 结构体，它有两个字段 `x` 和 `y`，我们可以使用 `let` 语句中的模式将其分解。

<Listing number="19-12" file-name="src/main.rs" caption="将结构体的字段解构为单独的变量">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

这段代码创建了变量 `a` 和 `b`，它们分别匹配 `p` 结构体的 `x` 和 `y` 字段的值。这个示例表明，模式中的变量名不必与结构体的字段名匹配。然而，通常会将变量名与字段名匹配，以便更容易记住哪些变量来自哪些字段。由于这种常见用法，并且因为写 `let Point { x: x, y: y } = p;` 包含大量重复，Rust 为匹配结构体字段的模式提供了一种简写形式：你只需要列出结构体字段的名称，模式创建的变量将具有相同的名称。Listing 19-13 的行为与 Listing 19-12 中的代码相同，但在 `let` 模式中创建的变量是 `x` 和 `y`，而不是 `a` 和 `b`。

<Listing number="19-13" file-name="src/main.rs" caption="使用结构体字段简写形式解构结构体字段">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

这段代码创建了变量 `x` 和 `y`，它们分别匹配 `p` 变量的 `x` 和 `y` 字段。结果是变量 `x` 和 `y` 包含来自 `p` 结构体的值。

我们还可以在结构体模式中使用字面值作为部分模式，而不是为所有字段创建变量。这样做允许我们在创建变量以解构其他字段的同时，测试某些字段的特定值。

在 Listing 19-14 中，我们有一个 `match` 表达式，它将 `Point` 值分为三种情况：直接位于 `x` 轴上的点（当 `y = 0` 时为真）、位于 `y` 轴上的点（`x = 0`）或不在任何轴上的点。

<Listing number="19-14" file-name="src/main.rs" caption="在一个模式中解构并匹配字面值">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

第一个分支将通过指定 `y` 字段的值与字面量 `0` 匹配来匹配任何位于 `x` 轴上的点。该模式仍然创建一个 `x` 变量，我们可以在该分支的代码中使用它。

类似地，第二个分支通过指定 `x` 字段的值与 `0` 匹配来匹配任何位于 `y` 轴上的点，并为 `y` 字段的值创建一个变量 `y`。第三个分支没有指定任何字面量，因此它匹配任何其他 `Point`，并为 `x` 和 `y` 字段创建变量。

在这个例子中，值 `p` 通过 `x` 包含 `0` 匹配第二个分支，因此这段代码将打印 `On the y axis at 7`。

记住，`match` 表达式在找到第一个匹配的模式后就会停止检查分支，所以即使 `Point { x: 0, y: 0}` 位于 `x` 轴和 `y` 轴上，这段代码也只会打印 `On the x axis at 0`。

#### 解构枚举

我们在本书中已经解构过枚举（例如 Listing 6-5），但我们还没有明确讨论过解构枚举的模式与枚举内存储的数据定义方式相对应。作为一个例子，在 Listing 19-15 中，我们使用了 Listing 6-2 中的 `Message` 枚举，并编写了一个 `match`，其中的模式将解构每个内部值。

<Listing number="19-15" file-name="src/main.rs" caption="解构包含不同类型值的枚举变体">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

这段代码将打印 `Change color to red 0, green 160, and blue 255`。尝试更改 `msg` 的值以查看其他分支的代码运行。

对于没有任何数据的枚举变体，如 `Message::Quit`，我们无法进一步解构该值。我们只能匹配字面量 `Message::Quit` 值，并且该模式中没有变量。

对于类似结构体的枚举变体，如 `Message::Move`，我们可以使用类似于我们指定匹配结构体的模式。在变体名称之后，我们放置花括号，然后列出带有变量的字段，以便我们在该分支的代码中使用这些部分。这里我们使用了 Listing 19-13 中的简写形式。

对于类似元组的枚举变体，如 `Message::Write`，它包含一个元素的元组，以及 `Message::ChangeColor`，它包含三个元素的元组，模式类似于我们指定匹配元组的模式。模式中的变量数量必须与我们匹配的变体中的元素数量相匹配。

#### 解构嵌套结构体和枚举

到目前为止，我们的示例都是匹配一层深度的结构体或枚举，但匹配也可以处理嵌套项！例如，我们可以重构 Listing 19-15 中的代码，以支持 `ChangeColor` 消息中的 RGB 和 HSV 颜色，如 Listing 19-16 所示。

<Listing number="19-16" caption="匹配嵌套枚举">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

`match` 表达式中的第一个分支的模式匹配一个包含 `Color::Rgb` 变体的 `Message::ChangeColor` 枚举变体；然后模式绑定到三个内部的 `i32` 值。第二个分支的模式也匹配一个 `Message::ChangeColor` 枚举变体，但内部枚举匹配 `Color::Hsv`。我们可以在一个 `match` 表达式中指定这些复杂的条件，即使涉及两个枚举。

#### 解构结构体和元组

我们可以以更复杂的方式混合、匹配和嵌套解构模式。以下示例展示了一个复杂的解构，我们在元组中嵌套结构体和元组，并解构出所有原始值：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

这段代码让我们将复杂类型分解为其组成部分，以便我们可以分别使用我们感兴趣的值。

使用模式进行解构是一种方便的方式，可以分别使用值的各个部分，例如结构体中的每个字段的值。

### 忽略模式中的值

你已经看到，有时忽略模式中的值是有用的，例如在 `match` 的最后一个分支中，以获得一个不执行任何操作但涵盖所有剩余可能值的 catchall。有几种方法可以忽略模式中的整个值或部分值：使用 `_` 模式（你已经见过）、在另一个模式中使用 `_` 模式、使用以下划线开头的名称，或使用 `..` 忽略值的剩余部分。让我们探讨如何以及为什么要使用这些模式。

<!-- Old link, do not remove -->

<a id="ignoring-an-entire-value-with-_"></a>

#### 使用 `_` 忽略整个值

我们已经使用下划线作为通配符模式，它将匹配任何值但不绑定到该值。这在 `match` 表达式的最后一个分支中特别有用，但我们也可以在任何模式中使用它，包括函数参数，如 Listing 19-17 所示。

<Listing number="19-17" file-name="src/main.rs" caption="在函数签名中使用 `_`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

这段代码将完全忽略作为第一个参数传递的值 `3`，并打印 `This code only uses the y parameter: 4`。

在大多数情况下，当你不再需要某个特定的函数参数时，你会更改签名，使其不包含未使用的参数。忽略函数参数在某些情况下特别有用，例如，当你实现一个需要特定类型签名的 trait 时，但你的实现中的函数体不需要其中一个参数。然后你可以避免收到关于未使用函数参数的编译器警告，就像你使用名称时那样。

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

#### 使用嵌套 `_` 忽略部分值

我们还可以在另一个模式中使用 `_` 来忽略值的部分，例如，当我们只想测试值的部分但不需要在相应的代码中使用其他部分时。Listing 19-18 显示了负责管理设置值的代码。业务要求是用户不应被允许覆盖设置的现有自定义值，但如果当前未设置，则可以取消设置并为其赋值。

<Listing number="19-18" caption="在匹配 `Some` 变体的模式中使用下划线，当我们不需要使用 `Some` 内部的值时">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

这段代码将打印 `Can't overwrite an existing customized value`，然后打印 `setting is Some(5)`。在第一个匹配分支中，我们不需要匹配或使用 `Some` 变体中的值，但我们确实需要测试 `setting_value` 和 `new_setting_value` 都是 `Some` 变体的情况。在这种情况下，我们打印不更改 `setting_value` 的原因，并且它不会被更改。

在所有其他情况下（如果 `setting_value` 或 `new_setting_value` 是 `None`），由第二个分支中的 `_` 模式表示，我们希望允许 `new_setting_value` 成为 `setting_value`。

我们还可以在一个模式中的多个位置使用下划线来忽略特定值。Listing 19-19 展示了忽略五元素元组中的第二个和第四个值的示例。

<Listing number="19-19" caption="忽略元组的多个部分">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs:here}}
```

</Listing>

这段代码将打印 `Some numbers: 2, 8, 32`，并且值 `4` 和 `16` 将被忽略。

<!-- Old link, do not remove -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

#### 通过以下划线开头的名称忽略未使用的变量

如果你创建了一个变量但没有在任何地方使用它，Rust 通常会发出警告，因为未使用的变量可能是一个错误。然而，有时能够创建一个你暂时不会使用的变量是有用的，例如当你进行原型设计或刚刚开始一个项目时。在这种情况下，你可以通过以下划线开头的变量名称告诉 Rust 不要警告你未使用的变量。在 Listing 19-20 中，我们创建了两个未使用的变量，但当我们编译这段代码时，我们应该只收到其中一个变量的警告。

<Listing number="19-20" file-name="src/main.rs" caption="以下划线开头的变量名称以避免收到未使用变量的警告">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

在这里，我们收到了关于未使用变量 `y` 的警告，但没有收到关于未使用 `_x` 的警告。

请注意，仅使用 `_` 和使用以下划线开头的名称之间存在细微差别。语法 `_x` 仍然将值绑定到变量，而 `_` 根本不绑定。为了展示这种区别的重要性，Listing 19-21 将为我们提供一个错误。

<Listing number="19-21" caption="以下划线开头的未使用变量仍然绑定值，这可能会获取值的所有权">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

我们会收到一个错误，因为 `s` 值仍然会被移动到 `_s`，这阻止了我们再次使用 `s`。然而，单独使用下划线不会绑定到值。Listing 19-22 将编译而没有任何错误，因为 `s` 不会被移动到 `_`。

<Listing number="19-22" caption="使用下划线不会绑定值">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

这段代码可以正常工作，因为我们从未将 `s` 绑定到任何东西；它没有被移动。

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

#### 使用 `..` 忽略值的剩余部分

对于具有多个部分的值，我们可以使用 `..` 语法来使用特定部分并忽略其余部分，避免为每个忽略的值列出下划线。`..` 模式忽略我们在模式的其余部分中未明确匹配的任何部分。在 Listing 19-23 中，我们有一个 `Point` 结构体，它保存了三维空间中的坐标。在 `match` 表达式中，我们只想操作 `x` 坐标并忽略 `y` 和 `z` 字段中的值。

<Listing number="19-23" caption="使用 `..` 忽略 `Point` 的所有字段，除了 `x`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

我们列出了 `x` 值，然后只包含了 `..` 模式。这比必须列出 `y: _` 和 `z: _` 要快得多，特别是当我们在处理具有许多字段的结构体时，只有一两个字段是相关的。

`..` 语法将根据需要扩展到尽可能多的值。Listing 19-24 展示了如何将 `..` 与元组一起使用。

<Listing number="19-24" file-name="src/main.rs" caption="仅匹配元组中的第一个和最后一个值，并忽略所有其他值">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

在这段代码中，第一个和最后一个值分别与 `first` 和 `last` 匹配。`..` 将匹配并忽略中间的所有内容。

然而，使用 `..` 必须是无歧义的。如果不清楚哪些值用于匹配，哪些应该被忽略，Rust 会给我们一个错误。Listing 19-25 展示了以歧义方式使用 `..` 的示例，因此它不会编译。

<Listing number="19-25" file-name="src/main.rs" caption="尝试以歧义方式使用 `..`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

当我们编译这个示例时，我们会得到这个错误：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

Rust 无法确定在匹配 `second` 之前要忽略元组中的多少个值，以及之后要忽略多少个值。这段代码可能意味着我们想忽略 `2`，将 `second` 绑定到 `4`，然后忽略 `8`、`16` 和 `32`；或者我们想忽略 `2` 和 `4`，将 `second` 绑定到 `8`，然后忽略 `16` 和 `32`；等等。变量名 `second` 对 Rust 没有任何特殊意义，因此我们得到一个编译器错误，因为在这种方式下使用 `..` 是歧义的。

### 使用匹配守卫的额外条件

_匹配守卫_ 是一个额外的 `if` 条件，在 `match` 分支的模式之后指定，该条件也必须匹配才能选择该分支。匹配守卫对于表达比单独模式更复杂的想法非常有用。请注意，它们仅在 `match` 表达式中可用，而在 `if let` 或 `while let` 表达式中不可用。

条件可以使用模式中创建的变量。Listing 19-26 展示了一个 `match`，其中第一个分支的模式为 `Some(x)`，并且还有一个匹配守卫 `if x % 2 == 0`（如果数字是偶数，则为 `true`）。

<Listing number="19-26" caption="向模式添加匹配守卫">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

这个示例将打印 `The number 4 is even`。当 `num` 与第一个分支中的模式进行比较时，它匹配，因为 `Some(4)` 匹配 `Some(x)`。然后匹配守卫检查 `x` 除以 2 的余数是否等于 0，因为它是，所以选择了第一个分支。

如果 `num` 是 `Some(5)` 而不是 `Some(4)`，第一个分支中的匹配守卫将是 `false`，因为 5 除以 2 的余数是 1，不等于 0。Rust 将转到第二个分支，该分支匹配，因为第二个分支没有匹配守卫，因此匹配任何 `Some` 变体。

没有办法在模式中表达 `if x % 2 == 0` 条件，因此匹配守卫为我们提供了表达这种逻辑的能力。这种额外表达能力的缺点是，当涉及匹配守卫表达式时，编译器不会尝试检查穷尽性。

在 Listing 19-11 中，我们提到我们可以使用匹配守卫来解决我们的模式遮蔽问题。回想一下，我们在 `match` 表达式的模式中创建了一个新变量，而不是使用 `match` 外部的变量。这个新变量意味着我们无法测试外部变量的值。Listing 19-27 展示了我们如何使用匹配守卫来解决这个问题。

<Listing number="19-27" file-name="src/main.rs" caption="使用匹配守卫测试与外部变量的相等性">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

这段代码现在将打印 `Default case, x = Some(5)`。第二个匹配分支中的模式没有引入一个会遮蔽外部 `y` 的新变量 `y`，这意味着我们可以在匹配守卫中使用外部 `y`。我们没有将模式指定为 `Some(y)`，这会遮蔽外部 `y`，而是指定为 `Some(n)`。这创建了一个新变量 `n`，它不会遮蔽任何东西，因为外部没有 `n` 变量。

匹配守卫 `if n == y` 不是一个模式，因此不会引入新变量。这个 `y` _是_ 外部的 `y` 而不是遮蔽它的新 `y`，我们可以通过比较 `n` 和 `y` 来查找与外部 `y` 具有相同值的值。

你还可以在匹配守卫中使用 _或_ 运算符 `|` 来指定多个模式；匹配守卫条件将应用于所有模式。Listing 19-28 展示了将使用 `|` 的模式与匹配守卫结合时的优先级。这个示例的重要部分是 `if y` 匹配守卫适用于 `4`、`5` _和_ `6`，即使看起来 `if y` 只适用于 `6`。

<Listing number="19-28" caption="将多个模式与匹配守卫结合">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

匹配条件规定，只有当 `x` 的值等于 `4`、`5` 或 `6` _并且_ `y` 为 `true` 时，该分支才匹配。当这段代码运行时，第一个分支的模式匹配，因为 `x` 是 `4`，但匹配守卫 `if y` 是 `false`，因此第一个分支未被选择。代码继续到第二个分支，该分支匹配，因此程序打印 `no`。原因是 `if` 条件适用于整个模式 `4 | 5 | 6`，而不仅仅是最后一个值 `6`。换句话说，匹配守卫相对于模式的优先级行为如下：

```text
(4 | 5 | 6) if y => ...
```

而不是：

```text
4 | 5 | (6 if y) => ...
```

运行代码后，优先级行为显而易见：如果匹配守卫仅适用于使用 `|` 运算符指定的值列表中的最后一个值，则该分支将匹配，程序将打印 `yes`。

### `@` 绑定

_at_ 运算符 `@` 允许我们在测试值是否匹配模式的同时创建一个变量来保存该值。在 Listing 19-29 中，我们想测试 `Message::Hello` 的 `id` 字段是否在 `3..=7` 范围内。我们还希望将该值绑定到变量 `id_variable`，以便我们可以在与该分支相关的代码中使用它。我们可以将这个变量命名为 `id`，与字段同名，但在这个示例中，我们将使用不同的名称。

<Listing number="19-29" caption="使用 `@` 在模式中绑定值的同时测试它">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

这个示例将打印 `Found an id in range: 5`。通过在范围 `3..=7` 之前指定 `id_variable @`，我们捕获了匹配该范围的任何值，同时测试该值是否匹配范围模式。

在第二个分支中，我们只在模式中指定了一个范围，与该分支相关的代码没有包含 `id` 字段实际值的变量。`id` 字段的值可能是 10、11 或 12，但与该模式相关的代码不知道它是哪一个。模式代码无法使用 `id` 字段的值，因为我们没有将 `id` 值保存在变量中。

在最后一个分支中，我们指定了一个没有范围的变量，我们确实有一个变量 `id` 可以在该分支的代码中使用。原因是我们使用了结构体字段简写语法。但我们没有像前两个分支那样对 `id` 字段中的值应用任何测试：任何值都会匹配这个模式。

使用 `@` 让我们可以在一个模式中测试一个值并将其保存在变量中。

## 总结

Rust 的模式在区分不同类型的数据时非常有用。当在 `match` 表达式中使用时，Rust 确保你的模式涵盖了所有可能的值，否则你的程序将无法编译。`let` 语句和函数参数中的模式使这些结构更有用，能够在将值分解为较小部分的同时将这些部分分配给变量。我们可以创建简单或复杂的模式来满足我们的需求。

接下来，作为本书的倒数第二章，我们将看看 Rust 各种功能的一些高级方面。
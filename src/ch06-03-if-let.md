## 使用 `if let` 和 `let else` 简化控制流

`if let` 语法允许你将 `if` 和 `let` 结合成一个更简洁的方式来处理匹配某个模式的值，而忽略其他值。考虑 Listing 6-6 中的程序，它匹配 `config_max` 变量中的 `Option<u8>` 值，但只希望在值为 `Some` 变体时执行代码。

<Listing number="6-6" caption="一个只关心当值为 `Some` 时执行代码的 `match`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

如果值是 `Some`，我们通过将值绑定到模式中的变量 `max` 来打印 `Some` 变体中的值。我们不想对 `None` 值做任何处理。为了满足 `match` 表达式，我们必须在处理一个变体后添加 `_ => ()`，这是令人烦恼的样板代码。

相反，我们可以使用 `if let` 以更短的方式编写这段代码。以下代码的行为与 Listing 6-6 中的 `match` 相同：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

`if let` 语法接受一个模式和一个由等号分隔的表达式。它的工作方式与 `match` 相同，其中表达式被传递给 `match`，而模式是其第一个分支。在这种情况下，模式是 `Some(max)`，而 `max` 绑定到 `Some` 内部的值。然后我们可以在 `if let` 块的主体中使用 `max`，就像我们在相应的 `match` 分支中使用 `max` 一样。`if let` 块中的代码仅在值匹配模式时运行。

使用 `if let` 意味着更少的输入、更少的缩进和更少的样板代码。然而，你失去了 `match` 强制执行的穷尽性检查。在 `match` 和 `if let` 之间选择取决于你在特定情况下的操作，以及获得简洁性是否是对失去穷尽性检查的适当权衡。

换句话说，你可以将 `if let` 视为 `match` 的语法糖，当值匹配一个模式时运行代码，然后忽略所有其他值。

我们可以在 `if let` 中包含一个 `else`。与 `else` 关联的代码块与 `match` 表达式中与 `_` 情况关联的代码块相同，该 `match` 表达式等同于 `if let` 和 `else`。回想一下 Listing 6-4 中的 `Coin` 枚举定义，其中 `Quarter` 变体还包含一个 `UsState` 值。如果我们想要计算所有非 `Quarter` 硬币的同时还宣布 `Quarter` 的状态，我们可以使用 `match` 表达式，如下所示：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

或者我们可以使用 `if let` 和 `else` 表达式，如下所示：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## 使用 `let...else` 保持在“快乐路径”上

一个常见的模式是在值存在时执行一些计算，否则返回默认值。继续我们带有 `UsState` 值的硬币示例，如果我们想根据 `Quarter` 上的州的年龄说一些有趣的话，我们可能会在 `UsState` 上引入一个方法来检查州的年龄，如下所示：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

然后我们可能会使用 `if let` 来匹配硬币的类型，在条件的主体中引入一个 `state` 变量，如 Listing 6-7 所示。

<Listing number="6-7" caption="通过在 `if let` 内部嵌套条件来检查一个州是否存在于 1900 年。" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

这完成了任务，但它将工作推到了 `if let` 语句的主体中，如果要完成的工作更复杂，可能很难准确跟踪顶级分支之间的关系。我们还可以利用表达式产生值的事实，从 `if let` 中产生 `state` 或提前返回，如 Listing 6-8 所示。（你也可以用 `match` 做类似的事情。）

<Listing number="6-8" caption="使用 `if let` 产生值或提前返回。" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

不过，这种方式也有点令人烦恼！`if let` 的一个分支产生一个值，而另一个分支则完全从函数返回。

为了使这种常见模式更易于表达，Rust 提供了 `let...else`。`let...else` 语法在左侧接受一个模式，在右侧接受一个表达式，与 `if let` 非常相似，但它没有 `if` 分支，只有一个 `else` 分支。如果模式匹配，它将把模式中的值绑定到外部作用域。如果模式不匹配，程序将进入 `else` 分支，该分支必须从函数返回。

在 Listing 6-9 中，你可以看到当使用 `let...else` 代替 `if let` 时，Listing 6-8 的样子。注意，它保持在函数主体的“快乐路径”上，而不像 `if let` 那样为两个分支显著不同的控制流。

<Listing number="6-9" caption="使用 `let...else` 澄清函数中的流程。" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

如果你遇到程序逻辑过于冗长而无法使用 `match` 表达的情况，请记住 `if let` 和 `let...else` 也在你的 Rust 工具箱中。

## 总结

我们现在已经介绍了如何使用枚举创建可以是一组枚举值之一的自定义类型。我们已经展示了标准库的 `Option<T>` 类型如何帮助你使用类型系统来防止错误。当枚举值内部包含数据时，你可以使用 `match` 或 `if let` 来提取和使用这些值，具体取决于你需要处理多少种情况。

你的 Rust 程序现在可以使用结构体和枚举来表达领域中的概念。创建自定义类型以在你的 API 中使用可以确保类型安全：编译器将确保你的函数只获取每个函数期望的类型的值。

为了向用户提供一个组织良好、易于使用且仅暴露用户所需内容的 API，我们现在转向 Rust 的模块。
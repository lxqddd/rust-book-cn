## 附录 C: 可派生的 Traits

在本书的多个地方，我们讨论了 `derive` 属性，你可以将其应用于结构体或枚举定义。`derive` 属性会生成代码，这些代码将在你使用 `derive` 语法注释的类型上实现一个具有默认实现的 trait。

在本附录中，我们提供了标准库中所有可以与 `derive` 一起使用的 trait 的参考。每个部分涵盖以下内容：

- 派生该 trait 将启用的操作符和方法
- `derive` 提供的 trait 实现的功能
- 实现该 trait 对类型的意义
- 允许或不允许实现该 trait 的条件
- 需要该 trait 的操作示例

如果你希望从 `derive` 属性提供的功能中获得不同的行为，请查阅[标准库文档](../std/index.html)<!-- ignore -->以获取有关如何手动实现它们的详细信息。

这里列出的 trait 是标准库中定义的唯一可以通过 `derive` 在你的类型上实现的 trait。标准库中定义的其他 trait 没有合理的默认行为，因此你需要根据你试图实现的目标来手动实现它们。

一个不能派生的 trait 示例是 `Display`，它处理最终用户的格式化。你应该始终考虑向最终用户显示类型的适当方式。最终用户应该被允许看到类型的哪些部分？他们会发现哪些部分相关？数据的哪种格式对他们最相关？Rust 编译器没有这种洞察力，因此无法为你提供适当的默认行为。

本附录中提供的可派生 trait 列表并不全面：库可以为它们自己的 trait 实现 `derive`，使得你可以使用 `derive` 的 trait 列表真正开放。实现 `derive` 涉及使用过程宏，这在第 20 章的[“宏”][macros]<!-- ignore -->部分中有详细介绍。

### `Debug` 用于程序员输出

`Debug` trait 允许在格式化字符串中进行调试格式化，你可以在 `{}` 占位符中添加 `:?` 来指示。

`Debug` trait 允许你打印类型的实例以进行调试，因此你和其他使用你的类型的程序员可以在程序执行的特定点检查实例。

例如，`Debug` trait 在使用 `assert_eq!` 宏时是必需的。如果相等断言失败，此宏会打印作为参数给出的实例的值，以便程序员可以看到为什么这两个实例不相等。

### `PartialEq` 和 `Eq` 用于相等比较

`PartialEq` trait 允许你比较类型的实例以检查相等性，并启用 `==` 和 `!=` 操作符。

派生 `PartialEq` 会实现 `eq` 方法。当在结构体上派生 `PartialEq` 时，两个实例只有在所有字段都相等时才相等，如果任何字段不相等，则实例不相等。当在枚举上派生时，每个变体等于自身，而不等于其他变体。

例如，`PartialEq` trait 在使用 `assert_eq!` 宏时是必需的，该宏需要能够比较两个类型的实例是否相等。

`Eq` trait 没有方法。它的目的是表明对于注释类型的每个值，该值等于自身。`Eq` trait 只能应用于也实现了 `PartialEq` 的类型，尽管并非所有实现了 `PartialEq` 的类型都可以实现 `Eq`。一个例子是浮点数类型：浮点数的实现规定，两个非数字（`NaN`）值的实例彼此不相等。

一个需要 `Eq` 的例子是在 `HashMap<K, V>` 中存储键，以便 `HashMap<K, V>` 可以判断两个键是否相同。

### `PartialOrd` 和 `Ord` 用于排序比较

`PartialOrd` trait 允许你比较类型的实例以进行排序。实现了 `PartialOrd` 的类型可以与 `<`、`>`、`<=` 和 `>=` 操作符一起使用。你只能将 `PartialOrd` trait 应用于也实现了 `PartialEq` 的类型。

派生 `PartialOrd` 会实现 `partial_cmp` 方法，该方法返回一个 `Option<Ordering>`，当给定的值无法产生排序时，它将返回 `None`。一个无法产生排序的值的例子是浮点数的非数字（`NaN`）值。即使该类型的大多数值可以比较，使用任何浮点数与 `NaN` 浮点值调用 `partial_cmp` 都会返回 `None`。

当在结构体上派生时，`PartialOrd` 通过按结构体定义中字段出现的顺序比较每个字段的值来比较两个实例。当在枚举上派生时，枚举定义中较早声明的变体被认为小于后面列出的变体。

例如，`PartialOrd` trait 在使用 `rand` crate 中的 `gen_range` 方法时是必需的，该方法生成由范围表达式指定的范围内的随机值。

`Ord` trait 允许你知道对于注释类型的任何两个值，将存在一个有效的排序。`Ord` trait 实现了 `cmp` 方法，该方法返回一个 `Ordering` 而不是 `Option<Ordering>`，因为总是会存在一个有效的排序。你只能将 `Ord` trait 应用于也实现了 `PartialOrd` 和 `Eq` 的类型（而 `Eq` 需要 `PartialEq`）。当在结构体和枚举上派生时，`cmp` 的行为与 `PartialOrd` 的派生实现中的 `partial_cmp` 相同。

一个需要 `Ord` 的例子是在 `BTreeSet<T>` 中存储值，这是一个根据值的排序顺序存储数据的数据结构。

### `Clone` 和 `Copy` 用于复制值

`Clone` trait 允许你显式创建一个值的深拷贝，复制过程可能涉及运行任意代码和复制堆数据。有关 `Clone` 的更多信息，请参见第 4 章中的[“变量和数据的交互：Clone”][variables-and-data-interacting-with-clone]<!-- ignore -->。

派生 `Clone` 会实现 `clone` 方法，当为整个类型实现时，它会调用类型的每个部分的 `clone` 方法。这意味着类型中的所有字段或值也必须实现 `Clone` 才能派生 `Clone`。

一个需要 `Clone` 的例子是在切片上调用 `to_vec` 方法。切片不拥有它包含的类型实例，但从 `to_vec` 返回的向量需要拥有其实例，因此 `to_vec` 会对每个项目调用 `clone`。因此，切片中存储的类型必须实现 `Clone`。

`Copy` trait 允许你通过仅复制存储在栈上的位来复制值；不需要运行任意代码。有关 `Copy` 的更多信息，请参见第 4 章中的[“仅栈数据：Copy”][stack-only-data-copy]<!-- ignore -->。

`Copy` trait 没有定义任何方法，以防止程序员重载这些方法并违反不运行任意代码的假设。这样，所有程序员都可以假设复制一个值会非常快。

你可以在任何其所有部分都实现了 `Copy` 的类型上派生 `Copy`。实现了 `Copy` 的类型也必须实现 `Clone`，因为实现了 `Copy` 的类型有一个简单的 `Clone` 实现，执行与 `Copy` 相同的任务。

`Copy` trait 很少需要；实现了 `Copy` 的类型有可用的优化，这意味着你不必调用 `clone`，从而使代码更简洁。

使用 `Copy` 可以完成的所有操作，你也可以使用 `Clone` 完成，但代码可能会更慢或必须在某些地方使用 `clone`。

### `Hash` 用于将值映射到固定大小的值

`Hash` trait 允许你获取一个任意大小的类型的实例，并使用哈希函数将该实例映射到一个固定大小的值。派生 `Hash` 会实现 `hash` 方法。`hash` 方法的派生实现会结合调用类型每个部分的 `hash` 方法的结果，这意味着所有字段或值也必须实现 `Hash` 才能派生 `Hash`。

一个需要 `Hash` 的例子是在 `HashMap<K, V>` 中存储键以高效地存储数据。

### `Default` 用于默认值

`Default` trait 允许你为类型创建默认值。派生 `Default` 会实现 `default` 函数。`default` 函数的派生实现会调用类型每个部分的 `default` 函数，这意味着类型中的所有字段或值也必须实现 `Default` 才能派生 `Default`。

`Default::default` 函数通常与第 5 章中讨论的结构体更新语法结合使用。你可以自定义结构体的几个字段，然后使用 `..Default::default()` 为其余字段设置并使用默认值。

例如，当你在 `Option<T>` 实例上使用 `unwrap_or_default` 方法时，`Default` trait 是必需的。如果 `Option<T>` 是 `None`，`unwrap_or_default` 方法将返回存储在 `Option<T>` 中的类型 `T` 的 `Default::default` 结果。

[creating-instances-from-other-instances-with-struct-update-syntax]: ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax
[stack-only-data-copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
[variables-and-data-interacting-with-clone]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-clone
[macros]: ch20-05-macros.html#macros
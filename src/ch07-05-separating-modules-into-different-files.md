## 将模块分离到不同的文件中

到目前为止，本章中的所有示例都是在一个文件中定义多个模块。当模块变得庞大时，你可能希望将它们的定义移动到单独的文件中，以使代码更易于导航。

例如，让我们从 Listing 7-17 中的代码开始，该代码包含多个餐厅模块。我们将模块提取到文件中，而不是将所有模块都定义在 crate 根文件中。在这种情况下，crate 根文件是 `src/lib.rs`，但这个过程也适用于 crate 根文件为 `src/main.rs` 的二进制 crate。

首先，我们将 `front_of_house` 模块提取到它自己的文件中。删除 `front_of_house` 模块大括号内的代码，只留下 `mod front_of_house;` 声明，这样 `src/lib.rs` 包含的代码如 Listing 7-21 所示。注意，在我们创建 `src/front_of_house.rs` 文件之前，这段代码不会编译通过。

<Listing number="7-21" file-name="src/lib.rs" caption="声明 `front_of_house` 模块，其主体将在 *src/front_of_house.rs* 中">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

接下来，将大括号内的代码放入一个名为 `src/front_of_house.rs` 的新文件中，如 Listing 7-22 所示。编译器知道要查找这个文件，因为它在 crate 根中遇到了名为 `front_of_house` 的模块声明。

<Listing number="7-22" file-name="src/front_of_house.rs" caption="`front_of_house` 模块中的定义在 *src/front_of_house.rs* 中">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

注意，你只需要在模块树中使用 `mod` 声明加载文件 _一次_。一旦编译器知道该文件是项目的一部分（并且知道代码在模块树中的位置，因为你放置了 `mod` 语句），项目中的其他文件应该使用声明路径来引用加载文件的代码，如 [“引用模块树中的项的路径”][paths]<!-- ignore --> 部分所述。换句话说，`mod` 不是你在其他编程语言中可能见过的“包含”操作。

接下来，我们将 `hosting` 模块提取到它自己的文件中。这个过程有点不同，因为 `hosting` 是 `front_of_house` 的子模块，而不是根模块的子模块。我们将 `hosting` 的文件放在一个新目录中，该目录将根据其在模块树中的祖先命名，在这种情况下是 `src/front_of_house`。

为了开始移动 `hosting`，我们将 `src/front_of_house.rs` 改为只包含 `hosting` 模块的声明：

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

然后我们创建一个 `src/front_of_house` 目录和一个 `hosting.rs` 文件，以包含 `hosting` 模块中的定义：

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

如果我们将 `hosting.rs` 放在 `src` 目录中，编译器会期望 `hosting.rs` 代码位于 crate 根中声明的 `hosting` 模块中，而不是作为 `front_of_house` 模块的子模块声明。编译器用于检查哪些模块代码的规则意味着目录和文件更紧密地匹配模块树。

> ### 替代文件路径
>
> 到目前为止，我们已经介绍了 Rust 编译器使用的最惯用的文件路径，但 Rust 也支持一种较旧的文件路径风格。对于在 crate 根中声明的名为 `front_of_house` 的模块，编译器将在以下位置查找模块的代码：
>
> - `src/front_of_house.rs`（我们介绍的内容）
> - `src/front_of_house/mod.rs`（较旧的风格，仍然支持的路径）
>
> 对于作为 `front_of_house` 子模块的名为 `hosting` 的模块，编译器将在以下位置查找模块的代码：
>
> - `src/front_of_house/hosting.rs`（我们介绍的内容）
> - `src/front_of_house/hosting/mod.rs`（较旧的风格，仍然支持的路径）
>
> 如果你对同一个模块使用两种风格，你会得到一个编译器错误。在同一项目中对不同模块使用两种风格的混合是允许的，但可能会让导航你的项目的人感到困惑。
>
> 使用名为 `mod.rs` 的文件的主要缺点是，你的项目可能会以许多名为 `mod.rs` 的文件结束，当你在编辑器中同时打开它们时，这可能会让人感到困惑。

我们已经将每个模块的代码移动到单独的文件中，模块树保持不变。`eat_at_restaurant` 中的函数调用将无需任何修改即可工作，即使定义位于不同的文件中。这种技术允许你在模块大小增长时将它们移动到新文件中。

注意，`src/lib.rs` 中的 `pub use crate::front_of_house::hosting` 语句也没有改变，`use` 也不会对哪些文件作为 crate 的一部分编译产生影响。`mod` 关键字声明模块，Rust 会在与模块同名的文件中查找该模块的代码。

## 总结

Rust 允许你将一个包拆分为多个 crate，并将一个 crate 拆分为模块，以便你可以从一个模块中引用另一个模块中定义的项。你可以通过指定绝对或相对路径来实现这一点。这些路径可以通过 `use` 语句引入作用域，以便你可以在该作用域中多次使用较短的路径。模块代码默认是私有的，但你可以通过添加 `pub` 关键字使定义公开。

在下一章中，我们将介绍标准库中的一些集合数据结构，你可以在你整洁组织的代码中使用它们。

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
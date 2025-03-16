## 定义和实例化结构体

结构体与元组类似，都是在[“元组类型”][tuples]<!-- ignore -->部分讨论的，两者都包含多个相关的值。与元组一样，结构体的各个部分可以是不同的类型。与元组不同的是，在结构体中，你会为每个数据片段命名，以便清楚地知道这些值的含义。添加这些名称意味着结构体比元组更灵活：你不必依赖数据的顺序来指定或访问实例的值。

要定义一个结构体，我们使用关键字 `struct` 并为整个结构体命名。结构体的名称应描述被分组在一起的数据片段的重要性。然后，在大括号内，我们定义数据片段的名称和类型，这些数据片段称为 _字段_。例如，Listing 5-1 展示了一个存储用户账户信息的结构体。

<Listing number="5-1" file-name="src/main.rs" caption="一个 `User` 结构体定义">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

在定义结构体之后，我们可以通过为每个字段指定具体的值来创建该结构体的 _实例_。我们通过声明结构体的名称来创建实例，然后添加包含 _`key: value`_ 对的大括号，其中键是字段的名称，值是我们想要存储在这些字段中的数据。我们不必按照在结构体中声明字段的顺序来指定字段。换句话说，结构体定义就像是该类型的通用模板，而实例则用特定的数据填充该模板以创建该类型的值。例如，我们可以声明一个特定的用户，如 Listing 5-2 所示。

<Listing number="5-2" file-name="src/main.rs" caption="创建一个 `User` 结构体的实例">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

要从结构体中获取特定的值，我们使用点号表示法。例如，要访问此用户的电子邮件地址，我们使用 `user1.email`。如果实例是可变的，我们可以通过点号表示法并赋值给特定字段来更改值。Listing 5-3 展示了如何更改可变 `User` 实例中 `email` 字段的值。

<Listing number="5-3" file-name="src/main.rs" caption="更改 `User` 实例中 `email` 字段的值">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

请注意，整个实例必须是可变的；Rust 不允许我们仅将某些字段标记为可变。与任何表达式一样，我们可以在函数体的最后一个表达式中构造一个新的结构体实例，以隐式返回该新实例。

Listing 5-4 展示了一个 `build_user` 函数，该函数返回一个具有给定电子邮件和用户名的 `User` 实例。`active` 字段的值为 `true`，`sign_in_count` 的值为 `1`。

<Listing number="5-4" file-name="src/main.rs" caption="一个 `build_user` 函数，它接受电子邮件和用户名并返回一个 `User` 实例">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

将函数参数命名为与结构体字段相同的名称是有意义的，但必须重复 `email` 和 `username` 字段名称和变量有点繁琐。如果结构体有更多字段，重复每个名称会更加烦人。幸运的是，有一个方便的简写！

<!-- 旧标题。不要删除或链接可能会中断。 -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### 使用字段初始化简写

因为在 Listing 5-4 中参数名称和结构体字段名称完全相同，我们可以使用 _字段初始化简写_ 语法来重写 `build_user`，使其行为完全相同，但不必重复 `username` 和 `email`，如 Listing 5-5 所示。

<Listing number="5-5" file-name="src/main.rs" caption="一个 `build_user` 函数，使用字段初始化简写，因为 `username` 和 `email` 参数与结构体字段同名">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

在这里，我们正在创建一个 `User` 结构体的新实例，该结构体有一个名为 `email` 的字段。我们希望将 `email` 字段的值设置为 `build_user` 函数的 `email` 参数中的值。因为 `email` 字段和 `email` 参数具有相同的名称，我们只需要写 `email` 而不是 `email: email`。

### 使用结构体更新语法从其他实例创建实例

通常，创建一个包含另一个实例的大部分值但更改某些值的新结构体实例非常有用。你可以使用 _结构体更新语法_ 来实现这一点。

首先，在 Listing 5-6 中，我们展示了如何在没有更新语法的情况下常规地创建一个新的 `User` 实例 `user2`。我们为 `email` 设置一个新值，但其他值使用我们在 Listing 5-2 中创建的 `user1` 的值。

<Listing number="5-6" file-name="src/main.rs" caption="使用 `user1` 的所有值（除了一个）创建一个新的 `User` 实例">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

使用结构体更新语法，我们可以用更少的代码实现相同的效果，如 Listing 5-7 所示。语法 `..` 指定未显式设置的其余字段应与给定实例中的字段具有相同的值。

<Listing number="5-7" file-name="src/main.rs" caption="使用结构体更新语法为 `User` 实例设置一个新的 `email` 值，但使用 `user1` 的其余值">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

Listing 5-7 中的代码也在 `user2` 中创建了一个实例，该实例的 `email` 值不同，但 `username`、`active` 和 `sign_in_count` 字段的值与 `user1` 相同。`..user1` 必须放在最后，以指定任何剩余的字段应从 `user1` 中的相应字段获取其值，但我们可以选择以任何顺序为任意多个字段指定值，而不管这些字段在结构体定义中的顺序如何。

请注意，结构体更新语法使用 `=` 类似于赋值；这是因为它会移动数据，就像我们在[“变量和数据交互：移动”][move]<!-- ignore -->部分看到的那样。在这个例子中，我们在创建 `user2` 之后不能再使用 `user1`，因为 `user1` 的 `username` 字段中的 `String` 被移动到了 `user2` 中。如果我们为 `user2` 的 `email` 和 `username` 都提供了新的 `String` 值，因此只使用了 `user1` 的 `active` 和 `sign_in_count` 值，那么在创建 `user2` 之后，`user1` 仍然有效。`active` 和 `sign_in_count` 都是实现了 `Copy` trait 的类型，因此我们在[“仅栈数据：复制”][copy]<!-- ignore -->部分讨论的行为将适用。在这个例子中，我们仍然可以使用 `user1.email`，因为它的值 _没有_ 被移出。

### 使用没有命名字段的元组结构体创建不同类型

Rust 还支持看起来类似于元组的结构体，称为 _元组结构体_。元组结构体具有结构体名称提供的额外含义，但没有与其字段关联的名称；相反，它们只有字段的类型。当你想要为整个元组命名并使元组成为与其他元组不同的类型时，元组结构体非常有用，而在常规结构体中为每个字段命名可能会显得冗长或多余。

要定义一个元组结构体，以 `struct` 关键字和结构体名称开头，后跟元组中的类型。例如，这里我们定义并使用两个名为 `Color` 和 `Point` 的元组结构体：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

请注意，`black` 和 `origin` 值是不同类型的，因为它们是不同元组结构体的实例。你定义的每个结构体都是其自己的类型，即使结构体中的字段可能具有相同的类型。例如，一个接受 `Color` 类型参数的函数不能接受 `Point` 作为参数，即使这两种类型都由三个 `i32` 值组成。除此之外，元组结构体实例与元组类似，你可以将它们解构为其单独的部分，并且可以使用 `.` 后跟索引来访问单个值。与元组不同，元组结构体在解构时需要命名结构体的类型。例如，我们会写 `let Point(x, y, z) = point`。

### 没有任何字段的类单元结构体

你也可以定义没有任何字段的结构体！这些被称为 _类单元结构体_，因为它们的行为类似于 `()`，即我们在[“元组类型”][tuples]<!-- ignore -->部分提到的单元类型。当你需要在某个类型上实现 trait 但不想在该类型本身中存储任何数据时，类单元结构体非常有用。我们将在第 10 章讨论 trait。以下是一个声明和实例化名为 `AlwaysEqual` 的单元结构体的示例：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

要定义 `AlwaysEqual`，我们使用 `struct` 关键字，然后是我们想要的名称，然后是一个分号。不需要大括号或圆括号！然后我们可以以类似的方式在 `subject` 变量中获取 `AlwaysEqual` 的实例：使用我们定义的名称，不需要任何大括号或圆括号。想象一下，稍后我们将为这种类型实现行为，使得每个 `AlwaysEqual` 实例总是等于任何其他类型的实例，可能是为了测试目的而具有已知结果。我们不需要任何数据来实现这种行为！你将在第 10 章看到如何定义 trait 并在任何类型（包括类单元结构体）上实现它们。

> ### 结构体数据的所有权
>
> 在 Listing 5-1 的 `User` 结构体定义中，我们使用了拥有的 `String` 类型，而不是 `&str` 字符串切片类型。这是一个有意的选择，因为我们希望此结构体的每个实例都拥有其所有数据，并且这些数据在整个结构体有效期间都有效。
>
> 结构体也可以存储对其他地方拥有的数据的引用，但这样做需要使用 _生命周期_，这是 Rust 的一个特性，我们将在第 10 章讨论。生命周期确保结构体引用的数据在结构体有效期间有效。假设你尝试在结构体中存储一个引用而不指定生命周期，如下所示；这将无法工作：
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> 编译器会抱怨需要生命周期说明符：
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> 在第 10 章，我们将讨论如何修复这些错误，以便你可以在结构体中存储引用，但现在，我们将使用拥有的类型（如 `String`）而不是引用（如 `&str`）来修复这些错误。

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
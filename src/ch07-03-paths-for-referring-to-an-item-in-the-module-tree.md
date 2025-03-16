## 模块树中引用项的路径

为了告诉 Rust 在模块树中查找某个项的位置，我们使用路径，就像在文件系统中导航时使用路径一样。要调用一个函数，我们需要知道它的路径。

路径可以有两种形式：

- **绝对路径** 是从 crate 根开始的完整路径；对于外部 crate 的代码，绝对路径以 crate 名称开头，而对于当前 crate 的代码，它以字面量 `crate` 开头。
- **相对路径** 从当前模块开始，使用 `self`、`super` 或当前模块中的标识符。

绝对路径和相对路径后面都跟着一个或多个由双冒号 (`::`) 分隔的标识符。

回到 Listing 7-1，假设我们想调用 `add_to_waitlist` 函数。这相当于问：`add_to_waitlist` 函数的路径是什么？Listing 7-3 包含了 Listing 7-1，但删除了一些模块和函数。

我们将展示两种从 crate 根定义的新函数 `eat_at_restaurant` 中调用 `add_to_waitlist` 函数的方法。这些路径是正确的，但还有一个问题会阻止这个示例按原样编译。我们稍后会解释原因。

`eat_at_restaurant` 函数是我们库 crate 的公共 API 的一部分，因此我们用 `pub` 关键字标记它。在 [“使用 `pub` 关键字暴露路径”][pub]<!-- ignore --> 部分，我们将详细介绍 `pub`。

<Listing number="7-3" file-name="src/lib.rs" caption="使用绝对路径和相对路径调用 `add_to_waitlist` 函数">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

</Listing>

在 `eat_at_restaurant` 中第一次调用 `add_to_waitlist` 函数时，我们使用了绝对路径。`add_to_waitlist` 函数与 `eat_at_restaurant` 定义在同一个 crate 中，这意味着我们可以使用 `crate` 关键字来开始绝对路径。然后我们依次包含每个后续模块，直到找到 `add_to_waitlist`。你可以想象一个具有相同结构的文件系统：我们会指定路径 `/front_of_house/hosting/add_to_waitlist` 来运行 `add_to_waitlist` 程序；使用 `crate` 名称从 crate 根开始，就像在 shell 中使用 `/` 从文件系统根开始一样。

在 `eat_at_restaurant` 中第二次调用 `add_to_waitlist` 时，我们使用了相对路径。路径以 `front_of_house` 开头，这是与 `eat_at_restaurant` 定义在同一模块树级别的模块名称。这里的文件系统等效路径是使用路径 `front_of_house/hosting/add_to_waitlist`。以模块名称开头意味着路径是相对的。

选择使用相对路径还是绝对路径是基于项目的决定，这取决于你是否更有可能将项定义代码与使用该项的代码分开移动或一起移动。例如，如果我们将 `front_of_house` 模块和 `eat_at_restaurant` 函数移动到一个名为 `customer_experience` 的模块中，我们需要更新 `add_to_waitlist` 的绝对路径，但相对路径仍然有效。然而，如果我们将 `eat_at_restaurant` 函数单独移动到一个名为 `dining` 的模块中，`add_to_waitlist` 调用的绝对路径将保持不变，但相对路径需要更新。我们通常倾向于指定绝对路径，因为我们更有可能希望独立地移动代码定义和项调用。

让我们尝试编译 Listing 7-3，看看为什么它还不能编译！我们得到的错误如 Listing 7-4 所示。

<Listing number="7-4" caption="构建 Listing 7-3 代码时的编译器错误">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

</Listing>

错误信息说 `hosting` 模块是私有的。换句话说，我们有 `hosting` 模块和 `add_to_waitlist` 函数的正确路径，但 Rust 不允许我们使用它们，因为它无法访问私有部分。在 Rust 中，所有项（函数、方法、结构体、枚举、模块和常量）默认情况下对父模块是私有的。如果你想使一个项（如函数或结构体）私有，你可以将其放在一个模块中。

父模块中的项不能使用子模块中的私有项，但子模块中的项可以使用其祖先模块中的项。这是因为子模块封装并隐藏了它们的实现细节，但子模块可以看到它们定义的上下文。继续我们的比喻，将隐私规则想象成餐厅的后台：那里发生的事情对餐厅顾客是私有的，但办公室经理可以看到并操作他们经营的餐厅中的一切。

Rust 选择让模块系统以这种方式工作，以便隐藏内部实现细节是默认行为。这样，你就知道可以更改内部代码的哪些部分而不会破坏外部代码。然而，Rust 确实给了你通过使用 `pub` 关键字将子模块代码的内部部分暴露给外部祖先模块的选项。

### 使用 `pub` 关键字暴露路径

让我们回到 Listing 7-4 中的错误，它告诉我们 `hosting` 模块是私有的。我们希望父模块中的 `eat_at_restaurant` 函数能够访问子模块中的 `add_to_waitlist` 函数，因此我们用 `pub` 关键字标记 `hosting` 模块，如 Listing 7-5 所示。

<Listing number="7-5" file-name="src/lib.rs" caption="将 `hosting` 模块声明为 `pub` 以便从 `eat_at_restaurant` 中使用它">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs:here}}
```

</Listing>

不幸的是，Listing 7-5 中的代码仍然导致编译器错误，如 Listing 7-6 所示。

<Listing number="7-6" caption="构建 Listing 7-5 代码时的编译器错误">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

</Listing>

发生了什么？在 `mod hosting` 前面添加 `pub` 关键字使模块变为公共的。有了这个更改，如果我们能访问 `front_of_house`，我们就能访问 `hosting`。但 `hosting` 的**内容**仍然是私有的；使模块公共并不会使其内容变为公共。模块上的 `pub` 关键字只允许其祖先模块中的代码引用它，而不是访问其内部代码。因为模块是容器，仅使模块公共并不能做太多事情；我们需要进一步选择使模块中的一个或多个项也变为公共。

Listing 7-6 中的错误说 `add_to_waitlist` 函数是私有的。隐私规则适用于结构体、枚举、函数和方法以及模块。

让我们也通过在 `add_to_waitlist` 函数的定义前添加 `pub` 关键字使其变为公共的，如 Listing 7-7 所示。

<Listing number="7-7" file-name="src/lib.rs" caption="将 `pub` 关键字添加到 `mod hosting` 和 `fn add_to_waitlist` 允许我们从 `eat_at_restaurant` 调用该函数">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs:here}}
```

</Listing>

现在代码将编译！为了了解为什么添加 `pub` 关键字允许我们在 `eat_at_restaurant` 中使用这些路径，让我们看一下绝对路径和相对路径。

在绝对路径中，我们从 `crate` 开始，这是我们 crate 模块树的根。`front_of_house` 模块定义在 crate 根中。虽然 `front_of_house` 不是公共的，但因为 `eat_at_restaurant` 函数与 `front_of_house` 定义在同一个模块中（即 `eat_at_restaurant` 和 `front_of_house` 是兄弟），我们可以从 `eat_at_restaurant` 引用 `front_of_house`。接下来是标记为 `pub` 的 `hosting` 模块。我们可以访问 `hosting` 的父模块，因此我们可以访问 `hosting`。最后，`add_to_waitlist` 函数标记为 `pub`，我们可以访问其父模块，因此这个函数调用是有效的！

在相对路径中，逻辑与绝对路径相同，除了第一步：路径不是从 crate 根开始，而是从 `front_of_house` 开始。`front_of_house` 模块与 `eat_at_restaurant` 定义在同一个模块中，因此从 `eat_at_restaurant` 定义的模块开始的相对路径是有效的。然后，因为 `hosting` 和 `add_to_waitlist` 标记为 `pub`，路径的其余部分也是有效的，这个函数调用是有效的！

如果你计划共享你的库 crate 以便其他项目可以使用你的代码，你的公共 API 是你与 crate 用户的合同，决定了他们如何与你的代码交互。有许多关于管理公共 API 更改的考虑，以使人们更容易依赖你的 crate。这些考虑超出了本书的范围；如果你对这个主题感兴趣，请参阅 [The Rust API Guidelines][api-guidelines]。

> #### 包含二进制文件和库的包的最佳实践
>
> 我们提到一个包可以同时包含一个 _src/main.rs_ 二进制 crate 根和一个 _src/lib.rs_ 库 crate 根，并且默认情况下两个 crate 都会使用包的名称。通常，包含库和二进制 crate 的包会在二进制 crate 中包含足够的代码来启动一个可执行文件，该可执行文件调用库 crate 中的代码。这使得其他项目可以受益于包提供的大部分功能，因为库 crate 的代码可以共享。
>
> 模块树应该在 _src/lib.rs_ 中定义。然后，任何公共项都可以通过在路径前加上包的名称在二进制 crate 中使用。二进制 crate 成为库 crate 的用户，就像完全外部的 crate 使用库 crate 一样：它只能使用公共 API。这有助于你设计一个好的 API；你不仅是作者，你还是客户！
>
> 在 [第 12 章][ch12]<!-- ignore --> 中，我们将通过一个包含二进制 crate 和库 crate 的命令行程序来演示这种组织实践。

### 使用 `super` 开始相对路径

我们可以通过在路径开头使用 `super` 来构造从父模块开始的相对路径，而不是从当前模块或 crate 根开始。这就像在文件系统路径中使用 `..` 语法一样。使用 `super` 允许我们引用我们知道在父模块中的项，这可以使在模块树中重新排列模块时更容易，因为模块与父模块密切相关，但父模块可能有一天会被移动到模块树的其他位置。

考虑 Listing 7-8 中的代码，它模拟了厨师修复错误订单并亲自将其送到顾客手中的情况。`back_of_house` 模块中定义的 `fix_incorrect_order` 函数通过指定以 `super` 开头的路径调用父模块中定义的 `deliver_order` 函数。

<Listing number="7-8" file-name="src/lib.rs" caption="使用以 `super` 开头的相对路径调用函数">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

</Listing>

`fix_incorrect_order` 函数在 `back_of_house` 模块中，因此我们可以使用 `super` 进入 `back_of_house` 的父模块，在本例中是 `crate`，即根。从那里，我们查找 `deliver_order` 并找到它。成功！我们认为 `back_of_house` 模块和 `deliver_order` 函数可能会保持彼此的关系，并在我们决定重新组织 crate 的模块树时一起移动。因此，我们使用了 `super`，以便将来如果这段代码被移动到不同的模块中，我们需要更新的代码会更少。

### 使结构体和枚举变为公共

我们也可以使用 `pub` 来将结构体和枚举标记为公共的，但在结构体和枚举中使用 `pub` 有一些额外的细节。如果我们在结构体定义前使用 `pub`，我们使结构体变为公共的，但结构体的字段仍然是私有的。我们可以逐个字段决定是否使其变为公共的。在 Listing 7-9 中，我们定义了一个公共的 `back_of_house::Breakfast` 结构体，其中 `toast` 字段是公共的，但 `seasonal_fruit` 字段是私有的。这模拟了餐厅中顾客可以选择餐点中的面包类型，但厨师根据季节和库存决定搭配的水果的情况。可用的水果变化很快，因此顾客不能选择水果，甚至看不到他们将得到哪种水果。

<Listing number="7-9" file-name="src/lib.rs" caption="一个带有一些公共字段和一些私有字段的结构体">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

</Listing>

因为 `back_of_house::Breakfast` 结构体中的 `toast` 字段是公共的，所以在 `eat_at_restaurant` 中我们可以使用点符号写入和读取 `toast` 字段。注意，我们不能在 `eat_at_restaurant` 中使用 `seasonal_fruit` 字段，因为 `seasonal_fruit` 是私有的。尝试取消注释修改 `seasonal_fruit` 字段值的行，看看你会得到什么错误！

另外，请注意，因为 `back_of_house::Breakfast` 有一个私有字段，结构体需要提供一个公共的关联函数来构造 `Breakfast` 的实例（我们在这里将其命名为 `summer`）。如果 `Breakfast` 没有这样的函数，我们就不能在 `eat_at_restaurant` 中创建 `Breakfast` 的实例，因为我们不能在 `eat_at_restaurant` 中设置私有字段 `seasonal_fruit` 的值。

相比之下，如果我们使枚举变为公共的，它的所有变体也会变为公共的。我们只需要在 `enum` 关键字前加上 `pub`，如 Listing 7-10 所示。

<Listing number="7-10" file-name="src/lib.rs" caption="将枚举标记为公共的会使它的所有变体变为公共的。">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

</Listing>

因为我们使 `Appetizer` 枚举变为公共的，所以我们可以在 `eat_at_restaurant` 中使用 `Soup` 和 `Salad` 变体。

枚举的变体如果不公开就没有什么用处；如果必须在每种情况下都用 `pub` 注释所有枚举变体，那将非常烦人，因此枚举变体的默认行为是公开的。结构体通常在没有其字段公开的情况下也很有用，因此结构体字段遵循默认情况下所有内容都是私有的规则，除非用 `pub` 注释。

还有一种涉及 `pub` 的情况我们还没有介绍，这是我们最后一个模块系统特性：`use` 关键字。我们将首先单独介绍 `use`，然后展示如何将 `pub` 和 `use` 结合使用。

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
## 定义模块以控制作用域和隐私

在本节中，我们将讨论模块以及模块系统的其他部分，即 _路径_，它允许你为项命名；`use` 关键字，它将路径引入作用域；以及 `pub` 关键字，用于使项公开。我们还将讨论 `as` 关键字、外部包和 glob 操作符。

### 模块速查表

在我们深入了解模块和路径的细节之前，这里我们提供了一个关于模块、路径、`use` 关键字和 `pub` 关键字在编译器中如何工作的快速参考，以及大多数开发者如何组织他们的代码。我们将在本章中通过示例逐一解释这些规则，但这是一个很好的地方，可以作为模块如何工作的提醒。

- **从 crate 根开始**：当编译一个 crate 时，编译器首先在 crate 根文件（通常是库 crate 的 _src/lib.rs_ 或二进制 crate 的 _src/main.rs_）中查找要编译的代码。
- **声明模块**：在 crate 根文件中，你可以声明新模块；假设你声明了一个“garden”模块，使用 `mod garden;`。编译器将在以下位置查找模块的代码：
  - 内联，在替换 `mod garden` 后面的分号的花括号内
  - 在文件 _src/garden.rs_ 中
  - 在文件 _src/garden/mod.rs_ 中
- **声明子模块**：在 crate 根文件之外的任何文件中，你可以声明子模块。例如，你可以在 _src/garden.rs_ 中声明 `mod vegetables;`。编译器将在父模块命名的目录中的以下位置查找子模块的代码：
  - 内联，直接在 `mod vegetables` 后面，使用花括号而不是分号
  - 在文件 _src/garden/vegetables.rs_ 中
  - 在文件 _src/garden/vegetables/mod.rs_ 中
- **模块中代码的路径**：一旦模块成为你的 crate 的一部分，只要隐私规则允许，你就可以使用路径从该 crate 的其他任何地方引用该模块中的代码。例如，garden vegetables 模块中的 `Asparagus` 类型可以在 `crate::garden::vegetables::Asparagus` 找到。
- **私有 vs. 公开**：模块中的代码默认对其父模块是私有的。要使模块公开，请使用 `pub mod` 而不是 `mod` 来声明它。要使公共模块中的项也公开，请在它们的声明前使用 `pub`。
- **`use` 关键字**：在作用域内，`use` 关键字创建项的快捷方式，以减少长路径的重复。在任何可以引用 `crate::garden::vegetables::Asparagus` 的作用域中，你可以使用 `use crate::garden::vegetables::Asparagus;` 创建一个快捷方式，然后你只需要写 `Asparagus` 就可以在该作用域中使用该类型。

在这里，我们创建了一个名为 `backyard` 的二进制 crate，以说明这些规则。crate 的目录，也命名为 `backyard`，包含以下文件和目录：

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

在这种情况下，crate 根文件是 _src/main.rs_，它包含以下内容：

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

`pub mod garden;` 行告诉编译器包含它在 _src/garden.rs_ 中找到的代码，即：

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

在这里，`pub mod vegetables;` 意味着 _src/garden/vegetables.rs_ 中的代码也被包含。该代码是：

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

现在让我们深入了解这些规则，并演示它们的实际应用！

### 在模块中分组相关代码

_模块_ 让我们可以在 crate 内组织代码以提高可读性和易于重用。模块还允许我们控制项的 _隐私_，因为模块中的代码默认是私有的。私有项是内部实现细节，不对外部使用。我们可以选择使模块及其中的项公开，从而允许外部代码使用和依赖它们。

作为一个例子，让我们编写一个提供餐厅功能的库 crate。我们将定义函数的签名，但将其主体留空，以专注于代码的组织而不是餐厅的实现。

在餐饮业中，餐厅的某些部分被称为 _前厅_，其他部分被称为 _后厨_。前厅是顾客所在的地方；这包括接待员安排顾客座位、服务员接受订单和付款以及调酒师制作饮料的地方。后厨是厨师和厨师在厨房工作、洗碗工清理和管理员进行行政工作的地方。

为了以这种方式构建我们的 crate，我们可以将其功能组织到嵌套模块中。通过运行 `cargo new restaurant --lib` 创建一个名为 `restaurant` 的新库。然后将代码清单 7-1 中的代码输入到 _src/lib.rs_ 中，以定义一些模块和函数签名；这段代码是前厅部分。

<Listing number="7-1" file-name="src/lib.rs" caption="包含其他模块的 `front_of_house` 模块，这些模块包含函数">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

我们使用 `mod` 关键字后跟模块名称（在本例中为 `front_of_house`）来定义一个模块。模块的主体然后放在花括号内。在模块内部，我们可以放置其他模块，如本例中的 `hosting` 和 `serving` 模块。模块还可以包含其他项的定义，例如结构体、枚举、常量、特性以及如代码清单 7-1 中的函数。

通过使用模块，我们可以将相关的定义分组并命名它们之间的关系。使用此代码的程序员可以根据组导航代码，而不必阅读所有定义，从而更容易找到与他们相关的定义。向此代码添加新功能的程序员将知道将代码放在何处以保持程序的组织性。

之前，我们提到 _src/main.rs_ 和 _src/lib.rs_ 被称为 crate 根。它们之所以得名，是因为这两个文件中的任何一个的内容都形成了一个名为 `crate` 的模块，位于 crate 模块结构的根部，称为 _模块树_。

代码清单 7-2 显示了代码清单 7-1 中结构的模块树。

<Listing number="7-2" caption="代码清单 7-1 中代码的模块树">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

这棵树显示了一些模块如何嵌套在其他模块中；例如，`hosting` 嵌套在 `front_of_house` 中。该树还显示了一些模块是 _兄弟_，这意味着它们在同一模块中定义；`hosting` 和 `serving` 是在 `front_of_house` 中定义的兄弟。如果模块 A 包含在模块 B 中，我们说模块 A 是模块 B 的 _子模块_，模块 B 是模块 A 的 _父模块_。请注意，整个模块树都根植于名为 `crate` 的隐式模块下。

模块树可能会让你想起计算机上的文件系统目录树；这是一个非常恰当的比喻！就像文件系统中的目录一样，你使用模块来组织代码。就像目录中的文件一样，我们需要一种方法来找到我们的模块。
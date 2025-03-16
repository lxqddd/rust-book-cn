## 使用 `use` 关键字将路径引入作用域

每次调用函数时都写出路径可能会让人感到不便和重复。在 Listing 7-7 中，无论我们选择的是绝对路径还是相对路径来调用 `add_to_waitlist` 函数，每次我们想要调用 `add_to_waitlist` 时，都必须指定 `front_of_house` 和 `hosting`。幸运的是，有一种方法可以简化这个过程：我们可以使用 `use` 关键字一次性创建一个路径的快捷方式，然后在作用域内的其他地方使用这个更短的名字。

在 Listing 7-11 中，我们将 `crate::front_of_house::hosting` 模块引入到 `eat_at_restaurant` 函数的作用域中，这样我们只需要指定 `hosting::add_to_waitlist` 就可以在 `eat_at_restaurant` 中调用 `add_to_waitlist` 函数。

<Listing number="7-11" file-name="src/lib.rs" caption="使用 `use` 将模块引入作用域">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

在作用域中添加 `use` 和路径类似于在文件系统中创建一个符号链接。通过在 crate 根目录中添加 `use crate::front_of_house::hosting`，`hosting` 现在在该作用域中是一个有效的名字，就像 `hosting` 模块被定义在 crate 根目录中一样。使用 `use` 引入作用域的路径也会像其他路径一样检查私有性。

需要注意的是，`use` 只会在其出现的作用域中创建快捷方式。Listing 7-12 将 `eat_at_restaurant` 函数移动到一个名为 `customer` 的新子模块中，这个子模块的作用域与 `use` 语句的作用域不同，因此函数体将无法编译。

<Listing number="7-12" file-name="src/lib.rs" caption="`use` 语句只在其所在的作用域内有效">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

编译器错误显示，快捷方式在 `customer` 模块中不再有效：

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

注意，还有一个警告提示 `use` 在其作用域中不再使用！要解决这个问题，可以将 `use` 也移动到 `customer` 模块中，或者在子模块 `customer` 中使用 `super::hosting` 引用父模块中的快捷方式。

### 创建惯用的 `use` 路径

在 Listing 7-11 中，你可能会好奇为什么我们指定了 `use crate::front_of_house::hosting`，然后在 `eat_at_restaurant` 中调用 `hosting::add_to_waitlist`，而不是直接将 `use` 路径一直延伸到 `add_to_waitlist` 函数，以达到相同的结果，如 Listing 7-13 所示。

<Listing number="7-13" file-name="src/lib.rs" caption="使用 `use` 将 `add_to_waitlist` 函数引入作用域，这种方式不惯用">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

尽管 Listing 7-11 和 Listing 7-13 完成了相同的任务，但 Listing 7-11 是使用 `use` 将函数引入作用域的惯用方式。使用 `use` 将函数的父模块引入作用域意味着我们在调用函数时必须指定父模块。在调用函数时指定父模块可以清楚地表明该函数不是在本地定义的，同时还能最大限度地减少完整路径的重复。Listing 7-13 中的代码不清楚 `add_to_waitlist` 是在哪里定义的。

另一方面，当使用 `use` 引入结构体、枚举和其他项时，惯用的做法是指定完整路径。Listing 7-14 展示了将标准库的 `HashMap` 结构体引入二进制 crate 作用域的惯用方式。

<Listing number="7-14" file-name="src/main.rs" caption="以惯用方式将 `HashMap` 引入作用域">

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

这种惯用方式背后没有特别的原因：这只是已经形成的惯例，人们已经习惯了以这种方式阅读和编写 Rust 代码。

这种惯用方式的例外情况是，如果我们使用 `use` 语句将两个同名的项引入作用域，因为 Rust 不允许这样做。Listing 7-15 展示了如何将两个同名但父模块不同的 `Result` 类型引入作用域，以及如何引用它们。

<Listing number="7-15" file-name="src/lib.rs" caption="将两个同名类型引入同一作用域需要使用它们的父模块">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

如你所见，使用父模块可以区分这两个 `Result` 类型。如果我们指定 `use std::fmt::Result` 和 `use std::io::Result`，我们将在同一作用域中有两个 `Result` 类型，Rust 将不知道我们使用 `Result` 时指的是哪一个。

### 使用 `as` 关键字提供新名称

使用 `use` 将两个同名类型引入同一作用域的另一种解决方案是：在路径之后，我们可以指定 `as` 和一个新的本地名称，或称为 _别名_。Listing 7-16 展示了通过使用 `as` 重命名其中一个 `Result` 类型来编写 Listing 7-15 中的代码的另一种方式。

<Listing number="7-16" file-name="src/lib.rs" caption="使用 `as` 关键字将类型引入作用域时重命名">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

在第二个 `use` 语句中，我们为 `std::io::Result` 类型选择了新名称 `IoResult`，这样它就不会与我们从 `std::fmt` 引入的 `Result` 冲突。Listing 7-15 和 Listing 7-16 都被认为是惯用的，所以选择权在你手中！

### 使用 `pub use` 重新导出名称

当我们使用 `use` 关键字将名称引入作用域时，新作用域中的名称是私有的。为了使调用我们代码的代码能够像在其作用域中定义的那样引用该名称，我们可以将 `pub` 和 `use` 结合起来。这种技术称为 _重新导出_，因为我们不仅将项引入作用域，还使其可供其他人引入他们的作用域。

Listing 7-17 展示了 Listing 7-11 中的代码，将根模块中的 `use` 改为 `pub use`。

<Listing number="7-17" file-name="src/lib.rs" caption="使用 `pub use` 使名称可供任何代码从新作用域中使用">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

在此更改之前，外部代码必须使用路径 `restaurant::front_of_house::hosting::add_to_waitlist()` 来调用 `add_to_waitlist` 函数，这也要求 `front_of_house` 模块被标记为 `pub`。现在，由于 `pub use` 从根模块重新导出了 `hosting` 模块，外部代码可以使用路径 `restaurant::hosting::add_to_waitlist()`。

重新导出在你代码的内部结构与调用你代码的程序员对领域的思考方式不同时非常有用。例如，在这个餐厅的比喻中，经营餐厅的人会想到“前厅”和“后厨”。但来餐厅用餐的顾客可能不会用这些术语来思考餐厅的各个部分。通过 `pub use`，我们可以用一种结构编写代码，但暴露另一种结构。这样做可以使我们的库对编写库的程序员和调用库的程序员都组织良好。我们将在第 14 章的 [“使用 `pub use` 导出方便的公共 API”][ch14-pub-use]<!-- ignore --> 中看到另一个 `pub use` 的例子以及它如何影响你的 crate 文档。

### 使用外部包

在第 2 章中，我们编写了一个猜数字游戏项目，该项目使用了一个名为 `rand` 的外部包来获取随机数。要在我们的项目中使用 `rand`，我们在 _Cargo.toml_ 中添加了这行代码：

<!-- 当更新 `rand` 使用的版本时，也更新这些文件中使用的 `rand` 版本，以便它们都匹配：
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

在 _Cargo.toml_ 中添加 `rand` 作为依赖项会告诉 Cargo 从 [crates.io](https://crates.io/) 下载 `rand` 包及其所有依赖项，并使 `rand` 可用于我们的项目。

然后，为了将 `rand` 的定义引入我们的包的作用域，我们添加了一个以 crate 名称 `rand` 开头的 `use` 行，并列出了我们想要引入作用域的项。回想一下，在第 2 章的 [“生成随机数”][rand]<!-- ignore --> 中，我们将 `Rng` trait 引入作用域并调用了 `rand::thread_rng` 函数：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Rust 社区的成员已经在 [crates.io](https://crates.io/) 上提供了许多包，将其中任何一个引入你的包都涉及相同的步骤：在你的包的 _Cargo.toml_ 文件中列出它们，并使用 `use` 将其 crate 中的项引入作用域。

需要注意的是，标准 `std` 库也是一个外部 crate。由于标准库是随 Rust 语言一起发布的，我们不需要更改 _Cargo.toml_ 来包含 `std`。但我们确实需要使用 `use` 来将其中的项引入我们包的作用域。例如，对于 `HashMap`，我们会使用这行代码：

```rust
use std::collections::HashMap;
```

这是一个以 `std` 开头的绝对路径，`std` 是标准库 crate 的名称。

### 使用嵌套路径清理大量的 `use` 列表

如果我们使用同一 crate 或同一模块中定义的多个项，将每个项单独列在一行可能会占用文件中的大量垂直空间。例如，我们在 Listing 2-4 的猜数字游戏中有这两个 `use` 语句，将 `std` 中的项引入作用域：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

相反，我们可以使用嵌套路径在一行中将相同的项引入作用域。我们通过指定路径的公共部分，后跟两个冒号，然后用花括号括起路径的不同部分来实现这一点，如 Listing 7-18 所示。

<Listing number="7-18" file-name="src/main.rs" caption="指定嵌套路径以将具有相同前缀的多个项引入作用域">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

在较大的程序中，使用嵌套路径从同一 crate 或模块中引入许多项可以大大减少所需的单独 `use` 语句的数量！

我们可以在路径的任何级别使用嵌套路径，这在组合两个共享子路径的 `use` 语句时非常有用。例如，Listing 7-19 展示了两个 `use` 语句：一个将 `std::io` 引入作用域，另一个将 `std::io::Write` 引入作用域。

<Listing number="7-19" file-name="src/lib.rs" caption="两个 `use` 语句，其中一个语句是另一个语句的子路径">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

这两个路径的公共部分是 `std::io`，这是第一个完整路径。要将这两个路径合并为一个 `use` 语句，我们可以在嵌套路径中使用 `self`，如 Listing 7-20 所示。

<Listing number="7-20" file-name="src/lib.rs" caption="将 Listing 7-19 中的路径合并为一个 `use` 语句">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

这行代码将 `std::io` 和 `std::io::Write` 引入作用域。

### 使用 Glob 运算符

如果我们想将路径中定义的所有公共项引入作用域，可以指定该路径后跟 `*` glob 运算符：

```rust
use std::collections::*;
```

这个 `use` 语句将 `std::collections` 中定义的所有公共项引入当前作用域。使用 glob 运算符时要小心！Glob 可能会使你在程序中使用的名称的来源变得难以辨认。

glob 运算符通常用于测试中，将测试中的所有内容引入 `tests` 模块；我们将在第 11 章的 [“如何编写测试”][writing-tests]<!-- ignore --> 中讨论这一点。glob 运算符有时也作为 prelude 模式的一部分使用：有关该模式的更多信息，请参阅 [标准库文档](../std/prelude/index.html#other-preludes)<!-- ignore -->。

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api-with-pub-use
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests
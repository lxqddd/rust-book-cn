# 编写一个猜数字游戏

让我们通过一个实际项目来快速入门 Rust！本章将通过一个真实的程序向你介绍一些常见的 Rust 概念。你将学习到 `let`、`match`、方法、关联函数、外部 crate 等等！在接下来的章节中，我们将更详细地探讨这些概念。在本章中，你只需练习基础知识。

我们将实现一个经典的初学者编程问题：猜数字游戏。它的工作原理如下：程序会生成一个 1 到 100 之间的随机整数。然后它会提示玩家输入一个猜测。输入猜测后，程序会提示猜测是太低还是太高。如果猜测正确，游戏将打印一条祝贺消息并退出。

## 设置一个新项目

要设置一个新项目，请进入你在第 1 章中创建的 _projects_ 目录，并使用 Cargo 创建一个新项目，如下所示：

```console
$ cargo new guessing_game
$ cd guessing_game
```

第一个命令 `cargo new` 将项目名称（`guessing_game`）作为第一个参数。第二个命令切换到新项目的目录。

查看生成的 _Cargo.toml_ 文件：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">文件名: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

正如你在第 1 章中看到的，`cargo new` 会为你生成一个“Hello, world!”程序。查看 _src/main.rs_ 文件：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

现在让我们使用 `cargo run` 命令编译并运行这个“Hello, world!”程序：

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

当你需要快速迭代项目时，`run` 命令非常有用，就像我们在这个游戏中要做的那样，快速测试每个迭代，然后再进行下一个迭代。

重新打开 _src/main.rs_ 文件。你将在该文件中编写所有代码。

## 处理猜测

猜数字游戏程序的第一部分将要求用户输入，处理该输入，并检查输入是否符合预期格式。首先，我们将允许玩家输入一个猜测。将代码清单 2-1 中的代码输入到 _src/main.rs_ 中。

<Listing number="2-1" file-name="src/main.rs" caption="从用户获取猜测并打印的代码">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

这段代码包含了很多信息，所以我们逐行来看。为了获取用户输入并打印结果，我们需要将 `io` 输入/输出库引入作用域。`io` 库来自标准库，称为 `std`：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

默认情况下，Rust 在标准库中定义了一组项目，并将其引入每个程序的作用域。这个集合称为 _prelude_，你可以在[标准库文档][prelude]中查看其中的所有内容。

如果你想使用的类型不在 prelude 中，你必须使用 `use` 语句显式地将该类型引入作用域。使用 `std::io` 库为你提供了许多有用的功能，包括接受用户输入的能力。

正如你在第 1 章中看到的，`main` 函数是程序的入口点：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

`fn` 语法声明了一个新函数；括号 `()` 表示没有参数；而大括号 `{` 表示函数体的开始。

正如你在第 1 章中学到的，`println!` 是一个将字符串打印到屏幕的宏：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

这段代码打印了一个提示，说明游戏是什么，并要求用户输入。

### 使用变量存储值

接下来，我们将创建一个 _变量_ 来存储用户输入，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

现在程序变得有趣了！这短短的一行代码中有很多内容。我们使用 `let` 语句创建变量。这是另一个例子：

```rust,ignore
let apples = 5;
```

这行代码创建了一个名为 `apples` 的新变量，并将其绑定到值 5。在 Rust 中，变量默认是不可变的，这意味着一旦我们给变量赋值，该值就不会改变。我们将在第 3 章的[“变量和可变性”][variables-and-mutability]<!-- ignore -->部分详细讨论这个概念。要使变量可变，我们在变量名前添加 `mut`：

```rust,ignore
let apples = 5; // 不可变
let mut bananas = 5; // 可变
```

> 注意：`//` 语法开始一个注释，直到行尾。Rust 会忽略注释中的所有内容。我们将在[第 3 章][comments]<!-- ignore -->中更详细地讨论注释。

回到猜数字游戏程序，你现在知道 `let mut guess` 将引入一个名为 `guess` 的可变变量。等号（`=`）告诉 Rust 我们现在要将某些东西绑定到变量上。等号的右边是 `guess` 绑定的值，它是调用 `String::new` 的结果，`String::new` 是一个返回 `String` 新实例的函数。[`String`][string]<!-- ignore --> 是标准库提供的一种字符串类型，它是可增长的、UTF-8 编码的文本。

`::new` 行中的 `::` 语法表示 `new` 是 `String` 类型的关联函数。_关联函数_ 是在类型上实现的函数，在这里是 `String`。这个 `new` 函数创建一个新的空字符串。你会在许多类型上找到 `new` 函数，因为它是创建某种新值的函数的常见名称。

总的来说，`let mut guess = String::new();` 这行代码创建了一个可变变量，该变量当前绑定到一个新的、空的 `String` 实例。哇！

### 接收用户输入

回想一下，我们在程序的第一行使用 `use std::io;` 包含了标准库的输入/输出功能。现在我们将调用 `io` 模块中的 `stdin` 函数，这将允许我们处理用户输入：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

如果我们在程序开头没有使用 `use std::io;` 导入 `io` 库，我们仍然可以通过将函数调用写为 `std::io::stdin` 来使用该函数。`stdin` 函数返回一个 [`std::io::Stdin`][iostdin]<!-- ignore --> 的实例，这是一个表示终端标准输入句柄的类型。

接下来，`.read_line(&mut guess)` 这行代码在标准输入句柄上调用 [`read_line`][read_line]<!-- ignore --> 方法以获取用户输入。我们还传递 `&mut guess` 作为 `read_line` 的参数，告诉它要将用户输入存储在哪个字符串中。`read_line` 的完整工作是获取用户输入到标准输入的任何内容，并将其附加到一个字符串中（而不覆盖其内容），因此我们将该字符串作为参数传递。字符串参数需要是可变的，以便方法可以更改字符串的内容。

`&` 表示这个参数是一个 _引用_，它让你可以让你代码的多个部分访问同一块数据，而不需要多次将数据复制到内存中。引用是一个复杂的功能，Rust 的主要优势之一是如何安全且容易地使用引用。你不需要了解很多细节来完成这个程序。现在，你只需要知道，像变量一样，引用默认是不可变的。因此，你需要写 `&mut guess` 而不是 `&guess` 来使其可变。（第 4 章将更详细地解释引用。）

<!-- 旧标题。不要删除，否则链接可能会中断。 -->

<a id="handling-potential-failure-with-the-result-type"></a>

### 使用 `Result` 处理潜在的错误

我们仍然在处理这行代码。我们现在讨论的是第三行文本，但请注意它仍然是单个逻辑代码行的一部分。接下来的部分是这个方法：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

我们可以将这行代码写成：

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

然而，一行太长的代码难以阅读，所以最好将其分开。当你使用 `.method_name()` 语法调用方法时，通常明智的做法是引入换行符和其他空白符来帮助分解长行。现在让我们讨论这行代码的作用。

如前所述，`read_line` 将用户输入的任何内容放入我们传递给它的字符串中，但它也返回一个 `Result` 值。[`Result`][result]<!-- ignore --> 是一个[_枚举_][enums]<!-- ignore -->，通常称为 _enum_，它是一种可以处于多种可能状态之一的类型。我们将每个可能的状态称为 _变体_。

[第 6 章][enums]<!-- ignore -->将更详细地介绍枚举。这些 `Result` 类型的目的是编码错误处理信息。

`Result` 的变体是 `Ok` 和 `Err`。`Ok` 变体表示操作成功，并且它包含成功生成的值。`Err` 变体表示操作失败，并且它包含有关操作失败的方式或原因的信息。

`Result` 类型的值，像任何类型的值一样，都有定义在其上的方法。`Result` 的实例有一个 [`expect` 方法][expect]<!-- ignore -->，你可以调用它。如果这个 `Result` 实例是一个 `Err` 值，`expect` 将导致程序崩溃并显示你作为参数传递给 `expect` 的消息。如果 `read_line` 方法返回 `Err`，这很可能是由于底层操作系统的错误。如果这个 `Result` 实例是一个 `Ok` 值，`expect` 将获取 `Ok` 所持有的返回值并将其返回给你，以便你可以使用它。在这种情况下，该值是用户输入的字节数。

如果你不调用 `expect`，程序将编译，但你会收到一个警告：

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust 警告你尚未使用从 `read_line` 返回的 `Result` 值，表明程序尚未处理可能的错误。

抑制警告的正确方法是实际编写错误处理代码，但在我们的情况下，我们只想在发生问题时使程序崩溃，所以我们可以使用 `expect`。你将在[第 9 章][recover]<!-- ignore -->中学习如何从错误中恢复。

### 使用 `println!` 占位符打印值

除了右大括号外，到目前为止的代码中只有一行需要讨论：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

这行代码打印现在包含用户输入的字符串。`{}` 大括号集是一个占位符：将 `{}` 想象为小螃蟹钳子，它们将一个值固定在适当的位置。当打印变量的值时，变量名可以放在大括号内。当打印表达式求值的结果时，将空的大括号放在格式字符串中，然后在格式字符串后面加上一个逗号分隔的表达式列表，以按顺序打印到每个空的大括号占位符中。在一个 `println!` 调用中打印变量和表达式结果如下所示：

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

这段代码将打印 `x = 5 and y + 2 = 12`。

### 测试第一部分

让我们测试猜数字游戏的第一部分。使用 `cargo run` 运行它：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

此时，游戏的第一部分已经完成：我们从键盘获取输入并打印它。

## 生成一个秘密数字

接下来，我们需要生成一个用户将尝试猜测的秘密数字。秘密数字每次都应该不同，这样游戏才能多次玩得有趣。我们将使用 1 到 100 之间的随机数，这样游戏不会太难。Rust 的标准库中还没有包含随机数功能。然而，Rust 团队确实提供了一个 [`rand` crate][randcrate]，它具有该功能。

### 使用 Crate 获取更多功能

记住，crate 是 Rust 源代码文件的集合。我们一直在构建的项目是一个 _二进制 crate_，它是一个可执行文件。`rand` crate 是一个 _库 crate_，它包含旨在用于其他程序的代码，不能单独执行。

Cargo 对外部 crate 的协调是 Cargo 真正闪耀的地方。在我们编写使用 `rand` 的代码之前，我们需要修改 _Cargo.toml_ 文件以将 `rand` crate 添加为依赖项。现在打开该文件，并在 Cargo 为你创建的 `[dependencies]` 部分标题下添加以下行。确保按照我们在这里的方式指定 `rand`，并带有此版本号，否则本教程中的代码示例可能无法工作：

<!-- 当更新 `rand` 使用的版本时，也更新这些文件中使用的 `rand` 版本，以便它们都匹配：
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">文件名: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

在 _Cargo.toml_ 文件中，标题后面的所有内容都是该部分的一部分，直到另一个部分开始。在 `[dependencies]` 中，你告诉 Cargo 你的项目依赖哪些外部 crate 以及你需要这些 crate 的哪些版本。在这种情况下，我们使用语义版本说明符 `0.8.5` 指定 `rand` crate。Cargo 理解[语义版本控制][semver]<!-- ignore -->（有时称为 _SemVer_），这是编写版本号的标准。说明符 `0.8.5` 实际上是 `^0.8.5` 的简写，这意味着任何至少为 0.8.5 但低于 0.9.0 的版本。

Cargo 认为这些版本具有与 0.8.5 版本兼容的公共 API，此规范确保你将获得最新的补丁版本，该版本仍将与本章中的代码一起编译。任何 0.9.0 或更高版本都不能保证具有与以下示例使用的 API 相同的 API。

现在，在不更改任何代码的情况下，让我们构建项目，如代码清单 2-2 所示。

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="添加 rand crate 作为依赖项后运行 `cargo build` 的输出">

```console
$ cargo build
  Updating crates.io index
   Locking 15 packages to latest Rust 1.85.0 compatible versions
    Adding rand v0.8.5 (available: v0.9.0)
 Compiling proc-macro2 v1.0.93
 Compiling unicode-ident v1.0.17
 Compiling libc v0.2.170
 Compiling cfg-if v1.0.0
 Compiling byteorder v1.5.0
 Compiling getrandom v0.2.15
 Compiling rand_core v0.6.4
 Compiling quote v1.0.38
 Compiling syn v2.0.98
 Compiling zerocopy-derive v0.7.35
 Compiling zerocopy v0.7.35
 Compiling ppv-lite86 v0.2.20
 Compiling rand_chacha v0.3.1
 Compiling rand v0.8.5
 Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
  Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.48s
```

</Listing>

你可能会看到不同的版本号（但它们都将与代码兼容，这要归功于 SemVer！）和不同的行（取决于操作系统），并且行的顺序可能不同。

当我们包含外部依赖项时，Cargo 会从 _注册表_ 中获取该依赖项所需的所有内容的最新版本，注册表是 [Crates.io][cratesio] 数据的副本。Crates.io 是 Rust 生态系统中的人们发布他们的开源 Rust 项目供他人使用的地方。

更新注册表后，Cargo 检查 `[dependencies]` 部分并下载列出的任何尚未下载的 crate。在这种情况下，尽管我们只列出了 `rand` 作为依赖项，但 Cargo 还抓取了 `rand` 依赖的其他 crate 以使其工作。下载 crate 后，Rust 编译它们，然后使用可用的依赖项编译项目。

如果你立即再次运行 `cargo build` 而不做任何更改，除了 `Finished` 行外，你将不会得到任何输出。Cargo 知道它已经下载并编译了依赖项，并且你没有在 _Cargo.toml_ 文件中更改它们的任何内容。Cargo 还知道你还没有更改代码的任何内容，所以它也不会重新编译代码。由于无事可做，它只是退出。

如果你打开 _src/main.rs_ 文件，做一个微小的更改，然后保存并再次构建，你将只看到两行输出：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

这些行显示 Cargo 仅更新了构建，因为你对 _src/main.rs_ 文件进行了微小的更改。你的依赖项没有更改，所以 Cargo 知道它可以重用已经下载和编译的内容。

#### 使用 _Cargo.lock_ 文件确保可重现的构建

Cargo 有一个机制，确保你或其他人每次构建代码时都可以重建相同的工件：Cargo 将只使用你指定的依赖项版本，直到你另有指示。例如，假设下周 `rand` crate 的 0.8.6 版本发布，该版本包含一个重要的错误修复，但它也包含一个会破坏你代码的回归。为了处理这个问题，Rust 在你第一次运行 `cargo build` 时创建了 _Cargo.lock_ 文件，所以我们现在在 _guessing_game_ 目录中有这个文件。

当你第一次构建项目时，Cargo 会找出所有符合标准的依赖项版本，然后将它们写入 _Cargo.lock_ 文件。当你将来构建项目时，Cargo 会看到 _Cargo.lock_ 文件存在，并将使用其中指定的版本，而不是再次进行所有版本确定的工作。这让你可以自动获得可重现的构建。换句话说，由于 _Cargo.lock_ 文件的存在，你的项目将保持在 0.8.5 版本，直到你明确升级。由于 _Cargo.lock_ 文件对于可重现的构建很重要，它通常与项目中的其余代码一起检入源代码控制。

#### 更新 Crate 以获取新版本

当你确实想要更新 crate 时，Cargo 提供了 `update` 命令，它将忽略 _Cargo.lock_ 文件并找出 _Cargo.toml_ 中符合你规范的所有最新版本。Cargo 然后将这些版本写入 _Cargo.lock_ 文件。在这种情况下，Cargo 只会寻找大于 0.8.5 且小于 0.9.0 的版本。如果 `rand` crate 发布了两个新版本 0.8.6 和 0.9.0，如果你运行 `cargo update`，你将看到以下内容：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
假设 rand 有一个新的 0.8.x 版本；否则使用另一个更新
作为创建此处显示的假设输出的指南 -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.9.0)
```

Cargo 忽略了 0.9.0 版本。此时，你还会注意到 _Cargo.lock_ 文件中的变化，指出你现在使用的 `rand` crate 版本是 0.8.6。要使用 `rand` 0.9.0 版本或 0.9._x_ 系列的任何版本，你必须将 _Cargo.toml_ 文件更新为如下所示：

```toml
[dependencies]
rand = "0.9.0"
```

下次你运行 `cargo build` 时，Cargo 将更新可用的 crate 注册表，并根据你指定的新版本重新评估你的 `rand` 要求。

关于 [Cargo][doccargo]<!-- ignore --> 和 [其生态系统][doccratesio]<!-- ignore --> 还有很多要说的，我们将在第 14 章讨论，但现在，这就是你需要知道的全部内容。Cargo 使得重用库变得非常容易，因此 Rustaceans 能够编写由多个包组装而成的小型项目。

### 生成随机数

让我们开始使用 `rand` 生成一个要猜测的数字。下一步是更新 _src/main.rs_，如代码清单 2-3 所示。

<Listing number="2-3" file-name="src/main.rs" caption="添加代码以生成随机数">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

首先我们添加了 `use rand::Rng;` 这一行。`Rng` trait 定义了随机数生成器实现的方法，这个 trait 必须在作用域内才能使用这些方法。第 10 章将详细介绍 trait。

接下来，我们在中间添加了两行。在第一行中，我们调用 `rand::thread_rng` 函数，它为我们提供了我们将要使用的特定随机数生成器：一个与当前执行线程本地相关并由操作系统播种的生成器。然后我们在随机数生成器上调用 `gen_range` 方法。这个方法由我们通过 `use rand::Rng;` 语句引入作用域的 `Rng` trait 定义。`gen_range` 方法接受一个范围表达式作为参数，并生成该范围内的随机数。我们在这里使用的范围表达式形式为 `start..=end`，并且在上下界都是包含的，所以我们需要指定 `1..=100` 来请求一个 1 到 100 之间的数字。

> 注意：你不会仅仅知道要使用哪些 trait 以及从 crate 中调用哪些方法和函数，所以每个 crate 都有使用说明的文档。Cargo 的另一个巧妙功能是运行 `cargo doc --open` 命令将在本地构建所有依赖项提供的文档，并在浏览器中打开它。如果你对 `rand` crate 中的其他功能感兴趣，例如，运行 `cargo doc --open` 并点击左侧边栏中的 `rand`。

第二行新代码打印了秘密数字。这在开发程序时很有用，以便能够测试它，但我们将在最终版本中删除它。如果程序一开始就打印答案，那就不太像游戏了！

尝试运行程序几次：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

你应该得到不同的随机数，并且它们都应该是 1 到 100 之间的数字。干得好！

## 比较猜测与秘密数字

现在我们有了用户输入和一个随机数，我们可以比较它们。这一步如代码清单 2-4 所示。请注意，这段代码还不能编译，我们将解释原因。

<Listing number="2-4" file-name="src/main.rs" caption="处理比较两个数字的可能返回值">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

首先我们添加了另一个 `use` 语句，从标准库中引入了一个名为 `std::cmp::Ordering` 的类型。`Ordering` 类型是另一个枚举，具有 `Less`、`Greater` 和 `Equal` 变体。这些是比较两个值时可能出现的三种结果。

然后我们在底部添加了五行新代码，使用 `Ordering` 类型。`cmp` 方法比较两个值，可以在任何可以比较的东西上调用。它接受一个你想要比较的东西的引用：在这里它比较 `guess` 和 `secret_number`。然后它返回我们通过 `use` 语句引入作用域的 `Ordering` 枚举的一个变体。我们使用 [`match`][match]<!-- ignore --> 表达式根据 `cmp` 调用返回的 `Ordering` 变体决定接下来要做什么。

`match` 表达式由 _分支_ 组成。一个分支由一个 _模式_ 和如果 `match` 给定的值符合该分支的模式时应运行的代码组成。Rust 获取 `match` 给定的值，并依次查看每个分支的模式。模式和 `match` 结构是 Rust 的强大功能：它们让你表达代码可能遇到的各种情况，并确保你处理所有这些情况。这些功能将在第 6 章和第 19 章中详细介绍。

让我们通过我们在这里使用的 `match` 表达式来走一个例子。假设用户猜了 50，而这次随机生成的秘密数字是 38。

当代码比较 50 和 38 时，`cmp` 方法将返回 `Ordering::Greater`，因为 50 大于 38。`match` 表达式获取 `Ordering::Greater` 值并开始检查每个分支的模式。它查看第一个分支的模式 `Ordering::Less`，发现值 `Ordering::Greater` 不匹配 `Ordering::Less`，所以它忽略该分支中的代码并移动到下一个分支。下一个分支的模式是 `Ordering::Greater`，它确实匹配 `Ordering::Greater`！该分支中的关联代码将执行并打印 `Too big!` 到屏幕。`match` 表达式在第一次成功匹配后结束，所以在这种情况下它不会查看最后一个分支。

然而，代码清单 2-4 中的代码还不能编译。让我们试试：

<!--
此输出中的错误编号应为 **不带** 锚点或片段注释的代码的错误编号
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

错误的核心是 _类型不匹配_。Rust 有一个强大的静态类型系统。然而，它也有类型推断。当我们写 `let mut guess = String::new()` 时，Rust 能够推断出 `guess` 应该是一个 `String`，并没有让我们写出类型。另一方面，`secret_number` 是一个数字类型。Rust 的几种数字类型可以具有 1 到 100 之间的值：`i32`，一个 32 位数字；`u32`，一个无符号的 32 位数字；`i64`，一个 64 位数字；以及其他。除非另有说明，Rust 默认为 `i32`，这是 `secret_number` 的类型，除非你在其他地方添加类型信息，导致 Rust 推断出不同的数字类型。错误的原因是 Rust 无法比较字符串和数字类型。

最终，我们希望将程序读取为输入的 `String` 转换为数字类型，以便我们可以将其与秘密数字进行数值比较。我们通过将这一行添加到 `main` 函数体中来做到这一点：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

这行代码是：

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

我们创建了一个名为 `guess` 的变量。但是等等，程序不是已经有一个名为 `guess` 的变量吗？确实有，但 Rust 允许我们用一个新的值来遮蔽前一个 `guess` 的值。_遮蔽_ 让我们可以重用 `guess` 变量名，而不是强迫我们创建两个唯一的变量，例如 `guess_str` 和 `guess`。我们将在[第 3 章][shadowing]<!-- ignore -->中更详细地讨论这一点，但现在，知道这个功能通常在你想要将一个值从一种类型转换为另一种类型时使用。

我们将这个新变量绑定到表达式 `guess.trim().parse()`。表达式中的 `guess` 指的是原始的 `guess` 变量，它包含作为字符串的输入。`String` 实例上的 `trim` 方法将消除开头和结尾的任何空白，这是我们在将字符串转换为 `u32` 之前必须做的，因为 `u32` 只能包含数字数据。用户必须按 <kbd>enter</kbd> 来满足 `read_line` 并输入他们的猜测，这会在字符串中添加一个换行符。例如，如果用户输入 <kbd>5</kbd> 并按 <kbd>enter</kbd>，`guess` 看起来像这样：`5\n`。`\n` 代表“换行”。（在 Windows 上，按 <kbd>enter</kbd> 会导致回车和换行，`\r\n`。）`trim` 方法消除 `\n` 或 `\r\n`，只留下 `5`。

字符串上的 [`parse` 方法][parse]<!-- ignore --> 将字符串转换为另一种类型。在这里，我们使用它将字符串转换为数字。我们需要通过使用 `let guess: u32` 告诉 Rust 我们想要的精确数字类型。`guess` 后面的冒号（`:`）告诉 Rust 我们将注释变量的类型。Rust 有一些内置的数字类型；这里看到的 `u32` 是一个无符号的 32 位整数。它是一个小正数的良好默认选择。你将在[第 3 章][integers]<!-- ignore -->中了解其他数字类型。

此外，此示例程序中的 `u32` 注释以及与 `secret_number` 的比较意味着 Rust 将推断 `secret_number` 也应该是一个 `u32`。所以现在比较将在两个相同类型的值之间进行！

`parse` 方法只能处理可以逻辑上转换为数字的字符，因此很容易导致错误。例如，如果字符串包含 `A👍%`，则无法将其转换为数字。因为它可能会失败，`parse` 方法返回一个 `Result` 类型，就像 `read_line` 方法一样（之前在[“使用 `Result` 处理潜在错误”](#handling-potential-failure-with-result)<!-- ignore-->中讨论过）。我们将以相同的方式处理这个 `Result`，再次使用 `expect` 方法。如果 `parse` 返回一个 `Err` `Result` 变体，因为它无法从字符串创建数字，`expect` 调用将使游戏崩溃并打印我们给它的消息。如果 `parse` 可以成功地将字符串转换为数字，它将返回 `Result` 的 `Ok` 变体，`expect` 将返回 `Ok` 值中我们想要的数字。

现在让我们运行程序：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

很好！尽管在猜测之前添加了空格，程序仍然识别出用户猜了 76。运行程序几次以验证不同输入的不同行为：正确猜出数字，猜一个太大的数字，猜一个太小的数字。

我们现在已经完成了游戏的大部分工作，但用户只能猜一次。让我们通过添加一个循环来改变这一点！

## 使用循环允许多次猜测

`loop` 关键字创建了一个无限循环。我们将添加一个循环，给用户更多机会猜测数字：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

如你所见，我们将从猜测输入提示开始的所有内容移入了一个循环。确保将循环内的每一行再缩进四个空格，然后再次运行程序。程序现在将永远要求另一个猜测，这实际上引入了一个新问题。用户似乎无法退出！

用户总是可以使用键盘快捷键 <kbd>ctrl</kbd>-<kbd>c</kbd> 中断程序。但还有另一种方法可以逃脱这个贪婪的怪物，正如在[“比较猜测与秘密数字”](#comparing-the-guess-to-the-secret-number)<!-- ignore -->中提到的 `parse` 讨论中提到的：如果用户输入一个非数字答案，程序将崩溃。我们可以利用这一点来允许用户退出，如下所示：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

输入 `quit` 将退出游戏，但你会注意到，输入任何其他非数字输入也会退出。这至少是不理想的；我们希望游戏在猜对数字时也停止。

### 在正确猜测后退出

让我们通过添加一个 `break` 语句来编程游戏在用户获胜时退出：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

在 `You win!` 之后添加 `break` 行，使程序在用户正确猜出秘密数字时退出循环。退出循环也意味着退出程序，因为循环是 `main` 的最后一部分。

### 处理无效输入

为了进一步完善游戏的行为，而不是在用户输入非数字时使程序崩溃，让我们让游戏忽略非数字，以便用户可以继续猜测。我们可以通过更改将 `guess` 从 `String` 转换为 `u32` 的行来实现这一点，如代码清单 2-5 所示。

<Listing number="2-5" file-name="src/main.rs" caption="忽略非数字猜测并请求另一个猜测，而不是使程序崩溃">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

我们从 `expect` 调用切换到 `match` 表达式，以从在错误时崩溃转变为处理错误。记住 `parse` 返回一个 `Result` 类型，而 `Result` 是一个枚举，包含 `Ok` 和 `Err` 两个变体。我们在这里使用 `match` 表达式，就像我们在 `cmp` 方法的 `Ordering` 结果中所做的那样。

如果 `parse` 能够成功地将字符串转换为数字，它将返回一个包含结果数字的 `Ok` 值。那个 `Ok` 值将与第一个 `match` 臂的模式匹配，并且 `match` 表达式将直接返回 `parse` 产生的并放在 `Ok` 值中的 `num` 值。这个数字将正好出现在我们正在创建的新 `guess` 变量中我们希望的位置。

如果 `parse` _不能_ 将字符串转换为数字，它将返回一个包含有关错误更多信息的 `Err` 值。`Err` 值不匹配第一个 `match` 臂中的 `Ok(num)` 模式，但它确实匹配第二个臂中的 `Err(_)` 模式。下划线 `_` 是一个通配符值；在这个例子中，我们说我们想要匹配所有 `Err` 值，不管它们内部包含什么信息。因此，程序将执行第二个臂的代码 `continue`，它告诉程序进入 `loop` 的下一次迭代并请求另一个猜测。因此，实际上，程序忽略了 `parse` 可能遇到的所有错误！

现在程序中的所有内容都应该按预期工作。让我们试试看：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

太棒了！只需进行一个微小的最终调整，我们就能完成猜数字游戏。回想一下，程序仍然在打印秘密数字。这对测试很有用，但会破坏游戏的乐趣。让我们删除输出秘密数字的 `println!`。清单 2-6 显示了最终代码。

<Listing number="2-6" file-name="src/main.rs" caption="Complete guessing game code">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

至此，你已经成功构建了猜数字游戏。恭喜你！

## 总结

这个项目是一个实践性的方式，向你介绍了许多新的 Rust 概念：`let`、`match`、函数、外部 crate 的使用等等。在接下来的几章中，你将更详细地学习这些概念。第 3 章涵盖了大多数编程语言都有的概念，如变量、数据类型和函数，并展示了如何在 Rust 中使用它们。第 4 章探讨了所有权，这是 Rust 区别于其他语言的一个特性。第 5 章讨论了结构体和方法语法，第 6 章解释了枚举的工作原理。

[prelude]: ../std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
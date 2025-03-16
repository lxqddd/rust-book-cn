## 泛型数据类型

我们使用泛型来为函数签名或结构体等项创建定义，然后可以将其用于许多不同的具体数据类型。首先，我们来看看如何使用泛型定义函数、结构体、枚举和方法。然后我们将讨论泛型如何影响代码性能。

### 在函数定义中

当定义一个使用泛型的函数时，我们将泛型放在函数的签名中，通常我们会在这里指定参数和返回值的数据类型。这样做使我们的代码更加灵活，并为函数的调用者提供更多功能，同时防止代码重复。

继续我们的 `largest` 函数，Listing 10-4 展示了两个函数，它们都在切片中查找最大值。然后我们将这些函数合并为一个使用泛型的单一函数。

<Listing number="10-4" file-name="src/main.rs" caption="两个函数，仅在名称和签名中的类型上有所不同">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

</Listing>

`largest_i32` 函数是我们在 Listing 10-3 中提取的函数，它在切片中查找最大的 `i32`。`largest_char` 函数在切片中查找最大的 `char`。函数体中的代码相同，因此我们通过在单一函数中引入泛型类型参数来消除重复。

为了在新函数中参数化类型，我们需要为类型参数命名，就像我们为函数的值参数命名一样。你可以使用任何标识符作为类型参数名称。但我们将使用 `T`，因为按照惯例，Rust 中的类型参数名称很短，通常只有一个字母，并且 Rust 的类型命名约定是驼峰式命名法。`T` 是 _type_ 的缩写，是大多数 Rust 程序员的默认选择。

当我们在函数体中使用参数时，我们必须在签名中声明参数名称，以便编译器知道该名称的含义。同样，当我们在函数签名中使用类型参数名称时，我们必须在使用之前声明类型参数名称。为了定义泛型 `largest` 函数，我们将类型名称声明放在尖括号 `<>` 中，放在函数名称和参数列表之间，如下所示：

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

我们将这个定义理解为：函数 `largest` 对某个类型 `T` 是泛型的。该函数有一个名为 `list` 的参数，它是类型 `T` 的值的切片。`largest` 函数将返回对相同类型 `T` 的值的引用。

Listing 10-5 展示了使用泛型数据类型在其签名中的组合 `largest` 函数定义。该列表还展示了如何使用 `i32` 值或 `char` 值的切片调用该函数。请注意，此代码尚无法编译，但我们将在本章稍后修复它。

<Listing number="10-5" file-name="src/main.rs" caption="使用泛型类型参数的 `largest` 函数；此代码尚无法编译">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

</Listing>

如果我们现在编译此代码，我们将得到以下错误：

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

帮助文本提到了 `std::cmp::PartialOrd`，这是一个 _trait_，我们将在下一节中讨论 trait。现在，只需知道此错误表明 `largest` 的函数体不适用于 `T` 可能的所有类型。因为我们希望在函数体中比较类型 `T` 的值，所以我们只能使用可以排序的类型。为了启用比较，标准库提供了 `std::cmp::PartialOrd` trait，你可以在类型上实现它（有关此 trait 的更多信息，请参见附录 C）。通过遵循帮助文本的建议，我们将 `T` 的有效类型限制为仅实现 `PartialOrd` 的类型，此示例将编译，因为标准库在 `i32` 和 `char` 上都实现了 `PartialOrd`。

### 在结构体定义中

我们还可以使用 `<>` 语法定义一个或多个字段使用泛型类型参数的结构体。Listing 10-6 定义了一个 `Point<T>` 结构体，用于保存任何类型的 `x` 和 `y` 坐标值。

<Listing number="10-6" file-name="src/main.rs" caption="一个 `Point<T>` 结构体，保存类型 `T` 的 `x` 和 `y` 值">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

</Listing>

在结构体定义中使用泛型的语法与函数定义中的语法类似。首先，我们在结构体名称后面的尖括号内声明类型参数的名称。然后我们在结构体定义中使用泛型类型，而不是指定具体的数据类型。

请注意，因为我们只使用了一个泛型类型来定义 `Point<T>`，所以这个定义表示 `Point<T>` 结构体对某个类型 `T` 是泛型的，并且字段 `x` 和 `y` 都是相同的类型，无论该类型是什么。如果我们创建一个 `Point<T>` 实例，其中包含不同类型的值，如 Listing 10-7 所示，我们的代码将无法编译。

<Listing number="10-7" file-name="src/main.rs" caption="字段 `x` 和 `y` 必须是相同类型，因为它们具有相同的泛型数据类型 `T`。">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

</Listing>

在这个例子中，当我们将整数值 `5` 赋给 `x` 时，我们让编译器知道泛型类型 `T` 对于这个 `Point<T>` 实例将是一个整数。然后当我们为 `y` 指定 `4.0` 时，我们将其定义为与 `x` 相同的类型，我们将得到一个类型不匹配的错误，如下所示：

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

为了定义一个 `Point` 结构体，其中 `x` 和 `y` 都是泛型但可以有不同的类型，我们可以使用多个泛型类型参数。例如，在 Listing 10-8 中，我们将 `Point` 的定义更改为对类型 `T` 和 `U` 是泛型的，其中 `x` 是类型 `T`，`y` 是类型 `U`。

<Listing number="10-8" file-name="src/main.rs" caption="一个 `Point<T, U>` 对两种类型是泛型的，因此 `x` 和 `y` 可以是不同类型的值">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

</Listing>

现在所有显示的 `Point` 实例都是允许的！你可以在定义中使用任意多个泛型类型参数，但使用过多会使代码难以阅读。如果你发现代码中需要大量泛型类型，这可能表明你的代码需要重构为更小的部分。

### 在枚举定义中

正如我们对结构体所做的那样，我们可以定义枚举以在其变体中保存泛型数据类型。让我们再看一下标准库提供的 `Option<T>` 枚举，我们在第 6 章中使用过它：

```rust
enum Option<T> {
    Some(T),
    None,
}
```

现在这个定义应该对你来说更有意义了。如你所见，`Option<T>` 枚举对类型 `T` 是泛型的，并且有两个变体：`Some`，它保存一个类型 `T` 的值，以及一个 `None` 变体，它不保存任何值。通过使用 `Option<T>` 枚举，我们可以表达可选值的抽象概念，并且因为 `Option<T>` 是泛型的，所以无论可选值的类型是什么，我们都可以使用这个抽象。

枚举也可以使用多个泛型类型。我们在第 9 章中使用的 `Result` 枚举的定义就是一个例子：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Result` 枚举对两种类型 `T` 和 `E` 是泛型的，并且有两个变体：`Ok`，它保存一个类型 `T` 的值，以及 `Err`，它保存一个类型 `E` 的值。这个定义使得在任何我们可能成功（返回某种类型 `T` 的值）或失败（返回某种类型 `E` 的错误）的操作中使用 `Result` 枚举变得方便。事实上，这就是我们在 Listing 9-3 中用来打开文件的内容，其中 `T` 在文件成功打开时被填充为类型 `std::fs::File`，而 `E` 在打开文件时遇到问题时被填充为类型 `std::io::Error`。

当你识别出代码中有多个结构体或枚举定义仅在它们保存的值的类型上有所不同时，你可以通过使用泛型类型来避免重复。

### 在方法定义中

我们可以在结构体和枚举上实现方法（正如我们在第 5 章中所做的那样），并在它们的定义中也使用泛型类型。Listing 10-9 展示了我们在 Listing 10-6 中定义的 `Point<T>` 结构体，并在其上实现了一个名为 `x` 的方法。

<Listing number="10-9" file-name="src/main.rs" caption="在 `Point<T>` 结构体上实现一个名为 `x` 的方法，该方法将返回对类型 `T` 的 `x` 字段的引用">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

</Listing>

在这里，我们在 `Point<T>` 上定义了一个名为 `x` 的方法，它返回对字段 `x` 中的数据的引用。

请注意，我们必须在 `impl` 之后声明 `T`，以便我们可以使用 `T` 来指定我们正在为类型 `Point<T>` 实现方法。通过在 `impl` 之后声明 `T` 为泛型类型，Rust 可以识别 `Point` 中尖括号内的类型是泛型类型而不是具体类型。我们可以为这个泛型参数选择一个与结构体定义中声明的泛型参数不同的名称，但使用相同的名称是惯例。如果你在 `impl` 中编写一个声明泛型类型的方法，那么无论最终替换泛型类型的具体类型是什么，该方法都将定义在该类型的任何实例上。

我们还可以在定义类型的方法时对泛型类型指定约束。例如，我们可以仅在 `Point<f32>` 实例上实现方法，而不是在任何泛型类型的 `Point<T>` 实例上实现方法。在 Listing 10-10 中，我们使用具体类型 `f32`，这意味着我们没有在 `impl` 之后声明任何类型。

<Listing number="10-10" file-name="src/main.rs" caption="一个 `impl` 块，仅适用于具有特定具体类型的泛型类型参数 `T` 的结构体">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

</Listing>

这段代码意味着类型 `Point<f32>` 将有一个 `distance_from_origin` 方法；其他 `Point<T>` 实例，其中 `T` 不是 `f32` 类型，将没有定义此方法。该方法测量我们的点距离坐标 (0.0, 0.0) 的点有多远，并使用仅适用于浮点类型的数学运算。

结构体定义中的泛型类型参数并不总是与你在同一结构体的方法签名中使用的泛型类型参数相同。Listing 10-11 使用泛型类型 `X1` 和 `Y1` 用于 `Point` 结构体，并使用 `X2` 和 `Y2` 用于 `mixup` 方法签名，以使示例更清晰。该方法创建一个新的 `Point` 实例，其中 `x` 值来自 `self` `Point`（类型为 `X1`），`y` 值来自传入的 `Point`（类型为 `Y2`）。

<Listing number="10-11" file-name="src/main.rs" caption="一个使用与其结构体定义不同的泛型类型的方法">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

</Listing>

在 `main` 中，我们定义了一个 `Point`，其中 `x` 是一个 `i32`（值为 `5`），`y` 是一个 `f64`（值为 `10.4`）。`p2` 变量是一个 `Point` 结构体，其中 `x` 是一个字符串切片（值为 `"Hello"`），`y` 是一个 `char`（值为 `c`）。在 `p1` 上调用 `mixup` 并传入 `p2` 会给我们 `p3`，它将有一个 `i32` 类型的 `x`，因为 `x` 来自 `p1`。`p3` 变量将有一个 `char` 类型的 `y`，因为 `y` 来自 `p2`。`println!` 宏调用将打印 `p3.x = 5, p3.y = c`。

这个例子的目的是展示一种情况，其中一些泛型参数在 `impl` 中声明，而另一些在方法定义中声明。在这里，泛型参数 `X1` 和 `Y1` 在 `impl` 之后声明，因为它们与结构体定义一起使用。泛型参数 `X2` 和 `Y2` 在 `fn mixup` 之后声明，因为它们仅与方法相关。

### 使用泛型的代码性能

你可能想知道使用泛型类型参数时是否存在运行时成本。好消息是，使用泛型类型不会使你的程序比使用具体类型时运行得更慢。

Rust 通过在编译时对使用泛型的代码进行单态化来实现这一点。_单态化_ 是通过填充编译时使用的具体类型将泛型代码转换为特定代码的过程。在这个过程中，编译器执行与我们创建泛型函数时相反的步骤：编译器查看所有调用泛型代码的地方，并为调用泛型代码的具体类型生成代码。

让我们通过使用标准库的泛型 `Option<T>` 枚举来看看这是如何工作的：

```rust
let integer = Some(5);
let float = Some(5.0);
```

当 Rust 编译此代码时，它会执行单态化。在这个过程中，编译器读取在 `Option<T>` 实例中使用的值，并识别出两种 `Option<T>`：一种是 `i32`，另一种是 `f64`。因此，它将 `Option<T>` 的泛型定义扩展为专门针对 `i32` 和 `f64` 的两个定义，从而用特定的定义替换泛型定义。

单态化后的代码看起来类似于以下内容（编译器使用与我们在此处使用的名称不同的名称以进行说明）：

<Listing file-name="src/main.rs">

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

</Listing>

泛型 `Option<T>` 被编译器创建的特定定义所替换。因为 Rust 将泛型代码编译为指定每个实例中类型的代码，所以我们使用泛型时不会产生运行时成本。当代码运行时，它的表现就像我们手动复制每个定义一样。单态化过程使 Rust 的泛型在运行时非常高效。
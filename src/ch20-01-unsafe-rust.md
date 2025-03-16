## 不安全的 Rust

到目前为止，我们讨论的所有代码都在编译时强制执行了 Rust 的内存安全保证。然而，Rust 内部还隐藏着另一种语言，它不强制执行这些内存安全保证：它被称为**不安全的 Rust**，其工作方式与常规 Rust 一样，但为我们提供了额外的超能力。

不安全的 Rust 存在的原因是，静态分析本质上是保守的。当编译器试图确定代码是否遵守这些保证时，拒绝一些有效的程序比接受一些无效的程序更好。尽管代码**可能**没问题，但如果 Rust 编译器没有足够的信息来确信，它就会拒绝该代码。在这种情况下，你可以使用不安全的代码来告诉编译器：“相信我，我知道我在做什么。”不过要注意，使用不安全的 Rust 是有风险的：如果你不正确使用不安全的代码，可能会因为内存不安全而出现问题，例如空指针解引用。

Rust 有不安全的另一面的另一个原因是，底层的计算机硬件本质上就是不安全的。如果 Rust 不允许你进行不安全的操作，你将无法完成某些任务。Rust 需要允许你进行低级系统编程，例如直接与操作系统交互，甚至编写自己的操作系统。进行低级系统编程是该语言的目标之一。让我们来探索一下我们可以用不安全的 Rust 做什么以及如何做。

### 不安全的超能力

要切换到不安全的 Rust，请使用 `unsafe` 关键字，然后开始一个新的代码块，其中包含不安全的代码。你可以在不安全的 Rust 中执行五个操作，这些操作在安全的 Rust 中是不允许的，我们称之为**不安全的超能力**。这些超能力包括：

- 解引用裸指针
- 调用不安全的函数或方法
- 访问或修改可变的静态变量
- 实现不安全的 trait
- 访问 `union` 的字段

重要的是要理解，`unsafe` 并不会关闭借用检查器或禁用 Rust 的其他安全检查：如果你在不安全的代码中使用引用，它仍然会被检查。`unsafe` 关键字只是让你能够访问这五个特性，然后编译器不会对这些特性进行内存安全检查。在不安全的代码块中，你仍然会获得一定程度的安全性。

此外，`unsafe` 并不意味着代码块中的代码必然危险或一定会出现内存安全问题：其意图是作为程序员，你将确保 `unsafe` 块中的代码以有效的方式访问内存。

人都会犯错，错误也会发生，但通过要求这五个不安全的操作必须在标记为 `unsafe` 的块中执行，你将知道任何与内存安全相关的错误都必须出现在 `unsafe` 块中。保持 `unsafe` 块尽可能小；当你调查内存错误时，你会感激这一点。

为了尽可能隔离不安全的代码，最好将这些代码封装在一个安全的抽象中，并提供一个安全的 API，我们将在本章后面讨论不安全的函数和方法时探讨这一点。标准库的部分内容是通过对经过审计的不安全代码进行安全抽象来实现的。将不安全的代码封装在安全抽象中可以防止 `unsafe` 的使用泄漏到所有你可能希望使用 `unsafe` 代码实现功能的地方，因为使用安全抽象是安全的。

让我们依次看看这五个不安全的超能力。我们还将探讨一些为不安全代码提供安全接口的抽象。

### 解引用裸指针

在第 4 章的[“悬垂引用”][dangling-references]<!-- ignore -->中，我们提到编译器确保引用始终有效。不安全的 Rust 有两个新类型，称为**裸指针**，它们类似于引用。与引用一样，裸指针可以是不可变的或可变的，分别写为 `*const T` 和 `*mut T`。星号不是解引用操作符；它是类型名称的一部分。在裸指针的上下文中，**不可变**意味着指针在解引用后不能直接赋值。

与引用和智能指针不同，裸指针：

- 允许忽略借用规则，可以同时拥有不可变和可变指针，或多个可变指针指向同一位置
- 不保证指向有效的内存
- 允许为空
- 不实现任何自动清理

通过选择不让 Rust 强制执行这些保证，你可以放弃有保证的安全性，以换取更高的性能或与 Rust 的保证不适用的其他语言或硬件进行交互的能力。

Listing 20-1 展示了如何创建一个不可变和一个可变的裸指针。

<Listing number="20-1" caption="使用裸借用操作符创建裸指针">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

注意，我们在这个代码中没有包含 `unsafe` 关键字。我们可以在安全代码中创建裸指针；只是不能在 `unsafe` 块之外解引用裸指针，稍后你会看到。

我们使用裸借用操作符创建了裸指针：`&raw const num` 创建了一个 `*const i32` 不可变裸指针，`&raw mut num` 创建了一个 `*mut i32` 可变裸指针。因为我们直接从局部变量创建它们，所以我们知道这些特定的裸指针是有效的，但我们不能对任何裸指针都做出这种假设。

为了演示这一点，接下来我们将创建一个裸指针，其有效性我们无法确定，使用 `as` 来转换值而不是使用裸借用操作符。Listing 20-2 展示了如何创建一个指向内存中任意位置的裸指针。尝试使用任意内存是未定义的：该地址可能有数据，也可能没有，编译器可能会优化代码，使其不访问内存，或者程序可能会因段错误而终止。通常，没有好的理由编写这样的代码，尤其是在可以使用裸借用操作符的情况下，但这是可能的。

<Listing number="20-2" caption="创建一个指向任意内存地址的裸指针">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

回想一下，我们可以在安全代码中创建裸指针，但我们不能**解引用**裸指针并读取指向的数据。在 Listing 20-3 中，我们在需要 `unsafe` 块的裸指针上使用解引用操作符 `*`。

<Listing number="20-3" caption="在 `unsafe` 块中解引用裸指针">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

创建指针不会造成任何伤害；只有当我们尝试访问它指向的值时，我们才可能会处理一个无效的值。

还要注意，在 Listing 20-1 和 20-3 中，我们创建了 `*const i32` 和 `*mut i32` 裸指针，它们都指向存储 `num` 的同一内存位置。如果我们尝试创建对 `num` 的不可变和可变引用，代码将无法编译，因为 Rust 的所有权规则不允许在存在不可变引用的同时存在可变引用。使用裸指针，我们可以创建一个可变指针和一个不可变指针指向同一位置，并通过可变指针更改数据，这可能会导致数据竞争。要小心！

既然有这么多危险，为什么还要使用裸指针呢？一个主要的用例是与 C 代码交互，你将在下一节[“调用不安全的函数或方法”](#calling-an-unsafe-function-or-method)<!-- ignore -->中看到。另一个用例是构建借用检查器无法理解的安全抽象。我们将介绍不安全的函数，然后看一个使用不安全代码的安全抽象示例。

### 调用不安全的函数或方法

你可以在不安全的块中执行的第二种操作是调用不安全的函数。不安全的函数和方法看起来与常规函数和方法完全一样，但它们在定义前多了一个 `unsafe`。在这个上下文中，`unsafe` 关键字表示该函数有一些我们在调用时需要遵守的要求，因为 Rust 无法保证我们已经满足这些要求。通过在 `unsafe` 块中调用不安全的函数，我们表示我们已经阅读了该函数的文档，并承担了遵守函数契约的责任。

这里有一个名为 `dangerous` 的不安全函数，它的函数体中没有做任何事情：

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

我们必须在单独的 `unsafe` 块中调用 `dangerous` 函数。如果我们尝试在没有 `unsafe` 块的情况下调用 `dangerous`，我们会得到一个错误：

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

使用 `unsafe` 块，我们向 Rust 断言我们已经阅读了函数的文档，理解了如何正确使用它，并验证了我们正在履行函数的契约。

要在不安全函数的函数体中执行不安全的操作，你仍然需要使用 `unsafe` 块，就像在常规函数中一样，如果你忘记了，编译器会警告你。这有助于保持 `unsafe` 块尽可能小，因为可能不需要在整个函数体中都使用不安全的操作。

#### 在不安全代码上创建安全抽象

仅仅因为一个函数包含不安全的代码并不意味着我们需要将整个函数标记为不安全的。事实上，将不安全的代码封装在安全函数中是一种常见的抽象。作为一个例子，让我们研究一下标准库中的 `split_at_mut` 函数，它需要一些不安全的代码。我们将探讨如何实现它。这个安全方法定义在可变切片上：它接受一个切片，并通过在给定的索引处拆分切片将其分成两个。Listing 20-4 展示了如何使用 `split_at_mut`。

<Listing number="20-4" caption="使用安全的 `split_at_mut` 函数">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

我们不能仅使用安全的 Rust 实现这个函数。尝试可能看起来像 Listing 20-5，但它不会编译。为了简单起见，我们将 `split_at_mut` 实现为一个函数而不是方法，并且只针对 `i32` 值的切片而不是泛型类型 `T`。

<Listing number="20-5" caption="尝试仅使用安全的 Rust 实现 `split_at_mut`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

这个函数首先获取切片的长度。然后它通过检查给定的索引是否小于或等于长度来断言该索引在切片内。这个断言意味着如果我们传递一个大于长度的索引来拆分切片，函数将在尝试使用该索引之前 panic。

然后我们返回一个元组中的两个可变切片：一个从原始切片的开头到 `mid` 索引，另一个从 `mid` 到切片的末尾。

当我们尝试编译 Listing 20-5 中的代码时，我们会得到一个错误。

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

Rust 的借用检查器无法理解我们正在借用切片的不同部分；它只知道我们正在从同一个切片中借用两次。借用切片的不同部分在本质上是没问题的，因为这两个切片不重叠，但 Rust 不够聪明，无法理解这一点。当我们知道代码没问题，但 Rust 不知道时，就该使用不安全的代码了。

Listing 20-6 展示了如何使用 `unsafe` 块、裸指针和一些对不安全函数的调用来使 `split_at_mut` 的实现工作。

<Listing number="20-6" caption="在 `split_at_mut` 函数的实现中使用不安全代码">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

回想一下第 4 章中的[“切片类型”][the-slice-type]<!-- ignore -->，切片是指向某些数据的指针和切片的长度。我们使用 `len` 方法获取切片的长度，使用 `as_mut_ptr` 方法访问切片的裸指针。在这种情况下，因为我们有一个 `i32` 值的可变切片，`as_mut_ptr` 返回一个类型为 `*mut i32` 的裸指针，我们将其存储在变量 `ptr` 中。

我们保留了 `mid` 索引在切片内的断言。然后我们进入不安全的代码：`slice::from_raw_parts_mut` 函数接受一个裸指针和一个长度，并创建一个切片。我们使用它来创建一个从 `ptr` 开始且长度为 `mid` 的切片。然后我们在 `ptr` 上调用 `add` 方法，参数为 `mid`，以获取一个从 `mid` 开始的裸指针，并使用该指针和 `mid` 之后的剩余项数作为长度创建一个切片。

`slice::from_raw_parts_mut` 函数是不安全的，因为它接受一个裸指针，并且必须相信这个指针是有效的。裸指针上的 `add` 方法也是不安全的，因为它必须相信偏移位置也是一个有效的指针。因此，我们必须在调用 `slice::from_raw_parts_mut` 和 `add` 时使用 `unsafe` 块。通过查看代码并添加 `mid` 必须小于或等于 `len` 的断言，我们可以确定 `unsafe` 块中使用的所有裸指针都是指向切片内数据的有效指针。这是一个可接受且适当的使用 `unsafe` 的方式。

注意，我们不需要将生成的 `split_at_mut` 函数标记为 `unsafe`，我们可以从安全的 Rust 中调用这个函数。我们已经创建了一个安全抽象，通过使用不安全代码的函数实现，以安全的方式使用不安全代码，因为它只从该函数可以访问的数据中创建有效的指针。

相比之下，Listing 20-7 中使用 `slice::from_raw_parts_mut` 的代码在使用切片时可能会崩溃。这段代码接受一个任意的内存位置并创建一个长度为 10,000 的切片。

<Listing number="20-7" caption="从任意内存位置创建切片">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

我们不拥有这个任意位置的内存，也没有保证这段代码创建的切片包含有效的 `i32` 值。尝试将 `values` 当作一个有效的切片使用会导致未定义行为。

#### 使用 `extern` 函数调用外部代码

有时，你的 Rust 代码可能需要与用另一种语言编写的代码进行交互。为此，Rust 提供了 `extern` 关键字，用于创建和使用**外部函数接口（FFI）**。FFI 是一种编程语言定义函数并允许另一种（外部的）编程语言调用这些函数的方式。

Listing 20-8 演示了如何设置与 C 标准库中的 `abs` 函数的集成。在 `extern` 块中声明的函数通常从 Rust 代码中调用是不安全的，因此 `extern` 块也必须标记为 `unsafe`。原因是其他语言不强制执行 Rust 的规则和保证，Rust 也无法检查它们，因此程序员有责任确保安全。

<Listing number="20-8" file-name="src/main.rs" caption="声明并调用另一种语言中定义的 `extern` 函数">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

在 `unsafe extern "C"` 块中，我们列出了我们想要调用的另一种语言的外部函数的名称和签名。`"C"` 部分定义了外部函数使用的**应用程序二进制接口（ABI）**：ABI 定义了如何在汇编级别调用函数。`"C"` ABI 是最常见的，遵循 C 编程语言的 ABI。

在 `unsafe extern` 块中声明的每个项都是隐式 `unsafe` 的。然而，一些 FFI 函数**是**安全调用的。例如，C 标准库中的 `abs` 函数没有任何内存安全考虑，我们知道它可以与任何 `i32` 一起调用。在这种情况下，我们可以使用 `safe` 关键字来表示这个特定的函数是安全调用的，即使它在 `unsafe extern` 块中。一旦我们做出这个更改，调用它就不再需要 `unsafe` 块，如 Listing 20-9 所示。

<Listing number="20-9" file-name="src/main.rs" caption="在 `unsafe extern` 块中显式标记函数为 `safe` 并安全调用">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

将函数标记为 `safe` 并不会使其变得安全！相反，这是你向 Rust 做出的承诺，即它是安全的。你仍然有责任确保这个承诺得到遵守！

> #### 从其他语言调用 Rust 函数
>
> 我们也可以使用 `extern` 创建一个接口，允许其他语言调用 Rust 函数。我们不需要创建整个 `extern` 块，而是在相关函数的 `fn` 关键字前添加 `extern` 关键字并指定要使用的 ABI。我们还需要添加一个 `#[unsafe(no_mangle)]` 注解，告诉 Rust 编译器不要对这个函数的名称进行混淆。**混淆**是编译器将我们给函数的名称更改为包含更多信息的名称，以供编译过程的其他部分使用，但人类可读性较差。每种编程语言编译器对名称的混淆方式略有不同，因此为了让 Rust 函数能够被其他语言调用，我们必须禁用 Rust 编译器的名称混淆。这是不安全的，因为如果没有内置的混淆，库之间可能会发生名称冲突，因此我们有责任确保我们选择的名称在不混淆的情况下是安全导出的。
>
> 在以下示例中，我们使 `call_from_c` 函数可以从 C 代码中访问，在将其编译为共享库并从 C 中链接后：
>
> ```rust
> #[unsafe(no_mangle)]
> pub extern "C" fn call_from_c() {
>     println!("Just called a Rust function from C!");
> }
> ```
>
> 这种 `extern` 的使用仅在属性中需要 `unsafe`，而不是在 `extern` 块中。

### 访问或修改可变的静态变量

在本书中，我们还没有讨论过全局变量，Rust 确实支持全局变量，但它们可能会与 Rust 的所有权规则产生问题。如果两个线程访问同一个可变的全局变量，可能会导致数据竞争。

在 Rust 中，全局变量称为**静态**变量。Listing 20-10 展示了一个以字符串切片为值的静态变量的声明和使用示例。

<Listing number="20-10" file-name="src/main.rs" caption="定义和使用不可变的静态变量">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

静态变量类似于我们在第 3 章的[“常量”][differences-between-variables-and-constants]<!-- ignore -->中讨论的常量。静态变量的名称通常使用 `SCREAMING_SNAKE_CASE` 命名约定。静态变量只能存储具有 `'static` 生命周期的引用，这意味着 Rust 编译器可以推断出生命周期，我们不需要显式注解它。访问不可变的静态变量是安全的。

不可变静态变量和常量之间的一个微妙区别是，静态变量的值在内存中有一个固定的地址。使用该值将始终访问相同的数据。另一方面，常量允许在每次使用时复制其数据。另一个区别是静态变量可以是可变的。访问和修改可变的静态变量是**不安全的**。Listing 20-11 展示了如何声明、访问和修改一个名为 `COUNTER` 的可变静态变量。

<Listing number="20-11" file-name="src/main.rs" caption="读取或写入可变的静态变量是不安全的">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

与常规变量一样，我们使用 `mut` 关键字指定可变性。任何读取或写入 `COUNTER` 的代码都必须在 `unsafe` 块中。这段代码可以编译并打印 `COUNTER: 3`，正如我们所期望的那样，因为它是单线程的。如果有多个线程访问 `COUNTER`，可能会导致数据竞争，因此这是未定义行为。因此，我们需要将整个函数标记为 `unsafe`，并记录安全限制，以便任何调用该函数的人都知道他们可以安全地做什么和不做什么。

每当我们编写一个不安全的函数时，习惯上写一个以 `SAFETY` 开头的注释，解释调用者需要做什么才能安全地调用该函数。同样，每当我们执行不安全的操作时，习惯上写一个以 `SAFETY` 开头的注释，解释如何遵守安全规则。

此外，编译器不允许你创建对可变静态变量的引用。你只能通过使用裸借用操作符创建的裸指针来访问它。这包括在引用被隐式创建的情况下，例如在此代码列表中的 `println!` 中使用时。要求对静态可变变量的引用只能通过裸指针创建，这有助于使使用它们的安全要求更加明显。

对于全局可访问的可变数据，很难确保没有数据竞争，这就是为什么 Rust 认为可变的静态变量是不安全的。在可能的情况下，最好使用我们在第 16 章中讨论的并发技术和线程安全的智能指针，以便编译器检查来自不同线程的数据访问是否安全。

### 实现不安全的 Trait

我们可以使用 `unsafe` 来实现一个不安全的 trait。当 trait 的至少一个方法有一些编译器无法验证的不变量时，该 trait 是不安全的。我们通过在 `trait` 前添加 `unsafe` 关键字并将 trait 的实现也标记为 `unsafe` 来声明一个 trait 是不安全的，如 Listing 20-12 所示。

<Listing number="20-12" caption="定义和实现一个不安全的 trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

通过使用 `unsafe impl`，我们承诺我们将遵守编译器无法验证的不变量。

作为一个例子，回想一下我们在第 16 章的[“使用 `Sync` 和 `Send` trait 扩展并发”][extensible-concurrency-with-the-sync-and-send-traits]<!-- ignore -->中讨论的 `Sync` 和 `Send` 标记 trait：如果我们的类型完全由其他实现了 `Send` 和 `Sync` 的类型组成，编译器会自动实现这些 trait。如果我们实现一个包含未实现 `Send` 或 `Sync` 的类型（例如裸指针）的类型，并且我们希望将该类型标记为 `Send` 或 `Sync`，我们必须使用 `unsafe`。Rust 无法验证我们的类型是否遵守可以安全地跨线程发送或从多个线程访问的保证；因此，我们需要手动进行这些检查，并使用 `unsafe` 表示。

### 访问 `union` 的字段

只能在 `unsafe` 中执行的最后一个操作是访问 `union` 的字段。`union` 类似于 `struct`，但在特定实例中只使用一个声明的字段。`union` 主要用于与 C 代码中的 `union` 进行交互。访问 `union` 字段是不安全的，因为 Rust 无法保证当前存储在 `union` 实例中的数据类型。你可以在 [Rust 参考][reference] 中了解更多关于 `union` 的信息。

### 使用 Miri 检查不安全代码

在编写不安全代码时，你可能希望检查你编写的代码是否实际上是安全和正确的。最好的方法之一是使用 Miri，这是一个官方的 Rust 工具，用于检测未定义行为。虽然借用检查器是一个在编译时工作的**静态**工具，但 Miri 是一个在运行时工作的**动态**工具。它通过运行你的程序或其测试套件来检查你的代码，并在你违反它理解的 Rust 工作规则时检测到。

使用 Miri 需要 Rust 的 nightly 版本（我们在[附录 G：Rust 的构建和“Nightly Rust”][nightly]中讨论更多）。你可以通过输入 `rustup +nightly component add miri` 来安装 Rust 的 nightly 版本和 Miri 工具。这不会改变你的项目使用的 Rust 版本；它只是将该工具添加到你的系统中，以便你可以在需要时使用它。你可以通过输入 `cargo +nightly miri run` 或 `cargo +nightly miri test` 在项目上运行 Miri。

为了了解这有多有帮助，考虑当我们对 Listing 20-11 运行它时会发生什么。

```console
{{#include ../listings/ch20-advanced-features/listing-20-11/output.txt}}
```

Miri 正确地警告我们，我们有对可变数据的共享引用。在这里，Miri 只发出警告，因为在这种情况下这并不保证是未定义行为，它也没有告诉我们如何修复问题。但至少我们知道存在未定义行为的风险，并可以思考如何使代码安全。在某些情况下，Miri 还可以检测到明显的错误——**肯定**错误的代码模式——并提出如何修复这些错误的建议。

Miri 并不能捕捉到你在编写不安全代码时可能犯的所有错误。Miri 是一个动态分析工具，因此它只能捕捉实际运行的代码中的问题。这意味着你需要将其与良好的测试技术结合使用，以增加对你编写的不安全代码的信心。Miri 也不涵盖你的代码可能不健全的每一种方式。

换句话说：如果 Miri **确实**捕捉到了问题，你知道有一个 bug，但仅仅因为 Miri **没有**捕捉到 bug 并不意味着没有问题。它可以捕捉到很多问题。尝试在本章的其他不安全代码示例上运行它，看看它会说什么！

你可以在 [Miri 的 GitHub 仓库][miri] 中了解更多关于 Miri 的信息。

### 何时使用不安全代码

使用 `unsafe` 来使用刚刚讨论的五个超能力并不是错误的，甚至不会受到批评，但正确使用 `unsafe` 代码更棘手，因为编译器无法帮助维护内存安全。当你有理由使用 `unsafe` 代码时，你可以这样做，并且显式的 `unsafe` 注释使得在问题发生时更容易追踪问题的根源。每当你编写不安全代码时，你可以使用 Miri 来帮助你更有信心地认为你编写的代码遵守了 Rust 的规则。

要更深入地探索如何有效地使用不安全的 Rust，请阅读 Rust 的官方指南 [Rustonomicon][nomicon]。

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references
[differences-between-variables-and-constants]: ch03-01-variables-and-mutability.html#constants
[extensible-concurrency-with-the-sync-and-send-traits]: ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits
[the-slice-type]: ch04-03-slices.html#the-slice-type
[reference]: ../reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/
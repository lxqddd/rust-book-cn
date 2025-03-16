## 高级函数与闭包

本节探讨了一些与函数和闭包相关的高级特性，包括函数指针和返回闭包。

### 函数指针

我们已经讨论过如何将闭包传递给函数；你也可以将常规函数传递给函数！当你想要传递一个已经定义的函数而不是定义一个新的闭包时，这种技术非常有用。函数会被强制转换为 `fn` 类型（小写的 _f_），不要与 `Fn` 闭包特征混淆。`fn` 类型被称为**函数指针**。通过函数指针传递函数允许你将函数作为参数传递给其他函数。

指定参数为函数指针的语法与闭包的语法类似，如 Listing 20-28 所示，我们定义了一个函数 `add_one`，它将参数加 1。函数 `do_twice` 接受两个参数：一个是指向任何接受 `i32` 参数并返回 `i32` 的函数的函数指针，另一个是 `i32` 值。`do_twice` 函数调用函数 `f` 两次，传递 `arg` 值，然后将两次函数调用的结果相加。`main` 函数使用 `add_one` 和 `5` 作为参数调用 `do_twice`。

<Listing number="20-28" file-name="src/main.rs" caption="使用 `fn` 类型接受函数指针作为参数">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-28/src/main.rs}}
```

</Listing>

这段代码会打印出 `The answer is: 12`。我们指定 `do_twice` 中的参数 `f` 是一个接受一个 `i32` 类型参数并返回 `i32` 的 `fn`。然后我们可以在 `do_twice` 的函数体中调用 `f`。在 `main` 中，我们可以将函数名 `add_one` 作为第一个参数传递给 `do_twice`。

与闭包不同，`fn` 是一个类型而不是一个特征，因此我们直接指定 `fn` 作为参数类型，而不是声明一个泛型类型参数并使用 `Fn` 特征作为特征约束。

函数指针实现了所有三个闭包特征（`Fn`、`FnMut` 和 `FnOnce`），这意味着你总是可以将函数指针作为参数传递给期望闭包的函数。最好使用泛型类型和闭包特征之一来编写函数，这样你的函数可以接受函数或闭包。

话虽如此，当你只想接受 `fn` 而不接受闭包时，一个例子是与没有闭包的外部代码交互：C 函数可以接受函数作为参数，但 C 没有闭包。

作为一个可以使用内联定义的闭包或命名函数的例子，让我们看看标准库中 `Iterator` 特征提供的 `map` 方法的使用。为了使用 `map` 方法将数字向量转换为字符串向量，我们可以使用闭包，如 Listing 20-29 所示。

<Listing number="20-29" caption="使用闭包与 `map` 方法将数字转换为字符串">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-29/src/main.rs:here}}
```

</Listing>

或者我们可以将函数名作为 `map` 的参数，而不是闭包。Listing 20-30 展示了这种情况。

<Listing number="20-30" caption="使用 `String::to_string` 方法将数字转换为字符串">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-30/src/main.rs:here}}
```

</Listing>

注意，我们必须使用我们在 [“高级特征”][advanced-traits] 中讨论的完全限定语法，因为有多个名为 `to_string` 的函数可用。

在这里，我们使用了 `ToString` 特征中定义的 `to_string` 函数，标准库为任何实现了 `Display` 的类型实现了这个特征。

回想一下第 6 章中的 [“枚举值”][enum-values]，我们定义的每个枚举变体的名称也会成为一个初始化函数。我们可以将这些初始化函数作为实现闭包特征的函数指针，这意味着我们可以将初始化函数指定为接受闭包的方法的参数，如 Listing 20-31 所示。

<Listing number="20-31" caption="使用枚举初始化函数与 `map` 方法从数字创建 `Status` 实例">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-31/src/main.rs:here}}
```

</Listing>

在这里，我们通过使用 `Status::Value` 的初始化函数，在 `map` 调用的范围内为每个 `u32` 值创建 `Status::Value` 实例。有些人喜欢这种风格，有些人喜欢使用闭包。它们编译为相同的代码，所以使用你觉得更清晰的风格。

### 返回闭包

闭包由特征表示，这意味着你不能直接返回闭包。在大多数情况下，你可能想要返回一个特征，你可以使用实现该特征的具体类型作为函数的返回值。然而，对于闭包，你通常不能这样做，因为它们没有可返回的具体类型。例如，如果闭包从其作用域中捕获了任何值，你就不允许使用函数指针 `fn` 作为返回类型。

相反，你通常会使用我们在第 10 章中学到的 `impl Trait` 语法。你可以返回任何函数类型，使用 `Fn`、`FnOnce` 和 `FnMut`。例如，Listing 20-32 中的代码将正常工作。

<Listing number="20-32" caption="使用 `impl Trait` 语法从函数返回闭包">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-32/src/lib.rs}}
```

</Listing>

然而，正如我们在第 13 章的 [“闭包类型推断和注解”][closure-types] 中指出的那样，每个闭包也是其自己独特的类型。如果你需要处理具有相同签名但不同实现的多个函数，你将需要为它们使用特征对象。考虑如果你编写如 Listing 20-33 所示的代码会发生什么。

<Listing file-name="src/main.rs" number="20-33" caption="创建一个由返回 `impl Fn` 的函数定义的闭包的 `Vec<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-33/src/main.rs}}
```

</Listing>

这里我们有两个函数，`returns_closure` 和 `returns_initialized_closure`，它们都返回 `impl Fn(i32) -> i32`。注意，它们返回的闭包是不同的，尽管它们实现了相同的类型。如果我们尝试编译这个代码，Rust 会告诉我们它不会工作：

```text
{{#include ../listings/ch20-advanced-features/listing-20-33/output.txt}}
```

错误信息告诉我们，每当我们返回一个 `impl Trait` 时，Rust 会创建一个独特的**不透明类型**，这是一种我们无法看到 Rust 为我们构建的细节的类型。因此，即使这些函数都返回实现了相同特征 `Fn(i32) -> i32` 的闭包，Rust 为每个生成的类型是不同的。（这与 Rust 为不同的异步块生成不同的具体类型类似，即使它们具有相同的输出类型，正如我们在第 17 章的 [“处理任意数量的 Futures”][any-number-of-futures] 中看到的那样。我们已经多次看到这个问题的解决方案：我们可以使用特征对象，如 Listing 20-34 所示。

<Listing number="20-34" caption="创建一个由返回 `Box<dyn Fn>` 的函数定义的闭包的 `Vec<T>`，以便它们具有相同的类型">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-34/src/main.rs:here}}
```

</Listing>

这段代码将正常编译。有关特征对象的更多信息，请参阅第 18 章的 [“使用允许不同类型值的特征对象”][using-trait-objects-that-allow-for-values-of-different-types] 部分。

接下来，让我们看看宏！

[advanced-traits]: ch20-02-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[closure-types]: ch13-01-closures.html#closure-type-inference-and-annotation
[any-number-of-futures]: ch17-03-more-futures.html
[using-trait-objects-that-allow-for-values-of-different-types]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
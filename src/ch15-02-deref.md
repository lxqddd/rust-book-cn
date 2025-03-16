## 使用 `Deref` 将智能指针当作常规引用处理

<!-- 旧链接，不要删除 -->

<a id="treating-smart-pointers-like-regular-references-with-the-deref-trait"></a>

实现 `Deref` 特性允许你自定义 _解引用运算符_ `*` 的行为（不要与乘法或通配符运算符混淆）。通过以某种方式实现 `Deref`，使得智能指针可以像常规引用一样被处理，你可以编写操作引用的代码，并将这些代码用于智能指针。

让我们首先看看解引用运算符如何与常规引用一起工作。然后我们将尝试定义一个行为类似于 `Box<T>` 的自定义类型，并看看为什么解引用运算符在我们新定义的类型上不能像引用一样工作。我们将探索如何通过实现 `Deref` 特性使智能指针能够以类似于引用的方式工作。然后我们将看看 Rust 的 _解引用强制转换_ 功能，以及它如何让我们能够同时使用引用或智能指针。

> 注意：我们将要构建的 `MyBox<T>` 类型与真正的 `Box<T>` 有一个很大的区别：我们的版本不会将其数据存储在堆上。我们将这个示例的重点放在 `Deref` 上，因此数据实际存储的位置不如指针行为重要。

<!-- 旧链接，不要删除 -->

<a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>

### 通过解引用运算符追踪指针到值

常规引用是一种指针，可以将指针视为指向存储在其他地方的值的箭头。在 Listing 15-6 中，我们创建了一个指向 `i32` 值的引用，然后使用解引用运算符来追踪引用到值。

<Listing number="15-6" file-name="src/main.rs" caption="使用解引用运算符追踪指向 `i32` 值的引用">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

</Listing>

变量 `x` 持有一个 `i32` 值 `5`。我们将 `y` 设置为指向 `x` 的引用。我们可以断言 `x` 等于 `5`。然而，如果我们想对 `y` 中的值进行断言，我们必须使用 `*y` 来追踪引用到它指向的值（因此称为 _解引用_），以便编译器可以比较实际值。一旦我们解引用 `y`，我们就可以访问 `y` 指向的整数值，并将其与 `5` 进行比较。

如果我们尝试编写 `assert_eq!(5, y);`，我们会得到以下编译错误：

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

比较一个数字和一个数字的引用是不允许的，因为它们是不同的类型。我们必须使用解引用运算符来追踪引用到它指向的值。

### 像引用一样使用 `Box<T>`

我们可以重写 Listing 15-6 中的代码，使用 `Box<T>` 而不是引用；Listing 15-7 中在 `Box<T>` 上使用的解引用运算符与 Listing 15-6 中在引用上使用的解引用运算符功能相同：

<Listing number="15-7" file-name="src/main.rs" caption="在 `Box<i32>` 上使用解引用运算符">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

</Listing>

Listing 15-7 和 Listing 15-6 之间的主要区别在于，在这里我们将 `y` 设置为指向 `x` 的复制值的 box 实例，而不是指向 `x` 值的引用。在最后的断言中，我们可以使用解引用运算符来追踪 box 的指针，就像 `y` 是引用时一样。接下来，我们将通过定义我们自己的类型来探索 `Box<T>` 的特殊之处，它使我们能够使用解引用运算符。

### 定义我们自己的智能指针

让我们构建一个类似于标准库提供的 `Box<T>` 类型的智能指针，以体验智能指针默认情况下与引用的不同行为。然后我们将看看如何添加使用解引用运算符的能力。

`Box<T>` 类型最终被定义为一个只有一个元素的元组结构体，因此 Listing 15-8 以相同的方式定义了一个 `MyBox<T>` 类型。我们还将定义一个 `new` 函数，以匹配 `Box<T>` 上定义的 `new` 函数。

<Listing number="15-8" file-name="src/main.rs" caption="定义一个 `MyBox<T>` 类型">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

</Listing>

我们定义了一个名为 `MyBox` 的结构体，并声明了一个泛型参数 `T`，因为我们希望我们的类型能够持有任何类型的值。`MyBox` 类型是一个只有一个类型为 `T` 的元素的元组结构体。`MyBox::new` 函数接受一个类型为 `T` 的参数，并返回一个持有传入值的 `MyBox` 实例。

让我们尝试将 Listing 15-7 中的 `main` 函数添加到 Listing 15-8 中，并将其更改为使用我们定义的 `MyBox<T>` 类型，而不是 `Box<T>`。Listing 15-9 中的代码将无法编译，因为 Rust 不知道如何解引用 `MyBox`。

<Listing number="15-9" file-name="src/main.rs" caption="尝试以我们使用引用和 `Box<T>` 的方式使用 `MyBox<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

</Listing>

以下是编译错误的结果：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

我们的 `MyBox<T>` 类型不能被解引用，因为我们还没有在我们的类型上实现这种能力。为了能够使用 `*` 运算符进行解引用，我们实现了 `Deref` 特性。

<!-- 旧链接，不要删除 -->

<a id="treating-a-type-like-a-reference-by-implementing-the-deref-trait"></a>

### 实现 `Deref` 特性

正如在 [“在类型上实现特性”][impl-trait]<!-- ignore --> 中讨论的那样，要实现一个特性，我们需要为特性的必需方法提供实现。标准库提供的 `Deref` 特性要求我们实现一个名为 `deref` 的方法，该方法借用 `self` 并返回对内部数据的引用。Listing 15-10 包含一个 `Deref` 的实现，将其添加到 `MyBox<T>` 的定义中。

<Listing number="15-10" file-name="src/main.rs" caption="在 `MyBox<T>` 上实现 `Deref`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

</Listing>

`type Target = T;` 语法为 `Deref` 特性定义了一个关联类型。关联类型是一种稍微不同的声明泛型参数的方式，但你现在不需要担心它们；我们将在第 20 章中更详细地介绍它们。

我们将 `deref` 方法的主体填充为 `&self.0`，以便 `deref` 返回我们想要使用 `*` 运算符访问的值的引用；回想一下 [“使用没有命名字段的元组结构体创建不同类型”][tuple-structs]<!-- ignore --> 中 `.0` 访问元组结构体中的第一个值。Listing 15-9 中调用 `*` 的 `main` 函数现在可以编译，并且断言通过！

没有 `Deref` 特性，编译器只能解引用 `&` 引用。`deref` 方法使编译器能够获取任何实现 `Deref` 的类型的值，并调用 `deref` 方法以获取它知道如何解引用的 `&` 引用。

当我们在 Listing 15-9 中输入 `*y` 时，Rust 实际上在幕后运行了以下代码：

```rust,ignore
*(y.deref())
```

Rust 将 `*` 运算符替换为对 `deref` 方法的调用，然后是一个普通的解引用，因此我们不必考虑是否需要调用 `deref` 方法。这个 Rust 特性让我们编写的代码无论我们拥有常规引用还是实现 `Deref` 的类型，都能以相同的方式工作。

`deref` 方法返回对值的引用，以及 `*(y.deref())` 中括号外的普通解引用仍然是必要的，这与所有权系统有关。如果 `deref` 方法直接返回值而不是对值的引用，则该值将从 `self` 中移出。在这种情况下，我们不希望获取 `MyBox<T>` 内部值的所有权，或者在我们使用解引用运算符的大多数情况下都不希望这样做。

请注意，每次我们在代码中使用 `*` 时，`*` 运算符都会被替换为对 `deref` 方法的调用，然后是对 `*` 运算符的一次调用。因为 `*` 运算符的替换不会无限递归，所以我们最终会得到类型为 `i32` 的数据，这与 Listing 15-9 中的 `assert_eq!` 中的 `5` 匹配。

### 使用函数和方法进行隐式解引用强制转换

_解引用强制转换_ 将实现了 `Deref` 特性的类型的引用转换为另一种类型的引用。例如，解引用强制转换可以将 `&String` 转换为 `&str`，因为 `String` 实现了 `Deref` 特性，使其返回 `&str`。解引用强制转换是 Rust 对函数和方法参数执行的一种便利操作，并且仅适用于实现了 `Deref` 特性的类型。当我们传递对特定类型值的引用作为参数给函数或方法时，如果该参数类型与函数或方法定义中的参数类型不匹配，Rust 会自动执行解引用强制转换。对 `deref` 方法的一系列调用将我们提供的类型转换为参数所需的类型。

Rust 添加了解引用强制转换，以便编写函数和方法调用的程序员不需要添加那么多显式的引用和解引用操作符 `&` 和 `*`。解引用强制转换功能还让我们能够编写更多可以同时适用于引用或智能指针的代码。

要查看解引用强制转换的实际效果，让我们使用 Listing 15-8 中定义的 `MyBox<T>` 类型以及我们在 Listing 15-10 中添加的 `Deref` 实现。Listing 15-11 显示了一个具有字符串切片参数的函数的定义。

<Listing number="15-11" file-name="src/main.rs" caption="一个具有 `&str` 类型参数 `name` 的 `hello` 函数">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

</Listing>

我们可以使用字符串切片作为参数调用 `hello` 函数，例如 `hello("Rust");`。解引用强制转换使得可以使用 `MyBox<String>` 类型的值的引用调用 `hello`，如 Listing 15-12 所示。

<Listing number="15-12" file-name="src/main.rs" caption="使用 `MyBox<String>` 值的引用调用 `hello`，由于解引用强制转换而有效">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

</Listing>

在这里，我们使用参数 `&m` 调用 `hello` 函数，`&m` 是对 `MyBox<String>` 值的引用。因为我们在 Listing 15-10 中为 `MyBox<T>` 实现了 `Deref` 特性，Rust 可以通过调用 `deref` 将 `&MyBox<String>` 转换为 `&String`。标准库提供了 `String` 上的 `Deref` 实现，它返回一个字符串切片，这在 `Deref` 的 API 文档中有说明。Rust 再次调用 `deref` 将 `&String` 转换为 `&str`，这与 `hello` 函数的定义匹配。

如果 Rust 没有实现解引用强制转换，我们将不得不编写 Listing 15-13 中的代码，而不是 Listing 15-12 中的代码，以使用 `&MyBox<String>` 类型的值调用 `hello`。

<Listing number="15-13" file-name="src/main.rs" caption="如果 Rust 没有解引用强制转换，我们将不得不编写的代码">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

</Listing>

`(*m)` 将 `MyBox<String>` 解引用为 `String`。然后 `&` 和 `[..]` 获取 `String` 的字符串切片，该切片等于整个字符串以匹配 `hello` 的签名。没有解引用强制转换的代码更难阅读、编写和理解，因为涉及所有这些符号。解引用强制转换允许 Rust 自动为我们处理这些转换。

当为相关类型定义了 `Deref` 特性时，Rust 将分析类型并根据需要多次使用 `Deref::deref` 以获取与参数类型匹配的引用。`Deref::deref` 需要插入的次数在编译时解决，因此利用解引用强制转换不会带来运行时开销！

### 解引用强制转换与可变性的交互

类似于你如何使用 `Deref` 特性覆盖不可变引用上的 `*` 运算符，你可以使用 `DerefMut` 特性覆盖可变引用上的 `*` 运算符。

Rust 在发现类型和特性实现时会在三种情况下执行解引用强制转换：

1. 从 `&T` 到 `&U`，当 `T: Deref<Target=U>`
2. 从 `&mut T` 到 `&mut U`，当 `T: DerefMut<Target=U>`
3. 从 `&mut T` 到 `&U`，当 `T: Deref<Target=U>`

前两种情况相同，只是第二种情况实现了可变性。第一种情况说明，如果你有一个 `&T`，并且 `T` 实现了 `Deref` 到某个类型 `U`，你可以透明地获得一个 `&U`。第二种情况说明，相同的解引用强制转换也适用于可变引用。

第三种情况更复杂：Rust 还会将可变引用强制转换为不可变引用。但反过来是不可能的：不可变引用永远不会强制转换为可变引用。由于借用规则，如果你有一个可变引用，那么该可变引用必须是该数据的唯一引用（否则，程序将无法编译）。将一个可变引用转换为一个不可变引用永远不会违反借用规则。将一个不可变引用转换为一个可变引用将要求初始的不可变引用是该数据的唯一不可变引用，但借用规则并不保证这一点。因此，Rust 不能假设将不可变引用转换为可变引用是可能的。

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
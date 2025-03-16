<!-- 旧标题。请勿删除，否则链接可能会失效。 -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>

## 闭包：可以捕获环境的匿名函数

Rust 的闭包是可以保存在变量中或作为参数传递给其他函数的匿名函数。你可以在一个地方创建闭包，然后在另一个地方调用它，以在不同的上下文中执行它。与函数不同，闭包可以从定义它们的作用域中捕获值。我们将展示这些闭包特性如何实现代码重用和行为定制。

<!-- 旧标题。请勿删除，否则链接可能会失效。 -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>

### 使用闭包捕获环境

我们将首先研究如何使用闭包从它们定义的环境中捕获值以供以后使用。以下是场景：每隔一段时间，我们的 T 恤公司会向邮件列表中的某个人赠送一件独家限量版 T 恤作为促销活动。邮件列表中的人可以选择将他们的最喜欢的颜色添加到他们的个人资料中。如果被选中获得免费 T 恤的人设置了他们最喜欢的颜色，他们将获得该颜色的 T 恤。如果该人没有指定最喜欢的颜色，他们将获得公司当前库存最多的颜色。

有许多方法可以实现这一点。在这个例子中，我们将使用一个名为 `ShirtColor` 的枚举，它有 `Red` 和 `Blue` 两个变体（为了简化，限制了可用的颜色数量）。我们用 `Inventory` 结构体表示公司的库存，该结构体有一个名为 `shirts` 的字段，包含一个 `Vec<ShirtColor>`，表示当前库存的 T 恤颜色。定义在 `Inventory` 上的 `giveaway` 方法获取免费 T 恤获奖者的可选 T 恤颜色偏好，并返回该人将获得的 T 恤颜色。此设置如代码清单 13-1 所示：

<Listing number="13-1" file-name="src/main.rs" caption="T 恤公司赠送场景">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

在 `main` 中定义的 `store` 有两件蓝色 T 恤和一件红色 T 恤剩余，用于此次限量版促销活动。我们为有红色 T 恤偏好的用户和没有任何偏好的用户调用 `giveaway` 方法。

再次强调，这段代码可以通过多种方式实现，在这里，为了专注于闭包，我们坚持使用你已经学过的概念，除了 `giveaway` 方法的主体使用了闭包。在 `giveaway` 方法中，我们获取用户偏好作为 `Option<ShirtColor>` 类型的参数，并在 `user_preference` 上调用 `unwrap_or_else` 方法。标准库定义了 [`Option<T>` 上的 `unwrap_or_else` 方法][unwrap-or-else]<!-- ignore -->。它接受一个参数：一个不带任何参数的闭包，返回一个值 `T`（与 `Option<T>` 的 `Some` 变体中存储的类型相同，在这种情况下是 `ShirtColor`）。如果 `Option<T>` 是 `Some` 变体，`unwrap_or_else` 返回 `Some` 中的值。如果 `Option<T>` 是 `None` 变体，`unwrap_or_else` 调用闭包并返回闭包返回的值。

我们将闭包表达式 `|| self.most_stocked()` 指定为 `unwrap_or_else` 的参数。这是一个不带参数的闭包（如果闭包有参数，它们将出现在两个竖线之间）。闭包的主体调用 `self.most_stocked()`。我们在这里定义闭包，`unwrap_or_else` 的实现将在需要结果时评估闭包。

运行此代码会打印：

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

这里一个有趣的方面是，我们传递了一个在当前 `Inventory` 实例上调用 `self.most_stocked()` 的闭包。标准库不需要了解我们定义的 `Inventory` 或 `ShirtColor` 类型，或者我们在此场景中想要使用的逻辑。闭包捕获了对 `self` `Inventory` 实例的不可变引用，并将其与我们指定的代码一起传递给 `unwrap_or_else` 方法。另一方面，函数无法以这种方式捕获它们的环境。

### 闭包类型推断和注解

函数和闭包之间还有更多区别。闭包通常不需要像 `fn` 函数那样注解参数或返回值的类型。函数需要类型注解，因为类型是暴露给用户的显式接口的一部分。严格定义此接口对于确保每个人都同意函数使用和返回的值类型非常重要。另一方面，闭包不会像这样暴露在接口中：它们存储在变量中，并在不命名它们的情况下使用，不会暴露给我们库的用户。

闭包通常很短，并且仅在狭窄的上下文中相关，而不是在任意场景中。在这些有限的上下文中，编译器可以推断参数和返回值的类型，类似于它能够推断大多数变量的类型（在极少数情况下，编译器也需要闭包类型注解）。

与变量一样，如果我们希望增加明确性和清晰度，可以添加类型注解，尽管这会使代码比严格必要的更冗长。为闭包添加类型注解将如代码清单 13-2 所示。在这个例子中，我们定义了一个闭包并将其存储在变量中，而不是像代码清单 13-1 中那样在传递它作为参数的地方定义闭包。

<Listing number="13-2" file-name="src/main.rs" caption="在闭包中添加可选的参数和返回值类型注解">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

添加类型注解后，闭包的语法看起来更类似于函数的语法。这里我们定义了一个将其参数加 1 的函数和一个具有相同行为的闭包，以进行比较。我们添加了一些空格以对齐相关部分。这说明了闭包语法与函数语法的相似之处，除了使用管道和语法的可选部分：

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

第一行显示了一个函数定义，第二行显示了一个完全注解的闭包定义。在第三行中，我们从闭包定义中删除了类型注解。在第四行中，我们删除了大括号，因为闭包主体只有一个表达式，所以大括号是可选的。这些都是有效的定义，当它们被调用时会产生相同的行为。`add_one_v3` 和 `add_one_v4` 行要求闭包被评估才能编译，因为类型将从它们的用法中推断出来。这类似于 `let v = Vec::new();` 需要类型注解或将某种类型的值插入 `Vec` 中，以便 Rust 能够推断类型。

对于闭包定义，编译器将为它们的每个参数和返回值推断一个具体类型。例如，代码清单 13-3 显示了一个短闭包的定义，它只返回它接收到的参数值。这个闭包除了用于此示例的目的外并不十分有用。请注意，我们没有在定义中添加任何类型注解。因为没有类型注解，我们可以用任何类型调用闭包，这里我们第一次用 `String` 调用它。如果我们随后尝试用整数调用 `example_closure`，我们会得到一个错误。

<Listing number="13-3" file-name="src/main.rs" caption="尝试用两种不同的类型调用一个类型推断的闭包">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

编译器给我们这个错误：

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

我们第一次用 `String` 值调用 `example_closure` 时，编译器推断 `x` 的类型和闭包的返回类型为 `String`。这些类型随后被锁定在 `example_closure` 中的闭包中，当我们下次尝试用不同的类型使用同一个闭包时，我们会得到一个类型错误。

### 捕获引用或移动所有权

闭包可以通过三种方式从环境中捕获值，这直接映射到函数可以接受参数的三种方式：不可变借用、可变借用和获取所有权。闭包将根据函数主体如何处理捕获的值来决定使用哪种方式。

在代码清单 13-4 中，我们定义了一个闭包，它捕获了对名为 `list` 的向量的不可变引用，因为它只需要一个不可变引用来打印值：

<Listing number="13-4" file-name="src/main.rs" caption="定义并调用一个捕获不可变引用的闭包">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

这个例子还说明了一个变量可以绑定到一个闭包定义，我们可以稍后通过使用变量名和括号来调用闭包，就像变量名是函数名一样。

因为我们可以同时拥有多个对 `list` 的不可变引用，`list` 在闭包定义之前、闭包定义之后但在调用闭包之前以及调用闭包之后仍然可以从代码中访问。这段代码编译、运行并打印：

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

接下来，在代码清单 13-5 中，我们更改了闭包主体，使其向 `list` 向量添加一个元素。闭包现在捕获了一个可变引用：

<Listing number="13-5" file-name="src/main.rs" caption="定义并调用一个捕获可变引用的闭包">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

这段代码编译、运行并打印：

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

请注意，在 `borrows_mutably` 闭包的定义和调用之间不再有 `println!`：当 `borrows_mutably` 被定义时，它捕获了对 `list` 的可变引用。我们在闭包调用后不再使用闭包，因此可变借用结束。在闭包定义和闭包调用之间，不允许进行不可变借用来打印，因为在存在可变借用时不允许其他借用。尝试在那里添加一个 `println!` 看看你会得到什么错误信息！

如果你希望强制闭包获取它使用的环境值的所有权，即使闭包的主体并不严格需要所有权，你可以在参数列表前使用 `move` 关键字。

这种技术主要在将闭包传递给新线程以移动数据以便新线程拥有数据时有用。我们将在第 16 章讨论线程以及为什么你想使用它们时详细讨论并发，但现在，让我们简要探讨一下使用需要 `move` 关键字的闭包生成新线程。代码清单 13-6 显示了代码清单 13-4 的修改版本，以在新线程而不是主线程中打印向量：

<Listing number="13-6" file-name="src/main.rs" caption="使用 `move` 强制线程的闭包获取 `list` 的所有权">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

我们生成一个新线程，将闭包作为参数传递给线程运行。闭包主体打印出列表。在代码清单 13-4 中，闭包只使用不可变引用捕获了 `list`，因为这是打印它所需的最少访问量。在这个例子中，即使闭包主体仍然只需要一个不可变引用，我们需要通过在闭包定义的开头放置 `move` 关键字来指定 `list` 应该移动到闭包中。新线程可能在主线程的其余部分完成之前完成，或者主线程可能先完成。如果主线程保持对 `list` 的所有权但在新线程完成之前结束并丢弃 `list`，线程中的不可变引用将无效。因此，编译器要求将 `list` 移动到给新线程的闭包中，以便引用有效。尝试删除 `move` 关键字或在闭包定义后在主线程中使用 `list`，看看你会得到什么编译器错误！

<!-- 旧标题。请勿删除，否则链接可能会失效。 -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>

### 将捕获的值移出闭包和 `Fn` 特性

一旦闭包从定义它的环境中捕获了引用或捕获了值的所有权（从而影响什么，如果有的话，被移动到闭包中），闭包主体中的代码定义了当闭包稍后被评估时引用或值会发生什么（从而影响什么，如果有的话，被移出闭包）。闭包主体可以执行以下任何操作：将捕获的值移出闭包、改变捕获的值、既不移动也不改变值，或者根本不从环境中捕获任何内容。

闭包从环境中捕获和处理值的方式会影响闭包实现的特性，而特性是函数和结构体可以指定它们可以使用哪些闭包的方式。闭包将根据闭包主体如何处理值自动实现一个、两个或所有三个 `Fn` 特性，以累加的方式：

1. `FnOnce` 适用于可以调用一次的闭包。所有闭包至少实现此特性，因为所有闭包都可以被调用。将捕获的值移出其主体的闭包将仅实现 `FnOnce` 而不实现其他 `Fn` 特性，因为它只能被调用一次。
2. `FnMut` 适用于不将捕获的值移出其主体但可能会改变捕获值的闭包。这些闭包可以被多次调用。
3. `Fn` 适用于不将捕获的值移出其主体且不改变捕获值的闭包，以及不捕获环境中任何内容的闭包。这些闭包可以多次调用而不改变它们的环境，这在诸如多次并发调用闭包的情况下很重要。

让我们看看我们在代码清单 13-1 中使用的 `Option<T>` 上的 `unwrap_or_else` 方法的定义：

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

回想一下，`T` 是表示 `Option` 的 `Some` 变体中值的类型的泛型类型。该类型 `T` 也是 `unwrap_or_else` 函数的返回类型：例如，在 `Option<String>` 上调用 `unwrap_or_else` 的代码将获得一个 `String`。

接下来，注意 `unwrap_or_else` 函数有一个额外的泛型类型参数 `F`。`F` 类型是名为 `f` 的参数的类型，它是我们在调用 `unwrap_or_else` 时提供的闭包。

在泛型类型 `F` 上指定的特性边界是 `FnOnce() -> T`，这意味着 `F` 必须能够被调用一次，不带参数，并返回一个 `T`。在特性边界中使用 `FnOnce` 表达了 `unwrap_or_else` 最多只会调用 `f` 一次的约束。在 `unwrap_or_else` 的主体中，我们可以看到如果 `Option` 是 `Some`，`f` 不会被调用。如果 `Option` 是 `None`，`f` 将被调用一次。因为所有闭包都实现 `FnOnce`，`unwrap_or_else` 接受所有三种闭包，并且尽可能灵活。

> 注意：如果我们想做的事情不需要从环境中捕获值，我们可以使用函数名而不是闭包。例如，我们可以在 `Option<Vec<T>>` 值上调用 `unwrap_or_else(Vec::new)`，如果值为 `None`，则获得一个新的空向量。编译器会自动为函数定义实现适用的 `Fn` 特性。

现在让我们看看标准库中定义在切片上的 `sort_by_key` 方法，看看它与 `unwrap_or_else` 有何不同，以及为什么 `sort_by_key` 使用 `FnMut` 而不是 `FnOnce` 作为特性边界。闭包以对切片中当前项的引用的形式获取一个参数，并返回一个可以排序的 `K` 类型的值。当你想按每个项的特定属性对切片进行排序时，此函数很有用。在代码清单 13-7 中，我们有一个 `Rectangle` 实例列表，我们使用 `sort_by_key` 按它们的 `width` 属性从低到高排序：

<Listing number="13-7" file-name="src/main.rs" caption="使用 `sort_by_key` 按宽度排序矩形">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

这段代码打印：

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

`sort_by_key` 被定义为接受 `FnMut` 闭包的原因是它多次调用闭包：对切片中的每个项调用一次。闭包 `|r| r.width` 不捕获、改变或移出环境中的任何内容，因此它满足特性边界要求。

相比之下，代码清单 13-8 显示了一个仅实现 `FnOnce` 特性的闭包示例，因为它从环境中移出了一个值。编译器不会让我们将此闭包与 `sort_by_key` 一起使用：

<Listing number="13-8" file-name="src/main.rs" caption="尝试将 `FnOnce` 闭包与 `sort_by_key` 一起使用">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

这是一种人为的、复杂的方式（不起作用）来尝试计算 `sort_by_key` 在排序 `list` 时调用闭包的次数。此代码试图通过将 `value`——闭包环境中的一个 `String`——推入 `sort_operations` 向量来进行此计数。闭包捕获 `value`，然后通过将 `value` 的所有权转移到 `sort_operations` 向量来将 `value` 移出闭包。此闭包可以被调用一次；尝试第二次调用它将不起作用，因为 `value` 将不再在环境中被推入 `sort_operations` 中！因此，此闭包仅实现 `FnOnce`。当我们尝试编译此代码时，我们得到此错误，指出 `value` 不能从闭包中移出，因为闭包必须实现 `FnMut`：

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

错误指向闭包主体中将 `value` 移出环境的行。要修复此问题，我们需要更改闭包主体，使其不移出环境中的值。在环境中保留一个计数器并在闭包主体中增加其值是计算闭包调用次数的更直接的方法。代码清单 13-9 中的闭包与 `sort_by_key` 一起工作，因为它只捕获了对 `num_sort_operations` 计数器的可变引用，因此可以被多次调用：

<Listing number="13-9" file-name="src/main.rs" caption="允许使用 `FnMut` 闭包与 `sort_by_key`">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

`Fn` 特性在定义或使用利用闭包的函数或类型时非常重要。在下一节中，我们将讨论迭代器。许多迭代器方法接受闭包参数，因此在我们继续时请记住这些闭包细节！

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
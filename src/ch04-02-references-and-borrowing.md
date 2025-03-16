## 引用与借用

在 Listing 4-5 中的元组代码的问题在于，我们必须将 `String` 返回给调用函数，以便在调用 `calculate_length` 后仍然可以使用 `String`，因为 `String` 被移动到了 `calculate_length` 中。相反，我们可以提供一个对 `String` 值的引用。**引用**类似于指针，它是一个地址，我们可以通过它访问存储在该地址的数据；这些数据由其他变量拥有。与指针不同的是，引用在其生命周期内保证指向某个特定类型的有效值。

下面是如何定义和使用一个 `calculate_length` 函数，该函数以引用作为参数，而不是获取值的所有权：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

首先，注意变量声明和函数返回值中的所有元组代码都消失了。其次，注意我们传递 `&s1` 给 `calculate_length`，并且在函数定义中，我们使用 `&String` 而不是 `String`。这些 `&` 符号表示**引用**，它们允许你引用某个值而不获取其所有权。图 4-6 描绘了这一概念。

<img alt="三个表：s 的表只包含指向 s1 表的指针。s1 的表包含 s1 的栈数据，并指向堆上的字符串数据。" src="img/trpl04-06.svg" class="center" />

<span class="caption">图 4-6: `&String s` 指向 `String s1` 的示意图</span>

> 注意：使用 `&` 进行引用的相反操作是**解引用**，它通过解引用操作符 `*` 完成。我们将在第 8 章看到解引用操作符的一些用法，并在第 15 章详细讨论解引用。

让我们仔细看看这里的函数调用：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

`&s1` 语法让我们创建一个引用，它**引用** `s1` 的值但不拥有它。因为引用不拥有它，所以当引用停止使用时，它指向的值不会被丢弃。

同样，函数的签名使用 `&` 来表明参数 `s` 的类型是一个引用。让我们添加一些解释性注释：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

变量 `s` 的有效范围与任何函数参数的范围相同，但当 `s` 停止使用时，引用指向的值不会被丢弃，因为 `s` 没有所有权。当函数使用引用作为参数而不是实际值时，我们不需要返回值来归还所有权，因为我们从未拥有过所有权。

我们将创建引用的行为称为**借用**。就像在现实生活中，如果一个人拥有某样东西，你可以从他们那里借用它。当你用完时，你必须归还它。你不拥有它。

那么，如果我们尝试修改我们借用的东西会发生什么？试试 Listing 4-6 中的代码。剧透警告：它不会工作！

<Listing number="4-6" file-name="src/main.rs" caption="尝试修改借用的值">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

以下是错误信息：

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

正如变量默认是不可变的一样，引用也是不可变的。我们不允许修改我们引用的东西。

### 可变引用

我们可以通过一些小的调整来修复 Listing 4-6 中的代码，以允许我们修改借用的值，使用**可变引用**：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

首先我们将 `s` 改为 `mut`。然后我们在调用 `change` 函数时使用 `&mut s` 创建一个可变引用，并更新函数签名以接受一个可变引用 `some_string: &mut String`。这使得 `change` 函数将改变它借用的值变得非常清晰。

可变引用有一个很大的限制：如果你有一个值的可变引用，你就不能有其他对该值的引用。尝试创建两个对 `s` 的可变引用的代码将失败：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

以下是错误信息：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

这个错误说明这段代码无效，因为我们不能同时多次借用 `s` 作为可变引用。第一次可变借用是在 `r1` 中，并且必须持续到它在 `println!` 中使用为止，但在创建该可变引用和它的使用之间，我们尝试在 `r2` 中创建另一个可变引用，它借用了与 `r1` 相同的数据。

防止同时存在多个对同一数据的可变引用的限制允许进行改变，但以非常受控的方式进行。这是新 Rust 程序员经常遇到的问题，因为大多数语言允许你随时进行改变。拥有这种限制的好处是，Rust 可以在编译时防止数据竞争。**数据竞争**类似于竞态条件，当以下三种行为发生时就会发生：

- 两个或更多指针同时访问同一数据。
- 至少有一个指针用于写入数据。
- 没有用于同步数据访问的机制。

数据竞争会导致未定义行为，并且在运行时尝试追踪它们时可能难以诊断和修复；Rust 通过拒绝编译带有数据竞争的代码来防止这个问题！

一如既往，我们可以使用花括号创建一个新的作用域，允许多个可变引用，只是不能**同时**存在：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

Rust 对结合可变和不可变引用执行了类似的规则。这段代码会导致错误：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

以下是错误信息：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

哇！我们**也不能**在有一个不可变引用的情况下拥有一个对同一值的可变引用。

不可变引用的用户不希望值突然在他们下面改变！然而，允许多个不可变引用，因为只读取数据的人没有能力影响其他人对数据的读取。

注意，引用的作用域从它被引入的地方开始，一直持续到最后一次使用该引用。例如，这段代码将编译，因为不可变引用的最后一次使用是在 `println!` 中，在引入可变引用之前：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

不可变引用 `r1` 和 `r2` 的作用域在它们最后一次使用的 `println!` 之后结束，这是在创建可变引用 `r3` 之前。这些作用域不重叠，所以这段代码是允许的：编译器可以判断出在作用域结束之前，引用不再被使用。

尽管借用错误有时可能令人沮丧，但请记住，这是 Rust 编译器在早期（在编译时而不是运行时）指出潜在错误，并准确地告诉你问题所在。这样你就不必追踪为什么你的数据不是你想象的那样。

### 悬垂引用

在有指针的语言中，很容易错误地创建一个**悬垂指针**——一个引用内存中可能已经分配给其他人的位置的指针——通过释放一些内存而保留指向该内存的指针。相比之下，在 Rust 中，编译器保证引用永远不会是悬垂引用：如果你有一个对某些数据的引用，编译器将确保数据不会在引用之前离开作用域。

让我们尝试创建一个悬垂引用，看看 Rust 如何通过编译时错误来防止它们：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

以下是错误信息：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

这个错误信息提到了我们尚未涉及的一个特性：生命周期。我们将在第 10 章详细讨论生命周期。但是，如果你忽略关于生命周期的部分，消息确实包含了为什么这段代码有问题的关键：

```text
此函数的返回类型包含一个借用值，但没有值可供借用
```

让我们仔细看看 `dangle` 代码的每个阶段发生了什么：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

因为 `s` 是在 `dangle` 内部创建的，当 `dangle` 的代码执行完毕时，`s` 将被释放。但我们试图返回一个对它的引用。这意味着这个引用将指向一个无效的 `String`。这不行！Rust 不会让我们这样做。

这里的解决方案是直接返回 `String`：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

这样就不会有任何问题。所有权被移出，没有任何东西被释放。

### 引用规则

让我们回顾一下我们讨论过的引用规则：

- 在任意给定时间，你可以**要么**拥有一个可变引用，**要么**拥有任意数量的不可变引用。
- 引用必须始终有效。

接下来，我们将看看另一种引用：切片。
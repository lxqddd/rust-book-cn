## 附录 A: 关键字

以下列表包含了 Rust 语言当前或将来保留的关键字。因此，它们不能用作标识符（除非作为原始标识符，我们将在“[原始标识符][raw-identifiers]<!-- ignore -->”部分讨论）。标识符是函数、变量、参数、结构体字段、模块、crate、常量、宏、静态值、属性、类型、特质或生命周期的名称。

[raw-identifiers]: #raw-identifiers

### 当前使用的关键字

以下是当前使用的关键字列表，并描述了它们的功能。

- `as` - 执行原始类型转换，消除包含项的特定特质的歧义，或在 `use` 语句中重命名项
- `async` - 返回一个 `Future` 而不是阻塞当前线程
- `await` - 暂停执行直到 `Future` 的结果准备好
- `break` - 立即退出循环
- `const` - 定义常量项或常量原始指针
- `continue` - 继续到下一个循环迭代
- `crate` - 在模块路径中，指代 crate 根目录
- `dyn` - 动态分派到特质对象
- `else` - `if` 和 `if let` 控制流结构的回退
- `enum` - 定义一个枚举
- `extern` - 链接外部函数或变量
- `false` - 布尔值 false 字面量
- `fn` - 定义一个函数或函数指针类型
- `for` - 遍历迭代器中的项，实现一个特质，或指定一个更高级的生命周期
- `if` - 根据条件表达式的结果进行分支
- `impl` - 实现固有或特质功能
- `in` - `for` 循环语法的一部分
- `let` - 绑定一个变量
- `loop` - 无条件循环
- `match` - 将值与模式匹配
- `mod` - 定义一个模块
- `move` - 使闭包获取其所有捕获的所有权
- `mut` - 表示引用、原始指针或模式绑定中的可变性
- `pub` - 表示结构体字段、`impl` 块或模块中的公共可见性
- `ref` - 通过引用绑定
- `return` - 从函数返回
- `Self` - 我们正在定义或实现的类型的别名
- `self` - 方法主体或当前模块
- `static` - 全局变量或持续整个程序执行的生命周期
- `struct` - 定义一个结构体
- `super` - 当前模块的父模块
- `trait` - 定义一个特质
- `true` - 布尔值 true 字面量
- `type` - 定义一个类型别名或关联类型
- `union` - 定义一个 [union][union]<!-- ignore -->；仅在 union 声明中作为关键字使用
- `unsafe` - 表示不安全的代码、函数、特质或实现
- `use` - 将符号引入作用域；为泛型和生命周期边界指定精确的捕获
- `where` - 表示约束类型的子句
- `while` - 根据表达式的结果有条件地循环

[union]: ../reference/items/unions.html

### 保留供将来使用的关键字

以下关键字目前没有任何功能，但 Rust 保留它们以供将来可能使用。

- `abstract`
- `become`
- `box`
- `do`
- `final`
- `gen`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

### 原始标识符

_原始标识符_ 是一种语法，允许你在通常不允许的地方使用关键字。你可以通过在关键字前加上 `r#` 来使用原始标识符。

例如，`match` 是一个关键字。如果你尝试编译以下使用 `match` 作为名称的函数：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

你会得到以下错误：

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

错误显示你不能使用关键字 `match` 作为函数标识符。要使用 `match` 作为函数名，你需要使用原始标识符语法，如下所示：

<span class="filename">文件名: src/main.rs</span>

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

这段代码将无错误地编译。注意函数定义中的 `r#` 前缀以及在 `main` 中调用函数时也使用了 `r#`。

原始标识符允许你选择任何单词作为标识符，即使该单词恰好是保留关键字。这为我们提供了更多选择标识符名称的自由，也让我们能够与这些单词不是关键字的语言编写的程序集成。此外，原始标识符允许你使用与你的 crate 使用的 Rust 版本不同的库。例如，`try` 在 2015 版本中不是关键字，但在 2018、2021 和 2024 版本中是关键字。如果你依赖一个使用 2015 版本编写的库，并且该库有一个 `try` 函数，你将需要使用原始标识符语法，在这种情况下是 `r#try`，以便在后续版本中从你的代码中调用该函数。有关版本的更多信息，请参见 [附录 E][appendix-e]<!-- ignore -->。

[appendix-e]: appendix-05-editions.html
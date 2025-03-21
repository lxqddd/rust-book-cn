# 函数式语言特性：迭代器与闭包

Rust 的设计灵感来源于许多现有的语言和技术，其中一个重要的影响是**函数式编程**。函数式编程风格通常包括将函数作为值使用，例如将它们作为参数传递、从其他函数返回、赋值给变量以便稍后执行等等。

在本章中，我们不会讨论什么是函数式编程或什么不是函数式编程，而是会讨论 Rust 中一些与许多被称为函数式语言中的特性相似的功能。

更具体地说，我们将涵盖以下内容：

- **闭包**，一种可以存储在变量中的类似函数的构造
- **迭代器**，一种处理一系列元素的方式
- 如何使用闭包和迭代器来改进第 12 章中的 I/O 项目
- 闭包和迭代器的性能（剧透警告：它们比你想象的要快！）

我们已经介绍了一些其他 Rust 特性，例如模式匹配和枚举，这些特性也受到了函数式风格的影响。由于掌握闭包和迭代器是编写地道、高效的 Rust 代码的重要部分，我们将用整章来讨论它们。
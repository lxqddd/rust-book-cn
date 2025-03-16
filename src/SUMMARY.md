# Rust 编程语言

[Rust 编程语言](title-page.md)
[前言](foreword.md)
[简介](ch00-00-introduction.md)

## 入门

- [开始入门](ch01-00-getting-started.md)
  - [安装](ch01-01-installation.md)
  - [Hello, World!](ch01-02-hello-world.md)
  - [Hello, Cargo!](ch01-03-hello-cargo.md)

- [编写一个猜数游戏](ch02-00-guessing-game-tutorial.md)

- [常见编程概念](ch03-00-common-programming-concepts.md)
  - [变量与可变性](ch03-01-variables-and-mutability.md)
  - [数据类型](ch03-02-data-types.md)
  - [函数](ch03-03-how-functions-work.md)
  - [注释](ch03-04-comments.md)
  - [控制流](ch03-05-control-flow.md)

- [理解所有权](ch04-00-understanding-ownership.md)
  - [什么是所有权？](ch04-01-what-is-ownership.md)
  - [引用与借用](ch04-02-references-and-borrowing.md)
  - [切片类型](ch04-03-slices.md)

- [使用结构体组织相关数据](ch05-00-structs.md)
  - [定义和实例化结构体](ch05-01-defining-structs.md)
  - [一个使用结构体的示例程序](ch05-02-example-structs.md)
  - [方法语法](ch05-03-method-syntax.md)

- [枚举与模式匹配](ch06-00-enums.md)
  - [定义枚举](ch06-01-defining-an-enum.md)
  - [`match` 控制流构造器](ch06-02-match.md)
  - [使用 `if let` 和 `let else` 实现简洁的控制流](ch06-03-if-let.md)

## Rust 基础

- [使用包、库和模块管理不断发展的项目](ch07-00-managing-growing-projects-with-packages-crates-and-modules.md)
  - [包和库](ch07-01-packages-and-crates.md)
  - [定义模块以控制作用域和私有性](ch07-02-defining-modules-to-control-scope-and-privacy.md)
  - [引用模块树中某个项的路径](ch07-03-paths-for-referring-to-an-item-in-the-module-tree.md)
  - [使用 `use` 关键字将路径引入作用域](ch07-04-bringing-paths-into-scope-with-the-use-keyword.md)
  - [将模块拆分到不同文件中](ch07-05-separating-modules-into-different-files.md)

- [常见集合](ch08-00-common-collections.md)
  - [使用向量存储值列表](ch08-01-vectors.md)
  - [使用字符串存储 UTF-8 编码的文本](ch08-02-strings.md)
  - [在哈希映射中存储键及其关联值](ch08-03-hash-maps.md)

- [错误处理](ch09-00-error-handling.md)
  - [使用 `panic!` 处理不可恢复的错误](ch09-01-unrecoverable-errors-with-panic.md)
  - [使用 `Result` 处理可恢复的错误](ch09-02-recoverable-errors-with-result.md)
  - [是否使用 `panic!`](ch09-03-to-panic-or-not-to-panic.md)

- [泛型、特性和生命周期](ch10-00-generics.md)
  - [泛型数据类型](ch10-01-syntax.md)
  - [特性：定义共享行为](ch10-02-traits.md)
  - [使用生命周期验证引用](ch10-03-lifetime-syntax.md)

- [编写自动化测试](ch11-00-testing.md)
  - [如何编写测试](ch11-01-writing-tests.md)
  - [控制测试的运行方式](ch11-02-running-tests.md)
  - [测试组织](ch11-03-test-organization.md)

- [一个 I/O 项目：构建命令行程序](ch12-00-an-io-project.md)
  - [接受命令行参数](ch12-01-accepting-command-line-arguments.md)
  - [读取文件](ch12-02-reading-a-file.md)
  - [重构以提高模块化和错误处理能力](ch12-03-improving-error-handling-and-modularity.md)
  - [使用测试驱动开发开发库的功能](ch12-04-testing-the-librarys-functionality.md)
  - [处理环境变量](ch12-05-working-with-environment-variables.md)
  - [将错误消息写入标准错误而非标准输出](ch12-06-writing-to-stderr-instead-of-stdout.md)

## Rust 思维方式

- [函数式语言特性：迭代器和闭包](ch13-00-functional-features.md)
  - [闭包：捕获其环境的匿名函数](ch13-01-closures.md)
  - [使用迭代器处理一系列项](ch13-02-iterators.md)
  - [改进我们的 I/O 项目](ch13-03-improving-our-io-project.md)
  - [比较性能：循环与迭代器](ch13-04-performance.md)

- [更多关于 Cargo 和 Crates.io 的内容](ch14-00-more-about-cargo.md)
  - [使用发布配置文件自定义构建](ch14-01-release-profiles.md)
  - [将 crate 发布到 Crates.io](ch14-02-publishing-to-crates-io.md)
  - [Cargo 工作区](ch14-03-cargo-workspaces.md)
  - [使用 `cargo install` 从 Crates.io 安装二进制文件](ch14-04-installing-binaries.md)
  - [使用自定义命令扩展 Cargo](ch14-05-extending-cargo.md)

- [智能指针](ch15-00-smart-pointers.md)
  - [使用 `Box<T>` 指向堆上的数据](ch15-01-box.md)
  - [使用 `Deref` 将智能指针当作常规引用处理](ch15-02-deref.md)
  - [使用 `Drop` 特性在清理时运行代码](ch15-03-drop.md)
  - [`Rc<T>`，引用计数智能指针](ch15-04-rc.md)
  - [`RefCell<T>` 和内部可变性模式](ch15-05-interior-mutability.md)
  - [引用循环可能导致内存泄漏](ch15-06-reference-cycles.md)

- [无畏并发](ch16-00-concurrency.md)
  - [使用线程同时运行代码](ch16-01-threads.md)
  - [使用消息传递在线程间传输数据](ch16-02-message-passing.md)
  - [共享状态并发](ch16-03-shared-state.md)
  - [使用 `Send` 和 `Sync` 特性实现可扩展的并发](ch16-04-extensible-concurrency-sync-and-send.md)

- [异步编程基础：Async、Await、Futures 和 Streams](ch17-00-async-await.md)
  - [Futures 和 Async 语法](ch17-01-futures-and-syntax.md)
  - [使用 Async 实现并发](ch17-02-concurrency-with-async.md)
  - [处理任意数量的 Futures](ch17-03-more-futures.md)
  - [Streams：顺序执行的 Futures](ch17-04-streams.md)
  - [深入研究 Async 的特性](ch17-05-traits-for-async.md)
  - [Futures、Tasks 和 Threads](ch17-06-futures-tasks-threads.md)

- [Rust 的面向对象编程特性](ch18-00-oop.md)
  - [面向对象语言的特性](ch18-01-what-is-oo.md)
  - [使用允许不同类型值的 Trait 对象](ch18-02-trait-objects.md)
  - [实现面向对象设计模式](ch18-03-oo-design-patterns.md)

## 高级主题

- [模式与匹配](ch19-00-patterns.md)
  - [模式的所有使用场景](ch19-01-all-the-places-for-patterns.md)
  - [可反驳性：模式是否可能匹配失败](ch19-02-refutability.md)
  - [模式语法](ch19-03-pattern-syntax.md)

- [高级特性](ch20-00-advanced-features.md)
  - [不安全的 Rust](ch20-01-unsafe-rust.md)
  - [高级特性](ch20-02-advanced-traits.md)
  - [高级类型](ch20-03-advanced-types.md)
  - [高级函数和闭包](ch20-04-advanced-functions-and-closures.md)
  - [宏](ch20-05-macros.md)

- [最终项目：构建多线程 Web 服务器](ch21-00-final-project-a-web-server.md)
  - [构建单线程 Web 服务器](ch21-01-single-threaded.md)
  - [将我们的单线程服务器转换为多线程服务器](ch21-02-multithreaded.md)
  - [优雅关闭和清理](ch21-03-graceful-shutdown-and-cleanup.md)

- [附录](appendix-00.md)
  - [A - 关键字](appendix-01-keywords.md)
  - [B - 运算符和符号](appendix-02-operators.md)
  - [C - 可派生特性](appendix-03-derivable-traits.md)
  - [D - 有用的开发工具](appendix-04-useful-development-tools.md)
  - [E - 版本](appendix-05-editions.md)
  - [F - 本书的翻译版本](appendix-06-translation.md)
  - [G - Rust 的构建过程与 “Nightly Rust”](appendix-07-nightly-rust.md)
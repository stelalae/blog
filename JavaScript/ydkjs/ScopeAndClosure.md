# 作用域和闭包

## 作用域是什么

### 编译原理

 JavaScript 通常被归类为“动态”或“解释执行”语言，但事实上它是一门编译语言。与传统的编译语言不同，它不是提前编译的，而是在目标环境的JavaScript引擎中解释执行的，所以JavaScript的编译结果不能在分布式系统中进行移植。



传统编译语言会有下面步骤，统称为“编译”：

- 分词/词法分析（Tokenizing/Lexing）：即把编写的代码行或块分析成词法单元。如 var a = 2；被分解成var、a、=、2 、;和空格。
- 解析/语法分析（Parsing）：将词法单元流（数组）转换成一个由元素逐级嵌套所组成的代表了程序语法 结构的树，这个树被称为“抽象语法树”（Abstract Syntax Tree，AST）。
- 代码生成：将 AST 转换为可执行代码的过程称被称为代码生成。这个过程与语言、目标平台等息息相关。



JavaScript 引擎要复杂得多。例如，在语法分析和代码生成阶段有特定的步骤来对运行性能进行优化，包括对冗余元素进行优化等。但JavaScript引擎不会有大量时间来进行优化，大部分情况下编译发生在代码执行前的几微秒（甚至更短！）的时 间内。即任何 JavaScript 代码片段在执行前都要进行编译（通常就在执行前）。

*笔者注，传统编译语言的编译器也会对传统语言代码进行代码优化和性能优化，这往往取决于编译器的设计，如C++编译器GCC和LLVM就有很大不同。*



### 理解作用域

### 作用域嵌套

### 异常

## 词法作用域
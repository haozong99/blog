### babel 是什么

> babel简介

Babel 是一个 JavaScript 编译器
Babel 是一个工具链，主要用于将采用 ECMAScript 2015+ 语法编写的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。下面列出的是 Babel 能为你做的事情：

![babel-flow](./_images/babel-flow.jpg)

- parse： 用于将源代码编译成AST抽象语法树
- transform： 用于对AST抽象语法树进行改造
- generator： 用于将改造后的AST抽象语法树转换成目标代码

很明显AST抽象语法树在这里充当了一个中间人的身份，作用就是可以通过对AST的操作还达到源代码到目标代码的转换过程，这将会比暴力使用正则匹配要优雅的多。

### AST抽象语法树

> 在计算机科学中，抽象语法树（Abstract Syntax Tree，AST） 是源代码语法结构的一种抽象表示。它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。

- AST抽象语法树是源代码语法结构的一种抽象表示
- 每个包含type属性的数据结构，都是一个AST节点
- 它以树状的形式表现编程语言的语法结构，每个节点都表示源代码中的一种结构

#### AST结构

> 为了统一ECMAScript标准的语法表达。社区中衍生出了ESTree Spec，是目前前端所遵循的一种语法表达标准

节点类型

| 类型 | 说明 |
| ---- | ---- |
| File | 文件 (顶层节点包含 Program) |
| Program | 整个程序节点 (包含 body 属性代表程序体) |
| Directive | 指令(例如 "use strict") | 
| Comment | 代码注释 |
| Statement | 语句 (可独立执行的语句) |
| Literal | 字面量 (基本数据类型、复杂数据类型等值类型) |
| Identifier | 标识符 (变量名、属性名、函数名、参数名等) |
| Declaration | 声明 (变量声明、函数声明、Import、Export 声明等) |
| Specifier | 关键字 (ImportSpecifier、ImportDefaultSpecifier、ImportNamespaceSpecifier、ExportSpecifier) |
| Expression | 表达式 |

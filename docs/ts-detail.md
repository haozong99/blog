## typescript 的优点

强大的类型检查, 包含泛型

- 特点
  + 跨平台
  + ES6 特性
  + 面向对象的语言
  + 静态类型检查

## Typescript 支持修饰符

- public 类的所用成员， 其子类以及该类的实例都可以访问。
- protected 该类及其子类的所有成员都可以访问他们，但是该类的实例无法访问
- private 只有类成员可以访问他们

## Declare 关键字有什么作用

生命已经存在于其他地方的变量、函数类、模块或命名空间；
这通常用于将javascript 代码或其他外部代码引入到typescript.使Typescript 编译器知道这些外部实体的存在，并能正确地进入类型检查。

## Typescript 中的枚举 Enum

枚举是Typescript 数据类型，他允许我们定义一组命名变量。使用枚举去创建一组不同案例变得更加容易。它是相关的集合， 可以是`数字值`或者`字符串值`. 没有特定值对应时，是下标

## Typescript 中什么是装饰器

> 装饰器是一种特殊类型的声明，它能过被附加到类声明，方法， 属性或者参数上，可以修改类的行为 @

```javascript
// 启用装饰器
{
  "compilerOptions": {
    "experimentalDecorators": true
  }
}
```

装饰器的分类： 类装饰器、属性装饰器、方法装饰器、参数装饰器

## Typescript 中的模块是什么

Typescript 中的模块是相关变量、函数、类和接口的集合。你可以将模块视为包含执行任务所需的一切容器。可以导入模块以轻松地在项目之间共享代码

```typescript
module module_name{
  class xyz{
    export sum(x, y){
      return x+y;
    }
  }
}
```

## Typescript 中 never 和 void 有什么区别

- void 表示没有任何类型（null , undefined）

- never 表示一个不包含值的类型， 表示不存在的值

- void 返回值类型的函数能正常运行。拥有never 返回类型的函数无法正常返回，无法终止，或会抛出异常。

## any 和 unknown 有什么区别吗？

- unknown类型会更加严格；
- unknown 是any 的安全版本；
- unknown 没有经过类型坚持不能赋值给任意类型

## Typescript 中的类型断言是什么

关键字 是 as , 直接赋予类型。

## 使用ts 实现一个判断传入参数是否是数组类型的方法？

```typescript
function isArray(x: unknown): boolean{
  if(Array.isArray(x)){
    return true;
  }
  return false;
}
```

## tsconfig.json 有什么作用

tsconfig.json文件是JSON格式的文件。

在tsconfig.json文件中，可以指定不同的选项来告诉编译器如何编译当前项目。

```


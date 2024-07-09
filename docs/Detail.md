## React 组件命名推荐的方式是哪个？

组件名： 大驼峰命名

文件名： 短横线命名法

## React  解决了什么的问题? 最新的版本添加了什么内容，

- react 19

添加了react compiler, 让组件更新只发生在需要更新的组件，从而减少re-render 的组件。

- react 18
发布了许多的新特性，uesTrasition, useId
知名开源库集成并发特性

- react 17
breaking change, 用于平缓过渡
删除一些componentWillReceiveProps, componentWillUpdate

- react 16
16.8 引入了Hooks, 

## React 怎么实现一个全局的dialog

通过createPortal, createContext;
在子组件通过useDialog 来使用

## React 数据持久化
- local Storage
- Session Storage
- Cookies
- IndexDB

## 使用typesctip 来配置react , 需要哪些配置

ts-loader typescript
添加tsconfig.js

## React 中props.children 和 React.Children 的区别
`props.children`是一个特殊属性，它包含组件的所有子元素
实例的内容
`React.Children`是一个React 提供的工具函数集， 用于处理`props.children`. 
一些api


## React 中状态提升是什么？ 使用场景有哪些?
就是两个子组件中同用一个state 来显示，可以提升到父组件
场景：表单输入同步，组件通信

## React 中constructor 和 getinitalState 的区别
constructor 是类组件
getinitalState 是对象组件

## react中的严格模式有什么用
StrictMode
> 帮助开发者识别代码中的潜在问题和不推荐的用法

1. 识别到不建议使用的什么周期
2. 检查意外的副作用
3. 标识旧字符 ref Api 
4. 验证状态更新的安全性

## react, react-dom, babel.js 这三个库有什么用处
react 主要是支撑组件react 中的一些api 
react-dom 是与dom 相关的一些，一个挂载更新与卸载
babel 是将jsx 中的模板语法装换为react.createElement.

## React.children.map 和 array.map 有什么区别

React.children.map 
1. 处理任意类型的子元素
    无论是单个元素还是数组，`react.children.map`都能正确处理
2. 保证key的存在
    `React.Children.map`会自动为每个元素分配一个key, 如果你没有显式提供`key`属性
3. 类型检查
    React.Children.map 会对 props.children 进行类型检查，确保传递给它的子元素是有效的 React 元素。

# 路由
## React-router 的实现原理是什么

### hash

1. hash 是 url 中 hash(#)及后面的部分，常用锚点在页面内做导航，
2. 通过hashchange 的事件监听URL 的改变. 改变URL 的方式只有以下几种：
    通过浏览器导航栏的前进后退、通过<a>标签、通过window.location。

### history


1. history 提供了pushstate 和 replaceState 这两个方法， 这两个方法改变URL 的path 部分不会引起页面刷新。
2. 通过popstate 事件监听URL的改变。需要注意只在通过浏览器导航栏的前进后退改变URL时会popstate

判断location.pathname 来识别是哪个路径


## react-router 里的link 标签和 a 标签的区别

1. 更新页面保持应用的状态和性能(redux);
2. 更多api 与React router 以及在应中可以使用的参数
3. 不用去处理a 标签的默认事件。(锚点，页面刷新)


## 如何解决props 层级过深的问题

1. 使用createContext, useContext;
2. 使用redux 
3. HOC 
4. 自定义hooks

## useEffect 与 useLayoutEffect 的区别

- useEffect 
    + 在浏览器完成布局和绘制之后异步调用;
    + 数据获取、事件订阅;

- useLayoutEffect 
    + 在浏览器完成布局和绘制之前同步调用;
    + 读取布局信息、同步布局


## memo、useMemo、 useCallback
- Memo 用于高阶函数组件
- useMemo 用来缓存一个计算属性
- useCallback 用于包装方法





## React 中设置页面更新的api 
- ReactDom.render HostRoot
- this.setState ClassComponent
- this.forceUpdate ClassComponent
- useState FunctionComponent
- useReducer FunctionComponent


4. 使用context

```javascript
const MyContext = React.createContext();
<MyContext.Provier value={newValue}>
  <ChildrenComp/>
<MyContext.Provier>

const value = useContext(MyContext);

```

5. useReducer

6. useEffect

## 说一下对jsx的理解
好处: 创建组件标记提供了一种更好的方法。

- jsx 是对react 中的模板语法中的一个特殊后缀名。
- 文件最上方需要导入react的库
- 函数组件或者类组件中的模板语法返回会被babel转换为react.createElement

## React的生命周期

- 初始化(Initialzation)
  - constructor
- 挂载(Mounting)
  - componentWillMount
  - render
  - componentDidMount
- 更新(Updation)
  - props 
    + componentWillReceiveProps(nextProps)
      在 React 版本 16.3.0 开始被废弃，计划在 React 17 中完全移除。
      UNSAFE_componentWillReceiveProps
      
    + shouldComponentUpdate(nextProps, nextState, nextContext)
      这个 api 中可以选择是否返回true (更新), 返回false (不更新)

    + componentWillUpdate(nextProps, nextState) 
      在 React 版本 16.3.0 开始被废弃，计划在 React 17 中完全移除。
      UNSAFE_componentWillReceiveProps

    + render
    + componentDidUpdate(prevProps, prevState)
   

- 卸载(Unmounting)
  - componentWillUnmount

## React事件机制和原生DOM事件流有什么区别

react中的事件是绑定到document上面的，

而原生的事件是绑定到dom上面的，

因此相对绑定的地方来说，dom上的事件要优先于document上的事件执行. react合并多次事件，一起来更新。减少事件绑定的时间。

## React hooks解决了什么问题? 函数组件与类组件的区别

- 状态逻辑复用;相同的hooks 逻辑可以抽离出来.
- 类组件的复杂性;this 
- 副作用处理; 每一个useHooks 用来单独处理逻辑，不用堆积在生命周期中

## setState是同步还是异步的，setState做了什么

异步。
每次的setState, 都触发react 中的reconciliation(协调器)来收集这个函数处理。
因为存储这个更新的是队列，所以在执行更新时，使用的this.state是原来的。

## React组件传值有哪些方式

- 通过props , 父传子
- 回调函数，父传给子，子调用回调传参数
- 通过context , 在根组件使用到createContext , 它的子的组件可以通过useContext 来使用.
  16的版本是对象来包裹。{Provier, Consumer}

## react如何做到和vue中keep-alive的缓存效果

- 使用的是react.createPortal, 传入不同的key , 里面有componentDidActivate
- componentWillUnactive,

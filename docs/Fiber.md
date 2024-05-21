## Fiber设计思想

Fiber 是对 React 核心算法的重构，facebook 团队使用两年多的时间去重构 React 的核心算法，在React16 以上的版本中引入了 Fiber 架构，其中的设计思想是非常值得我们学习的。

> 为什么需要Fiber

我们知道，在浏览器中，页面是一帧一帧绘制出来的，渲染的帧率与设备的刷新率保持一致。一般情况下，设备的屏幕刷新率为1s 60次，当每秒内绘制的帧数（FPS）超过60时，页面渲染是流畅的；而当FPS小于60时，会出现一定程度的卡顿现象。下面来看完整的一帧中，具体做了哪些事情：

1. 首先需要处理输入事件，能够让用户得到最早的反馈
2. 接下来是处理定时器，需要检查定时器是否到时间，并执行对应的回调
3. 接下来处理 Begin Frame（开始帧），即每一帧的事件，包括 window.resize、scroll、media query change 等
4. 接下来执行请求动画帧 requestAnimationFrame（rAF），即在每次绘制之前，会执行 rAF 回调
5. 紧接着进行 Layout 操作，包括计算布局和更新布局，即这个元素的样式是怎样的，它应该在页面如何展示
6. 接着进行 Paint 操作，得到树中每个节点的尺寸与位置等信息，浏览器针对每个元素进行内容填充
7. 到这时以上的六个阶段都已经完成了，接下来处于空闲阶段（Idle Peroid），可以在这时执行 requestIdleCallback 里注册的任务（后面会详细讲到这个 requestIdleCallback ，它是 React Fiber 实现的基础）

js引擎和页面渲染引擎是在同一个渲染线程之内，两者是互斥关系。如果在某个阶段执行任务特别长，例如在定时器阶段或Begin Frame阶段执行时间非常长，时间已经明显超过了16ms，那么就会阻塞页面的渲染，从而出现卡顿现象。

> 什么是Fiber

Fiber 可以理解为是一个执行单元，也可以理解为是一种数据结构。
一个执行单元.

- 首先 React 向浏览器请求调度，浏览器在一帧中如果还有空闲时间，会去判断是否存在待执行任务，不存在就直接将控制权交给浏览器，如果存在就会执行对应的任务，执行完成后会判断是否还有时间，有时间且有待执行任务则会继续执行下一个任务，否则就会将控制权交给浏览器。这里会有点绕，可以结合上述的图进行理解。

- Fiber 可以被理解为划分一个个更小的执行单元，它是把一个大任务拆分为了很多个小块任务，一个小块任务的执行必须是一次完成的，不能出现暂停，但是一个小块任务执行完后可以移交控制权给浏览器去响应用户，从而不用像之前一样要等那个大任务一直执行完成再去响应用户。

- 一种数据结构
Fiber 还可以理解为是一种数据结构，React Fiber 就是采用链表实现的。每个 Virtual DOM 都可以表示为一个 fiber，如下图所示，每个节点都是一个 fiber。一个 fiber包括了 child（第一个子节点）、sibling（兄弟节点）、return（父节点）等属性，React Fiber 机制的实现，就是依赖于以下的数据结构。在下文中会讲到基于这个链表结构，Fiber 究竟是如何实现的。

## Fiber节点如何产生的

Fiber树的产生源自于js模板，如以下代码：

假设我们有一个简单的 React 函数组件 `ExampleComponent`：
```js
import react from 'React';

function ExampleComponent() {
  return (
    <div className="container">
      <h1>Hello, World!</h1>
      <p>This is an example component.</p>
    </div>
  );
}
```

举例webpack, 打包来说.webpack打包文件中使用配置了这几个配置, 
- "@babel/core"
- "@babel/preset-env"
- "@babel/preset-react"

能把函数中返回的内容转换为

```js
import react from 'React';

function ExampleComponent() {
  return React.createElement(
    'div',
    { className: 'container' },
    React.createElement('h1', null, 'Hello, World!'),
    React.createElement('p', null, 'This is an example component.')
  );
}
```

> React.createElement做了什么事情呢?

React.createElement() 是 React 中用于创建虚拟 DOM 元素的函数。它接收三个参数：类型、属性对象和子元素，然后返回一个描述该元素的 JavaScript 对象

```js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child =>
        typeof child === "object" ? child : createTextElement(child)
      )
    }
  };
}

function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: []
    }
  };
}

```

上述代码是一个简化版的示例，实际的 React.createElement() 实现还包含其他功能，例如处理 key 和 ref 等特殊属性，以及一些性能优化。以下是编译后的产物，可以认为是vNode

```js
{
  "type": "div",
  "props": {
    "className": "container",
    "children": [{
      "type": "h1",
      "props": {
        "children": [{
          "type": "TEXT_ELEMENT",
          "props": {
            "nodeValue": "Hello, World!",
            "children": []
          }
        }]
      }
    }, {
      "type": "p",
      "props": {
        "children": [{
          "type": "TEXT_ELEMENT",
          "props": {
            "nodeValue": "This is an example component.",
            "children": []
          }
        }]
      }
    }]
  }
}
```

React生成Fiber树的过程是由`调度器`和`协调器`共同完成的
(Fiber节点并不是通过递归函数创建)
简单来说就是在第一次挂载的时候和更新的时候生成新的Fiber

```js
// 定义 Fiber 对象
class Fiber {
  constructor(vNode) {
    this.type = vNode.type;
    this.props = vNode.props;
    this.key = vNode.key;
    this.child = null;
    this.sibling = null;
    this.alternate = null;
    // 其他需要用到的属性和状态
  }
}

// 构建 Fiber 树
function createFiberTree(vNode) {
  const fiber = new Fiber(vNode);

  if (vNode.children) {
    let prevChildFiber = null;
    vNode.children.forEach(childVNode => {
      const childFiber = createFiberTree(childVNode);
      if (prevChildFiber === null) {
        fiber.child = childFiber;
      } else {
        prevChildFiber.sibling = childFiber;
      }
      prevChildFiber = childFiber;
    });
  }

  return fiber;
}


// 将 vNode 转化为 Fiber 结构
const fiberTree = createFiberTree(vNode);
```

createFiberTree 函数用于递归构建 Fiber 树，根据 vNode 的层次结构创建相应的 Fiber 对象，并将子节点和兄弟节点连接起来。


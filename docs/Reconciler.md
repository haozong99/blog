## Reconciler的作用

`Reconciler`是react中运行时的模块，根据jsx产生的element,调用宿主环境中的api,最终显示到ui界面

Reconciler操作element和Fiber
已知react jsx 会产出element,结构如下：

```js
const ReactElement = function (
	type: Type,
	key: Key,
	ref: Ref,
	props: Props,
): ReactElement {
    const element = {
        $$typeof: REACT_ELEMENT_TYPE,
        type,
        key,
        ref,
        props,
        _mark: 'wy set',
    };
    return element;
};
```

element除了和子element之间有关联关系，和别的element之间的关系难以获知,且不便于扩展。

> fiberNode关联节点,结构如下

fiberNode通过return, sibling, child 关联父，兄弟，孩子fiberNode,通过flags标价增删改等副作用

```js
// react-source-learn/packages/react-Reconciler/src/ReactFiber.ts
import { Key, Ref, Props } from 'shared/ReactTypes';
import { WorkTag } from './ReactWorkTags';
import { Flags, NoFlags } from './ReactFiberFlags';
export class FiberNode {
	tag: WorkTag; // fiberNode 是什么类型的节点
	ref: Ref;

	key: Key;
	stateNode: any; // 当前fiberNode对应的真实的dom 例如 div
	type: any; // 对应的element的函数， 例如functionComponent的函数

	return: FiberNode | null;
	sibling: FiberNode | null;
	child: FiberNode | null;
	alternate: FiberNode | null; // current 和 wip 之间的fiberNode相互指向
	index: number;

	pendingProps: Props; // 在fiberNode作为工作单元刚开始工作的时候的props
	memoizedProps: Props; // 在fiberNode作为工作单元结束工作的时候props

	flags: Flags; // 当前节点的操作类型 例如 插入，删除 flags 被统称为副作用

	constructor(tag: WorkTag, pendingProps: Props, key: Key) {
		this.tag = tag;
		this.ref = null;
		this.key = key;
		this.stateNode = null;
		this.type = null;

		this.return = null;
		this.sibling = null;
		this.child = null;
		this.index = 0;

		this.alternate = null;

		this.pendingProps = pendingProps;
		this.memoizedProps = null;

		this.flags = NoFlags;
	}
}
```

## Reconciler工作方式

Reconciler工作方式是比较reactElement和fiberNode，来创建fiberNode或者产生子fiberNode

- 当reactElement存在，fiberNode为null，就创建当前reactElement对应的fiberNode
- 当reactElement，fiberNode都存在，根据reactElement，fiberNode的差异，做增删改

reactElement，fiberNode比较之后，会产生current fiber tree 和workInProgress fiber Tree

- current: 真实ui对应的fiber tree
- workInProgress：更新时，在后台Reconciler中计算产生的workInProgress

当前workInProgress完成之后，current和workInProgress立即相互交换，这个过程称为双缓冲

## Reconciler访问reactElement方式

Reconciler中访问reactElement采用深度优先遍历的方式，这是一个递归的过程。Reconciler把这个过程分为“递（beginWork）”和“归（completeWork）”

> 递归的操作伪代码

```js
// react-source-learn/packages/react-Reconciler/src/ReactFiberWorkLoop.ts
import { FiberNode } from './ReactFiber';
import { beginWork } from './ReactFiberBeginWork';
import { completeWork } from './ReactFiberCompleteWork';
let workInProgressRoot: FiberNode | null; // 正在被执行的fiberNode

function prepareFreshStack(root: FiberNode) {
	workInProgressRoot = root;
}

function renderRoot(root: FiberNode) {
	prepareFreshStack(root);
	do {
		try {
			workLoop();
		} catch (e) {
			console.warn('workLoop出错', e);
			workInProgressRoot = null;
		}
	} while (true);
}

function workLoop() {
	while (workInProgressRoot !== null) {
		performUnitOfWork(workInProgressRoot);
	}
}

// 递操作
function performUnitOfWork(fiber: FiberNode) {
	const next = beginWork(fiber); // 子fiberNode
	if (next !== null) {
		workInProgressRoot = next;
	} else {
		completeUnitWork(fiber);
	}
}

// 归操作
function completeUnitWork(fiber: FiberNode) {
	let node: FiberNode | null = fiber;
	do {
		completeWork(node);
		const sibling = node.sibling;
		if (sibling !== null) {
			workInProgressRoot = sibling;
			return;
		}

		node = node.return;
		workInProgressRoot = node;
	} while (node !== null);
}
```
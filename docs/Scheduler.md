## Scheduler是什么

Scheduler是一个任务调度器，它会根据任务的优先级对任务进行调用执行。
在有多个任务的情况下，它会先执行优先级高的任务。如果一个任务执行的时间过长，Scheduler会中断当前任务，让出线程的执行权，避免造成用户操作时界面的卡顿。在下一次恢复未完成的任务的执行。

Reac通过下面的代码让Fiber树的构建进入调度流程：

```js
   function ensureRootIsScheduled(root: FiberRoot, currentTime: number){
       ...
       let schedulerPriorityLevel;
       // 通过lanesToEventPriority函数将lane优先级转化为Scheduler优先级
       switch (lanesToEventPriority(nextLanes)) {
          case DiscreteEventPriority:
            schedulerPriorityLevel = ImmediateSchedulerPriority;
            break;
          case ContinuousEventPriority:
            schedulerPriorityLevel = UserBlockingSchedulerPriority;
            break;
          case DefaultEventPriority:
            schedulerPriorityLevel = NormalSchedulerPriority;
            break;
          case IdleEventPriority:
            schedulerPriorityLevel = IdleSchedulerPriority;
            break;
          default:
            schedulerPriorityLevel = NormalSchedulerPriority;
            break;
        }
        //将react与scheduler连接，将react产生的事件作为任务使用scheduler调度
        newCallbackNode = scheduleCallback(
          schedulerPriorityLevel,
          performConcurrentWorkOnRoot.bind(null, root),
        );
   }
```

为什么这里需要做一次优先级的转换呢？因为React和Scheduler都是相对独立的，它们自己内部都有自己的一套优先级机制，所以当React产生的事件需要被Scheduler调度时，需要将React的事件优先级转换为Scheduler的调度优先级。

## 调度入口-scheduleCallback

接下来我们点进去查看scheduleCallback内部代码：

```js
function scheduleCallback(priorityLevel, callback) {
  ...
  return Scheduler_scheduleCallback(priorityLevel, callback);
}

function unstable_scheduleCallback(priorityLevel, callback, options) {
 ......
}
```

这个方法就是react与Scheduler连接的函数。

下面我们详细分析一下Sheduler的基本配置：

> Scheduler中的优先级

```js
export const NoPriority = 0; //没有优先级
export const ImmediatePriority = 1; // 立即执行任务的优先级，级别最高
export const UserBlockingPriority = 2; // 用户阻塞的优先级
export const NormalPriority = 3; // 正常优先级
export const LowPriority = 4; // 较低的优先级
export const IdlePriority = 5; // 优先级最低，表示任务可以闲置（在没有任务执行的时候，才会执行闲置的任务）

```

> Scheduler中的任务管理队列 

Scheduler中有两个任务队列：timerQueue 和 taskQueue。 timerQueue 和 taskQueue都是最小堆的数据结构。

1. timerQueue：所有没有过期的任务会放在这个队列中。
2. taskQueue：所有过期的任务会放在该队列中，并且按过期时间排序，过期时间越小则排在越前面，并且越先执行。

当Scheduler开始地调度任务执行时，首先会从taskQueue过期任务队列中获取任务执行，一个任务执行完成则会从taskQueue中弹出，当taskQueue中所有的任务都执行完成了，那么则会去timerQueue中检查是否有过期的任务，有的话则会拿出放到taskQueue中去执行。

```js
function unstable_scheduleCallback(priorityLevel, callback, options) {
  var currentTime = getCurrentTime(); //当前时间

  var startTime; //任务开始执行的时间
  if (typeof options === 'object' && options !== null) {
    var delay = options.delay;
    if (typeof delay === 'number' && delay > 0) {
      startTime = currentTime + delay;
    } else {
      startTime = currentTime;
    }
  } else {
    startTime = currentTime;
  }

  var timeout; //任务延时的时间

  // 根据任务的优先级，给定任务的超时时间
  // 优先级越高超时时间越小，反之越大
  switch (priorityLevel) {
    case ImmediatePriority:
      timeout = IMMEDIATE_PRIORITY_TIMEOUT;
      break;
    case UserBlockingPriority:
      timeout = USER_BLOCKING_PRIORITY_TIMEOUT;
      break;
    case IdlePriority:
      timeout = IDLE_PRIORITY_TIMEOUT;
      break;
    case LowPriority:
      timeout = LOW_PRIORITY_TIMEOUT;
      break;
    case NormalPriority:
    default:
      timeout = NORMAL_PRIORITY_TIMEOUT;
      break;
  }

  var expirationTime = startTime + timeout; //任务过期时间

  //以react产生的事件来创建一个新的任务
  var newTask = {
    id: taskIdCounter++,
    callback, // callback = performConcurrentWorkOnRoot
    priorityLevel,
    startTime,
    expirationTime,
    sortIndex: -1,
  };
  if (enableProfiling) {
    newTask.isQueued = false;
  }

  //scheduler有两个任务队列：timerQueue 和 taskQueue
  //timerQueue中存放延时任务，也就是未过期的任务
  //taskQueue中存放的是过期任务，也就是需要立即执行的任务
  //timerQueue 和 taskQueue是一个最小堆的数据结构

  //对任务开始时间与当前时间进行比较
  //任务开始时间大于当前时间，表示当前任务是一个延时任务
  if (startTime > currentTime) {
    // This is a delayed task.
    //将开始时间作为排序id，越小排在越靠前
    newTask.sortIndex = startTime;
    //将新建的任务添加进延时任务队列中
    push(timerQueue, newTask);

    //当过期任务队列中执行完所有的任务，
    //则需要不断遍历延时队列中的任务，一旦有任务过期则需要立即添加到过期任务队列中进行执行
    if (peek(taskQueue) === null && newTask === peek(timerQueue)) {
      // All tasks are delayed, and this is the task with the earliest delay.
      //当前是否有requestHostTimeout（就是一个setTimeout）正在执行，有的话则停止，避免多个requestHostTimeout一起运行，造成资源的不必要浪费
      //重新调用requestHostTimeout检查延时队列中是否有过期任务
      if (isHostTimeoutScheduled) {
        // Cancel an existing timeout.
        cancelHostTimeout();
      } else {
        isHostTimeoutScheduled = true;
      }
      // Schedule a timeout.
      // 创建一个timeout作为调度者
      requestHostTimeout(handleTimeout, startTime - currentTime);
    }
  } else {
    //将过期时间作为排序id，越小排在越靠前
    newTask.sortIndex = expirationTime;
    //将新建的任务添加进过期任务队列中
    push(taskQueue, newTask);
    if (enableProfiling) {
      markTaskStart(newTask, currentTime);
      newTask.isQueued = true;
    }
    // Schedule a host callback, if needed. If we're already performing work,
    // wait until the next time we yield.
    //判断是都已有Scheduled正在调度任务
    //没有的话则创建一个调度者开始调度任务，有的话则直接使用上一个调度者调度任务
    if (!isHostCallbackScheduled && !isPerformingWork) {
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);
    }
  }

  return newTask;
}
```

scheduleCallback中主要是创建一个新的任务，并且根据任务的开始时间来判断任务是否过期，针对未过期的任务则会添加到timerQueue中，使用startTimer做为排序的依据。如果taskQueue中任务全部执行完成，则会调用requestHostTimeout，实际上这个函数是创建了一个setTimeout，把第一个任务的超时时间作为setTimeout的时间间隔调用handleTimeout。
那么handleTimeout中又做了哪些事情，我们来看下源码：


```js 
function handleTimeout(currentTime) {
  isHostTimeoutScheduled = false;

  //检查延时任务队列中是否有已过期的任务
  //有的话则将过期任务拿出添加到过期任务队列中进行执行
  advanceTimers(currentTime);

  //isHostCallbackScheduled判断是否已经发起过调度
  //如果当前没有正在执行的调度，则会创建一个调度去执行任务
  if (!isHostCallbackScheduled) {
    if (peek(taskQueue) !== null) {
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);
    } else {
      const firstTimer = peek(timerQueue);
      if (firstTimer !== null) {
        requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
      }
    }
  }
}
```

handleTimeout中主要是检查timerQueue中是否有已过期的任务，有的话则会将已过期的任务添加到taskQueue中去执行。这项工作主要是advanceTimers这个函数去来实现的：

```js 
function advanceTimers(currentTime) {
  //检查延时任务队列中是否有已过期的任务
  //有的话则将过期任务拿出添加到过期任务队列中进行执行
  // Check for tasks that are no longer delayed and add them to the queue.
  let timer = peek(timerQueue);
  while (timer !== null) {
    if (timer.callback === null) {
      // Timer was cancelled.
      pop(timerQueue);
    } else if (timer.startTime <= currentTime) {
      // Timer fired. Transfer to the task queue.
      pop(timerQueue);
      timer.sortIndex = timer.expirationTime;
      push(taskQueue, timer);
      if (enableProfiling) {
        markTaskStart(timer, currentTime);
        timer.isQueued = true;
      }
    } else {
      // Remaining timers are pending.
      return;
    }
    timer = peek(timerQueue);
  }
}
```

针对过期的任务，则会将过期时间作为排序依据，然后调用requestHostCallback函数创建调度者开始调度流程。

```js
if (!isHostCallbackScheduled && !isPerformingWork) {
  isHostCallbackScheduled = true;
  requestHostCallback(flushWork);
}

```

创建调度者-requestHostCallback

```js
function requestHostCallback(callback) {
  scheduledHostCallback = callback;
  if (!isMessageLoopRunning) {
    isMessageLoopRunning = true;
    schedulePerformWorkUntilDeadline();
  }
}
```

这里我们先记住callback是调用requestHostCallback传入的flushWork函数，会在后面调用。 schedulePerformWorkUntilDeadline则是创建调度者真正的函数，我们来看下它的实现：

```js
let schedulePerformWorkUntilDeadline;
if (typeof localSetImmediate === 'function') {
  // 使用setImmediate的主要原因是因为在服务端渲染，MessageChannel会阻止nodejs的进程退出
  schedulePerformWorkUntilDeadline = () => {
    localSetImmediate(performWorkUntilDeadline);
  };
} else if (typeof MessageChannel !== 'undefined') {
  // 使用MessageChannel的原因是因为
  // setTimeout如果嵌套的层级超过了 5 层，并且 timeout 小于 4ms，则设置 timeout 为 4ms。
  const channel = new MessageChannel();
  const port = channel.port2;
  channel.port1.onmessage = performWorkUntilDeadline;
  schedulePerformWorkUntilDeadline = () => {
    port.postMessage(null);
  };
} else {
  //在以上方案都不能实现的时候，则降级使用setTimeout来实现创建调度者
  schedulePerformWorkUntilDeadline = () => {
    localSetTimeout(performWorkUntilDeadline, 0);
  };
}
```
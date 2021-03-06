# React Fiber 是如何实现更新过程可控的

> [https://www.zoo.team/article/about-react-fiber](https://www.zoo.team/article/about-react-fiber)

## 前言

从 React 16 开始，React 采用了 Fiber 机制替代了原先基于原生执行栈递归遍历 VDOM 的方案，提高了页面渲染性能和用户体验。乍一听 Fiber 好像挺神秘，在原生执行栈都还没搞懂的情况下，又整出个 Fiber，还能不能愉快的写代码了。别慌，老铁！下面就来唠唠关于 Fiber 那点事儿。

## 什么是 Fiber

Fiber 的英文含义是“纤维”，它是比线程（Thread）更细的线，比线程（Thread）控制得更精密的执行模型。在广义计算机科学概念中，Fiber 又是一种协作的（Cooperative）编程模型，帮助开发者用一种【既模块化又协作化】的方式来编排代码。

![图片](https://www.zoo.team/images/upload/upload_e39013db1440289d93a15bd960d74359.jpeg)

简单点说，Fiber 就是 React 16 实现的一套新的更新机制，让 React 的更新过程变得可控，避免了之前一竿子递归到底影响性能的做法。

## 关于 Fiber 你需要知道的基础知识

### **1 浏览器刷新率（帧）**

页面的内容都是一帧一帧绘制出来的，浏览器刷新率代表浏览器一秒绘制多少帧。目前浏览器大多是 60Hz（60帧/s），每一帧耗时也就是在 16ms 左右。原则上说 1s 内绘制的帧数也多，画面表现就也细腻。那么在这一帧的（16ms） 过程中浏览器又干了啥呢？

![图片](https://www.zoo.team/images/upload/upload_fa4bed183d5043a2f350fde149d2a9d2.png)

通过上面这张图可以清楚的知道，浏览器一帧会经过下面这几个过程：

1. 接受输入事件
2. 执行事件回调
3. 开始一帧
4. 执行 RAF (RequestAnimationFrame)
5. 页面布局，样式计算
6. 渲染
7. 执行 RIC (RequestIdelCallback)

第七步的 RIC 事件不是每一帧结束都会执行，只有在一帧的 16ms 中做完了前面 6 件事儿且还有剩余时间，才会执行。这里提一下，如果一帧执行结束后还有时间执行 RIC 事件，那么下一帧需要在事件执行结束才能继续渲染，所以 RIC 执行不要超过 30ms，如果长时间不将控制权交还给浏览器，会影响下一帧的渲染，导致页面出现卡顿和事件响应不及时。

### **2. JS 原生执行栈**

React Fiber 出现之前，React 通过原生执行栈递归遍历 VDOM。当浏览器引擎第一次遇到 JS 代码时，会产生一个全局执行上下文并将其压入执行栈，接下来每遇到一个函数调用，又会往栈中压入一个新的上下文。比如：

```javascript
function A(){
  B();
  C();
}
function B(){}
function C(){}
A();
```

引擎在执行的时候，会形成如下这样的执行栈： ![图片](https://www.zoo.team/images/upload/upload_b8b8ceab13185b05aea8bbbded36852d.jpg)

浏览器引擎会从执行栈的顶端开始执行，执行完毕就弹出当前执行上下文，开始执行下一个函数，直到执行栈被清空才会停止。然后将执行权交还给浏览器。由于 React 将页面视图视作一个个函数执行的结果。每一个页面往往由多个视图组成，这就意味着多个函数的调用。

如果一个页面足够复杂，形成的函数调用栈就会很深。每一次更新，执行栈需要一次性执行完成，中途不能干其他的事儿，只能"一心一意"。结合前面提到的浏览器刷新率，JS 一直执行，浏览器得不到控制权，就不能及时开始下一帧的绘制。如果这个时间超过 16ms，当页面有动画效果需求时，动画因为浏览器不能及时绘制下一帧，这时动画就会出现卡顿。不仅如此，因为事件响应代码是在每一帧开始的时候执行，如果不能及时绘制下一帧，事件响应也会延迟。

### **3. 时间分片（Time Slicing）**

时间分片指的是一种将多个粒度小的任务放入一个时间切片（一帧）中执行的一种方案，在 React Fiber 中就是将多个任务放在了一个时间片中去执行。

### 4. 链表

在 React Fiber 中用链表遍历的方式替代了 React 16 之前的栈递归方案。在 React 16 中使用了大量的链表。例如：

- 使用多向链表的形式替代了原来的树结构

例如下面这个组件：

```html
<div id="id">
  A1
  <div id="B1">
    B1
     <div id="C1"></div>
  </div>
  <div id="B2">
      B2
  </div>
</div>
```

会使用下面这样的链表表示： ![图片](https://www.zoo.team/images/upload/upload_01f79abb306a7f1a9291bcf78d14297b.jpg)

- 副作用单链表

![图片](https://www.zoo.team/images/upload/upload_7fa2aa387b906771c58c120dba52d295.jpg)

- 状态更新单链表

![图片](https://www.zoo.team/images/upload/upload_a154f8d85a6764de892ad2fc25d55756.jpg)

- ...

链表是一种简单高效的数据结构，它在当前节点中保存着指向下一个节点的指针，就好像火车一样一节连着一节

![图片](https://www.zoo.team/images/upload/upload_03f29c3bedd7df6bb616f2c7142fe492.png)

遍历的时候，通过操作指针找到下一个元素。但是操作指针时（调整顺序和指向）一定要小心。

链表相比顺序结构数据格式的好处就是：

1. 操作更高效，比如顺序调整、删除，只需要改变节点的指针指向就好了。
2. 不仅可以根据当前节点找到下一个节点，在多向链表中，还可以找到他的父节点或者兄弟节点。

但链表也不是完美的，缺点就是：

1. 比顺序结构数据更占用空间，因为每个节点对象还保存有指向下一个对象的指针。
2. 不能自由读取，必须找到他的上一个节点。

React 用空间换时间，更高效的操作可以方便根据优先级进行操作。同时可以根据当前节点找到其他节点，在下面提到的挂起和恢复过程中起到了关键作用。

## React Fiber 是如何实现更新过程可控？

前面讲完基本知识，现在正式开始介绍今天的主角 Fiber，看看 React Fiber 是如何实现对更新过程的管控。

![图片](https://www.zoo.team/images/upload/upload_6462f050728d2493d46faa3102c80974.jpeg)

更新过程的可控主要体现在下面几个方面：

1. 任务拆分
2. 任务挂起、恢复、终止
3. 任务具备优先级

### **1. 任务拆分**

前面提到，React Fiber 之前是基于原生执行栈，每一次更新操作会一直占用主线程，直到更新完成。这可能会导致事件响应延迟，动画卡顿等现象。

在 React Fiber 机制中，它采用"化整为零"的战术，将调和阶段（Reconciler）递归遍历 VDOM 这个大任务分成若干小任务，每个任务只负责一个节点的处理。例如：

```javascript
import React from "react";
import ReactDom from "react-dom"
const jsx = (
    <div id="A1">
    A1
    <div id="B1">
      B1
      <div id="C1">C1</div>
      <div id="C2">C2</div>
    </div>
    <div id="B2">B2</div>
  </div>
)
ReactDom.render(jsx,document.getElementById("root"))
```

这个组件在渲染的时候会被分成八个小任务，每个任务用来分别处理 A1(div)、A1(text)、B1(div)、B1(text)、C1(div)、C1(text)、C2(div)、C2(text)、B2(div)、B2(text)。再通过时间分片，在一个时间片中执行一个或者多个任务。这里提一下，所有的小任务并不是一次性被切分完成，而是处理当前任务的时候生成下一个任务，如果没有下一个任务生成了，就代表本次渲染的 Diff 操作完成。

### **2. 挂起、恢复、终止**

再说挂起、恢复、终止之前，不得不提两棵 Fiber 树，workInProgress tree 和 currentFiber tree。

workInProgress 代表当前正在执行更新的 Fiber 树。在 render 或者 setState 后，会构建一颗 Fiber 树，也就是 workInProgress tree，这棵树在构建每一个节点的时候会收集当前节点的副作用，整棵树构建完成后，会形成一条完整的副作用链。

currentFiber 表示上次渲染构建的 Filber 树。在每一次更新完成后 workInProgress 会赋值给 currentFiber。在新一轮更新时 workInProgress tree 再重新构建，新 workInProgress 的节点通过 alternate 属性和 currentFiber 的节点建立联系。

在新 workInProgress tree 的创建过程中，会同 currentFiber 的对应节点进行 Diff 比较，收集副作用。同时也会复用和 currentFiber 对应的节点对象，减少新创建对象带来的开销。也就是说无论是创建还是更新，挂起、恢复以及终止操作都是发生在 workInProgress tree 创建过程中。workInProgress tree 构建过程其实就是循环的执行任务和创建下一个任务，大致过程如下：

![图片](https://www.zoo.team/images/upload/upload_777fdfe8d50d7a355115c2c2117d7c93.png)

当没有下一个任务需要执行的时候，workInProgress tree 构建完成，开始进入提交阶段，完成真实 DOM 更新。

在构建 workInProgressFiber tree 过程中可以通过挂起、恢复和终止任务，实现对更新过程的管控。下面简化了一下源码，大致实现如下：

```javascript
let nextUnitWork = null;//下一个执行单元
//开始调度
function shceduler(task){
     nextUnitWork = task; 
}
//循环执行工作
function workLoop(deadline){
  let shouldYield = false;//是否要让出时间片交出控制权
  while(nextUnitWork && !shouldYield){
    nextUnitWork = performUnitWork(nextUnitWork)
    shouldYield = deadline.timeRemaining()<1 // 没有时间了，检出控制权给浏览器
  }
  if(!nextUnitWork) {
    conosle.log("所有任务完成")
    //commitRoot() //提交更新视图
  }
  // 如果还有任务，但是交出控制权后,请求下次调度
  requestIdleCallback(workLoop,{timeout:5000}) 
}
/*
 * 处理一个小任务，其实就是一个 Fiber 节点，如果还有任务就返回下一个需要处理的任务，没有就代表整个
 */
function performUnitWork(currentFiber){
  ....
  return FiberNode
}
```

#### 挂起

当第一个小任务完成后，先判断这一帧是否还有空闲时间，没有就挂起下一个任务的执行，记住当前挂起的节点，让出控制权给浏览器执行更高优先级的任务。

#### 恢复

在浏览器渲染完一帧后，判断当前帧是否有剩余时间，如果有就恢复执行之前挂起的任务。如果没有任务需要处理，代表调和阶段完成，可以开始进入渲染阶段。这样完美的解决了调和过程一直占用主线程的问题。

那么问题来了他是如何判断一帧是否有空闲时间的呢？答案就是我们前面提到的 RIC (RequestIdleCallback) 浏览器原生 API，React 源码中为了兼容低版本的浏览器，对该方法进行了 Polyfill。

当恢复执行的时候又是如何知道下一个任务是什么呢？答案在前面提到的链表。在 React Fiber 中每个任务其实就是在处理一个 FiberNode 对象，然后又生成下一个任务需要处理的 FiberNode。顺便提一嘴，这里提到的FiberNode 是一种数据格式，下面是它没有开美颜的样子：

```javascript
class FiberNode {
  constructor(tag, pendingProps, key, mode) {
    // 实例属性
    this.tag = tag; // 标记不同组件类型，如函数组件、类组件、文本、原生组件...
    this.key = key; // react 元素上的 key 就是 jsx 上写的那个 key ，也就是最终 ReactElement 上的
    this.elementType = null; // createElement的第一个参数，ReactElement 上的 type
    this.type = null; // 表示fiber的真实类型 ，elementType 基本一样，在使用了懒加载之类的功能时可能会不一样
    this.stateNode = null; // 实例对象，比如 class 组件 new 完后就挂载在这个属性上面，如果是RootFiber，那么它上面挂的是 FiberRoot,如果是原生节点就是 dom 对象
    // fiber
    this.return = null; // 父节点，指向上一个 fiber
    this.child = null; // 子节点，指向自身下面的第一个 fiber
    this.sibling = null; // 兄弟组件, 指向一个兄弟节点
    this.index = 0; //  一般如果没有兄弟节点的话是0 当某个父节点下的子节点是数组类型的时候会给每个子节点一个 index，index 和 key 要一起做 diff
    this.ref = null; // reactElement 上的 ref 属性
    this.pendingProps = pendingProps; // 新的 props
    this.memoizedProps = null; // 旧的 props
    this.updateQueue = null; // fiber 上的更新队列执行一次 setState 就会往这个属性上挂一个新的更新, 每条更新最终会形成一个链表结构，最后做批量更新
    this.memoizedState = null; // 对应  memoizedProps，上次渲染的 state，相当于当前的 state，理解成 prev 和 next 的关系
    this.mode = mode; // 表示当前组件下的子组件的渲染方式
    // effects
    this.effectTag = NoEffect; // 表示当前 fiber 要进行何种更新
    this.nextEffect = null; // 指向下个需要更新的fiber
    this.firstEffect = null; // 指向所有子节点里，需要更新的 fiber 里的第一个
    this.lastEffect = null; // 指向所有子节点中需要更新的 fiber 的最后一个
    this.expirationTime = NoWork; // 过期时间，代表任务在未来的哪个时间点应该被完成
    this.childExpirationTime = NoWork; // child 过期时间
    this.alternate = null; // current 树和 workInprogress 树之间的相互引用
  }
}
```

额…看着好像有点上头，这是开了美颜的样子：

![图片](https://www.zoo.team/images/upload/upload_d174c0ca815186a390c31c5b99c097f5.jpg)

是不是好看多了？在每次循环的时候，找到下一个执行需要处理的节点。

```javascript
function performUnitWork(currentFiber){
    //beginWork(currentFiber) //找到儿子，并通过链表的方式挂到currentFiber上，每一偶儿子就找后面那个兄弟
  //有儿子就返回儿子
  if(currentFiber.child){
    return currentFiber.child;
  } 
  //如果没有儿子，则找弟弟
  while(currentFiber){//一直往上找
    //completeUnitWork(currentFiber);//将自己的副作用挂到父节点去
    if(currentFiber.sibling){
      return currentFiber.sibling
    }
    currentFiber = currentFiber.return;
  }
}
```

在一次任务结束后返回该处理节点的子节点或兄弟节点或父节点。只要有节点返回，说明还有下一个任务，下一个任务的处理对象就是返回的节点。通过一个全局变量记住当前任务节点，当浏览器再次空闲的时候，通过这个全局变量，找到它的下一个任务需要处理的节点恢复执行。就这样一直循环下去，直到没有需要处理的节点返回，代表所有任务执行完成。最后大家手拉手，就形成了一颗 Fiber 树。

![图片](https://www.zoo.team/images/upload/upload_23b508e1c8ca82420a8fcd7a6cc9b580.jpg)

#### 终止

其实并不是每次更新都会走到提交阶段。当在调和过程中触发了新的更新，在执行下一个任务的时候，判断是否有优先级更高的执行任务，如果有就终止原来将要执行的任务，开始新的 workInProgressFiber 树构建过程，开始新的更新流程。这样可以避免重复更新操作。这也是在 React 16 以后生命周期函数 componentWillMount 有可能会执行多次的原因。

### **3. 任务具备优先级**

React Fiber 除了通过挂起，恢复和终止来控制更新外，还给每个任务分配了优先级。具体点就是在创建或者更新 FiberNode 的时候，通过算法给每个任务分配一个到期时间（expirationTime）。在每个任务执行的时候除了判断剩余时间，如果当前处理节点已经过期，那么无论现在是否有空闲时间都必须执行改任务。

![图片](https://www.zoo.team/images/upload/upload_9645b2801ae5be8cb989086b23ac6a82.jpeg)

同时过期时间的大小还代表着任务的优先级。

任务在执行过程中顺便收集了每个 FiberNode 的副作用，将有副作用的节点通过 firstEffect、lastEffect、nextEffect 形成一条副作用单链表 AI(TEXT)-B1(TEXT)-C1(TEXT)-C1-C2(TEXT)-C2-B1-B2(TEXT)-B2-A。

![图片](https://www.zoo.team/images/upload/upload_42558caf76119a7ca2b465b43ba8d12b.png)

其实最终都是为了收集到这条副作用链表，有了它，在接下来的渲染阶段就通过遍历副作用链完成 DOM 更新。这里需要注意，更新真实 DOM 的这个动作是一气呵成的，不能中断，不然会造成视觉上的不连贯。

## 关于 React Fiber 的思考

### **1. 能否使用生成器（generater）替代链表**

在 Fiber 机制中，最重要的一点就是需要实现挂起和恢复，从实现角度来说 generator 也可以实现。那么为什么官方没有使用 generator 呢？猜测应该是是性能方面的原因。生成器不仅让您在堆栈的中间让步，还必须把每个函数包装在一个生成器中。一方面增加了许多语法方面的开销，另外还增加了任何现有实现的运行时开销。性能上远没有链表的方式好，而且链表不需要考虑浏览器兼容性。

### **2. Vue 是否会采用 Fiber 机制来优化复杂页面的更新**

这个问题其实有点搞事情，如果 Vue 真这么做了是不是就是变相承认 Vue 是在"集成" Angular 和 React 的优点呢？React 有 Fiber，Vue 就一定要有？

两者虽然都依赖 DOM Diff，但是实现上且有区别，DOM Diff 的目的都是收集副作用。Vue 通过 Watcher 实现了依赖收集，本身就是一种很好的优化。所以 Vue 没有采用 Fiber 机制，也无伤大雅。

## 总结

React Fiber 的出现相当于是在更新过程中引进了一个中场指挥官，负责掌控更新过程，足球世界里管这叫前腰。抛开带来的性能和效率提升外，这种“化整为零”和任务编排的思想，可以应用到我们平时的架构设计中。
# react 中 key 值得使用http://www.conardli.top/blog/article/React深入系列/React中key的正确使用方式.html#_1-为什么要使用key

> [http://www.conardli.top/blog/article/React 深入系列/React 中 key 的正确使用方式.html#\_1-为什么要使用 key](http://www.conardli.top/blog/article/React深入系列/React中key的正确使用方式.html#_1-为什么要使用key)

![image](https://lsqimg-1257917459.cos-website.ap-beijing.myqcloud.com/key1.png)

在开发 react 程序时我们经常会遇到这样的警告，然后就会想到：哦！循环子组件忘记加 key 了～

出于方便，有时候会不假思索的使用循环的索引作为 key，但是这样真的好吗？什么样的值才是 key 的最佳选择？

为了弄明白，本文将从三个方面来分析"key"：

1.为什么要使用 key

2.使用 index 做 key 存在的问题

3.正确的选择 key

## 1.为什么要使用 key

react 官方文档是这样描述 key 的：

> Keys 可以在 DOM 中的某些元素被增加或删除的时候帮助 React 识别哪些元素发生了变化。因此你应当给数组中的每一个元素赋予一个确定的标识。

react 的 diff 算法是把 key 当成唯一 id 然后比对组件的 value 来确定是否需要更新的，所以如果没有 key，react 将不会知道该如何更新组件。

你不传 key 也能用是因为 react 检测到子组件没有 key 后，会默认将数组的索引作为 key。

react 根据 key 来决定是销毁重新创建组件还是更新组件，原则是：

- key 相同，组件有所变化，react 会只更新组件对应变化的属性。
- key 不同，组件会销毁之前的组件，将整个组件重新渲染。

## 2.使用 index 做 key 存在的问题

### 2.1 受控组件

单纯的展示组件比如 span，这些组件是受控组件，意味着他们的值将是我们给定好的。

如果子组件只是受控组件，使用 index 作为 key，可能表面上不会有什么问题，实际上性能会受很大的影响。例如下面的代码：

```html
// ['张三','李四','王五']=>
<ul>
  <li key="0">张三</li>
  <li key="1">李四</li>
  <li key="2">王五</li>
</ul>
// 数组重排 -> ['王五','张三','李四'] =>
<ul>
  <li key="0">王五</li>
  <li key="1">张三</li>
  <li key="2">李四</li>
</ul>
```

当元素数据源的顺序发生改变时，对应的：

key 为 0，1，2 的组件都发生了变化，三个子组件都会被重新渲染。（这里的重新渲染不是销毁，因为 key 还在）

相反，我们使用唯一 id 作为 key：

```html
// ['张三','李四','王五']=>
<ul>
  <li key="000">张三</li>
  <li key="111">李四</li>
  <li key="222">王五</li>
</ul>
// 数组重排 -> ['王五','张三','李四'] =>
<ul>
  <li key="222">王五</li>
  <li key="000">张三</li>
  <li key="111">李四</li>
</ul>
```

根据上面的更新原则，子组件的值和 key 均未发生变化，只是顺序发生改变，因此 react 只是将他们做了移动，并未重新渲染。

### 2.2 非受控组件

像 input 这样可以由用户任意改变值，不受我们控制的组件，在使用了 index 作为 key 时可能会发生问题，看如下的栗子：

子组件：

```js
  render() {
    return (
      <div>
        <p >值：{this.props.value}</p>
        <input />
      </div>
    );
  }
}
```

父组件

```js
{
  this.state.data.map((element, index) => {
    return <Child value={element} key={index} />;
  });
}
```

我们在前两个输入框分别输入对应的值：

![image](https://lsqimg-1257917459.cos-website.ap-beijing.myqcloud.com/key2.png)

然后在头部添加一个元素：

![image](https://lsqimg-1257917459.cos-website.ap-beijing.myqcloud.com/key3.png)

很明显，这个结果并不符合我们的预期，我们来分析一下发生了什么：

```html
<div key="0">
  <p>值：0</p>
  <input />
</div>
<div key="1">
  <p>值：1</p>
  <input />
</div>
<div key="2">
  <p>值：2</p>
  <input />
</div>
```

变化后：

```html
<div key="0">
  <p>值：5</p>
  <input />
</div>
<div key="1">
  <p>值：0</p>
  <input />
</div>
<div key="2">
  <p>值：1</p>
  <input />
</div>
<div key="3">
  <p>值：2</p>
  <input />
</div>
```

可以发现：key 0，1，2 并没有发生改变，根据规则，不会卸载组件，只会更新改变的属性。

react 只 diff 到了 p 标签内值的变化，而 input 框中的值并未发生改变，因此不会重新渲染，只更新的 p 标签的值。

当使用唯一 id 作为 key 后：

![image](https://lsqimg-1257917459.cos-website.ap-beijing.myqcloud.com/key5.png)

```html
<div key="000">
  <p>值：0</p>
  <input />
</div>
<div key="111">
  <p>值：1</p>
  <input />
</div>
<div key="222">
  <p>值：2</p>
  <input />
</div>
```

变化后：

```html
<div key="555">
  <p>值：5</p>
  <input />
</div>
<div key="000">
  <p>值：0</p>
  <input />
</div>
<div key="111">
  <p>值：1</p>
  <input />
</div>
<div key="222">
  <p>值：2</p>
  <input />
</div>
```

可以很明显的发现：key 为 111，222，333 的组件没有发生任何改变，react 不会更新他们，只是新插入了子组件 555，并改变了其他组件的位置。

## 3.正确的选择 key

### 3.1 纯展示

如果组件单纯的用于展示，不会发生其他变更，那么使用 index 或者其他任何不相同的值作为 key 是没有任何问题的，因为不会发生 diff，就不会用到 key。

### 3.2 推荐使用 index 的情况

并不是任何情况使用 index 作为 key 会有缺陷，比如如下情况：

你要分页渲染一个列表，每次点击翻页会重新渲染：

使用唯一 id：

```html
第一页
<ul>
  <li key="000">张三</li>
  <li key="111">李四</li>
  <li key="222">王五</li>
</ul>
第二页
<ul>
  <li key="333">张三三</li>
  <li key="444">李四四</li>
  <li key="555">王五五</li>
</ul>
```

翻页后，三条记录的 key 和组件都发生了改变，因此三个子组件都会被卸载然后重新渲染。

使用 index：

```html
第一页
<ul>
  <li key="0">张三</li>
  <li key="1">李四</li>
  <li key="2">王五</li>
</ul>
第二页
<ul>
  <li key="0">张三三</li>
  <li key="1">李四四</li>
  <li key="2">王五五</li>
</ul>
```

翻页后，key 不变，子组件值发生改变，组件并不会被卸载，只发生更新。

### 3.3 子组件可能发生变更/使用了非受控组件

大多数情况下，使用唯一 id 作为子组件的 key 是不会有任何问题的。

这个 id 一定要是唯一，并且稳定的，意思是这条记录对应的 id 一定是独一无二的，并且永远不会发生改变。

不推荐使用 math.random 或者其他的第三方库来生成唯一值作为 key。

因为当数据变更后，相同的数据的 key 也有可能会发生变化，从而重新渲染，引起不必要的性能浪费。

如果数据源不满足我们这样的需求，我们可以在渲染之前为数据源手动添加唯一 id，而不是在渲染时添加。

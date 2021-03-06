## 不要再滥用useMemo了！你应该重新思考Hooks memoization

> [https://mp.weixin.qq.com/s/YEs5nH4aOAxOPYuW8oVlBA](https://mp.weixin.qq.com/s/YEs5nH4aOAxOPYuW8oVlBA)

在使用 React Hooks 的过程中，作者发现过渡频繁的应用 useMemo 用会影响程序的性能。在本文中作者将与大家分享如何避免过度使用 useMemo，使程序远离性能问题。

经过观察总结，我发现在两种情况下 useMemo 要么没什么用，要么就是用得太多了，而且可能会影响应用程序的性能表现。

第一种情况很容易就能推断出来，但是第二种情况就比较隐蔽了，很容易被忽略。如果你在生产环境的应用程序中使用了 Hook，那么你就可能会在这两个场景中使用 useMemo Hook。

下面我就会谈一谈为什么这些 useMemo 没什么必要，甚至可能影响你的应用性能。此外我会教大家在这些场景中避免过度使用 useMemo 的方法。

我们开始吧。

不需要 useMemo 的情况

为了方便，我们把这两类场景分别称为狮子和变色龙。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/XIibZ0YbvibkVbAzbuKiaWfT5rWmdqxaQA5yhp6Vd6ASQickZuic0zu5LoNazNjFZD8JS9LvJOz3c8rceEpP5WFxNlQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

先不用纠结为什么这么叫，继续读下去就是。

当你撞上一头雄狮，你的第一反应就是撒丫子跑，不要成为狮子的盘中餐，然后活下来跟别人吹牛。这时候可没空思考那么多。

这就是场景 A。它们是狮子，你应该下意识地躲开它们。

但在谈论它们之前，我们先来看看更隐蔽的变色龙场景。

相同的引用和开销不大的操作

参考下面的示例组件：

```js
/**
  @param {number} page
  @param {string} type
**/
const myComponent({page, type}) {
  const resolvedValue = useMemo(() => {
     getResolvedValue(page, type)
  }, [page, type])

  return <ExpensiveComponent resolvedValue={resolvedValue}/>
}
```

如上所示，显然作者使用了 useMemo。这里他们的思路是，当对 resolvedValue 的引用出现更改时，他们不想重新渲染 ExpensiveComponent。

虽说这个担忧是正确的，但无论何时要用 useMemo 之前都应该考虑两个问题：

- 首先，传递给 useMemo 的函数开销大不大？在上面这个示例中就是要考虑 getResolvedValue 的开销大不大？

  JavaScript 数据类型的大多数方法都是优化过的，例如 Array.map、Object.getOwnPropertyNames() 等。如果你执行的操作开销不大（想想大 O 符号），那么你就不需要记住返回值。使用 useMemo 的成本可能会超过重新评估该函数的成本。

- 其次，给定相同的输入值时，对记忆（memoized）值的引用是否会发生变化？例如在上面的代码块中，如果 page 为 2，type 为“GET”，那么对 resolvedValue 的引用是否会变化？简单的回答是考虑 resolvedValue 变量的数据类型。如果 resolvedValue 是原始值（如字符串、数字、布尔值、空值、未定义或符号），则引用就不会变化。也就是说 ExpensiveComponent 不会被重新渲染。

修正过的代码如下：

```js
/**
  @param {number} page
  @param {string} type
**/
const MyComponent({page, type}) {
  const resolvedValue = getResolvedValue(page, type)
  return <ExpensiveComponent resolvedValue={resolvedValue}/>
}
```

如前所述，如果 resolvedValue 返回一个字符串之类的原始值，并且 getResolvedValue 这个操作的开销没那么大，那么这段代码就非常合理，效率够高了。

只要 page 和 type 是一样的，比如说没有 prop 更改，resolvedValue 的引用就会保持不变，只是返回的值不是原始值了（例如变成了对象或数组）。

记住这两个问题：要记住的函数开销很大吗，返回的值是原始值吗？每次都思考这两个问题的话，你就能随时判断使用 useMemo 是否合适。

出于多种原因需要记住默认状态

参考以下代码块：

```js
/**
  @param {number} page
  @param {string} type
**/
const myComponent({page, type}) {
  const defaultState = useMemo(() => ({
    fetched: someOperationValue(),
    type: type
  }), [type])

  const [state, setState] = useState(defaultState);
  return <ExpensiveComponent />
}
```

有人会觉得上面的代码没什么问题，但这里 useMemo 调用肯定是没什么意义的。

首先我们来试着理解一下这段代码背后的思想。作者的思路很不错。当 type prop 更改时他们需要新的 defaultState 对象，并且不希望在每次重新渲染时都引用 defaultState 对象。

虽说这些问题都很实际，但这种方法是错误的，违反了一个基本原则：useState 是不会在每次重新渲染时都重新初始化的，只有在组件重载时才会初始化。

传递给 useState 的参数改名为 INITIAL_STATE 更合理。它只在组件刚加载时计算（或触发）一次。

```js
useState(INITIAL_STATE)
```

虽然作者担心在 useMemo 的 type 数组依赖项发生更改时获取新的 defaultState 值，但这是错误的判断，因为 useState 忽略了新计算的 defaultState 对象。

懒惰初始化 useState 时也是一样的道理，如下所示：

```js
/**
   @param {number} page
   @param {string} type
**/
const myComponent({page, type}) {
  // default state initializer
  const defaultState = () => {
    console.log("default state computed")
    return {
       fetched: someOperationValue(),
       type: type
    }
  }

  const [state, setState] = useState(defaultState);
  return <ExpensiveComponent />
}
```

在上面的示例中，defaultState 初始函数只会在加载时调用一次。这个函数不会在每次重新渲染时再被调用。因此“默认状态计算”这条日志只会出现一次，除非组件又重载了。

上面的代码改成这样：

```js
/**
   @param {number} page
   @param {string} type
**/
const myComponent({page, type}) {
  const defaultState = () => ({
     fetched: someOperationValue(),
     type,
   })

  const [state, setState] = useState(defaultState);

  // if you really need to update state based on prop change,
  // do so here
  // pseudo code - if(previousProp !== prop){setState(newStateValue)}

  return <ExpensiveComponent />
}
```

下面来谈一些更隐蔽的场景。

把useMemo当作ESLint Hook警告
的救命稻草

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/XIibZ0YbvibkVbAzbuKiaWfT5rWmdqxaQA5yGH6YLyjPibo2qVlzedj9FVZ9MNnqVPjHSiaQpwq4Kv1stBBoPs9YjEw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

看看这些评论（详情见下方链接）就能知道，人们在想方设法避免官方的 ESLint Hooks 插件发出 lint 警告。我也很理解他们的困境。

> 评论链接: https://github.com/facebook/create-react-app/issues/6880

我同意 Dan Abramov 的观点（详情见下方链接）。遏制插件中的 eslint-warnings 可能会在将来某天付出相应的代价。

> Dan Abramov 的观点:
> https://github.com/facebook/create-react-app/issues/6880#issuecomment-485912528

一般来说，我认为我们不应该在生产环境的应用程序中遏制这些警告，这样做的话将来就更有可能出现一些隐蔽的错误。

话虽如此，有些情况下我们还是想要遏制这些 lint 警告。以下是我遇到的一个例子。这里的代码是简化过的，方便理解：

```js
function Example ({ impressionTracker, propA, propB, propC }) {
  useEffect(() => {
    // 追踪初始展示
    impressionTracker(propA, propB, propC)
  }, [])

  return <BeautifulComponent propA={propA} propB={propB} propC={propC} />
}
```

这是一个相当棘手的问题。

在上面这个场景中你不关心 props 是否改变。你只想用随便哪个初始 props 调用 track 函数。这就是展示跟踪（impression tracking）的工作机制。你只能在组件加载时调用展示跟踪函数。这里的区别是你需要使用一些初始 props 调用该函数。

你可能会想只要简单地将 props 重命名为 initialProps 之类的东西就能解决问题了，但这是行不通的。这是因为 BeautifulComponent 也需要接收更新的 prop 值。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/XIibZ0YbvibkVbAzbuKiaWfT5rWmdqxaQA5ia6GtN5GkotUoiad61AFjtxpA1pV6lgH9wAjtCm1SiaRdYibyRCFrQBiaKA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在这个示例中，你将收到 lint 警告消息：“React Hook useEffect 缺少依赖项：'impressionTracker'、'propA'、'propB'和'propC'。可以包含它们或删除依赖数组。“

这条消息语气很让人不爽，但 linter 也只是在做自己的工作而已。简单的解决方案是使用 eslint-disable 注释，但这种方法不见得是最合适的，因为将来你可能在同一个 useEffect 调用中引入错误。

```js
useEffect(() => {
  impressionTracker(propA, propB, propC)
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, [])
```

我的建议是使用 useRef Hook 来保持对不需要更新的初始 prop 值的引用。

```js
function Example({impressionTracker, propA, propB, propC}) {
  // 保持对初始值的引用
  const initialTrackingValues = useRef({
      tracker: impressionTracker,
      params: {
        propA,
        propB,
        propC,
    }
  })

  // 展示跟踪
  useEffect(() => {
    const { tracker, params } = initialTrackingValues.current;
    tracker(params)
  }, []) // 对 tracker 或 params 没有 ESLint 警告

  return <BeautifulComponent propA={propA} propB={propB} propC={propC} />
}
```

根据我的测试，在这些情况下 linter 只会考虑 useRef。使用 useRef 后，linter 就明白引用的值不会改变，因此你不会收到任何警告！**哪怕你用 useMemo 也逃不开这些警告的**。

例如：

```js
function Example({impressionTracker, propA, propB, propC}) {

  // useMemo 记住这个值，使它保持不变
  const initialTrackingValues = useMemo({
    tracker: impressionTracker,
    params: {
       propA,
       propB,
       propC,
    }
  }, []) //  这里出现 lint 警告

  // 展示跟踪
  useEffect(() => {
    const { tracker, params} = initialTrackingValues
    tracker(params)
  }, [tracker, params]) //  这些依赖项必须放在这里

  return <BeautifulComponent propA={propA} propB={propB} propC={propC} />
}
```

上面这个方案就是错误的，即使我用 useMemo 记忆初始 prop 值来跟踪初始值，最后还是无济于事。在 useEffect 调用中，记忆值 tracker 和 params 仍然必须作为数组依赖项输入。

有些人就会这样用 useMemo，这种用法不对，应该避免。我们应该使用 useRef Hook，如前所述。

总而言之，如果你真的想要消除 lint 警告的话，你会发现 useRef 是你的好朋友。

useMemo 只用于引用相等

很多人都喜欢使用 useMemo 来处理开销较大的计算并保持引用相等。我同意第一条但不同意第二条。useMemo Hook 不应该只用于引用相等。只有一种情况下可以这样做，稍后会提到。

为什么 useMemo 只用于引用相等是不对的呢？人们不都是这么做的吗？

参考下面的示例：

```js
function Bla() {
  const baz = useMemo(() => [1, 2, 3], [])
  return <Foo baz={baz} />
}
```

在组件 Bla 中，baz 值之所以被记忆不是因为对数组 [1,2,3] 的评估开销很大，而是因为对 baz 变量的引用在每次重新渲染时都会改变。

虽然这看起来不是个问题，但我认为这里不应该使用 useMemo 这个 Hook。

首先，我们看看数组依赖。

```jsx
useMemo(() => [1, 2, 3], [])
```

这里，一个空数组被传递给 useMemo Hook。也就是说值 [1,2,3] 仅计算一次——也就是组件加载的时候。

因此我们得出：被记忆的值计算开销并不大，并且在加载之后不会重新计算。

出现这种情况时，希望你能重新考虑要不要用 useMemo Hook。你正在记忆一个不是计算开销并不大的值，它将来也不会重新计算。这不符合“memoization”一词的定义。

这个 useMemo Hook 的用法大错特错。它在语义上就错了，而且会消耗更多内存和计算资源。

那你该怎么办？

首先，作者在这里究竟想要做什么？他们不是要记住一个值；相反，他们希望在重新渲染时保持对值的 **引用** 不变。

别让那条黏糊糊的变色龙钻了空子。在这种情况下请使用 useRef Hook。

例如，如果你真的讨厌使用当前属性（就像我的很多同事一样），那么只需解构并重命名即可，如下所示：

```js
function Bla() {
  const { current: baz } = useRef([1, 2, 3])
  return <Foo baz={baz} />
}
```

问题解决了。

实际上，你可以使用 useRef 来保持对开销较大的函数评估的引用——只要该函数不需要在 props 更改时重新计算就没问题。

在这些情况下 useRef 才是正确的 Hook，useMemo Hook 不合适。

使用 useRef Hook 来模仿实例变量是 Hook 的强大武库中用的最少的武器之一。useRef Hook 能做的事情远不止保持对 DOM 节点的引用。尽情拥抱它吧。

请记住这里的条件，不要只为了保持一致的引用就记忆一个值。如果你需要根据更改的 prop 或值重新计算该值，那就请随意使用 useMemo Hook。在某些情况下你仍然可以使用 useRef——但是给定数组依赖列表时 useMemo 最方便。




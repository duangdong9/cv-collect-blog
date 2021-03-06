## 送你21道高频JavaScript手写面试题

> [https://mp.weixin.qq.com/s/9o-MXJrblYqUjX5_Uq_rvw](https://mp.weixin.qq.com/s/9o-MXJrblYqUjX5_Uq_rvw)

### 前言

基本上面试的时候，经常会遇到手撕XXX之类的问题，这次准备梳理总结一遍，巩固我们原生JS基础的同时，下次想复习面试手撕题的时候，找起来方便，也节省时间。

> 代码在这里👉GitHub

梳理的顺序是随机的，不按照难度高低程度。

### 实现一个事件委托（易错）

事件委托这里就不阐述了，比如给li绑定点击事件

看错误版，(容易过的，看**「面试官水平了」**)👇

```js
ul.addEventListener('click', function (e) {
            console.log(e,e.target)
            if (e.target.tagName.toLowerCase() === 'li') {
                console.log('打印')  // 模拟fn
            }
})
```

**「有个小bug，如果用户点击的是 li 里面的 span，就没法触发 fn，这显然不对」**👇

```html
<ul id="xxx">下面的内容是子元素1
        <li>li内容>>> <span> 这是span内容123</span></li>
        下面的内容是子元素2
        <li>li内容>>> <span> 这是span内容123</span></li>
        下面的内容是子元素3
        <li>li内容>>> <span> 这是span内容123</span></li>
</ul>
```

这样子的场景就是不对的，那我们看看高级版本👇

```js
		function delegate(element, eventType, selector, fn) {
      element.addEventListener(eventType, e => {
        let el = e.target
        while (!el.matches(selector)) {
          if (element === el) {
            el = null
            break
          }
          el = el.parentNode
        }
        el && fn.call(el, e, el)
      },true)
      return element
    }
```

### 实现一个可以拖拽的DIV

这个题目看起来简单，你可以试一试30分钟能不能完成，直接贴出代码吧👇

```
<div id="xxx"></div>
```

```
var dragging = falsevar position = null
xxx.addEventListener('mousedown',function(e){  dragging = true  position = [e.clientX, e.clientY]})

document.addEventListener('mousemove', function(e){  if(dragging === false) return null  const x = e.clientX  const y = e.clientY  const deltaX = x - position[0]  const deltaY = y - position[1]  const left = parseInt(xxx.style.left || 0)  const top = parseInt(xxx.style.top || 0)  xxx.style.left = left + deltaX + 'px'  xxx.style.top = top + deltaY + 'px'  position = [x, y]})document.addEventListener('mouseup', function(e){  dragging = false})
```

### 手写防抖和节流函数

**「节流throttle」**，规定在一个单位时间内，只能触发一次函数。如果这个单位时间内触发多次函数，只有一次生效。场景👇

- scroll滚动事件，每隔特定描述执行回调函数
- input输入框，每个特定时间发送请求或是展开下拉列表，（防抖也可以）

节流重在加锁**「flag = false」**

```
function throttle(fn, delay) {    let flag = true,        timer = null    return function(...args) {        let context = this        if(!flag) return                flag = false        clearTimeout(timer)        timer = setTimeout(function() {            fn.apply(context,args)            flag = true        },delay)    }}
```

**「防抖debounce」**,在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。场景👇

- 浏览器窗口大小resize避免次数过于频繁
- 登录，发短信等按钮避免发送多次请求
- 文本编辑器实时保存

防抖重在清零**「clearTimeout(timer)」**

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
function debounce(fn, delay) {    let timer = null    return function(...args) {        let context = this        if(timer) clearTimeout(timer)        timer = setTimeout(function(){            fn.apply(context,args)        },delay)    }}
```

### 实现数组去重

这个是Array数组测试用例👇

- 
- 
- 
- 
- 
- 

```
var array = [1, 1, '1', '1', null, null,                 undefined, undefined,                 new String('1'), new String('1'),                 /a/, /a/,                NaN, NaN            ];
```

如何通过一个数组去重，给面试官留下深印象呢👇

使用Set

- 
- 

```
let unique_1 = arr => [...new Set(arr)];
```

使用filter

- 
- 
- 
- 
- 
- 
- 

```
function unique_2(array) {    var res = array.filter(function (item, index, array) {        return array.indexOf(item) === index;    })    return res;}
```

使用reduce

- 
- 

```
let unique_3 = arr => arr.reduce((pre, cur) => pre.includes(cur) ? pre : [...pre, cur], []);
```

使用Object 键值对🐂🐂，这个也是去重最好的效果👇

- 
- 
- 
- 
- 
- 
- 

```
function unique_3(array) {    var obj = {};    return array.filter(function (item, index, array) {        return obj.hasOwnProperty(typeof item + item) ? false : (obj[typeof item + item] = true)    })}
```

使用`obj[typeof item + item] = true`，原因就在于`对象的键值只能是字符串`，所以使用`typeof item + item`代替

### 实现柯里化函数

柯里化就是把接受**「多个参数」**的函数变换成接受一个**「单一参数」**的函数，并且返回接受**「余下参数」**返回结果的一种应用。

思路：

- 判断传递的参数是否达到执行函数的fn个数
- 没有达到的话，继续返回新的函数，并且返回curry函数传递剩余参数

- 
- 
- 
- 
- 

```
let currying = (fn, ...args) =>            fn.length > args.length ?            (...arguments) => currying(fn, ...args, ...arguments) :            fn(...args)
```

测试用例👇

- 
- 
- 
- 
- 
- 

```
let addSum = (a, b, c) => a+b+clet add = curry(addSum)console.log(add(1)(2)(3))console.log(add(1, 2)(3))console.log(add(1,2,3))
```

### 实现数组flat

**「将多维度的数组降为一维数组」**

- 
- 
- 
- 
- 

```
Array.prototype.flat(num)// num表示的是维度// 指定要提取嵌套数组的结构深度，默认值为 1使用 Infinity，可展开任意深度的嵌套数组
```

写这个给面试官看的话，嗯嗯，应该会被打死，写一个比较容易的👇

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
let flatDeep = (arr) => {    return arr.reduce((res, cur) => {        if(Array.isArray(cur)){            return [...res, ...flatDep(cur)]        }else{            return [...res, cur]        }    },[])}
```

**「你想给面试官留下一个深刻印象的话」**，可以这么写，👇

- 
- 
- 
- 
- 
- 
- 
- 
- 

```
function flatDeep(arr, d = 1) {    return d > 0 ? arr.reduce((acc, val) => acc.concat(Array.isArray(val) ? flatDeep(val, d - 1) : val),    []) :        arr.slice();};
// var arr1 = [1,2,3,[1,2,3,4, [2,3,4]]];// flatDeep(arr1, Infinity);
```

可以传递一个参数，数组扁平化几维，简单明了，看起来逼格满满🐂🐂🐂

### 深拷贝

深拷贝解决的就是**「共用内存地址所导致的数据错乱问题」**

思路：

- 递归
- 判断类型
- 检查环（也叫循环引用）
- 需要忽略原型

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
function deepClone(obj, map = new WeakMap()) {    if (obj instanceof RegExp) return new RegExp(obj);    if (obj instanceof Date) return new Date(obj);
    if (obj == null || typeof obj != 'object') return obj;    if (map.has(obj)) {        return map.get(obj);    }    let t = new obj.constructor();    map.set(obj, t);    for (let key in obj) {        if (obj.hasOwnProperty(key)) {            t[key] = deepClone(obj[key], map);        }    }    return t;}//测试用例let obj = {    a: 1,    b: {        c: 2,        d: 3    },    d: new RegExp(/^\s+|\s$/g)}
let clone_obj = deepClone(obj)obj.d = /^\s|[0-9]+$/gconsole.log(clone_obj)console.log(obj)
```

### 实现一个对象类型的函数

核心：Object.prototype.toString

- 
- 
- 
- 
- 
- 

```
let isType = (type) => (obj) => Object.prototype.toString.call(obj) === `[object ${type}]`
// let isArray = isType('Array')// let isFunction = isType('Function')// console.log(isArray([1,2,3]),isFunction(Map))
```

isType函数👆，也属于**「偏函数」**的范畴，偏函数实际上是返回了一个包含**「预处理参数」**的新函数。

### 手写call和apply

改变this指向，唯一区别就是传递参数不同👇

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
// 实现callFunction.prototype.mycall = function () {    let [thisArg, ...args] = [...arguments]    thisArg = Object(thisArg) || window    let fn = Symbol()    thisArg[fn] = this    let result = thisArg[fn](...args)    delete thisArg[fn]    return result}// 实现applyFunction.prototype.myapply = function () {    let [thisArg, args] = [...arguments];    thisArg = Object(thisArg)    let fn = Symbol()    thisArg[fn] = this;    let result = thisArg[fn](...args);    delete thisArg.fn;    return result;}
//测试用例let cc = {    a: 1}
function demo(x1, x2) {    console.log(typeof this, this.a, this)    console.log(x1, x2)}demo.apply(cc, [2, 3])demo.myapply(cc, [2, 3])demo.call(cc,33,44)demo.mycall(cc,33,44)
```

### 手写bind

bind它并不是立马执行函数，而是有一个延迟执行的操作，就是生成了一个新的函数，需要你去执行它👇

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
// 实现bindFunction.prototype.mybind = function(context, ...args){    return (...newArgs) => {        return this.call(context,...args, ...newArgs)    }}
// 测试用例let cc = {    name : 'TianTian'}function say(something,other){    console.log(`I want to tell ${this.name} ${something}`);    console.log('This is some'+other)}let tmp = say.mybind(cc,'happy','you are kute')let tmp1 = say.bind(cc,'happy','you are kute')tmp()tmp1()
```

### 实现new操作

核心要点👇

1. 创建一个新对象，这个对象的`__proto__`要指向构造函数的原型对象
2. 执行构造函数
3. 返回值为object类型则作为new方法的返回值返回，否则返回上述全新对象

代码如下👇

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
function _new() {    let obj = {};    let [constructor, ...args] = [...arguments];    obj.__proto__ = constructor.prototype;    let result = constructor.apply(obj, args);    if (result && typeof result === 'function' || typeof result === 'object') {        return result;    }    return obj;}
```

### 实现instanceof

**「instanceof」** **「运算符」**用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上。

语法👇

- 
- 
- 
- 

```
object instanceof constructorobject 某个实例对象construtor 某个构造函数
```

原型链的向上找，找到原型的最顶端，也就是Object.prototype，代码👇

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
function my_instance_of(leftVaule, rightVaule) {    if(typeof leftVaule !== 'object' || leftVaule === null) return false;    let rightProto = rightVaule.prototype,        leftProto = leftVaule.__proto__;    while (true) {        if (leftProto === null) {            return false;        }        if (leftProto === rightProto) {            return true;        }        leftProto = leftProto.__proto__    }}
```

### 实现sleep

某个时间后就去执行某个函数，使用Promise封装👇

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
function sleep(fn, time) {    return new Promise((resolve, reject) => {        setTimeout(() => {            resolve(fn);        }, time);    });}let saySomething = (name) => console.log(`hello,${name}`)async function autoPlay() {    let demo = await sleep(saySomething('TianTian'),1000)    let demo2 = await sleep(saySomething('李磊'),1000)    let demo3 = await sleep(saySomething('掘金的好友们'),1000)}autoPlay()
```

### 实现数组reduce

更多的手写实现数组方法，看我之前这篇👉 「数组方法」从详细操作js数组到浅析v8中array.js

直接给出简易版👇

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
Array.prototype.myreduce = function(fn, initVal) {    let result = initVal,        i = 0;    if(typeof initVal  === 'undefined'){        result = this[i]        i++;    }    while( i < this.length ){        result = fn(result, this[i])    }    return result}
```

### 实现Promise.all和race

不清楚两者用法的话，异步MDN👉Promise.race()  Promise.all()

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
// 实现Promise.all 以及 race
Promise.myall = function (arr) {    return new Promise((resolve, reject) => {        if (arr.length === 0) {            return resolve([])        } else {            let res = [],                count = 0            for (let i = 0; i < arr.length; i++) {                // 同时也能处理arr数组中非Promise对象                if (!(arr[i] instanceof Promise)) {                    res[i] = arr[i]                    if (++count === arr.length)                        resolve(res)                } else {                    arr[i].then(data => {                        res[i] = data                        if (++count === arr.length)                            resolve(res)                    }, err => {                        reject(err)                    })                }
            }        }    })}
Promise.myrace = function (arr) {    return new Promise((resolve, reject) => {        for (let i = 0; i < arr.length; i++) {            // 同时也能处理arr数组中非Promise对象            if (!(arr[i] instanceof Promise)) {                Promise.resolve(arr[i]).then(resolve, reject)            } else {                arr[i].then(resolve, reject)            }
        }    })}
```

测试用例👇

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
// 测试用例let p1 = new Promise((resolve, reject) => {    setTimeout(() => {        resolve(11)    }, 2000);});let p2 = new Promise((resolve, reject) => {    reject('asfs')
});let p3 = new Promise((resolve) => {    setTimeout(() => {        resolve(33);    }, 4);});
Promise.myall([p3, p1, 3, 4]).then(data => {    // 按传入数组的顺序打印    console.log(data); // [3, 1, 2]}, err => {    console.log(err)});
Promise.myrace([p1, p2, p3]).then(data => {    // 谁快就是谁    console.log(data); // 2}, err => {    console.log('失败跑的最快')})
```

### 手写继承

继承有很多方式，这里不过多追溯了，可以看看这篇 JS原型链与继承别再被问倒了

主要梳理的是 寄生组合式继承 和Class继承怎么使用

**「寄生组合式继承」**

- 
- 
- 
- 
- 
- 
- 
- 
- 

```
function inheritPrototype(subType, superType) {    // 创建对象，创建父类原型的一个副本    var prototype = Object.create(superType.prototype);     // 增强对象，弥补因重写原型而失去的默认的constructor 属性    prototype.constructor = subType;     // 指定对象，将新创建的对象赋值给子类的原型    subType.prototype = prototype; }
```

测试用例👇

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
// 父类初始化实例属性和原型属性function Father(name) {    this.name = name;    this.colors = ["red", "blue", "green"];}Father.prototype.sayName = function () {    alert(this.name);};
// 借用构造函数传递增强子类实例属性（支持传参和避免篡改）function Son(name, age) {    Father.call(this, name);    this.age = age;}
// 将父类原型指向子类inheritPrototype(Son, Father);
// 新增子类原型属性Son.prototype.sayAge = function () {    alert(this.age);}
var demo1 = new Son("TianTian", 21);var demo2 = new Son("TianTianUp", 20);
demo1.colors.push("2"); // ["red", "blue", "green", "2"]demo2.colors.push("3"); // ["red", "blue", "green", "3"]
```

Class实现继承👇

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
class Rectangle {    // constructor    constructor(height, width) {        this.height = height;        this.width = width;    }    // Getter    get area() {        return this.calcArea()    }    // Method    calcArea() {        return this.height * this.width;    }}
const rectangle = new Rectangle(40, 20);console.log(rectangle.area);// 输出 800// 继承class Square extends Rectangle {    constructor(len) {        // 子类没有this,必须先调用super        super(len, len);
        // 如果子类中存在构造函数，则需要在使用“this”之前首先调用 super()。        this.name = 'SquareIng';    }    get area() {        return this.height * this.width;    }}const square = new Square(20);console.log(square.area);// 输出 400
```

`extends`继承的核心代码如下，其实和上述的寄生组合式继承方式一样👇

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
function _inherits(subType, superType) {      // 创建对象，创建父类原型的一个副本    // 增强对象，弥补因重写原型而失去的默认的constructor 属性    // 指定对象，将新创建的对象赋值给子类的原型    subType.prototype = Object.create(superType && superType.prototype, {        constructor: {            value: subType,            enumerable: false,            writable: true,            configurable: true        }    });        if (superType) {        Object.setPrototypeOf             ? Object.setPrototypeOf(subType, superType)             : subType.__proto__ = superType;    }}
```

把实现原理跟面试官扯一扯，这小子基础还行。

### 手写一下AJAX

写的初略版的，详细版的就不梳理了，面试的时候，跟面试官好好探讨一下吧🐂🐂🐂

- 
- 
- 
- 
- 
- 
- 
- 

```
var request = new XMLHttpRequest() request.open('GET', 'index/a/b/c?name=TianTian', true); request.onreadystatechange = function () {   if(request.readyState === 4 && request.status === 200) {     console.log(request.responseText);   }}; request.send();
```

### 用正则实现 trim()

去掉首位多余的空格👇

- 
- 
- 
- 
- 
- 
- 
- 

```
String.prototype.trim = function(){    return this.replace(/^\s+|\s+$/g, '')}//或者 function trim(string){    return string.replace(/^\s+|\s+$/g, '')}
```

### 实现Object.create方法

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
//实现Object.create方法function create(proto) {    function Fn() {};    Fn.prototype = proto;    Fn.prototype.constructor = Fn;    return new Fn();}let demo = {    c : '123'}let cc = Object.create(demo)
```

### 实现一个同时允许任务数量最大为n的函数

使用Promise封装，给你一个数组，数组的每一项是一个Promise对象

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
function limitRunTask(tasks, n) {  return new Promise((resolve, reject) => {    let index = 0, finish = 0, start = 0, res = [];    function run() {      if (finish == tasks.length) {        resolve(res);        return;      }      while (start < n && index < tasks.length) {        // 每一阶段的任务数量++        start++;        let cur = index;        tasks[index++]().then(v => {          start--;          finish++;          res[cur] = v;          run();        });      }    }    run();  })  // 大概解释一下：首先如何限制最大数量n  // while 循环start < n，然后就是then的回调}
```

### 10进制转换

给定10进制数，转换成[2~16]进制区间数，就是简单模拟一下。

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
function Conver(number, base = 2) {  let rem, res = '', digits = '0123456789ABCDEF', stack = [];
  while (number) {    rem = number % base;    stack.push(rem);
    number = Math.floor(number / base);  }
  while (stack.length) {    res += digits[stack.pop()].toString();  }    return res;}
```

### 数字转字符串千分位

写出这个就很逼格满满🐂🐂🐂

- 
- 
- 
- 

```
function thousandth(str) {  return str.replace(/\d(?=(?:\d{3})+(?:\.\d+|$))/g, '$&,');}
```
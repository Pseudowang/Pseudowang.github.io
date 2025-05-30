+++
title = 'Iterable Object In JavaScript'
date = 2025-05-30T10:39:26+08:00
draft = false
categories = ["Notes"]
tags = ["Notes", "Input"]

+++

> 一些粗略地见解，如有写错或低级错误，欢迎您的指正。
> 我们在去学习 iterable object ，首先我们要去了解一下什么是 iteration。

### Iteration

迭代是重复的一个过程，以生成一系列 (可能是无限的) 结果，每次重复这个过程称为一次迭代，每次迭代的结果成为下一次迭代的起点

循环就是执行迭代最常用的语言结构，以下伪代码通过一个 for 循环"迭代"了 begin 和 end 之间的代码行三次，并使用值作为增量

```bash
a := 0
for i := 1 to 3 do       { loop three times }
begin
    a := a + i;          { add the current value of i to a }
end;
print(a);                { the number 6 is printed (0 + 1; 1 + 2; 3 + 3) }
```

### Iteration protocols

**Iteration protocols** 不是新的内置或语法，而是一套协议。任何对象只需遵循特定的约定规范，即可实现这些协议。

共有两种协议： [iterable protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_iterable_protocol) 和 [iterator protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_iterator_protocol).

#### The iterable protocol

可迭代协议允许 JavaScript 对象定义或自定义它们的迭代行为，例如在 `for...of` 结构中有哪些值被循环。一些内置类型同时是内置的可迭代对象，并且有默认的迭代行为，比如 `Array` 或者 `Map`，而其他的内置类型则不是 (比如 `Object`)

内置的可迭代对象有:

- [`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)
- [`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [`Map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- [`Set`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
- [`TypedArray`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)
- [`NodeList`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList)
- [`arguments`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments)
- [Generator functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)

为了实现可迭代，对象必须实现  `[Symbol.iterator]()`  方法，这意味着该对象 (或其原型链上的某个对象) 必须有一个键为 `[Symbol.iterator]`  的属性，可通过常量  [`Symbol.iterator`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator)  访问该属性：
[` [Symbol.iterator]()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#symbol.iterator)
零参数函数，返回一个符合迭代器协议的对象

每当需要迭代一个对象时 (例如在 `for...of` 循环的开头)，就会调用该对象的 `[Symbol.iterator]()` 方法 (不带参数)，并使用返回的迭代器来获取要迭代的值

在调用这个零参数函数时，它是作为可，迭代对象上的一个方法被调用的。因此，在函数内部，`this` 关键字可用于访问可迭代对象的属性，以决定在迭代过程中提供哪些属性。就像下方中的

```js
current: this.from,
last: this.to,
```

访问的就是 `range` 这个可迭代对象的 `from` 和 `to` 属性。等于 `range.to` 和 `range.from`

#### The iterator protocol

**迭代器协议** 定义了产生一系列值 (无论是有限个还是无限个) 的标准方式，当值为有限个时，所有的值都被迭代完毕后，则会返回一个默认的返回值

只有实现了一个拥有一下语义 (semantic) 的 `next()` 方法，一个对象才能称为迭代器

`next()`
无参数或接受一个参数的函数，并返回符合  `IteratorResult`  接口的对象。如果在使用迭代器内置的语言特征（例如  `for...of`）时，得到一个非对象返回值（例如  `false`  或  `undefined`），将会抛出  [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError)（`"iterator.next() returned a non-object value"`）。

`done` (可选)
迭代器返回的任何 JavaScript 值。`done`  为  `true`  时可省略。

实际上，两者都不是严格要求的；如果返回没有任何属性的对象，则实际上等价于  `{ done: false, value: undefined }`。

### 例子

例如，我们有一个对象，它并不是数组，但是看起来很很适合时候 `for...of` 循环。
比如一个 `range` 对象，它代表了一个数字区间:

```js
let range = {
  from: 1,
  to: 5,
};

// 我们希望 for..of 这样运行：
// for(let num of range) ... num=1,2,3,4,5
```

为了让 `range` 对象可迭代我们需要为对象添加一个名为 `Symbol.iterator` 的方法 (一个专门用于使对象可迭代的内建 symbol)。

1. 当 `for..of` 循环启动时，它就会调用这个方法 (如果没找到，就会报错)。这个方法必须返回一个 **迭代器 (iterator)**-- 一个有 `next` 方法的对象
2. 从此开始，`for...of` **仅适用于这个被返回的对象**
3. 当 `for..of`  循环希望取得下一个数值，它就调用这个对象的  `next()`  方法。
4. `next()`  方法返回的结果的格式必须是  `{done: Boolean, value: any}`，当  `done=true`  时，表示循环结束，否则  `value`  是下一个值。

这是带有注释的 `range` 的完整实现:

```js
let range = {
  from: 1,
  to: 5,
};

// 1. for..of 调用首先会调用这个：
range[Symbol.iterator] = function () {
  // ……它返回迭代器对象（iterator object）：
  // 2. 接下来，for..of 仅与下面的迭代器对象一起工作，要求它提供下一个值
  return {
    current: this.from,
    last: this.to,

    // 3. next() 在 for..of 的每一轮循环迭代中被调用
    next() {
      // 4. 它将会返回 {done:.., value :...} 格式的对象
      if (this.current <= this.last) {
        return { done: false, value: this.current++ };
      } else {
        return { done: true };
      }
    },
  };
};

// 现在它可以运行了！
for (let num of range) {
  alert(num); // 1, 然后是 2, 3, 4, 5
}
```

请注意可迭代对象的核心功能：关注点分离。

- `range`  自身没有  `next()`  方法。
- 相反，是通过调用  `range[Symbol.iterator]()`  创建了另一个对象，即所谓的“迭代器”对象，并且它的  `next`  会为迭代生成值。

因此，迭代器对象和与其迭代的对象是分开的

> 这里例子中的可迭代对象(iterable object)是 `range`，因为它有 `[Symbol.iterator]` 方法，当我们 `for(let num of range)` 的时候 , js 会自动调用 `range[Symbol.iterator]`，这个方法**返回的对象**（就是有 current, last, next 这些属性的方法的对象），才是**迭代器对象**。
> 与其迭代的对象：指的是拥有 `[Symbol.iterator]` 的本体 (这个例子就是 range)

所以我们可以将它们合并，并使用 `range` 自身作为迭代器来简化代码.

```js
let range = {
  from: 1,
  to: 5,

  [Symbol.iterator]() {
    this.current = this.from;
    return this;
  },

  next() {
    if (this.current <= this.to) {
      return { done: false, value: this.current++ };
    } else {
      return { done: true };
    }
  },
};

for (let num of range) {
  alert(num); // 1, 然后是 2, 3, 4, 5
}
```

现在  `range[Symbol.iterator]()`  返回的是  `range`  对象自身：它包括了必需的  `next()`  方法，并通过  `this.current`  记忆了当前的迭代进程。

但缺点是，现在不可能同时在对象上运行两个  `for..of`  循环了：它们将共享迭代状态，因为只有一个迭代器，即对象本身。但是两个并行的  `for..of`  是很罕见的，即使在异步情况下。

### 字符串是可迭代的

数组和字符串是使用最广泛的内建可迭代对象，就像上面所说的。

可以使用 `for...of` 来遍历它的每个字符:

```js
for (let char of "test") {
  // 触发 4 次，每个字符一次
  alert(char); // t, then e, then s, then t
}
```

### 显式调用迭代器

这段代码创建了一个字符串迭代器，并“手动”从中获取值

```js
let str = "Hello";

// 和 for..of 做相同的事
// for (let char of str) alert(char);

let iterator = str[Symbol.iterator]();

while (true) {
  let result = iterator.next();
  if (result.done) break;
  alert(result.value); // 一个接一个地输出字符
}
```

### 可迭代（iterable）和类数组（array-like）

- **iterable** 如上所述，是实现了 `Symbol.iterator` 方法的对象
- **Array-like** 是有索引和 `length` 属性的对象，所以它们看起来很像数组

字符串即是可迭代的（`for..of`  对它们有效），又是类数组的（它们有数值索引和  `length`  属性）。

但是一个可迭代对象也许不是类数组对象。反之亦然，类数组对象可能不可迭代。

例如，上面例子中的  `range`  是可迭代的，但并非类数组对象，因为它没有索引属性，也没有  `length`  属性。

下面这个对象则是类数组的，但是不可迭代：

```js
let arrayLike = {
  // 有索引和 length 属性 => 类数组对象
  0: "Hello",
  1: "World",
  length: 2,
};

// Error (no Symbol.iterator)
for (let item of arrayLike) {
}
```

可迭代对象和类数组对象通常都  **不是数组**，它们没有  `push`  和  `pop`  等方法。

#### Array. From

有一个全局方法  [Array.from](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Array/from)  可以接受一个可迭代或类数组的值，并从中获取一个“真正的”数组。然后我们就可以对其调用数组方法了。

```js
let arrayLike = {
  0: "Hello",
  1: "World",
  length: 2,
};

let arr = Array.from(arrayLike); // (*)
alert(arr.pop()); // World（pop 方法有效）
```

在  `(*)`  行的  `Array.from`  方法接受对象，检查它是一个可迭代对象或类数组对象，然后创建一个新数组，并将该对象的所有元素复制到这个新数组。

如果是**可迭代对象**，也是同样：

```js
// 假设 range 来自上文的例子中
let arr = Array.from(range);
alert(arr); // 1,2,3,4,5 （数组的 toString 转化方法生效）
```

`Array.from`  的完整语法允许我们提供一个可选的“映射（mapping）”函数：

```js
Array`.``from``(`obj`[``,` mapFn`,` thisArg`]``)`
```

可选的第二个参数 `mapFn` 可以是一个函数，该函数会在对象中的元素被添加到数组前，被应用于每个元素，此外 `thisArg` 允许我们为该函数设置 `this` [[数组方法 - info#大多数方法都支持 "thisArg"]]

```js
// 假设 range 来自上文例子中

// 求每个数的平方
let arr = Array.from(range, (num) => num * num);

alert(arr); // 1,4,9,16,25
```

现在我们用  `Array.from`  将一个字符串转换为单个字符的数组：

```js
let str = "𝒳😂";

// 将 str 拆分为字符数组
let chars = Array.from(str);

alert(chars[0]); // 𝒳
alert(chars[1]); // 😂
alert(chars.length); // 2
```

与  `str.split`  方法不同，它依赖于字符串的可迭代特性。因此，就像  `for..of`  一样，可以正确地处理代理对（surrogate pair）。（译注：代理对也就是 UTF-16 扩展字符。）

### 总结

可以应用 `for..of` 的对象被称为 **可迭代的**

- 技术上来说，可迭代对象必须实现 `Symbol.iterator` 方法。
  - `obj[Symbol.iterator]()`  的结果被称为  **迭代器（iterator）**。由它处理进一步的迭代过程。
  - 一个迭代器必须有  `next()`  方法，它返回一个  `{done: Boolean, value: any}`  对象，这里  `done:true`  表明迭代结束，否则  `value`  就是下一个值。
- `Symbol.iterator`  方法会被  `for..of`  自动调用，但我们也可以直接调用它。
- 内建的可迭代对象例如字符串和数组，都实现了  `Symbol.iterator`。
- 字符串迭代器能够识别代理对（surrogate pair）。（译注：代理对也就是 UTF-16 扩展字符。）

有索引属性和  `length`  属性的对象被称为  **类数组对象**。这种对象可能还具有其他属性和方法，但是没有数组的内建方法。
`Array.from(obj[, mapFn, thisArg])`  将可迭代对象或类数组对象  `obj`  转化为真正的数组  `Array`，然后我们就可以对它应用数组的方法。可选参数  `mapFn`  和  `thisArg`  允许我们将函数应用到每个元素。

## 📖 参考文献

1. https://zh.javascript.info/iterable
2. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols
3. https://www.bilibili.com/video/BV1ca411t7A9/?spm_id_from=333.337.search-card.all.click&vd_source=e7ad5d4353f32ffd738d97fb70b7d567
4. https://en.wikipedia.org/wiki/Iterator

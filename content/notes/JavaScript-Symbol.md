+++
title = 'JavaScript Symbol'
date = 2025-05-20T18:57:04+08:00
draft = false
categories = ["Notes"]
tags = ["Notes", "Input"]
+++

> 一些粗略地见解，如有写错或低级错误，欢迎您的指正

根据规范，只有[两种原始类型](https://developer.mozilla.org/zh-CN/docs/Glossary/Primitive)可以用作对象属性键:

- 字符串类型
- Symbol 类型

如果不用这两种原始类型作为对象的属性键，例如[[数据类型 - info#Number 类型|数字]]，它会被自动转换为字符串。所以  `obj[1]`  与  `obj["1"]`  相同，而  `obj[true]`  与  `obj["true"]`  相同。

### Symbol

"symbol" 值表示唯一的标识符
可以使用 `Symbol()` 来创建这种类型的值：

```js
let id = Symbol();
```

在我们创建时，也可以给 symbol 一个描述 (也称为 symbol 名)

```js
// 变量 id 是一个 Symbol 类型的变量，其描述为 "id"。
let id = Symbol("id");
```

Symbol 保证是唯一的。即使我们创建了许多相同描述的 symbol，它们的值与使不同的。描述仅仅只是一个标签，不会影响任何东西

```js
let id1 = Symbol("id");
let id2 = Symbol("id");
console.log(id1 === id2); // false
```

#### Symbol 属性与字符串属性的区别

- 字符串键会覆盖同名属性:

```js
let user = { id: 1 };
user.id = 2; // 覆盖原值
console.log(user.id); // 2
```

- Symbol 键不会覆盖同名属性 (因为它表示唯一的标识符)

```js
let user = { id: 1 };
let idSymbol = Symbol("id");
user[idSymbol] = 2; // 添加新属性，不影响字符串键id
console.log(user.id); // 1
console.log(user[idSymbol]); // 2
```

**Symbol 类型的属性键不能用点语法访问，只能通过方括号语法 `obj[symbolKey]` 来访问对应的 Symbol 属性。**

例如，`user.id` 会访问字符串键"id"，而不是 Symbol 键 id。因此，访问 Symbol 属性必须使用方括号和对应的 Symbol 变量，如 `user[id]`。

> [!NOTE] symbol 不会被自动转换为字符串
> JavaScript 中的大多数值都支持字符串的隐式转换。例如，我们可以  `alert`  任何值，都可以生效。Symbol 比较特殊，它不会被自动转换。
> 例如，这个 `alert` 将会提示出错
>
> ```js
> Let id = Symbol ("id");
> Alert (id); // 类型错误：无法将 symbol 值转换为字符串。
> ```
>
> 如果真的很想显示一个 symbol，我们需要在它上面调用 `.toString()` 或者获取 `symbol.Description` 属性，只显示描述 (description)
>
> ```js
> Let id = Symbol ("id");
> Alert (id.ToString ()); // Symbol (id)，现在它有效了
> Alert (id. Description); // id
> ```

### "隐藏"属性

Symbol 允许我们创建对象的 “隐藏”属性，代码的任何其他部分都不能意外或重写这些属性

例如，如果我们使用的是属于第三方代码的  `user`  对象，我们想要给它们添加一些标识符。

我们可以给它们使用 symbol 键：

```js
let user = {
  // 属于另一个代码
  name: "John",
};

let id = Symbol("id");

user[id] = 1;

alert(user[id]); // 我们可以使用 symbol 作为键来访问数据
```

由于  `user`  对象属于另一个代码库，所以向它们添加字段是不安全的，因为我们可能会影响代码库中的其他预定义行为。但 symbol 属性不会被意外访问到。第三方代码不会知道新定义的 symbol，因此将 symbol 添加到  `user`  对象是安全的。

### 对象字面量中的 symbol

如果我们要在对象字面量  `{...}`  中使用 symbol，则需要使用方括号把它括起来。

就像这样：

```js
let id = Symbol("id");

let user = {
  name: "John",
  [id]: 123, // 而不是 "id"：123
};
这是因为我们需要;
```

### Symbol 在 for... In 中会被跳过

symbol 属性不参与  `for..in`  循环。
例如

```js
let id = Symbol("id");
let user = {
  name: "John",
  age: 30,
  [id]: 123,
};

for (let key in user) alert(key); // name, age（没有 symbol）

// 使用 symbol 任务直接访问
alert("Direct: " + user[id]); // Direct: 123
```

[Object.keys(user)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)  也会忽略它们。这是一般“隐藏符号属性”原则的一部分。如果另一个脚本或库遍历我们的对象，它不会意外地访问到符号属性。

相反，[Object.assign](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)  会同时复制字符串和 symbol 属性：

```js
let id = Symbol("id");
let user = {
  [id]: 123,
};

let clone = Object.assign({}, user);

alert(clone[id]); // 123
```

### 全局 symbol

通常所有的 symbol 都是不同的，即使它们有相同的名字。但有时我们想要名字相同的 symbol 具有相同的实例。例如，应用程序的不同部分想要访问的 symbol `"id"`  指的是完全相同的属性。

这里有一个 **全局 symbol 注册表**。我们可以在其中创建 symbol 并在稍后访问它们，它可以确保每次访问相同名字 symbol 时，返回的都是相同的 symbol。

#### Symbol. For (key)

要从注册表中读取（不存在则创建）symbol，要使用  **`Symbol.for(key)`**。

在 JavaScript 引擎维护的 **全局 symbol 注册表**中，如果有一个描述为  `key`  的 symbol，则返回该 symbol，否则将创建一个新 symbol（`Symbol(key)`），并通过给定的  `key`  将其存储在注册表中。

```js
// 创建或获取全局注册的 Symbol
let sym1 = Symbol.for("id"); // 注册表中没有 "id"，创建新 Symbol
let sym2 = Symbol.for("id"); // 找到已注册的 "id"，返回同一 Symbol
console.log(sym1 === sym2); // true
```

注册表内的 symbol 被称为  **全局 symbol**。

#### Symbol. KeyFor

按名字返回一个 symbol。相反，通过全局 symbol 返回一个名字，我们可以使用  `Symbol.keyFor(sym)`：

```js
// 通过 name 获取 symbol
let sym = Symbol.for("name");
let sym2 = Symbol.for("id");

// 通过 symbol 获取 name
alert(Symbol.keyFor(sym)); // name
alert(Symbol.keyFor(sym2)); // id
```

`Symbol.keyFor`  内部使用全局 symbol 注册表来查找 symbol 的键。所以它不适用于非全局 symbol。如果 symbol 不是全局的，它将无法找到它并返回  `undefined`。

## 📖 参考文献

1. https://zh.javascript.info/symbol

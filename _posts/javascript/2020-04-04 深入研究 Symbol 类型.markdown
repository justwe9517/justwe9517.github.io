---
title: 深入研究 Symbol 类型
date: 2020-04-04 15:37:00 Z
categories:
- javascript
tags:
- javascript
- 笔记
layout: post
---

# 深入研究 Symbol 类型

Symbol 是 ES6 中新增加的一种基本数据类型，此类型使用 `Symbol()` 方式创建，创建出来的值是一个唯一的 symbol 类型值，它有一个比较有意思的特性就是用于为对象创建不可迭代的成员（属性、方法都可以）。除此之外你还可以使用它创建常量。下面让我们来看一看此类型的用途。

让我们先看看他的类型定义：

```typescript
Symbol(description?: string | number): symbol;
```

由此可见，此“对象”接收一个字符串或数字类型的可选参数，用于为 Symbol 定义描述。我之所以在对象上面加了一个引号是因为，Symbol 并非真正意义上的对象，因为它不能被 `new` 关键字创建，但通过 `Symbol()` 创建出来的 symbol 对象可以访问三个属性，分别是 `toString()`, `valueOf()` 和 ES2019 中新增的 `description` 属性。

下面看一看 Symbol 类型的特点：

```typescript
let tmp1 = Symbol("tmp");
let tmp2 = Symbol("tmp");
console.log(tmp1 == tmp2);      // false
```

上面的代码中我们创建了两个变量 `tmp1` 和 `tmp2`，他们分别被赋值成 `Symbol("tmp")`，但在进行比较时却返回了 `false`，这便是 Symbol 类型最大的特色“唯一性”。那它的作用是什么呢？你可以想想，如果将 Symbol 作为常量的值，你就无需在担心命名重复的问题了。

接下来让我们再来看看它的类型：

```typescript
console.log(typeof tmp1);   // symbol
```

通过上面的代码我们发现他的类型在预期之内，是一等公民没错。下面让我们看看 Symbol 最具特色的功能。

## 为对象创建不可被枚举的成员

故名思意，你可以通过 `Symbol("name")` 去作为对象键值并为他创建一个不可枚举的成员，不可被枚举就是无法被 `for in` 遍历出来的成员。

让我们从下面这个例子上看一看：

```typescript
let obj = {
    tmp: "new Object"
};
let sym = Symbol("hello");
obj[sym] = "world";
console.log(obj);    // {tmp: "new Object", Symbol(hello): "world"}
for (let n in obj) console.log(n);    // tmp
```

首先第一行是创建一个对象，第二行是通过 Symbol 为对象添加一个属性。在第三行中，我们通过 console.log 方法打印了 obj 的值，从中看到了一个类型为 `Symbol(hello): "world"` 的属性，第四行中我们又尝试遍历整个对象，发现这个 Symbol 并没有被遍历出来，这说明通过 `Symbol` 创建的属性确实是无法被枚举的。那有其他办法来访问这个属性吗，有的 `console.log(obj[sym])` 通过这种方式就可以了。

如果你在控制台执行 `window.location` 的话，同样能看见 `location` 对象中存在一个 `Symbol(Symbol.toPrimitive): undefined` 的属性，关于它的内容，我们放在最后在讲，现在让我们把重点回到 `Symbol` 上。

在 JavaScript 中，有两种方式可以访问创建的 symbol：

```typescript
Object.getOwnPropertySymbols(obj);    // [Symbol(hello)]
Reflect.ownKeys(obj);                 // (2) ["tmp", Symbol(hello)]
```

## 全局 Symbol

在 Symbol 中存在一个 symbol 注册表，此注册表的作用是为了共享 symbol 的属性到其他页面中。所以创建一个全局 symbol 于创建一个普通 symbol 是不同的。

Symbol 提供了两个 API 来访问全局注册表，分别是 `Symbol.for()` 和 `Symbol.keyFor()` 方法，它俩的定义如下：

```typescript
// 通过 symbol 描述在 symbol 注册表查找 symbol，如果不存在则创建一个
Symbol.for(key: string): symbol;
// 通过 symbol 在 symbol 注册表查找 symbol，如果不存在则返回 undefined
Symbol.keyFor(sym: symbol): string | undefined;
```

仔细看关于这两个方法的定义，`for` 接受的是 `string` 类型的数据，而 `keyFor` 接受的是 `symbol` 类型的数据。

我们通过下面的例子在了解一下通过 `Symbol()` 和 `Symbol.for()` 创建的 symbol 有何不同：

```typescript
let s1 = Symbol("symbol");
let s2 = Symbol("symbol");
console.log(s1 == s2);    // false

let s3 = Symbol.for("symbol");
let s4 = Symbol.for("symbol");
console.log(s3 == s4);    // true
```

可以看到，通过 `Symbol.for()` 创建的 symbol 是一致的。

## JavaScript 内置的 symbols

还记得上文提到的 `location` 对象中的 `Symbol(Symbol.toPrimitive): undefined` 属性吗，它便是 JavaScript 内置的 symbol 属性。

JavaScript 提供了非常多的 symbol 用来扩展对象属性：

类型 | 属性 | 描述
-|-|-
迭代 | Symbol.iterator | 一个返回一个对象默认迭代器的方法。被 for...of 使用。 |
迭代 | Symbol.asyncIterator | 一个返回对象默认的异步迭代器的方法。被 for await of 使用。 |
正则表达式 | Symbol.match | 一个用于对字符串进行匹配的方法，也用于确定一个对象是否可以作为正则表达式使用。被 String.prototype.match() 使用。 |
正则表达式 | Symbol.replace | 一个替换匹配字符串的子串的方法. 被 String.prototype.replace() 使用。|
正则表达式 | Symbol.search | 一个返回一个字符串中与正则表达式相匹配的索引的方法。被String.prototype.search() 使用。 |
正则表达式 | Symbol.split | 一个在匹配正则表达式的索引处拆分一个字符串的方法.。被 String.prototype.split() 使用。 |
其他 | Symbol.hasInstance | 一个确定一个构造器对象识别的对象是否为它的实例的方法。被 instanceof 使用。 |
其他 | Symbol.isConcatSpreadable | 一个布尔值，表明一个对象是否应该flattened为它的数组元素。被 Array.prototype.concat() 使用。 |
其他 | Symbol.unscopables | 拥有和继承属性名的一个对象的值被排除在与环境绑定的相关对象外。 |
其他 | Symbol.species | 一个用于创建派生对象的构造器函数。 |
其他 | Symbol.toPrimitive | 一个将对象转化为基本数据类型的方法。 |
其他 | Symbol.toStringTag | 用于对象的默认描述的字符串值。被 Object.prototype.toString() 使用。 |

关于内置 symbols 的更多信息请参考 [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 此文。

我这里仅使用 `Symbol.iterator` 举一个例子：

在 es6 中增加了一个新的循环，称为 `for of` 循环，一个数组可以直接通过 `for of` 进行循环，但一个对象却不行，这是因为对象缺少迭代器，如果想把对象放入 `for of` 中进行循环，我们就需要在对象内部实现一个迭代器，告诉 `for of` 如何处理数据。

下面我们花时间来改造一下 `obj` 对象：

```typescript

let obj = {
    tmp: "new Object",
    // 关于迭代器的介绍，我放在以后在说
    [Symbol.iterator]: function () {
    let index = 0;
    let propKeys = Reflect.ownKeys(obj);
    return {
        next() {
            if (index < propKeys.length) {
                let key = propKeys[index];
                index++;
                return { value: [key, obj[key]] };
            } else {
                return { done: true };
            }
        }
    }
};
```

在实现了这个迭代器后，obj 对象就可以被放入 `for of` 中愉快的迭代了。
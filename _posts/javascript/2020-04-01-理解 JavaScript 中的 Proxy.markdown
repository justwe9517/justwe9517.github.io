---
title: 理解 JavaScript 中的 Proxy
date: 2020-04-01 15:37:00 Z
categories:
- javascript
tags:
- javascript
- 笔记
layout: post
---

# 理解 JavaScript 中的 Proxy

Proxy 是 ES6 中提供的一种新 API，翻译过来就是代理的意思，指代理人帮助委托人处理有关的事务，你可以简单的理解为通过 Proxy 你可以对 Object 中的任何行为并加以修饰，并返回修饰后的数据。

下面先看一下关于 JavaScript Proxy 中的声明信息：

```ts
new proxy(target: {}, handler: ProxyHandler<{}>) => {}
```

由此声明可知，Proxy 接收两个参数，分别是 target 和 handler，其中 target 是指需要被代理的对象（可以是任何类型），而 handler 是一个代理句柄，他接收的是一个 ProxyHandler 对象，此对象在 mdn 中解释为 `包含陷阱（trap）的占位符对象，可译为处理器对象。`，关于此对象的详细信息如下：

```ts
interface ProxyHandler<T extends object> {
    // 当读取代理对象的原型时，该方法会被调用
    getPrototypeOf? (target: T): object | null;
    // 当使用 Object.setPrototypeOf() 方法为代理对象设置新的原型时被触发
    setPrototypeOf?(target: T, v: any): boolean;
    // 当使用 Object.isExtensible() 方法检查代理对象是否可扩展时触发
    isExtensible?(target: T): boolean;
    // 当使用 Object.preventExtensions() 方法让一个代理对象变成不可扩展时触发
    preventExtensions?(target: T): boolean;
    // 当使用 Object.getOwnPropertyDescriptor() 方法获取代理对象上一个自有属性对应的属性描述符时触发
    getOwnPropertyDescriptor?(target: T, p: PropertyKey): PropertyDescriptor | undefined;
    // 当使用 in 操作符检查代理对象属性时触发
    has?(target: T, p: PropertyKey): boolean;
    // 当读取代理对象属性时触发
    get?(target: T, p: PropertyKey, receiver: any): any;
    // 当设置代理对象属性时触发
    set?(target: T, p: PropertyKey, value: any, receiver: any): boolean;
    // 当使用 delete 关键字操作代理对象的属性时触发
    deleteProperty?(target: T, p: PropertyKey): boolean;
    // 当使用 Object.defineProperty() 方法为代理对象创建或修改属性时触发
    defineProperty? (target: T, p: PropertyKey, attributes: PropertyDescriptor): boolean;
    // 当枚举代理对象属性时触发
    enumerate?(target: T): PropertyKey[];
    // 当使用 Reflect.ownKeys() 方法操作代理对象时触发
    ownKeys?(target: T): PropertyKey[];
    // 当被代理对象尝试使用函数方式调用时被触发
    apply?(target: T, thisArg: any, argArray?: any): any;
    // 用于拦截 new 关键字，为了使 new 操作符在生成的 Proxy 对象上生效，用于初始化代理的目标对象自身必须具有[[Construct]]内部方法（即 new target 必须是有效的）。
    construct? (target: T, argArray: any, newTarget?: any): object;
}
```

通过上面的简单介绍，我们可以知道 JavaScript Proxy 提供了很多代理方法，这些方法被称为 `trap` 捕获器，这些 `trap` 都是可选参数，如果一个代理中未定义任何 `trap` 那么会保留代理对象的默认行为。

## 如何使用 Proxy

我们通过一个简单的实例来了解一下如何使用 proxy 来代理对象：

```ts
let Source = {
    world: 'hello'
};

let SourceProxy = new Proxy(Source, {
    get: function (target, p) {
        return p;
    }
});

console.log(Source.world, SourceProxy.world); // hello world
```

上面的代码中 Source 为源对象，SourceProxy 为代理后的新对象，通过调用两个对象中的 world 属性可以看到，Source.world 输出了 hello，而 SourceProxy.world 却输出了 world 字符串。

由此可见，当调用 SourceProxy 的对象中的 world 属性时，此 world 属性在 Proxy get 中被修改了。

通过上面这个例子我们已经知道如何简单的去使用 Proxy 进行代理。现在让我们在看一个例子：

```ts
class Source {
    constructor() {
        this.world = 'hello';
    }
}

let SourceProxy = new Proxy(Source, {
    get: function (target, p) {
        return p;
    }
});

console.log(new Source().world);
console.log(new SourceProxy().world);
```

先别急着看答案，简单花些时间思考一下上面这段代码会输出什么，并思考一下为什么会出现这种情况。

思考完了吗，现在我公布一下答案：

```ts
console.log(new Source().world);         // hello
console.log(new SourceProxy().world);    // TypeError: 'get' on proxy: property 'prototype' is a read-only and non-configurable data property on the proxy target but the proxy did not return its actual value (expected '#<Source>' but got 'prototype')
```

当我们调用 `console.log(new SourceProxy().world);` 属性时会发现给我们抛出了一个 ` TypeError: 'get' on proxy: property 'prototype' is a read-only and non-configurable data property on the proxy target but the proxy did not return its actual value (expected '#<Source>' but got 'prototype')` 异常信息。

让我们研究一下为为什么会出现这个问题，先简单回顾一下 `ProxyHandler` 中的 `trap` 有哪些，如果不记得可以在文章开头看看关于 ProxyHandler 的定义，我们会发现有一个 `construct` 方法。关于此方法的说明是 `为了使 new 操作符在生成的 Proxy 对象上生效，用于初始化代理的目标对象自身必须具有[[Construct]]内部方法（即 new target 必须是有效的）`，知道问题对症下药，只需要实现 `construct` 方法就好了。

修改完的代码是这样的：

```ts
class Source {
    constructor() {
        this.world = 'hello';
    }
}

let SourceProxy = new Proxy(Source, {
    construct: function (target, args) {
        return new target(...args)
    },
    get: function (target, p) {
        return p;
    }
});

console.log(new Source().world);      // hello
console.log(new SourceProxy().world); // hello
```

执行代码后我们发现 `get` 这个方法并没有被执行，如果你不理解为什么 `get` 方法没有执行，那我推荐你去了解一下 JavaScript getter setter 的相关知识。

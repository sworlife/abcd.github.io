---
title: Promises/A+ 规范
date: 2019-07-14 16:29:52
tags: 翻译
categories: 前端技术
comments: true
---

## Promises
[原文链接 - Promises](http://wiki.commonjs.org/wiki/Promises)

### Promises

Promises 提供一个定义良好的接口，用于与一个对象交互，该对象表示异步执行的操作的结果，在任何给定的时间点可以完成，也可以不完成。通过利用一个接口标准，不同部件可以返回异步操作的 promises，并且其消费者能以可预测的方式利用这个 promises 。Promises 也能提供基本实体，以便使用它们进行语法上更方便的语言级扩展，从而帮助实现异步性。

### 现有的技术
**略**

### 提案
- [Promises/A](http://wiki.commonjs.org/wiki/Promises/A) by Kris Zyp — "Thenables"
- [Promises/B](http://wiki.commonjs.org/wiki/Promises/B) by Kris Kowal — Opaque Q API
- [Promises/KISS](http://wiki.commonjs.org/wiki/Promises/KISS) by AJ ONeal
- Promises/C has been redacted
- [Promises/D](http://wiki.commonjs.org/wiki/Promises/D) by Kris Kowal — "Promise-sendables", for interoperable instances of Promises/B.

### 需求

- 1. 如果一个函数接受一个 promise 作为它的输入，那么函数本身必须返回一个 promise 作为它的输出。提供一个简单机制用以保证，处理程序员错误，非决定论和恶意：

- 1.1. 保护函数内部的一致性，使其避免来自提供输入 promise 的提供者的攻击
- 1.2. 保护函数内部的一致性，使其避免来自消费输出 promise 的消费者的攻击
- 1.3. 保护每个接受输出 promise 的接收方不受其他接收方的影响
- 1.4. 阻止把输入的 promise 泄露给输出消费者。表达式的返回值绝不能是输入 promise，也不能让消费者获取对输入promise 直接接的访问权

```
function API(input) {
    return when(input, function (input) {
    });
};
```
例如，一个密码校验 API 必须能持有对密码散列表的引用，但绝不能让使用者直接访问这些散列表中的任何一个。
```
function PasswordChecker(accounts) {
    return function (user, password) {
        var passwordHash = Q.get(accounts, user);
        return when(passwordHash, function (passwordHash) {
            if (hash(password) === passwordHash)
                return true;
        };
    };
}

// returns a password checking capability
// but does not return access to the passwordHash
return PasswordChecker(accounts);
```

- 1.4. 当输入解析后，如果发现还有额外未解析的输入，允许函数转发一个新的 promise
- 1.5. 允许函数要么转发其输入的失败，要么通过转发已解析的值从失败中恢复，要么通过转发一个新的承诺来恢复，最终解析或稍后失败。

- 2. 避免交叉危险
- 2.1. 防止 proimise 消费者在以下反例中直接使用 promise 提供者的回调:
```
var internalCounter = 0;

// /!\ HAZARD: untrusted promise given direct access to a
// function that has the capability to alter internal
// state, either during the same turn or in any future turn
promise("when", function () {
    internalCounter++;
});

// /!\ HAZARD: untrusted promise given direct access to a
// function that has the capability to alter internal
// state, either during the same turn or in any future turn
promise.when(function () {
    internalCounter++;
});

// /!\ HAZARD: untrusted promise given direct access to a
// function that has the capability to alter internal
// state, either during the same turn or in any future turn
promise.then(function () {
    internalCounter++;
});

ASSERT.equal(internalCounter, 0);
```

- 2.2. 防止 promise 提供者在调用消费者的内部回调时调用与注册时相同的轮数。
```
// /!\ HAZARD: untrusted promise given direct access to a
// function that has the capability to alter internal
// state, either during the same turn or in any future turn
var internalCounter = 0;
when(promise, function () {
    internalCounter++;
});
ASSERT.equal(internalCounter, 0);
```

- 2.3. 防止 promise 提供者多次调用消费者内部回调或者错误回调。
```
var internalCounter = 0;
when(promise, function (value) {
    internalCounter++;
}, function (error) {
    internalCounter++;
});
setTimeout(function () {
    ASSERT.ok(internalCounter == 0 || internalCounter == 1);
}, forever);
```

- 3. 将“解决”与“观察解决”的关注点分开。

- 3.1. 允许API安全地生成“延迟”，保留“解析器”部分，并将“promise”组件分发给任意数量的相互怀疑的消费者。

- 3.2. “竞争性竞争”:允许API安全地生成“延迟”，保留“承诺”部分，并将“解析器”组件分发给任何数量的相互怀疑的消费者。

- 4. 提供一种机制，允许承诺使用者在可疑承诺尝试多次解析的任何情况下抛出异常。

- 5. 提供对任意消息传递到承诺的支持，包括支持拦截没有显式提供处理程序的任意消息。

### 相关讨论
**略**

## Promises/A
[原文链接 - Promises/A](http://wiki.commonjs.org/wiki/Promises/A)

### 提议

`promise` 相当于单个操作完成后返回的最终的结果。`promise` 的状态可能是 `unfulfilled , fulfilled , failed `三种中的一个。

它或许只能从 `unfulfilled ` 状态迁移到 `fulfilled ` 状态，或者从`unfulfilled `状态迁移到 `failed `状态。

一旦 `promise` 处于`fulfilled ` 或者 `failed ` 状态，这个 `promise` 的值必须不能被修改，仅作为一个值，存在于 JavaScript 中，就像“原语”（primitives）和“对象标识”（object identities）一样，不能改变（尽管对象本身可能总是可变的，但对象标识不会。）。

这中不可改变的特性于 `promise` 来说很重要，它既能避免来自监听者的副作用，监听者在其行为中能创造意想不到的变化；也允许 `promises` 被传递给其他函数而不影响其调用者，这与“原语”被传递给函数而不用担心其调用者的变量被该函数的执行改变的方式一样。

这个 API 没有定义如何创建 `promises`，而仅定义了 `promise` 必须提供必要的接口，`promise` 的消费者通过它来交流信息。实现者可以自由定义如何创建 `promises`。一些 `promises` 可以提供自己的函数来实现 `promise`，而其他 `promises` 则可能通过一种对 `promise` 消费者不可见的机制来实现。请参阅[Promise Manager] API以获得有用的方便函数的建议。

承诺本身也可能包括其他额外的便利方法。例如，可以在承诺(如addCallback和addBoth)上实现Dojo的延迟函数。

promise被定义为一个对象，它有属性 `then`，且其值为函数 :

**then(fulfilledHandler, errorHandler, progressHandler)**
为完成的 `promise` 添加将要被调用的 `fulfilledHandler`、`errorHandler` 和 `progressHandler` 回调函数。当 `promise` 被实现时，调用 `fullledhandler`。当 `promise` 失败时调用 `errorHandler`。`progressHandler` 用于调用进度事件。所有参数都是可选的，非函数值将被忽略。`progressHandler` 不仅是一个可选参数，而且进度事件完全是可选的。`Promise` 实现者不需要调用 `progressHandler` (progressHandler可以被忽略)，这个参数的存在使得实现者可以调用它，如果他们有进度事件要报告的话。

这个函数应该返回一个新的 `promise`，这个 `promise` 将在给定的 `fulfilledHandler` 或 `errorHandler`回调函数条件下完成实现。并允许将 `promise` 通过链式操作链接在一起。回调处理程序返回的值是返回的 `promise` 的实现值。如果回调引发错误，返回的 `promise` 将被移动到failed状态。

使用CommonJS兼容承诺的一个例子:
```
 asyncComputeTheAnswerToEverything().
    then(addTwo).
    then(printResult, onError);
  44
```

可交互的 `promise` 是一种扩展的 `promise`，支持以下附加功能：

**get(propertyName)**
从此承诺的目标请求给定的属性。返回一个承诺，以提供来自此承诺的目标的声明属性的值。

**call(functionName, arg1, arg2, ...)**
请求在此承诺的目标上调用给定的方法/函数。返回一个承诺，以提供所请求函数调用的返回值。

### Open Issues
对回调处理程序的调用应该放在事件队列中还是立即执行?-目前还不确定，我也不确定什么是最好的。

### 实现
略...


## Promises/A+
[原文链接](https://promisesaplus.com)

一个开放的标准，为可交互的 Javascript promises 而发声。                    —— 实现者，为实现者

`promise` 表示异步操作的最终结果。与 `promise` 交互的主要方式是通过其 `then` 方法，该方法注册回调以接收承诺的最终值或无法实现承诺的原因。

该规范详细描述了then方法的行为，提供了一个可互操作的基础，所有承诺/A+符合约定的承诺实现都可以依赖于该基础来提供。因此，应该认为该规范非常稳定。虽然 `promise /A+` 组织可能偶尔会修改这个规范，进行少量向后兼容的更改来处理新发现的角用例，但是只有在仔细考虑、讨论和测试之后，我们才会集成大型的或向后不兼容的更改。

从历史上看，`promise /A+` 阐明了早期[promise /A proposal](http://wiki.commonjs.org/wiki/Promises/A)的行为条款，将其扩展到涵盖事实行为，并省略了未指定或有问题的部分。

最后，核心 `promist/A+` 规范不处理如何创建、实现或拒绝承诺，而是选择专注于提供一个互操作的then方法。将来在配套规范中的工作可能会涉及这些主题。

### 1. 术语

1.1. “promise” 是一个带有 `then` 方法的对象或者函数，它的行为遵从改规范
1.2. “thenable” 是一个定义 `then` 方法的对象或者函数
1.3. “value” 是一个任意 Javascript 法定的值（包括 `undefined`, `thenable`, `promise`）
1.4. “exception” 是一个使用 `throw` 声明抛出的值
1.5. “reason” 是一个表示 `promise` 被拒绝的原因的值

### 2. 需求
#### 2.1. Promise 状态

一个 `promise` 必须处于以下三种状态中的一种：`pending`, `fulfilled`, `rejected`

2.1.1. `pending` 状态时，`promise`：
- 2.1.1.1. 可能转变为 `fulfilled` 或者 `rejected` 状态

2.1.2. `fulfilled` 状态时，`promise`：
- 2.1.2.1. 绝不能再转变为其他任何状态
- 2.1.2.2. 必须有 `value`，且不可改变

2.1.3. `rejected` 状态时，`promise`:
- 2.1.3.1. 绝不能再转变为其他任何状态
- 2.1.3.2. 必须有 `reason`, 且不可改变

这儿，“不可改变” 意思是不变的身份（即：`===`）, 但并不意味着深层次的不变性

#### 2.2. `then` 方法

`promise` 必须提供 `then` 方法来访问当前或最终的 `value` 或者 `reason` 

`promise` 的 `then` 方法接受两个参数：
```
promise.then(onFulfilled, onRejected)
```

2.2.1. `onFulfilled` 和 `onRejected` 都是可选参数：
- 2.2.1.1. 如果 `onFulfilled` 不是函数，则必须忽略
- 2.2.1.2. 如果 `onRejected` 不是函数，则必须忽略

2.2.2. 如果 `onFulfilled` 是函数：
- 2.2.2.1 它必须在 `promise` 实现之后被调用，把 `promise` 的 `value` 作为其第一个参数
- 2.2.2.2. 它绝不能在 `promise` 实现之前被调用
- 2.2.2.3. 它绝不能调用多次

2.2.3. 如果 `onRejected` 是函数：
- 2.2.3.1. 它必须在 `promise` 拒绝之后被调用，把 `promise` 的 `reason` 作为其第一个参数
- 2.2.3.2. 它绝不能在 `promise` 拒绝之前被调用
- 2.2.3.3.  它绝不能调用多次

2.2.4. 在上下文堆栈中只包含**平台代码**之前，绝不能调用 `onFulfilled` 或 `onRejected` [3.1]

2.2.5. `onFulfilled` 或 `onRejected` 只能作为函数被调用。[3.2]

2.2.6. `then` 可能被同一个 `promise` 多次调用。
- 2.2.6.1. 如果/当 `promise` 是完成的，所有各自的 `onFulfilled` 回调必须按照初始调用 `then` 的顺序执行。
- 2.2.6.2. 如果/当 `promise` 是拒绝的，所有各自的 `onRejected` 回调必须按照初始调用 `then` 的顺序执行。

2.2.7. `then` 必须返回一个 `promise`。
```
promise2 = promise1.then(onFulfilled, onRejected);
```
- 2.2.7.1. 如果 `onFulfilled ` 或者 `onRejected ` 返回的 `value` 为 `x`，则运行 `Promise` 决议子程序 `[[Resolve]](promise2, x)`。
- 2.2.7.2. 如果 `onFulfilled ` 或者 `onRejected ` 抛出一个异常 `e`, `promise2` 必须把 `e` 作为 `reason` 进行拒绝。
- 2.2.7.3. 如果 `onFulfilled` 不是函数，并且 `promise1` 已经完成，`promise2` 必须以与 `promise1` 相同的 `value` 来完成
- 2.2.7.4. 如果 `onRejected ` 不是函数，并且 `promise1` 已经拒绝，`promise2` 必须以与 `promise1` 相同的 `reason` 来拒绝

#### 2.3. Promise Resolution 子程序
Promise Resolution 子程序是以 `promise` 和 `value` 为输入的抽象操作，表示为：`[[Resolve]](promise, x)`。如果 `x` 是 `thenable`，它企图使 `promise` 采用 `x` 的状态，在这一假定下，`x` 行为至少有点像 `promise`。否则，它将用 `x` 作为 `value` 来完成 `promise`。

`thenables` 的这种处理运行 `promise` 实现互操作，只要他们公开 `Promise/A+` 兼容的 `then` 方法。它也允许 `Promise/A+` 实现使用合理的 `then` 方法“吸收”不一致的实现。

运行 `[[Resolve]](promise, x)`，执行以下步骤：

2.3.1. 如果 `promise` 和 `x` 指向同一个对象，则以 `TypeError` 为由拒绝 `promise`

2.3.2. 如果 `x` 是一个 `promise`, 则采用它的状态 [3.4]：
- 2.3.2.1. 如果 `x` 状态为 `pending`，`promise` 必须保持 `pending`  状态，直到 `x` 被完成或者拒绝。
- 2.3.2.2. 如果/当 `x` 被完成，则用相同的 `value` 完成 `promise`
- 2.3.2.3. 如果/当 `x` 被拒绝，则用相同的 `reason` 拒绝 `promise`

2.3.3. 否则，如果 `x` 是一个对象或者函数，
- 2.3.3.1. 让 `then` 为 `x.then`. [3.5]
- 2.3.3.2. 如果检索属性 `x.then` 导致抛出异常 `e`，则以 `e` 为由拒绝 `promise`
- 2.3.3.3. 如果 `then` 是函数，用 x 做上下文 `this`,  `resolvePromise` 做第一个参数，`rejectPromise` 做第二个参数来执行它，这儿：
    - 2.3.3.3.1. 如果/当 `resolvePromise` 被调用时，它的 `value` 是 `y`，运行 `[[Resolve]](promise, y)`.
    - 2.3.3.3.2. 如果/当 `rejectPromise` 被调用时，它的 `reason` 时 `r`，则以 `r` 理由拒绝 `promise`.
    - 2.3.3.3.3. 如果同时调用resolvePromise和rejectPromise，或者对同一个参数进行多个调用，那么第一个调用优先，任何进一步的调用都被忽略。
    - 2.3.3.3.4. 如果调用 `then` 抛出异常 `e`,
        - 2.3.3.3.4.1. 如果 `resolvePromise ` 和 `rejectPromise ` 都被调用，则忽略。
        - 2.3.3.3.4.2. 否则以 `e` 为理由拒绝 `promise`。
- 2.3.3.4. 如果`then` 不是函数，用 `x` 完成 `promise`.

2.3.4. 如果 `x` 不是对象或者函数，则用 `x` 完成 `promise`

如果 `promise`以参与循环的 thenable 链作为 `value` 来完成，使得 `[[Resolve]](promise, thenable)` 的递归性质最终导致再次调用 `[[Resolve]](promise, thenable)`，那么按照上面的算法将导致无限递归。我们鼓励实现(但不是必需的)检测这种递归并拒绝承诺，理由是信息类型错误。[3.6]

#### 3. 备注

3.1. 这儿所说的“平台代码”是指引擎、环境、和 `promise` 的代码实现。在实践中，这个需求确保 `onFulfilled` 和 `onRejected ` 在调用 `then` 的事件循环之后异步执行，和一个新的栈。这可以通过“宏任务”机制(如setTimeout或setimmediation)实现，也可以通过“微任务”机制(如MutationObserver或process.nextTick)实现。由于 `promise` 实现被认为是平台代码，所以它本身可能包含一个任务调度队列或“蹦床”，在其中调用处理程序。

3.2. 也就是说，在严格模式下，这在它们内部是没有定义的;在非严格模式下，它将是全局对象。

3.3. 实现可能允许 `promise2 === promise1`，前提是实现满足所有需求。每个实现都应该记录它是否可以生成 `promise2 === promise1`，以及在什么条件下生成。

3.4. 通常，只有当 `x` 来自当前实现时才知道它是一个真正的 `promise`。此子句允许使用特定于实现的方法来采用已知一致性 promises 的状态。

3.5. 这个过程首先存储对AA的引用，然后测试该引用，然后调用该引用，从而避免了对AA属性的多次访问。这些预防措施对于确保访问器属性的一致性非常重要，因为访问器属性的值在检索之间可能会发生变化。

3.6. 实现不应该对 thenable 链的深度设置任意的限制，并且假设超过这个任意的限制，递归将是无限的。只有真正的死循环才会导致 `TypeError `; 如果遇到一个由不同 thenable 组成的无限链，则永远递归是正确的行为。

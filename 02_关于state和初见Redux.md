```
// Tutorial 02 - about-state-and-meet-redux.js
```

## 02 - 关于 state 和初见 Redux

```
// Sometimes the actions that we'll handle in our application will not only inform us
// that something happened but also tell us that data needs to be updated.
```

在我们的程序中，有时候我们要处理的 actions 不仅能够通知我们会发生什么事, 同时也能告诉我们哪些数据会被更新.

```
// This is actually quite a big challenge in any app.
// Where do I keep all the data regarding my application along its lifetime?
// How do I handle modification of such data?
// How do I propagate modifications to all parts of my application?
```
事实上对于任何的程序, 这都是一个相当大的挑战.
- 我该在什么地方沿着程序中所有数据的生命周期来保存他们呢?
- 我该怎样处理这些数据的修改呢？
- 我该如何将修改消息通知到我程序中的其它部分呢？

```
// Here comes Redux.
```
这时 Redux 出现了.

```
// Redux (https://github.com/rackt/redux) is a "predictable state container for JavaScript apps"
```
Redux 是一个 "JavaScript应用程序的可预测状态容器".

```
// Let's review the above questions and reply to them with
// Redux vocabulary (flux vocabulary too for some of them):
```
我们先回顾一下上述问题并用 Redux 来回答:

```
// Where do I keep all the data regarding my application along its lifetime?
//     You keep it the way you want (JS object, array, Immutable structure, ...).
//     Data of your application will be called state. This makes sense since we're talking about
//     all the application's data that will evolve over time, it's really the application's state.
//     But you hand it over to Redux (Redux is a "state container", remember?).
// How do I handle data modifications?
//     Using reducers (called "stores" in traditional flux).
//     A reducer is a subscriber to actions.
//     A reducer is just a function that receives the current state of your application, the action,
//     and returns a new state modified (or reduced as they call it)
// How do I propagate modifications to all parts of my application?
//     Using subscribers to state's modifications.
```
* 我该在什么地方沿着程序中所有数据的生命周期来保存他们呢?
  - 你可以用任何的方式保存程序中的数据 (比如: JS对象, 数组, 不可变结构体, ...).
  - 你程序中的数据被称为 state . 这个称谓合情合理, 因为程序中所有我们提到的数据都会随着时间而演变, 这其实就是程序的 state.
  - 只不过你将他们交给了 Redux (Redux 是一个 "state container", 还记得吗？).
* 我该怎样处理这些数据的修改呢？
  - 使用 reducers (在传统的 Flux 中被称为 "stores" ).
  - 一个 reducer 就是 actions 的一个 subscriber.
  - reducer 只不过是一个函数, 该函数可以接受你程序当前的 state 和 action , 并返回一个新的被修改后的 state .
* 我该如何将修改消息通知到我程序中的其它部分呢？
  - 使用对于 state 修改的 subscribers.

```
// Redux ties all this together for you.
// To sum up, Redux will provide you:
//     1) a place to put your application state
//     2) a mechanism to dispatch actions to modifiers of your application state, AKA reducers
//     3) a mechanism to subscribe to state updates
```
Redux 为你捆绑了这一切. 总之, Redux 将为你提供:
* 一个存放你程序 state 的地方
* 一个将 action 分发给你程序 state 修改者的机制, 又名 reducers
* 一个 state 更新的订阅机制

```js
// The Redux instance is called a store and can be created like this:
// Redux实例被称为Store, 可以这样创建:

    import { createStore } from 'redux'
    var store = createStore()
```

```
// But if you run the code above, you'll notice that it throws an error:
```
但如果运行上面的代码, 你将发现它会抛出一个异常:
```js
     Error: Invariant Violation: Expected the reducer to be a function.
```

```
// That's because createStore expects a function that will allow it to reduce your state.
```
这是因为 **createStore** 需要传入一个函数用于改变 (reduce) 你程序的 state

```
// Let's try again
```
让我们再试一试

```js
import { createStore } from 'redux'

var store = createStore(() => {})
```

```
// Seems good for now...
```
现在看起来没问题了...


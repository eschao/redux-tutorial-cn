```
// Tutorial 09 - middleware.js
```

## 09 - 中间件 middleware

```
// We left dispatch-async-action-2.js with a new concept: "middleware". Somehow middleware should help us
// to solve async action handling. So what exactly is middleware?
```
在上一节中我们留下了一个新的概念: **"中间件 middleware"**. 不管怎样 middleware 可以帮助我们解决异步 action 的处理问题. 那么到底什么是 middleware?

```
// Generally speaking middleware is something that goes between parts A and B of an application to
// transform what A sends before passing it to B. So instead of having:
// A -----> B
// we end up having
// A ---> middleware 1 ---> middleware 2 ---> middleware 3 --> ... ---> B
```
总的来说 middleware 是一种运行在程序 A 部分和 B 部分之间的组件, 该组件用于将 A 发送给 B 的数据在传送给 B 之前先做一些转换. 

因此程序流程将不再是:

  **A -----> B**

而最终会是:

  **A ---> middleware 1 ---> middleware 2 ---> middleware 3 ---> ... ---> B**

```
// How could middleware help us in the Redux context? Well it seems that the function that we are
// returning from our async action creator cannot be handled natively by Redux but if we had a
// middleware between our action creator and our reducers, we could transform this function into something
// that suits Redux:
```
在 Redux 的上下文中 middleware 是如何帮助我们的呢？好吧, 看起来从我们异步 action creator 中返回的函数是不能够直接地被 Redux 处理, 但如果有一个 middleware 位于我们的 action creator 和 reducers 之间, 我们就能够将该函数转换成适合 Redux 的东西:

```
// action ---> dispatcher ---> middleware 1 ---> middleware 2 ---> reducers
```
**action ---> dispatcher ---> middleware 1 ---> middleware 2 ---> reducers**

```
// Our middleware will be called each time an action (or whatever else, like a function in our
// async action creator case) is dispatched and it should be able to help our action creator
// dispatch the real action when it wants to (or do nothing - this is a totally valid and
// sometimes desired behavior).
```
我们的 middleware 在每次 action (或者别的东西, 比如我们异步 action creator 中的函数)被分发的时候都会被调用,
并且它能够在需要的时候帮助我们的 action creator 去分发一个真实的 action (也或者什么也不做 - 这是一个有效行为, 并且有时候也是我们所期望的行为)

```
// In Redux, middleware are functions that must conform to a very specific signature and follow
// a strict structure:
```
在 Redux 中, middleware 就是一些必须遵循特殊签名和严格结构的函数:

```js
    var anyMiddleware = function ({ dispatch, getState }) {
        return function(next) {
            return function (action) {
                // your middleware-specific code goes here
            }
        }
    }
```

```
// As you can see above, a middleware is made of 3 nested functions (that will get called sequentially):
// 1) The first level provides the dispatch function and a getState function (if your
//     middleware or your action creator needs to read data from state) to the 2 other levels
// 2) The second level provides the next function that will allow you to explicitly hand over
//     your transformed input to the next middleware or to Redux (so that Redux can finally call all reducers).
// 3) the third level provides the action received from the previous middleware or from your dispatch
//     and can either trigger the next middleware (to let the action continue to flow) or process
//     the action in any appropriate way.
```
正如你在上面所看到的, 一个 middleware 由3个内嵌的函数组成(他们将顺序地被调用):
* 第一层为其他两层提供一个 dispatch 函数和 getState 函数(如果你的 middleware 或 action creator 需要从 state 中读取数据)
* 第二层会提供一个 next 函数, 该函数允许你显示地将转换后的输入交给下一层 middleware 或 Redux(以便 Redux 能最终调用到所有的 reducers ).
* 第三层提供一个 action 参数, 该参数来自于前一个 middleware 或你自己的 dispatch 函数, 然后要么触发下一个 middleware (以便 action 能继续的向前流动) 要么正确地处理掉该 action .

```
// Those of you who are trained to functional programming may have recognized above an opportunity
// to apply a functional pattern: currying (if you aren't, don't worry, skipping the next 10 lines
// won't affect your Redux understanding). Using currying, you could simplify the above function like that:
```
那些具有函数编程背景的人可能意会识到有机会将一个函数编程模式应用到上述描述中: currying 柯里化 (如果你不知道，也不用担心，跳过下面10行内容并不会影响你对 Redux 的理解). 使用 currying, 就能可以将上述函数简化为:

```js
    // "curry" may come from any functional programming library (lodash, ramda, etc.)
    var thunkMiddleware = curry(
        ({dispatch, getState}, next, action) => (
            // your middleware-specific code goes here
        )
    );
```

```
// The middleware we have to build for our async action creator is called a thunk middleware and
// its code is provided here: https://github.com/gaearon/redux-thunk.
// Here is what it looks like (with function body translated to es5 for readability):
```
我们为异步 action creator 所构建的 middleware 被称为 thunk middleware , 可以从 https://github.com/gaearon/redux-thunk 获取其代码.

我们的代码看起来就变成了这样(为了易于阅读, 函数内部已转换成ES5的语法了):

```js
var thunkMiddleware = function ({ dispatch, getState }) {
    // console.log('Enter thunkMiddleware');
    return function(next) {
        // console.log('Function "next" provided:', next);
        return function (action) {
            // console.log('Handling action:', action);
            return typeof action === 'function' ?
                action(dispatch, getState) :
                next(action)
        }
    }
}
```

```
// To tell Redux that we have one or more middlewares, we must use one of Redux's
// helper functions: applyMiddleware.
```
为了告诉 Redux 我们拥有一个或者多个 middleware , 我们必须使用 Redux 提供的辅助函数: applyMiddleware.

```
// "applyMiddleware" takes all your middlewares as parameters and returns a function to be called
// with Redux createStore. When this last function is invoked, it will produce "a higher-order
// store that applies middleware to a store's dispatch".
// (from https://github.com/rackt/redux/blob/v1.0.0-rc/src/utils/applyMiddleware.js)
```
**applyMiddleware** 接受你所有的 middlewares 作为参数并返回一个能被 Redux 的 createStore 调用的函数. 当该函数被调用的时候,
它将生成一个"高阶 store 用于将 middleware 应用到 store 的 dispatch 函数上".

```js
// Here is how you would integrate a middleware into your Redux store:
// 下面展示了如何将middleware整合到你的Redux store中:

import { createStore, combineReducers, applyMiddleware } from 'redux'

const finalCreateStore = applyMiddleware(thunkMiddleware)(createStore)
// For multiple middlewares, write: applyMiddleware(middleware1, middleware2, ...)(createStore)
// 对于多个middleware组件, 可以这样调用: applyMiddleware(middleware1,
middleware2, ...)(createStore)

var reducer = combineReducers({
    speaker: function (state = {}, action) {
        console.log('speaker was called with state', state, 'and action', action)

        switch (action.type) {
            case 'SAY':
                return {
                    ...state,
                    message: action.message
                }
            default:
                return state
        }
    }
})

const store_0 = finalCreateStore(reducer)
```
```
// Output:
//     speaker was called with state {} and action { type: '@@redux/INIT' }
//     speaker was called with state {} and action { type: '@@redux/PROBE_UNKNOWN_ACTION_s.b.4.z.a.x.a.j.o.r' }
//     speaker was called with state {} and action { type: '@@redux/INIT' }
```
运行后输出:
```js
     speaker was called with state {} and action { type: '@@redux/INIT' }
     speaker was called with state {} and action { type: '@@redux/PROBE_UNKNOWN_ACTION_s.b.4.z.a.x.a.j.o.r' }
     speaker was called with state {} and action { type: '@@redux/INIT' }
```

```js
// Now that we have our middleware-ready store instance, let's try again to dispatch our async action:
// 这样我们就有了一个包含middleware的store实例,
接下来我们试着再次分发我们的异步action:

var asyncSayActionCreator_1 = function (message) {
    return function (dispatch) {
        setTimeout(function () {
            console.log(new Date(), 'Dispatch action now:')
            dispatch({
                type: 'SAY',
                message
            })
        }, 2000)
    }
}

console.log("\n", new Date(), 'Running our async action creator:', "\n")

store_0.dispatch(asyncSayActionCreator_1('Hi'))
```
```
// Output:
//     Mon Aug 03 2015 00:01:20 GMT+0200 (CEST) Running our async action creator:
//     Mon Aug 03 2015 00:01:22 GMT+0200 (CEST) 'Dispatch action now:'
//     speaker was called with state {} and action { type: 'SAY', message: 'Hi' }
```
运行后输出:
```
     Mon Aug 03 2015 00:01:20 GMT+0200 (CEST) Running our async action creator:
     Mon Aug 03 2015 00:01:22 GMT+0200 (CEST) 'Dispatch action now:'
     speaker was called with state {} and action { type: 'SAY', message: 'Hi' }
```

```
// Our action is correctly dispatched 2 seconds after our call the async action creator!
```
在调用异步 action creator 2秒后，我们的 action 被正确地分发了.

```
// Just for your curiosity, here is how a middleware to log all actions that are dispatched, would
// look like:
```
出于你的好奇, 下面这个 middleware 说明了如何记录所有被分发的 action:

```js
function logMiddleware ({ dispatch, getState }) {
    return function(next) {
        return function (action) {
            console.log('logMiddleware action received:', action)
            return next(action)
        }
    }
}
```

```
// Same below for a middleware to discard all actions that are dispatched (not very useful as is
// but with a bit of more logic it could selectively discard a few actions while passing others
// to next middleware or Redux):
```
```js
function discardMiddleware ({ dispatch, getState }) {
    return function(next) {
        return function (action) {
            console.log('discardMiddleware action received:', action)
        }
    }
}
```
同样, 下面这个 middleware 用于丢弃所有被分发的 actions (如下这样不是很有用, 但多一点逻辑的话, 它就可以在将 actions 传入下一个 middleware 或者Redux 的时候选择性地丢弃一些 actions) 
```
// Try to modify finalCreateStore call above by using the logMiddleware and / or the discardMiddleware
// and see what happens...
// For example, using:
//     const finalCreateStore = applyMiddleware(discardMiddleware, thunkMiddleware)(createStore)
// should make your actions never reach your thunkMiddleware and even less your reducers.
```
试着用上面的 logMiddleware 和 discardMiddleware 去修改一下 finalCreateStore 的调用, 然后看看会发生什么...

比如这样调用:
```js
     const finalCreateStore = applyMiddleware(discardMiddleware, thunkMiddleware)(createStore)
```
这将导致你的 actions 永远也运行不到 thunkMiddleware 甚至你的一些 reducers 函数.

```
// See http://redux.js.org/docs/introduction/Ecosystem.html#middleware, section Middleware, to
// see other middleware examples.
```
在 Redux 文档的 [Middleware](http://redux.js.org/docs/introduction/Ecosystem.html#middleware) 这一节中，你可以看到一些其他的 middleware 例子.

```
// Let's sum up what we've learned so far:
// 1) We know how to write actions and action creators
// 2) We know how to dispatch our actions
// 3) We know how to handle custom actions like asynchronous actions thanks to middlewares
```
总结一下我们目前已学到的:
* 我们知道了如何编写一个 actions 以及 action creator
* 我们知道了如何分发我们的 actions
* 我们知道了如何的处理定制的 actions , 比如像异步 actions 等，当然这都要感谢 middleware 这个好东西.

```
// The only missing piece to close the loop of Flux application is to be notified about
// state updates in order to react to them (by re-rendering our components for example).
```
对于 Flux 程序的完整闭环, 唯一所缺的一部分就是有关 state 更新的通知以便对其做出响应 (比如 UI 组件的重绘)



title: react flux 入门教程
date: 2015-09-04 20:55:29
categories: react  
tags: ['react','javascript','前端','flux']  
description:
---
react flux 入门实例-todo  
<!--more-->
#### 什么是flux
*flux* 是一个由*facebook*创造出来的术语,是*facebook* 使用的一套前端应用的架构模式. 用来描述单向的数据流的设计模型. 这里的*flux*不是指一定要使用*flux*库, 不过的确需要*flux dispathcer*以及事件库.   
##### *flux* 应用的四个部分
- *the dispatcher*  
处理动作分发，维护 *store* 之间的依赖关系
- *the stores*  
数据和逻辑部分
- *the views*  
*react* 组件，这一层可以看作 *controller-views*，作为视图同时响应用户交互
- *the actions*  
提供给 *dispatcher* 传递数据给 *store*

##### 单向数据流
![flux](http://hulufei.gitbooks.io/react-tutorial/content/image/flux-overview.png)
整个流程如下：
- 首先要有 *action*，通过定义一些 *action creator* 方法根据需要创建 *Action* 提供给 *dispatcher*
- *View* 层通过用户交互会触发 *Action*
- *Dispatcher* 会分发触发的 *Action* 给所有注册的 *Store* 的回调函数
- *Store* 回调函数根据接收的 *Action* 更新自身数据之后会触发一个 *change* 事件通知 *View* 数据更改了
- *View* 会监听这个 *change* 事件，拿到对应的新数据并调用 *setState* 更新组件 UI  

所有的状态都由 *Store* 来维护，通过 *Action* 传递数据，构成了如上所述的单向数据流循环，所以应用中的各部分分工就相当明确，高度解耦了。  
这种单向数据流使得整个系统都是透明可预测的。

#### flux核心概念
##### dispather  
一个应用只需要一个 *dispatcher* 作为分发中心，管理所有数据流向，分发动作给 *store*，没有太多其他的逻辑.  
*dispatcher* 分发动作给 *store* 注册的回调函数，这和一般的订阅/发布模式不同的地方在于：
- 回调函数不是订阅到某一个特定的事件/频道，每个动作会分发给所有注册的回调函数
- 回调函数可以指定在其他回调之后调用

*dispatcher.js*提供的 API 很简单：  
- *register(function callback): string*   
注册回调函数，返回一个 *token* 供在*waitFor()* 使用
- *unregister(string id): void*   
通过 *token* 移除回调
- *waitFor(array ids): void*  
在指定的回调函数执行之后才执行当前回调。这个方法只能在分发动作的回调函数中使用
- *dispatch(object payload): void*  
分发动作 *payload* 给所有注册回调
- *isDispatching(): boolean*  
返回 *dispatcher* 当前是否处在分发的状态

##### stores
*stores* 包含应用的状态和逻辑. *stores* 管理着数据，接收数据的方法和 *dispatcher* 的回调函数。  
*store*在*dispatcher*注册(*AppDispatcher.register*)并提供*dispatcher*一个回调函数, 这个回调函数接受*action*作为参数. 在*store*的已注册的回调函数内, 有一个*switch*声明, 根据*action type*来提供进入不同*store*内部函数的钩子. 这允许了一个*action*可以通过*dispatcher*来更新*store*中的*state*. 待*state*更新后, *store*广播一个事件来告诉大家*state*已经更新啦, 然后*views*就可以获取新的*state*并更新自己.  
- 一个 *store* 除了包含一個数据模型，还包含获取和更新数据的方法。
- *store* 在程序中是唯一知道該如何更新数据的角色。
- 一个 *store* 只负责处理单一需求。例如如果有其他部分需要关天图片的数据就应该再创建一个 *store* 并叫做*ImageStore*。
- *store* 里面不会暴露直接操作数据的方法给外部，暴露给外部调用的方法都是 *getter* 方法，没有 *setter* 方法，唯一更新数据的手段就是通过在 *dispatcher* 注册的回调函数。

##### views
*view* 就是 *react* 组件，从 *store* 获取状态（数据），绑定 *change* 事件处理。
大型的系统中应用*flux*的情况
![flux](https://dn-linx4200.qbox.me/img/2015/01/complex-Flux-data-flow.jpg)

#### 实例用到的三方库介绍
##### keymirror
JS的仿enum的函数库  
输入:
```javascript
var keyMirror = require('react/lib/keyMirror');
module.exports = keyMirror({
  LOAD_SHOES: null
});
```
输出:  
```javascript
{ LOAD_SHOES: 'LOAD_SHOES' }
```
映射key, 值就等于key
##### object-assign
合并对象.   
```javascript
//格式
objectAssign(target, source, [source, ...])
```
如果source中的key重复, 后面的会把前面的覆盖掉.
##### EventEmitter
事件触发器.
```javascript
emitter.on(event, listener) //向指定的事件监听器数组尾部添加一个新监听器。
emitter.removeListener(event, listener) //从指定监听器数组中删除一个监听器。需要注意的是，此操作将会改变处于被删监听器之后的那些监听器的索引。
emitter.emit(event, [arg1], [arg2], [...]) //使用所提供的参数，依次执行事件监听器数组中的每一个监听函数。
```
##### classnames
将className组合起来.
```javascript
classNames({foo:true}, {bar:false}, {duck:true}); //=>'foo duck'
```
#### 运行实例
[todo source](https://github.com/boomya/react/tree/master/flux-todo)
```
npm install -g gulp
npm install 
gulp serve
```
访问 http://localhost:5000/
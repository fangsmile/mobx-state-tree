# mobx-state-tree
理念和概述
mobx-state-tree is a state container that combines the simplicity and ease of mutable data with the traceability of immutable data and the reactiveness and performance of observable data.
mobx-state-tree 是一个状态容器，它将可变数据的简单性和易变性与不可变数据的可追溯性以及可观察数据的响应性和性能结合起来。
Simply put, mobx-state-tree tries to combine the best features of both immutability (transactionality, traceability and composition) and mutability (discoverability, co-location and encapsulation) based approaches to state management; everything to provide the best developer experience possible. Unlike MobX itself, mobx-state-tree is very opinionated on how data should be structured and updated. This makes it possible to solve many common problems out of the box.
简单地说，mobx-state-tree试图将不可变性(事务性、可跟踪性和组合性)和可变性(可发现性、协同定位和封装性)的最佳特性结合到状态管理中;一切都能提供最好的开发人员体验。与MobX不同的是，mobx-state-tree对数据应该如何进行结构化和更新非常有主见。这使得解决许多常见的问题成为可能。
Central in MST (mobx-state-tree) is the concept of a living tree. The tree consists of mutable, but strictly protected objects enriched with runtime type information. In other words; each tree has a shape (type information) and state (data). From this living tree, immutable, structurally shared, snapshots are generated automatically.
MST (mobx-state-tree)中的中心是一棵活树的概念。树由可变的、但严格受保护的对象组成，这些对象包含了运行时类型信息。换句话说;每棵树都有一个形状(类型信息)和状态(数据)。从这个活的树中，不可变的、结构上共享的快照是自动生成的。

```javascript
import { types, onSnapshot } from "mobx-state-tree"

const Todo = types.model("Todo", {
    title: types.string,
    done: false
}).actions(self => ({
    toggle() {
        self.done = !self.done
    }
}))

const Store = types.model("Store", {
    todos: types.array(Todo)
})

// create an instance from a snapshot
const store = Store.create({ todos: [{
    title: "Get coffee"
}]})

// listen to new snapshots
onSnapshot(store, (snapshot) => {
    console.dir(snapshot)
})

// invoke action that modifies the tree
store.todos[0].toggle()
// prints: `{ todos: [{ title: "Get coffee", done: true }]}`
```

By using the type information available, snapshots can be converted to living trees, and vice versa, with zero effort. Because of this, time travelling is supported out of the box, and tools like HMR are trivial to support example.
通过使用可用的类型信息，快照可以转换为活的树，反之亦然。正因为如此，历史追溯是被支持的，而像HMR这样的工具就是支持范例。
The type information is designed in such a way that it is used both at design- and run-time to verify type correctness (Design time type checking works in TypeScript only at the moment; Flow PR's are welcome!)
类型信息的设计方式是在设计和运行时都使用它来验证类型的正确性(设计时类型检查只在当前的TypeScript中进行;)
```
[mobx-state-tree] Value '{\"todos\":[{\"turtle\":\"Get tea\"}]}' is not assignable to type: Store, expected an instance of Store or a snapshot like '{ todos: { title: string; done: boolean }[] }' instead.
```

_Runtime type error_

![typescript error](https://github.com/mobxjs/mobx-state-tree/raw/master/docs/tserror.png)

_Designtime type error_

Because state trees are living, mutable models, actions are straight-forward to write; just modify local instance properties where appropriate. See `toggleTodo()` above or the examples below. It is not necessary to produce a new state tree yourself, MST's snapshot functionality will derive one for you automatically.
因为状态树是活的，可变的模型，操作是直接的;只需在适当的地方修改本地实例属性。参见上面的“toggleTodo()”或下面的例子。没有必要自己生成一个新的状态树，MST的快照功能将自动为您派生出一个。
Although mutable sounds scary to some, fear not: actions have many interesting properties. By default trees can only be modified by using an action that belongs to the same subtree. Furthermore, actions are replayable and can be used to distribute changes (example).
虽然可变听起来有些吓人，但不要害怕:动作有很多有趣的属性。默认情况下，树只能通过属于同一子树的action来修改。此外，actions是可重新播放的，可以用来分配变更(示例)。
Moreover, because changes can be detected on a fine grained level, JSON patches are supported out of the box. Simply subscribing to the patch stream of a tree is another way to sync diffs with, for example, back-end servers or other clients (example).
此外，由于可以在细粒度级别上检测到更改，因此支持JSON补丁。简单地订阅树的补丁流是另一种同步差异的方法，例如，后端服务器或其他客户端(示例)。
([example](https://github.com/mobxjs/mobx-state-tree/blob/master/packages/mst-example-boxes/src/stores/socket.js)).

![patches](https://github.com/mobxjs/mobx-state-tree/raw/master/docs/patches.png)


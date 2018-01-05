每一个Vuex应用的核心都是store(仓库)，store基本上是一个容器，它包含着你的应用中的大部分的状态state。

Vuex和单纯的全局对象有一下两点不同：
vuex的状态存储是响应式的。当vue组件从store中读取状态的时候，若store中的状态发生变化，那么相应的组件也会相应地得到高效更新。

你不能直接改变store中的状态。改变store中的状态的唯一途径就是显式提交(commit) mutation。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好了解我们的应用。

我们通过提交mutation的方式，而非直接改变store.state.count, 是因为我们想要更明确的追踪到状态的变化。这个简单的约定能够让你的意图更加明显，这样在阅读代码的时候能更容易解读应用内部的状态变化。此外，也让我们有机会去实现一些能记录每次状态改变，保存状态快照的调试工具。

由于store中的状态是响应式的，在组件中调用store中的状态简单到仅需要在计算属性中返回即可。触发变化也仅仅是在组件的methods中提交mutation。

## 核心概念：
### 1.state
#### 单一状态树

vuex使用单一状态树，用一个对象包含了全部的应用层级状态。作为一个「唯一数据源」而存在。每个应用将仅仅包含一个store实例。单一状态树让我们能够直接定位任一特定的状态片段，在调试过程中能轻易取得整个当前应用状态的快照。

由于vuex的状态存储是响应式的，从store实例中读取状态最简单的办法就是在计算属性中返回某个状态。

Vuex通过store选项，提供了一种机制将状态从根组件注入到每一个子组件中。（需调用Vue.use(Vuex)）。

```javascript
const app = new Vue({
  el: "#app",
  store,//把store对象提供给'store'选项，这可以把store的实例注入到所有的子组件中
  components: {Counter},
  template: `<div>...</div>`
});
```

通过在根实例中注册store选项，该store实例会注入到根组件下的所有子组件中，且子组件能通过this.$store访问到。如下：

```javascript
const Counter = {
  template: `<div>...</div>`,
  computed: {
    count() {
      return this.$store.state.count
    }
  }
};
```


#### mapState辅助函数：

当一个组件需要获取多个状态的时候，将这些状态都声明为计算属性会有些重复和冗余。为了解决这个问题，可以使用mapState辅助函数帮助我们生成计算属性。

````javascript
// 在单独构建的版本中辅助函数为 Vuex.mapState
import { mapState } from 'vuex'
export default {
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,
    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',
    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
````

当映射的计算属性的名称与state的子节点名称相同时，可以给mapState传一个字符串数组。

```javascript
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
]);
```

mapState函数返回的是一个对象。需要使用一个工具函数将多个对象合并为一个，使我们可以将最终对象传给computed属性。自从有了对象展开运算符，可以极大简化写法：

```javascript
computed: {
  localComputed() {/*...*/},
  //使用对象展开运算符将此对象混入到外部对象中。
  ...mapState({
    // ... 
  })
}
```
组件仍然保有局部状态

使用vuex并不意味着需要将所有的状态放入vuex。虽然将所有的状态放到vuex会使状态变化更显式和易调试，但也会使代码变得冗长和不直观。如果有些状态严格属于单个组件，最好还是作为组件的局部状态。应该根据应用开发需要进行权衡和确定。


### 2.Getters

vuex允许我们在store中定义getter(可以认为是store的计算属性)。就像计算属性一样，getter的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。
Getter接受state作为第一个参数。

```javascript
const store = new Vuex.Store({
  state: {
    todos: [
      {
        id: 1,text: '...1',done: true
      },
      {
        id: 2,text: '...2',done: false
      }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done);
    }
  }
});
```

Getter会暴露为store.getters对象

```javascript
store.getters.doneTodos // -> [{id: 1,text: '...1',done: true}]
```

mapGetters辅助函数

mapGetters辅助函数仅仅是将store中的getter映射到局部计算属性。

```javascript
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}
```

如果你想将一个getter属性另取一个名字，使用对象形式

```javascript
mapGetters({
  // 映射'this.doneCount' 为'store.getters.doneTodosCount'
  doneCount: 'doneTodosCount'
})
```


### 3.Mutation

更改Vuex的store中的状态的唯一方法是提交mutation。vuex中的mutation非常类似于事件：每个mutation都有一个字符串的事件类型（type）和一个回调函数（handler）。这个回调函数就是我们实际进行状态更改的地方，并且它会接收state作为第一个参数。

```javascript
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment(state) {
      state.count++;
    }
  }
});
```

不能直接调用一个mutation handler。这个选项更像是事件注册：'当触发一个类型为increment的mutation时，调用此函数'。要唤醒一个mutation handler，需要以相应的type调用store.commit方法：
store.commit('increment');

#### 提交载荷：

可以向store.commit传入额外的参数，即mutation的载荷（payload）

```javascript
mutations: {
  increment(state, n) {
    state.count += n;
  }
}
store.commit('increment', 10);
```

对象风格的提交方式：

提交mutation的另一种方式是直接使用包含type属性的对象：

```javascript
store.commit({
  type: 'increment',
  amount: 10
});
```

当使用对象风格的提交方式，整个对象都作为载荷传给mutation函数，因此handler保持不变：
```javascript
mutations: {
  increment(state, payload) {
    state.count += payload.amount
  }
}
```

#### Mutation需要遵守vue的响应规则：
既然vuex的store中的状态是响应式的，那么当我们变更状态时，监视状态的vue组件也会自动更新。这也意味着vuex中的mutation也需要与使用vue一样遵守一些注意事项：
* 最好提前在你的store中初始化好所有的所需属性
* 当需要在对象上添加新属性时，应该：
 > 使用Vue.set(obj, 'newProp', 123), 或者
 > 以新对象替换老对象。例如，利用state-3的对象展开运算符可以这样写：
```javascript
state.obj = {...state.obj, newProp: 123}
```


#### mutation必须是同步函数。

当mutation触发的时候，回调函数还没有调用，devtools不知道什么时候回调函数实际上被调用，实质上任何在回调函数中进行的状态的改变都是不可追踪的。

在vuex中，mutation都是同步事物：
```javascript
// 任何由'increment'导致的状态变更都应该在此刻完成
store.commit('increment');
```

为了处理异步操作，可以使用actions。

Actions:
类似于mutation, 不同在于：
* action提交的是mutation，而不是直接变更状态
* action可以包含任意异步操作。

注册一个action
```javascript
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
});
```

Action函数接受一个与store实例具有相同方法和属性的context对象，因此可以调用context.commit()提交一个mutation，或者通过context.state和context.getters来获取state和getters。

可以使用参数解构来简化代码：
```javascript
actions: {
  increment({commit}) {
    commit('increment');
  }
}
```

分发Action：

action通过store.dispatch方法触发
```javascript
store.dispatch('increment');
```

#### 在组件中分发action
你在组件中使用this.$store.dispatch('xxx') 分发action, 或者使用mapAction辅助函数将组件的methods映射为store.dispatch调用（需要先在根节点注入store）。
```javascript
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`

      // `mapActions` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```

#### dispatch和commit的区别：

异步操作和同步操作的区别。

当你的操作行为中含有一步操作，比如向后台发送请求获取数据，就需要用action的dispatch去完成，其他使用commit即可。

vuex中的模块  module

由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store对象就有可能变得相当臃肿。

vuex允许我们将store分割成模块module。每个模块拥有自己的state, mutation, action, getter, 甚至是嵌套子模块。从上至下进行同样方式的分割：
```javascript
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```

默认情况下，模块内部的action, mutation, getter是注册在全局命名空间中的，这样使得多个模块能够对同一个mutation或action作出响应。

如果希望模块具有更高的封装度和复用性，可以通过添加namespaced: true；的方式使其成为命名空间模块。当模块被注册后，它的所有getter, action, mutation 都会自动根据模块注册的路径调整命名。
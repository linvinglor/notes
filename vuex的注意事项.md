## 项目结构：

vuex并不限制代码结构，但是，规定了一些需要遵守的规则：
* 应用层级的状态应该集中到单个store对象中
* 提交mutation是更改状态的唯一办法，并且这个过程是同步的。 --提交方式commit
* 异步逻辑应该封装到action中。--提交方式 dispatch

如果store文件太大，只需要将action, mutation 和 getter分割到单独的文件。

### 严格模式：
开启严格模式，需要在创建store的时候传入 strict: true

```javascript
const store = new Vuex.Store({
  strict: true,
  state: {},
  getters: {}
});
```

在严格模式下，无论何时发生了状态变更且不是由mutation函数引起的，将会抛出错误。真能保证所有的状态变更都能被调试工具跟踪到。

### 开发环境和发布环境

不要在发布环境下启用严格模式。
严格模式会深度监测状态树来检测不合规的状态变更。
请确保在发布环境下关闭严格模式，以避免性能损失。

类似于插件，可以让构建工具来处理这种情况：
```javascript
const store = new Vuex.Store({
  strict: process.env.NODE_ENV !== 'production';
});
```
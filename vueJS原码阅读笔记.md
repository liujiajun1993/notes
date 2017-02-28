关键代码：
tips:
- vm一般代指当前vue实例，vm.$options代表实例化时的自定义选项，也就是构造函数的参数

## 2. 双向绑定
### 2.1 Observer对象
每个要被observe的对象都有一个observer对象作为属性
该对象使用object.defineProperty, 将要观察的object的所有属性转换为*getter、setter*方法，同时管理其依赖，并分发其update事件。
set方法中，使用dep.notify()方法【参考Dep类】，把value的update状态通知给所有依赖于它的Watcher对象。

### 2.2 Dep对象
相当于一个消息订阅
通过addSub添加watcher进入订阅队列
通过notify通知所有订阅的watcher更新

### 2.3 Watcher对象
src/core/observer/watch.js
由Dep.nofify()方法通知，调用自身的 update方法

而watcher通过addDep将自身添加到Dep对象的订阅队列中

*最关键的问题：如何将watch调用addDep呢？*
Watcher的构造函数中，调用了this.get方法，其中调用了this.getter.call(this.vm)，也就是说，Watcher在创建的时候，会访问它需要订阅的对象的getter方法
因此我们只需要在Observer重写的getter方法中判断，当前是否有一个Watcher正在被创建。如果有，那么就调用Watcher的addDep方法
所以，使用一个全局变量Dep.target来存储当前正在被创建的Watcher
  1. Watcher被创建时，Dep.target=当前创建Watcher对象
  2. Observer重写要观察对象的getter方法。当该对象setter方法被调用时，如果此时Dep.target不为空，说明当前是Dep.target在查看该对象（而不是我们），应该订阅该对象，于是将该Dep.target加入该对象的dep的订阅者列表
  3. Watcher创建完，Dep.target=null

## 3. virtual DOM
[原理分析](https://segmentfault.com/a/1190000008291645)
**VNode对象(虚拟节点)**，参考types/vnode.d.ts这个typescript文件
定义了vNode对象的属性信息
![VNode分类](https://segmentfault.com/img/bVITTR?w=100&h=150)

**createElement函数**
创建vNode或者component（最后还是返回vNode）
![](https://segmentfault.com/img/bVIT2U?w=609&h=705/view)

**patch函数**
src/core/vdom/patch.js
1. 当第一个参数是真实DOM节点，而第二个参数是虚拟节点时，如何第三个属性为true，则将虚拟节点插入*真实DOM*
2. 对virtual DOM进行操作，删除、创建或者更新
核心是patchVNode函数，对VDOM的更新

Vue.prototype.__patch__实际就是patch函数



## 1.主要模块
> initMixin(Vue$3);
> stateMixin(Vue$3);
> eventsMixin(Vue$3);
> lifecycleMixin(Vue$3);
> renderMixin(Vue$3);

使用mixin方式进行模块化

### 1.1 initMixin
>  initLifecycle(vm);
   initEvents(vm);
   callHook(vm, 'beforeCreate');
   initState(vm);
   callHook(vm, 'created');
   initRender(vm);

其中，initLifecycle将生命周期状态重置。
initEvents初始化on、off函数，并重置listeners。
callHook，调用vm.$options[beforeCreate]中的所有函数，也即此时生命周期走到了beforeCreate
initState执行一系列初始化操作。
注意，**beforeCreate和create的生命周期状态之间，只差一个initState函数**。
initRender判断是否存在el，如果存在调用$mount函数将需要的元素插入DOM树。 <span style="color:red">此处存疑</span>

#### 1.1 initState
beforeCreate和create的生命周期状态之间执行的操作。
该函数基本上是**处理构造函数中的各种参数**，包括props、data、computed、methods、watch等
> function initState (vm) {
  vm._watchers = [];
  initProps(vm);
  initData(vm);
  initComputed(vm);
  initMethods(vm);
  initWatch(vm);
}

其中，initProps判断是否存在**props**属性，如果存在从父组件传入的有效（非保留属性）props属性，将observerState.shouldConvert属性置为true，即需要观察父组件传入props的变化。
initData处理构造函数中的**data**属性（对象或返回对象的函数），同时调用observe(data)新建一个**Observer实例**来观察data。
initComputed处理构造函数中的**computed**属性，建立对应的get方法和set方法（set方法为空）。
initMethods将methods中的所有方法bind到当前实例
initWatch处理构造函数中的**watch**属性，通过调用createWatcher方法，将响应函数（或响应函数array）绑定到对应data属性上。原理是新建一个**Watcher实例**。

### 1.2 stateMixin
好吧我没看懂这是要干嘛。具体说来就是定义了Vue.prototype中的$set, $delete, $watch, $data等方法和属性

### 1.3 eventsMixin
定义了原型中的on/once/off/emit等方法，都是基于vm._events队列进行
**vm._events[event]**存储触发event事件时的所有响应函数

### 1.4 lifecycleMixin
定义了\_mount/\_update/\_updateFromParent/$forceUpdate/$destroy等方法
\_mount函数中，**beforeMount**状态和**mounted**状态之间，只相差vm._watcher的新建
\_update函数中，**beforeUpdate**和**updated**状态之间，只相差DOM树根据属性变化进行更新的\__patch函数，该函数从createPatchFunction中演变而来，包含createElm等函数实现DOM树操作。
\_updateFromParent
\_forceUpdate强制vm._watcher更新
\_destroy函数中，**beforeDestroy**和**destroyed**状态之间，相差从父组件中移除依赖，删除watcher，取消事件绑定，调用\__patch更新DOM树等操作


### 1.5 renderMixin
利用渲染函数将数据渲染为v-tree结构(虚拟树)
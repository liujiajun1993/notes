关键代码：
tips:
- vm一般代指当前vue实例，vm.$options代表实例化时的自定义选项，也就是构造函数的参数

## 1.主要模块
> initMixin(Vue$3);
  stateMixin(Vue$3);
  eventsMixin(Vue$3);
  lifecycleMixin(Vue$3);
  renderMixin(Vue$3);

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

## Observer
## Watcher

## Vue

#### Diff算法

每次发生更新时，都会生成的新的VDOM Tree和旧的VDOM Tree进行Diff，这个过程会进行逐层Diff

**每一层的Diff**

**有动态内容的结点（不需要对比结点顺序）进行Block Diff**

在生成VDOM Tree时，Vue会标记需要动态更新的结点，也就是有动态值的结点进行Diff，而那些静态内容的结点不会发生更新

**需要对比结点顺序**（v-for）

**无Key的Diff**

1. 取新旧节点数组较小长度为公共长度
2. 以公共长度为结尾进行循环，按序对比新旧节点
3. 循环结束，若旧节点数组仍有剩余，卸载对应节点，若新节点数组仍有剩余，挂载相应节点

![](./images/diff.png)



**有key的Diff**

1. 从头部开始Diff，判断key和type均相同的为可复用结点，进行patch
2. 从尾部开始Difff，判断key和type均相同的为可复用结点，进行patch
3. 如果旧列表所有结点Diff结束，新列表仍有剩余结点，则挂载剩余结点

​	  eg: 原始序列 **a -> b -> c** 改变后的序列 **a -> b -> c -> d -> e**

4. 如果新列表所有结点Diff结束，旧列表仍有剩余结点，则卸载剩余结点

​      eg: 原始序列 **a -> b -> c -> d -> e** 改变后的序列 **a -> b -> c**

5. 首尾公共部分Diff结束，剩余中间部分进行Diff

​      eg: 原始序列 **a -> b -> c -> d -> e** 改变后的序列 **a -> c -> b -> e -> d** 

​			剩余Diff序列 old: **b -> c -> d -> e**    new: **c -> b -> e - > d**

​      5.1 生成新列表的key和index之间的Map **{ c:  1, b: 2,  e:3, d:4 }**

​      5.2 生成新旧列表结点的顺序索引，通过key从Map查找新索引，计算其顺序索引，即**[3, 2, 5, 4]**，值越大代表结点在原序列中顺序靠前

​	  5.3 

#### 响应式原理

首先第一步需要拦截数据的getter，setter，**reactive**函数通过Proxy来进行拦截，通过**effect**传入依赖触发**getter**拦截，在**getter**拦截中需要收集依赖，触发**track**创建依赖对应的数据结构（三个Map），通过**trackEffect**收集依赖。

在数据发生更新触发**setter**拦截，在**setter**拦截中触发**trigger**函数取出对应的数据结构，**trigger**函数会触发**triggerEffect**函数执行调度，触发所有依赖的更新（**run**方法）

1. **reactive**函数内部是通过**createReactiveObject**函数来创建一个响应式对象，核心就是创建**Proxy**对原始对象进行代理

2. **effect**函数收集依赖，首次执行（没有lazy）触发**getter**收集依赖，并初始化对应的**_effect**对象（**new ReactiveEffect**，内部传入调度器，调度器为**run**方法）

3. **getter**触发track函数收集依赖

4. **track**会获取或者创建相应的数据结构，对应1个WeakMap **targetMap**，2个Map **depsMap和dep**

​         **targetMap：**key为对象，value为key对应的Map **depsMap**，选择WeakMap的原因是WeakMap是弱引用，方便GC进行内存回收，防止内存泄漏

​         **depsMap：**key为对象的key值，value为**dep**

​         **dep：**key为依赖（副作用函数的effect对象），value为_trackId，目的是防止重复收集依赖

​         创建完成触发**trackEffect**函数将依赖进行收集，即将effec设置到dep中

5. obj[key]发生改变时，触发**setter**来触发**trigger**函数
6. **trigger**函数从对应的数据结构中取出依赖，targetMap => obj => depsMap => key => dep，取出依赖后暂停调度，依次通过**triggerEffect**函数通过所有依赖进行更新
7. **triggerEffect**会将依赖的调度器（调度器为传入的_effect.run方法）加入到调度队列，然后恢复调度，执行更新

![](./images/dep数据结构.png)
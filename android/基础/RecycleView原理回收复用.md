### RecyclerView架构中核心组件
- 1.回收池：是一个View的集合，能回收任意Item控件，并返回符合类型的Item控件，比如onBinderHolder方法中的第一个参数就是从回收池中返回的
- 2.适配器：Adapter接口，辅助RecyclerView实现列表展示，适配器模式，将用户界面展示与交互分离
- 3.RecyclerView，根据用户触摸反馈，协调回收池对象与适配器对象之间的工作

### 最大优势
- 加载无限view，不会发生卡顿，为什么？
- 第一屏加载方式：recyclerView 加载第一屏，将需求交给回收池，而回收池中是空的，就会找适配器，适配器中会协调onCreateViewHolder，它会返回一个View，这样依次把第一屏加载满。
- 第一屏的第一个view划出去了：会把划出去的View放到栈中，它是多个栈，因为view对应不同的布局类型。回收的时候有个type类型。
- 第二屏加载：底部出现空缺，触发加载机制，从回收池中找到对应的view，通过Adapter中onBinderViewHolder进行数据刷新

### 多种布局类型
- 重写getItemViewType 来实现不同类型的布局


### RecyclerView的复用机制，View的回收与复用过程


### RecyclerView支持多个不同类型的布局，他们怎么缓存并查找的


### RecyclerView适配器的原理，为什么要适配器模式

### RecyclerView一屏加载个数是怎么确定的，他是怎么做到不显示item的Item缓存到内存中

### 手写RecyclerView



### 参考致谢
- https://www.bilibili.com/video/BV1Fi4y1x7p5?spm_id_from=333.337.search-card.all.click
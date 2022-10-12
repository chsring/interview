### RecyclerView架构中核心组件
- 1.回收池：是一个View的集合，能回收任意Item控件，并返回符合类型的Item控件，比如onBinderHolder方法中的第一个参数就是从回收池中返回的
- 2.适配器：Adapter接口，辅助RecyclerView实现列表展示，适配器模式，将用户界面展示与交互分离
- 3.RecyclerView，根据用户触摸反馈，协调回收池对象与适配器对象之间的工作

### RecyclerView的复用机制，View的回收与复用过程
- 加载无限view，不会发生卡顿，为什么？
- 第一屏加载方式：recyclerView 加载第一屏，将需求交给回收池，而回收池中是空的，就会找适配器，适配器中会协调onCreateViewHolder，它会返回一个View，这样依次把第一屏加载满。
- 第一屏的第一个view划出去了：会把划出去的View放到栈中，它是多个栈，因为view对应不同的布局类型。回收的时候有个type类型。
- 第二屏加载：底部出现空缺，触发加载机制，从回收池中找到对应的view，通过Adapter中onBinderViewHolder进行数据刷新

### 多种布局类型
- 重写getItemViewType ，和 getViewTypeCount 来实现不同类型的布局

### RecyclerView支持多个不同类型的布局，他们怎么缓存并查找的
- 1.在回收池中有不同类型的stack集合，回收池中通过type来区分不同类型的View
- 2.回收池是为了回收复用经常存或者取的view

### RecyclerView适配器的原理，为什么要适配器模式
- 适配器就是为了 ui和处理业务的分离

### RecyclerView中每一个View的tag是空的吗？
- 不是空的，打tag是通过键值对的方式。因为在移除view的时候是获取不到type值的，所以在填充view的时候，需要把type打进tag值，在remove的时候获取。
- 用到键值对是因为要把tag值留给用户用。

### RecyclerView一屏加载个数是怎么确定的，他是怎么做到不显示item的Item缓存到内存中
- onLyaout中确定的。在填充过程中，每填充一个item，那它的bottom都会加上它的高度，直到大于等于屏幕的高度，我们就认为填充完。

### 手写RecyclerView

### notifyDataSetChanged 刷新原理，观察者模式
- 

### notifyItemChanged 刷新原理
- Adapter.notifyItemChanged(int position, @Nullable Object payload) 方法会导致 RecyclerView 的 onMeasure() 和 onLayout() 方法调用。
- 在 onLayout() 方法中会调用 dispatchLayoutStep1()、dispatchLayoutStep2() 和 dispatchLayoutStep3() 三个方法
- 其中 dispatchLayoutStep1() 将更新信息存储到 ViewHolder 中
- dispatchLayoutStep2() 进行子 View 的布局，dispatchLayoutStep3()触发动画。在dispatchLayoutStep2()中，会通过DefaultItemAnimator的canReuseUpdatedViewHolder()方法判断position处是否复用之前的ViewHolder，如果调用notifyItemChanged()时的payload不为空，则复用；否则，不复用。在dispatchLayoutStep3()中，如果position处的ViewHolder与之前的ViewHolder相同，则执行DefaultItemAnimator的move动画；如果不同，则执行DefaultItemAnimator的change动画，旧View动画消失（alpha值从1到0），新View动画展现（alpha值从0到1），这样就出现了闪烁效果。



### AsyncListDiffer原理（性能提升）

### 参考致谢
- https://www.bilibili.com/video/BV1Fi4y1x7p5?spm_id_from=333.337.search-card.all.click
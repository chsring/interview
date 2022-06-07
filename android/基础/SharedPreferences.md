## 简单介绍
- SharedPreferences(以下统称为sp)是Android提供的数据持久化的一种手段，适合单进程、小批量的数据存储与访问
- 由于sharedPreferences是基于xml文件实现的，所有持久化数据都是一次性加载，如果数据过大是不适合采用SP存放。
- 实际上是用xml文件存放数据，文件存保存放在/data/data//shared_prefs/







### 参考致谢
 - https://juejin.cn/post/6926825172066369544
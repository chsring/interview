### Intent
- Android 中通过 Intent 对象来表示一条消息，一个 Intent 对象不仅包含有这个消息的目的地，还可以包含消息的内容，通过 Intent 可以实现各种系统组件的调用与激活

### 显示意图
- 显式意图：调用Intent.setComponent（）或Intent.setClass（）方法明确指定了组件名的Intent为显式意图，显式意图明确指定了Intent应该传递给哪个组件，它一般用在知道目标组件名称的前提下，显式Intent更多用于在应用程序内部传递消息。

### 隐式意图
- 隐式意图：没有明确指定组件名的Intent为隐式意图，通过Intent Filter来实现的，Android系统会根据隐式意图中设置的动作（action）、类别（category）、数据（Uri和数据类型）找到最合适的组件来处理这个意图，它一般用在没有明确指出目标组件名称的前提下，它更广泛地用于在不同应用程序之间传递消息。


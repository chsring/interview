### 自定义View
- 重写构造方法
- 重写onMeasure测量
- 重写onDraw

### 自定义ViewGroup
- 重写构造方法
- 重写onMeasure测量
- 重写onLayout

### View绘制和加载过程
![img.png](../resource/View与Window逻辑结构.png)
![img.png](../resource/View绘制.png)
- ViewRoot对应于ViewRootImpl类，它是连接WindowManager和DecorView的纽带，View的绘制流程开始于ViewRoot的performTraversals()方法，只有经过measure、layout、draw三个流程才能最终把View绘制出来
- 当Activity对象创建完毕后，会将DecorView添加到Window中，同时会创建ViewRootImpl对象，并将该对象和DecorView建立关联，在ViewRootImpl里面performTraversals分发。
- 这个流程在onCreate之前，所以会有一段时间的黑屏、白屏。所以在启动的时候解决黑白屏问题，我们会设置主题，因为在onCreate之前会把主题设置进来，之后才会加载xml布局中的问题。
- performTraversals()依次调用performMeasure()、performLayout()和performDraw()三个方法，分别完成顶级View的绘制。
- 其中performMeasure()会调用view的measure()，measure()中又调用view的onMeasure()，实现对其所有子元素的measure过程，这样就完成了一次measure过程；
- 接着子元素会重复父容器的measure过程，如此反复至完成整个View树的遍历
- onLayout和onDraw跟measure方法类似

#### View的measure过程
![img.png](../resource/View的Measure过程.png)
- View的测量过程首先会调用View的measure()方法，而measure()方法又会调用onMeasure()方法实现具体的测量。onMeasure()主要通过setMeasuredDimension()方法来设置View宽高的测量值
- View的实际测量宽高是通过getDefaultSize()方法来获取的（返回值实际上就是View的SpecSize），该方法的主要逻辑是：根据传进来的View的MeasureSpec，获取对应的SpecMode值和SpecSize值，并判断SpecMode三种测量模式下对应的View的SpecSize的取值。
- 在这里主要关注EXACTLY和AT_MOST两种模式，这两种模式下都是直接返回View的SpecSize值，这个SpecSize就是View的测量宽高大小。
- UNSPECIFIED测量模式的话，则会使用getSuggestedMinimumWidth()和getSuggestedMinimumHeight()提供的默认大小，那么默认大小是多少呢？通过getSuggestedMinimumWidth()方法可以看到：如果View没有设置背景，那么View的测量宽度等于XML布局文件中android:minWidth属性指定的值，如果没有指定则默认为0；如果View设置了背景，那么View的测量宽度等于android:minWidth属性指定的值与背景图Drawable的原始宽度（若无原始宽度则默认为0）两者中的最大值。

#### ViewGroup的measure过程
![img.png](../resource/ViewGroup的measure过程.png)
- ViewGroup的测量过程除了完成自身的测量之外，还会遍历去调用子View的measure()方法。
- ViewGroup是一个抽象类，没有重写View的onMeasure()方法，所以需要子类去实现onMeasure()方法规定具体的测量规则。
- ViewGroup子类复写onMeasure()方法一般有如下三个步骤：
- （1）遍历所有子View并测量其宽高，直接调用ViewGroup的measureChildren()方法；
- （2）合并计算所有子View测量的宽高，最终得到父View的实际测量宽高；
- （3）存储父View实际测量宽高值；
- ViewGroup中提供了measureChildren()方法，该方法主要遍历所有的子View并调用其measureChild()方法，measureChild()主要的逻辑是：取出子View的LayoutParams参数，结合传进来的父View的MeasureSpec参数，通过getChildMeasureSpec()来计算并创建子View的MeasureSpec
- getChildMeasureSpec()方法主要获取父View测量规格中的SpecMode值和SpecSize值，并根据三种SpecMode模式结合子View的LayoutParams参数计算出子View的SpecMode值和SpecSize值，并通过makeMeasureSpec()方法创建对应的MeasureSpec测量规格，然后再把MeasureSpec传递给子View的measure()方法进行测量
- 如此递归下去遍历所有的子View并测量子View的宽高从而得出ViewGroup的实际测量大小。

### MeasureSpec测量规格
#### MeasureSpec的定义
- MeasureSpec是一个32位的int值，前2位表示SpecMode测量模式，后30位表示SpecSize测量大小；在一个View控件的measure过程中，系统会将这个View的LayoutParams结合父容器的MeasureSpec生成一个MeasureSpec，这个MeasureSpec即规定好了如何去测量这个View的规格大小。
- 子View的MeasureSpec  =  父View的MeasureSpec  +  子View的LyaoutParams

#### SpecMode有三种测量模式
- UNSPECIFIED：不确定模式，父控件不会对子控件有任何约束；
- EXACTLY：精确模式，父容器知道View所需要的精确大小，这时候View的最终大小就是SpecSize所指定的值；对应于LyaoutParams中的match_parent或具体数值。
- AT_MOST：最多模式，父容器指定了一个可用的大小SpecSize，View的大小不能超过这个值，View的最终大小要看View的具体实现；对应于LyaoutParams中的wrap_content。

#### View的MeasureSpec创建规则
- 当子View的LyaoutParams为固定宽高时，不管父View的SpecMode是什么，子View的MeasureSpec = EXACTLY模式 + LyaoutParams的实际大小。
- 当子View的LyaoutParams为match_parent时：如果父View的SpecMode为EXACTLY精确模式时，子View的MeasureSpec = EXACTLY模式 + 父容器的剩余空间；如果父View的SpecMode为AT_MOST最大模式时，子View的MeasureSpec = AT_MOST模式 + 父容器的剩余空间（不允许超过）。
- 当子View的LyaoutParams为wrap_content时：不管父View的SpecMode是EXACTLY精确模式还是AT_MOST最大模式，子View的MeasureSpec的SpecMode总是AT_MOST模式且SpecSize不会超过父容器的剩余空间（SpecSize具体大小由子View实现）。

#### 为什么ViewGroup要设计成抽象类自己去实现onMeasure
- ViewGroup抽象类需要子类自己实现onMeasure()方法，因为不同的ViewGroup具有不同的布局特性（LinearLayout、RelativeLayout等），导致测量的细节也不一样，所以没有跟View测量过程一样做onMeasure()的统一处理。

#### 如何正确获取View的测量宽高
- onLayout()中去获取View的测量宽高和最终宽高，getMeasureWidth()和getMeasureHeight()用来获取测量宽高，getWidth()和getHeight()用来获取最终宽高。

#### 在Activity启动时，如何正确获取一个View的宽高
- 由于View的measure过程和Activity的生命周期是不同步的，所以无法保证Activity的onCreate()或者onResume()方法执行时某个View已经测量完毕，可以通过以下方法来解决：
- （1）在onWindowFocusChanged()方法中获取View的宽高，该方法可能会被频繁调用；
- （2）通过ViewTreeObserver的OnGlobalLayoutListener监听接口，当View树的状态发生改变或者View树内部的View的可见性发生改变时，就会回调onGlobalLayout()方法，在该方法中可以准确获取View的实际宽高；

#### View的Layout过程
- 首先调用View的layout()方法，在该方法中通过setFrame()方法来设定View的四个顶点的位置，即 初始化mLeft、mTop、mRight、mBottom这四个值，View的四个顶点一旦确定，那么View在父容器的位置也就确定了。
- 接着会调用onLayout()方法确定所有子View在父容器中的位置，由于是单一View的layout过程，所以 onLayout()方法为空实现，因为没有子View（如果是ViewGroup需要子类实现 onLayout()方法）。

#### ViewGroup的Layout过程
![img.png](../resource/ViewGroup的Layout过程.png)
- 首先会调用自身layout()方法，但和View的layout过程不一样的是，ViewGroup需要子类实现onLayout()方法，循环遍历所有的子View并调用其layout()方法确定子View的位置，从而最终确定ViewGroup在父容器的位置。

#### 如何实现ViewGroup的onLayout()，以LinearLayout为例
- 在onLayout()中首先判断水平或者垂直方向进入相应的处理函数
- 查看垂直处理函数layoutVertical()主要遍历所有子View并调用setChildFrame()
- 在setChildFrame()中调用子View的layout()来确定每个子View的位置，从而最终确定自身的位置。

#### View的Draw过程 和 ViewGroup的Draw过程
![img.png](../resource/View的Draw过程.png)
- drawBackground()：绘制背景；
- 保存当前的canvas层（不是必须的）；
- onDraw()： 绘制View的内容，这是一个空实现，需要子View根据要绘制的颜色、线条等样式去具体实现，所以要在子View里重写该方法；
- dispatchDraw()： 对所有子View进行绘制；单一View的dispatchDraw()方法是一个空方法，因为单一View没有子View，不需要实现dispatchDraw ()方法，而ViewGroup就不一样了，它实现了dispatchDraw()方法去遍历所有子View进行绘制；
- onDrawForeground()：绘制装饰，比如滚动条；
![img.png](../resource/ViewGroup的Draw过程.png)

### 刷新View的方法
- invalidate()： 不会经过measure和layout过程，只会调用draw过程；
- requestLayout() ：会调用measure和layout过程重新测量大小和确定位置，不会调用draw过程。

### 参考致谢
- https://www.jianshu.com/p/a5ea8174d912

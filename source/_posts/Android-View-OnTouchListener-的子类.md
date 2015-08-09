title: Android View.OnTouchListener 的子类
date: 2015-04-26 10:27:24
category: Android
tags: [View, OnTouchListener]
thumbnail: http://rocko-blog.qiniudn.com/Android%20View.OnTouchListener%20%E7%9A%84%E5%AD%90%E7%B1%BB_1.gif?imageView2/2/w/100/h/100/q/100
toc: true
---

如下是几个实现了 [OnTouchListener](http://developer.android.com/reference/android/view/View.OnTouchListener.html) 接口的子类，OnTouchListener 我们是再熟悉不过了，在 Hello World 开始就接触了，但在 Support V4 中还有它的 3 个子类我们平时可能使用的较少但就其功能而言还是对我们很有帮助的。

- ** [AutoScrollHelper](http://developer.android.com/reference/android/support/v4/widget/AutoScrollHelper.html) ** 抽象类，用于控件边缘触发自动滚动。
- ** [ListViewAutoScrollHelper](http://developer.android.com/reference/android/support/v4/widget/ListViewAutoScrollHelper.html) ** 用于 ListView，目前 SDK 里的唯一 AutoScrollHelper 实现类。
- ** [ZoomButtonsController](http://developer.android.com/reference/android/widget/ZoomButtonsController.html) ** 用于控制缩放控件。

三者的功能体现在 AutoScrollHelper 和 ZoomButtonsController，前者用于实现控件的自动滚动而后者用于对缩放控件（缩小放大按钮）的处理。

## AutoScrollHelper
为了更好阐述它的功能，我们先来看如下的 Gif 图：
![ListViewAutoScrollHelper](http://rocko-blog.qiniudn.com/Android%20View.OnTouchListener%20%E7%9A%84%E5%AD%90%E7%B1%BB_1.gif)
所以，他能完成的功能就是在 View 的边缘长按时能自动地滚动视图。下面是它的主要方法说明：

<!--more-->

``` Java
// 构造方法，不用说了
AutoScrollHelper(View target)

/* 3个子类必须实现的抽象方法 */

// 判断 View 能否在水平方向上滚动
public abstract boolean canTargetScrollHorizontally (int direction)

// 判断 View 能否在垂直方向上滚动
public abstract boolean canTargetScrollVertically (int direction)

// 最重要的方法，控制 View 的滚动实现，参数分别表示在水平和垂直方向上滚动的像素值
public abstract void scrollTargetBy (int deltaX, int deltaY)


/* 如下是一些基本的属性配置方法 */

// 长按边缘后开始滚动的的延迟时间
public AutoScrollHelper setActivationDelay (int delayMillis)

// 边缘触发类型，有3种：EDGE_TYPE_INSIDE: 在自身 View 内边缘区域才会触发滚动，（手指）移动到 View 外的区域时即停止滚动；
// EDGE_TYPE_INSIDE_EXTEND：在自身 View 内边缘区域才会触发滚动，但手指移动到 View 外时仍会滚动；
// EDGE_TYPE_OUTSIDE：在自身 View 外边缘处才有触发滚动，手指向内移动到 View 内则会停止。
public AutoScrollHelper setEdgeType (int type)

// 是否消耗掉触摸事件
public AutoScrollHelper setExclusive (boolean exclusive)

// 开始滚动后到达预定速度的时间
public AutoScrollHelper setRampUpDuration (int durationMillis)

// 开始停止滚动时，速度减为 0 的时间
public AutoScrollHelper setRampDownDuration (int durationMillis)

// 还有诸如：触摸边缘的距离范围、滚动速度等方法，比较好理解，不一一列举了。
```


### ListViewAutoScrollHelper
使用：
``` Java
AutoScrollHelper autoScrollHelper = new ListViewAutoScrollHelper(listView);
listView.setOnTouchListener(autoScrollHelper); 
autoScrollHelper.setEnabled(true); // 这个不要忘了
```

ListViewAutoScrollHelper 的效果图如上，Google 帮我们实现了在 ListView 上的实现，ListViewAutoScrollHelper 也只能用于 ListView，在其它可滚动视图上又怎么办呢？很明显，继承实现 AutoScrollHelper，下面就来在 RecyclerView 和 ScrollView 上实现 RecyclerViewAutoScrollHelper 和 ScrollViewAutoScrollHelper，得益于 RecyclerView 能干很多事，这也就基本涵盖了滚动视图了。

### RecyclerViewAutoScrollHelper
RecyclerView 要实现 AutoScrollHelper，只需要写 3 行代码就够了，支持水平和垂直的方向上的操作，相比 ListViewAutoScrollHelper 的实现简单许多。
``` Java
public class RecyclerViewAutoScrollHelper extends AutoScrollHelper {
    protected RecyclerView mTarget;

    public RecyclerViewAutoScrollHelper(RecyclerView target) {
        super(target);
        this.mTarget = target;
    }

    @Override
    public void scrollTargetBy(int deltaX, int deltaY) {
        mTarget.scrollBy(deltaX, deltaY); // 1 行
    }

    @Override
    public boolean canTargetScrollHorizontally(int direction) {
        return mTarget.getLayoutManager().canScrollHorizontally(); // 2 行
    }

    @Override
    public boolean canTargetScrollVertically(int direction) {
        return mTarget.getLayoutManager().canScrollVertically(); // 3 行
    }
}
```
效果如下：
![RecyclerViewAutoScrollHelper](http://rocko-blog.qiniudn.com/Android View.OnTouchListener 的子类_2.gif)

### ScrollViewAutoScrollHelper
ScrollView 的也很简单，如下：
``` Java
public class ScrollViewAutoScrollHelper extends AutoScrollHelper {
    protected ScrollView mTarger;

    public ScrollViewAutoScrollHelper(ScrollView target) {
        super(target);
        this.mTarger = target;
    }

    @Override
    public void scrollTargetBy(int deltaX, int deltaY) {
        mTarger.smoothScrollBy(deltaX, deltaY);
    }

    @Override
    public boolean canTargetScrollHorizontally(int direction) {
        return mTarger.canScrollHorizontally(direction);
    }

    @Override
    public boolean canTargetScrollVertically(int direction) {
        return mTarger.canScrollVertically(direction);
    }
}
```
此外，` HorizontalScrollView ` 的实现也是类似就不贴了。

## ZoomButtonsController
使用方式也很简单，其相关 API 可以 [戳这里](http://www.cnblogs.com/over140/archive/2010/12/02/1894065.html)。在构造方法中传进一个 View，然后缩放控件就依附绑定在此 View 当中，然后在 OnZoomListener 回调函数中处理放大和缩小事件。需要注意的是在生命周期结束时需要把它注销掉 ` zoomButtonsController.setVisible(false) `，否则会发生 ANR、内存泄露。效果如下：
![ZoomButtonsController](http://rocko-blog.qiniudn.com/Android View.OnTouchListener 的子类_3.gif)

## End
源码传送门：[touchlistener-subclasses](https://github.com/zhengxiaopeng/Rocko-Android-Demos/tree/master/touchlistener-subclasses)
title: 【译】解决共享元素在系统UI上绘制的问题
date: 2015-06-19 12:49:07
category: Android
tag: [动画, 译文]
thumbnail: http://rocko-blog.qiniudn.com/【译】解决共享元素在系统UI上绘制的问题-1.jpg?imageView2/2/w/400/h/280/q/100
toc: true
---



- 原文链接 ：[Dealing with shared elements that draw on top of the system UI](https://plus.google.com/+AlexLockwood/posts/RPtwZ5nNebb)

目前为止，我在使用 Activity transitions 中遇到的一个比较烦人的小问题：共享元素会部分地覆盖掉 Status/Navigation/Action Bar，一旦开始过渡动画，共享元素就会很唐突地从系统 UI 下 `弹出`。这一不和谐的表现可以看看下面这个视频：

<!--more-->

{% youtube  yAbDPjhftlQ %}


更多关于 Lollipop 中共享元素过渡动画的内容，可查看我的系列博客：[http://www.androiddesignpatterns.com/2014/12/activity-fragment-transitions-in-android-lollipop-part1.html](http://www.androiddesignpatterns.com/2014/12/activity-fragment-transitions-in-android-lollipop-part1.html)。PS.这一系列文章在 [开发技术前线](http://www.devtf.cn/) 已有译文，见附录～

## 问题

出现这一问题的原因是因为共享元素默认是在整个窗口视图层的顶层 ViewOverlay 上绘制的。这一默认行为确保了过渡元素总是 Activity 过渡动画的中心焦点，同时这也会使得共享元素会意外地被绘制在正在调用和被调用 Activity 的视图层级顶部。所以不幸的，不小心的话就会造成共享元素可能被绘制在 `应用的 Action Bar、状态栏背景、导航栏背景 之上`，然而这种情况我们绝对是需要避免的。

## 解决方案

好在我发现有两种方法可以避免出现这种情况：

**（1）** 在你得 XML 中设置 `android:windowSharedElementsUseOverlay="false"` 来关闭 overlay。关掉后共享元素就会被作为被调用 Activity 的视图层级的一部分来绘制，使得共享元素不会在系统 UI bars 上发生意外重叠的情况。然而，关掉这一功能后新的问题可能也会随之而来。。。举个例子，你可能会发现在过渡动画的过程中共享元素的动画绘制过程并没有出现，大多数情况下你可以在共享元素所属的 View 中设置如下属性来避免：`android:cliptoChildren="false" 和 "android:clipToPadding="false"` （译者注：关于 clip 可参见：[android:clipToPadding和android:clipChildren](http://www.alloyteam.com/2014/10/androidcliptopadding-he-androidclipchildren/)）。

**（2）** 另一个解决这一问题的办法就是把 **Action Bar、 Status Bar background、Navigation Bar background 作为 Activity 过渡动画的额外共享元素** 。把系统 bars 作为共享元素可以确保原来的共享元素和这些系统 UI 可以被绘制在窗口视图层级的同一层级上。这一解决办法的参考代码如下：

``` Java
View decor = getWindow().getDecorView();
View statusBar = decor.findViewById(android.R.id.statusBarBackground);
View navBar = decor.findViewById(android.R.id.navigationBarBackground);
View actionBar = decor.findViewById(getResources().getIdentifier(
        "action_bar_container", "id", "android"));
```

（注意：如果你使用 appcompat 支持库，你应该直接使用 `R.id.action_bar_container` 来替代上面代码中使用程序的资源来提取 ID 的办法）
（译者注：第二种方法在 googlesamples 的 android-topeka 中被使用到，[CategorySelectionFragment.java](https://github.com/googlesamples/android-topeka/blob/master/app/src/main/java/com/google/samples/apps/topeka/fragment/CategorySelectionFragment.java)）

上述的两种办法中，我个人发现第二种实现起来比起第一种更加容易简单（在我遇到的情况中，禁用掉共享元素 view overlay 会造成很多不良副作用）。

这个话题在我之后的博文中有更加详细的描述～ [Shared Element Transitions In-Depth (part 3a)](http://www.androiddesignpatterns.com/2015/01/activity-fragment-shared-element-transitions-in-depth-part3a.html) 希望能帮到你。:)

## 附录

开发技术前线 Transitions 系列译文：
[开始使用 Transitions（过渡动画） (part 1)](https://github.com/bboyfeiyu/android-tech-frontier/tree/master/others/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAAndroid%20%E6%96%B0%E7%89%B9%E6%80%A7-Transition-Part-1)
[深入理解Content Transition (part 2)](https://github.com/bboyfeiyu/android-tech-frontier/blob/master/others/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAAndroid%20%E6%96%B0%E7%89%B9%E6%80%A7-Transition-Part-2)
[深入理解 Shared Element Transition (part 3a)](https://github.com/bboyfeiyu/android-tech-frontier/tree/master/issue-7/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAAndroid%E6%96%B0%E7%89%B9%E6%80%A7-Transition-Part-3a)
[延迟共享元素的过渡动画 (part 3b)](https://github.com/bboyfeiyu/android-tech-frontier/tree/master/issue-7/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAAndroid%E6%96%B0%E7%89%B9%E6%80%A7-Transition-Part-3b)
……
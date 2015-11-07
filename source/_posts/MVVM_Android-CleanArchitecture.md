title: MVVM_Android-CleanArchitecture
date: 2015-11-7 12:49:07
category: Android
tag: [MVVM, Data Binding, 架构]
thumbnail: http://rocko-blog.qiniudn.com/MVVM_Android-CleanArchitecture-0.png?imageView2/2/w/400/h/300/q/100
banner: http://rocko-blog.qiniudn.com/MVVM_Android-CleanArchitecture-0.png
toc: true
---

## 前言

"Architecture is About Intent, not Frameworks"    - [Robert C. Martin (Uncle Bob)](http://cleancoder.com/)

Uncle Bob 的这句话套在 MVVM 上也是适用的, MVVM 也仅仅是架构`模式`（Architectural pattern），其有一套自己的理论概念（pattern）而不是规定的具体实现（或 Frameworks）。早之前在知乎上相关问题的回答（[android UI设计MVVM设计模式讨论？](http://www.zhihu.com/question/30976423/answer/50181505)）中也简单提到过 MVVM 了，M-V-X 的关系如上图，那么这一次博主把 [Fernando Cejas(android10)](https://github.com/android10) 的 [Android-CleanArchitecture](https://github.com/android10/Android-CleanArchitecture) 项目中的 MVP 实现重构成了用 MVVM 来实现。整个历程也算比较愉快，没什么不良反应，这篇文章理所当然会重点说说 MVVM 的实现、 Data Binding 等相关的东西。那为什么拥抱 MVVM 呢。当然是 Google 推出官方的 [data binding](https://developer.android.com/tools/data-binding/guide.html) 啦，下一次的 Android MVVM 热潮应该就是 data binding 放出正式版了。

## 分层架构与 M-V-X

首先还是先来说说 `分层架构` 与 `MVC` or `M-V-X` 之间的关系。分层架构是一种常见的软件应用架构，在 Java 程序中可以算是一种应用标准了，通常又叫 N 层架构，而最常见的是 3 层架构，它包含如下 3 层：

- 展示层（Presentation tier），也称为 UI 层，也就是程序的界面部分。

- 业务层（business logic(domain) tier）， 业务层，是最为核心的一层。

- 持久层（Data tier），数据持久层。

3 层架构是存在物理上分层概念的，从上往下即展示层、业务层、持久层，也从上往下由上一层依赖下一层。不同层之间也是 `高内聚低耦合` 的体现，层内高内聚，层间低耦合，`层` 是层内具体工作的高度抽象。低耦合则是依赖倒转原则体现出来，高层依赖于下层的抽象而不是具体。

接下来先说 M-V-X 的鼻祖 MVC，*Model–View–Controller (MVC) is a software architectural pattern for implementing user interfaces.*，所以 MVC 模式是为用户界面设计的，在 3 层架构中，MVC 是属于展现层的部分，所以 MVVM 作为 MVC 的演进在与分层架构的关系上也是一样。经常会看到 3 层架构与 M-V-X 混为一谈的内容，这是不正确的，虽然都是 3 部分的内容但是不能 **简单地** 把两者的每一部分对应起来，我们应该理解为，在分层架构中 M-V-X 是在展示层（Presentation tier）的应用。这些软件工程理论的东西就这样了，都是大师们留下来的东西，不要随便套上自己的概念。

## Android-CleanArchitecture

[Fernando Cejas(android10)](https://github.com/android10) 的 [Android-CleanArchitecture](https://github.com/android10/Android-CleanArchitecture) 项目中也是采用典型的 3 层架构，其中 Presentation 展现层采用了 MVP 模式，如果还未了解过 MVP，可以看看我之前写的文章：[Android中的MVP](http://rocko.xyz/2015/02/06/Android%E4%B8%AD%E7%9A%84MVP/)。不过 Google 推出官方的 `data binding` 之后我觉得基本可以不用采用 MVP 了，在 M-V-X 中， MVP 与 MVVM 算是比较接近的了但 MVP 中的一堆 View 接口也是让人头疼的，而拥有 data binding 的 MVVM 则解决了这个问题，所以请大胆拥抱 MVVM。So，下面几点是当中除 MVVM 外涉及到的东西，MVVM 放到下一节再讲。

### Dagger

<i class="fa fa-eye-slash"></i> 自带 `Dagger` 信仰光环者障眼之术开启<i class="fa fa-eye-slash"></i>， 哈哈，你们看不到接下来的这句话了。。我现在也是持不赞成 di 的观点的人(在 Android 中、、、)，可以看看这场撕逼：[依赖注入是否值得？](http://www.infoq.com/cn/news/2007/12/does-di-pay-off)。结果就是我把原项目的依赖注入模块去掉了，对于测试中的类中的成员变量来说本来就是 Mock 抽象接口，那就直接对接口或抽象类直接 Mock 操作就可，毕竟依赖注入的解耦依然是取决于需要注入的对象的抽象，维护依赖注入模块（Module）也是负担，测试代码中又要多写一套注入控制的 Dagger Module 代码。。

### RxJava、RxAndroid

先说 AsyncTask，对其已经不再想吐槽，这么重要的异步实现，版本间（Android Api）代码改来改去，又顺序又无序、又单线程执行又并发执行、内存泄露、、、。所以对于采用 RxJava 即使不采用函数响应式编程的大概念，用它来替换 AsyncTask 和 Thread + Handler 也是推荐的。此外使用 Rx 后也不需要事件总线的框架了，对于回调监听直接在数据操作的 Observable 上注册观察者即可，相对于事件总线来说是更精准的（单线）的监听。而事件总线的话则是更加松耦合的，出错的话会更加难排查，这里就不再展开了。


### Lambda

个人、团队喜好，代码简洁了很多但是代码的逻辑比较不好直观理解了，原项目也只有 3 处代码用到，故而去掉了。


此外，对于领域驱动设计（DDD）中 Repository，原项目中把其实现放到了 data 层去实现，造成 data 层会依赖其上一层（domain 业务层），作为分层架构个人认为不合适所以重新把它调整了，这其中应该是使用了依赖注入造成的。根据 DDD，个人认为 Repository 它的存在让领域层（domain 业务）感觉不到数据访问层的存在，它提供一个类似集合的接口提供给领域层进行领域对象的访问。Repository 是仓库管理员，领域层需要什么东西只需告诉仓库管理员，由仓库管理员把东西拿给它，并不需要知道东西实际放在哪。

## MVVM


### 理论简述

按照常理，先来说基本概念：

- Model，domain model（领域模型）或是数据层代表的数据模型，也可以理解为用户界面需要显示数据的抽象（数据）

- View， 应用的界面

- ViewModel，binder 所在之处，是 View 的抽象，对外暴露出公共属性和命令，是 View 与 Model 的（绑定）连接器

此外还有必不可少的一部分：Binder，Android 中也就是 Data binding 了，提供 View 与 Model 的绑定功能。下面是结构图：

![MVVM结构图](http://rocko-blog.qiniudn.com/MVVM_Android-CleanArchitecture-1.png)


### Android 中实现

目前 Android 的 data binding 还是 beta，还只是 `one-way` 单向绑定，功能上还有所欠缺、控制性也还不强，但是把它写出来还是没问题的。对于 Activity、Fragment 而言仅仅是作为 Java View 看待，与 XML 对应，所以里面只有 View 的展现逻辑，此外没有其它代码。一个 Activity 或 Fragment（一般都 with XML） 对应一个 ViewModel，对于一个基础 View（XML）可以通过继承对应的 ViewModel 实现重用，本文的代码也有体现。对于 Activity 和 Fragment 的View 状态保存恢复也通过 ViewModel 处理。因为 binding 的入口在 Activity 或 Fragment 中，所以为了方便写个基类处理 ViewModel 和 Binding 的初始化，然后在 对应的 XML 里加上 ViewModel 的 variable， XML 里不再有其它数据对象的 variable。

``` Java
public abstract class BaseActivity<VM extends ViewModel, B extends ViewDataBinding> extends Activity {

  private VM viewModel;
  private B binding;

  public void setViewModel(@NonNull VM viewModel) {
    this.viewModel = viewModel;
  }

  public VM getViewModel() {
    if (viewModel == null) {
      throw new NullPointerException("You should setViewModel first!");
    }
    return viewModel;
  }

  public void setBinding(@NonNull B binding) {
    this.binding = binding;
  }

  public B getBinding() {
    if (binding == null) {
      throw new NullPointerException("You should setBinding first!");
    }
    return binding;
  }

}
```


ViewModel 中通过 ObservableField 来达到细粒度的控制，绑定操作都放在 ViewModel 里，然后 ViewModel 里可以有多个 domain 中的 Interator(UseCase) 来得到 View 需要渲染的数据 Model。对于 ObservableField 的绑定操作和命令操作（Command）都是暴露的，也易于测试。binding 现在缺少手动在 Java 代码中注册通知事件的功能，比如有些 model 的渲染必须通过 Java 代码来操作的话就需要了，在 Activity（Java View） 中通过向 Binding 注册通知回调，而目前只能在 XML 中知道，当然目前也可以自己实现，方法也有多种：接口回调、EventBus、RxBus。。


### 架构图

除了 Persenter 改成 ViewModel 的逻辑、Repository 的抽象和具体都在 domain 外，其他部分基本一致，采用的测试也一致。

- **Clean Architecture：**

![Clean Architecture](http://rocko-blog.qiniudn.com/MVVM_Android-CleanArchitecture-2.png)


- ****MVVM_Clean-Architecture tier：****
![MVVM_Clean-Architecture 分层结构](http://rocko-blog.qiniudn.com/MVVM_Android-CleanArchitecture-3.png)

- **MVVM_Clean-Architecture put all：**
![MVVM_Clean-Architecture put all 应用在一起](http://rocko-blog.qiniudn.com/MVVM_Android-CleanArchitecture-4.png)

## Refactor

Talk is cheap. Show you the code. **↓↓↓**
[MVVM_Android-CleanArchitecture](https://github.com/zhengxiaopeng/MVVM_Android-CleanArchitecture)

需要注意的是 include 标签的 XML 节点中要使用到根节点中 data 标签里设置的 viewModel variable 的话需要这样设置；

``` XML
  <include
      layout="@layout/view_retry"
      bind:viewModel="@{viewModel}"/>
```

抽象类 ViewModel 中设置了 @Command 和 @BindView 注解，只起到清晰提醒作用。

具体重构更改可以查看 commit 记录：[MVVM_Android-CleanArchitecture commits](https://github.com/zhengxiaopeng/MVVM_Android-CleanArchitecture/commits/master)。可以看到 Activity 和 Fragment 的代码是很清爽的，比 MVP 更清爽，因为 View 的数据渲染操作交给 binder 去处理了。对 Activity 或其对应界面进行 UI 测试的话，Mock 出 model 代表的数据然后传递给 ViewModel 中 @BindView 暴露出的方法，然后检验视图对数据的正确显示就行了，也就是 View 对 Model 做了正确的渲染。



## 参考
[企业应用架构模式](http://book.douban.com/subject/4826290/)
[Architecting Android…The clean way?](http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/)
[Architecting Android…The evolution](http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/)
[Approaching Android with MVVM](https://getpocket.com/a/read/1048441260)
[ANDROID DATABINDING: GOODBYE PRESENTER, HELLO VIEWMODEL!](http://tech.vg.no/2015/07/17/android-databinding-goodbye-presenter-hello-viewmodel/)

## END

本文源码：[MVVM_Android-CleanArchitecture](https://github.com/zhengxiaopeng/MVVM_Android-CleanArchitecture) or [Rocko-Android-Demo(-MVVM_Android-CleanArchitecture)](https://github.com/zhengxiaopeng/Rocko-Android-Demos/tree/master/architecture/MVVM_Android-CleanArchitecture)

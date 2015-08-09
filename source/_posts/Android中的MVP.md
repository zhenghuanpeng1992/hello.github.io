title: Android中的MVP
date: 2015-02-06 12:28:48
category: Android
tags: [MVP]
toc: true
---

## 前言
MVP作为一种MVC的演化版本在Android开发中受到了越来越多的关注，但在项目开发中选择一种这样的软件设计模式需保持慎重心态，一旦确定使用MVP作为你App的开发模式那么你就最好坚持做下去，如果在使用MVP模式开发过程中发现问题而且坑越来越大，这时你想用MVC等来重新设计的话基本上就等于推倒重来了。要知道在Android上MVP在现在为止并没有统一的标准或者框架，不像SSH这三个成熟稳重强而有力的三剑客支持推动着Java EE的开发，所以在运用MVP时一定要做好自己的理解，并且尽量预知自己App各模块的需求（客户说改改改，我们就改改改 :-( ）以便提前做好充分的设计工作。当然MVP既然能出现那么必然有它的优点的，不然谁会理会这个冒出来的东西，下面就对Android中MVP做一些阐述。

## MVP简介
相信大家对MVC都是比较熟悉了：`M-Model-模型`、`V-View-视图`、`C-Controller-控制器`，MVP作为MVC的演化版本，那么类似的MVP所对应的意义：`M-Model-模型`、`V-View-视图`、`P-Presenter-表示器`。从MVC和MVP两者结合来看，Controlller/Presenter在MVC/MVP中都起着逻辑控制处理的角色，起着控制各业务流程的作用。而MVP与MVC最不同的一点是M与V是不直接关联的也是就Model与View不存在直接关系，这两者之间间隔着的是Presenter层，其负责调控View与Model之间的间接交互，MVP的结构图如下所示，对于这个图理解即可而不必限于其中的条条框框，毕竟在不同的场景下多少会有些出入的。在Android中很重要的一点就是对UI的操作基本上需要异步进行也就是在MainThread中才能操作UI，所以对View与Model的切断分离是合理的。此外Presenter与View、Model的交互使用接口定义交互操作可以进一步达到松耦合也可以通过接口更加方便地进行单元测试。
![MVP结构图](http://rocko-blog.qiniudn.com/Android中的MVP_1.png)

<!--more-->

## MVP之Model
模型这一层之中做的工作是具体业务逻辑处理的实现，都伴随着程序中各种数据的处理，复杂一些的就明显需要实现一个Interface来松耦合了。

## MVP之View
视图这一层体现的很轻薄，负责显示数据、提供友好界面跟用户交互就行。MVP下Activity和Fragment体现在了这一层，Activity一般也就做加载UI视图、设置监听再交由Presenter处理的一些工作，所以也就需要持有相应Presenter的引用。例如，Activity上滚动列表时隐藏或者显示Acionbar（Toolbar），这样的UI逻辑时也应该在这一层。另外在View上输入的数据做一些判断时，例如，EditText的输入数据，假如是简单的非空判断则可以作为View层的逻辑，而当需要对EditText的数据进行更复杂的比较时，如从数据库获取本地数据进行判断时明显需要经过Model层才能返回了，所以这些细节需要自己掂量。

## MVP之Presenter
Presenter这一层处理着程序各种逻辑的分发，收到View层UI上的反馈命令、定时命令、系统命令等指令后分发处理逻辑交由Model层做具体的业务操作。

## 演示demo
动手写起代码来才有更好的感觉。demo很简单,还是上个图更直观，输入城市的代号，点击按钮获取城市的天气信息然后显示出来，网络操作使用Volley框架，解析用Gson，其它的就手写了。整个项目的包设计如下：
![包结构](http://rocko-blog.qiniudn.com/Android中的MVP_2.png?imageView2/2/w/450/h/450/q/100)
![项目效果预览](http://rocko-blog.qiniudn.com/Android中的MVP_3.png?imageView2/2/w/450/h/450/q/100)
包图中明显的三层：Model包、Presenter包、UI包，其中，三者都实现各自的结构，Model为WeatherModel、Presenter为WeatherPresenter、View为Weather，那么具体实现类就是impl包里的了，View层的即为Activity。此外的app和util包无关紧要可以不看。可以看到采用MVP设计后项目明显多了很多东西，这也是不可避免的，使用原始方法可以使项目开起来简单些但是以后还有维护呢、测试呢、加功能呢、。。。
entity里的实体属性基本上对应[json里的这些属性了](http://www.weather.com.cn/data/sk/101010100.html)，代码不贴了，View里面的接口：
``` java
public interface WeatherView {
    void showLoading();
    void hideLoading();
    void showError();
    void setWeatherInfo(Weather weather);
}
```
WeatherPresenter的接口：
``` java
public interface WeatherPresenter {
    /**
     * 获取天气的逻辑
     */
    void getWeather(String cityNO);
}
```
WeatherModel接口：
``` java
public interface WeatherModel {
    void loadWeather(String cityNO, OnWeatherListener listener);
}
```
prestener里面还有个OnWeatherListener，其在Presenter层实现，给Model层回调，更改View层的状态，确保Model层不直接操作View层。如果没有这一接口在WeatherPresenterImpl实现的话，WeatherPresenterImpl只有View和Model的引用那么Model怎么把结果告诉View呢？当然这只是一种解决方案，在实际项目中可以使用Dagger、EventBus、Otto等第三方框架结合进来达到更加松耦合的设计。
``` java
public interface OnWeatherListener {
    /**
     * 成功时回调
     *
     * @param weather
     */
    void onSuccess(Weather weather);
    /**
     * 失败时回调，简单处理，没做什么
     */
    void onError();
}
```
所以demo的代码流程：Activity做了一些UI初始化的东西并需要实例化对应WeatherPresenter的引用和实现WeatherView的接口，监听界面动作，Go按钮按下后即接收到查询天气的事件，在onClick里接收到即通过WeatherPresenter的引用把它交给WeatherPresenter处理。WeatherPresenter接收到了查询天气的逻辑就知道要查询天气了，然后把查询天气的具体业务实现交给WeatherModel去实现同时把WeatherListener即WeatherPresenter自己传给WeatherModel。WeatherModel进行查询天气业务后即把结果通过WeatherListener回调通知WeatherPresenter，WeatherPresenter再把结果返回给View层的Activity，最后Activity显示结果。就这样，拍砖之处请拍。
## End
采用哪种软件设计模式都是为了达到如下目的，找到合适的加以运用就是最好的：
- 易于维护
- 易于测试
- 松耦合度
- 复用性高
- 健壮稳定

本文demo
[Rocko's MVP demo](https://github.com/zhengxiaopeng/Rocko-Android-Demos/tree/master/android-mvp)
MVP相关demo
[androidmvp](https://github.com/antoniolg/androidmvp)
[ActivityFragmentMVP](https://github.com/spengilley/ActivityFragmentMVP)
[EffectiveAndroidUI](https://github.com/pedrovgs/EffectiveAndroidUI)
[MvpCleanArchitecture](https://github.com/glomadrian/MvpCleanArchitecture)
[Material-Movies](https://github.com/saulmm/Material-Movies)

相关参考文章
[50个Android开发技巧(20 使用MVP模式)](http://blog.csdn.net/vector_yi/article/details/24719873)
[MVP for Android: how to organize the presentation layer(英文原版)](http://antonioleiva.com/mvp-android/)
[MVP for Android: how to organize the presentation layer(中文译文)](http://blog.jobbole.com/71209/)
[从三层架构到MVC,MVP](http://www.cnblogs.com/daizhj/archive/2009/04/30/1447035.HTML)
[Architecting Android…The clean way?(中文译文)](https://github.com/AWCNTT/ArticleTranslateProject/blob/master/translated/Issue%23118/2014-09-11-Architecting%20Android%E2%80%A6The%20clean%20way.md)
[TED MOSBY - SOFTWARE ARCHITECT](http://hannesdorfmann.com/android/mosby/)

title: Android 错误集锦ing...
date: 2015-02-05 13:44:42
category: Android
tags: [错误]

---
Fork from my csdn blog: [Android 错误集锦(ing...)](http://blog.csdn.net/bbld_/article/details/39520249).
Last update time: *2015-8-22 14:47:39*.   
   

温馨提示：`Ctrl+F查找`


---

**系统环境：**
Windows7 64位
**问题描述：**
Eclipse真机无法打印log信息
**错误提示：**
...
**解决方案：**
window-->show view-->android->devices，打开devices，点击右边的截屏图片的按钮。等到出现截图的时候，logcat就出来信息了（不保证每次都有用）

---
**系统环境：**
Windows7 64位
**问题描述：**
xml（资源）文件里面的错误
**错误提示：**
``` bash
android: invalid start tag xxxxx 错误原因
```
**解决方案：**
今天在学shape这个属性,结果创建的xml总是提示这个错误百思不得其解,后来找到原因了我把这个xml文件放错了位置,放到了res/layout路径下应该放在drawable的路径下才对

---
**系统环境：**
Windows7 64位
**问题描述：**
无法run(运行)工程
**错误提示：**
``` bash
Conversion to Dalvik format failed with error 1
```
**解决方案：**
第一种情况包导入错误.点击工程-->build path-->libraries-->选中android1.x 或者android2.x ，点击remove。然后再点击add library-->User Library -->next-->User Libraries-->new 你取一个名字 比如android2.1 点击OK，选中android2.1-->add jars-->\android-sdk-windows\platforms\android-7\android.jar 点击打开，点击ok-->finish.   

第二种情况签名时没有成功。签名：java -jar signapk.jar platform.x509.pem platform.pk8 e:huaworkspace\hua\bin\hua.apk e:huaworkspace\hua\bin\hua_signaed.apk，如果hua_signaed.apk签名失败，那么请到你的工作目录中将hua_signaed.apk delete掉。   

第三种情况包冲突，请到工程目录下将相同的包删除，重新导入一个，这一点和第一种情况类似，不过这是针对其他包，不是android包 

---
**系统环境：**
Windows7 64位
**问题描述：**
导入SlidingMenu和SlidingMenu所依赖的actionbarsherlock包后再导入supportv7（用来支持ActionBar），工程一直报错、无法生成R文件。
**错误提示：**
...
**解决方案：**
不用导入v7包了，因为actionbarsherlock已经支持ActionBar，再导入v7会有冲突。

---
**系统环境：**
Windows7 64位
**问题描述：**
FragmentTransaction使用问题。
**错误提示：**
``` bash
java.lang.IllegalStateException: commit already called.
```
**解决方案：**
是因为你的ft事务是全局的变量，只能commit一次。所以用两个局部ft事务去做commit即可。 原文地址：http://blog.csdn.net/knxw0001/article/details/9363411    
补充： 
``` Java
FragmentManager fragmentManager = getSupportFragmentManager(); 
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction(); 
detailFragment = new ProductDetailFragment(productId); 
commentFragment = new ProductCommentFragment(productId); 
fragmentTransaction.add(R.id.viewgroup, detailFragment); 
fragmentTransaction.add(R.id.viewgroup, commentFragment); 
fragmentTransaction.commit(); 
//下面这个是调用的时候需要用新的局部变量  
getSupportFragmentManager().beginTransaction().hide(commentFragment).show(detailFragment).commit();
```
---
**系统环境：**
Windows7 64位
**问题描述：**
使用Genymotion调试出现错误INSTALL_FAILED_CPU_ABI_INCOMPATIBLE
**错误提示：**
``` bash
Installation error: INSTALL_FAILED_CPU_ABI_INCOMPATIBLE
Please check logcat output for more details.
Launch canceled!
```
**解决方案：**
点击下载[Genymotion-ARM-Translation.zip](http://pan.baidu.com/s/1c0ELjzi#dir/path=%2FJustDoIt%2F%E4%B8%80%E4%BA%9B%E4%B8%8B%E8%BD%BD%E9%93%BE%E6%8E%A5%2F%E6%9C%89%E9%81%93)
将你的虚拟器运行起来，将下载好的zip包用鼠标拖到虚拟机窗口中，出现确认对跨框点OK就行。然后重启你的虚拟机。


<!--more-->


---
**系统环境：**
Windows7 64位
**问题描述：**
自定义View（RemoteViews）无法发出通知，程序报错
**错误提示：**
``` bash
android.app.RemoteServiceException: Bad notification posted from package com.gdut.repairsystem:Couldn't expand RemoteViews for: StatusBarNotification(package=com.gdut.repairsystem id=0 tag=null notification=Notification(vibrate=null,sound=null,defaults=0x0,flags=0x10))
```
**解决方案：**
在自定义布局中使用了不自持的组件（这里居然是使用了自定义style的原因！！！(最外层的layout不能，里面的可以)）。

---
**系统环境：**
Windows7 64位
**问题描述：**
jni代码里：Type Method 'NewStringUTF' could not be resolved  
**错误提示：**
``` bash
Type Method 'NewStringUTF' could not be resolved
```
**解决方案：**
点开problems窗口把这条错误删除，ok！

---
**系统环境：**
Windows7 64位
**问题描述：**
Intent使用serializable传递复杂数据时报错
**错误提示：**
``` bash
Parcelable encountered IOException writing serializable object
```
**解决方案：**
在Activity之间传递数据必须所有的内容都实现serializable接口才行。

---
**系统环境：**
Windows7 64位
**问题描述：**
Intent使用Parcelable传递复杂数据时报错
**错误提示：**
``` bash
Unmarshalling unknown type code 7471205 at offset 232
```
**解决方案：**
在两个activitiy之间，传递一个实现了Parcelable的ArrayList，就出现了这个错误，但是当我传递其它类型的数据时（int、String）却没有问题，显然问题出现了Parcelable身上，简单找了找答案
![](http://img.blog.csdn.net/20140924103915303)

---
**系统环境：**
Windows7 64位
**问题描述：**
使用开源控件NumberPicker，inflate时一直错误
**错误提示：**
``` bash
Android - Error inflating SimonVT NumberPicker class in my layout xml
```
**解决方案：**
activity的主题的numberPickerStyle item（numberpicker的主题）要使用它项目中的主题！坑爹···
``` xml
<item name="numberPickerStyle">@style/NPWidget.Holo.NumberPicker</item>
```

---
**系统环境：**
Windows7 64位
**问题描述：**
使用sherlockactionbar创建searchview一直报错
**错误提示：**
``` bash
sherlockactionbar Binary XML file line #29: Error inflating class
```
**解决方案：**
values-v11等其它资源文件夹里不是使用sherlockactionbar的主题！！！

---
**系统环境：**
Ubuntu12.04 64位
**问题描述：**
Android环境搭建完毕，但指定了sdk路径没问题依然报错，搭建JDK，Android环境，把android SDK复制过来后，里面的adb和其它命令的都不能使用。
**错误提示：**
``` bash
android-sdk-linux_86/platform-tools/adb: 没有那个文件或目录。
android-sdk-linux/platform-tools/adb: 没有那个文件或目录
AndroidSDK/sdk/build-tools/19.0.1/aapt: error while loading shared libraries:
Failed to get the adb version: Cannot run program "/home/android-sdk-linux/platform-tools/adb":
error=2, 没有那个文件或目录`
```
**解决方案：**
由于是64bit的系统，而Android sdk只有32bit的程序，需要安装ia32-libs，才能使用。
运行如下命令：
``` bash
# sudo apt-get install ia32-libs
```

---
**系统环境：**
Windows7 64位
**问题描述：**
SlidingMenu、ActionBarSherLock编译问题
**错误提示：**
...
**解决方案：**
1、新版的SlidingMenu-master需要使用google api编译。
2、SlidingMenu的library编译通过后，把编译好的ActionBarSherLock作为一个library导入SlidingMenu。
    导入方法是 右键-properties-android-add-选择ActionBarSherLock，因为SlidingMenu稍后也是以liberary的形式导入自己的项目中，所以此处勾选is a liberary。
3、新建项目，将SlidingMunu作为liberary导入，方法同上。
4、可能报找不到getSupportActionBar等ActionBarSherLock的方法。原因是使用ActionBarSherLock的Activity需继承于SherlockActivity，修改SlidingMenu liberary中的SlidingFragmentActivity，让它继承于SherlockFragmentActivity，重新编译liberary导入。
5、项目红叉或红叹号，删除support_v4包，ActionBarSherLock已包含此包，会冲突。也有可能是主题问题，注意appication theme是否正确，参照exsample。
6、注意把ActionBar、某些Fragment等替换成ActionBarSherLock包中的类。
7、左上角的指示图片是在application theme引用的style里设的。
8、 actionBar.setNavigationMode设置不同模式使用的监听类不同。

---
**系统环境：**
Ubuntu12.04 64位
**问题描述：**
真机连接无法识别。
**错误提示：**
``` bash
adb devices
List of devices attached
???????????? no permissions
```
**解决方案：**
1、设置usb权限
``` bash
$lsusb
Bus 005 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 004 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 003 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 002 Device 004: ID 093a:2510 Pixart Imaging, Inc. Hama Optical Mouse
Bus 002 Device 002: ID 413c:2003 Dell Computer Corp. Keyboard
Bus 002 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 001 Device 022: ID 0fce:6146 Sony Ericsson Mobile Communications AB 
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```
列表中，Bus 001 Device 022: ID 0fce:6146 Sony Ericsson Mobile Communications AB. 这一行为手机的usb使用端口，记录一下，id为0fce
``` bash
sudo gedit /etc/udev/rules.d/70-android.rules
```
加入以下内容：
SUBSYSTEM=="usb", ATTRS{idVendor}=="0fce", ATTRS{idProduct}=="6146",MODE="0666"
运行命令，重启udev：
``` bash
sudo chmod a+rx /etc/udev/rules.d/70-android.rules
sudo service udev restart
```
2、拔掉usb重新连上再执行：
//重新启动adb server
``` bash
sudo ./adb kill-server
./adb devices
./adb root
```
设置完成了
``` bash
adb devices
List of devices attached 
434235313151564C4D45    device
```

---
**系统环境：**
Windows7 64位
**问题描述：**
AndroidStudio无法更新(已加了google host)
**错误提示：**
``` bash
Connection failed. Please check your network connection and try again
```
**解决方案：**
修改安装目录下bin\studio.exe.vmoptions文件,如E:\Android\android-studio\bin\studio.exe.vmoptions，64位的还有studio64.exe.vmoptions
添加内容: 
```
-Djava.net.preferIPv4Stack=true 
-Didea.updates.url=http://dl.google.com/android/studio/patches/updates.xml
-Didea.patches.url=http://dl.google.com/android/studio/patches/
```
注意更新后这两个文件会被重新覆盖。

---
**系统环境：**
Windows7 64位
**问题描述：**
使用Jackson解析key为大写的json(复杂)数据报错
**错误提示：**
``` bash
Jackson with JSON: Unrecognized field, not marked as ignorable
```
**解决方案：**
1、给对应的解析实体类加上注解：
``` java
@JsonIgnoreProperties(ignoreUnknown = true)
```
2、在json解析处设置属性
``` java
ObjectMapper objectMapper = getObjectMapper();
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
```

---
**系统环境：**
Windows7 64位 Android Studio 1.0.2
**问题描述：**
run项目时报错
**错误提示：**
``` bash
Error :: duplicate files during packaging of APK
...
```
**解决方案：**
在出问题的module的build.gradle中的android节点加上：
``` gradle
packagingOptions {
        exclude 'META-INF/DEPENDENCIES'
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/license.txt'
        exclude 'META-INF/NOTICE'
        exclude 'META-INF/NOTICE.txt'
        exclude 'META-INF/notice.txt'
        exclude 'META-INF/ASL2.0'
    }
```

---
**系统环境：**
Windows7 64位 Android Studio 1.0.2
**问题描述：**
Android Studio导入项目后问题，无法run、rebuild等。
**错误提示：**
``` bash
Error:FAILURE: Build failed with an exception.
* What went wrong:
Task '' not found in root project 'android-project'.
* Try:
Run gradle tasks to get a list of available tasks. Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.
```
**解决方案：**
gradle tasks命令执行正常。一般都为Gradle的问题造成的。
从其它正常项目里复制替换掉当前项目的`gradlew`、`gradlew.bat`文件（也可自己视情况替换掉相关的gradle文件）
打开设置，打开Gradle选项，如图，点击使用Gradle home（你本地的Gradle目录），然后点击Apply和OK，还不行可以在Use选项中来回切换Apply一下，我就遇到点多几下才正常。。
![](http://rocko-blog.qiniudn.com/Android 错误集锦ing..._1.png)

---

**系统环境：**
Windows7 64 位 Android Studio 1.1.0
**问题描述：**
使用 ZoomButtonsController，在 Activity 重建后发生 ANR 异常退出
**错误提示：**
``` bash
 android.view.WindowLeaked: Activity org.rocko.touchlistener.subclasses.ZoomButtonsActivity has leaked window android.widget.ZoomButtonsController$Container@2b992988 that was originally added here
            ......
            at dalvik.system.NativeStart.main(Native Method)

org.rocko.touchlistener.subclasses E/ActivityThread﹕ Activity org.rocko.touchlistener.subclasses.ZoomButtonsActivity has leaked IntentReceiver android.widget.ZoomButtonsController$1@2b992778 that was originally registered here. Are you missing a call to unregisterReceiver()?
    android.app.IntentReceiverLeaked: Activity org.rocko.touchlistener.subclasses.ZoomButtonsActivity has leaked IntentReceiver android.widget.ZoomButtonsController$1@2b992778 that was originally registered here. Are you missing a call to unregisterReceiver()?
            ......
            at dalvik.system.NativeStart.main(Native Method)
01-31 18:28:29.022  11475-11475/org.rocko.touchlistener.subclasses E/AndroidRuntime﹕ FATAL EXCEPTION: main
    java.lang.IllegalArgumentException: Receiver not registered: android.widget.ZoomButtonsController$1@2b992778
            ......
            at dalvik.system.NativeStart.main(Native Method)

```
**解决方案：**
ZoomButtonsController 造成了内存泄露，在 onDestroy 中把其注销：
``` Java
    @Override
    protected void onDestroy() {
        super.onDestroy();
        zoomButtonsController.setVisible(false);
    }
```

---

**系统环境：**
Ubuntu 14.04 64 位 Android Studio 1.3.1
**问题描述：**
Android 系统源码（AOSP）导入 Android Studio 问题, idegen.jar 已经生成，但是执行生成项目文件的脚本一直报错。
**错误提示：**
``` Bash
Couldn't find idegen.jar. Please run make first.
```
**解决方案：**
可以直接在 `源码路径` 下执行 idegen.jar 的命令，`idegen.jar` 的路径换成你的位置
``` Bash
java -cp /repo/aosp/output/android-5.1.1_r12/host/linux-x86/framework/idegen.jar Main
```

---
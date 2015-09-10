title: VPS主机搭建Git服务器
date: 2015-02-12 00:06:07
category: Git
tags: [Git, VPS, 错误]
toc: true
---
## 前言
上篇说道搬瓦工上搞了个主机挂起了`SS`,单单用来干这个有些浪费了，所以它的第二个用处就是用来搭`Git服务`，以后私用的代码就往上堆了毕竟`Github`不能全都往上放，本地又有些不保险。所以就用VPS在搬瓦工（bandwagonhost）的Ubuntu 14.04上搭建了这个Git服务，因为过程有些曲折，所以记录一下，在出现错误时在错误方向上瞎折腾了一下午，悲伤，看来程序员要学会遇到bug时要保持淡定的心态，不要思路一走进就不会拐弯了，这点要牢记。

## 搭建
搭建一个几个人的小团队的Git服务器的方法比较简单，推荐的教程有[服务器上的 Git - 架设服务器](http://git-scm.com/book/zh/v1/%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84-Git-%E6%9E%B6%E8%AE%BE%E6%9C%8D%E5%8A%A1%E5%99%A8)、[搭建Git服务器](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000)，本文不再介绍因为本文主要说的是在这个过程中的错误。当然也可以使用`gitosis`等方式搭建更为强大、控制力更强的Git服务器。

<!--more-->
 
## 错误
在VPS上按标准教程搭建好服务器上的Git服务后，兴高采烈地在自己电脑上(`Windows7 64`，如果你同样使用Windows而且使用SSH遇到问题那么可以看看我的这篇博客：[Git 错误集锦ing...](http://rocko.xyz/2014/11/25/Git-%E9%94%99%E8%AF%AF%E9%9B%86%E9%94%A6ing/))clone了服务器上的repo后，然后就出错误了（如果你也是用VPS搭建而使用其它操作系统客户端的话可能也是会遇到）：
``` bash
Cloning into 'sample'...
ssh: connect to host 104.224.xxx.xx port 22: Bad file number
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
使用SSH测试的结果，-T参数是直接显示连接结果，-V参数可以打印所有过程：
``` bash
D:\Repositories>ssh -v git@104.224.xxx.xx
OpenSSH_4.6p1, OpenSSL 0.9.8e 23 Feb 2007
debug1: Reading configuration data /c/Users/Administrator/.ssh/
debug1: Applying options for 104.224.xxx.xx
debug1: Connecting to 104.224..xxx.xx [104.224..xxx.xx] port 22.
debug1: connect to address 104.224.xxx.xx port 22: Attempt to c
without establishing a connection
ssh: connect to host 104.224.xxx.xx port 22: Bad file number
```
其实这个ssh连接测试结果已经可以看出端倪了，IP是没问题的，就出在端口22上，有经验的看到应该就会有反应的了，而我则是跑在服务器上瞎折腾，又是重装、换平台的。。我们知道，VPS上会给我们`IP、ssh远程连接的端口、系统账号、账号密码`，这里关键就是ssh的端口了，一般默认的端口是`22`的，但是VPS上一般给我们的有所差异，像搬瓦工一般给我们的连接端口都是5位数的。解决方法就是把自己电脑上git ssh连接的端口改为VPS的ssh端口号即可，怎么配置呢，我们需要修改ssh的配置信息 `~/.ssh/config` 就是放你密钥的目录下，没有`config`文件就新建, 内容如下：
``` bash
Host 104.224.xxx.xx
User zhengxiaopeng
Hostname 104.224.xxx.xx
Port 27382 #这个一定要改为VPS上ssh的连接端口
```
然后就OK了。
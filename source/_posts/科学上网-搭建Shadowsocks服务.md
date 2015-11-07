title: 科学上网-搭建Shadowsocks服务
date: 2015-02-09 16:49:40
category: FGFW
tags: [FQ, VPS]
toc: true
---
## 前言
生命不息，折腾不止，方校长的丰功伟绩我天朝程序员为之叹息，说多了都是泪。既然现在不太存在把Q推倒的可能，那我们就只好自个F过去，之前一直用都在用[Goagent](https://github.com/goagent/goagent)上Youtube看视频，一卡一卡的，而且Goagent也要常更新才能使用，比较蛋疼。而用Hosts上谷歌虽然是一种最直接的方法，但要找到一封就更新IP的服务比较少，这里推荐两个：[google-hosts](https://github.com/txthinking/google-hosts), [netsh](http://www.netsh.org/)，这些hosts都是说不定突然就会被封的，但自己使用来看netsh目前还挺靠谱，把它放在hosts那里也还不错的。额，其它的不扯了，下面来把Shadowsocks搭起来。

## VPS的选择
VPS自己用的不多只根据网上的各种对比测评，最终还是选择了`搬瓦工(Bandwagon Host logo)`，因为够便宜，选了个4.99刀一年的，人民币31.24元。配置：`112M内存 24Mswap 3G HDD硬盘`，够用了，其它的配置价格[戳这里](https://bandwagonhost.com/cart.php)，当然也可以使用[我的优惠链接](https://bandwagonhost.com/aff.php?aff=2049)。个人的话如果能申请到[Student Developer Pack](https://education.github.com/pack)的选择[DigitalOcean](https://www.digitalocean.com/)最好不过了，DigitalOcean还是按小时计费的，不用的话可以暂停掉不再计费，成功申请有Student Developer Pack能有100刀送，最低配置的能用两年了，无奈我的申请杳无音讯。。这里VPS就选好了，把支付链接什么的先弄好等待下一步。

<!--more-->

## VPS费用支付
有信用卡的最好了，没有的最好选择[Paypal](https://www.paypal.com)支持银联支付，一般有国内银行卡即可，注册也很简单，个人信息最好如实填写。然后从搬瓦工的购买链接点击进去按它的提示一步一步把钱送到它口袋即可，支付成功后等下下就会有几条邮件发往你邮箱了，其实都可以不看，直接去搬瓦工后台，Services->My Services->Kiwivm Control Panel。

## 主机配置
先上个控制台的图：
![VPS控制面板](http://rocko-blog.qiniudn.com/科学上网-搭建Shadowsocks服务-1.png?imageView2/2/w/900/h/500/q/100)
### 重装系统
现在控制台Main Controls选项里把系统Stop掉，再来到Install new OS，选择ubuntu-14.04-x86_64-minimal然后agree Reload等待几秒即可，很快。然后就是在机器上装Shadowsocks前先SSH连上机器，Linux下SSH连直接命令行即可操作，不细述了。Windows下我使用[Putty](http://www.putty.org/)(官网被墙，[点我下载](http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe)),然后按下图填写选择，IP即为控制面板上的`IP addres`，端口为控制面板上的`SSH Port`，最后Open。
![Putty内容](http://rocko-blog.qiniudn.com/科学上网-搭建Shadowsocks服务-2.png?imageView2/2/w/500/h/500/q/100)
连接上后就是正常的Linux终端了，输入用户名密码，我们选择root登陆，拥有最高权限所以操作要小心。名字：`root`，密码：点击控制面板上的`Root password modidication`重新`Generate`一个新的root密码，复制保存下来，也可以随意重新生成，输入终端回车进入，如下：
![用户登陆后](http://rocko-blog.qiniudn.com/科学上网-搭建Shadowsocks服务-3.png)
然后就可以安装Shadowsocks,可以看[wiki教程](https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E),推荐使用配置文件开启服务，过程如下：
``` bash
root@rocko:~# apt-get update
root@rocko:~# apt-get install python-pip
root@rocko:~# pip install shadowsocks
```
安装完毕使用vim新建配置文件：
``` bash
root@rocko:~# vim /etc/shadowsocks.json
```
然后写入如下配置，server和password换点就好其它随意，`:w`保存、不再修改`:q`退出：
``` vim
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
最后就是在后台开启/关闭服务命令：
``` bash
root@rocko:~# ssserver -c /etc/shadowsocks.json -d start
root@rocko:~# ssserver -c /etc/shadowsocks.json -d stop
```

## 各平台使用
安装各[平台下的Shadowsocks客户端](https://github.com/shadowsocks/shadowsocks/wiki/Ports-and-Clients)，基本上都是填入`/etc/shadowsocks.json`的对应信息即可，最后打开Google，成功！！！
可以使用命令查看日志信息：
``` bash
root@rocko:~# cat /var/log/shadowsocks.log
``` 
然后就可以看到运行记录，最明显的就是你客户端上的IP了，对比一下，就是自己~。不过时间不是中国的，看起来不直接所以我们可以把系统时区改在中国，执行以下命令
``` bash
root@rocko:~# dpkg-reconfigure tzdata
```
按着提示选就可以，截图如下：
![市区更改过程图](http://rocko-blog.qiniudn.com/科学上网-搭建Shadowsocks服务-4.png?imageView2/2/w/800/h/500/q/100)

## End
整个过程还是很顺畅的，如果都有必要的账号信息10分钟搞定是没问题的。体验体验之后下一步就是优化了，当然这也是后话了，弄好后有什么特别的再贴出来。
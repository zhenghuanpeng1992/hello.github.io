title: Python小程序-多说评论通知
date: 2015-03-10 00:40:06
category: Python
tags: [多说, Hexo]
toc: true
---

## 前言

这段时间在学习Python，写了第一个稍有实用价值的Python脚本小程序，发文以作纪念。实现多说评论邮件通知的功能：利用[多说的API](http://dev.duoshuo.com/docs)定时去检查后台数据从而检测是否有新评论的数据产生，有的话就发邮件到指定邮箱。之所以做这个是因为多说官方的方法一直尝试失败，可以戳这个：[同步用户到多说实现文章被评论时的提醒功能](http://haomwei.com/technology/import-users-to-duoshuo.html)，作者用户是导入了，但测试发现一直没反应；另外一个就是多说官方的通知不是及时的，一天一封吧，强迫症患者可能受不了。写的不好求轻喷。

## 撸码

### 配置
方便他人修改使用。
``` bash
[email_info]
email_host = 电子邮件主机，如：smtp.qq.com，注意你的邮箱是否开启了POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务
from_address = 你要发送邮件的邮件地址
password = 邮箱的登陆密码
to_address = 你要接受邮件的地址

[duoshuo_account]
name = 多说的二级域名名, 如我的：rocko
secret = 多说的秘钥，在后台的设置查看

[period_time]
period = 定时检查评论的时间（s）
```

### Python代码
代码比较简单，已加注释。用到多说的接口是这个，[实时同步评论回本地数据库](http://dev.duoshuo.com/docs/50037b11b66af78d0c000009)。需要注意的是在后台操作和文章有新评论时就会产生log的json数据返回，返回的数据可以去它的文档看一下，然后就找自己感兴趣的数据了，做为我们判断是否是文章有评论而不是自己在后台产生的是`action`这个字段，为**create**时就是我们要的。还有就是请求时的`limit`参数，文档上说默认是50，所以如果你没加使用了默认而你的log数>50时那就只能获取到50条了，不过是<200但我使用>200时区请求是没有问题的。

<!--more-->

``` Python
#! /usr/bin/env python
# -*- coding: utf-8 -*-

from email.mime.text import MIMEText
from email.utils import parseaddr, formataddr
# http://requests-docs-cn.readthedocs.org/zh_CN/latest/user/quickstart.html
import requests
import smtplib
import time
import ConfigParser


def monitor():
    # 加载配置文件信息
    config = ConfigParser.ConfigParser()
    config.read('./ds.config') # 当前目录下的ds.config
    # 初始化配置信息
    duoshuo_account = {}
    email_info = {}
    period_time = {}
    items2dict(duoshuo_account, config.items('duoshuo_account'))
    items2dict(email_info, config.items('email_info'))
    items2dict(period_time, config.items('period_time'))

    # 第一次获取账户后台的初始信息
    current_count, meta = get_duoshuo_log(duoshuo_account)
    last_count = current_count
    # print last_count
    name = duoshuo_account.get('name')
    period = int(period_time.get('period'))
    while True:
        # print '>>>>>get_duoshuo_log'
        try:  # 防止get_duoshuo_log和send_email挂掉
            current_count, meta = get_duoshuo_log(duoshuo_account)
            # send_email(email_info, name, current_count, (current_count - last_count), meta)
            # print current_count
            # print str(meta)
            if (len(meta)) > 0 and (current_count > last_count):
                send_email(email_info, name, current_count, (current_count - last_count), meta)
                last_count = current_count

            time.sleep(period)
        except Exception, e:
            # print 'Error:', str(e)
            time.sleep(period)


# 把option的items映射到dict中
def items2dict(options_dict, items_list):
    for item in items_list:
        options_dict[item[0]] = item[1]


# 获取多说账户的后台信息og
def get_duoshuo_log(duoshuo_account):
    url = 'http://api.duoshuo.com/log/list.json?' \
          + 'short_name=' + duoshuo_account.get('name') \
          + '&secret=' + duoshuo_account.get('secret') + '&limit=5000'
    # print url
    r = requests.get(url)
    resp = r.json()

    if (resp['code'] == 0):  # code为0时才是正常的log的json信息
        length = len(resp['response'])
        meta = resp['response'][length - 1]['meta']
        action = resp['response'][length - 1]['action']
        if (action == 'create'):  # action为create时才是新增评论，去除其它如delete等操作的影响
            return length, meta
        else:
            return length, {}

    return 0, {}


# 发送邮件
def send_email(email_info, name, current_count, count, meta):
    # print '>>>send email'
    last_meta_message = u'最新评论信息：' \
                        + u'\n用户地址：' + unicode(meta.get('ip')) \
                        + u'\n用户昵称：' + unicode(meta.get('author_name')) \
                        + u'\n用户邮箱：' + unicode(meta.get('author_email')) \
                        + u'\n用户网站：' + unicode(meta.get('author_url')) \
                        + u'\n评论时间：' + unicode(meta.get('created_at')) \
                        + u'\n评论内容：' + unicode(meta.get('message')) \
                        + u'\n审核状态：' + unicode(meta.get('status'))

    duoshuo_admin_url = 'http://' + name + '.duoshuo.com/admin/'
    text = u'后台记录变更数：' + str(count) + u'\n多说后台：' + duoshuo_admin_url + u'\n\n' + last_meta_message;
    # print text
    msg = MIMEText(text, 'plain', 'utf-8')
    msg['Subject'] = u'多说评论通知 #' + str(current_count)
    msg['From'] = email_info.get('from_address')
    msg['To'] = email_info.get('to_address')
    # print msg
    server = smtplib.SMTP()
    server.connect(email_info.get('email_host'))
    server.login(email_info.get('from_address'), email_info.get('password'))
    server.sendmail(email_info.get('from_address'), [email_info.get('to_address')], msg.as_string())
    server.close()


if __name__ == '__main__':
    monitor()
```

### 开机自启

在自己的VPS里往Linux系统的`/etc/rc.local`里的`exit 0`前加上启动程序的命令
``` bash
nohup python your_path/ds.py &
```
Windows的开机自启动方法，[戳我](http://www.cnblogs.com/MikeZhang/archive/2013/02/04/pythonautorunwindows20130204.html)。

### 测试效果
![测试效果](http://rocko-blog.qiniudn.com/Python小程序-多说评论通知-1.png)

## End
代码放到了Github上，[duoshuo-comment-notification](https://github.com/zhengxiaopeng/duoshuo-comment-notification)，有需要的去看看。
Python的函数设计的不错，书写简单功能强大。编码格式是个大坑~，缩进识别代码段（域）有些蛋疼。

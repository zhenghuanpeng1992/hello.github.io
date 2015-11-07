title: Git 错误集锦ing...
date: 2014-11-25 18:21:52
category: Git
tags: [Git, 错误]

---
**系统环境：**
Windows7 64位
**问题描述：**
Windows使用Git客户端gitscm时，当clone或commit时报错。
**错误提示：**
``` bash
warning: templates not found /share/git-core/templates
fatal: Unable to find remote helper for 'https'
```
**解决方案：**
把Git换个安装目录重新安装问题解决!!!具体原因找了很久未明，估计不知怎么把Git搞乱了。

---
**系统环境：**
Windows7 64位
**问题描述：**
ssh-keygen命令报错，一般在Windows上配置git的ssh会出现这个问题
**错误提示：**
``` bash
Could not create directory '//.ssh'.
...
```
**解决方案：**
找不到主目录，在系统环境变量配置`HOME`变量：系统变量上新建->变量名：`HOME`、变量值：`C:\Users\Administrator`,Administrator为你当前的用户名。

---


<!--more-->
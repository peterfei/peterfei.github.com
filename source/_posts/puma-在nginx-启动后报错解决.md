title: puma 在nginx 启动后报错解决
date: 2015-10-31 11:46:44
tags:
---
##在Centos上部署时遇到如下错误：
>puma nginx connect() Permission denied

解决方法：在nginx.conf 配置文件中加入了puma 用户：
`user deploy` 
>puma 启动时的用户，如此启动正常。

title: Ionic 更新默认图标
date: 2015-11-04 13:52:01
tags:
categories: Ionic
---
项目中要替换ionic 默认的icon 图标，`Google` 一下，大多是让图标生成svg的格式，下面有一种比较容易的方式去实现：
>e.g. 在浏览器的调试模式，选中需要替换的图标，找到如下代码：
```
.ion-ios-home-outline:before {
    content: "\f447";
   }
```
在/img/找出要替换的icon图，如`car.png`
在样式表里写入如下代码：
```
.ion-ios-home-outline:before {
    /*content: "\f447";*/
    content: url('../img/car.png') !important;
}
```
![效果图](http://7xjzsc.com1.z0.glb.clouddn.com/icon-finsih.png)

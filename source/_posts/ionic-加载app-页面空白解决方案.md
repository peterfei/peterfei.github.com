title: 'ionic 加载app,页面空白解决方案'
date: 2016-05-21 10:55:25
tags:
- ionic 
- splash
categories: Ionic
---
给公司开发的app 经常会遇到打开加载空白页问题，虽然已经装了`cordova.splashscreen.SplashScreen`,但问题依然存在。
Google之后得到如下解决方案：
```
  <preference name="AutoHideSplashScreen" value="false"/>
  <preference name="ShowSplashScreenSpinner" value="false"/>
  <preference name="SplashMaintainAspectRatio" value="true"/>
  <preference name="SplashShowOnlyFirstTime" value="false"/>
  <preference name="SplashScreenDelay" value="10000"/>
  <preference name="FadeSplashScreen" value="false"/>
  <feature name="SplashScreen">
    <param name="android-package" value="org.apache.cordova.splashscreen.SplashScreen"/>
  </feature>
```
在app.js里加入：
```
if (window.cordova && window.cordova.plugins && window.cordova.plugins.Keyboard) {
            cordova.plugins.Keyboard.hideKeyboardAccessoryBar(true);
            cordova.plugins.Keyboard.disableScroll(true);
            setTimeout(function () {
                navigator.splashscreen.hide();
            }, 100);
}
```

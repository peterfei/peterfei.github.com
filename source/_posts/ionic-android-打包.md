title: ionic android 打包
date: 2015-08-05 10:54:14
tags:
- ionic 
- android
-  打包
categories: 前端
---


*生成证书*

	`keytool -genkey -v -keystore **-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000`

*生成release 版本:*
	
	cordova build --release android

*jarsinger:*

	jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore ezu110-release-key.keystore /Users/liufei/ionic/yongche/platforms/android/build/outputs/apk/android-release-unsigned.apk  alias_name
	
*生成apk:*

	/Library/Android/android-sdk-macosx/build-tools/22.0.1/zipalign -v 4 /Users/liufei/ionic/yongche/platforms/android/build/outputs/apk/android-release-unsigned.apk ezu110.apk
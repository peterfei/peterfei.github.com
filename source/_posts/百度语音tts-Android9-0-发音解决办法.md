title: 百度语音tts Android9.0 发音解决办法
date: 2019-04-08 16:58:00
tags: 百度 android9.0
categories: 前端
---
百度语音在Android9.0不能发音解决办法:
在 `AndroidManifest.xml` `<application>`下加入:

```
<uses-library android:name="org.apache.http.legacy" android:required="false"/>
```
title: Android 调试|日志脚本
date: 2019-04-08 17:04:56
tags: Android
categories: 前端
---
```bash
#!/bin/bash
PackageName=$1

#PackageName=com.example.appinfomanagertinno
pid="$(adb shell ps | grep $PackageName | awk '{print $2}')"

echo "PackageName-----"
echo "$PackageName"

echo "-----------------------------------------"
echo "-----------------------------------------"

echo "pid-----"
echo "$pid"

echo "-----------------------------------------"
echo "-----------------------------------------"

adb logcat | grep $pid

```
> 使用方法
> `./logcat_package.sh com.XXX.XXX`
---
layout: post
title: "ionic学习笔记1"
date: 2015-07-03 09:24:56 +0800
comments: true
tags:
- ionic
- 打包
- EAEEES ERROR
categories: 
---
#Ionic Add Platform EACCES Error 解决办法
*当出现` platform add [platformname]`

Error: spawn EACCES
at exports._errnoException (util.js:746:11)
at ChildProcess.spawn (child_process.js:1155:11)
at Object.exports.spawn (child_process.js:988:9)
at Object.exports.spawn (/usr/local/lib/node_modules/cordova/node_modules/cordova- lib/src/cordova/superspawn.js:100:31)
at runScriptViaChildProcessSpawn (/usr/local/lib/node_modules/cordova/node_modules/cordova-
lib/src/hooks/HooksRunner.js:188:23)
at runScript (/usr/local/lib/node_modules/cordova/node_modules/cordova- lib/src/hooks/HooksRunner.js:131:16)
at /usr/local/lib/node_modules/cordova/node_modules/cordova- lib/src/hooks/HooksRunner.js:114:20
at _fulfilled (/usr/local/lib/node_modules/cordova/node_modules/q/q.js:787:54)
at self.promiseDispatch.done (/usr/local/lib/node_modules/cordova/node_modules/q/q.js:816:30)
at Promise.promise.promiseDispatch (/usr/local/lib/node_modules/cordova/node_modules/q/q.js:749:13)
*
  
加入`ionic hooks add`
then
`ionic platform add ios
`
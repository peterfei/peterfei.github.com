title: ionic 发布为webserver
date: 2015-07-13 17:41:38
tags:
- ionic
- webserver
categories: Ionic
---
*公司在做ionic开发时，遇到要发布为webserver:*
```nodejs
$ cd [ionic project]
$ ionic platform add browser
$ cd [ionic project]/platforms/browser/    
```

安装 `Install Node.js`

*Install connect and serve-static*
`$ cd [webapp] $ npm install connect serve-static`
*编写server.js*
```
var connect = require('connect');
var serveStatic = require('serve-static');
connect().use(serveStatic(__dirname)).listen(8080)
```
*运行：*
```$ node server.js &```
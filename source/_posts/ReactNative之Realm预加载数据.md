title: ReactNative之Realm预加载数据
date: 2019-04-11 11:28:51
tags:
- react-native
- realm
categories: 前端
---
> realm 数据预加载


##### Require Env:


```
npm i realm
npm i react-native-fs
react-native link 

```

分别拷贝数据库至原生公用目录

##### Android端
`android/app/src/main/assets/demo.realm`

##### IOS端

`Build Phases->Copy Bundle Resources->demo.realm`

##### 前端
执行Realm 提供的copyBundledRealmFiles,可以将共用的数据库文件拷贝至应用安装的`Documents`目录.
> `Realm.copyBundledRealmFiles();	`


##### 实例
##### [prepare-realm-data-demo](https://github.com/peterfei/prepare-realm-data-demo)



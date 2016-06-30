title: ionic 手动清除页面缓存方法
date: 2015-12-08 16:23:31
tags:
- ionic 
- angular
---
ionic手动清除页面方法：

```javascript
$scope.$on("$ionicView.beforeEnter", function () {
  $ionicHistory.clearCache();
  $ionicHistory.clearHistory();
});

```
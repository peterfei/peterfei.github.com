title: Angularjs 判断服务器401错误
date: 2015-10-21 14:42:53
tags:
categories: 前端
---
之前在`github`上有篇判断`augularjs`401状态码的:
```javascript
var logsOutUserOn401 = function($location, $q, SessionService, FlashService) {
    var success = function(response) {
      return response;
    };

    var error = function(response) {
      if(response.status === 401) {
        SessionService.unset('authenticated');
        $location.path('/login');
        FlashService.show(response.data.flash);
      }
      return $q.reject(response);
    };

    return function(promise) {
      return promise.then(success, error);
    };
  };

  $httpProvider.responseInterceptors.push(logsOutUserOn401);
```
今天用时才发现`$httpProvider.responseInterceptors`方法已经被官方弃用了。
改写了方案如下：
```javascript
app.factory('errorInterceptor', ['$q', '$rootScope',  '$location','FlashService',
    function ($q, $rootScope, $location,FlashService) {
        return {
            request: function (config) {
                return config || $q.when(config);
            },
            requestError: function(request){
                return $q.reject(request);
            },
            response: function (response) {
                return response || $q.when(response);
            },
            responseError: function (response) {
                if(response.status === 401) {
                  
                  // SessionService.unset('authenticated');
                  FlashService.show(response.data.error);
                  $location.path('/login');
                 
                }
                if (response && response.status === 404) {
                }
                if (response && response.status >= 500) {
                }
                return $q.reject(response);
            }
        };
}]);

```
在app.js config 方法调用时：
```javascript
$httpProvider.interceptors.push('errorInterceptor');
```
title: ionic 发送POST请求
date: 2015-07-22 00:40:03
tags:
- ionic 
- angularjs
---
*今天写一个`POST`请求给远程API，`FORM`数据一直不能发送，最终用如下factory解决问题：*
****************************************
```javascript
.factory('LoginService', ['$http', function($http) {
        var users = [];
        return {
            getUsers: function(loginData) {
                var username = loginData.username,
                    password = loginData.password;
                return $http({
                    method: 'POST',
                    url: 'http://172.16.100.222/zf/app/login.php?act=login',
                    data: $.param(loginData),
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded'
                    }
                }).success(function(response) {
                    // debugger
                    users = response;
                    return users;
                });
            }
	}

   }])
```
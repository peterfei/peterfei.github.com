title: ionic 页面对象传值
date: 2015-12-28 16:13:15
tags:
- ionic
- obj
categories: Ionic
---

ionic 项目里常常遇到页面间传值的情况，一般是这样的：
```
.state('tab.chat-detail', {
				url: '/chats/:chatId',
				views: {
					'tab-chats': {
						templateUrl: 'templates/chat-detail.html',
						controller: 'ChatDetailCtrl'
					}
				}
			})
```

但遇到对象时,应做如下处理:
```
$stateProvider.state('users', {
    url: '/users',
    controller: 'UsersCtrl',
    params: {
        obj: null
    }
})
```

```
function UserCtrl($stateParams) {
    conrole.log($stateParams);
}
```

```
$state.go('users', {obj:yourObj});
```
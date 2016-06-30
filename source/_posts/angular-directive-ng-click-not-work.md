title: angular directive ng-click not work
date: 2015-12-07 15:32:14
tags:
- ionic 
- angularjs
- directive
---
###隔离事件
中午在项目里遇到angular directive 里加入click事件，不能正常工作，查询API后，基本可以通过angularjs 的隔离scope 完成，具体实现如下：
>html

```html
<mobi edit-click="selectMobiFlag(1)" item="item"></mobi>
```
html 端定义了`mobi`的指令， 具体点击事件方法如`selectMobileFlat`,
>指令（Directive）

```js
.directive('mobi', function() {
	return {
		restrict: 'EA',
		scope: {
			'item':'=',
			'editClick': '&'
		},
		template: '<div ng-click="toggleState(item)" >111</div>',
		link: function($scope, element, attrs) {
			$scope.toggleState = function(item){
				console.log(item);
				$scope.editClick(item);
			};

		}
	};
})
```

指令里绑定了item,传入的是点击 `HTML`片段本身，在`directive`里渲染了`template` 模板，主要负责点击的声明，在隔离作用域里，应用`editClick:"&"` 和 `$scope.toggleState` 实现关联。
###最后
实现`selectMobiFlag`的具体实现:
```js
$scope.selectMobiFlag = function(item) {
	
	$scope.mobiFlag = item;
}
```

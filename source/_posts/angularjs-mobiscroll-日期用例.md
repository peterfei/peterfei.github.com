title: angularjs mobiscroll 日期用例
date: 2015-10-27 16:23:20
tags:
---
Angularjs 写了个mobiscroll directive ：
```javascript
	.directive('mobiDateTimePicker', function() {
	  return {
	    restrict: 'A',
	    link: function($scope, element, attrs) {
	      return $(element).mobiscroll().datetime({
	        theme: 'default',
	        display: 'bottom',
	        lang:'zh',
	        minWidth:40,
	        steps:{  minute: 15},
	        rows:3,
	        // dayText: '日', monthText: '月', yearText: '年',
	        dateFormat: 'yy-MM-dd',
	        timeFormat: 'HH:ii:ss',
	        // timeWheels: 'HHii',
	        showNow: true, 
	        showLabel:true,
	        dateOrder: 'yyyyMMddDD',
	        // tap:true,
	        // invalid:[{'10/27',start:'18:00',end:"19:00"}]
	        headerText: function (valueText) { //自定义弹出框头部格式
	            return valueText;
	        },
	        // dayNames:['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday']
	
	      });
	    }
	  };
	})
```
其中：`minWidth:40`是控件的显示高度，`dateOrder: 'yyyyMMddDD',`可显示友好的星期+日期：
![](http://7xjzsc.com1.z0.glb.clouddn.com/15-10-27/48605780.jpg)

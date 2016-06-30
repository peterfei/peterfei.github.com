title: AngularJS 指令的 Scope (作用域)
date: 2016-04-19 21:58:37
tags:
---
当一个指令被创建的时候，都会有这样一个选择，是继承自己的父作用域（一般是外部的Controller提供的作用域或者根作用域（$rootScope）），还是创建一个新的自己的作用域，当然AngularJS为我们指令的scope参数提供了三种选择，分别是：`false`,`true`,`{}`；默认情况下是false。
JS:
```javascript
    angular.module("MyApp", [])
        .controller("MyController", function ($scope) {
        $scope.name = "dreamapple";
        $scope.age = 20;
        $scope.changeAge = function(){
            $scope.age = 0;
        }
        })
        .directive("myDirective", function () {
        var obj = {
            restrict: "AE",
            scope: {
                name: '@myName',
                age: '=',
                changeAge: '&changeMyAge'
            },
            replace: true,
            template: "<div class='my-directive'>" +
                "<h3>下面部分是我们创建的指令生成的</h3>" +
                "我的名字是：<span ng-bind='name'></span><br/>" +
                "我的年龄是：<span ng-bind='age'></span><br/>" +
                "在这里修改名字：<input type='text' ng-model='name'><br/>" +
                "<button ng-click='changeAge()'>修改年龄</button>" +
                " </div>"
        }
        return obj;
        });
```

HTML:
```html
<div ng-app="MyApp">
    <div class="container" ng-controller="MyController">
        <div class="my-info">我的名字是：<span ng-bind="name"></span>
            <br/>我的年龄是：<span ng-bind="age"></span>
            <br />
        </div>
        <div class="my-directive" my-directive my-name="{{name}}" age="age"  change-my-age="changeAge()"></div>
    </div>
</div>
```

`@`
这是一个单项绑定的前缀标识符
使用方法：在元素中使用属性，好比这样`<div my-directive my-name="{{name}}"></div>`

注意，属性的名字要用-将两个单词连接，因为是数据的单项绑定所以要通过使用`｛｛｝｝`来绑定数据。


`=`

这是一个双向数据绑定前缀标识符
使用方法：在元素中使用属性，好比这样`<div my-directive age="age"></div>`,注意，数据的双向绑定要通过=前缀标识符实现，所以不可以使用*｛｛｝｝*。

`&`

这是一个绑定函数方法的前缀标识符
使用方法：在元素中使用属性，好比这样**`<div my-directive change-my-age="changeAge()"></div>`**，注意，属性的名字要用-将多个个单词连接。

>注意：在新创建指令的作用域对象中，使用属性的名字进行绑定时，要使用驼峰命名标准，比如下面的代码。


```
scope: {
            // `myName` 就是原来元素中的`my-name`属性
            name: '@myName', 
            age: '=',
            // `changeMyAge`就是原来元素中的`change-my-age`属性
            changeAge: '&changeMyAge' 
        }
```


1.  @ 当指令编译到模板的name时，就会到scope中寻找是否含有name的键值对，如果存在，就像上面那样，看到@就知道这是一个单向的数据绑定，然后寻找原来的那个使用指令的元素上（或者是指令元素本身）含有这个值的属性即my-name={{name}}，然后在父作用域查找{{name}}的值，得到之后传递给模板中的name。
2.  =和&与@差不多，只不过=进行的是双向的数据绑定，不论模板还是父作用域上的属性的值发生改变都会使另一个值发生改变，而&是绑定函数而已。


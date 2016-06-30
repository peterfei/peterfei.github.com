title: ionic 生成动态生成右拉城市菜单
date: 2015-12-30 09:59:35
tags:
- Ionic
- ion-side-menus
- $ionicSideMenuDelegate
---
昨天有时间想重写下之前技术部小黄写的城市右拉， 主要用的功能是`ionic` 自带的 `ion-side-menus` 和 `ionicSideMenuDelegate`。
配置路由：
```
.state('choosePovince',{
  cache:false,
  url:'/choose_port',
  abstract:true,
  templateUrl:'templates/jiesongji/choose_porvince_main.html',
  controller:'JiejiCtrl'
})

/**
 * 右侧滑动
 * @type {应用框架自带的滑动更好解决}
 */
.state('choosePovince.city',{
  cache:false,
  url:'/city',
  views:{
    'menuContent' :
    {
      templateUrl: "templates/jiesongji/porvince_main.html",
      controller:'JiejiCtrl'
    },
    'right-menu':
    {
      templateUrl : "templates/jiesongji/city_right.html",
      controller:'JiejiCtrl'
    }
  },
  params: {
    p_obj: null
  }
})

```
第一层路由是声明右拉的父级关联模板。
`Html`模板`choose_porvince_main.html`：
```html
<ion-side-menus>
        <ion-pane ion-side-menu-content>
          <ion-nav-bar class="bar bar-header top-header">

              <ion-nav-buttons>
                <button class="button button-clear" ng-click="goBack()"><i class="icon ion-chevron-left"></i></button>
              </ion-nav-buttons>
              <ion-nav-back-button class="button-clear"><i class="icon ion-chevron-left"></i> Back</ion-nav-back-button>
          </ion-nav-bar>
            <ion-nav-view name="menuContent" animation="slide-left-right"></ion-nav-view>
        </ion-pane>
        <ion-side-menu side="right">
              <header class="bar bar-header top-header">
                  <h1 class="title">选择城市</h1>
              </header>
              <ion-content class="has-header">
                <ion-nav-view name="right-menu">
                </ion-nav-view>
              </ion-content>
        </ion-side-menu>

</ion-side-menus>

```
第二层路由声明右拉的模板与主模板。
`Html` 模板 `porvince_main.html`：
```html
<ion-view title="选择城市">
  <ion-content>
      <ion-list>
        <ion-item ng-click="right_menu(province.pro_id);" ng-repeat="province in ProvinceList">
        <!-- <ion-item ng-click="show_province();"> -->

             <span>{{province.province}}</span>

        </ion-item>
      </ion-list>
  </ion-content>
</ion-view>

```
`Html` 模板 `city_right.html`
```html
<ion-item ng-repeat="city in citys" style="height:50px;">
        <span>{{city.city_name}}</span>
        <span></span>
</ion-item>
```

`Javascript`:
```javascript
/**
 * [show_province 加载省份]
 * @return {[type]} [description]
 */
$scope.show_province = function(){
    var param = {
        code: "25c85dd791fd81f528eb550d6c107dcf"
    };
    chaXunProvinceFactory.getChaXunProvince(param).then(function(callback) {
        if (callback.status == 200) {
            // debugger
            $state.go('choosePovince.city',{p_obj:callback.data.content});
            // $scope.ProvinceList = callback.data.content;
        }
    });

}
/**
 * 通过路由的对象传值，渲染ProvinceList显示数据
 * @param  {[type]} angular.isObject($stateParams.p_obj) [description]
 * @return {[type]}                                      [description]
 */
if (angular.isObject($stateParams.p_obj)) {
    $scope.ProvinceList=$stateParams.p_obj;
};
/**
 * 这里是重点，当点击事件发生时，向父级scope对象发送监听对象请求，传值。
 * @param  {[type]} obj [description]
 * @return {[type]}     [description]
 */
$scope.right_menu = function(obj){
    $ionicSideMenuDelegate.toggleRight();
    $scope.$emit('cities',obj);


}
/**
 * 父页面监听，如果接收到点击请求，则通过接口获取动态数据，同时赋值给$scope对象。
 * @param  {[type]} e    [description]
 * @param  {Object} d){                var param [description]
 * @return {[type]}      [description]
 */
$scope.$on('cities',function(e,d){
   var param = {
        code: "25c85dd791fd81f528eb550d6c107dcf",
        pro_id: d
    };

    chaXunProvinceFactory.getChaXunCity(param).then(function(callback) {
        if (callback.status == 200) {
            $scope.citys = callback.data.content;
        }
    });
});
```

最终效果：
![效果图](http://7xjzsc.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202015-12-30%20%E4%B8%8B%E5%8D%881.47.50.png)
![效果图](http://7xjzsc.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202015-12-30%20%E4%B8%8B%E5%8D%881.48.15.png)

title: IOS swift UITabBarController 开发
date: 2016-09-09 15:30:42
tags:
- ios
- swift
- UITabBarController
---
Swift 开发中，省去storyboard,应用UITabBarController也可以生成:
![效果](http://pic.yupoo.com/peterfei_s/FQ13FI9K/Pi7pg.png)
首页写`AppDelegate.swift`:
```swift

import UIKit

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?


    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        window = UIWindow(frame:UIScreen.mainScreen().bounds)
        window?.rootViewController = YMTabBarController()
        window?.makeKeyAndVisible()
        return true
    }

    func applicationWillResignActive(application: UIApplication) {
        // Sent when the application is about to move from active to inactive state. This can occur for certain types of temporary interruptions (such as an incoming phone call or SMS message) or when the user quits the application and it begins the transition to the background state.
        // Use this method to pause ongoing tasks, disable timers, and throttle down OpenGL ES frame rates. Games should use this method to pause the game.
    }

    func applicationDidEnterBackground(application: UIApplication) {
        // Use this method to release shared resources, save user data, invalidate timers, and store enough application state information to restore your application to its current state in case it is terminated later.
        // If your application supports background execution, this method is called instead of applicationWillTerminate: when the user quits.
    }

    func applicationWillEnterForeground(application: UIApplication) {
        // Called as part of the transition from the background to the inactive state; here you can undo many of the changes made on entering the background.
    }

    func applicationDidBecomeActive(application: UIApplication) {
        // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
    }

    func applicationWillTerminate(application: UIApplication) {
        // Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.
    }


}


```
*YMTabBarController.swift:*

```swift
import UIKit

class YMTabBarController: UITabBarController {

    override func viewDidLoad() {
        super.viewDidLoad()
        tabBar.tintColor = UIColor(red: 245 / 255, green: 80 / 255, blue: 83 / 255, alpha: 1.0)
        
        addChildViewControllers()

        // Do any additional setup after loading the view.
    }
    
    private func addChildViewControllers(){
        addChildViewController("YMDanTangViewController", title: "单糖", imageName: "TabBar_home_23x23_")
//        addChildViewController("YMProductViewController", title: "单品", imageName: "TabBar_gift_23x23_")
//        addChildViewController("YMCategoryViewController", title: "分类", imageName: "TabBar_category_23x23_")
        addChildViewController("YMMeViewController", title: "我", imageName: "TabBar_me_boy_23x23_")

    }
    
    
    /**
     # 初始化子控制器
     
     - parameter childControllerName: 需要初始化的控制器
     - parameter title:               标题
     - parameter imageName:           图片名称
     */
    private func addChildViewController(childControllerName: String, title: String, imageName: String) {
        // 动态获取命名空间
        let ns = NSBundle.mainBundle().infoDictionary!["CFBundleExecutable"] as! String
        // 将字符串转化为类，默认情况下命名空间就是项目名称，但是命名空间可以修改
        let cls: AnyClass? = NSClassFromString(ns + "." + childControllerName)
        let vcClass = cls as! UIViewController.Type
        let vc = vcClass.init()
        // 设置对应的数据
        vc.tabBarItem.image = UIImage(named: imageName)
        vc.tabBarItem.selectedImage = UIImage(named: imageName + "selected")
        vc.title = title
        // 给每个控制器包装一个导航控制器
        let nav = YMNavigationController()
        nav.addChildViewController(vc)
        addChildViewController(nav)
    }

}
```
这里面有几个知识点：
###如何通过字符串创建类对象
    1. 在swift中打印对象时，会发现在类型前面总会有命名空间 .+类名
    2. 在swift中用字符串生成类对象就需要拼接成这样的格式，才能成功生成类
    3. 注意，命名空间不要加特殊符号，不然依然无法获取控制器类
![](http://upload-images.jianshu.io/upload_images/1370647-d321f25de36b5d2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/1370647-c7a027382a6a2048.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


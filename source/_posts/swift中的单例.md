title: swift中的单例
date: 2016-09-09 17:51:27
tags:
- swift
- IOS
categories: Swift
---
##关于单例
. 单例必须是唯一的，所以它才被称为单例。在一个应用程序的生命周期里，有且只有一个实例存在。单例的存在给我们提供了一个唯一的全局状态。比如我们熟悉的NSNotification，UIApplication和NSUserDefaults都是单例。
. 为了保持一个单例的唯一性，单例的构造器必须是私有的。这防止其他对象也能创建出单例类的实例。感谢所有帮我指出这点的人
. 为了确保单例在应用程序的整个生命周期是唯一的，它就必须是线程安全的。当你一想到并发肯定一阵恶心，简单来说，如果你写单例的方式是错误的，就有可能会有两个线程尝试在同一时间初始化同一个单例，这样你就有潜在的风险得到两个不同的单例。这就意味着我们需要用GCD的dispatch_once来确保初始化单例的代码在运行时只执行一次。
## swift 中单例写法

```
class TheOneAndOnlyKraken {
    static let sharedInstance = TheOneAndOnlyKraken()
}
```

## 项目中的应用

```
class YMNetworkTool: NSObject {
    /// 单例
    static let shareNetworkTool = YMNetworkTool()
    
    /// 获取首页数据
    func loadHomeInfo(id: Int, finished:(homeItems: [YMHomeItem]) -> ()) {
//        SVProgressHUD.showWithStatus("正在加载...")
        let url = BASE_URL + "v1/channels/" + String(id) + "/items?gender=1&generation=1&limit=20&offset=0"
        Alamofire
            .request(.GET, url)
            .responseJSON { (response) in
                guard response.result.isSuccess else {
//                    SVProgressHUD.showErrorWithStatus("加载失败...")
                    return
                }
                if let value = response.result.value {
                    let dict = JSON(value)
                    let code = dict["code"].intValue
                    let message = dict["message"].stringValue
                    guard code == RETURN_OK else {
//                        SVProgressHUD.showInfoWithStatus(message)
                        return
                    }
                    // 移除加载提示
//                    SVProgressHUD.dismiss()
                    let data = dict["data"].dictionary
                    //  字典转成模型
                    if let items = data!["items"]?.arrayObject {
                        var homeItems = [YMHomeItem]()
                        for item in items {
                            let homeItem = YMHomeItem(dict: item as! [String: AnyObject])
                            homeItems.append(homeItem)
                        }
                        finished(homeItems: homeItems)
                    }
                }
        }
    }
    
    /// 获取首页顶部选择数据
    func loadHomeTopData(finished:(ym_channels: [YMChannel]) -> ()) {
        
        let url = BASE_URL + "v2/channels/preset?gender=1&generation=1"
        Alamofire
            .request(.GET, url)
            .responseJSON { (response) in
                guard response.result.isSuccess else {
//                    SVProgressHUD.showErrorWithStatus("加载失败...")
                    return
                }
                if let value = response.result.value {
                    let dict = JSON(value)
                    let code = dict["code"].intValue
                    let message = dict["message"].stringValue
                    guard code == RETURN_OK else {
//                        SVProgressHUD.showInfoWithStatus(message)
                        return
                    }
//                    SVProgressHUD.dismiss()
                    let data = dict["data"].dictionary
                    if let channels = data!["channels"]?.arrayObject {
                        var ym_channels = [YMChannel]()
                        for channel in channels {
                            let ym_channel = YMChannel(dict: channel as! [String: AnyObject])
                            ym_channels.append(ym_channel)
                        }
                        finished(ym_channels: ym_channels)
                    }
                }
        }
        
    }
    
}
```


title: 'swift实现代理,通知,闭包传值'
date: 2016-12-20 09:58:21
tags:
categories: Swift
---
## 区别
第一就是代理，这也是很常用的方式，特点是一对一的形式，而且逻辑结构非常清晰。实现起来较为简单：写协议 ，设置代理这个属性， 最好在你想通知代理做事情的方法中调用即可。当然这里面有一些细节，包括 ①协议定义时，请用关键字@required，和@optional来明确代理是否必须实现某些方法 ②代理的类型需用id类型，并写明要遵守的协议 ③就是在调用代理方法的时候需要判断代理是否实现该方法。

第二就是通知，通知的优点很明显，他是一对多的形式，而且可以在任意对象之间传递，不需要二者有联系，当然他的实现和代理相比较稍微绕一点，注册，发通知，收通知。这里面的注意点就是 ①对于系统没有定义的事件监听时需要自己发通知，这是你就需要定义一个key，字符串类型，这也是通知的一个弊端，你需要拷贝到收通知的对象，避免写错一个字母而无法收通知的尴尬 ②就是注册的通知中心需要手动移除，不然除了性能的问题还会有其他的问题出现，比如说一个控制器消失了之后还有因为某些事件而发出通知，造成不想要的结果。

第三就是block了，这是苹果后来才加入的，也是目前开发比较常用的一种方式，功能比较强大，但是在理解和使用上可能需要一段时间摸索和熟悉。他的最大特点就是回调，而且回调时可以传入参数，最重要的是，无论在哪调用，block的执行都会回到block创建的地方执行，而非调用的地方。而block本身可以封装一段代码，一段代码你懂的，很多人在初学时会被搞晕，甚至在block的声明上就纠结，其实很正常，多用就好。

## 代理
如下面代码,我们定义了一个协议方法, 其结构就是protocol关键字+协议的名称 + : + NSObjectProtocol {},注意Swift中的自定义协议其本身必须遵循NSObjectProtocol协议

```
protocol TextViewControllerDelegate: NSObjectProtocol{
    // 点击delegate按钮回调
    func delegateBtnWillClick(message : String)
}


```
并且在OC中,我们一般会定义一个属性来保存代理如@property (nonatomic,weak)id< TextViewControllerDelegate > delegate;但是在Swift中我们依然要这样做,只不过写法变成了weak var delegate : TextViewControllerDelegate?,注意一定要加上weak关键字,避免循环引用问题.这是我的demo中的代码,回调一段字符串

```
/MARK: - 点击代理传值按钮
func delegateBtnClick()
{
    print("代理传值按钮被点击")
    let str = "代理传值按钮被点击,把上个界面的值传了过来"
    delegate?.delegateBtnWillClick(str)
    self.navigationController?.popViewControllerAnimated(true)
}

```

然后就在别的控制器,设置那个控制器为该控制器的代理,去实现协议方法就可以了
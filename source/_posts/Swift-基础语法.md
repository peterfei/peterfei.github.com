title: Swift 基础语法
date: 2016-08-20 12:43:37
tags:
---

##声明常量和变量
`let` 声明的是常量 `var` 声明变量
e.g. 
```
let maximumNumberOfLoginAttempts = 10
var currentLoginAttempt = 0
```

多个变量`,`分隔
`var x = 0.0, y = 0.0, z = 0.0`

##类型标注
`var welcomeMessage: String` 
>注: `:`后有空格

welcomeMessage变量现在可以被设置成任意字符串：`welcomeMessage = "Hello"`

>常量一经声明就不允许改值，否则编译器报错

>同ruby 语法类似 ，swift 在字符串中可这样输出变量||常量
将常量或变量名放入圆括号中，并在开括号前使用反斜杠将其转义

```
 print("The current value of friendlyWelcome is \(friendlyWelcome)")
```

##属性观察器
属性观察器监控和响应属性值的变化，每次属性被设置值的时候都会调用属性观察器，甚至新的值和现在的值相同的时候也不例外
可以为属性添加如下的一个或全部观察器：
1. willSet在新的值被设置之前调用
2. didSet在新的值被设置之后立即调用

```
class StepCounter {
    var totalSteps: Int = 0 {
        willSet(newTotalSteps) {
            print("About to set totalSteps to \(newTotalSteps)")
        }
        didSet {
            if totalSteps > oldValue  {
                print("Added \(totalSteps - oldValue) steps")
            }
        }
    }
}
let stepCounter = StepCounter()
stepCounter.totalSteps = 200
// About to set totalSteps to 200
// Added 200 steps
stepCounter.totalSteps = 360
// About to set totalSteps to 360
// Added 160 steps
stepCounter.totalSteps = 896
// About to set totalSteps to 896
// Added 536 steps
```

##泛型函数

```
func swapTwoValues<T>(inout a: T, inout _ b: T) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```


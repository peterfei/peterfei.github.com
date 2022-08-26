title: swift中闭包使用
date: 2016-09-04 23:02:24
tags:
- swift
categories: Swift
---
    闭包是自包含的函数代码块,可以在代码中被传递和使用。
    闭包可以捕获和存储其所在上下文中任意常量和变量的引用。这就是所谓的闭合并包裹着这些常量和变量,俗称 闭包。Swift 会为您管理在捕获过程中涉及到的所有内存操作。
Swift 的闭包表达式拥有简洁的风格,并鼓励在常见场景中进行语法优化,主要优化如下:
1. 利用上下文推断参数和返回值类型
2. 隐式返回单表达式闭包,即单表达式闭包可以省略 return 关键字
3. 参数名称缩写
4. 尾随(Trailing)闭包语法

##闭包表达式语法(Closure Expression Syntax)
闭包表达式语法有如下一般形式:
```
{ (parameters) -> returnType in
    statements
}
```

demo:
```
@IBAction func operate(sender: UIButton) {
        let operate = sender.currentTitle!
        print("current operate is \(operate)")
        switch operate {
        case "+": performOperation({$0+$1})
            
        case "−": performOperation({$1-$0})
        case "×": performOperation({$0*$1})
        case "÷": performOperation({$1/$0})
        default:
            break
        }
    }
    
    func performOperation(operation:(Double, Double) -> Double) {
        if operandStack.count>=2 {
                displayValue = operation(operandStack.removeLast(),operandStack.removeLast())
                print("displayValue is \(displayValue)")
                enter()
            }
    }
```

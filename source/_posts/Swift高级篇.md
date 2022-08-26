title: Swift高级篇
date: 2016-12-20 13:55:50
tags:
	- swift
categories: Swift
---
## 扩展(Extension)
任务： 求数字的平方。

```
// 菜鸟版
func square(x: Int) -> Int { return x * x }
var squaredOfFive = square(x: 5)
square(x: squaredOfFive) // 625

```

为了求5的四次方我们被迫创建变量 squaredOfFive — 高手可不喜欢被迫定义一个无用的变量。

```
// 高手版
extension Int { 
 var squared: Int { return self * self }
}
5.squared // 25
5.squared.squared // 625
```

## 泛型(Generics)
任务：打印输出数组内所有的元素。
```
// 菜鸟版
var stringArray = ["苍老师", "范老师", "优衣库"]
var intArray = [1, 3, 4, 5, 6]
var doubleArray = [1.0, 2.0, 3.0]
func printStringArray(a: [String]) { 
        for s in a { 
              print(s) 
        }
}
func printIntArray(a: [Int]) { for i in a { print(i) } }
func printDoubleArray(a: [Double]) {for d in a { print(d) } }
```
居然要定义这么多函数？ 菜鸟能忍高手不能忍！

```
// 高手版
func printElementFromArray(a: [T]) {
        for element in a { 
            print(element) 
        } 
}
```

##  For 遍历 vs While 遍历
任务：打印 5 次 大雁塔

```
// 菜鸟版
var i = 0
while 5 > i {
      print("大雁塔")
      i += 1 
}
```
被迫定义了变量 i 来确保打印大雁塔5次

```
// 高手版
for _ in 1...5 { 
    print("大雁塔") 
}
```
## 计算属性 vs 函数
任务：计算圆的直径
```
// 菜鸟版
funcgetDiameter(radius: Double) -> Double { return radius * 2}
funcgetRadius(diameter: Double) -> Double { return diameter / 2}
getDiameter(radius: 10) // return 20
getRadius(diameter: 200) // return 100
getRadius(diameter: 600) // return 300
```

上面我们创建了2个毫无关系的函数，可是直径和周长两者真的没有关系吗？
```
// 高手版
var radius: Double = 10
var diameter: Double {
      get { return radius * 2}
      set { radius = newValue / 2} 
}
radius // 10
diameter // 20
diameter = 1000
radius // 500
```
## 函数式编程
任务： 获取偶数。

```
// 菜鸟版
var newEvens = [Int]()
for i in 1...10 {
  if i % 2 == 0 { 
      newEvens.append(i) 
    } 
}
print(newEvens) // [2, 4, 6, 8, 10]
```
这种for循环真是冗长，让人看的昏昏欲睡。
```
// 高手版
var evens = (1...10).filter { $0 % 2 == 0 } 
print(evens) 
// [2, 4, 6, 8, 10]

```

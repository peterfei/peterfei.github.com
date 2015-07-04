---
layout: post
title: "建立类方法"
date: 2013-06-14 19:15
comments: true
tags: 
- 类方法
- ruby 
- ror
categories: 
---
```ruby 
First way:
class MyClass
    class << self
        def my_method; end
    end
end

Second Way:
class MyClass
    def self.my_method; end
end

Third Way:
def MyClass.my_other_method; end
```

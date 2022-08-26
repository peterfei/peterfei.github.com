title: Sublime 中自定义snippets
date: 2015-10-29 10:00:20
tags:
categories: 工具
---
##选择`tool`->`new snippet`
内容如下：
```
<snippet>
    <content><![CDATA[
Hello, ${1:this} is a ${2:snippet}.
]]></content>
    <!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
    <!-- <tabTrigger>hello</tabTrigger> -->
    <!-- Optional: Set a scope to limit where the snippet will trigger -->
    <!-- <scope>source.python</scope> -->
</snippet>
```
1. Hello, ${1:this} is a ${2:snippet} 这正是你要插入到文档中的文本。在此节放入的任何片段都会被插入到你的文档中。
2. tabTrigger 此节是可选的配置，默认是被注释掉的。默认 hello 的意思是：如果你在某个文档里输入了单词 hello ，然后按下 Tab 键，接着 hello 就会被替换为1中定义的代码片段。再次按下Tab键，接着snippet会被替换为2中定义的代码片段
3. scope 此节也是可选配置，默认是被注释掉的。默认 source.php 的意思是：只有在编辑 php 源码的时候，才能用此代码片段

##保存代码片段文件
在能自定义代码片段之前，应该首先保存。在保存文件对话框里，需要明确的指定文件的扩展名为.sublime-snippet，然后把文件保存到默认目录（当前用户主目录下的\Sublime Text 3\Packages\User目录中）。

保存文件之后，就可以测试上面提到的tabTrigger功能了。如果不再使用此功能，可以在此注释掉.

##修改代码片段文件
在创建新代码片段中提到过 Hello, ${1:this} is a ${2:snippet} ，其中 ${1:this} 和 ${2:snippet} 是占位符。在插入代码片段后，单词 this 被选中，如果键入内容，this 将会被替换掉，接着按下 Tab 键，将会选中单词 snippet，如果键入内容，snippet 将会被替换掉。

##绑定快捷键

可以将上述的操作绑定到一个快捷键，在不键入任何文本的情况下，直接按快捷键插入代码片段。

点击菜单栏的 Preferences 的子菜单 Key Binding – User，在打开的文件的方括号内部粘贴如下配置：

```
	{ “keys”: [“ctrl+1″], “command”: “insert_snippet”, “args”: {“name”: “Packages/User/example.sublime-snippet”} }
```

现在简单介绍一下这段配置：

1.   “keys”: [“ctrl+1″] 这个定义了触发此命令的快捷键。

2.   “command”: “insert_snippet” 这个是需要触发的命令的名字。

3.   “args”: {“name”: “Packages/User/example.sublime-snippet”} 这个是需要传入到上述命令的参数。这里把代码片段文件的相对路径传递过去。

保存配置文件，现在就可以用快捷键插入代码片段了。
>以下是我Ruby的注释代码：

```
<snippet>
	<content><![CDATA[
# ########################################################
# | Software: ${1}
# | Version: 2015.10
# | Site: http://peterfei.me
# |--------------------------------------------------------
# | Author: peterfei <peterfeispace@gmail.com>
# | Copyright (c) 2012-2015, http://peterfei.me. All Rights
#Reserved.
# | Time:  ${2} 
# ########################################################
${3}
]]></content>
	<!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
	<!-- <tabTrigger>hello</tabTrigger> -->
	<!-- Optional: Set a scope to limit where the snippet will trigger -->
	<!-- <scope>source.python</scope> -->
</snippet>
```

因为上例中有写Time 获取当前时间的，于是写个获取当前时间的：

> 创建插件：`Tools → New Plugin:`
```
import datetime
import sublime_plugin
class AddCurrentTimeCommand(sublime_plugin.TextCommand):
    def run(self, edit):
        self.view.run_command("insert_snippet", 
            {
                "contents": "%s" % datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S") 
            }
        )
```
保存为Sublime Text 2\Packages\User\addCurrentTime.py

> 创建快捷键：`Preference → Key Bindings - User:`
```
	[
	    {
	        "command": "add_current_time",
	        "keys": [
	            "ctrl+shift+."
	        ]
	    }
	]
```
> 此时使用快捷键ctrl+shift+.即可在当前光标处插入当前时间，如下:

```
# ########################################################
# | Software: test
# | Version: 2015.10
# | Site: http://peterfei.me
# |--------------------------------------------------------
# | Author: peterfei <peterfeispace@gmail.com>
# | Copyright (c) 2012-2015, http://peterfei.me. All Rights
#Reserved.
# | Time:  2015-10-29 09:58:53 
# ########################################################
```
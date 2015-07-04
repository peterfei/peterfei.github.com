---
layout: post
title: "ROR layout+content_for 配合用法"
date: 2014-09-18 14:29:09 +0800
comments: true
tags:
- content_for
categories: 
---
*content_for 一般要这样用才好
在view先找个地方声明*
```
	<% content_for :head do %>
 	 <%= stylesheet_link_tag 'projects' %>
	<% end %>
```
之后

```
	<head>
  	<title>Todo List</title>
  	<%= stylesheet_link_tag 'application' %>
  	<%= yield :head %>
	</head>
```
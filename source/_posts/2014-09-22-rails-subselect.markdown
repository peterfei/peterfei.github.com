---
layout: post
title: "Rails中的子查询"
date: 2014-09-22 14:16:16 +0800
comments: true
tags: ruby
categories: 
---
###大致是这样子的
```ruby 
    def favarate_post options={}
        subselect = Favarate.send(
                :construct_finder_sql,
                    :select =>"post_id",
                    :conditions=>{:blog_id=>self.id},
                    :order=>"published_at desc",
                    :limit => option[:limit]||10,:offset=>option[:offset]
            )
        Post.find(:all,:conditions =>'posts.id in #{subselect}',:order =>"published_at DESC")
    end
```    
construct_finder_sql 同下句SQL类似：
```sql
    SELECT * FROM posts WHERE posts.id IN (
    SELECT post_id FROM favorites WHERE blog_id = 42 ORDER BY published_at DESC LIMIT 10 OFFSET 10
  ) ORDER BY published_at DESC
```

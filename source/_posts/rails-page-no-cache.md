title: rails page no cache
date: 2016-08-16 15:21:21
tags:
- rails
- cache
---
Rails 页面在完成权限时总是出现加载缓存页，导致加载出admin权限json渲染页。查看了浏览器请求，是页面cache引起，以下是解决方案:
```
before_filter :set_cache_headers

def set_cache_headers
  response.headers["Cache-Control"] = "no-cache, no-store"
  response.headers["Pragma"] = "no-cache"
  response.headers["Expires"] = "Fri, 01 Jan 2016 00:00:00 GMT"
end
```

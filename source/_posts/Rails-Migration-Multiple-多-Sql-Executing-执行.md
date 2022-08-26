title: 'Rails Migration Multiple(多) Sql Executing(执行) '
date: 2015-11-16 13:46:17
tags:
categories: ROR
---
>在migration 里生成的文件写下如下要批量执行的sql:
```sql
	execute <<-SQL
     ALTER TABLE distributors ADD CONSTRAINT zipchk CHECK (char_length(zipcode) = 5) NO INHERIT;
     ALTER TABLE zipchk ADD COLUMN board_time int(10) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '时间';      
    SQL
      
```
>执行一直报sql错误，下面是Google Group 社区给出的解决方案：
```
sql = <<-SQL 
 <statement_1>; 
 <statement_2>; 
 <statement_3>; 
SQL 

sql.strip.split(';').each do |s| 
  ActiveRecord::Base.connection.execute(s) 
end 
```
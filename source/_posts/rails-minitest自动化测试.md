title: rails minitest自动化测试
date: 2015-11-12 09:32:09
tags:
categories: Ruby
---
##Gemfile加入相关测试驱动：
> ```ruby
    #测试组
	group :test do
	    gem "minitest"
	    gem 'minitest-spec-rails'
	    gem 'minitest-reporters'
	    gem 'spring'
	    gem 'guard'
	    gem 'guard-minitest'
	    # 添加调用跟踪静默程序
	    # gem 'mini_backtrace'
	    gem 'ruby-prof'
	    gem 'minitest-rails-capybara'
	    gem 'autotest-standalone' # The file '.autotest' makes sure the tests are run via test server (spork).
	    gem 'autotest-rails-pure' # -pure gives us autotest without ZenTest gem.
	    gem 'autotest-growl'      # growl notifications, complains a little bit if growl isn't installed
	    gem 'autotest-fsevent'    # react to filesystem events, save your CPU
	    gem 'spork-minitest'
	end
```

##初始化guard:
>`guard init minitest`

##配置guard:
	```
		# A sample Guardfile
		# More info at https://github.com/guard/guard#readme
		
		## Uncomment and set this to only include directories you want to watch
		# directories %w(app lib config test spec features) \
		#  .select{|d| Dir.exists?(d) ? d : UI.warning("Directory #{d} does not exist")}
		
		## Note: if you are using the `directories` clause above and you are not
		## watching the project directory ('.'), then you will want to move
		## the Guardfile to a watched dir and symlink it back, e.g.
		#
		#  $ mkdir config
		#  $ mv Guardfile config/
		#  $ ln -s config/Guardfile .
		#
		# and, you'll have to watch "config/Guardfile" instead of "Guardfile"
		
		guard :minitest, spring: true, all_on_start: false do
		  # with Minitest::Unit
		  watch(%r{^test/(.*)\/?test_(.*)\.rb$})
		  watch(%r{^lib/(.*/)?([^/]+)\.rb$})     { |m| "test/#{m[1]}test_#{m[2]}.rb" }
		  watch(%r{^test/test_helper\.rb$})      { 'test' }
		
		  # with Minitest::Spec
		  # watch(%r{^spec/(.*)_spec\.rb$})
		  # watch(%r{^lib/(.+)\.rb$})         { |m| "spec/#{m[1]}_spec.rb" }
		  # watch(%r{^spec/spec_helper\.rb$}) { 'spec' }
		
		  # Rails 4
		  watch(%r{^app/(.+)\.rb$})                               { |m| "test/#{m[1]}_test.rb" }
		  watch(%r{^app/controllers/application_controller\.rb$}) { 'test/controllers' }
		  watch(%r{^app/controllers/(.+)_controller\.rb$})        { |m| "test/integration/#{m[1]}_test.rb" }
		  watch(%r{^app/views/(.+)_mailer/.+})                   { |m| "test/mailers/#{m[1]}_mailer_test.rb" }
		  watch(%r{^lib/(.+)\.rb$})                               { |m| "test/lib/#{m[1]}_test.rb" }
		  watch(%r{^test/.+_test\.rb$})
		  watch(%r{^test/test_helper\.rb$}) { 'test' }
		
		  # Rails < 4
		  # watch(%r{^app/controllers/(.*)\.rb$}) { |m| "test/functional/#{m[1]}_test.rb" }
		  # watch(%r{^app/helpers/(.*)\.rb$})     { |m| "test/helpers/#{m[1]}_test.rb" }
		  # watch(%r{^app/models/(.*)\.rb$})      { |m| "test/unit/#{m[1]}_test.rb" }
		end

	```

##运行Guard:
>输入 `guard` 命令运行测试

##例外
>如果database不用默认的数据库迁移，测试时可以在db下放入`structure.sql`, 该配置为数据库结构，然后更改配置文件 `application.rb`:
>``` 
	 config.active_record.schema_format = :sql
```
运行：`rake db:test:prepare`

##生成集成化测试文件 
>`rails g test_unit:integration testName`
##生成model测试文件 
>`rails g test_unit:model`
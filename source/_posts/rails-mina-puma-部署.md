title: rails mina+puma 部署
date: 2015-10-19 14:26:47
tags:
---
出于安全考虑，不要使用 root 帐号运行 web 应用。这里新建一个专门用于部署的用户，例如 deploy 或者其它你喜欢的名字。运行以下命令创建用户：

`# useradd -m -s /bin/bash deploy`

将用户加入 sudo 群组，以便使用 sudo 命令：
`# adduser deploy sudo`
为 deploy 用户设置密码：
`# passwd deploy`
给Linux deploy用户 加入rvm 权限
`usermod -a -G rvm deploy`
退出当前 SSH 链接，用 deploy 帐号重新登陆。
##加入`gem mina`
`mina init`, 生成config/deploy.rb
```ruby 
	require 'mina/rails'
	require 'mina/git'
	require 'mina/bundler'
	require 'mina/rvm'
	#服务器地址,是使用ssh的方式登录服务器
	set :domain, 'deploy@yourIPServer'
	#服务器中项目部署位置
	set :deploy_to, '/home/deploy/api'
	#git代码仓库
	set :repository, 'git@XXX.git'
	#git分支
	set :branch, 'master'
	set :rvm_path, '/usr/local/rvm/bin/rvm'
	set :bundle_gemfile, "app/Gemfile"
	# 中括号里的文件 会出现在服务器项目附录的shared文件夹中，这里加入了secrets.yml，环境密钥无需跟开发计算机一样
	set :shared_paths, ['config/database.yml', 'log', 'config/secrets.yml']
	set :rails_env, 'development'
	task :environment do
	  # 如果使用的是rbenv,这么设置,但需确保.rbenv-version(rbenv local 1.9.3-p374)已经存在于你的项目中
	  invoke :'rvm:use[ruby-2.1.2@default]'
	  # 如果使用rvm，可以这样加载一个RVM version@gemset
	  # invoke :'rvm:use[ruby-1.9.3-p374@default]'
	end
	# 这个块里面的代码表示运行 mina setup时运行的命令
	task :setup => :environment do
	  # 在服务器项目目录的shared中创建log文件夹
	  queue! %[mkdir -p "#{deploy_to}/#{shared_path}/log"]
	  queue! %[chmod g+rx,u+rwx "#{deploy_to}/#{shared_path}/log"]
	  # 在服务器项目目录的shared中创建config文件夹 下同
	  queue! %[mkdir -p "#{deploy_to}/#{shared_path}/config"]
	  queue! %[chmod g+rx,u+rwx "#{deploy_to}/#{shared_path}/config"]
	
	  queue! %[touch "#{deploy_to}/#{shared_path}/config/database.yml"]
	  queue! %[touch "#{deploy_to}/#{shared_path}/config/secrets.yml"]
	
	  # puma.rb 配置puma必须得文件夹及文件
	  queue! %[mkdir -p "#{deploy_to}/shared/tmp/pids"]
	  queue! %[chmod g+rx,u+rwx "#{deploy_to}/shared/tmp/pids"]
	
	  queue! %[mkdir -p "#{deploy_to}/shared/tmp/sockets"]
	  queue! %[chmod g+rx,u+rwx "#{deploy_to}/shared/tmp/sockets"]
	
	  queue! %[touch "#{deploy_to}/shared/config/puma.rb"]
	  queue  %[echo "-----> Be sure to edit 'shared/config/puma.rb'."]
	
	  # tmp/sockets/puma.state
	  queue! %[touch "#{deploy_to}/shared/tmp/sockets/puma.state"]
	  queue  %[echo "-----> Be sure to edit 'shared/tmp/sockets/puma.state'."]
	
	  # log/puma.stdout.log
	  queue! %[touch "#{deploy_to}/shared/log/puma.stdout.log"]
	  queue  %[echo "-----> Be sure to edit 'shared/log/puma.stdout.log'."]
	
	  # log/puma.stdout.log
	  queue! %[touch "#{deploy_to}/shared/log/puma.stderr.log"]
	  queue  %[echo "-----> Be sure to edit 'shared/log/puma.stderr.log'."]
	
	  queue  %[echo "-----> Be sure to edit '#{deploy_to}/#{shared_path}/config/database.yml'."]
	end
	
	#这个代码块表示运行 mina deploy时执行的命令
	desc "Deploys the current version to the server."
	task :deploy => :environment do
	  to :before_hook do
	  end
	  deploy do
	    #重新拉git服务器上的最新版本，即使没有改变
	    invoke :'git:clone'
	
	    #重新设定shared_path位置
	    invoke :'deploy:link_shared_paths'
	    invoke :'bundle:install'
	    # invoke :'rails:db_migrate'
	    invoke :'rails:assets_precompile'
	    invoke :'deploy:cleanup'
	
	    to :launch do
	      queue "mkdir -p #{deploy_to}/#{current_path}/tmp/"
	      # queue "chown -R www-data #{deploy_to}"
	      queue "touch #{deploy_to}/#{current_path}/tmp/restart.txt"
	    end
	  end
	end
```
这样一来mina的基本配置就完成，接下来只要将你开发环境的项目上传到git服务器，然后运行下面的命令就完成了

`mina deploy`

##puma配置
```
	#!/usr/bin/env puma
	
	#rails的运行环境
	# environment 'production'
	environment 'development'
	threads 2, 64
	workers 4
	
	#项目名
	app_name = "api"
	#项目路径
	application_path = "/home/deploy/#{app_name}"
	#这里一定要配置为项目路径下地current
	directory "#{application_path}/current"
	
	#下面都是 puma的配置项
	pidfile "#{application_path}/shared/tmp/pids/puma.pid"
	state_path "#{application_path}/shared/tmp/sockets/puma.state"
	stdout_redirect "#{application_path}/shared/log/puma.stdout.log", "#{application_path}/shared/log/puma.stderr.log"
	bind "unix://#{application_path}/shared/tmp/sockets/#{app_name}.sock"
	activate_control_app "unix://#{application_path}/shared/tmp/sockets/pumactl.sock"
	
	#后台运行
	daemonize true
	on_restart do
	  puts 'On restart...'
	end
	preload_app!
	

```

##启动puma
进入/yourprj/config
`pumactl -F puma.rb start`

##Nginx 配置
```
	  worker_processes  1;
	
	      error_log  /var/log/nginx/error.log warn;
	      pid        /var/run/nginx.pid;
	
	      events {
	          worker_connections  1024;
	      }
	
	      http {
	          include       /etc/nginx/mime.types;
	          default_type  application/octet-stream;
	
	          log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
	                            '$status $body_bytes_sent "$http_referer" '
	                            '"$http_user_agent" "$http_x_forwarded_for"';
	
	          access_log  /var/log/nginx/access.log  main;
	
	          sendfile        on;
	          #tcp_nopush     on;
	
	          keepalive_timeout  65;
	
	          #gzip  on;
	
	          #include /etc/nginx/conf.d/*.conf;
	          upstream deploy {
	                  server unix:///var/www/ruby_sample/shared/tmp/sockets/ruby_sample.sock;
	          }
	
	          server {
	              listen 80;
	              server_name your.server.domain.ip; # change to match your URL
	              root /var/www/ruby_sample/current/public; # I assume your app is located at this location
	
	              location / {
	                  proxy_pass http://deploy; # match the name of upstream directive which is defined above
	                  proxy_set_header Host $host;
	                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	              }
	
	              location ~* ^/assets/ {
	                  # Per RFC2616 - 1 year maximum expiry
	                  expires 1y;
	                  add_header Cache-Control public;
	                          # Some browsers still send conditional-GET requests if there's a
	                  # Last-Modified header or an ETag header even if they haven't
	                  # reached the expiry date sent in the Expires header.
	                  add_header Last-Modified "";
	                  add_header ETag "";
	                  break;
	              }
	       }
	      }
	

```

接下里只需要重启nginx服务器，整个rails的环境就搭建完成了
`nginx -s reload`
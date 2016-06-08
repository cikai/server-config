##环境配置

###1.Ruby

#####1.1 安装RVM

	curl -L https://get.rvm.io | bash

安装完成后重新login，使RVM生效。

	/bin/bash -login

测试RVM是否安装成功，查看RVM信息。

	rvm info

#####1.2 修改RVM下载源为Ruby China的镜像

	echo "ruby_url=https://cache.ruby-china.org/pub/ruby" > ~/.rvm/user/db
	或者
	sed -i 's!ftp.ruby-lang.org/pub/ruby!cache.ruby-china.org/pub/ruby/!' $rvm_path/config/db

#####1.3 用RVM安装Ruby环境

	rvm requirements
	rvm install 2.3.0

安装完成后，Ruby, Ruby Gems 就安装好了。

#####1.4 设置系统默认的Ruby版本

	rvm use 2.3.0 --default

#####1.5 修改gem下载源为Ruby China的镜像

	gem sources --remove https://rubygems.org/
	gem sources -a https://gems.ruby-china.org/
	gem sources -l [请确保只有Ruby China的源地址]

#####1.6 更新gem版本

	gem update --system 

###2.Rails

	gem install rails -v=5.0.0.rc1

版本号根据自己的需求填写，不写版本会默认更新为当前稳定版。

测试Rails是否安装成功，查看Rails信息。

	rails -v

###3.Nodejs
	sudo apt-get install nodejs


###4.Bundler

#####4.1 安装

	gem install bundler

#####4.2 修改bundler下载源为Ruby China的镜像

	bundle config mirror.https://rubygems.org https://gems.ruby-china.org

###5.PostgreSQL

#####5.1 安装

	sudo apt-get install postgresql

#####5.2 启动与停止

	sudo /etc/init.d/postgresql start  # 启动
	sudo /etc/init.d/postgresql stop   # 停止

#####5.3 设置postgres用户密码

以postgres这个系统用户的身份运行psql命令

	sudo su postgres -c psql template1

设置密码

	ALTER USER postgres WITH PASSWORD 'root'; # 请替换密码root

#####5.4 配置PostgreSQL远程访问

允许远程访问，需要修改两个配置文件。

	文件位置：
	/etc/postgresql/9.3/main

1）postgresql.conf

找到

	listen_addresses = 'localhost'

修改为

	listen_addresses = '*'

2）pg_hba.conf

在IPv4 local connections位置追加以下配置。

	host all all 0.0.0.0/0 md5

如果不希望允许所有IP访问，可以将0.0.0.0设置为特定的IP地址。

#####5.5 管理工具pgAdmin III

[https://www.pgadmin.org/](https://www.pgadmin.org/)

###6.Nginx

#####6.1 安装

	sudo apt-get install nginx

#####6.2 nginx配置

编辑nginx配置文件，位置： /etc/nginx/nginx.conf

	worker_processes  4;

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
			server 127.0.0.1:8080;
		}

		server {
			listen 80;
			server_name your_server_ip; # change to match your URL
			root /var/www/your_project; # I assume your app is located at this location

			location / {
				# match the name of upstream directive which is defined above
				proxy_pass http://127.0.0.1:8080;
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
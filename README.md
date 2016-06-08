##服务器环境配置文档

###1.Ruby

#####1.1 安装RVM

	curl -L https://get.rvm.io | bash

安装完成后重新login，使RVM生效。

	/bin/bash -login

测试RVM是否安装成功，查看RVM信息。

	rvm info

#####1.2 修改RVM下载源为Ruby China的镜像

	echo "ruby_url=https://cache.ruby-china.org/pub/ruby" > ~/.rvm/user/db

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

###2.Rails

	gem install rails –v5.0.0.rc1

版本号根据自己的需求填写，不写版本会默认更新为当前稳定版。

测试Rails是否安装成功，查看Rails信息。

	rails -v

###3.Bundler

	gem install bundler

###4.PostgreSQL

#####4.1 安装

	sudo apt-get install postgresql

#####4.2 启动与停止

	sudo /etc/init.d/postgresql start  # 启动
	sudo /etc/init.d/postgresql stop   # 停止

#####4.3 设置postgres用户密码

以postgres这个系统用户的身份运行psql命令

	sudo su postgres -c psql template1

设置密码

	ALTER USER postgres WITH PASSWORD 'root'; # 请替换密码root

#####4.4 配置PostgreSQL远程访问

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


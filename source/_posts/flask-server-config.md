---
title: Flask项目部署的常规操作
date: 2023-09-19 17:39:24
tags:
    - Flask
categories:
    - Python
    - Flask
description: Flask项目部署的一些配置
---

## 离线安装python3.9.8与相关依赖

- 解压源码包到指定目录

	```sh
	tar -xzvf /target_path/Python-3.9.8.tar.gz -C /target_path/
	```
	
- 进入解压目录并配置

	```sh
	cd /target_path/Python-3.9.8
	./configure --prefix=/target_path/python3
	```

- 在python解压目录中执行编译并安装

	```sh
	make
	sudo make install
	```

- 更改python3文件所有者与所属群组

	```sh
	cd ~
	sudo chown -R user:user /target_path/python3
	```

- 安装项目依赖模块
	
	+ 解压对应架构依赖压缩包到指定目录

		```sh
		unzip -d /unzip_path/whl python3-xxx.zip
		```
	
	+ 创建项目隔离环境

		```sh
		cd /target_path/python3 && mkdir venvs
		/target_path/python3/bin/python3 -m venv /target_path/python3/venvs/flask
		```

	+ 安装依赖模块 

		```shell
		cd ~
		/target_path/python3/venvs/flask/bin/pip3 install --no-index --find-links=/unzip_path/whl -r /unzip_path/whl/requirements.txt
		```

## 项目代码解压与uwsgi配置

- 解压项目代码到指定目录并进入目录

	```sh
	unzip -d /target_path/ python_code.zip
	cd /target_path/flask
	```

- 编辑 uwsgi-name.ini

	```sh
	vim uwsgi-name.ini
	```

	编辑内容如下：

	```ini
	[uwsgi]
	http = 0.0.0.0:xxxx
	chdir = /target_path/flask
	wsgi-file = manage.py
	processes = 4
	threads = 2
	post-buffering = 8192
	buffer-size = 65535
	socket-timeout = 10
	uid = user
	master = true
	protocol = uwsgi
	pidfile = /target_path/flask/uwsgi.pid
	py-autoreload = 1
	callable = app
	```

## supervisor配置与启动

- 安装supervisor

	```sh
	/target_path/python3/bin/pip3 install --no-index /unzip_path/whl/supervisor-4.2.5-py2.py3-none-any.whl
	```

- 创建supervisor目录并初始化supervisor配置

	```sh
	mkdir /target_path/supervisor
	cd /target_path/supervisor && mkdir conf.d
	cd ~ && /target_path/python3/bin/echo_supervisord_conf > /target_path/supervisor/supervisord.conf
	```

- 更改supervisor配置

	```sh
	vim /target_path/supervisor/supervisord.conf
	```

	跳转到文件最后并修改最后两行为：
	
	```ini
	[include]
	files = /target_path/supervisor/conf.d/*.ini
	```

- 创建项目配置文件并启动

	```sh
	cd /target_path/supervisor/conf.d
	vim flask.ini
	```

	内容如下：

	```ini
	[program:flask]
	command=/target_path/python3/venvs/flask/bin/uwsgi --ini /target_path/imsp-python/uwsgi-name.ini
	directory=/target_path/flask
	startsecs=10
	startretries=5
	autostart=true
	autorestart=true
	stdout_logfile=/target_path/flask/logs/stdout.log
	stdout_logfile_maxbytes=10MB
	redirect_stderr=true
	user=user
	stopasgroup=true
	killasgroup=true
	environment=FLASK_ENV=DEV  # 以项目中实际配置为准

	```

	启动项目：

	```sh
	# 启动supervisor服务
	/target_path/python3/bin/supervisord -c /target_path/supervisor/supervisord.conf

	# 启动flask项目
	/target_path/python3/bin/supervisorctl -c /target_path/supervisor/supervisord.conf start all
	```

	其他命令：

	```shell
	# 查看flask项目状态
	/target_path/python3/bin/supervisorctl -c /target_path/supervisor/supervisord.conf status all

	# 更新supervisord.conf配置
	/target_path/python3/bin/supervisorctl -c /target_path/supervisor/supervisord.conf update

	# 重载flask项目配置
	/target_path/python3/bin/supervisorctl -c /target_path/supervisor/supervisord.conf reload
	```

---
title: linux下项目环境搭建部署
date: 2017-03-29 14:54:40
tags: linux
categories: 备忘
---
- centos7安装
    不做展示,自行解决
- 数据库安装:

    ```
    yum install mariadb mariadb-server
    ```
    1. 启动mariadb服务程序并添加到开机启动项中：
    ```
    [root@linuxprobe ~]# systemctl start mariadb
    [root@linuxprobe ~]# systemctl enable mariadb
    ```
    2. 为了保证数据库的安全性，一定要进行初始化工作：
    
    第1步：设定root用户密码。
    第2步：删除匿名帐号。
    第3步：禁止root用户从远程登陆。
    第4步：删除test数据库并取消对其的访问权限。
    第5步：刷新授权表，让初始化后的设定立即生效。
    ```
    [root@linuxprobe ~]# mysql_secure_installation
    Enter current password for root (enter for none): 当前数据库密码为空，直接敲击回车。
    Set root password? [Y/n] y
    New password: 输入要为root用户设置的数据库密码。
    Re-enter new password: 重复再输入一次密码。
    Password updated successfully!
    Remove anonymous users? [Y/n] y（删除匿名帐号）
     ... Success!
    Disallow root login remotely? [Y/n] y(禁止root用户从远程登陆)
     ... Success!
    Remove test database and access to it? [Y/n] y(删除test数据库并取消对其的访问权限)
     - Dropping test database...
     ... Success!
     - Removing privileges on test database...
     ... Success!
    Reload privilege tables now? [Y/n] y(刷新授权表，让初始化后的设定立即生效)
     ... Success!
     ```
    3. 设置防火墙对数据库服务的允许策略：
    ```
    [root@linuxprobe ~]# firewall-cmd --permanent --add-service=mysql
    success
    [root@linuxprobe ~]# firewall-cmd --reload
    success
    ```
    4. 使用root用户登陆到数据库中：
    ```
    [root@linuxprobe ~]# mysql -u root -p
    Enter password: 此处输入root用户在数据库中的密码。
    Welcome to the MariaDB monitor. Commands end with ; or \g.
    ```
    5. 创建一个新的数据库用户：
    
    创建数据库用户的命令:CREATE USER 用户名@主机名 IDENTIFIED BY '密码';
    ```
    MariaDB [(none)]> create user 'herakles'@'%' IDENTIFIED BY '123456';
    Query OK, 0 rows affected (0.00 sec)
    ```
	'%'允许远程登陆此用户
	6. 给予herakles用户对user表单的查询、更新、删除、插入权限：
	```
	MariaDB [mysql]> GRANT SELECT,UPDATE,DELETE,INSERT on mysql.user to luke@localhost;
	Query OK, 0 rows affected (0.00 sec)
	```
 
- redis

    安装必要包
    ```
    yum install gcc
    ```

	安装redis
	```
	#下载
	wget http://download.redis.io/releases/redis-3.0.0.tar.gz
	tar zxvf redis-3.0.0.tar.gz
	cd redis-3.0.0
	#如果不加参数,linux下会报错
	make MALLOC=libc
	```

	启动
	```
	#启动redis
	src/redis-server &
	#关闭redis
	src/redis-cli shutdown
	```

	测试

    ```
    $ src/redis-cli
    127.0.0.1:6379> set foo bar
    OK
    127.0.0.1:6379> get foo
    "bar"
    $
    ```

- MongoDB

	```
	yum install -y mongodb-org
	```
	```
	mongodb服务使用

	#启动
	service mongod start
	#停止
	service mongod stop
	#重启
	service mongod restart
	#增加开机启动
	chkconfig mongod on
	```

未完待续...


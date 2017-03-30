---
title: Linux日常记录
date: 2017-03-27 17:37:05
tags: linux
categories: 备忘
---

- linux开机启动服务目录:/etc/init.d

- 启动redis客户端:/usr/local/redis-3.2.4/src/redis-cli 

- Linux下剪切拷贝命令

命令格式: mv   source    dest

mv: 命令字 source: 源文件 dest: 目的地址

- Linux下拷贝命令

命令格式：cp  [-rf]  source  dest

使用备注：源文件在前，目标文件在后。

参数说明：

-r  若 source 中含有目录，则将目录下之档案亦皆依序拷贝至目的地。
-f  若目的地已经有相同档名的档案存在，则在复制前先予以删除再行复制。

- 删除命令

1、删除home目录下的test目录

 rm /home/test

2、这种不带参数的删除方法经常会提示无法删除，因为权限不够。

 rm -r /home/test

3、-r是递归的删除参数表中的目录及其子目录。 目录将被清空并且删除。 当删除目录包含的具有写保护的文件时用户通常是被提示的。

rm -rf /home/test

-4、f是不提示用户，删除目录下的所有文件。请注意检查路径，输成别的目录就悲剧了。

 rm -ir /home/test

5、-i是交互模式。使用这个选项，rm命令在删除任何文件前提示用户确认。

- Yum软件仓库

|命令|	作用|
| ---- | ---- |
|yum r...epolist all|	列出所有仓库。|
|yum list all|	列出仓库中所有软件包|
|yum info 软件包名称|	查看软件包信息|
|yum install 软件包名称|	安装软件包|
|yum reinstall 软件包名称|	重新安装软件包|
|yum update 软件包名称|	升级软件包|
|yum remove 软件包|	移除软件包|
|yum clean all|	清除所有仓库缓存|
|yum check-update|	检查可更新的软件包|
|yum grouplist|	查看系统中已经安装的软件包组|
|yum groupinstall 软件包组|	安装指定的软件包组|
|yum groupremove软件包组	|移除指定的软件包组|
|yum groupinfo 软件包组|	查询指定的软件包组信息|


- systemctl管理服务的启动、重启、停止、重载、查看状态的命令：

|Sysvinit命令(红帽RHEL6系统)	|Systemctl命令（红帽RHEL7系统）	|作用|
|---|---|---|
|service foo start	|systemctl start foo.service	|启动服务|
|service foo restart	|systemctl restart foo.service	|重启服务|
|service foo stop	|systemctl stop foo.service	|停止服务|
|service foo reload	|systemctl reload foo.service	|重新加载配置文件（不终止服务）|
|service foo status	|systemctl status foo.service	|查看服务状态|

- systemctl设置服务的开机启动、不启动、查看各级别下服务启动状态的命令：

|Sysvinit命令(红帽RHEL6系统)	|Systemctl命令（红帽RHEL7系统）	|作用|
|---|---|---|
|chkconfig foo on	|systemctl enable foo.service	|开机自动启动|
|chkconfig foo off	|systemctl disable foo.service	|开机不自动启动|
|chkconfig foo	|systemctl is-enabled foo.service	|查看特定服务是否为开机自启动|
|chkconfig --list	|systemctl list-unit-files --type=service	|查看各个级别下服务的启动与禁用情况|


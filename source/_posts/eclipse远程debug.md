---
title: eclipse远程debug
date: 2017-03-27 15:11:43
tags: eclipse
categories: 有关技术
---

远程Debug(Jboss-eap-6.2为例):

服务器:

..\bin\standalone.conf

\#JAVA_OPTS="$JAVA_OPTS -agentlib:jdwp=transport=dt_socket,address=8787,server=y,suspend=n"

打开注释

eclipse:

右键项目->debug as->debug configuration->remote java application

host:服务器IP

port:8787(可以自己改跟address=xxxx,一样)

注意,

同时只能一个用户远程debug.否则会报错.

远程服务器部署的代码跟本地要一样.
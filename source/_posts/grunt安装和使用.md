---
title: grunt安装和使用
date: 2017-03-27 15:23:15
tags: gtunt
categories: 备忘
---
## 安装grunt

Grunt运行离不开NodeJS和NPM。因此要使用Grunt首要的条件，你的系统需要安装NodeJS和NPM。

> * 安装NodeJS

node安装的方法很多，如果是wiondow系统,可以直接进入Nodejs官网中下载各系统所需要的安装包进行安装。

> * 安装NPM

装好NodeJS后，可以在你的终端执行下面的命令安装NPM:

curl http://npmjs.org/install.sh | sh

如果这样安装失败，或许你要在上面的命令之前加上sudo，并按提示输入你的用户密码。

如果需要检验安装NodeJS或NPM是否要成功，可以通过下面的命令来检验：

$ node -v

v0.10.13

$ npm -v

1.3.2

这样你的NodeJS和NPM表示安装成功。

> * 安装CLI

为了方便使用Grunt，你应该在全局范围内安装Grunt的命令行接口(CLI)。在命令行中执行：

$ npm install -g grunt-cli

这条命令将会把grunt命令植入到你的系统路径中，这样就允许你从任意目录中运行Grunt(定位到任意目录运行grunt命令)

> * 运行grunt

进入到Gruntfile.js的目录,在命令行执行：

$ npm install

等待命令执行后，再执行:

$ grunt

即可把要压缩处理的文件处理到Gruntfile.js配置的目录

前端开发同学在开发过程中不需要每次都运行上面的grunt命令，可以在命令行运行一个：

$ grunt dev

即可时时监听修改过的文件。

## 不能运行npm install时,运行下面的命令...

1.通过config命令

npm config set registry https://registry.npm.taobao.org  npm info underscore （如果上面配置正确这个命令会有字符串response）

2.命令行指定

npm --registry https://registry.npm.taobao.org info underscore

3.编辑~/.npmrc加入下面内容

registry = https://registry.npm.taobao.org

npm可能是默认的proxy http://127.0.0.1.8087

$ npm config set proxy null

把代理去掉就行了
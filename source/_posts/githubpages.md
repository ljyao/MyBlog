---
title: github搭建blog
date: 2016-01-12 16:30:16
category: hexo
tags: hexo
---
# 1.创建github仓库
在github中创建一个名字为 xxx.github.io 的仓库,xxx为github为账号。
# 2.使用hexo 主题
## 2.1 安装软件
- 安装Node.js
[node.js官网](https://nodejs.org/en/)
- 安装Hexo
```
$ npm install -g hexo-cli
```
  [hexo官网](https://hexo.io/zh-cn/)
## 2.2 使用jacman主题
```
$ git clone https://github.com/wuchong/jacman.git themes/jacman
```
[github地址](https://github.com/wuchong/jacman)
# 3.域名绑定
## 3.1在freemon申请免费一级域名
[freenom官网](http://www.freenom.com/)
## 3.2添加DNS
在freenom->My DoMains->Manage Domain->Manage Freenom DNS->Add Record 添加2条
	Target：192.30.252.154
	Target：192.30.252.153
## 3.3添加CNAME
在xxx.github.io仓库中添加没有后缀的CNAME文件，内容为域名URL
# 4.git
## 4.1忽略blog源码工程中的文件
 -.gitinore 中添加/public，忽略生成的blog网站代码
 -.gitinore 中添加/.deploy_git
## 4.2上传blog到github
 ```
 $ hexo g
 $ hexo d
 ```
防止每次hexo d删除CNANE,README.md文件，把这2个文件放在blog源码工程的/source目录，
并且README.md重命名为README.MDOWN。






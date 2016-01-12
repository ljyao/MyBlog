# MyBlog
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
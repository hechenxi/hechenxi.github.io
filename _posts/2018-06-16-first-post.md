---
layout:     post
title:      个人建站记录
subtitle:   踩过的一些坑
date:       2018-06-16
author:     HCX
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - 个人建站
---

## 2018-6-16
想在创建本地服务器调试这个blog，所以就要安装jekyll和jekyll bundler。

结果死活装不上jekyll。具体原因不太清楚，基本意思就是`ruby`没有正确地被`./configure`。最后使用的简单粗暴的方式重装了我的`ruby`，那就是：
1. 找到所有和ruby有关的文件夹`whereis ruby`
2. 全部删除，没有权限的给权限`sudo rm -rf ruby`……然后`ruby -v`,发现已经没有`ruby`了
3. 再重新编译、安装我下载的`ruby`，`./configure`,`make`,`sudo make install`

重新安装上了`ruby`，也就自动安装上了`RubyGems`，然后：

`gem install jekyll`

`gem install jekyll boundler`

`gem install jekyll-paginate`

成功安装上`jekyll`，进入Blog的目录下，打开终端

`jekyll s`

成了，提示我们，可以通过127.0.0.1:4000访问我们的Blog啦。
ctrl+c可以停止服务，不过我一直都是ctrl+z



### 参考

- [利用 GitHub Pages 快速搭建个人博客](https://www.jianshu.com/p/e68fba58f75c)
- [Jekyll官方文档](http://jekyllcn.com/docs/home/)

 


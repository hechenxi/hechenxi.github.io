---
layout:     post
title:      Git笔记
subtitle:   推送到GitHub
date:       2019-10-21
author:     HCX
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Git
---

## 配置git用户名和邮箱
```bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```
## 添加和提交
```bash
git add -A
git commit -m "your commit"
```
## push到远程服务器
### 设置密钥
```bash
$ ssh-keygen -t rsa -C "youremail@example.com"
```
### 推送
```bash
git push origin master
```
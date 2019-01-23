---
layout:     post
title:      GitLab 离线安装简易指引
subtitle:   
author:     WeYunx
header-style: text
catalog: true
tags:
    - GitLab
#
---

## 前言

近期为满足持续集成的需要，需要在内网搭建GitLab。

## GitLab 简介

GitLab 是利用 Ruby On Rails 开发的一个开源版本管理系统，实现了一个自托管的 Git 项目仓库，是集代码托管，测试，部署于一体的开源 git 仓库管理软件，可通过 web 界面来进行访问公开的或私人项目。与 Github 类似，GitLab 能够浏览代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本，并提供一个文件历史库。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后需要的时候查找。

## 准备工作

此次安装环境为RHEL7.3，首先下载离线安装包，可以在[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/)中找到。

如将最新版[gitlab-ce-11.5.7-ce.0.el7.x86_64.rpm](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-11.5.7-ce.0.el7.x86_64.rpm)拷贝到内网环境中。

## 开始

```bash
# 进入安装包目录，安装
rpm -ivh gitlab*rpm
# 如果提示缺少 policycoreutils-python 可按如下安装后再安装
yum install policycoreutils-python
```

## 配置

GitLab 的相关参数配置都存在 `/etc/gitlab/gitlab.rb`文件里。GitLab需要你设置好哪个url才是用户可以访问到GitLab，需要编辑下面这个文件

`/etc/gitlab/gitlab.rb`：

```bash
# (MUST) 配置域名访问，替换为你自己的地址
external_url "http://gitlab.example.com"
```

运行 `sudo gitlab-ctl reconfigure` 使修改生效。

其它的配置可参考[这里](https://docs.gitlab.com.cn/omnibus/settings/README.html)

## GitLab 使用

在浏览器的地址栏中输入 IP 即可登录 GitLab 的界面，用户是`root`，首次登录需要修改密码。



*未完待续...*



## 参考

- https://docs.gitlab.com.cn/





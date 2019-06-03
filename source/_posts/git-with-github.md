---
title: Git 与 Github 关联
categories:
  - GitHub Pages
tags:
  - Git
  - Github
date: 2019-06-03 15:32:30
---
## 安装Git

什么是`Git` ?简单来说 `Git` 是开源的分布式版本控制系统，用于敏捷高效地处理项目。我们网站在本地搭建好了，需要使用Git同步到GitHub上。如果想要了解Git的细节，参看廖雪峰老师的Git教程：Git教程 从Git官网下载：Git - Downloading Package 现在的机子基本都是64位的，选择64位的安装包，下载后安装，在命令行里输入git测试是否安装成功，若安装失败，参看其他详细的Git安装教程。安装成功后，将你的Git与GitHub帐号绑定，鼠标右击打开Git Bash

{% asset_img git_bash_here.jpg Git Bash Here %}

或者在菜单里搜索 `Git Bash`，设置 `user.name` 和 `user.email` 配置信息：
```bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub注册邮箱"
```

## 生成ssh密钥文件：
```
ssh-keygen -t rsa -C "你的GitHub注册邮箱"
```

然后直接三个回车即可，默认不需要设置密码
将生成的 `ssh key` 复制到剪贴板，执行命令：
```bash
clip < ~/.ssh/id_rsa.pub
```
（或者找到生成的.ssh的文件夹中的 `id_rsa.pub` 密钥，将内容全部复制)
{% asset_img id_rsa_pub.jpg id_ras.pub %}

## 添加 Github new SSH Key

打开 `GitHub_Settings_keys` 页面，新建 `new SSH Key`
{% asset_img newkey.jpg new key %}

Title 为标题，任意填即可，将刚刚复制的 `id_rsa.pub` 内容粘贴进去，最后点击 `Add SSH key`。
---
title: github 上适应多终端工作的 Hexo 博客
date: 2019-05-26 14:29:18
tags:
---
## 创建 github 个人仓库

在GitHub.com中看到一个New repository，新建仓库创建一个和你用户名相同的仓库，后面加.github.io，
只有这样，将来要部署到GitHub page的时候，才会被识别，也就是xxxx.github.io，其中xxx就是你注册GitHub的用户名。

例如我这里是: `anntiz.github.io`

点击 create repository。

## 创建分支
在上面创建好的仓库中新建分支 `hexo`，并设置为默认分支

## 克隆分支

然后在本地的任意目录下，打开git bash，执行以下命令克隆仓库：
```bash
git clone git@github.com:anntiz/anntiz.github.io.git
```
接下来在克隆到本地的 `anntiz.github.io` 中，把所有文件都删掉移动到其他地方备份。

在本地的 `anntiz.github.io` 文件夹中通过 `Git bash` 执行以下命令（此时当前分支应显示为`hexo`）：

```bash
npm init -f
npm install hexo
hexo init
npm install
npm install hexo-deployer-git
```

修改_config.yml中的deploy参数，分支应为master，例如：
```yaml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/anntiz/anntiz.github.io.git
  branch: master
```

## 关于日常的改动流程

在本地对博客进行修改（添加新博文、修改样式等等）后，通过下面的流程进行管理。
1. 依次执行git add .、git commit -m "..."、git push origin hexo (git push)指令将改动推送到GitHub（此时当前分支应为hexo）；
2. 然后才执行hexo g、hexo d 发布网站到`master`分支上。


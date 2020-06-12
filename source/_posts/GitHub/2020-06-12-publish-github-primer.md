---
title: GitHub入门
date: 2020-06-12 14:38:20
tags: GitHub
categories: GitHub
toc: true
summary: 上传项目到GitHub
top: false
typora-root-url: 2020-06-12-publish-github
---

# 上传项目到GitHub

在本地新建了项目需要上传到GitHub时使用

1.登陆自己的github账号创建项目

2.把需要上传的项目放在文件中，将鼠标箭头到文件上，鼠标右键，选择Git Bash Here.

使用命令初始化本地仓库   git init
把当前目录下所有的文件及子目录都添加管理，也可以把.换成相应的文件名

```bash
 git add .或git add [filename]
```
把本地仓库暂存区的文件提交到本地仓库

```bash
 git commit -m 'first comment'
```

把本地仓库和远程仓库相关联，其中origin是远程仓库的别名。

```bash
git remote add origin [复制的git仓库地址]
```

要把本地仓库和远程仓库做同步时使用。否则可以省略此步骤，其中master为远程仓库的分支名。

```bash
git pull --rebase origin master
```

把本地仓库中的文件同步到远程仓库中。其中master为远程仓库的分支名。

```bash
git push -u origin master
```

最后可以使用命令查看状态

```bash
git status
```


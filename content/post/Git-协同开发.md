---
title: "Git 协同开发"
date: 2023-02-11 17:50:38
categories: ["技术总结"]
tags: ["Git"]
url: git-notes

################################目录################################
toc: false
autoCollapseToc: false
################################公式渲染################################
mathjax: false

################################基本不动################################
# lastmod: {{ .Date }}
draft: false
# keywords: []
# description: ""
# tags: []
author: "Yaxing"

comment: false
postMetaInFooter: false
hiddenFromHomePage: false
contentCopyright: true
---

本文是本人在实习期间总结的一份关于使用 Git 进行协同开发的小记。<!--more-->

背景：上游分支 release-branch，现在要在 release-branch 基础上进行开发

1、先克隆整个项目，此时本地应该只有 master 分支

2、克隆 release-branch 分支，`git checkout -b release-branch origin/release-branch`，此时本地 release-branch 和远程 release-branch 建立了映射关系，以后可以直接 pull 获得最新版本（tip：如果不想建立追踪关系，可以使用 `git fetch origin release-branch:release-branch`）

3、拉好该分支后，此时处于 release-branch 分支，基于 release-branch 分支拉出来一个开发分支 dev-branch，可以使用 idea 里面的 Git 右键选择 release-branch，然后点击 new branch from ...，建立本地的开发分支 dev-branch

4、在本地开发一段时间后，commit 之后不能直接 push 到远程，因为此时 release-branch 上可能也有更新，所以此时要先使用 rebase 更新本地开发分支，然后再 push，具体操作如下：

- `git checkout <主分支>`
- `git pull`
- `git checkout <开发分支>`
- `git rebase <主分支>`（将当前分支的基底设置为主分支，也就可以把主分支的更新拉过来到当前分支）
- `git push --force origin <开发分支>`
- 提 MR 合并开发分支到下个版本的主分支上

git push

- 第一次使用 push，可以使用 idea 里面的 git，右键 push，可以设置远程分支，没有也会自动创建并设置关联关系。或者使用git 命令：`git push origin feature-branch:feature-branch`

- 后续使用push，可以直接切换到该分支上，git push，但是一般都要进行 rebase 更新，所以使用 `git push --force origin <本地分支名>`

git pull

- 第一次拉某个分支，可以是用 `git checkout -b <主分支> orign/<主分支>`

- 以后要更新，可以直接 git pull，完整版 `git pull origin <远程分支名>:<本地分支名>`

回退

- `git reflog` 查看版本号，在 Head{1/2...} 前面的短字符串

- `git reset --hard <版本号>`

参考

- [git操作之pull拉取远程指定分支以及push推送到远程指定分支](https://blog.51cto.com/u_15262460/2883040)

- [git推送本地分支到远程分支](https://www.cnblogs.com/qyf404/p/git_push_local_branch_to_remote.html)

- [GIT使用rebase和merge的正确姿势](https://zhuanlan.zhihu.com/p/34197548)

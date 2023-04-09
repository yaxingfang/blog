---
title: "fork 后如何同步源的更新内容"
date: 2023-04-09T12:34:49+08:00
categories: ["技术总结"]
tags: ["Git"]
url: git-fork

################################目录################################
toc: false
autoCollapseToc: false
################################评论################################
comment: false
################################公式渲染################################
mathjax: false

################################基本不动################################
author: "Yaxing"
draft: false
hiddenFromHomePage: false
contentCopyright: true
postMetaInFooter: false
# keywords: []
# description: ""
---

最近在玩 ChatGPT 衍生项目的时候，fork 了一个项目，之后看到原项目有了更新，那么就想着怎样把这些更新同步到自己 fork 的项目上。<!--more-->

1、clone 自己 fork 的项目

```bash
git clone https://aaa.git
```

2、添加上有项目

```bash
git remote add https://bbb.git
```

3、拉取更新

```bash
git fetch upstream
```

4、合并更新到自己项目中

```bash
git rebase/merge upstream/main
```

5、更新远程自己项目

```bash
git push origin main
```


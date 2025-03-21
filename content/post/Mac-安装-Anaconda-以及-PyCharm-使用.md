---
title: "Mac 安装 Anaconda 以及 PyCharm 使用"
date: 2023-02-15 17:48:25
categories: ["Mac开发环境配置"]
tags: ["Anaconda"]
url: anaconda-pycharm

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

本文介绍在 Mac 上安装 Anaconda 以及 PyCharm 使用 。<!--more-->

1、官网下载 .dmg 文件，可以到一些国内镜像站上下载速度较快。

2、下载完成后一路 next 安装。

3、配置环境变量

```sh
vim ~/.bash_profile
export PATH=/Users/yaxing/opt/anaconda3/bin:$PATH
```

4、check 是否安装配置成功

```sh
conda info
```

P.S. 每次打开终端都会自动激活 conda 的基础环境 base，通过以下方法取消自动激活

```sh
conda config --set auto_activate_base false
```

5、配置 [清华大学镜像](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)，方便后续下载包

6、打开 PyCharm，new project > Add Interpreter > Add Local Interpreter > 选择左侧的 Conda Environment

其中 location 表示环境的名称及其所在路径，选择 Python version 3.7，配置 Conda executable 为 `/Users/yaxing/opt/anaconda3/bin/conda`，路径修改为自己的安装位置。

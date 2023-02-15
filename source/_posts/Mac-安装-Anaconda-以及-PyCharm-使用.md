---
title: Mac 安装 Anaconda 以及 PyCharm 使用
copyright: true
mathjax: false
categories:
  - Mac开发环境配置
toc: false
date: 2023-02-15 17:48:25
tags:
urlname: anaconda-pycharm
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

5、配置 [清华大学镜像](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)，方便后续下载包

6、打开 PyCharm，new project > Add Interpreter > Add Local Interpreter > 选择左侧的 Conda Environment

其中 location 表示环境的名称及其所在路径，选择 Python version 3.7，配置 Conda executable 为 `/Users/yaxing/opt/anaconda3/bin/conda`，路径修改为自己的安装位置。

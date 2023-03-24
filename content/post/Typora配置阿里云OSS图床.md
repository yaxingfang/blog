---
title: "Typora配置阿里云OSS图床"
date: 2022-12-12 22:11:09
categories: ["技术总结"]
tas: ["博客", "Typora"]
url: image-host

################################目录################################
toc: true
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

comment: true
postMetaInFooter: false
hiddenFromHomePage: false
contentCopyright: true
---

在我们日常使用 Typora 写文档时，一般图片都是放在本地的一个相对路径下，如果将 markdown 文件发送给别人，那么这个文件中的图片别人自然是访问不到的。图床的作用就是把图片存在网站上，对应的图片生成一个链接，markdown 文件中使用该链接访问图片。在本文中我们使用 Typora + PicGo + 阿里云 OSS 图床实现图片自动上传并自由访问。

<!--more-->

## 下载 Typora

[Typora官网](https://typora.io/) 官网拉到最下，点击 [History Releases](https://typora.io/releases/all)，选择 Dev/Beta Releases，拉到最下 More Beta,

选择对应系统的链接 [old Windows / Linux beta](https://typora.io/windows/dev_release.html)，[old macOS beta](https://typora.io/dev_release.html)

Windows x64：[Download old version (Windows x64)](https://download.typora.io/windows/typora-update-x64-1016.exe)

Mac OS：[Download v0.11.18](https://download.typora.io/mac/Typora-0.11.18.dmg)

## 常用设置

### 通用

1. 打开自动保存
2. 关闭自动检查更新
3. 关闭发送匿名使用数据

### 外观

1. 自定义字体

2. 勾选侧边栏大纲视图折叠和展开

3. 配置 Next 主题

	1. 官网：https://theme.typora.io/theme/NexT/

	2. 下载 zip 压缩包：[typora-theme-next.zip](https://github.com/BillChen2k/typora-theme-next/releases/download/1.1.1/typora-theme-next.zip)

	3. Typora 左上角，文件 - 偏好设置 - 外观 - 打开主题文件夹`xxx`

	4. 把 typora-theme-next.zip 解压，将所有文件复制/剪切到主题文件夹`xxx`下
	5. 重启 Typora，设置外观选择 Next 即可
	6. 如果感觉页面宽度太小，可以打开 next.css，搜索并配置 max-width
	7. 如果感觉整体页面偏小，可以`Ctrl Shift +/-`增大/减小整体页面

### 编辑器

1. 设置默认换行符
2. 取消拼写检查，勾选`不使用拼写检查`

### 图像

1. 插入图片时

	选择上传图片，勾选`对本地位置的图片应用上述规则`和`对网络位置的图片应用上述规则`

2. 上传服务设定

	接下来会具体讲到图床的配置

### Markdown

1. 勾选显示代码块行号

## 图床配置

### 1、安装PicGo

下载地址：https://github.com/Molunerfinn/PicGo/releases

安装 PicGo-Setup-2.3.0-beta.7-ia32.exe，在 PicGo 中打开 PicGo 设置，找到设置 Server，点击设置，点击开启 Server，点击确定即可。

### 2、配置Typora

文件-偏好设置-图像-设置插入图片时 上传图片-上传服务选择 picgo.app，选择 picgo 的安装路径，验证图片上传选项

### 3、阿里云OSS搭建图床

#### 3.1、开通阿里云对象存储

开通阿里云对象存储https://www.aliyun.com/product/oss，注册阿里云账号后，开通对象储存，进入对象存储OSS的控制台

#### 3.2、创建bucket

- 在左侧选择概览，然后在右侧 Bucket 管理中创建一个新的 bucket

- 创建 Bucket 具体配置

	> Bucket名字不能有大写字母、地域就近选择、存储类型选择`标准存储`，读写权限`公共读`

- 创建成功后，可以在 Bucket 列表中查看，记住自己的访问域名和地域节点，后面会用到。

#### 3.3、创建AccessKey

页面右上角，鼠标放在头像处，在弹出的框里选择 AccessKey 管理，在弹出的选项框里，选择`继续使用AccessKey`。

进入后，创建一个`AccessKey`。

在弹出的界面里，记住你的`accessKeyId`和`accessKeySecret`。

#### 3.4、了解收费标准

使用默认的 0.12 元/1GB/1 个月即可。

### 4、配置PicGo

我们打开打开 PicGo 的主界面,在图床设置里面选择阿里云 OSS，依照下面注意事项填写信息。

设定 Keyld：填写我们在第三步中获得的 AccessKeyID

设定 KeySecret：填写我们在第三步中获得的 AccessKeyIDSecret

设定储存空间名：填写我们在第二步中填写的 bucket 名称

确认存储区域：填写我们在第二步中查看的地域节点，注意复制的格式：只需要复制 oss-cn-Xxxx 即可，不需要后面的.aliyuncs.com

### 5、测试使用

经过上面的一系列配置之后，我们就完成了 Typora 的图床配置，现在我们可以使用 Markdown 开始写文章了，图片、截图会在粘贴之后，自动通过 PicGo 上传到了远端图床。同时也可以手动将以前在本地存储的图片上传到图床上。

## 参考

https://blog.csdn.net/qq_51808107/article/details/124044961

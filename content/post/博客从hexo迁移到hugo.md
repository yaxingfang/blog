---
title: "博客从 hexo 迁移到 hugo"
date: 2023-03-23T10:49:01+08:00
categories: ["博客"]
tags: ["hexo", "hugo"]
url: hero-to-hugo

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
# tags: []
# lastmod: 2023-03-23T10:49:01+08:00
---

为啥要换博客框架呢？闲的，哈哈哈:no_mouth:。 <!--more-->

其实一方面是听说 hugo 比较快，不用像 hexo 那样猛装依赖，另一方面是看中了 hugo 的一些主题，于是花了一下午时间，折腾了一下。迁移主要涉及博客搭建、博客配置、文章迁移、博客系统这几个部分。

## 博客搭建

1、Mac下直接使用 `Homebrew` 安装：

```sh
brew install hugo
```

2、查看版本信息，检查是否成功

```sh
hugo version
# hugo v0.108.0+extended darwin/amd64 BuildDate=unknown
```

3、搭建

```sh
hugo new site hugoblog
```

4、配置git

```sh
cd hugoblog 
git init    
# 已初始化空的 Git 仓库于 /Users/yaxing/hugoblog/.git/
```

5、设置主题

```sh
git submodule add https://github.com/olOwOlo/hugo-theme-even themes/even
```

此处我是fork了这个主题，然后进行clone，方便后续的favicon的设置。

```sh
git clone -b main git@github.com:yaxingfang/hugo-theme-even.git themes/even
```

6、修改文件夹名称

将 content/posts/ 修改为 content/post/


7、修改 netlify 的部署规则

在 [netilify部署](https://app.netlify.com/sites/yaxing97/settings/deploys) 中Build settings - Build command 由 npm run netlify 修改为 hugo

8、项目生成并上传

```sh
hugo
// hugo server -D 本地预览
git add *
git commit -m "feat: hugo blog first commit"
git remote add origin 远程仓库地址
git branch -M main 
git push -u origin main -f
```

## 博客配置

```toml
baseURL = "https://yaxing97.com/"
languageCode = "en"
defaultContentLanguage = "zh-cn"                             # en / zh-cn / ... (This field determines which i18n file to use)
title = "Yaxing's Blog"
preserveTaxonomyNames = true
enableRobotsTXT = true
enableEmoji = true
theme = "even"
enableGitInfo = false # use git commit log to generate lastmod record # 可根据 Git 中的提交生成最近更新记录。

# Syntax highlighting by Chroma. NOTE: Don't enable `highlightInClient` and `chroma` at the same time!
pygmentsOptions = "linenos=table"
pygmentsCodefences = true
pygmentsUseClasses = true
pygmentsCodefencesGuessSyntax = true

hasCJKLanguage = true     # has chinese/japanese/korean ? # 自动检测是否包含 中文\日文\韩文
paginate = 5                                              # 首页每页显示的文章数
disqusShortname = ""      # disqus_shortname
googleAnalytics = ""      # UA-XXXXXXXX-X
copyright = ""            # default: author.name ↓        # 默认为下面配置的author.name ↓

[author]                  # essential                     # 必需
  name = "Yaxing"

[params]
  version = "4.x"           # Used to give a friendly message when you have an incompatible update
  debug = false             # If true, load `eruda.min.js`. See https://github.com/liriliri/eruda

  since = "2022"            # Site creation time          # 站点建立时间
  # use public git repo url to link lastmod git commit, enableGitInfo should be true.
  # 指定 git 仓库地址，可以生成指向最近更新的 git commit 的链接，需要将 enableGitInfo 设置成 true.
  # gitRepo = ""

  # site info (optional)                                  # 站点信息（可选，不需要的可以直接注释掉）
  logoTitle = "Yaxing's Blog"        # default: the title value    # 默认值: 上面设置的title值
  keywords = ["Java", "日记", "思考", "总结"]
  description = "Yaxing 的个人博客"

  # paginate of archives, tags and categories             # 归档、标签、分类每页显示的文章数目，建议修改为一个较大的值
  archivePaginate = 50

  # show 'xx Posts In Total' in archive page ?            # 是否在归档页显示文章的总数
  showArchiveCount = false

  # The date format to use; for a list of valid formats, see https://gohugo.io/functions/format/
  dateFormatToUse = "2006-01-02"

  # show word count and read time ?                       # 是否显示字数统计与阅读时间
  moreMeta = true

  # Syntax highlighting by highlight.js
  highlightInClient = false

  # 一些全局开关，你也可以在每一篇内容的 front matter 中针对单篇内容关闭或开启某些功能，在 archetypes/default.md 查看更多信息。
  # Some global options, you can also close or open something in front matter for a single post, see more information from `archetypes/default.md`.
  toc = true                                                                            # 是否开启目录
  autoCollapseToc = false   # Auto expand and collapse toc                              # 目录自动展开/折叠
  fancybox = true           # see https://github.com/fancyapps/fancybox                 # 是否启用fancybox（图片可点击）

  # mathjax
  mathjax = false           # see https://www.mathjax.org/                              # 是否使用mathjax（数学公式）
  mathjaxEnableSingleDollar = true                                                     # 是否使用 $...$ 即可進行inline latex渲染
  mathjaxEnableAutoNumber = false                                                       # 是否使用公式自动编号
  mathjaxUseLocalFiles = false  # You should install mathjax in `your-site/static/lib/mathjax`

  postMetaInFooter = true   # contain author, lastMod, markdown link, license           # 包含作者，上次修改时间，markdown链接，许可信息
  linkToMarkDown = false    # Only effective when hugo will output .md files.           # 链接到markdown原始文件（仅当允许hugo生成markdown文件时有效）
  contentCopyright = '<a rel="license noopener" href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>'     # e.g. '<a rel="license noopener" href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>'

  baiduPush = false        # baidu push                  # 百度
  baiduAnalytics = ""      # Baidu Analytics
  baiduVerification = ""   # Baidu Verification
  googleVerification = ""  # Google Verification         # 谷歌

  # Link custom CSS and JS assets
  #   (relative to /static/css and /static/js respectively)
  customCSS = []
  customJS = []

  uglyURLs = false          # please keep same with uglyurls setting

  # Show language selector for multilingual site.
  showLanguageSelector = false

  [params.gitalk]           # Gitalk is a comment system based on GitHub issues. see https://github.com/gitalk/gitalk
    enable = true # 启用评论功能
    owner = ""              # Your GitHub ID
    repo = ""               # The repo to store comments
    clientId = ""           # Your client ID
    clientSecret = ""       # Your client secret
    createIssueManually= true 

  [params.busuanzi]         # count web traffic by busuanzi                             # 是否使用不蒜子统计站点访问量
    enable = true
    siteUV = true
    sitePV = true
    pagePV = true
```

## 文章迁移

由于 hugo 的文章 front matter 和 hexo 的不太一样，所以重新为每篇文章修改下 front matter。

修改 archetypes/default.md 文件，这个是每次 hugo new post/title.md 所使用的骨架/模板文件。

```toml
title: "{{ replace .TranslationBaseName "-" " " | title }}"
date: {{ .Date }}
categories: ["category1", "category2"]
# 设置url为xxx时，部署后的该文章链接为 https://yaxing97.com/xxx
url: 

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
# tags: []
# lastmod: {{ .Date }}
```

## 评论系统

使用 gitalk 搭建博客系统

1、Register a new OAuth application

到 https://github.com/settings/applications/new，其中

```
Application name: blogcomment（随便起个名字无所谓）
Homepage URL: 部署博客主页（https://yaxing97.com）
Authorization callback URL: 部署博客主页（https://yaxing97.com）
```

注册申请后，点击`Generate a new client secret`生产 `Client secret`，`Client ID`和`Client secret`在后面config.toml 中配置时需要用到。

2、新增 `gitalk.html` 文件

在 `themes/even/layouts/partials/` 路径下新建 `gitalk.html`

编辑创建好的`gitalk.html`文件：

```html
{{ if .Site.Params.enableGitalk }}
<div id="gitalk-container"></div>
<link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
<script src="https://unpkg.com/gitalk/dist/gitalk.min.js"></script>
<script>
  const gitalk = new Gitalk({
    clientID: '{{ .Site.Params.Gitalk.clientID }}',
    clientSecret: '{{ .Site.Params.Gitalk.clientSecret }}',
    repo: '{{ .Site.Params.Gitalk.repo }}',
    owner: '{{ .Site.Params.Gitalk.owner }}',
    admin: ['{{ .Site.Params.Gitalk.owner }}'],
    id: location.pathname, // Ensure uniqueness and length less than 50
    distractionFreeMode: false // Facebook-like distraction free mode
  });
  (function() {
    if (["localhost", "127.0.0.1"].indexOf(window.location.hostname) != -1) {
      document.getElementById('gitalk-container').innerHTML = 'Gitalk comments not available by default when the website is previewed locally.';
      return;
    }
    gitalk.render('gitalk-container');
  })();
</script>
{{ end }}
```

3、配置config.toml

最后一步便是配置toml文件（根目录下），因为之前的html中很多关键变量使用了表达式的形式，这就使我们可以利用配置文件灵活更改。

```toml
  [params.gitalk]           # Gitalk is a comment system based on GitHub issues. see https://github.com/gitalk/gitalk
    enable = true # 启用评论功能
    owner = ""              # Your GitHub ID
    repo = ""               # The repo to store comments
    clientId = ""           # Your client ID
    clientSecret = ""       # Your client secret
    createIssueManually= true 
```


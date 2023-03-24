---
title: "博客从hexo迁移到hugo"
date: 2023-03-23T10:49:01+08:00
categories: ["博客"]
tags: ["hexo", "hugo"]
url: hero-to-hugo

################################目录################################
toc: true
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

其实一方面是听说 hugo 比较快，不用像 hexo 那样猛装依赖，另一方面是看中了 hugo 的一些主题，于是花了一下午时间，折腾了一下。迁移主要涉及博客搭建、部署设置、博客配置、文章迁移、博客系统这几个部分。

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

6、修改文件夹名称

将 content/posts/ 修改为 content/post/

## 部署设置

1、修改主题目录下的 netlify文件

```toml
[build]
  publish = "blog/public"
  command = "hugo -s blog"

[build.environment]
  HUGO_THEME = "repo"
  HUGO_THEMESDIR = "/opt/build"
  HUGO_VERSION = "0.108.0"

[context.production.environment]
  HUGO_BASEURL = "https://yaxing97.com/"

[context.deploy-preview]
  command = "hugo -s blog -b $DEPLOY_PRIME_URL"

[[headers]]
  for = "/*"
  [headers.values]
    Cache-Control = "public, max-age=600"

[[headers]]
  for = "*.(css|js|woff|woff2|ttf|png|jpg|jpeg)"
  [headers.values]
    Cache-Control = "public, max-age=2592000"
```

2、修改 netlify 的部署规则

在 [netilify部署](https://app.netlify.com/sites/yaxing97/settings/deploys) 中Build settings - Build command 由 npm run netlify 修改为 hugo

3、项目生成并上传

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

[sitemap]                 # essential                     # 必需
  changefreq = "weekly"
  priority = 0.5
  filename = "sitemap.xml"

[[menu.main]]             # config your menu              # 配置目录
  name = "Home"
  weight = 10
  identifier = "home"
  url = "/"
[[menu.main]]
  name = "Archives"
  weight = 20
  identifier = "archives"
  url = "/post/"
# [[menu.main]]
#   name = "Tags" 由于不想要tag所以注释掉了
#   weight = 30
#   identifier = "tags"
#   url = "/tags/"
[[menu.main]]
  name = "Categories"
  weight = 40
  identifier = "categories"
  url = "/categories/"

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

  changyanAppid = ""        # Changyan app id             # 畅言
  changyanAppkey = ""       # Changyan app key

  livereUID = ""            # LiveRe UID                  # 来必力

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

  [params.publicCDN]        # load these files from public cdn                          # 启用公共CDN，需自行定义
    enable = true
    jquery = '<script src="https://cdn.jsdelivr.net/npm/jquery@3.2.1/dist/jquery.min.js" integrity="sha256-hwg4gsxgFZhOsEEamdOYGBf13FyQuiTwlAQgxVSNgt4=" crossorigin="anonymous"></script>'
    slideout = '<script src="https://cdn.jsdelivr.net/npm/slideout@1.0.1/dist/slideout.min.js" integrity="sha256-t+zJ/g8/KXIJMjSVQdnibt4dlaDxc9zXr/9oNPeWqdg=" crossorigin="anonymous"></script>'
    fancyboxJS = '<script src="https://cdn.jsdelivr.net/npm/@fancyapps/fancybox@3.1.20/dist/jquery.fancybox.min.js" integrity="sha256-XVLffZaxoWfGUEbdzuLi7pwaUJv1cecsQJQqGLe7axY=" crossorigin="anonymous"></script>'
    fancyboxCSS = '<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fancyapps/fancybox@3.1.20/dist/jquery.fancybox.min.css" integrity="sha256-7TyXnr2YU040zfSP+rEcz29ggW4j56/ujTPwjMzyqFY=" crossorigin="anonymous">'
    timeagoJS = '<script src="https://cdn.jsdelivr.net/npm/timeago.js@3.0.2/dist/timeago.min.js" integrity="sha256-jwCP0NAdCBloaIWTWHmW4i3snUNMHUNO+jr9rYd2iOI=" crossorigin="anonymous"></script>'
    timeagoLocalesJS = '<script src="https://cdn.jsdelivr.net/npm/timeago.js@3.0.2/dist/timeago.locales.min.js" integrity="sha256-ZwofwC1Lf/faQCzN7nZtfijVV6hSwxjQMwXL4gn9qU8=" crossorigin="anonymous"></script>'
    flowchartDiagramsJS = '<script src="https://cdn.jsdelivr.net/npm/raphael@2.2.7/raphael.min.js" integrity="sha256-67By+NpOtm9ka1R6xpUefeGOY8kWWHHRAKlvaTJ7ONI=" crossorigin="anonymous"></script> <script src="https://cdn.jsdelivr.net/npm/flowchart.js@1.8.0/release/flowchart.min.js" integrity="sha256-zNGWjubXoY6rb5MnmpBNefO0RgoVYfle9p0tvOQM+6k=" crossorigin="anonymous"></script>'
    sequenceDiagramsCSS = '<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/bramp/js-sequence-diagrams@2.0.1/dist/sequence-diagram-min.css" integrity="sha384-6QbLKJMz5dS3adWSeINZe74uSydBGFbnzaAYmp+tKyq60S7H2p6V7g1TysM5lAaF" crossorigin="anonymous">'
    sequenceDiagramsJS = '<script src="https://cdn.jsdelivr.net/npm/webfontloader@1.6.28/webfontloader.js" integrity="sha256-4O4pS1SH31ZqrSO2A/2QJTVjTPqVe+jnYgOWUVr7EEc=" crossorigin="anonymous"></script> <script src="https://cdn.jsdelivr.net/npm/snapsvg@0.5.1/dist/snap.svg-min.js" integrity="sha256-oI+elz+sIm+jpn8F/qEspKoKveTc5uKeFHNNVexe6d8=" crossorigin="anonymous"></script> <script src="https://cdn.jsdelivr.net/npm/underscore@1.8.3/underscore-min.js" integrity="sha256-obZACiHd7gkOk9iIL/pimWMTJ4W/pBsKu+oZnSeBIek=" crossorigin="anonymous"></script> <script src="https://cdn.jsdelivr.net/gh/bramp/js-sequence-diagrams@2.0.1/dist/sequence-diagram-min.js" integrity="sha384-8748Vn52gHJYJI0XEuPB2QlPVNUkJlJn9tHqKec6J3q2r9l8fvRxrgn/E5ZHV0sP" crossorigin="anonymous"></script>'

  # Display a message at the beginning of an article to warn the readers that it's content may be outdated.
  # 在文章开头显示提示信息，提醒读者文章内容可能过时。
  [params.outdatedInfoWarning]
    enable = false
    hint = 30               # Display hint if the last modified time is more than these days ago.    # 如果文章最后更新于这天数之前，显示提醒
    warn = 180              # Display warning if the last modified time is more than these days ago.    # 如果文章最后更新于这天数之前，显示警告

  [params.gitment]          # Gitment is a comment system based on GitHub issues. see https://github.com/imsun/gitment
    owner = ""              # Your GitHub ID
    repo = ""               # The repo to store comments
    clientId = ""           # Your client ID
    clientSecret = ""       # Your client secret

  [params.utterances]       # https://utteranc.es/
    owner = ""              # Your GitHub ID
    repo = ""               # The repo to store comments

  [params.gitalk]           # Gitalk is a comment system based on GitHub issues. see https://github.com/gitalk/gitalk
    enable = true # 启用评论功能
    owner = ""              # Your GitHub ID
    repo = ""               # The repo to store comments
    clientId = ""           # Your client ID
    clientSecret = ""       # Your client secret
    createIssueManually= true 

  # Valine.
  # You can get your appid and appkey from https://leancloud.cn
  # more info please open https://valine.js.org
  [params.valine]
    enable = false
    appId = '你的appId'
    appKey = '你的appKey'
    notify = false  # mail notifier , https://github.com/xCss/Valine/wiki
    verify = false # Verification code
    avatar = 'mm'
    placeholder = '说点什么吧...'
    visitor = false

  [params.flowchartDiagrams]# see https://blog.olowolo.com/example-site/post/js-flowchart-diagrams/
    enable = false
    options = ""

  [params.sequenceDiagrams] # see https://blog.olowolo.com/example-site/post/js-sequence-diagrams/
    enable = false
    options = ""            # default: "{theme: 'simple'}"

  [params.busuanzi]         # count web traffic by busuanzi                             # 是否使用不蒜子统计站点访问量
    enable = true
    siteUV = true
    sitePV = true
    pagePV = true

  [params.reward]                                         # 文章打赏
    enable = false
    wechat = "/path/to/your/wechat-qr-code.png"           # 微信二维码
    alipay = "/path/to/your/alipay-qr-code.png"           # 支付宝二维码

  [params.social]                                         # 社交链接
    a-email = "mailto:your@email.com"
    b-stack-overflow = "https://yaxing97.com"
    c-twitter = "https://yaxing97.com"
    d-facebook = "https://yaxing97.com"
    e-linkedin = "https://yaxing97.com"
    f-google = "https://yaxing97.com"
    g-github = "https://yaxing97.com"
    h-weibo = "https://yaxing97.com"
    i-zhihu = "https://yaxing97.com"
    j-douban = "https://yaxing97.com"
    k-pocket = "https://yaxing97.com"
    l-tumblr = "https://yaxing97.com"
    m-instagram = "https://yaxing97.com"
    n-gitlab = "https://yaxing97.com"
    o-bilibili = "https://yaxing97.com"

# See https://gohugo.io/about/hugo-and-gdpr/
[privacy]
  [privacy.googleAnalytics]
    anonymizeIP = true      # 12.214.31.144 -> 12.214.31.0
  [privacy.youtube]
    privacyEnhanced = true

# see https://gohugo.io/getting-started/configuration-markup
[markup]
  [markup.tableOfContents]
    startLevel = 1
  [markup.goldmark.renderer]
    unsafe = true

# 将下面这段配置取消注释可以使 hugo 生成 .md 文件
# Uncomment these options to make hugo output .md files.
#[mediaTypes]
#  [mediaTypes."text/plain"]
#    suffixes = ["md"]
#
#[outputFormats.MarkDown]
#  mediaType = "text/plain"
#  isPlainText = true
#  isHTML = false
#
#[outputs]
#  home = ["HTML", "RSS"]
#  page = ["HTML", "MarkDown"]
#  section = ["HTML", "RSS"]
#  taxonomy = ["HTML", "RSS"]
#  taxonomyTerm = ["HTML"]
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


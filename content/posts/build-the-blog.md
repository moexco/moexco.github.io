---
title: "博客搭建指北"
subtitle: ""
date: 2021-03-11T15:58:16+08:00
lastmod: 2021-03-11T15:58:16+08:00
draft: false
author: "moexco"
authorLink: ""
description: ""

tags: ["hugo"]
categories: ["博客相关"]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

![miku镇楼 pid:76329537](/build-the-blog/76329537_p0.jpg)

## 指北说明(~~个人记录~~)

> 根据国际惯例，记录一下搭建(踩坑)过程，虽然全程不会涉及什么技术向的东西，但希望读者对 `GitHub` 和 `git` 有所了解

> 该博客使用 `hugo` 生成，搭建在 `Github` 上，使用了自己的域名进行解析，这三步都不是必须的，请根据自己需要食用

## 准备工作
首先你需要：<br>
* ~~一个能连上GitHub的网络~~ (能打开这个已经说明能连上了)

然后你还需要：

1. 下载 `hugo` <br>
  下载地址：`https://github.com/gohugoio/hugo/releases` <br>
  或查阅：[hugo中文文档](https://www.gohugo.org/)下载安装

2. 拥有一个 `GitHub` 账号 <br>
  请前往[Github](https://www.github.com)注册

3. 下载安装 `git` <br>
  下载地址：`https://git-scm.com/downloads`


> OK！到这里你已经准备好了需要的所有东西了

## 创建博客

现在开始在本地创建博客，如果你使用的是 `Windows` 系统的话，我建议你使用 `PowerShell` 或 `git bash` 完成接下来的操

### 一、创建博客目录

1. **使用hugo快速生成目录**

  在你想要存放博客文件的地方，如`~/`根目录下，在终端键入命令：`hugo new site myblog` <br>
  结果如下：<br>
  ![创建hogu站点目录](/build-the-blog/new_site.png)

  键入命令进入目录：`cd myblog` 
  并且查看生成了什么东西：`ls -lp` <br>
  ![站点目录结构](/build-the-blog/myblog.png)

  简单介绍一下用得到的几个，<br>`layout` 是存放自定义布局的，<br>`static` 是存放静态资源的，比方说图片, <br>`themes` 是存放主题的，<br>`content` 是存放博客文章的，<br>`archetypes` 是存放文章模板的，<br>而`config.toml`配置文件则是配置站点最重要的东西了

   
2. **运行站点**

  站点创建好了，当然是创建点文章看看效果了

  直接键入命令：`hugo new posts/about.md`

  会看到在`content`下自动创建了`posts`和`about.md`文件

  运行一下：`hugo server -D`，然后电脑浏览器打开`localhost:1313`即可查看效果，这里就不上图了

  当然如果说你想用手机看一下运行效果，你可以这样运行：`hugo server --bind 0.0.0.0 -D`，然后手机访问电脑的IP端口

> 这时候的站点其实已经可以用了，但是我相信你不会满足于此，这个站点可以说什么都没有，设置都不够好看，所以接下来让它好看点吧

  

  随后，修改`posts.md`里的内容，

### 二、下载主题

> 你可能会说，怎么自己搞一套样式出来，当然，不过这不在这篇使用文章的讨论范围内

#### 选择主题
hugo非常人性化的提供了一个集中大部分hugo主题的网站，你可以在那里选择你喜欢的主题直接下载使用

英文官方网站：`https://themes.gohugo.io/` <br>
中文网站：`https://www.gohugo.org/theme/` <br>

你要问我这俩是不是内容一样，我不敢肯定，至少第一眼内容是有点不一样的

#### 下载导入使用

如果你在上面已经选择好了想要使用的主题，那就快点导入试试吧，我这里以本站的主题[LoveIt](https://themes.gohugo.io/loveit/)为例

万幸的是它不仅有中文说明，更有[完备的文档](https://hugoloveit.com/zh-cn/theme-documentation-basics/)

首先将它作为`git`子模块下载到`themes`里：<br>
你需要先初始化本地git仓库：`git init`，然后clone下载：`git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt`
![下载主题](/build-the-blog/clone_theme.png)

> 如果clone命令显示不大一样，你可能需要查看一下是不是网络不顺畅



随后，我们删掉之前创建的about.md文件，从主题复制一份文章模板：`cp themes/LoveIt/archetypes/default.md ./archetypes/posts.md`，并且根据自己的个人情况修改模板里的内容，比方说`auther: "moexco"`，将文章作者修改为自己，毕竟网站用户大概只有自己，就不用每次创建文章都手动写了

然后重新创建文章，命令跟之前一样：`hugo new posts/about.md`

> 别着急，你现在启动不起来的，因为还需要配置config.toml

### 三、配置主题

#### 基础配置
文档里会有一份基础配置文件，我们需要把它复制到config.toml里，我这里贴一个备份，建议从文档里复制：<br>
```
baseURL = "http://example.org/"
# [en, zh-cn, fr, ...] 设置默认的语言
defaultContentLanguage = "zh-cn"
# 网站语言, 仅在这里 CN 大写
languageCode = "zh-CN"
# 是否包括中日韩文字
hasCJKLanguage = true
# 网站标题
title = "我的全新 Hugo 网站"

# 更改使用 Hugo 构建网站时使用的默认主题
theme = "LoveIt"

[params]
  # LoveIt 主题版本
  version = "0.2.X"

[menu]
  [[menu.main]]
    identifier = "posts"
    # 你可以在名称 (允许 HTML 格式) 之前添加其他信息, 例如图标
    pre = ""
    # 你可以在名称 (允许 HTML 格式) 之后添加其他信息, 例如图标
    post = ""
    name = "文章"
    url = "/posts/"
    # 当你将鼠标悬停在此菜单链接上时, 将显示的标题
    title = ""
    weight = 1
  [[menu.main]]
    identifier = "tags"
    pre = ""
    post = ""
    name = "标签"
    url = "/tags/"
    title = ""
    weight = 2
  [[menu.main]]
    identifier = "categories"
    pre = ""
    post = ""
    name = "分类"
    url = "/categories/"
    title = ""
    weight = 3

# Hugo 解析文档的配置
[markup]
  # 语法高亮设置 (https://gohugo.io/content-management/syntax-highlighting)
  [markup.highlight]
    # false 是必要的设置 (https://github.com/dillonzq/LoveIt/issues/158)
    noClasses = false

```

> 请将baseURL这一项修改为你的域名或{yourname}.github.io
> 这时候启动会遇到一个报错，该报错其实在[issuse](https://github.com/dillonzq/LoveIt/issues/518)中有说明，这里也需要修改一下

#### 修复启动错误

编辑 `themes/LoveIt/layouts/partials/function/content.html`，将里面内容改为：<br>
```
{{- $content := .Content -}}

{{- if ne "" $content -}}

{{- if .Ruby -}}
    {{- $content = partial "function/ruby.html" $content -}}
{{- end -}}

{{- if .Fraction -}}
    {{- $content = partial "function/fraction.html" $content -}}
{{- end -}}

{{- if .Fontawesome -}}
    {{- $content = partial "function/fontawesome.html" $content -}}
{{- end -}}

{{- $content = partial "function/checkbox.html" $content -}}

{{- $content = partial "function/escape.html" $content -}}

{{- end -}}

{{- return $content -}}
```

这时候就可以启动了

#### 详细配置

> 接下来的事情就是更详细的自定义了，这里查看文档里的详细配置，根据自己需要，一个个复制进去，然后修改配置项就行了，这里也贴一份详细配置备份，注意不能直接覆盖整份配置文件，除非你知道你在做什么

```
[params]
  #  LoveIt 主题版本
  version = "0.2.X"
  # 网站描述
  description = "这是我的全新 Hugo 网站"
  # 网站关键词
  keywords = ["Theme", "Hugo"]
  # 网站默认主题样式 ("light", "dark", "auto")
  defaultTheme = "auto"
  # 公共 git 仓库路径，仅在 enableGitInfo 设为 true 时有效
  gitRepo = ""
  #  哪种哈希函数用来 SRI, 为空时表示不使用 SRI
  # ("sha256", "sha384", "sha512", "md5")
  fingerprint = ""
  #  日期格式
  dateFormat = "2006-01-02"
  # 网站图片, 用于 Open Graph 和 Twitter Cards
  images = ["/logo.png"]

  #  应用图标配置
  [params.app]
    # 当添加到 iOS 主屏幕或者 Android 启动器时的标题, 覆盖默认标题
    title = "LoveIt"
    # 是否隐藏网站图标资源链接
    noFavicon = false
    # 更现代的 SVG 网站图标, 可替代旧的 .png 和 .ico 文件
    svgFavicon = ""
    # Android 浏览器主题色
    themeColor = "#ffffff"
    # Safari 图标颜色
    iconColor = "#5bbad5"
    # Windows v8-10磁贴颜色
    tileColor = "#da532c"

  #  搜索配置
  [params.search]
    enable = true
    # 搜索引擎的类型 ("lunr", "algolia")
    type = "lunr"
    # 文章内容最长索引长度
    contentLength = 4000
    # 搜索框的占位提示语
    placeholder = ""
    #  最大结果数目
    maxResultLength = 10
    #  结果内容片段长度
    snippetLength = 50
    #  搜索结果中高亮部分的 HTML 标签
    highlightTag = "em"
    #  是否在搜索索引中使用基于 baseURL 的绝对路径
    absoluteURL = false
    [params.search.algolia]
      index = ""
      appID = ""
      searchKey = ""

  # 页面头部导航栏配置
  [params.header]
    # 桌面端导航栏模式 ("fixed", "normal", "auto")
    desktopMode = "fixed"
    # 移动端导航栏模式 ("fixed", "normal", "auto")
    mobileMode = "auto"
    #  页面头部导航栏标题配置
    [params.header.title]
      # LOGO 的 URL
      logo = ""
      # 标题名称
      name = ""
      # 你可以在名称 (允许 HTML 格式) 之前添加其他信息, 例如图标
      pre = ""
      # 你可以在名称 (允许 HTML 格式) 之后添加其他信息, 例如图标
      post = ""
      #  是否为标题显示打字机动画
      typeit = false

  # 页面底部信息配置
  [params.footer]
    enable = true
    #  自定义内容 (支持 HTML 格式)
    custom = ''
    #  是否显示 Hugo 和主题信息
    hugo = true
    #  是否显示版权信息
    copyright = true
    #  是否显示作者
    author = true
    # 网站创立年份
    since = 2019
    # ICP 备案信息，仅在中国使用 (支持 HTML 格式)
    icp = ""
    # 许可协议信息 (支持 HTML 格式)
    license = '<a rel="license external nofollow noopener noreffer" href="https://creativecommons.org/licenses/by-nc/4.0/" target="_blank">CC BY-NC 4.0</a>'

  #  Section (所有文章) 页面配置
  [params.section]
    # section 页面每页显示文章数量
    paginate = 20
    # 日期格式 (月和日)
    dateFormat = "01-02"
    # RSS 文章数目
    rss = 10

  #  List (目录或标签) 页面配置
  [params.list]
    # list 页面每页显示文章数量
    paginate = 20
    # 日期格式 (月和日)
    dateFormat = "01-02"
    # RSS 文章数目
    rss = 10

  # 主页配置
  [params.home]
    #  RSS 文章数目
    rss = 10
    # 主页个人信息
    [params.home.profile]
      enable = true
      # Gravatar 邮箱，用于优先在主页显示的头像
      gravatarEmail = ""
      # 主页显示头像的 URL
      avatarURL = "/images/avatar.png"
      #  主页显示的网站标题 (支持 HTML 格式)
      title = ""
      # 主页显示的网站副标题
      subtitle = "这是我的全新 Hugo 网站"
      # 是否为副标题显示打字机动画
      typeit = true
      # 是否显示社交账号
      social = true
      #  免责声明 (支持 HTML 格式)
      disclaimer = ""
    # 主页文章列表
    [params.home.posts]
      enable = true
      # 主页每页显示文章数量
      paginate = 6
      #  被 params.page 中的 hiddenFromHomePage 替代
      # 当你没有在文章前置参数中设置 "hiddenFromHomePage" 时的默认行为
      defaultHiddenFromHomePage = false

  # 作者的社交信息设置
  [params.social]
    GitHub = "xxxx"
    Linkedin = ""
    Twitter = "xxxx"
    Instagram = "xxxx"
    Facebook = "xxxx"
    Telegram = "xxxx"
    Medium = ""
    Gitlab = ""
    Youtubelegacy = ""
    Youtubecustom = ""
    Youtubechannel = ""
    Tumblr = ""
    Quora = ""
    Keybase = ""
    Pinterest = ""
    Reddit = ""
    Codepen = ""
    FreeCodeCamp = ""
    Bitbucket = ""
    Stackoverflow = ""
    Weibo = ""
    Odnoklassniki = ""
    VK = ""
    Flickr = ""
    Xing = ""
    Snapchat = ""
    Soundcloud = ""
    Spotify = ""
    Bandcamp = ""
    Paypal = ""
    Fivehundredpx = ""
    Mix = ""
    Goodreads = ""
    Lastfm = ""
    Foursquare = ""
    Hackernews = ""
    Kickstarter = ""
    Patreon = ""
    Steam = ""
    Twitch = ""
    Strava = ""
    Skype = ""
    Whatsapp = ""
    Zhihu = ""
    Douban = ""
    Angellist = ""
    Slidershare = ""
    Jsfiddle = ""
    Deviantart = ""
    Behance = ""
    Dribbble = ""
    Wordpress = ""
    Vine = ""
    Googlescholar = ""
    Researchgate = ""
    Mastodon = ""
    Thingiverse = ""
    Devto = ""
    Gitea = ""
    XMPP = ""
    Matrix = ""
    Bilibili = ""
    Email = "xxxx@xxxx.com"
    RSS = true # 

  #  文章页面配置
  [params.page]
    #  是否在主页隐藏一篇文章
    hiddenFromHomePage = false
    #  是否在搜索结果中隐藏一篇文章
    hiddenFromSearch = false
    #  是否使用 twemoji
    twemoji = false
    # 是否使用 lightgallery
    lightgallery = false
    #  是否使用 ruby 扩展语法
    ruby = true
    #  是否使用 fraction 扩展语法
    fraction = true
    #  是否使用 fontawesome 扩展语法
    fontawesome = true
    # 是否在文章页面显示原始 Markdown 文档链接
    linkToMarkdown = true
    #  是否在 RSS 中显示全文内容
    rssFullText = false
    #  目录配置
    [params.page.toc]
      # 是否使用目录
      enable = true
      #  是否保持使用文章前面的静态目录
      keepStatic = true
      # 是否使侧边目录自动折叠展开
      auto = true
    #  代码配置
    [params.page.code]
      # 是否显示代码块的复制按钮
      copy = true
      # 默认展开显示的代码行数
      maxShownLines = 10
    #  KaTeX 数学公式
    [params.page.math]
      enable = true
      # 默认块定界符是 $$ ... $$ 和 \\[ ... \\]
      blockLeftDelimiter = ""
      blockRightDelimiter = ""
      # 默认行内定界符是 $ ... $ 和 \\( ... \\)
      inlineLeftDelimiter = ""
      inlineRightDelimiter = ""
      # KaTeX 插件 copy_tex
      copyTex = true
      # KaTeX 插件 mhchem
      mhchem = true
    #  Mapbox GL JS 配置
    [params.page.mapbox]
      # Mapbox GL JS 的 access token
      accessToken = ""
      # 浅色主题的地图样式
      lightStyle = "mapbox://styles/mapbox/light-v9"
      # 深色主题的地图样式
      darkStyle = "mapbox://styles/mapbox/dark-v9"
      # 是否添加 NavigationControl
      navigation = true
      # 是否添加 GeolocateControl
      geolocate = true
      # 是否添加 ScaleControl
      scale = true
      # 是否添加 FullscreenControl
      fullscreen = true
    #  文章页面的分享信息设置
    [params.page.share]
      enable = true
      Twitter = true
      Facebook = true
      Linkedin = false
      Whatsapp = true
      Pinterest = false
      Tumblr = false
      HackerNews = false
      Reddit = false
      VK = false
      Buffer = false
      Xing = false
      Line = true
      Instapaper = false
      Pocket = false
      Digg = false
      Stumbleupon = false
      Flipboard = false
      Weibo = true
      Renren = false
      Myspace = true
      Blogger = true
      Baidu = false
      Odnoklassniki = false
      Evernote = true
      Skype = false
      Trello = false
      Mix = false
    #  评论系统设置
    [params.page.comment]
      enable = true
      # Disqus 评论系统设置
      [params.page.comment.disqus]
        # 
        enable = false
        # Disqus 的 shortname，用来在文章中启用 Disqus 评论系统
        shortname = ""
      # Gitalk 评论系统设置
      [params.page.comment.gitalk]
        # 
        enable = false
        owner = ""
        repo = ""
        clientId = ""
        clientSecret = ""
      # Valine 评论系统设置
      [params.page.comment.valine]
        enable = false
        appId = ""
        appKey = ""
        placeholder = ""
        avatar = "mp"
        meta= ""
        pageSize = 10
        lang = ""
        visitor = true
        recordIP = true
        highlight = true
        enableQQ = false
        serverURLs = ""
        #  emoji 数据文件名称, 默认是 "google.yml"
        # ("apple.yml", "google.yml", "facebook.yml", "twitter.yml")
        # 位于 "themes/LoveIt/assets/data/emoji/" 目录
        # 可以在你的项目下相同路径存放你自己的数据文件:
        # "assets/data/emoji/"
        emoji = ""
      # Facebook 评论系统设置
      [params.page.comment.facebook]
        enable = false
        width = "100%"
        numPosts = 10
        appId = ""
        languageCode = "zh_CN"
      #  Telegram Comments 评论系统设置
      [params.page.comment.telegram]
        enable = false
        siteID = ""
        limit = 5
        height = ""
        color = ""
        colorful = true
        dislikes = false
        outlined = false
      #  Commento 评论系统设置
      [params.page.comment.commento]
        enable = false
      #  Utterances 评论系统设置
      [params.page.comment.utterances]
        enable = false
        # owner/repo
        repo = ""
        issueTerm = "pathname"
        label = ""
        lightTheme = "github-light"
        darkTheme = "github-dark"
    #  第三方库配置
    [params.page.library]
      [params.page.library.css]
        # someCSS = "some.css"
        # 位于 "assets/"
        # 或者
        # someCSS = "https://cdn.example.com/some.css"
      [params.page.library.js]
        # someJavascript = "some.js"
        # 位于 "assets/"
        # 或者
        # someJavascript = "https://cdn.example.com/some.js"
    #  页面 SEO 配置
    [params.page.seo]
      # 图片 URL
      images = []
      # 出版者信息
      [params.page.seo.publisher]
        name = ""
        logoUrl = ""

  #  TypeIt 配置
  [params.typeit]
    # 每一步的打字速度 (单位是毫秒)
    speed = 100
    # 光标的闪烁速度 (单位是毫秒)
    cursorSpeed = 1000
    # 光标的字符 (支持 HTML 格式)
    cursorChar = "|"
    # 打字结束之后光标的持续时间 (单位是毫秒, "-1" 代表无限大)
    duration = -1

  # 网站验证代码，用于 Google/Bing/Yandex/Pinterest/Baidu
  [params.verification]
    google = ""
    bing = ""
    yandex = ""
    pinterest = ""
    baidu = ""

  #  网站 SEO 配置
  [params.seo]
    # 图片 URL
    image = ""
    # 缩略图 URL
    thumbnailUrl = ""

  #  网站分析配置
  [params.analytics]
    enable = false
    # Google Analytics
    [params.analytics.google]
      id = ""
      # 是否匿名化用户 IP
      anonymizeIP = true
    # Fathom Analytics
    [params.analytics.fathom]
      id = ""
      # 自行托管追踪器时的主机路径
      server = ""

  #  Cookie 许可配置
  [params.cookieconsent]
    enable = true
    # 用于 Cookie 许可横幅的文本字符串
    [params.cookieconsent.content]
      message = ""
      dismiss = ""
      link = ""

  #  第三方库文件的 CDN 设置
  [params.cdn]
    # CDN 数据文件名称, 默认不启用
    # ("jsdelivr.yml")
    # 位于 "themes/LoveIt/assets/data/cdn/" 目录
    # 可以在你的项目下相同路径存放你自己的数据文件:
    # "assets/data/cdn/"
    data = ""

  #  兼容性设置
  [params.compatibility]
    # 是否使用 Polyfill.io 来兼容旧式浏览器
    polyfill = false
    # 是否使用 object-fit-images 来兼容旧式浏览器
    objectFit = false

# Hugo 解析文档的配置
[markup]
  # 语法高亮设置
  [markup.highlight]
    codeFences = true
    guessSyntax = true
    lineNos = true
    lineNumbersInTable = true
    # false 是必要的设置
    # (https://github.com/dillonzq/LoveIt/issues/158)
    noClasses = false
  # Goldmark 是 Hugo 0.60 以来的默认 Markdown 解析库
  [markup.goldmark]
    [markup.goldmark.extensions]
      definitionList = true
      footnote = true
      linkify = true
      strikethrough = true
      table = true
      taskList = true
      typographer = true
    [markup.goldmark.renderer]
      # 是否在文档中直接使用 HTML 标签
      unsafe = true
  # 目录设置
  [markup.tableOfContents]
    startLevel = 2
    endLevel = 6

# 作者配置
[author]
  name = "xxxx"
  email = ""
  link = ""

# 网站地图配置
[sitemap]
  changefreq = "weekly"
  filename = "sitemap.xml"
  priority = 0.5

# Permalinks 配置
[Permalinks]
  # posts = ":year/:month/:filename"
  posts = ":filename"

# 隐私信息配置
[privacy]
  #  Google Analytics 相关隐私 (被 params.analytics.google 替代)
  [privacy.googleAnalytics]
    # ...
  [privacy.twitter]
    enableDNT = true
  [privacy.youtube]
    privacyEnhanced = true

# 用于输出 Markdown 格式文档的设置
[mediaTypes]
  [mediaTypes."text/plain"]
    suffixes = ["md"]

# 用于输出 Markdown 格式文档的设置
[outputFormats.MarkDown]
  mediaType = "text/plain"
  isPlainText = true
  isHTML = false

# 用于 Hugo 输出文档的设置
[outputs]
  # 
  home = ["HTML", "RSS", "JSON"]
  page = ["HTML", "MarkDown"]
  section = ["HTML", "RSS"]
  taxonomy = ["HTML", "RSS"]
  taxonomyTerm = ["HTML"]

```


#### 浏览器图标和头像

头像是可以在本地环境显示的，但是浏览器标签页图标在`hugo server`下是不会显示的，至少我这里是，所以你需要将博客上云，当然制作图标的工具可以使用这个网站：`https://realfavicongenerator.net/`，下载后放到`static`目录下即可，当然配置文件里的图片目录需要跟你放的一致

> 注意：hugo的静态资源根目录/是static，而不是真的根目录

> 如果你的图片放在/static/img/图片.jpg，你使用时应该是/img/图片.jpg


#### 生产静态文件

生成静态文件非常简单，只需要执行：`hugo`，就在public在生成了一大堆文件

## Github Pages

### 一、创建Repositorie

进入GitHub网站：
![在这里创建](/build-the-blog/create_repo.png)

创建一个github.io项目，注意不要勾选其他文件：
![不要勾选多余的文件](/build-the-blog/github_io.png)

### 二、上传到Github

在上面的yourname.github.io仓库中，有帮助你将本地git上传到Github的说明，你只需要按照它说的上传即可

在`public`依次执行：<br>
```
git add .
git commit -m "first commit"
git remote add origin git@github.com:{你在仓库看到的那一行}
git push origin main
```
这样你就把本地的静态文件推送到GitHub了，然后我们让它可以被访问到吧

### 三、可访问的github.io

进入你的github.io仓库，找到`settings`，往下找，找到`Github Pages` <br>
![github pages](/build-the-blog/page.png)
保存即可，随后等待它可以被访问，你可以在浏览器键入：`yourname.github.io`查看效果

> 这样虽然将博客搭建并可访问了，但是操作会很麻烦，文章md文件在本地，而上传到GitHub的只有public里的文件，还需要每次写完手动hugo一下再上传，如果本地环境没了呢，岂不是完蛋了？别急，用Github Actions自动构建

## Github Actions

> 我们希望，我们可以 `git push` 整个博客源文件到github然后自动更新博客，所以我们要将博客也上传到github上

> 因为我们的public里已经有一个git仓库了，所以我们这里先删掉public，重新hugo生成一个


在博客根目录，开始创建自动构建配置：<br>
`./.github/workflows/hugo.yml`，hugo.yml名字可以换，但是后缀只能是yaml或者yml，路径不能错

在hugo.yml文件中写入：<br>
```
name: github pages

on:
  push:
    branches:
      - pages  # Set a branch to deploy

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.HUGO_BUILD_TOKEN }}
          PUBLISH_BRANCH: main
          publish_dir: ./public

```


在博客目录依次执行：<br>
```
git add .
git commit -m "first commit"
git remote add origin git@github.com:{你在仓库看到的那一行}
git push origin pages
```
对的，每次，跟上面基本一样，唯一不同的就是上传到分支不一样，上面是上传到main分支，这里是pages，当然你要是觉得pages不好听，你可以自己命名，不过修改了pages分支名，hugo.yml配置文件里的pages也需要同步修改

> 进行到这里，还是可以看到GitHub Actions里已经有一个项目在运行了，但是会报错，这是因为需要一个token

进入github，点击用户菜单：<br>
![用户菜单的settings](/build-the-blog/get_token.png)

然后在左侧找到`Developer settings`，选择：<br>
![在Developer settings里](/build-the-blog/access_tokens.png)

点击右侧的`Generate new token`，按照图中勾选repo和workflow，名字也要一样：<br>
![一定要一样](/build-the-blog/new_token.png)

> 然后你会获取到一个token，复制它并保存好它，退出页面后将看不到这个token了

接着进入github.io仓库页，在`settings`中选择`Secrets`，右上角`New repository secret`：<br>
![一定要一样](/build-the-blog/insert.png)

> 名字一定要一样，value里写入刚刚复制的token，之后再提交一次pages分支，或者重新运行一下actions就行了

## 自定义域名

自定义域名可以在博客的/static里创建一个`CNAME`文件，注意没有后缀，然后写上你的域名，域名DNS解析请查看GitHub的文档
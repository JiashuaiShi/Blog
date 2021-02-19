---
title: Hugo使用记录
date: '2020-06-07'
slug: how-to-use-hugo
categories:
  - learn
tags:
  - use-record
---

# hugo安装
windows环境下，下载二进制文件安装，配置环境变量。
开源发布地址：https://github.com/gohugoio/hugo/releases

# 检查环境
```
>>> hugo version
Hugo Static Site Generator v0.68.3-157669A0 linux/amd64 BuildDate: 2020-06-07T12:05:34Z
```
# 创建站点
hugo 安装成功后，使用`hugo new site` 命令创建博客：

```
# 博客项目的名字为myblog
hugo new site myblog
```

这个命令会创建一个名为myblog 的目录，这就是博客的根目录。目录结构如下：
```
├── archetypes
│   └── default.md
├── config.toml         # 博客站点的配置文件
├── content             # 博客文章所在目录
├── data                
├── layouts             # 网站布局
├── static              # 一些静态内容
└── themes              # 博客主题
```

# 新建文章
在content目录新建about.md页面

```
hugo new about.md
```
在post目录新建first.md页面
```
$ hugo new post/first.md
```


# 配置主题
https://themes.gohugo.io/
把主题文件放置到`themes`目录。
```
git clone https://github.com/alanorth/hugo-theme-bootstrap4-blog.git themes/hugo-theme-bootstrap4-blog
```
下载下来的主题会放在themes 目录中：
```
└── hugo-theme-bootstrap4-blog
    ├── CHANGELOG.md
    ├── LICENSE.txt
    ├── README.md
    ├── archetypes
    ├── assets
    ├── exampleSite         # 本主题示例内容
    |      ├── content      # 示例博客文章
    │      |-- static
    │      |-- config.toml  # 本主题配置
    ├── i18n
    ├── images
    ├── layouts
    ├── package-lock.json
    ├── package.json
    ├── screenshot.png
    ├── source
    ├── theme.toml      
    └── webpack.config.js
```

# 本地启动服务
`hugo server` 启动Hugo服务。可以看到服务默认会在占用1313 端口，在浏览器中访问http://localhost:1313/ 地址。
```
PS D:\04_Blog\1_Blog\ShiJiashuai_Blog> hugo server
Building sites … WARN 2021/02/18 00:30:34 .File.Path on zero object. Wrap it in if or with: {{ with .File }}{{ .Path }}{{ end }}

                   | EN
-------------------+-----
  Pages            | 16
  Paginator pages  |  0
  Non-page files   |  0
  Static files     | 38
  Processed images |  0
  Aliases          |  0
  Sitemaps         |  1
  Cleaned          |  0

Built in 133 ms
Watching for changes in D:\04_Blog\1_Blog\ShiJiashuai_Blog\{content,layouts,static,themes}
Watching for config changes in D:\04_Blog\1_Blog\ShiJiashuai_Blog\config.toml
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at //localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```
在myblog 目录下，运行`hugo`命令：
执行成功后，会生成一个public 目录，这个目录中的内容，就是我们博客系统的所有内容，我们需要将这些内容存放在Git 仓库中。

# 编写博客文章
章放在`myblog/content/posts`目录下面。
```
---
文章属性内容
---
Markdown 正文
```
上面是文章属性，下面是正文内容。
常见的属性：
```
---
title: "文章标题"           # 文章标题
author: "作者"              # 文章作者
description : "描述信息"    # 文章描述信息
date: 2015-09-28            # 文章编写日期
lastmod: 2015-04-06         # 文章修改日期

tags = [                    # 文章所属标签
    "文章标签1",
    "文章标签2"
]
categories = [              # 文章所属标签
    "文章分类1",
    "文章分类2",
]
keywords = [                # 文章关键词
    "Hugo",
    "static",
    "generator",
]

next: /tutorials/github-pages-blog      # 下一篇博客地址
prev: /tutorials/automated-deployments  # 上一篇博客地址
---
```
# 本地git管理
```
# 初始化仓库
git init

# 将所有内容添加到git
git add .

# 提交到git 本地
git commit -m "我的博客第一次提交"

# 关联到远程git，注意这里需要写你自己的git 地址
git remote add origin https://github.com/xxx/xxx.github.io.git

# 推送到远程git
git push origin master
```

# GitPages部署
假设你需要部署在 GitHub Pages 上，首先在GitHub上创建一个Repository，命名为：xxx.github.io （xxx替换为你的github用户名）。
```
$ hugo --theme=hyde --baseUrl="http://xxx.github.io/"
```
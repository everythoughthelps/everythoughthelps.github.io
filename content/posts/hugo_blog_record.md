+++
date = '2025-03-24T22:40:57+08:00'
draft = false
title = 'Hugo+Papermod+github pages个人博客搭建记录'
categories = ["通用技术"]
tags = ["博客搭建", "hugo", "papermod"]
comments = true
+++
 
很早就想搭建一个博客了，选了好久感觉Papermod模板不错，接入了mathjax和giscus记录一下搭建过程。
## Hugo和站点的初始化
1. 安装Hugo
```bash 
brew install hugo
hugo version
```
2. 使用hugo在本地创建一个站点，这个命令会创建一个静态站点，其中大部分是默认的脚本，这份代码就是要维护的博客仓库。
```bash
hugo new site myblog
```
3. 将目录初始化为git仓库。
```bash
git init 
git add . 
git commit -m "first commit"
```
4. 添加PaperMod主题。这个命令执行完之后新增的内容，其实就是往 thems 目录下添加了一个主题，而 `.gitmodules` 则是记录了添加的这个主题的模块的信息，
```bash
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```
5. 然后，可以添加一下 `.gitignore` 文件，我这里就直接照抄 PaperMod 的作者部署的那个网站的文件了，
```bash
# Compiled Object files, Static and Dynamic libs (Shared Objects)
*.o
*.a
*.so

# Folders
_obj
_test

# Architecture specific extensions/prefixes
*.[568vq]
[568vq].out

*.cgo1.go
*.cgo2.c
_cgo_defun.c
_cgo_gotypes.go
_cgo_export.*

_testmain.go

*.exe
*.test

/public
.DS_Store
.hugo_build.lock
resources/_gen/
```
## 整体的配置文件
 删除hugo.toml, 新建hugo.yaml
```bash
baseURL: "https://sonnycalcr.github.io/" # 主站的 URL
title: SonnyCalcr's Blog # 站点标题
copyright: "[©2024 SonnyCalcr's Blog](https://sonnycalcr.github.io/)" # 网站的版权声明，通常显示在页脚
theme: PaperMod # 主题
languageCode: zh-cn # 语言

enableInlineShortcodes: true # shortcode，类似于模板变量，可以在写 markdown 的时候便捷地插入，官方文档中有一个视频讲的很通俗
hasCJKLanguage: true # 是否有 CJK 的字符
enableRobotsTXT: true # 允许生成 robots.txt
buildDrafts: false # 构建时是否包括草稿
buildFuture: false # 构建未来发布的内容
buildExpired: false # 构建过期的内容
enableEmoji: true # 允许 emoji
pygmentsUseClasses: true
defaultContentLanguage: zh # 顶部首先展示的语言界面
defaultContentLanguageInSubdir: false # 是否要在地址栏加上默认的语言代码
```
## 配置导航栏
```bash
languages:
  zh:
    languageName: "中文" # 展示的语言名
    weight: 1 # 权重
    taxonomies: # 分类系统
      category: categories
      tag: tags
    # https://gohugo.io/content-management/menus/#define-in-site-configuration
    menus:
      main:
        - name: 首页
          pageRef: /
          weight: 4 # 控制在页面上展示的前后顺序
        - name: 归档
          pageRef: archives/
          weight: 5
        - name: 分类
          pageRef: categories/
          weight: 10
        - name: 标签
          pageRef: tags/
          weight: 10
        - name: 搜索
          pageRef: search/
          weight: 20
        - name: 关于
          pageRef: about/
          weight: 21
```
## 配置归档
在 content 目录下新建 archives.md 文件，内容如下，
```bash
---
title: "归档"
layout: "archives"
url: "/archives/"
summary: archives
---
```
## 配置分类和标签
以本博客为例：
```bash
+++
date = '2025-03-24T22:40:57+08:00'
draft = true
title = 'Hugo_blog_record'
categories = ["通用技术"]
tags = ["博客搭建"]
+++
```
## 配置搜索
在 output参数中加上JSON，
```bash
# https://github.com/adityatelange/hugo-PaperMod/wiki/Features#search-page
outputs:
  home:
    - HTML # 生成的静态页面
    - RSS # 这个其实无所谓
    - JSON # necessary for search, 这里的配置修改好之后，一定要重新生成一下
```
然后，在 content 目录下新建一个 search.md 文件，
```bash
---
title: "搜索" # in any language you want
layout: "search" # necessary for search
summary: "search"
placeholder: "搜索"
---
```
搜索的个性化设置：
```bash
params:
  # 搜索
  fuseOpts:
      isCaseSensitive: false # 是否大小写敏感
      shouldSort: true # 是否排序
      location: 0
      distance: 1000
      threshold: 0.4
      minMatchCharLength: 0
      # limit: 10 # refer: https://www.fusejs.io/api/methods.html#search
      keys: ["title", "permalink", "summary", "content"]
      includeMatches: true
```
## 配置关于about页面
新建两个文件，一个是 layouts\_default 目录下下的 about.html，
```bash
{{- define "main" }}
 
<header class="page-header">
    <h1>{{ .Title }}</h1>
    {{- if .Description }}
    <div class="post-description">
      {{ .Description }}
    </div>
    {{- end }}
  </header>
 
<section>
  <br>
  {{ .Content }}
</section>
 
{{- end }}{{/* end main */}}
```
另一个是 content 目录下的 about.md,
```bash
---
title: "关于"
layout: "about"
url: "/about/"
summary: about
---

这里就可以写一些关于的相关信息了。
```

## 配置评论
使用了[giscus](https://giscus.app/zh-CN)插件实现评论功能。
1. 首先先在 layouts\partials 下新建一个 comments.html 文件，
```bash
<div id="tw-comment"></div>
<script>
    // 默认是暗色，根目录下的配置中的主题默认也是暗色
    const getStoredTheme = () => localStorage.getItem("pref-theme") === "light" ? "{{ .Site.Params.giscus.lightTheme }}" : "{{ .Site.Params.giscus.darkTheme }}";
    const setGiscusTheme = () => {
        const sendMessage = (message) => {
            const iframe = document.querySelector('iframe.giscus-frame');
            if (iframe) {
                iframe.contentWindow.postMessage({giscus: message}, 'https://giscus.app');
            }
        }
        sendMessage({setConfig: {theme: getStoredTheme()}})
    }

    document.addEventListener("DOMContentLoaded", () => {
        const giscusAttributes = {
            "src": "https://giscus.app/client.js",
            "data-repo": "{{ .Site.Params.giscus.repo }}",
            "data-repo-id": "{{ .Site.Params.giscus.repoId }}",
            "data-category": "{{ .Site.Params.giscus.category }}",
            "data-category-id": "{{ .Site.Params.giscus.categoryId }}",
            "data-mapping": "{{ .Site.Params.giscus.mapping }}",
            "data-strict": "{{ .Site.Params.giscus.strict }}",
            "data-reactions-enabled": "{{ .Site.Params.giscus.reactionsEnabled }}",
            "data-emit-metadata": "{{ .Site.Params.giscus.emitMetadata }}",
            "data-input-position": "{{ .Site.Params.giscus.inputPosition }}",
            "data-theme": getStoredTheme(),
            "data-lang": "{{ .Site.Params.giscus.lang }}",
            "data-loading": "lazy",
            "crossorigin": "anonymous",
        };

        // 动态创建 giscus script
        const giscusScript = document.createElement("script");
        Object.entries(giscusAttributes).forEach(
                ([key, value]) => giscusScript.setAttribute(key, value));
        document.querySelector("#tw-comment").appendChild(giscusScript);

        // 页面主题变更后，变更 giscus 主题
        const themeSwitcher = document.querySelector("#theme-toggle");
        if (themeSwitcher) {
            themeSwitcher.addEventListener("click", setGiscusTheme);
        }
        const themeFloatSwitcher = document.querySelector("#theme-toggle-float");
        if (themeFloatSwitcher) {
            themeFloatSwitcher.addEventListener("click", setGiscusTheme);
        }
    });
</script>
```


2. 访问[giscus官网](https://giscus.app/zh-CN)，按照指引配置，分别是打开评论，需要在github仓库中开启Discussions功能，在仓库的Settings -> Features -> Discussions中开启。然后安装giscus, 最后公开仓库，配置正确的话会生成一段代码，放置在配置中，以我的代码为例：
```bash
params:
    # 评论的设置
    giscus:
      repo: "everythoughthelps/everythoughthelps.github.io"
      repoId: "R_kgDOON0ckg"
      category: "Announcements"
      categoryId: "DIC_kwDOON0cks4CoaCL"
      mapping: "pathname"
      strict: "0"
      reactionsEnabled: "1"
      emitMetadata: "0"
      inputPosition: "bottom"
      lightTheme: "preferred_color_scheme"
      darkTheme: "dark"
      lang: "zh-CN"
      crossorigin: "anonymous"
```
1. 在博客的开头配置中，启用评论功能：
```bash
title: "My First Post"
date: 2025-03-24T12:00:00Z
comments: true
```
## 配置mathjax
我们需要添加两个文件，一个是 layouts\partials 下的 mathjax.html 文件，如下，
```bash
<script>
window.MathJax = {
  tex: {
    inlineMath: [['$', '$'], ['\\(', '\\)']],
    displayMath: [['$$', '$$'], ['\\[', '\\]']],
    processEscapes: true,
    processEnvironments: true,
    tags: 'ams'
  },
  options: {
    skipHtmlTags: ['script', 'noscript', 'style', 'textarea', 'pre']
  }
};
</script>
<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>
```
另一个是 layouts\partials 下的 extend_head.html 文件，
```bash
{{- /* Head custom content area start */ -}}
{{- /*     Insert any custom code (web-analytics, resources, etc.) - it will appear in the <head></head> section of every page. */ -}}
{{- /*     Can be overwritten by partial with the same name in the global layouts. */ -}}
{{ partial "mathjax.html" . }}
{{- /* Head custom content area end */ -}}
```
在配置文件中打开数学公式编译：
```bash
params:
  math: true
```
现在数学公式就可以使用了，现在写几个公式测试一下。
公式渲染测试：
行内数学公式：$a^2 + b^2 = c^2$。
块公式，
$$
a^2 + b^2 = c^2
$$

<div>
$$
\boldsymbol{x}_{i+1}+\boldsymbol{x}_{i+2}=\boldsymbol{x}_{i+3}
$$
</div>

自动编号公式：
\begin{equation}
E = mc^2 \label{eq:energy}
\end{equation}

如公式 $\eqref{eq:energy}$ 所示，质量和能量的关系如下。渲染完成！

## 部署到GitHub pages
在GitHub新建一个仓库，仓库名称必须是username.github.io,仓库必须是公开的，不然后续无法使用giscus。然后把博客代码跟新建的仓库地址关联起来。
```bash
git remote add origin https://github.com/yourgithubusername/yourgithubusername.github.io.git
git branch -M main
git add .
git commit -m "Initial commit"
git push -u origin main
```
后面按照hugo的官方部署指导操作即可，[部署连接](https://gohugo.io/host-and-deploy/host-on-github-pages/)

## 发布或者更新博客
后续的博客统一放在content/posts/目录下：
```bash
hugo new content content/posts/my-first-post.md
```
使用编辑器编辑mu-first-post.md后，启动hugo本地服务器预览博客：
```bash
# -D 参数表示包含草稿文章
hugo server -D
```
预览无误后，将原数据中draft: true 改为 flase，取消草稿状态，生成最终静态文件：
```bash
hugo
```
将生成的 public 目录部署到你的托管平台：
```bash
~cd public~ # 这句可以去掉，我们在.gitigonre中忽略了public，这步会网页部署的时候自动生成
git add .
git commit -m "release first post"
git push origin main
```
## 后续完善参考文献引用

## 参考链接
1. https://sonnycalcr.github.io/posts/build-a-blog-using-hugo-papermod-github-pages/

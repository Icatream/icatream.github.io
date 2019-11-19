---
title: GitHub-Pages Hexo Travis NexT
date: 2019-11-14 13:39:36
tags:
categories: essay
---

# Gitpage

用静态 html 编写渲染 markdown 显然是一个好主意.

Gitpage 本身提供的是 Jekyll, Jekyll 在 windows 下环境准备就很麻烦了, 于是找了找其他的, 就 hexo 了.

# Hexo

## 部署

### Travis CI

[GitHub Pages](https://hexo.io/zh-cn/docs/github-pages) 按照官方说明配置 *Travis CI*, 在 *Step 10* 的时候遇到了问题.

在 Hexo 官方文档的配置中, Travis CI 将 build 推到了 *gh-pages* 分支. 而 GitHub Pages 的部署分支无法修改.

根据 github 官方说明

> User pages must be built from the master branch

*username.github.io* 被认为是 *user pages*, 而 *user pages* 只能从 master branch 构建.

既然 Gitpages 的构建分支无法修改, 那只能修改 Travis CI 配置, 把 build 结果推到 master 分支.

在 [Travis Docs](https://docs.travis-ci.com/user/deployment/pages/) 中找到目标分支的配置是 ```deploy.target_branch```

修改 *.travis.yml*

```
deploy:
  target_branch: master
```

这样, Travis CI 会将编译结果推到 github/repository 的 master 分支.

这就得需要另一个分支了, 用于存放源文件, 创建一个新分支, 姑且命名为 *src* 吧, 将源文件推到 src 分支. 同时, 需要修改 *.travis.yml*.

```
branches:
  only:
    - src
deploy
  on:
    branch: src
```

这样, Travis CI 只会在收到 src 分支 push 的时候才会编译 src 分支.

同时, 再建一个 dev 分支, 用于开发, 总不能写到一半的东西就放那, 确定写完之后, 再 merge dev 到 src.

### hexo-deployer-git

有没有必要用 Travis CI? 其实是没有的.

就在上一篇坑爹文档下面 2 行, 就有个使用 [hexo-deployer-git](https://hexo.io/zh-cn/docs/one-command-deployment) 进行部署的文档.

*hexo-deployer* 是本地插件, 在本地编译后上传到指定分支. 官网文档里配置也写的很清楚.

## 主题

### 挑选主题

部署算是完了, 接下来得选择一个 theme. 不同的 theme 差距还是很大的. 

[Hexo Themes](https://hexo.io/themes/index.html) 里有很多主题, 但使用情况就有些堪忧了.

Hexo 的 theme 的使用方式是将 theme 下载下来, 放到 *./themes/* 目录下. 在 *_config.yml* 中修改 *theme* 设置.

绝大部分主题会推荐 ```git clone *** themes/{theme_name}```, git clone 到 themes目录下. 这样 clone 下来的是默认分支, 很多主题维护不佳, 默认分支甚至有些bug...

贴一下我找的几个主题吧.

[clexy](https://github.com/mkkhedawat/clexy) 这是我找的第一个主题, 相当的简洁.

[chan](https://github.com/denjones/hexo-theme-chan) 同样也是一个简洁的主题, 但是不知为何, 修改 *_config.yml* 无法生效, 放弃了.

期间, 还看见 2 个很漂亮的主题.

[diaspora](https://github.com/Fechin/hexo-theme-diaspora) 相当精美的一个主题, 喜欢展示图片的朋友会很喜欢.

[fluid](https://github.com/fluid-dev/hexo-theme-fluid) material design, 很漂亮, 维护人数还挺多的. 主题好不好用, 主要看维护的好不好. 除非像第一个主题一样, 真的很简洁.

我懒得折腾一堆图片, 最终选择了 [next](https://github.com/theme-next/hexo-theme-next).

### NexT

NexT 维护的很好, 功能丰富, 使用起来很方便.

NexT 推荐的安装方式是 ```git clone https://github.com/theme-next/hexo-theme-next themes/next```. 
而 NexT 的配置在 *./themes/next/_config.yml* 里.
如果修改这个, 更新时 pull 会比较麻烦, 很可能会引起冲突.
里面有个 *override* 属性, 查看注释.

> If false, merge configs from `_data/next.yml` into default configuration (rewrite).
> If true, will fully override default configuration by options from `_data/next.yml` (override). Only for NexT settings.
> And if true, all config from default NexT `_config.yml` must be copied into `next.yml`. Use if you know what you are doing.
> Useful if you want to comment some options from NexT `_config.yml` by `next.yml` without editing default config.
> override: false

也就是说, 默认情况下, *_data/next.yml* 会覆盖掉 *./themes/next/_config.yml* 的配置. 这样就很容易了. 建一个 *./source/_data/next.yml* 文件, 把配置写进去. 这样就不用修改 *./themes/next* 里的内容.

### 主题与部署

目前 *./themes/next* 是没有上传 github 的, 这样整个工程是不完整的, Travis CI 是无法编译的. 

要不要把 *./themes/next* 上传?
* 上传也会导致 next 更新的时候变得有些麻烦.
* 不上传, 就只能本地编译, *hexo-deployer* 部署

想想别的办法.

Travis CI 的工作流程是, 执行 *script* 再到 *after*. Travis CI 是绑定 github 的, Travis CI 环境应该也能执行 ```git``` 命令.
在 *.travis.yml* 里 ```hexo generate``` 前加上一行
```
script:
  - git clone https://github.com/theme-next/hexo-theme-next themes/next
  - hexo generate # generate static files
```

## 404

网上找了很多关于 404 配置的内容, 都无法生效. 后面, 找到了一个 [github issues](https://github.com/ppoffice/hexo-theme-icarus/issues/66#issuecomment-166110566) 才明白, 本地重定向是无效的. 所以, 无论怎么设置, 本地 ```hexo server``` 都无法进行 404 页面重定向. 不过, 还是可以通过 <http://localhost:4000/404.html> 进行查看修改页面的.

我没特别准备 404 页面, 就写了一个 md page.

```
---
title: 404
comments: false
permalink: /404
---
```

是否需要这个 ```permalink: /404``` 我就不清楚了, 毕竟本地无法重定向, 得部署到 *github pages* 上才能检查.

## sitemap

blog 里能加的配置还挺多的, 今天还研究了下 sitemap 与 google search console.

```
npm install hexo-generator-sitemap --save
```

首先, 添加 ```hexo``` 的 ```sitemap``` 插件. 再在 *_config.yml* 里加上配置.

```
sitemap:
    path: sitemap.xml
    rel: true
```

部署完成之后, 就可以去 [Google Search Console](https://search.google.com/search-console/) 里添加站点.

在 ```/source/``` 下添加 *robots.txt*.

```
User-agent: *
Allow: /
Allow: /about/
Allow: /archives/
Allow: /categories/
Allow: /tags/

Allow: /images/

Allow: /css/
Allow: /js/
Allow: /lib/

Sitemap: https://icatream.github.io
```

网上看到的内容都是

```
Disallow: /css/
Disallow: /js/
Disallow: /lib/
```

但这样会导致 Googlebot 的抓取异常. 这些内容也都应该被允许抓取.

```hexo``` 官方的 ```hexo-generator-sitemap``` 似乎不够 *SEO (search engine optimization)* 友好. 考虑换其他的 *sitemap-generator*.

# 总结

最终没有上传 *./themes/* , Travis CI 通过 ```git clone``` 获取 *./themes/next*, 之后编译.

那和直接使用 *hexo-deployer* 有什么差距吗?

不知道...
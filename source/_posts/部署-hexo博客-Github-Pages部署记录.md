---
title: '[部署]hexo博客 & Github Pages部署记录'
date: 2024-04-22 21:41:20
tags: 部署
categories: 部署
---

# 1.Windows环境配置

- Git https://git-scm.com/
- Node https://nodejs.org/en

# 2.安装Hexo

```shell
npm config set registry https://registry.npmmirror.com

npm install -g hexo-cli

npm install --save hexo-deployer-git

# 本地测试
hexo init # 初始化博客
hexo generate # 申城网站信息
hexo server # 开始网站服务
```

# 3.Github仓库

在你的github中新建仓库“username.github.io”，其中，username就是你注册时使用的用户名。

修改你博客根目录下的_config.yml文件中的deploy配置项：

```yaml
deploy:
   type: git
   repository: git@github.com:username/username.github.io.git  # 你的仓库地址
   branch: gh-pages
```

他会在你的仓库的gh-pages分支部署你的页面，然后再master分支保存你网站的源代码，不会相互覆盖。

通过如下代码进行部署：

```shell
hexo clean # 清除缓存
hexo generate # 生成网站
hexo deploy # 部署至远程网络

# 简写
hexo cl
hexo g
hexo d

# 简简写
hexo cl
hexo g -d  # hexo d -g
```

然后就可以在浏览器上输入[http://username.github.io](https://link.zhihu.com/?target=http%3A//username.github.io)访问你的博客了。

# 4.Github保存源码

```shell
git init

git add .

git commit -m '...'

git remote add origin git@github.com:oixel64/oixel64.github.io.git

git pull --rebase git@github.com:oixel64/oixel64.github.io.git master

git push
```

设置`.gitignore`

``` gitignore
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
_multiconfig.yml
public/
```

再上GitHub设置master为主分支，删除main分支，以gh-pages分支部署Github Pages

# 5.主题

https://github.com/saicaca/hexo-theme-vivia

```shell
npm install hexo-theme-vivia

mv node_modules/hexo-theme-vivia/example_zh_CN_config.vivia.yml _config.vivia.yml

hexo config theme vivia

hexo new page about
```

编辑`_config.yml` 添加:

```yaml
archive_generator:
  per_page: 0
```

# 6.LaTex公式

https://github.com/next-theme/hexo-filter-mathjax

https://github.com/hexojs/hexo-renderer-pandoc

```shell
npm install hexo-filter-mathjax

hexo clean
```

配置_config.yml:

```yaml
mathjax:
  tags: none # or 'ams' or 'all'
  single_dollars: true # enable single dollar signs as in-line math delimiters
  cjk_width: 0.9 # relative CJK char width
  normal_width: 0.6 # relative normal (monospace) width
  append_css: true # add CSS to pages rendered by MathJax
  every_page: true # if true, every page will be rendered by MathJax regardless the `mathjax` setting in Front-matter
  packages: # extra packages to load
  extension_options: {}
    # you can put your extension options here
    # see http://docs.mathjax.org/en/latest/options/input/tex.html#tex-extension-options for more detail
```
本地电脑安装配置Pandoc后：
```shell
npm uninstall hexo-renderer-marked

npm install hexo-renderer-pandoc --save
```

# 7.写作

```shell
hexo new "[部署]hexo博客 & Github Pages部署记录"

hexo cl

hexo g -d

git add .

git commit -m '...'

git push
```


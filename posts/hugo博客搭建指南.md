+++
title = ' hugo博客搭建指南 '
date = 2024-02-19T21:00:30+08:00
draft = false
+++
## 整体架构

在 github 托管两个仓库，仓库 1 保存博客内容源文件，仓库 2 保存 Hugo 生成的网站文件，博客内容仓库通过 git submodule 的方式在仓库 2 管理。使用 Obsidian git 拉取博客内容仓库，通过 ob 编写博客并推送到仓库 1，推送后触发仓库 2 github action 使用 hugo 构建网站并部署到 github pages。

## 创建 github 仓库

创建博客内容仓库 `blog-content` 和网站仓库 `plyer.github.io`。

网站仓库名称使用 `{github_username}.github.io` 的格式，这样可以直接通过 https://{github_username}.github.io 的 URL 访问博客网站，而不需要加上仓库名称作为 URL Path。

## 使用 hugo 创建网站

首先在本地 PC 中安装 hugo extend，使用 `hugo new site blog` 创建出网站内容。

进入 blog 目录初始化 `plyer.github.io` 仓库，运行以下命令：

```bash
git init
git remote add origin git@github.com:Plyer/plyer.github.io.git
git fetch
git checkout -b master origin/master
```

添加 meme 主题：

```bash
git submodule add --depth 1 git@github.com:reuixiy/hugo-theme-meme.git themes/meme
# 修改配置
rm config.toml && cp themes/meme/config-examples/zh-cn/config.toml config.toml
```

添加 `blog-content`：

```bash
git rm -f content
rm -rf content
git submodule add origin git@github.com:Plyer/blog-content.git content
# 初始化子模块
git submodule update --init --recursive
# 更新子模块仓库
git submodule update --remote
```

编写博客内容并预览：

```bash
hugo new post/test.md
vim post/test.md
# 启动本地服务预览
hugo server
```

输出静态文件到 public 目录命令：`hugo`。这个目录可以不上传 git 远程仓库，github workflow 能处理。

推送到 `plyer.github.io` 仓库：

```bash
git add .
git commit -m "init"
git push
```
## 配置 github workflow

**一、配置 `blog-content` 仓库的 workflow**

创建一个 github ak，包含 plyer 仓库的 workflow 权限。使用 `gh workflow run build.yml -R plyer/plyer.github.io` 触发 ` plyer.github.io` 仓库的 `build.yml` workflw。

二、配置 `plyer.github.io` 仓库的 workflow

1. Checkout 本仓库和子模块
2. 更新子模块内容
3. 安装 hugo 并构建发布到 github pages

## 发布博客

Obsidian 中增加命为 blog 的文件夹，在其中拉取 `blog-content` 仓库，写一篇文章并推送到 github 仓库中，触发 github action 自动构建发布。

---
## 参考文献

- [如何用 GitHub Pages + Hugo 搭建个人博客 · 小绵尾巴 (cuttontail.blog)](https://cuttontail.blog/blog/create-a-wesite-using-github-pages-and-hugo/#2-%E5%AE%89%E8%A3%85-hugo)
- [Hugo + GitHub Action，搭建你的博客自动发布系统 · Pseudoyu](https://www.pseudoyu.com/en/2022/05/29/deploy_your_blog_using_hugo_and_github_action/)
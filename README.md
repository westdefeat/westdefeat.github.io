# WestDefeat Blog

这是一个基于 GitHub Pages 和 Jekyll 的个人博客。

## 写新文章

在 `_posts` 目录下创建 Markdown 文件，文件名格式为：

```text
YYYY-MM-DD-title.md
```

文章开头需要包含 Front Matter：

```md
---
layout: post
title: "文章标题"
date: 2026-04-29 16:12:00 +0800
categories: blog
---
```

## 本地预览

安装 Ruby 后运行：

```bash
bundle install
bundle exec jekyll serve
```

然后访问：

```text
http://localhost:4000
```

---
layout: post
title: "给 VS Code 做了一个 SSH 服务器搜索插件"
date: 2026-04-29 18:25:00 +0800
categories: blog
---

公司服务器一多，VS Code 自带的 Remote SSH 用起来就有点累。

不是不能用，而是它更像一个“能连上”的工具，不太像一个“好找机器”的工具。几十台服务器还能靠记忆和滚动列表硬扛，到了上百台之后，每次找机器都像在翻电话簿：知道大概叫什么，但不确定全名；知道 IP 段，但列表里要慢慢扫；临时要开一个终端，也得重新输入一遍。

所以我写了一个 VS Code / Cursor 扩展：**SSH Targets Manager**。

GitHub：[https://github.com/westdefeat/ssh-targets-manager](https://github.com/westdefeat/ssh-targets-manager)

VS Code Marketplace：[https://marketplace.visualstudio.com/items?itemName=fanxiaolin.ssh-targets-manager](https://marketplace.visualstudio.com/items?itemName=fanxiaolin.ssh-targets-manager)

## 它解决什么问题

我的需求其实很朴素：

1. 能从本地 `~/.ssh/config` 里直接读取服务器。
2. 能搜索服务器，不要再手动翻列表。
3. 能一键连接到当前窗口，也能新开窗口连接。
4. 能直接打开一个 SSH 终端。
5. 不破坏已有 SSH 配置，尤其是 `ProxyCommand`、跳板机这类配置。

换句话说，我不想重新发明 SSH，也不想维护另一份服务器清单。我只想让已经存在的 `ssh/config` 在编辑器里变得更好用。

## 效果

![SSH Targets Manager 预览]({{ "/assets/images/ssh-targets-manager/preview.gif" | relative_url }})

## 实现思路

核心思路很简单：把 `~/.ssh/config` 当作唯一数据源，解析里面的 `Host` 块，然后在 VS Code 侧边栏里渲染成一个 Tree View。

解析时只把明确的主机条目拿出来，像 `Host *`、`Host *.example.com` 这种通配配置不会当成一台具体服务器显示。每个 Host 下面的字段会被收集起来，常用的 `HostName`、`User`、`Port` 单独展示，其它配置保留在原始配置里。

这点对公司内网环境挺重要。很多服务器不是直连，而是通过跳板机：

```ssh-config
Host prod-web-01
  HostName 10.10.12.34
  User deploy
  ProxyCommand ssh jump-host -W %h:%p
```

插件不会尝试自己实现一套 SSH 连接逻辑。真正连接时，还是交给 VS Code Remote SSH：

```text
vscode-remote://ssh-remote+prod-web-01/home/deploy
```

这样 `ProxyCommand`、密钥、端口、别名这些配置仍然由 OpenSSH 和 Remote SSH 处理。插件只负责把目标找出来，然后把连接动作变得顺手一点。

## 搜索和连接

搜索支持按这些字段过滤：

- Host 别名
- HostName
- User
- 已记录的远程目录

我日常最常用的是按 IP 段或服务名搜索。比如只记得机器在 `10.10.12.x`，直接输入 `10.10.12` 就能缩小范围；如果别名里有 `web`、`api`、`gpu` 这种关键词，也能很快过滤出来。

连接方式目前放在右键菜单里：

- Connect：在当前窗口打开远程目录
- Connect in New Window：新建窗口打开
- Open SSH Terminal：打开集成终端并执行 SSH

当前窗口连接适合快速切过去看一个项目；新窗口连接适合同时开多台机器；终端连接适合只执行几条命令，不想进入完整 Remote SSH 环境的时候。

## 分组、收藏和目录

服务器太多时，只有搜索还不够。

插件里加了几个轻量功能：

- Favorites：常用机器收藏到顶部。
- 自动按子网分组：比如 `192.168.9.x`、`192.168.49.x`。
- 自定义正则分组：可以按机房、环境、业务线分组。
- 记录远程目录：同一台机器上常去的目录可以直接挂在 Host 下面。

这些都不是复杂功能，但每天用的话能少很多重复操作。

## 为什么不做成独立配置

很多工具会让你再维护一份服务器列表。我不太想这么做。

原因很现实：公司服务器配置经常变，跳板机、用户名、密钥、端口都可能调整。如果插件也保存一份连接配置，迟早会和 `ssh/config` 不一致。

所以这个插件只增强入口，不接管配置。你原来怎么写 SSH，还是怎么写；命令行里能 `ssh prod-web-01` 连上，插件里也应该能连上。

## 局限

目前它主要服务普通的 Remote SSH 工作流。

暂时不支持：

- WSL
- Dev Containers

这两个方向当然也有价值，但精力有限，先把我每天最痛的 SSH 服务器搜索和连接问题解决掉。

## 结尾

这个插件不复杂，也不是一个“大而全”的远程开发平台。它只是把我每天都在做的一件小事变快：从一堆服务器里找到目标，然后连上去。

如果你也在 VS Code 或 Cursor 里管理很多 SSH 服务器，可以试试：

- GitHub：[https://github.com/westdefeat/ssh-targets-manager](https://github.com/westdefeat/ssh-targets-manager)
- VS Code Marketplace：[https://marketplace.visualstudio.com/items?itemName=fanxiaolin.ssh-targets-manager](https://marketplace.visualstudio.com/items?itemName=fanxiaolin.ssh-targets-manager)

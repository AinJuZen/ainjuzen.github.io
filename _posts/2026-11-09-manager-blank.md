---
layout: post
title: "Manager 装好了，打开是空的"
date: 2026-11-09 00:15:00 +0800
description: "它默认拉 raw.githubusercontent.com。这个域名在很多网络里连不上。"
tags: [comfyui, manager, 镜像]
---

```
ComfyUI-Manager 已安装 ✓
点开 Model Manager
→ 一片空白
→ 控制台：Failed to get external model list
```

**它默认去 `raw.githubusercontent.com` 拿节点清单。那个域名，在很多网络里是连不上的。**

**不是 Manager 写错了，是这个默认值对一部分人不成立。**

---

## 我一开始写了个「一键修复」脚本

四个步骤，跑完就重启，能用。

**然后我意识到一个问题：Manager 一升级，它就被顶掉了。**

**用户不知道要重跑一次，于是列表又空了，然后他会觉得「这工具坏了」。**

---

## 所以换了个形态：诊断器优先

```
--check     只诊断：Manager 哪一版、哪些地址没走镜像、镜像通不通
--fix       改（幂等，可反复跑）
--verify    只验：真去把清单拉一遍
```

**`--check` 是主角，`--fix` 是配角。**

**为什么？因为「改官方源码」这件事一定会随版本失效。
但即使失效，`--check` 也应该准确地告诉你「我修不了」，而不是静默失败。**

**一个会说「我修不了」的工具有用；
一个假装修好了的工具有害。**

---

## 它到底改什么（四处，都列出来）

| # | 改哪 | 改成什么 |
|---|---|---|
| 1 | `user\__manager\channels.list` | 6 个频道地址加镜像前缀 |
| 2 | `user\__manager\config.ini` | `channel_url` 指向镜像 |
| 3 | `custom_nodes\ComfyUI-Manager` 里 `.py` / `.template` 中写死的地址 | 加镜像前缀 |
| 4 | `glob\manager_downloader.py` | 模型下载默认走 hf-mirror |

**每个改动的文件旁边留一份 `.bak_managerfix`，原样可回滚。**

---

## 两个技术上的坑

**① 幂等不能靠「跑两次结果一样」，要靠正则**

镜像地址里包含原始地址：

```
https://ghfast.top/https://raw.githubusercontent.com/...
                 ↑ 这一段会被下次替换再包一层
```

**不处理的话，第二次跑就变成 `ghfast.top/ghfast.top/https://...`。**

**解决办法是负向后视断言，只在「还没被包过」的位置替换，再加一条折叠正则兜底。**

**② 自动找 ComfyUI 目录比想象中难**

**桌面版把实例放在 `installPath\ComfyUI\`——往下还有一层。**

**第一版只查了一层，结果在真机上「找不到 ComfyUI」。**

**修法是：拿到候选目录后，自己也查一次，再查它的 `ComfyUI` 子目录。**

---

## 一句话

> **会碎不是不发的理由，是要发出一个碎的时候会说实话的版本。**

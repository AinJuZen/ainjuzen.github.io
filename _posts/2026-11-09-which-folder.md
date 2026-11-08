---
layout: post
title: "下了模型，然后站在目录前面发呆"
date: 2026-11-09 00:12:00 +0800
description: "文件名不会告诉你它是什么模型。但 safetensors 头里的张量名会。"
tags: [comfyui, safetensors, 模型]
---

迅雷把文件拽到桌面了。`moss_image_edit.safetensors`，5 GB。

然后呢？

```
checkpoints？vae？loras？diffusion_models？
```

**放错了，ComfyUI 里就看不到它；更糟的是，看到了但加载报错。**

---

## 网上的答案：看文件名

```
带 vae 的放 vae
带 lora 的放 loras
带 clip 的放 text_encoders
```

**新模型根本不在文件名里写这些。**

```
krea2_turbo_fp8_scaled.safetensors     ← 名字里没有任何结构线索
moss_image_edit.safetensors            ← 看着像主干，其实是 LoRA
z_image_turbo_bf16.safetensors         ← 看不出是 DiT
```

**光靠文件名，这三个我一个都放不对。**

---

## 换个思路：读文件内部

`.safetensors` 的头里写着每个张量的名字。**那些名字就是结构的指纹。**

```
encoder.down.* + decoder.up.*        →  VAE
blocks.0.attn.wq/wk/wv + mlp.gate    →  DiT 主干（Krea / Chroma 那类）
layers.0.proj.k.lora_A / lora_B      →  LoRA（哪怕文件名里没有 lora）
model.language_model.*               →  文本编码器（Qwen3-VL 那类）
cap_embedder / noise_refiner         →  DiT（Z-Image 那类）
```

**只读头部，通常几百 KB，不加载权重。**

---

## 12 条规则是这么一条条试出来的

**没有人给我这份表。是我拿本机模型库里每个文件，把它的前几个张量名打出来，一条条对照着写的。**

```
MOSS-Image   → layers.N.proj.*     （没有 model.diffusion_model 前缀）
Krea / Chroma → blocks.N.attn.wq   （没有 double_blocks，也不是 layers）
Qwen3-VL     → model.language_model.layers.*  （比 model.layers 多一层）
```

**每认出一个新结构，就往表里加一条。加完再跑一遍全库，看有没有把别的判错。**

---

## 现在的成绩

```
本机模型库 15 个真实文件
判出        15 / 15
与现有目录一致  14 / 15
```

**那个「不一致」的，是 `moss_image_edit.safetensors`——它现在躺在一个叫 `moss-image-edit\` 的非标准目录里。**

**但它的张量是 `layers.N.proj.k.lora_A/lora_B`，它确实是 LoRA。**

**所以是我的判定对，是那个文件放错了地方。**

---

## 三条我坚持要有的规矩

**它要移动你的文件，所以：**

```
① 绝不覆盖
   同名同大小 → 跳过
   同名不同大小 → 另存 xxx_1.safetensors，原文件不动

② 判不出来就不动
   只说「判断不出来」，不猜、不乱放

③ --dry-run 先看
   报告会放到哪，一个字节都不动
```

**「判不出来」比「猜错了」好一百倍。猜错了用户不会知道，直到加载报错。**

---

## 一句话

> **文件名是别人起的，张量名是模型自己长的。**
> **信模型自己。**

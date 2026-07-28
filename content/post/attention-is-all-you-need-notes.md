---
title: "Attention Is All You Need"
description: "Transformer 奠基之作，改变了 NLP 的论文"
date: 2026-07-26T12:00:00+08:00
draft: false
tags:
  - paper
  - deep learning
  - transformer
  - NLP
categories:
  - 技术
---

## 基本信息

- **标题**：Attention Is All You Need
- **作者**：Ashish Vaswani 等（Google Brain / Google Research）
- **发表**：NeurIPS 2017
- **论文地址**：[https://arxiv.org/abs/1706.03762](https://arxiv.org/abs/1706.03762)

## 背景：为什么需要这篇论文

2017 年之前，序列转录任务（比如机器翻译）的主流方案是 RNN / LSTM，配上 encoder-decoder 架构。RNN 的问题是：计算是串行的，每一个时间步依赖上一个时间步的隐状态，没法并行。长序列上更明显——梯度消失、训练慢。

注意力机制当时已经有人用了，但通常只是作为 RNN 的附属品，比如在 decoder 端对 encoder 的输出做 attention。

这篇论文的核心思路很直接：**把 RNN 扔掉，只用注意力机制。**

## 模型架构

### 整体结构

标准的 encoder-decoder 架构，但 encoder 和 decoder 都由 Transformer 块堆叠而成（base 模型是 6 层）。

### Scaled Dot-Product Attention

核心公式：

```
Attention(Q, K, V) = softmax(QK^T / √d_k) V
```

Q 和 K 做点积，除以 √d_k 做 scaling，softmax 归一化成权重，再对 V 加权求和。

Scaling 这步很关键——维度大了以后点积的值会变大，把 softmax 推到梯度很小的区域，除以 √d_k 拉回来。

### Multi-Head Attention

不只用一组注意力，而是把 Q、K、V 分别线性投影到 h 个子空间（base 模型 h=8），各算一次 attention，然后拼接起来再过一次线性层。

效果是让模型能在不同的表示子空间里关注不同位置的信息。

### Positional Encoding

既然没有 RNN 也没有 CNN，模型天生不知道词的顺序。解决方案是给输入加位置编码。

用了正弦/余弦函数：

```
PE(pos, 2i)   = sin(pos / 10000^{2i/d_model})
PE(pos, 2i+1) = cos(pos / 10000^{2i/d_model})
```

这个设计有个好处：对于任意固定的偏移 k，PE(pos+k) 可以表示为 PE(pos) 的线性函数，模型容易学到相对位置关系。

作者也试过用可学习的位置编码，效果差不多，但正弦版本能外推到更长的序列。

### 其他组件

- **Feed-Forward Network**：每个 attention 子层之后接一个两层全连接网络（ReLU 激活），d_ff = 2048，d_model = 512
- **残差连接 + LayerNorm**：每个子层输出为 LayerNorm(x + Sublayer(x))
- **Dropout**：子层输出、embedding 和位置编码之和上都加了 dropout（rate=0.1）

## Why Self-Attention

作者用一个表格直接对比了三种层的计算复杂度：

| 层类型 | 每层复杂度 | 序列操作数 | 最大路径长度 |
|--------|-----------|-----------|------------|
| Self-Attention | O(n²·d) | O(1) | O(1) |
| Recurrent | O(n·d²) | O(n) | O(n) |
| Convolutional | O(k·n·d²) | O(1) | O(log_k(n)) |

n 是序列长度，d 是表示维度。通常 n < d（比如 n=100, d=512），所以 Self-Attention 的 O(n²·d) 实际不比 RNN 的 O(n·d²) 差，而且最关键的是：**路径长度是 O(1)**，任意两个位置可以直接交互，不存在长距离依赖问题。

## 训练与结果

### 机器翻译

在 WMT 2014 英德翻译上，transformer big 模型 BLEU 28.4，超过之前所有模型（包括 ensemble）2 个点以上。8 块 P100 训了 3.5 天。

在英法翻译上，BLEU 41.8，单模型 SOTA，训练成本不到之前最好模型的 1/4。

### 消融实验

作者做了一组 ablation，核心发现：
- 单头注意力比多头差 ~0.9 BLEU
- 减少 key 的维度（d_k）会明显掉点
- 模型越大效果越好
- Dropout 对防止过拟合很重要
- 正弦位置编码和可学习的版本效果基本一样

### 句法分析

在 Penn Treebank 上用 4 层 transformer 做成分句法分析，WSJ 训练集上 F1 91.3，半监督设置下 92.7，超过了大部分已有模型。证明 transformer 不只对翻译有效，能泛化到其他 NLP 任务。

## 影响

这篇论文的影响力不用多说。它直接催生了：

- **BERT**：双向 transformer encoder，把 NLP 各个任务的 SOTA 刷了一遍
- **GPT 系列**：自回归 transformer decoder，一路从 GPT-1 到 GPT-4
- **Vision Transformer（ViT）**：把 transformer 用到了图像上
- **几乎所有的现代 LLM**：无一例外都是 transformer 架构

可以说 2017 年之后的 NLP 和 LLM 发展史，都是在吃这篇论文的老本。

## 总结

这篇论文最大的贡献不是发明了一个新的机制（attention 之前就有），而是做出了一个**大胆的简化决策**：把 RNN 和 CNN 全部去掉，只用 attention，结果效果更好、训练更快。这种"少即是多"的思路，后来也影响了 ViT 把卷积去掉只用 attention。

论文写作也很干净，实验设计规范，ablation 充分，读起来很舒服。如果你想理解现在所有大模型的基础架构，这篇是绕不过去的必读论文。

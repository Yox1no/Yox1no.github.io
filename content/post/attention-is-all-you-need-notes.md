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

## 背景

在 Transformer 出现之前，序列转录任务（如机器翻译）的主流方案是 RNN/LSTM + Attention。RNN 的核心问题是计算是时序依赖的——每个时间步的隐状态 h_t = f(h_{t-1}, x_t)，必须等上一步算完才能算下一步。这导致训练时无法并行，长序列上尤其严重。

Attention 机制当时已经存在，被用来让 decoder 在生成每个词时，可以"关注" encoder 的不同位置。但 attention 只是 RNN 的一个补充模块，整体架构依然受限于 RNN 的串行计算。

与此同时，也有一些非RNN的尝试，比如用 CNN 做序列建模（ByteNet、ConvS2S），通过卷积层堆叠来捕捉长距离依赖，但路径长度随距离增长（线性或对数），依然不够理想。

这篇论文的核心贡献就是：**抛弃 RNN 和 CNN，只用 attention 机制构建一个完整的序列转录模型。**

## 模型架构

### 整体结构

标准的 encoder-decoder 架构。Encoder 和 decoder 都由 N=6 个相同的层堆叠而成。每个 encoder 层包含两个子层：multi-head self-attention 和 feed-forward network。Decoder 层在中间多插了一个 encoder-decoder attention 子层，用于关注 encoder 的输出。

每个子层都加了残差连接和 layer normalization，即输出为 LayerNorm(x + Sublayer(x))。所有子层和 embedding 层的输出维度都是 d_model = 512。

### Scaled Dot-Product Attention

这是最核心的计算单元。给定 query、key、value，输出是 value 的加权和，权重由 query 和 key 的兼容度决定：

```
Attention(Q, K, V) = softmax(Q K^T / √d_k) V
```

具体来说：Q 和 K 做点积，得到每个 query 对所有 key 的注意力分数。除以 √d_k 做 scaling——这是因为 d_k 较大时点积的值会变大，把 softmax 推到梯度极小的区域，缩放后能保持梯度的稳定。然后 softmax 归一化得到权重，最后对 V 加权求和。

实验中用的 d_k = 64，√d_k = 8，刚好把点积的方差拉回 1 左右。

### Multi-Head Attention

与其用单个 attention 函数，作者把 Q、K、V 分别线性投影到 h=8 个低维子空间（d_k = d_v = d_model/h = 64），在每个子空间独立算 attention，然后把结果拼接起来再过一次线性层：

```
MultiHead(Q, K, V) = Concat(head_1, ..., head_h) W^O
head_i = Attention(Q W_i^Q, K W_i^K, V W_i^V)
```

这么做的好处是模型可以在不同的子空间里学到不同类型的注意力模式——有的 head 关注语法关系，有的关注长距离依赖，有的关注指代消解。

### 三种 Attention 用法

1. **Encoder self-attention**：每个位置可以关注 encoder 前一层的所有位置
2. **Decoder masked self-attention**：每个位置只能关注它之前的位置（包括自己），通过 mask 禁止看到未来信息，保证自回归生成
3. **Encoder-decoder attention**：decoder 的 query 关注 encoder 的输出，类似传统 seq2seq 的 attention

### Positional Encoding

Self-attention 本身是置换不变的——改变输入顺序，输出也会跟着置换，模型感知不到词的先后顺序。所以需要显式注入位置信息。

作者用了正弦/余弦函数生成位置编码：

```
PE(pos, 2i)   = sin(pos / 10000^{2i/d_model})
PE(pos, 2i+1) = cos(pos / 10000^{2i/d_model})
```

这个设计的精妙之处在于：
- 每个位置的编码是唯一的
- 不同维度对应不同的频率，低维变化快（编码局部位置），高维变化慢（编码全局位置）
- 对于任意偏移 k，PE(pos+k) 可以用 PE(pos) 的线性变换表示，方便模型学习相对位置

作者也试过用可学习的位置嵌入，效果几乎一样，但正弦版本能泛化到训练时没见过的序列长度。

### Position-wise FFN

每个 attention 子层后面接一个两层全连接网络，d_ff = 2048，d_model = 512：

```
FFN(x) = max(0, x W_1 + b_1) W_2 + b_2
```

隐层用 ReLU 激活，相当于对每个位置独立做两个线性变换，中间扩维再降维。从卷积角度看，就是两层 kernel size=1 的卷积。

### 其他训练技巧

- **Dropout**：rate=0.1，用在每个子层输出上、embedding 和位置编码的加和上
- **Label Smoothing**：ε_ls=0.1，让模型不那么自信，虽然困惑度会变差但 BLEU 更高

## Why Self-Attention

作者从三个角度对比了 self-attention、recurrent 和 convolutional 三种层：

| 层类型 | 每层复杂度 | 序列操作数 | 最大路径长度 |
|--------|-----------|-----------|------------|
| Self-Attention | O(n²·d) | O(1) | O(1) |
| Recurrent | O(n·d²) | O(n) | O(n) |
| Convolutional | O(k·n·d²) | O(1) | O(log_k(n)) |

n 是序列长度，d 是表示维度。实践中 n 远小于 d（句子通常几十到一百词，d=512），所以 self-attention 的 O(n²·d) 实际算起来并不比 RNN 的 O(n·d²) 差。而且 self-attention 的最大路径长度是 O(1)，任意两个位置的词可以直接交互，不存在长距离依赖问题。RNN 需要 O(n) 步才能把信息传过去，CNN 也需要 O(log_k(n)) 层。

## 训练与结果

### 机器翻译

| 模型 | EN-DE BLEU | EN-FR BLEU | 训练成本 |
|------|-----------|-----------|---------|
| Transformer (base) | 27.3 | 38.1 | 3.3 × 10^18 FLOPs |
| Transformer (big) | 28.4 | 41.8 | 2.3 × 10^19 FLOPs |
| 之前最佳 (ensemble) | 26.36 | 41.29 | ~10^20 FLOPs |

在英德翻译上，big 模型 BLEU 28.4，超过之前所有模型（包括 ensemble）2 个 BLEU。在英法翻译上，BLEU 41.8，单模型 SOTA，训练成本不到之前最佳模型的 1/4，8 块 P100 训了 3.5 天。

### 消融实验

作者对关键设计做了系统的消融实验：

- **注意力头数**：单头比最优设置差约 0.9 BLEU，头数太多也不行（16 头开始掉）
- **key 维度**：减小 d_k 会明显掉点，说明计算 query-key 兼容性并不容易
- **模型大小**：更大的 d_model、d_ff 都有提升
- **Dropout**：不加 dropout 会严重过拟合，BLEU 从 25.8 掉到 24.6
- **Label Smoothing**：确实有帮助
- **位置编码**：正弦 vs 可学习，效果几乎一样

### 句法分析

在 Penn Treebank 上用 4 层 transformer 做成分句法分析，WSJ 训练集 F1 91.3，半监督 92.7，超过大部分已有模型。说明 transformer 不只适用于翻译，能泛化到其他 NLP 任务。

## 影响

这篇论文的影响怎么强调都不过分。它是现代 LLM 的基石：

- **BERT（2018）**：双向 transformer encoder，在 11 个 NLP 任务上刷 SOTA
- **GPT 系列**：自回归 transformer decoder，从 GPT-1 到 GPT-4，一路 scaling
- **ViT（2020）**：把 transformer 用到图像分类，证明 attention 可以替代卷积
- **后续几乎所有 LLM**：无一例外都基于 transformer 架构

可以说 2017 年之后的 NLP 发展史，很大程度上是在这篇论文划定的框架内迭代。

## 几个有意思的点

- 论文标题"Attention Is All You Need"刚好 5 个词，和作者人数一致，可能是故意的
- 作者贡献说明里提到"Listing order is random"，这在 ML 论文里不多见
- 消融实验中只用了一个开发集（newstest2013）做调参，没有反复在测试集上试，实验规范

## 总结

Attention Is All You Need 最大的贡献不是发明了 attention（之前就有），而是做了一个大胆的简化：**把 RNN 和 CNN 全部去掉，只用 attention 搭了整个模型**，结果效果更好、训练更快。这种"少即是多"的思路后来也启发了 ViT 把卷积去掉只用 attention。

如果你想理解现在所有大模型的基础架构，这篇论文是绕不过去的必读文献。

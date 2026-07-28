---
title: "Attention Is All You Need"
description: "Transformer 奠基之作，改变了 NLP 的论文"
date: 2026-07-26T12:00:00+08:00
draft: false
image: featured.jpg
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

在 2017 年之前，处理序列数据——特别是机器翻译这类 seq2seq 任务——的主流方法是 RNN/LSTM + Attention。

RNN 的工作方式是逐时间步串行计算的：隐状态 h_t = f(h_{t-1}, x_t)，每个时间步的计算依赖上一步的结果。这种"串行"是 RNN 的根本弱点：

**训练慢**：一个序列中的每个词必须排队处理，无法并行。对长序列来说，即使 batch 内其他样本算完了，也得等最长的那个。GPU 的大规模并行能力被浪费了。

**长距离依赖难搞**：信息要从序列开头传到末尾，理论上需要经过所有中间时间步。虽然 LSTM 引入了门控机制缓解了梯度消失，但路径长度依然是 O(n)，远距离依赖依然不稳定。

**Attention 的出现**：2014-2015 年，Bahdanau 等人把注意力机制引入 seq2seq 模型。Decoder 在生成每个目标词时，不再只依赖 encoder 的最后一个隐状态，而是"注意" encoder 所有位置的输出，动态加权。这大大改善了翻译质量，但 attention 当时只是 RNN 上的一个附加模块。

**CNN 的尝试**：也有人试图用卷积替代 RNN。ConvS2S 在 encoder 端用 CNN 做并行计算，但 CNN 捕捉远距离依赖需要堆多层，路径长度随距离对数增长。ByteNet 用扩张卷积缓解了这个问题，但依然不够直接。

这篇论文做出了一个看似激进的决策：**把 RNN 和 CNN 全部砍掉，整个模型就靠 attention 机制**。单看想法好像很大胆，但论文的实验证明：效果确实更好，而且训得快得多。

## 模型架构

### 整体结构

Transformer 依然是经典的 encoder-decoder 结构，但 encoder 和 decoder 内部不再有 RNN。

Encoder 由 N=6 个相同的层堆叠。每层包含两个子层：
1. Multi-Head Self-Attention
2. Position-wise Feed-Forward Network

Decoder 同样 N=6 层，但每层有三个子层。前两个和 encoder 一样，中间多插了一个：
- Masked Multi-Head Self-Attention（只看左边）
- Multi-Head Cross-Attention（看 encoder 的输出）
- Position-wise Feed-Forward Network

所有子层都使用了残差连接（Residual Connection，来自 ResNet）和层归一化（Layer Normalization）。输出形式统一为：

```
LayerNorm(x + Sublayer(x))
```

这样做有两个好处：残差连接让梯度无阻碍地反向传播到深层，层归一化稳定训练。所有输入输出维度统一为 d_model = 512。

### Scaled Dot-Product Attention

这是 Transformer 最核心的计算单元，论文把它定义为：

```
Attention(Q, K, V) = softmax(Q K^T / √d_k) V
```

理解这个公式可以拆成三步：

1. **计算注意力分数**：Q 和 K 的转置相乘，得到一个 n×n 的矩阵，第 i 行第 j 列表示第 i 个 query 和第 j 个 key 的"相似度"。本质上就是拿 Q 去查 K，看哪些位置跟自己相关。

2. **Scaling**：除以 √d_k（d_k 是 key 的维度，默认为 64）。为什么要除？直观理解：当 d_k 较大时，QK^T 的值域很大，softmax 的输出会呈现出极端的"硬"分布——只有一两个位置得分接近 1，其余接近 0。这意味着梯度过小，模型很难学习。除以 √d_k 能把分布拉回到更适合 softmax 的区间。

   有更严谨的解释：假设 q 和 k 的每个分量是均值 0、方差 1 的独立随机变量，那么它们的点积 q·k 的方差就是 d_k。用这个值缩放后，点积的方差回到 1。

3. **加权**：softmax 归一化得到注意力权重（每行和为 1），再和 V 相乘。输出是 V 的加权求和——模型根据 Q 和 K 的匹配程度，决定"该看 V 的哪些位置"。

在 Transformer 里，self-attention 的 Q、K、V 来自同一个输入（所以叫"自"注意力），而 cross-attention 的 Q 来自 decoder，K 和 V 来自 encoder 的输出。

### Multi-Head Attention

与其用一组大的 Q、K、V 算一次 attention，不如把它们分割成多个"头"：

```
MultiHead(Q, K, V) = Concat(head_1, ..., head_h) W^O
head_i = Attention(Q W_i^Q, K W_i^K, V W_i^V)
```

具体来说：把 d_model=512 的 Q、K、V 分别通过线性变换投影到 h=8 个 d_k=64 的低维子空间。每个子空间独立算一次 scaled dot-product attention，然后把 8 个头的输出拼回 d_model=512，最后再过一次线性层。

一个直观的类比：每个 head 就像是模型的一双"眼睛"，各自关注输入的不同方面。有的 head 看语法结构、有的看长距离依赖、有的看指代消解。把多头的结果拼接起来，相当于从不同视角理解序列。即使某些 head 关注到了噪声，只要有一个 head 学到了有用的模式，模型就能用。

实验证明，单头比多头差约 0.9 BLEU，但头数超过 8 以后收益递减。

### 三种 Attention 用法

Transformer 的 attention 在三个地方被用到，各有不同的角色：

**Encoder Self-Attention**：每个词关注整个输入序列的所有词（包括自己）。比如翻译"The animal didn't cross the street because it was too tired"时，"it"可以关注到"animal"，捕捉这种远距离的指代关系。没有距离限制，没有路径衰减。

**Decoder Masked Self-Attention**：decoder 在生成时每个位置只能看它之前的词。为什么？因为推理时模型必须自回归地逐词生成——t 时刻的输出只应该依赖 <t 时刻的已知信息。Mask 的实现是在 softmax 之前把未来的位置设成 -∞，softmax 之后分数自然就是 0。

**Encoder-Decoder Cross-Attention**：decoder 的 Q 来自当前层，K 和 V 来自 encoder 的输出。这相当于 decoder 在生成每个目标词时，向 encoder 的所有源词"查询"信息。和传统 seq2seq 里的 attention 类似，区别在于这里用的也是 multi-head scaled dot-product attention。

### Positional Encoding

Self-attention 是**置换不变的**——调换输入序列里两个词的位置，输出也只会相应地调换。这意味着模型天然感知不到词的先后顺序。

为了解决这个问题，作者在 embedding 之后、进入 encoder/decoder 之前，把位置编码和输入 embedding 直接相加。位置编码维度也是 d_model=512，这样两个向量可以直接做 element-wise add。

位置编码使用正弦/余弦函数生成：

```
PE(pos, 2i)   = sin(pos / 10000^{2i/d_model})
PE(pos, 2i+1) = cos(pos / 10000^{2i/d_model})
```

pos 是词在序列中的位置，i 是编码向量的维度索引。这个公式有几个精妙之处：

- **唯一性**：每个位置的编码向量是唯一的，不同位置不会混淆
- **多尺度**：低维度的正弦波变化快，编码"邻居关系"；高维度变化慢，编码"全局位置"
- **相对位置感知**：因为 sin 和 cos 的线性性质，对于任意固定偏移 k，PE(pos+k) 能被表示为 PE(pos) 的线性变换。这意味着模型不需要显式存储"位置 3 和位置 7 的关系"，就能通过可学习的线性层自然捕捉

论文也试过直接学习一个位置嵌入矩阵（类似 word embedding），效果和正弦编码差不多。但正弦编码有个额外优势：理论上可以处理比训练时更长的序列（尽管实际效果取决于频率的覆盖范围）。

### Position-wise Feed-Forward Network

每个 token 位置在通过 attention 之后，再独立过一遍两层全连接：

```
FFN(x) = max(0, x W_1 + b_1) W_2 + b_2
```

隐层维度 d_ff = 2048，比 d_model = 512 大得多，先扩维再降维。从实现角度看，等价于两层 kernel_size=1 的卷积。

为什么需要 FFN？Attention 只是在 tokens 之间"通信"，把不同位置的信息融入每个 token。但融入之后，每个 token 还需要进行一些非线性变换来整合这些信息——这就是 FFN 的作用。某种意义上，attention 负责"交换信息"，FFN 负责"消化信息"。

### 其他训练技巧

- **Dropout**：rate=0.1，位置包括：每个子层的输出（残差加之前）、embedding 和位置编码的加和
- **Label Smoothing**：ε_ls=0.1。训练时不给 one-hot 标签，而是用平滑后的分布，比如"正确 token 概率 0.9，其余平分 0.1"。模型会变得不那么自信，困惑度可能更高，但 BLEU 分数更好

## Why Self-Attention

论文专门用一节给出了选择 self-attention 而不是 RNN 或 CNN 的理论论证，从三个维度来比较：

| 层类型 | 每层计算复杂度 | 最少顺序操作 | 最大路径长度 |
|--------|----------|---------|---------|
| Self-Attention | O(n²·d) | O(1) | O(1) |
| Recurrent | O(n·d²) | O(n) | O(n) |
| Convolutional | O(k·n·d²) | O(1) | O(log_k(n)) |
| Self-Attention (局部) | O(r·n·d) | O(1) | O(n/r) |

n 是序列长度，d 是表示维度，k 是卷积核宽度。

**复杂度**：看起来 self-attention 的 O(n²·d) 比 RNN 的 O(n·d²) "大"，但在 NLP 实践中 n（句子长度，一般几十到几百词）远小于 d（512），所以实际计算量 self-attention 并不吃亏。

**顺序操作**：这是最关键的指标。RNN 需要 O(n) 步才能处理完一个序列——第 n 个词必须等前面 n-1 个词都算完。Self-attention 只需要 O(1)——所有位置的矩阵乘法可以一步完成。这正是为什么 Transformer 训练比 RNN 快那么多的原因。

**路径长度**：任意两个词之间传递信息的"最短步数"。Self-attention 让任意两词直接相连（O(1)），RNN 要走 n 步，CNN 需要堆 log_k(n) 层。对长距离依赖来说，这个差异巨大。

## 训练细节

Backbone 设置：
- **Base 模型**：N=6, d_model=512, d_ff=2048, h=8, 总参数 65M
- **Big 模型**：N=6, d_model=1024, d_ff=4096, h=16, 总参数 213M

训练细节：
- **数据**：WMT 2014 英德约 4.5M 句子对，英法约 36M
- **分词**：字节对编码（BPE），共享源-目标词表约 37K
- **硬件**：8 × NVIDIA P100 GPU
- **优化器**：Adam，β_1=0.9, β_2=0.98, ε=10^-9
- **学习率**：先 warmup 4000 步，warmup 期间从 0 线性升到峰值，之后按 1/√step 衰减。公式：lr = d_model^{-0.5} · min(step^{-0.5}, step · warmup^{-1.5})
- **批次**：每批约 25K 源词 + 25K 目标词
- **Base 训练**：10 万步，每步约 0.4 秒，12 小时
- **Big 训练**：30 万步，3.5 天

这里 learning rate schedule 的设计值得留意：warmup 是逐步提升学习率，避免训练初期梯度不稳定。之后按 1/√step 衰减——不是常见的线性衰减。这个 schedule 后来被广泛采纳，成为训练 Transformer 的标配。

## 结果

### 机器翻译

| 模型 | EN-DE BLEU | EN-FR BLEU | 训练成本 (FLOPs) |
|------|-----------|-----------|-----------------|
| ConvS2S | 25.16 | 40.46 | 9.6 × 10^18 |
| Transformer (base) | 27.3 | 38.1 | 3.3 × 10^18 |
| Transformer (big) | 28.4 | 41.8 | 2.3 × 10^19 |
| 之前最佳 Ensemble | 26.36 | 41.29 | ~10^20 |

英德翻译上 big 模型 28.4 BLEU，超过所有已发表模型（包括 ensemble）2 个 BLEU 以上。英法翻译 41.8，训练成本只有之前 SOTA 的 1/4。

### 消融实验

作者基于英德翻译的开发集做了一系列消融，验证每个设计选择的有效性：

- **注意力头数**：单头 BLEU 24.9，比最优（8 头，25.8）差 0.9。头数增加到 16 后持平、32 后微降
- **Key 维度 d_k**：从 64 减到 16 掉 0.4 BLEU，说明计算 query-key 兼容性确实需要一定的表示空间
- **模型大小**：d_model 从 512 提到 1024，BLEU 从 25.8 涨到 26.0；d_ff 从 2048 提到 4096，涨到 26.2
- **Dropout**：不加 dropout（rate=0），BLEU 从 25.8 掉到 24.6，过拟合明显
- **位置编码**：用可学习嵌入替代正弦编码，BLEU 25.7，几乎一样

### 句法分析（成分句法分析）

在 Penn Treebank 上测试，4 层 transformer 在纯 WSJ 训练集上 F1 91.3，半监督 92.7，超过大部分 RNN-based parser。证明 transformer 不是翻译特化方案，而是通用的序列建模架构。

## 影响与后续工作

这篇论文直接开启了 NLP 的 Transformer 时代，衍生出一系列里程碑式工作：

**BERT（2018）**：用 transformer encoder 做双向预训练，NSP + MLM 两个目标任务，在 11 个 NLP 基准上刷新 SOTA。核心思路是——transformer 不仅能做翻译，还能做一个通用的文本表示模型。

**GPT 系列**：OpenAI 从 GPT-1 开始就用 transformer decoder 做单向语言模型预训练，一路 scaling 到 GPT-4。GPT 系列的核心信念是：只要模型和数据足够大，transformer 就能涌现出各种能力。

**ViT（2020）**："An Image is Worth 16x16 Words"，把图片切成 patch 当 token 输入 transformer，证明在足够多的数据下，transformer 在视觉任务上也能超越 CNN。

**后续 LLM**：Llama、Claude、Gemini、DeepSeek 等等，无一例外都基于 transformer 架构。这篇论文是整个现代 LLM 技术栈的基石。

## 总结

Attention Is All You Need 最大的贡献不是发明了一个新机制（attention 和 self-attention 都早就有），而是做出了一个大尺度的简化：**把 RNN 和 CNN 都砍掉，全用 attention 搭一整个 seq2seq 模型**。结果不仅没有变差，反而在翻译质量、训练速度、可解释性上全面超越了旧架构。

当时看来是大胆的赌注，现在看是天才的工程决策。

如果你想深入理解现在所有 LLM 的底层架构，这篇论文是起点中的起点。

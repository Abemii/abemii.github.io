---
author: "im"
title: "[Reading] Training data-efficient image transformers & distillation through attention"
date: "2022-02-19"
tags: [
    "ViT",
    "Transformer",
    "Deep Learning",
    "Computer Vision"
]
---

DeiT 論文を読んだのでそのメモ。

多くの ViT 研究において、 DeiT の学習スキームがフォローされている。
最近読んだ ShiftViT[^shiftvit] において言及されており、ちゃんと読んでおこうと思っていた。
[^shiftvit]: https://arxiv.org/pdf/2201.10801.pdf


## 書誌情報

```bib
@misc{touvron2021training,
      title={Training data-efficient image transformers & distillation through attention}, 
      author={Hugo Touvron and Matthieu Cord and Matthijs Douze and Francisco Massa and Alexandre Sablayrolles and Hervé Jégou},
      year={2021},
      eprint={2012.12877},
      archivePrefix={arXiv},
      primaryClass={cs.CV}
}
```

- 図表は特に言及のない限り本論文からの引用。

## 概要

- ViT と同じネットワーク構造のモデルを用い、学習方法を工夫することで精度を上げ、
また、学習に必要な計算リソース・時間を大きく短縮した。
- __Distillation token__ を用いた蒸留手法を提案。 class token とは別の token を用いて
教師モデルからの教師信号を学習させることで、より性能を高められる。
- 学習時の工夫について詳細な分析を行い、よくまとめている。

## Distillation token

- Distillation (蒸留) の方法は、 Soft と Hard に大別される。
    - Soft なほうは、生徒モデルの logits $ Z_s $ と 教師モデルの logits $ Z_t $ を KL-loss を使って最小化する。
    - Hard なほうでは、 教師モデルの hard decision $ y_t = \argmax_c Z_t(c) $ を使い、 Cross entropy で最適化する。
        - この方法は Soft と異なりパラメータフリーであるという良さがある。
        - 実験では、更に、 label smoothing と呼ばれる手法を取り入れている。
        すなわち、 真なクラスを $ 1 - \epsilon $ とし、その他のクラスを $\epsilon$ で等分するといった方法である。

- Distillation token
    - class token とは別に、 distillation token を加える。
    - distillation token も class token と同様に使うが、 class token と異なり、
    こちらは教師モデルの hard decision を教師信号とする。
    - class token と distillation token は同じような出力をするが、全く同じではない。

- 実験結果から、
    - distillation token なしの場合、no/soft/hard distillation のうち hard が最も良かった。
    - distillation token あり, hard distillation の場合、クラスの予測に使うものとして、
    class embedding/distil. embedding/両方の3種類がある。
    - このうち最も結果がよかったのは、 distil. embedding を使うもので、
    これは、教師モデルである convnet の帰納バイアスの恩恵を受けているからだろうと著者は考えている。

{{< resize-image src="figures/fig2.png" >}}

## 学習戦略に関する ablation study

- 重みの初期化
    - 切断正規分布を用いた。


### data augmentation

- Transformer では convnets に比べより多くのデータ量が必要。
- Auto-augment, rand-augment により性能が向上。
- 画像領域において、 random erasing を使用。

### 正則化と最適化

- オリジナルの ViT では学習率は 0.03 で固定であったが、交差検証の結果、
Deit では $ \mathrm{lr}_{\mathrm{scaled}} = \frac{\mathrm{lr}}{512} \times \mathrm{batchsize} $ を採用。 
- 最適化アルゴリズムとして ViT と同様 AdamW を用いるが、
Weight Decay の値はより小さい値を用いることとした。
- Stochastic depth
    - 特に深い transformer の収束を促進させる。
- Mixup, Cutmix
- repeated augmentation

### Exponential Moving Average (EMA)

- EMA を行うことで、0.1 pt 程度性能が向上したが、 fine tuning すると、同じ結果となった。

### 異なる解像度での fine tuning

- FixefficientNet と同じスケジュール、正則化、最適化を行ったが、学習時の augmentation は FixefficientNet と異なり、
damper せずに keep する。
- Positional embeddings の補間: bilinear ではなく、 bicubic を用いる。
    - bilinear を用いると、その補間された verctor の l2 ノルムが 隣接する vector のそれよりも小さくなってしまう。
    - これを fine-tuning せずに Transformer に適用すると、性能が著しく落ちる。
    - bicubic を用いると、vector の norm が維持される。
- 学習は解像度 224 で行い、 fine-tuning は 384 で行った。

### 学習時間

- 300 epochs, 37 hours with 2 nodes, 53 hours with single node for DeiT-B.
    - RegNetY-16GF の場合は、これより 20 % 遅い。
- 更に、 Fine-tuning を行うと、 20 epochs, 20 hours with single node for FixDeiT-B.
- batch norm を使っていないので、 batch size を減らしても性能に影響しない。
よって、大きなモデルを学習するのが簡単になる。


{{< resize-image src="figures/tab9.png" >}}


{{< resize-image src="figures/tab8.png" >}}



## 参照

- https://zenn.dev/takoroy/scraps/ced7059a36d846
    - takoroy さんによる本論文のメモ。

- [20] T. He, Z. Zhang, H. Zhang, Z. Zhang, J. Xie, and M. Li (2019) Bag of tricks for image classification with convolutional neural networks. In Conference on Computer Vision and Pattern Recognition, Cited by: §2, §6. 

---
layout: post
title: "[Machine Translation] Attention Is All You Need(Transformer)"
author: kjy
date: 2023-09-07 13:23:00 +09:00
categories: [Paper Review, NLP]
tags: [Paper Review, NLP, Machine Translation]
comments: true
toc: true
math: true
---

해당 포스트에서는 [Attention Is All You Need(Transformer)](https://arxiv.org/pdf/1706.03762.pdf) 논문 리뷰를 진행해보겠습니다.

---

## Abstract

해당 논문은 이전 연구들과 달리 오직 attention이라는 메커니즘만으로 machine translation task를 수행한 연구입니다. 오직 attention 메커니즘만을 사용한 모델을 transformer라고 해당 논문에서는 부르는데 이러한 transformer 모델이 두 가지 machine translation에서 좋은 성능과 더 빠른 처리 속도를 달성했습니다.

해당 포스트에서는 자세한 학습 방법보다는 모델의 구조와 데이터가 어떻게 들어가는지 살폅보겠습니다. 자세한 학습 hyperparameter가 궁금하신 분들께서는 논문을 직접 참고해주시면 좋을 것 같습니다.

## Transformer Model 구조

Transformer model의 구조를 하나씩 알아보겠습니다.

### Encoder-Decoder Stacks

Transformer model의 큰 구조는 encoder-decoder 구조입니다. Encoder Decoder 구조를 처음 machine translation에 가져온 [Seq2Seq](https://jjjuuuun.github.io/posts/Seq2Seq/)에서의 구조와 달리 encoder(decoder) layer를 여러개 쌓아 하나의 encoder(decoder) block을 만듭니다.

Transformer는 동일한 구조의 encoder layer 6개와 동일한 구조의 decoder layer 6개로 구성되어 있으며 이를 간단히 그려보면 아래와 같이 그릴 수 있습니다.

![](../../assets/img/transformer/transformer_1.png){: class="align-center"}

🤔 위의 그림에서 encoder에서 decoder 방향으로 화살표가 그려져 있는데 이것은 뒤에서 설명드리도록 하겠습니다. 우선 큰 틀에서 encoder와 decoder의 구조가 저렇게 생겼다는 것만 알고 넘어가시면 될 것 같습니다.

### Data

해당 모델에서는 두 가지의 datasets에 대한 성능을 평가했습니다. 각 datasets의 구성은 아래와 같습니다.

- WMT 2014 English-German dataset

  - 450만 개의 문장
  - [BPE(Byte-Pair Encoding)](https://jjjuuuun.github.io/posts/BPE/) subtokenizer를 사용해 32,000개의 token으로 dataset을 구성

- WMT 2014 English-French dataset

  - 3600만 개의 문장
  - [WPM(WordPiece Model)](https://jjjuuuun.github.io/posts/WordPiece/) subtokenizer를 사용해 37,000개의 token으로 dataset을 구성

Transformer model의 구조를 설명하기 위해서 `Attention Is All You Need`라는 영어 문장을 `Aufmerksamkeit ist alles, was Sie brauchen`라는 독일어 문장으로 기계 번역을 하는 상황을 예시로 사용하겠습니다. 또한 위에서 말씀드린 tokenizer를 직접 적용하여 설명하기에는 무리가 있어 공백을 기준으로 pre-tokenize했다는 가정하에 설명을 드리도록 하겠습니다.

Datasets을 준비하고 예시 문장을 tokenize한 결과를 아래와 같이 그릴 수 있습니다. 이때 $n$과 $n'$은 각각 encoder와 decoder의 maximum sequence length 입니다.

![](../../assets/img/transformer/transformer_2.png){: class="align-center"}

🔎 위의 사진에서 <u>Decoder Input</u>의 경우 오른쪽에서 하나씩 token이 이동되어 있는 것을 볼 수 있는데 논문에서는 이것을 <u>shifted right</u>라고 표현하고 있습니다.

### Embedding

Transformer에서는 $d_{model} = 512$으로 embedding을 진행하고 있습니다. $512$차원을 그림으로 그리기에는 무리가 있으므로 아래의 그림과 같이 간단하게 표현하도록 하겠습니다.

![](../../assets/img/transformer/transformer_3.png){: class="align-center"}

🔎 Embedding을 진행하는 과정은 encoder와 decoder 모두 같습니다.
🔎 Embedding layer에 $\sqrt{d_{model}}$을 곱합니다.

### Positional Encoding

위에서 말씀드린 것처럼 transformer는 오직 attention 메커니즘으로만 구성한 첫 모델입니다. Transformer를 이렇게 구성하면서 단어들의 위치를 모델에게 알려주는 방법을 저자들은 고안했어야 했습니다.

이렇게 단어들의 위치를 모델에게 알려주는 방법으로는 positional embedding과 positional encoding이 있습니다. Positional embedding 같은 경우에는 학습이 가능하지만 그만큼 weight가 늘어나는 문제와 training에서 만나지 못한 길이의 문장이 inference할 때 들어온다면 대응을 하지 못하는 단점이 존재합니다. 반대로 positional encoding 같은 경우에는 학습을 하지 않아도 되므로 training에서보다 긴 문장이 inference할 때 들어오더라도 대응이 충분히 가능합니다.

이렇게만 생각을 해보면 무조건 positional encoding을 사용해야 할 것 같지만 만약 성능적인 측면에서 positional embedding이 월등히 좋다면 psotional embedding을 사용해야 합니다. 그러나 해당 논문에서 실험을 해 본 결과 두 방법의 성능 차이가 별로 없어서 저자들은 positional encoding을 사용했습니다.

Positional encoding에 사용하는 함수로 저자들은 sinusoid를 사용했습니다.

$$
PE_{pos, i} = \begin{cases} sin(\frac{pos}{10000^{i/d_{model}}}), &  \mbox{if }i\mbox{ is even} \\ cos(\frac{pos}{10000^{i/d_{model}}}), & \mbox{if }i\mbox{ is odd}\end{cases}
$$

🔎 Sinusoid는 주기함수인데 저자들이 주기함수를 사용한 이유는 $PE_{pos+k}$를 $PE_{pos}$로 나타낼 수 있기 때문에 모델이 상대적인 위치를 잘 배울 수 있다고 가정했기 때문이라고 말합니다.

또한 고정된 차원($d_{model}$)로 encoding을 하기 때문에 이것을 embdding한 결과에 더하여 모델에게 input으로 넣어줄 수 있습니다. 그 결과는 아래 그림과 같이 그릴 수 있습니다.

![](../../assets/img/transformer/transformer_4.png){: class="align-center"}

🔎 Positional encoding을 진행하는 과정은 encoder와 decoder 모두 같습니다.

### Multi-Head Attention

#### Query, Key, Value

Query, key, value를 묶어서 생각해보면 DataBase 분야에서 많이 사용되는 용어입니다. 그래서 저는 DB와 연관지어서 transformer에서 사용되는 query, key, value를 이해하고자 했습니다. 그 결과는 아래와 같습니다.

|       | DB                                                       | Transformer                                              |
| :---- | :------------------------------------------------------- | :------------------------------------------------------- |
| Query | DB에 특정 정보를 요청                                    | 해당 단어를 어떻게 표현하면 될까요?                      |
| Key   | DB에 저장되어 있는 정보를 유일하게 구별할 수 있는 식별자 | 요청된(Query) 단어와 관련있는 단어를 찾을 수 있는 식별자 |
| Value | DB에 실제 저장되어 있는 정보                             | 실제 관련이 있는 단어                                    |

Transformer에서 query, key, value는 linear projection을 통해 구합니다. 물론 linear projection을 하지 않고 **embdding + positional encoding**을 한 결과를 바로 query, key, value로 사용가능하지만 저자들은 query, key, value를 각각 $d_k, d_k, d_v$로 linear projection하는 것이 유익하다는 것을 발견했습니다.

Query, Key, Value를 구하는 과정을 그림으로 나타내면 아래와 같습니다.

![](../../assets/img/transformer/transformer_5.png){: class="align-center"}

#### SDPA(Scaled Dot-Product Attention)

Attention을 구함에 있어서 사용되는 함수는 일반적으로 2가지 정도가 있습니다. 바로 additive attention과 dot-product attention입니다.

이 두 방법은 복잡성 측면에서는 유사하지만 `numpy`와 같은 matrix multiplication code를 사용할 수 있기 때문에 dot-product attention function이 훨씬 빠르고 공간이 효율적이기 때문에 transformer에서는 dot-product attention을 사용했습니다.

추가적으로 dot-product attention function에 $\sqrt{d_k}$로 scaling을 하게 되는데 그 이유는 $d_k$가 너무 큰 경우 이후 $\text{softmax}$의 gradient를 굉장히 작게 만들기 때문에 dot-product attention보다 additive attention의 성능이 더 좋아지기 때문이라고 합니다.

위에 내용들을 종합했을 때 transformer에서 사용되는 attention은 scaled dot-product attention이고 구하는 공식은 아래와 같습니다.

$$ \text{Attention}(Q, K, V) = \text{softmax}(\frac{QK^T}{\sqrt{d_k}})V $$

이 과정을 그림으로 나타내면 아래와 같습니다.

![](../../assets/img/transformer/transformer_6.png){: class="align-center"}

또한 해당 파트에서 마스킹 처리를 추가적으로 진행합니다. 아래와 같은 이유로 마스킹 처리를 합니다.

- Encoder
  - 길이가 짧은 문장에서 남은 길이에 대한 처리
  - 사용하지 않은 token 처리
- Decoder
  - 길이가 짧은 문장에서 남은 길이에 대한 처리
  - 사용하지 않은 token 처리
  - 다음 단어를 예측하도록 하기 위해 이전 단어들과 현재 단어를 제외한 나머지 단어들 처리

만약 마스킹 처리를 하게 된다면 SDPA 과정은 아래와 같으며 아래의 그림은 decoder에서 다음 단어를 예측하도록 하기 위해 현재 단어 기준 오른쪽 단어들을 모두 마스킹 처리한 경우입니다.

![](../../assets/img/transformer/transformer_7.png){: class="align-center"}

마스킹 처리는 위의 그림과 같이 $\text{softmax}$값을 처리하기 전에 하게 되는데 이때 마스킹 된 부분을 $-\infty$로 하게 되면 $\text{softmax}$의 값이 0이 되므로 모델이 무시하게 됩니다.

위의 과정들을 논문에서는 아래와 같이 간단하게 표현하고 있습니다.

![](../../assets/img/transformer/transformer_8.png){: class="align-center"}

#### Multi-Head Attention

SDPA의 결과를 Single-Head attention이라하며 여러 개의 SDPA를 조합한 것이 Multi-Head Attention 입니다.

Multi-Head Attention은 $h$개의 SDPA output을 concatenate한 후 linear projection해서 $d_{model}$차원의 output을 얻습니다. 또한 계산 비용을 Single-Head Attention과 유사하도록 하기 위해 $d_k = d_v = d_{model}/h = 64 (h = 8)$로 hyperparameter를 설정합니다.

![](../../assets/img/transformer/transformer_9.png){: class="align-center"}

### FFN(Feed Forward Network)

Encoder layer와 decoder layer 모두에서 마지막 Multi-Head Attention의 output을 FFN을 한 번 더 통과시키는데 FFN의 공식은 아래와 같습니다.

$$ \text{FFN}(x) = \text{ReLU}(xW_1\ +\ b_1)W_2 + b_2 $$

이를 그림으로 나타내면 아래와 같습니다.

![](../../assets/img/transformer/transformer_10.png){: class="align-center"}

### Encoder Layer와 Decoder Layer

![](../../assets/img/transformer/transformer_11.png){: class="align-center"}

#### Encoder Layer

Encoder layer는 Multi-Head attention과 FFN으로 구성되어 있으며 추가적으로 residual connection, layer normalization, dropout을 적용했습니다.

![](../../assets/img/transformer/transformer_12.png){: class="align-center" width="300px" height="500px"}

#### Decoder Layer

Decoder layer는 Masked Multi-Head attention과 Multi-Head attention(encoder-decoder attention), FFN으로 구성되어 있으며 추가적으로 residual connection, layer normalization, dropout을 적용했습니다.

여기서 encoder-decoder attention이 등장하는데 구조는 Multi-Head attention과 같지만 Query는 decoder layer의 이전 Masked Multi-Head attention의 결과로부터 오고 Key와 Value는 Encoder(마지막 encoder layer)의 attention 값으로부터 온다는 점이 다릅니다. 해당 부분이 encoder와 decoder를 연결하는 부분이라고 생각하시면 될 것 같습니다.

🔎 위에서 언급하지는 않았지만 Query, key, Value가 모두 같은 결과로부터 계산되면 self-attention이라고 부릅니다. 즉, encoder-decoder attention을 제외한 transformer에서의 모든 attention은 self-attention이라고 봐도 무방합니다.

![](../../assets/img/transformer/transformer_13.png){: class="align-center" width="300px" height="500px"}

Encoder-Decoder attention을 자세히 살펴보면 아래와 같습니다.

![](../../assets/img/transformer/transformer_14.png){: class="align-center"}

## Transformer Model의 결과

- WMT 2014 English-German dataset
- BLEU Score 28.4로 SOTA 달성

- WMT 2014 English-to-French dataset
  - BLEU Score 41.0으로 SOTA 달성

## Key Point

- Transformer는 recurrent layer(Seq2Seq, LSTM), Convolutional layer을 기반으로 한 architecture보다 훨씬 빠르게 훈련이 됩니다.
- Recurrent layer, attention을 함께 쓰는 모델에서 attention만을 사용한 최초의 sequence transduction model이 transformer 입니다.
- WMT 2014 English-to-German and WMT 2014 English-to-French에서 모두 SOTA 달성

## Conclusion

🤔 해당 논문을 공부하면서 얼핏 알고 있었던 transformer의 구조, 학습과정을 자세히 알 수 있었던 것 같습니다. 사실 해당 논문이 나오고 한참 뒤에 논문을 읽는 것이라 해당 논문이 현재 딥러닝 모델에 얼마나 큰 영향을 끼쳤는지는 알고 있기 때문에 대단해 보이는 것 같지만 읽으면서 생각의 전환이라는 관점에서 정말 대단한 논문임을 다시 느꼈던 것 같습니다.

🤔 이후 transformer가 어떻게 발전되어 갔는지 공부해 가면서 transformer에 대한 이해도를 높일 필요가 있는 것 같습니다. 아직 공부를 제대로 해보진 못했지만 BERT, GPT, ViT, Swin Transformer, U-Net에 transformer 사용과 같은 경우들을 봤기 때문에 앞으로 트렌드를 따라가기 위해서라면 transformer 자체에 대한 심도있는 이해가 필요해 보입니다.

## ※ Reference

- [https://github.com/hyunwoongko/transformer](https://github.com/hyunwoongko/transformer)

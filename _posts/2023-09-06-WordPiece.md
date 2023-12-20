---
layout: post
title: "[Tokenizer] WordPiece Model(WPM)"
author: kjy
date: 2023-09-06 22:23:00 +09:00
categories: [Deep Learning, Tokenizer]
tags: [Deep Learning, Tokenizer, NLP]
comments: true
toc: true
math: true
---

해당 포스트에서는 tokenizer의 한 종류인 WordPiece Tokenizer에 대해 알아보겠습니다.

---

## WordPiece

WordPiece는 Google이 [Japanese and Korean voice search](https://static.googleusercontent.com/media/research.google.com/ko//pubs/archive/37842.pdf)에서 처음 제안한 subtokenizer 방법입니다. [BPE(Byte-Pair Encoding)](https://jjjuuuun.github.io/posts/BPE/)과 매우 유사하지만 몇몇 다른 점이 있습니다. 첫째, 단어의 첫번째 글자(character)를 제외한 글자에 접두사(`##`)를 모두 추가하여 subword를 식별합니다. 둘째, WordPiece에서는 단순히 빈도가 높은 것을 병합하는 것이 아니라 병합한 쌍의 우도(likelihood)를 높이는 방향으로 병합할 쌍을 찾습니다. 셋째, tokenizer를 적용할 때 BPE와 다르게 적용됩니다.

위에서 살펴본 차이점들에 유의해서 WordPiece를 설명드리겠습니다.

## WordPiece Training

1. Pre-tokenize

   Pre-tokenize를 띄어쓰기를 기준으로 했다고 했을 때의 결과가 아래와 같다고 가정을 하고 설명드리겠습니다.

   | Token | Frquency |
   | :---- | :------- |
   | hug   | 10       |
   | pug   | 5        |
   | pun   | 12       |
   | bun   | 4        |
   | hugs  | 5        |

2. Split character

   위의 token들을 가지고 vocabulary를 구성한다고 할 때 `h`, `##u`, `##g`, `p`, `##n`, `b`, `##s`와 같이 구성할 수 있습니다.

   | Token            | Frquency |
   | :--------------- | :------- |
   | h, ##u, ##g      | 10       |
   | p, ##u, ##g      | 5        |
   | p, ##u, ##n      | 12       |
   | b, ##u, ##n      | 4        |
   | h, ##u, ##g, ##s | 5        |

3. Byte-Pair & Scoring

   위에서 분리한 token을 byte-pair로 만든 후 WordPiece score를 구합니다.

   WordPiece score를 구하는 공식은 아래와 같습니다.

   $$\text{score} = \frac{\text{freq_of_pair}}{\text{freq_of_first_element} \times \text{freq_of_second_element}}$$

   | Byte-Pair | Freq_of_first_element | Freq_of_second_element | Freq_of_pair | Score |
   | :-------- | :-------------------- | :--------------------- | :----------- | :---- |
   | h, ##u    | 15                    | 36                     | 15           | 0.02  |
   | ##u, ##g  | 36                    | 20                     | 20           | 0.02  |
   | p, ##u    | 17                    | 36                     | 17           | 0.02  |
   | ##u, ##n  | 36                    | 16                     | 16           | 0.02  |
   | b, ##u    | 4                     | 36                     | 4            | 0.02  |
   | ##g, ##s  | 20                    | 5                      | 5            | 0.05  |

4. Merge
   Score가 가장 큰 `##g, ##s`를 하나의 token으로 merge 합니다. 그럼 `h`, `##u`, `##g`, `p`, `##n`, `b`, `##s`, `##gs`로 vocabulary가 update됩니다. 그 결과는 아래와 같습니다.

   | Token        | Frquency |
   | :----------- | :------- |
   | h, ##u, ##g  | 10       |
   | p, ##u, ##g  | 5        |
   | p, ##u, ##n  | 12       |
   | b, ##u, ##n  | 4        |
   | h, ##u, ##gs | 5        |

5. Iteration
   이후 3번과 4번을 사용자가 정한 vocabulary 크기가 될 때까지 진행합니다.

   만약 사용자가 vocabulary 크기를 `10`으로 정했다면 아래와 같은 순서로 3번과 4번을 반복하니다.

   - Vocabulary : `h`, `##u`, `##g`, `p`, `##n`, `b`, `##s`, `##gs`

     | Byte-Pair | Freq_of_first_element | Freq_of_second_element | Freq_of_pair | Score |
     | :-------- | :-------------------- | :--------------------- | :----------- | :---- |
     | h, ##u    | 15                    | 36                     | 15           | 0.02  |
     | ##u, ##g  | 36                    | 20                     | 20           | 0.02  |
     | p, ##u    | 17                    | 36                     | 17           | 0.02  |
     | ##u, ##n  | 36                    | 16                     | 16           | 0.02  |
     | b, ##u    | 4                     | 36                     | 4            | 0.02  |
     | ##u, ##gs | 36                    | 5                      | 5            | 0.02  |

     > 위와 같이 모든 쌍의 score가 같은 경우 가장 앞에 나오는 쌍이 merge 됩니다.

     | Token       | Frquency |
     | :---------- | :------- |
     | hu, ##g     | 10       |
     | p, ##u, ##g | 5        |
     | p, ##u, ##n | 12       |
     | b, ##u, ##n | 4        |
     | hu, ##gs    | 5        |

   - Vocabulary : `h`, `##u`, `##g`, `p`, `##n`, `b`, `##s`, `##gs`, `hu`

     | Byte-Pair | Freq_of_first_element | Freq_of_second_element | Freq_of_pair | Score                               |
     | :-------- | :-------------------- | :--------------------- | :----------- | :---------------------------------- |
     | hu, ##g   | 15                    | 15                     | 10           | 0.04 (Reference의 예시와 값이 다름) |
     | p, ##u    | 17                    | 21                     | 17           | 0.04                                |
     | ##u, ##g  | 21                    | 15                     | 5            | 0.01 (Reference의 예시와 값이 다름) |
     | ##u, ##n  | 21                    | 16                     | 16           | 0.04                                |
     | b, ##u    | 4                     | 21                     | 4            | 0.04                                |
     | hu, ##gs  | 15                    | 5                      | 5            | 0.06                                |

     | Token       | Frquency |
     | :---------- | :------- |
     | hu, ##g     | 10       |
     | p, ##u, ##g | 5        |
     | p, ##u, ##n | 12       |
     | b, ##u, ##n | 4        |
     | hugs        | 5        |

   - Vocabulary : `h`, `##u`, `##g`, `p`, `##n`, `b`, `##s`, `##gs`, `hu`, `hugs`

6. Result

   사용자가 정한 크기(`10`)의 vocabulary가 되면 아래와 같은 결과들을 얻을 수 있습니다. WordPiece에서는 merge oreder는 필요하지 않으므로 vocabulary만 저장하면 됩니다.

   - Vocabulary : `h`, `##u`, `##g`, `p`, `##n`, `b`, `##s`, `##gs`, `hu`, `hugs`

## WordPiece Apply

WordPiece의 경우 토큰화할 단어에서 시작해 vocabulary에 있는 가장 긴 하위 단어를 찾은 다음 분할합니다. 아래의 예시를 통해 자세히 알아보겠습니다.

- hugs

  ⅰ. Vocabulary에 있는 단어 중 `h`로 시작하며 해당 단어의 글자와 일치하는 단어를 먼저 찾으면 `h`, `hu`, `hugs`입니다. 이 중 가장 긴 하위 단어는 `hugs`이므로 hugs는 `hugs`로 나눠집니다.

- bugs

  ⅰ. Vocabulary에 있는 단어 중 `b`로 시작하며 해당 단어의 글자와 일치하는 단어를 먼저 찾으면 `b`입니다. 따러서 bugs는 `b`, `##ugs`로 나눠집니다.\
  ⅱ. Vocabulary에 있는 단어 중 `##u`로 시작하며 해당 단어의 글자와 일치하는 단어를 먼저 찾으면 `##u`입니다. 따러서 bugs는 `b`, `##u`, `##gs`로 나눠집니다.\
  ⅲ. Vocabulary에 있는 단어 중 `##g`로 시작하며 해당 단어의 글자와 일치하는 단어를 먼저 찾으면 `##g`, `##gs`입니다. 이 중 가장 긴 하위 단어는 `##gs`이므로 bugs는 `b`, `##u`, `##gs`로 나눠집니다.

- mug

  ⅰ. Vocabulary에 있는 단어 중 `m`으로 시작하는 vocab이 없으므로 `<unk>`로 대체합니다.

- bum

  ⅰ. Vocabulary에 있는 단어 중 `b`로 시작하며 해당 단어의 글자와 일치하는 단어를 먼저 찾으면 `b`입니다. 따러서 bum은 `b`, `##um`으로 나눠집니다.\
  ⅱ. Vocabulary에 있는 단어 중 `##u`로 시작하며 해당 단어의 글자와 일치하는 단어를 먼저 찾으면 `##u`입니다. 따러서 bum은 `b`, `##u`, `##m`으로 나눠집니다.\
  ⅲ. `##m`은 vocabulary에 없으므로 bum은 `<unk>`로 나타냅니다.( `b`, `##u`, `<unk>`로 나타낸다고 생각할 수 있지만 아닙니다.)

## 해결하지 못한 점

아래 reference의 예시들을 가지고 제가 이해한 방식대로 WordPiece tokenize를 해봤습니다. 그러나 중간에 아래 reference 예시들의 결과와 다른 값을 도출했습니다. 아직 이 부분을 해결하지 못해 우선 제가 이해한 방식을 정리해봤습니다.

😁 혹시 해결하신 분이 있다면 댓글로 남겨주시면 감사하겠습니다!!

# ※ Reference

- [https://wikidocs.net/166826](https://wikidocs.net/166826)
- [https://huggingface.co/learn/nlp-course/chapter6/6?fw=pt](https://huggingface.co/learn/nlp-course/chapter6/6?fw=pt)

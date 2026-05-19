---
layout: post
title: HRM-Text 1B 공개
date: '2026-05-19T22:16:05+09:00'
tags:
- llm
summary: Sapient Intelligence가 계층적 추론(HRM) 아키텍처 기반 1B 텍스트 모델 HRM-Text를 공개했다. 약 1천
  달러, 40B 토큰만으로 사전학습됐는데 MATH·DROP 벤치마크에서 Llama3.2 3B를 능가한다는 결과가 화제다. 저비용 사전학습 프레임워크
  공개와 잠재공간 추론 강화가 핵심 포인트다.
description: Sapient Intelligence가 계층적 추론(HRM) 아키텍처 기반 1B 텍스트 모델 HRM-Text를 공개했다. 약
  1천 달러, 40B 토큰만으로 사전학습됐는데 MATH·DROP 벤치마크에서 Llama3.2 3B를 능가한다는 결과가 화제다. 저비용 사전학습 프레임워크
  공개와 잠재공간 추론 강화가 핵심 포인트다.
slug: hrm-text-1b-공개-c1fa5a
categories:
- llm
category_ko: 대규모 언어 모델
image:
  path: https://haangman.github.io/J-Blog-AI/assets/img/2026/05/hrm-text-1b-공개-c1fa5a.jpg
  alt: HRM-Text 1B 공개
mermaid: true
sources:
- url: https://www.reddit.com/r/LocalLLaMA/comments/1thjgwr/sapient_intelligence_releases_hrmtext_1b_40b/
  title: 'Sapient Intelligence releases HRM-Text 1B: 40B tokens, ~$1k pretrain, beats
    Llama3.2 3B on MATH and DROP'
- url: https://github.com/sapientinc/HRM-Text
  title: 'HRM-Text: Trained on only 1k$ and 40b tokens with brain inspired hierarchical
    latent architecture'
---

HRM-Text 1B 라는 모델이 어제 공개됐다. Sapient Intelligence — HRM(Hierarchical Reasoning Model, 계층적 추론) 아키텍처를 들고 작년부터 이런저런 결과를 던지던 곳 — 이 텍스트 생성용으로 10억 파라미터짜리를 풀었거든. 타임라인이 잠깐 시끄러웠다.

사실은 1B 모델 자체는 별 게 아니다. 요즘 기준 작은 축. 근데 옆에 적힌 학습 비용이 좀 충격이었거든 — **GPU 16장, 40B 토큰, 약 1천 달러**. Llama 3 류를 처음부터 돌리려면 보통 수십만 달러 단위로 깨지는데, 여기는 0이 두세 개 빠진다. 게다가 MATH 랑 DROP — 수학 추론·독해 벤치마크 두 종 — 에서 Llama 3.2 3B 를 누른다고 차트를 들이밀었다.

솔직히 이런 류 발표는 일단 의심부터 한다. 벤치마크 차트는 마케팅 1번 무기고, 1B 가 3B 를 이긴다는 말은 보통 어딘가 조건이 빠져 있기 마련이라.



![Abstract geometric pattern of yellow and red lines.](https://haangman.github.io/J-Blog-AI/assets/img/2026/05/hrm-text-1b-공개-c1fa5a-1.jpg)
*[Photo by MARIOLA GROBELSKA on Unsplash](https://unsplash.com/@mariolagr?utm_source=blogmaker&utm_medium=referral)*



HRM 을 짧게 풀면 이런 그림. 보통의 트랜스포머는 토큰을 한 줄로 깔아두고 한 번에 쫙 계산하는 구조인데, HRM 은 토큰 하나를 뱉기 전에 **잠재공간(latent space)** 안에서 사고를 여러 번 굴린다. 사람으로 치면 한 문제를 보고 머릿속으로 두세 번 다시 따져보는 그 과정에 가까운 거. "brain-inspired" 라는 마케팅 단어가 붙는 이유 — 근데 이 표현 자체는 좀 과하다고 본다.

대충 흐름은 이렇다:

```mermaid
graph LR
  Input[입력 토큰] --> Embed[임베딩]
  Embed --> H[고수준 추론 루프]
  H --> L[저수준 토큰 루프]
  L --> H
  L --> Output[출력]
```

가운데 `H ↔ L` 왔다갔다 하는 부분이 핵심이지. 한 번 출력 내기 전에 잠재공간 안에서 여러 번 사고를 굴린다는 의미. 모델이 작아도 추론 단계를 깊게 끌고 가니까, 같은 파라미터 수 대비 풀어내는 문제 결이 달라진다는 게 그쪽 주장.



![researcher reading code on dual monitor desk](https://haangman.github.io/J-Blog-AI/assets/img/2026/05/hrm-text-1b-공개-c1fa5a-2.jpg)
*[AI generated (Pollinations · Flux)](https://pollinations.ai/)*



실제로 돌려본 사람들 후기를 좀 훑어봤다. 체감상 일반 채팅·요약 같은 잡일에서는 그냥 1B 짜리 답이 나오더라 — 짧고 살짝 어색하고. 근데 수학 문제 던지면 풀이 과정을 좀 더 끈질기게 따라가는 느낌이 있다더라. 누가 "느린데 영리한 1B" 라고 표현했는데, 그 말이 좀 와닿았다.

내가 더 흥미로웠던 건 모델보다 **사전학습 프레임워크를 같이 풀었다는 점**. 1천 달러로 1B 를 처음부터 돌릴 수 있는 레시피가 공개됐다는 거. 이게 사실이면, 그동안 "사전학습은 빅테크 영역" 이라고 굳어 있던 벽에 작은 금이 가는 셈. 대학원생이나 1인 개발자가 자기 도메인 데이터로 베이스 모델을 직접 굽는 길이 살짝 열린다는 의미.

물론 1천 달러는 16장 GPU 시간 비용만 계산한 거고, 데이터 준비·실험 반복까지 끼면 훨씬 더 든다. 그리고 MATH·DROP 두 벤치에서 3B 를 이긴다고 모든 태스크에서 3B 를 이기는 건 절대 아니지 — 이 둘은 추론 깊이가 결정적인 종목이라, HRM 구조가 유리할 수밖에 없는 트랙이라고 봐야 맞다.

그래도 며칠 안에 누군가 같은 레시피로 한국어나 코드 도메인에 붙여서 돌려보지 않을까. 그때 나오는 숫자가 진짜 시그널일 것 같은데.

***

**참고**

- [Sapient Intelligence releases HRM-Text 1B: 40B tokens, ~$1k pretrain, beats Llama3.2 3B on MATH and DROP](https://www.reddit.com/r/LocalLLaMA/comments/1thjgwr/sapient_intelligence_releases_hrmtext_1b_40b/)
- [HRM-Text: Trained on only 1k$ and 40b tokens with brain inspired hierarchical latent architecture](https://github.com/sapientinc/HRM-Text)
---
layout: post
title: Triton MoE 커널 공개
date: '2026-05-28T17:38:22+09:00'
tags:
- inference
summary: NVIDIA·AMD 공용으로 돌아가는 순수 Triton 기반 Mixture-of-Experts 디스패치 커널 TritonMoE 프리프린트가
  공개됐다. 추론 배치(최대 512 토큰)에서 Stanford의 CUDA 최적화 Megablocks 대비 89~131% 성능을 내며, 벤더 종속
  코드 없이 AMD에서도 수정 없이 동작한다는 점이 화제다.
description: NVIDIA·AMD 공용으로 돌아가는 순수 Triton 기반 Mixture-of-Experts 디스패치 커널 TritonMoE
  프리프린트가 공개됐다. 추론 배치(최대 512 토큰)에서 Stanford의 CUDA 최적화 Megablocks 대비 89~131% 성능을 내며,
  벤더 종속 코드 없이 AMD에서도 수정 없이 동작한다는 점이 화제다.
slug: triton-moe-커널-공개-b01dd4
categories:
- inference
category_ko: 추론·최적화
image:
  path: https://haangman.github.io/J-Blog-AI/assets/img/2026/05/triton-moe-커널-공개-b01dd4.jpg
  alt: Triton MoE 커널 공개
sources:
- url: https://www.reddit.com/r/MachineLearning/comments/1tpj6e5/crossplatform_fused_moe_dispatch_in_triton/
  title: 'Cross-Platform Fused MoE Dispatch in Triton: Portable Expert Routing Without
    CUDA [R]'
- url: https://i.redd.it/5ktg9x36go3h1.png
  title: 'Fused MoE dispatch kernel in pure Triton: 89-131% of Megablocks, runs on
    AMD with zero code changes'
---

요즘 추론(inference) 쪽 논문을 자주 들춰보는데, **TritonMoE** 라는 프리프린트가 올라온 게 좀 재밌어서 메모. Mixture-of-Experts — 줄여서 MoE, 큰 모델 안에 여러 개의 "전문가" 작은 네트워크를 두고 입력 토큰마다 골라 쓰는 구조 — 그 디스패치 커널을 **순수 Triton** 으로 짰다는 얘기다. CUDA 한 줄도 안 쓰고.

이게 왜 큰 얘기냐. 보통 MoE 커널은 NVIDIA CUDA 위에서 손으로 깎인다. 대표적인 게 Stanford 의 Megablocks. 빠르긴 한데 AMD GPU 에서는 그냥 안 돈다, 코드를 다시 짜야 한다. 근데 이번 TritonMoE 는 같은 코드를 AMD MI300X 에 그대로 올려서 돌렸다는 거다. *Zero code changes.* 이 한 줄이 사실 글의 절반이다.



성능은 어땠냐. 추론 배치 사이즈 최대 512 토큰 기준으로 Megablocks 대비 **89~131%**. 입력 분포에 따라 더 빠르기도, 살짝 느리기도. 솔직히 100% 근방이면 이미 충분히 의미 있다. 벤더 종속 없이 그 정도 나오면, 운영하는 입장에선 선택지가 갑자기 두 배가 되는 셈이니까.

핵심 트릭은 gate 와 up projection 의 GEMM 을 하나로 융합(fused)했다는 점이다. SwiGLU — 요즘 Llama·Mixtral 류가 쓰는 활성화 함수 — 의 두 projection 을 같은 타일 로드에서 한 번에 처리해서 메모리 왕복을 줄였다. 논문이 벤치마크에 쓴 모델이 Mixtral-8x7B 인데, 이게 정확히 그런 구조라 잘 맞물린다.

```python
gate, up = fused_gemm(x_tile, W_gate_up_concat)  # 한 번의 타일 로드
y = silu(gate) * up                              # SwiGLU
```



![developer typing code on laptop screen](https://haangman.github.io/J-Blog-AI/assets/img/2026/05/triton-moe-커널-공개-b01dd4-1.jpg)
*[AI generated (Pollinations · Flux)](https://pollinations.ai/)*



내가 본 한에서 이런 류의 "Triton 만으로 충분히 빠르다" 가 쌓이는 추세는 작년부터 슬슬 보였는데, MoE 처럼 디스패치가 까다로운 영역에서까지 이런 숫자가 나오는 건 처음 봤다. 체감상 GPU 벤더 락인이 풀리는 속도가 생각보다 빠르다.

다만 캐치 몇 개. **학습이 아니라 추론 배치 기준**이다. 학습은 배치가 훨씬 크고 그라디언트 경로까지 잡혀서 또 다른 얘기. 학습용으로도 같은 결과가 나올지는 좀 더 봐야 할 것 같다. 그리고 페이퍼가 프리프린트라 동료 평가를 거친 건 아니고, 실측 환경(드라이버 버전·ROCm 버전)에 따라 숫자가 흔들릴 여지도 있다. 내가 직접 MI300X 를 돌려본 게 아니라 어디까지나 논문 숫자 위에서의 인상.

그래도 — AMD 가 추론 서버 시장에 점점 비집고 들어오는 지금, 이런 portable 커널이 하나둘 자리잡으면 H100 못 구해서 줄 서는 그림도 조금은 풀릴지 모르겠다. 아직은 가능성 얘기지만, 한 번 직접 돌려볼 만한 가치는 충분해 보인다.

***

**참고**

- [Cross-Platform Fused MoE Dispatch in Triton: Portable Expert Routing Without CUDA [R]](https://www.reddit.com/r/MachineLearning/comments/1tpj6e5/crossplatform_fused_moe_dispatch_in_triton/)
- [Fused MoE dispatch kernel in pure Triton: 89-131% of Megablocks, runs on AMD with zero code changes](https://i.redd.it/5ktg9x36go3h1.png)
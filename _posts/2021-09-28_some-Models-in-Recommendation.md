---
layout: post
categories: posts
title: DeepLight, Fi-GNN, PLE.
subtitle: some Models in Recommendation.
tags: [Recommendation System]
date-string: SEPTEMBER 28, 2021
---

<p>
*** 추천 시스템의 작년까지의 논문 3가지를 정리해두려 한다.


1. DeepLight (FM + DNN)
2. Fi-GNN (GNN)
3. PLE (Multitask)

*결론부터 말하면 DeepLight의 AUROC > Fi-GNN의 AUROC 이지만 Fi-GNN이 좀 더 고차원의 Interaction을 잡아낼 수 있을 것으로 기대되는 부분이 존재한다. PLE는 Task들이 연관되어 있다면 좋은 성능을 낼 것으로 기대됨. 다만 우리 쇼핑몰에서 클릭과 구매가 연관되어 있는지는 의문.

## Issue 1. DeepFwFM의 Pruning 버전. DeepLight.

*DeepLight를 알기 전에 DeepFwFM을 알아야 하고, DeepFwFM을 알기 전에 DeepFM을 알아야 하며, DeepFM을 알기 전에 Wide and Deep을 알아야하고, 기본적으로 FM을 알아야 한다.

* FM: Feature Latent Vector의 내적을 통한 Interaction을 추가로 반영. 이를 통해 단일 Feature 로는 학습하기 힘들었던 부분을 반영 가능

  > 참고로, FFM은 Field 별로 다른 가중치를 반영한다는 차이가 있음

* Wide and Deep: Deep한 부분 (*실제로 깊어보이진 않음)에 추가로 Wide한 부분을 추가함으로서, 각 Fields 사이의 관계를 반영

  > (Cross-product Transformation: 참조하려는 k개 Field 값이 모두 1이면 1, 나머지 모든 경우에는 0 입력) 

* DeepFM: Wide and Deep 에서 Wide 부분을 FM으로 바꾼 것.

* DeepFwFM: DeepFM에서 Linear Part 계산 방식을 바꾼 것.

  > *기존에는 Sparse Feature, 즉 Field의 Raw Data에서 가중치를 곱하여 나온 결과를 활용했다면, DeepFwFM에서는 Embedding Vector를 활용하도록 바꿈.
  > *또한 가중치 없이 다 더하는 방식 (Weight-1 Connection)에서 각 Field의 가중치를 고려하는 방식으로 바꿈

* DeepLight: DeepFwFM 에서 특정 부분들의 계산을 제외함으로서, 약 27~46배의 속도 향상을 가져옴 (Criteo, Avazu Dataset 기준)

  > 정확히 말하면, ① DNN의 Fully Connected 부분에서 필요 없는 가중치 제외, ② FM 부분에서 필요 없는 Interactions 제외, ③ Embedding Vector 에서 필요 없는 요소 제외

## Issue 2. Spatial GNN의 Fi-GNN

*Fi-GNN은 다음 흐름을 따라오면 이해하기 좋다.

1. Field-aware Embedding
   
   > m개의 Field 를 One-hot Encoding (아마 고차원) 이를 m개의 Embedding Vector (아마 저차원)로 변환
   > 이를 통해 모든 Field의 차원 수(d)를 같게 만들어주는 효과를 가져옴.
   
2. Multi-head Self-Attention

   > 각 Fields Pair의 Dependency(관계)를 파악하기 위한 과정.
   > Embedding Vector의 차원 수를 d라고 했을 때 m*d의 Initial Node State를 얻게 됨.
   > (*Self-Attention이 현재 State와의 연관성을 뽑아내는 것이므로, 위의 FM에서의 Interaction을 반영하는 것과 비슷하다고 생각함)

3. Fi-GNN

   > Inital Node State 에서 이웃을 통해(=Spatial 방식) T번 업데이트.
   > (*여기에서 a는 attention score가 아닌 aggregated를 의미)

4. Attention Scoring Layer

   > 이후 최종 Score 계산.

## Issue 3. MMOE의 발전된 형태. PLE.

*Multitask Model 의 기대점은 하나의 모델이 여러 개의 Tasks를 한 번에 학습해서, 옆의 Task에서 배운 것을 다른 Task에도 활용하기를 기대하는 것이다.

* 기존 Multitask Learning Model의 문제점

  > a. Task 간의 연관성이 없다면 전체적으로 성능이 하락함.
  > b. Seesaw Phenomenom: A Task를 상승시키면 B Task 성능이 과도하게 하락함.

해당 논문에서는 이러한 문제를 모두 잡고 아래와 같이 두 Tasks 에서 모두 성능 향상을 가져왔다고 설명하고 있음.

* CGC

  > PLE는 CGC의 Single-level Model을 여러 개 쌓아서 만든 것이므로, CGC를 이해하면 된다.
  > c)와 d)를 약간씩 합친 것으로도 볼 수 있는데, 이렇게 Gate가 존재하면서, 각 Task를 담당하는 Experts를 구분해 놓으면 (빨강은 A, 초록은 B, 파랑은 Shared) 초기 적합에 유리한 부분이 존재한다고 함.

* Loss Function

  > 쇼핑몰에서는 보통 Funnel 에 따라서 Action이 발생하는데, 노출 → 클릭 → 구매와 같은 것을 의미한다.
  > 그래서 노출 까지만 이루어진 로그, 클릭 까지만 이루어진 로그, 구매까지 이루어진 로그를 각각 분류하여서 각 Task를 학습할 때에는 클릭에는 구매까지 이루어진 로그는 반영하지 않도록 하는 방식을 선택했다.

</p>

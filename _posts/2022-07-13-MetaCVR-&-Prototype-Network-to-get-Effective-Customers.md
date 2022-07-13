---
layout: post
categories: posts
title: MetaCVR & Prototype Network to get Effective Customers
subtitle: MetaCVR, Prototype, Optimization
tags: [Recommendation System]
date-string: JULY 13, 2022
---

*** Representation Learning 중에서 Small Scenario 의 CVR 모델에 적합한 MetaCVR 과, 해당 모델의 바탕이 되는 Prototypical Network 에 대해 알아보자.

* 보통 E-commerce 환경에서  Item 추천은 매번 Small Scenario에 가까우며, 장기적인 유저 반응에 더해, 단기적인 Promotion 의 영향을 크게 받는다.

* 이에 대해서 장기적으로는 "User Behavior Sequence" 의 Self-Attention 의 Representation Learning 을, 단기적으로는 각 Promotion Case 에 대한 Prototype Distance 계산을 통해 고객 Conversion 을 예측하려는 MetaCVR 이 고안되었다.

* 다만, 우리 회사에서 적용하려는 시스템은, User Behavior Sequence 가 존재하지 않아, 이 부분은 다른 방식으로 Representation 생성하도록 만들고, Prototype Network 를 주요 예측 모델로 활용하였다.

## Basics.

- Meta Learning
  
  - 적은 Sample 만 가지고도 새로운 환경에 빠르게 적용시켜보자.
    
    → 추천 시스템과 같은 Small Scenario에 적합.

- Few Shot Learning
  
  - Few 한 Data 에 대한 학습도 적절하게 수행해보자.
  - Few 는 학습 데이터가 적다는 의미가 아니며, 예측해야 하는 새로운 Data 에 대해서 Class 별로 데이터 수가 적게 있는 경우를 의미함.
  - e.g. 먼저, 이마트, 배민 학습 데이터(Not Few)로 학습
    → 이후 새롭게 M포인트몰 추천을 해야 하는데, M포인트몰 구매 건 수가 3건(Few) 밖에 안 됨.
    → 그래도 3건에 대해 학습(Few Shot Learning)을 진행.

- Prototypical Network
  
  - Neural Network 를 통해 임베딩 공간에 대한 비선형 매핑을 학습. 각 Class 에 대해 Prototype Representation 생성.

- MetaCVR
  
  - User Behavior Sequence, Item, User Feature 에서 Attention 을 통해 Representation 추출.
  
  - 각 Class의 Representation 을 통해 대표 Prototype 을 뽑음.
  
  - 새로운 Data 에 대해서 Representation 을 추출하고, 기존 Prototype 들과의 거리를 통해 어느 Class 에 할당할 것인지 정함.

## MetaCVR

<center>
    <div class="photoset-grid-custom" data-layout="213">
        <img src="/images/2022-07-13-MetaCVR-&-Prototype-Network-to-get-Effective-Customers/Architecture.png">
    </div>
</center>

- FRN
  - MainNet
  1. 각 Feature 들을 동일 크기의 Embedding Vector 로 변환.
  
  2. User Behavior Sequence Embedding 을 Self-Attention. → 고객 선호에 대한 정보를 뽑음.
  
  3. 2의 결과를 각각 Item, User Embedding 과 Attention. → 고객 행동에서 개인화된 정보와, 대상 상품에 대한 선호 정보를 뽑음.
  
  4. 3의 결과를 Interaction Embedding 과 Concat 후 MLP 적용
  - BiasNet
    - 시기에 따라 달라지는 User 정보를 반영하기 위함.
    - User Embedding 과 Context Embedding 을 함께 학습.
  
  → MainNet 과 BiasNet 결과를 합하여 BaseCVR Score 뽑음
  또한, 해당 결과로 DMN 을 위한 Prototype Representation 도 생성.
- DMN
  - FRN 의 결과를 바탕으로 Case 별 대표 Prototype 을 뽑음.
  - 논문에서 Case는 Promotion 이전, 중, 이후, 아예 없음. 의 4가지를 들었음.
- EPN
  - FRN 과 EPN 의 결과를 합하여 최종 모델 Score 를 산출.



개인적으로, 위와 같은 Representation 기반 모델의 경우

앞의 Representation 만 잘 뽑아 놓는다면 Prototype Distance 를 구하든, 단순한 트리 모델을 돌리든, 거의 다 결과는 잘 나온다고 생각하여, 모델의 핵심은 좋은 Representation 을 뽑는 기법이라고 생각함.



논문의 경우 User Behavior 라는 검증된 Positive Action 을 통해 좋은 Representation 을 뽑았지만, 우리는 그런 정보는 없어서 단순 User 와 Item Feature 의 Encoder 를 통해 결과를 도출해낼 수 밖에 없었다.

추가로, Promotion 의 Case 를 나눈 것과는 다르게, 우리는 Prototype 을 뽑는데 있어서 다른 기준을 적용했지만, 이 부분은 너무 회사 내용이 많이 들어가 있어 다루지는 않았다. 다만 개인적으로 상당히 흥미롭고 재밌는 접근이었다.

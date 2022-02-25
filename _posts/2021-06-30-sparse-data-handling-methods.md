---
layout: post
categories: posts
title: RBU, Focal Loss.
subtitle: some Methods in sparse data recommendation.
tags: [Recommendation System]
date-string: JUNE 30, 2021
---

흔히 말하는 불균형 데이터.

이런 경우에 활용하는 방법은 두 가지가 있다.

Over Sampling 또는 Under Sampling.

GAN 등 생성 모델을 통해 True Label을 만들어내는 것이 Over Sampling이고

Major Class를 줄여 불균형을 해소하는 것이 Under Sampling이다.

우리는 여러 방법들을 고민했고, 시간이나 인프라상 이유로 Under Sampling을 시도했다.

핵심은 너무 쉬운 정답 데이터는 학습 할 때 고려를 적게 하겠다는 것이다.

그래서 활용한 것이 2가지가 존재한다.

1. RBU(Radial-based Undersampling)

<center>
    <div class="photoset-grid-custom" data-layout="213">
        <img src="/images/2021-06-30-sparse-data-handling-methods/RBU.png">
    </div>
</center>

기존의 Undersampling 방법이 Cluster를 기반하으로 하는 등의 방식을 선택해왔는데

RBU는 Major Class의 밀집도가 높은 지역에서 Minor Class와 거리가 먼 Sample을 제거하는 방식을 택했다.

즉, True와 False의 경계에 있는 Decision을 학습하는 것이 더 중요하다고 생각한 것이다.

논문 상에서는 같은 조건의 다른 Undersampling 방식들에 비해 AUPRC 등의 성능이 높았다고 하였다.

실제로 우리도 성능이 높아진 것을 확인 할 수 있었다!

* 계산 방식 (참고)

> Binary Label이니까 Major와 Minor에 대해서 [exp(-(Major Sample과의 유클리디안 거리)\*\*2)]의 합 - [exp(-(Minor Sample과의 유클리디안 거리)**2)]의 합을 계산해서 (=Majority Class Potential) Major Group에 얼마나 가까운지를 계산할 수 있고, 순위까지 매길 수 있어서 이 순위가 높은 애들부터 제거하는 방식을 사용.
> 
> 우리는 거리를 계산하는 벡터는 따로 임베딩 없이 범주형 변수만 빼고, 연속형 변수만 가져다가 거리 계산 했음!

2. Focal Loss

Loss Function 상에서 Major Class의 영향을 줄이고자 한 것이 Focal Loss인데

먼저 Binary Classification에서 활용하는 Cross Entropy Loss Function을 알 필요가 있다.

<center>
    <div class="photoset-grid-custom" data-layout="213">
        <img src="/images/2021-06-30-sparse-data-handling-methods/Focal Loss 1.png">
    </div>
</center>

<center>
    <div class="photoset-grid-custom" data-layout="213">
        <img src="/images/2021-06-30-sparse-data-handling-methods/Focal Loss 2.png">
    </div>
</center>

우리에게 Major Class는 구매 안 함(False Label)을 의미하며 이 경우 y=0이다.

그리고 Major Class의 대부분의 쉬운 Sample들은 p값이 작겠지만 이들을 모두 더하게 되면 학습에서 꽤나 큰 비중을 차지하게 되고, 적절한 학습이 이루어지지 않을 가능성이 존재한다.

그래서 쉬운 Major Class Sample의 Loss 값을 더 Zero에 가깝게 하겠다는 것이 Focal Loss의 개념이다.

<center>
    <div class="photoset-grid-custom" data-layout="213">
        <img src="/images/2021-06-30-sparse-data-handling-methods/Focal Loss 3.png">
    </div>
</center>

별로 어려울 것도 없이, 쉬운 Sample은 정보량도 작게 만들어주겠다는 것이었다.

감마는 하이퍼파라미터로 당연히 클수록 쉬운 Sample의 정보량이 더 많이 작아지게 된다.

두 가지 방법을 활용하여 우리는 불균형 문제를 해결했다.

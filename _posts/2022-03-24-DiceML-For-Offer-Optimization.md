---
layout: post
categories: posts
title: DiceML For Offer Optimization
subtitle: DiceML, Optimization
tags: [Recommendation System]
date-string: MARCH 24, 2022
---

*** 최근에 우리 회사에서 제공중인 추천 서비스에 대해서 오퍼 금액의 최적화를 해야하는 이슈가 발생했다.

* 추천 시스템의 성과는 보통 Binary 로, 특정 컨텐츠에 특정 고객이 반응 했는지 여부이다.

* 다만 마케팅적인 측면에서, 반응을 한 고객이 있다고 하더라도 더 적은 금액에 반응을 했을 수도 있으며, 반응 안 한 고객에게 조금 더 많은 금액을 제시할 경우 반응을 했을 수도 있다.

그리고 우리는 추천 모델에서 특정 변수를 최적화하기 위해 **<u>DiCEML</u>** 을 사용하기로 했다.

## Intro.

* 우리는 먼저, Perturbation Theory(섭동 이론)을 이해하고 넘어가야 한다.
  
  * 섭동 이론이란 특정 공간에서 아주 미세한 일부 변수를 조정했을 때, 결과가 어떻게 바뀌는지에 대한 이론이다.
  
  * 보통, 결과를 수식으로 풀기에 너무 어렵거나 하는 경우에 많이 사용한다.

* 마찬가지로, 우리에게 특정 공간은 우리가 추천하려는 대상의 **<u>변수</u>** 공간이라 할 수 있고, 여기서 변화를 줄 일부 변수는 우리가 정할 수 있으며 특히 **<u>오퍼 금액</u>** 이 될 것이다. 그리고 우리가 관찰하려는 결과값은 **<u>반응 여부</u>** 가 될 것이다.

* 즉, 오퍼 금액에 어떤 변화를 주었을 때 반응 여부가 바뀌는지를 관찰하려는 것이 DiCEML에서 하려는 것과 같다.

* 그리고 여기에서 반응 여부가 바뀐 애들을 **<u>CounterFactual(CF)</u>** 이라고 한다.

그런데, 단순히 반응 여부가 바뀌었다고 해서 모두 같은 수준의 CF라고 할 순 없다. 최대한 오퍼 금액이 적게 바뀐 CF가 좋은 CF라고 할 수 있다.

- 먼저, Wacher et al이 제시한 Counter Factual에 대한 식을 보자.
  
  뒤의 항이 기존 Point로부터 너무 먼 거리에 존재하는 Counter Factual일 경우 덜 선택되도록 제한을 둔 것을 알 수 있다.
  
  - <center>
        <div class="photoset-grid-custom" data-layout="213">
            <img src="/images/2022-03-24-DiceML-For-Offer-Optimization/wachter_optimization.jpg">
        </div>
    </center>

- 위의 식을 바탕으로 만들어진 우리의 Optimization 식은 아래와 같다. 이를 통해 최적의 Counter Factual을 찾아내보도록 하자.
  
  - 간단하게 해석하자면, Loss 값은 작으면서, 기존 Point x 와의 거리는 가깝고, 다양성은 높은 Counter Factual을 구하겠다는 것이다.

<center>
    <div class="photoset-grid-custom" data-layout="213">
        <img src="/images/2022-03-24-DiceML-For-Offer-Optimization/Optimization.jpg">
    </div>
</center>

## Loss

* Hinge Loss Function

* 위에서도 계속 언급했지만, 우리가 찾으려는 Counter Factual은, 원래의 Point와 굉장히 가까우면서도 결과를 바꾸는 Point이다. 그래서 이 식에서 사용한 Loss Function에는 특정 Threshold를 넘게 만드는 (결과가 반대로만 나오면 된다는 마인드) Point라면, 얼마나 더 Threshold를 넘기든 Zero Panelty를 주도록 설계되었다.
  
  그러니까, 예를 들면, Classification Model일 경우, Proba 값이 0.9999든, 0.5001이든 같은 Loss 값을 주겠다는 것이다.
  
  * <center>
        <div class="photoset-grid-custom" data-layout="213">
            <img src="/images/2022-03-24-DiceML-For-Offer-Optimization/hinge loss.jpg">
        </div>
    </center>

## Distance

* Continuous

* 먼저, Countinous 변수의 경우, 각 변수들의 Feature Range의 상이성이 존재하기 때문에, 각 변수의 MAD를 계산해서 나누어주었다. 그 후 차이 값을 통해 "거리가 먼 정도"를 구해주었다.
  
  * <center>
        <div class="photoset-grid-custom" data-layout="213">
            <img src="/images/2022-03-24-DiceML-For-Offer-Optimization/dist_cont.jpg">
        </div>
    </center>

* Category

* Category Feature의 경우, 거리르 구할 식은 존재하기는 하나, Continous와 같이, Feature별 Range의 상이성을 보완할 마땅한 방법이 없다. 그래서 본 논문에서는 기존과 다르면 거리 1을 부여하는 방식을 사용하였다.
  
  * <center>
        <div class="photoset-grid-custom" data-layout="213">
            <img src="/images/2022-03-24-DiceML-For-Offer-Optimization/dist_cat.jpg">
        </div>
    </center>

## Diversity & Proximity

* DPP Diversity

* 본문에서 다양성이란 것의 의미는 결과로 나온 Counter Factual들의 유사성에 대한 내용이다. 이론적으로, 각 Counter Factual들이 모두 직교하면 DPP Diversity는 커질 것이다.
  
  * <center>
        <div class="photoset-grid-custom" data-layout="213">
            <img src="/images/2022-03-24-DiceML-For-Offer-Optimization/dpp diversity.jpg">
        </div>
    </center>

* Proximity

* 우리가 뽑은 Counter Factual들이, 기존 Point x와 가까운 경우에, 우리가 활용할 수 있는 여지가 많다고, 논문은 얘기한다. 그래서 아래와 같이 Proximity를 반영한다.
  
  * <center>
        <div class="photoset-grid-custom" data-layout="213">
            <img src="/images/2022-03-24-DiceML-For-Offer-Optimization/Proximity.jpg">
        </div>
    </center>

## Search Method

* Random
  
  * Baseline 처럼 활용 가능, 가능한 공간에서 Random으로 Point를 Search.

* Genetic
  
  * 우리가 보통 알고 있는 유전 알고리즘.
  
  * 구체적으로 표현하면 아래와 같다.
    
    * 각 Feature별 Weight는 1/MAD로 시작.
    
    * 특정 시점의 Gene들에 대해서 Loss Value가 낮은 N개는 그대로 유지.
    
    * N개를 제외한 나머지 중 50%는 부모 2명을 골라 자손으로 대체.
      
      * 이 중 40%는 부모 1의 Feature를, 40%는 부모 2의 Feature를, 나머지 20%는 새로운 Feature Value를 부여받음.

* KD-Tree
  
  * KD Tree 공간에서 Nearest Neighbor를 찾는 방식.

그리고, 논문의 후반쯤에 the Local Decision Boundary 근사에 대한 내용이 나오는데, Original Input과, 생성된 CounterFactuals로 학습된 새로운 ML Model을 통해서 기존 모델과, 이렇게 생성된 모델을 통해 Decision Boundary를 계산할 수 있다고 표현.

개인적으로, 회사에서 사용하기에는 위에서 말한 Local Boundary를 근사하는 것이 필수가 아닐까 싶음. 분석가들 말고, 기획자들을 설득하기 위한 Tool로서.

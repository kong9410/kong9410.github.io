---
title: 오라클 인덱스 생성도의 비밀
tags: oracle database
layout: post
description: 오라클 인덱스 생성도의 비밀
---

## 원문

[개발자를 위한 인덱스 생성과 SQL 작성 노하우 | 이병국 | 글봄크리에이티브 - 교보문고 (kyobobook.co.kr)](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788996560081)

![indexBook](https://user-images.githubusercontent.com/37204770/176987914-9581fbc2-b19f-4876-bd33-8ea793fbf002.jpg)

## 오라클의 RBO 방식과 CBO 방식

RBO는 순위가 있는 규칙을 이용해 SQL에 대한 실행계획을 결정한다.

CBO는 SQL에 대한 최소한의 비용이 소요되는 실행계획을 선택한다. 오라클 10g부터는 CBO만 지원한다.

CBO 방식은 주기적으로 통계정보를 갱신해줘야한다. 통계정보를 근거로 해 여러 경우의  실행계획 중에서 가장 비용이 적게 드는 실행계획을 결정한다.

통계정보가 제공되지 않는다면 오라클은 실행가능한 모든 계획을 확인해봐야할 것이고 너무 많은 비용이 발생할 수 있다.

현재 우리가 할 수 있는 최선의 방법은 정확한 통계정보를 정기적으로 제공하는 것이다. 오라클이 알아서 좋은 실행계획을 제공하기를 기대하면 안된다. 우리 스스로 인덱스 생성도를 통해 최적의 인덱스 생성 포인트를 찾아야 하며, 그에 따라서 인덱스도 생성해주어야 한다.

## 인덱스 생성도의 기본 규칙

## 인덱스 생성도의 이해

CBO 방식에서 실행계획의 결정은 오라클이 주도적으로 하는 것처럼 보이지만 실제로는 현재 시점에서 알고있는 통계정보 범위 안에서 최소 비용이 되는 실행계획을 보여주는 것 뿐이다. 인덱스가 없거나 잘못된 인덱스가 있다면 잘못된 실행계획을 보여줄 뿐이다. 결국 개발자가 인덱스 생성도를 통해 최적의 인덱스를 생성해 주도적으로 실행계획을 결정해야한다.

다음과 같이 테이블이 있을때 성별 컬럼의 분포도는 50%이고 나이 컬럼의 분포도는 1%내외이다. 따라서 성별과 나이 컬럼은 인덱스 컬럼으로 사용하기에 부적합하고 분포도가 좋은 고객명 컬럼만 인덱스 생성도에 표시한다.

![column_img_1505.jpg](https://dataonair.or.kr/publishing/img/knowledge/column_img_1505.jpg)

![column_img_1506.jpg](https://dataonair.or.kr/publishing/img/knowledge/column_img_1506.jpg)

![column_img_1507.jpg](https://dataonair.or.kr/publishing/img/knowledge/column_img_1507.jpg)

![column_img_1508.jpg](https://dataonair.or.kr/publishing/img/knowledge/column_img_1508.jpg)![column_img_1508.jpg](https://dataonair.or.kr/publishing/img/knowledge/column_img_1508.jpg)![column_img_1509.jpg](https://dataonair.or.kr/publishing/img/knowledge/column_img_1509.jpg)
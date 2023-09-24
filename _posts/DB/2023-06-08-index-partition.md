---
title: 인덱스 파티셔닝
tags: oracle database
layout: post
description: 오라클의 인덱스 파티셔닝
---

![img](http://wiki.gurubee.net/download/attachments/28117622/img_local_part.JPG)

## 로컬 파티션 인덱스

- 테이블 파티션과 인덱스 파티션이 1:! 대응관계가 되도록 오라클이 자동으로 관리하는 파티션 인덱스를 말한다
- 파티션 키를 사용자가 정의하지 않아도 오라클이 자동으로 관리한다
- 로컬 파티션 인덱스는  1:! 관계를 형성하므로 테이블이 결합 파티셔닝이 되어있다면 인덱스도 같은 단위로 파티셔닝 된다
- 로컬 파티션 인덱스가 갖는 장점은 관리적 편의성에 있다. 테이블 파티션 구성에 변경이 있어도 인덱스를 재생성할 필요가 없다

> 결합파티셔닝
>
> 서브 파티션마다 세그먼트 하나씩 할당하고, 서브 파티션 단위로 데이터를 저장 주 파티션키에 따라 1차적으로 데이터를 분배하고 서브 파티션 키에 따라 최종적으로 저장할 위치 결정

## Non Partition Index

- 파티셔닝하지 않은 인덱스를 말한다
- 테이블만 파티셔닝이 되어있다면 1:M 관계를 갖는다
- 하나의 인덱스 세그먼트가 여러 테이블 파티션 세그먼트와 관계를 갖는다
- 글로벌 비 파티션 인덱스라고도 부른다

## Global Partition Index

- 테이블 파티션과 독립적인 구성을 갖도록 파티셔닝을 하는 것을 말한다
- 테이블은 파티셔닝이 되어있지 않을 수도 있다
- 몇몇 제약사항 때문에 효용성은 낮은 편이다
- 파티션 DDL로 영향 받는 레코드가 전체의 5% 미만일 때 유용하다

예1) 테이블은 월별 파티셔닝, 인덱스는 분기별 파티셔닝이 되어있다 가정했을 때, 글로벌 인덱스는 Prefixed만 사용한다. 그러면 날짜조건이 컬럼 선두에 있어야한다. 날짜 조건은 대게 범위검색조건이 사용되므로 인덱스 스캔효율면에서 불리하다.

예2) 테이블 파티션 기준인 날짜 이외 컬럼으로 인덱스를 글로벌 파티셔닝 할 수 있다. 그런 경우 적정 크기로 유지하려는데 목적이 있다

## Prefixed Vs NonPrefixed

- Prefixed: 파티션 인덱스를 생성할 때 파티션 키 컬럼을 인덱스 키 컬럼 왼쪽 선두에 두는것을 말한다
- NonPrefixed: 왼쪽선두가 아니고 파티션 키가 인덱스 컬럼에 아예 속하지 않을때도 여기에 속한다
- 글로벌 파티션 인덱스는 Prefixed만 지원된다
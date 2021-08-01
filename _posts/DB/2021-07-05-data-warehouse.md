---
title: 데이터 웨어하우스(Data Warehouse, DW)
tags: database
---

# 데이터 웨어하우스(DW)

사용자의 의사 결정에 도움을 주기 위하여 분석 가능한 형태로 정보들이 저장되어 있는 중앙 저장소이다. 일반적인 정의는 의사 결정에 필요한 정보처리 기능을 효율적으로 지원하기 위해 통합된 데이터를 가진 양질의 데이터 베이스이다.

## 특성

### 주제지향

기존의 데이터베이스가 대출, 예금, 적금과 같은 기능이나 업무 처리를 중심으로 설계된다면, 데이터웨어 하우스는 고객, 거래처, 공급자 등과 같은 주제 중심으로 구성된다.

### 통합

데이터 속성의 이름, 코드의 구조, 도량형 단위 등의 일관성을 유지하며 전사적 관점에서 하나로 통합한다.

### 시계열

기존 데이터베이스는 사용자가 사용하는 현재 시간을 기준으로 최신의 값을 유지하지만, 데이터 웨어하우스는 일정 기간 수집된 데이터를 갱신 없이 보관하며 일, 월, 분기, 년 등과 같은 기간 정보를 함께 저장한다.

### 비휘발성

기존 데이터베이스는 갱신 작업이 레코드 단위로 지속적으로 발생하지만, 웨어하우스 내의 데이터는 적재가 완료되면 읽기 전용 형태의 스냅 샷 데이터로 존재하게 된다.





## 데이터 웨어하우징

데이터의 수집 및 처리에서 도출되는 정보의 활용에 이르는 일련의 **프로세스**이다.

### ETL(Extract, Transform, Load)

웨어하우스 구축 시 데이터를 운영 시스템에서 추출, 가공하고 웨어하우스에 적재하는 과정을 말한다.

- Extract: 하나 또는 그 이상의 데이터 원천들로 부터 데이터 획득
- Transform: 데이터 클렌징, 형식 변환 및 표준화, 통합 또는 다수 애플리케이션에 내장된 비즈니스룰 적용
- Load: 변형 단계의 처리가 완료된 데이터를 특정 목표 시스템에 적재



## 데이터 웨어하우스의 이점

- 정보 기반한 의사 결정을 할 수 있다.
- 여러 소스의 데이터를 통합할 수 있다.
- 과거 데이터 분석
- 데이터 품질, 일관성, 정확성을 보장할 수 있다.
- 트랜잭션 데이터베이스와 분석 처리를 분리하여 두 시스템의 성능을 모두 향상시킬 수 있다.



## 데이터 레이크

정형, 비정형을 모두 포함한 가공되지 않은 다양한 종류의 데이터를 한 곳에 모아둔 리포지토리

## 데이터 마트

금융, 마케팅 또는 영업과 같은 특정 팀 또는 사업 단위의 요구를 충적 시키는 데이터 웨어하우스이다. 규모가 좀 더 작다.
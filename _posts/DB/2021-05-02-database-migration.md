---
title: 데이터베이스 마이그레이션(database migration)
tags: oracle database
---

데이터베이스 마이그레이션은 하나 이상의 source 데이터베이스에서 하나 이상의 대상(target) 데이터베이스로 데이터를 마이그레이션을 하는 것이다. 대부분 여러 db를 하나의 db로 합치거나 현재 DB를 타 DB로 옮기는 경우 등을 의미한다.

## 용어

- 소스 데이터베이스: 마이그레이션할 데이터가 있는 데이터베이스
- 대상 데이터베이스: 마이그레이션된 데이터를 수신하는 데이터베이스
- 데이터베이스 마이그레이션: 마이그레이션이 완료된 후 소스 데이터베이스 시스템을 종료하기 위해 소스데이터베이스에서 대상 데이터베이스로의 데이터 마이그레이션이다.
- 동종 마이그레이션: 소스 데이터베이스와 대상 데이터베이스의 DMBS가 동일할 때의 마이그레이션
- 이기종 마이그레이션: 소스 데이터베이스와 대상데이터베이스의 DMBS가 다른 경우
- 데이터베이스 마이그레이션 시스템: 소스 데이터베이스에서 대상 데이터베이스에 연결하고 데이터 마이그레이션을 수행하는 소프트웨어 시스템을 말한다.
- 데이터 마이그레이션 프로세스: 소스 데이터베이스에서 대상 데이터베이스로 데이터를 전송하여 전송 중에 데이터를 변환하기 위해 마이그레이션 시스템에서 실행하는 구성 또는 구현 프로세스다.
- 데이터베이스 복제: 소스 데이터베이스를 종료하지 않고 소스 데이터베이스에서 대상 데이터베이스로 데이터를 지속적으로 전송한다. 

### 복제와 마이그레이션

- 마이그레이션
  - 소스 데이터베이스에 대상 데이터베이스로 데이터를 이동시키고 소스 데이터베이스는 삭제
  - 클라이언트 액세스는 대상 데이터베이스로 리다이렉트
- 복제
  - 소스 데이터베이스를 삭제하지 않고 대상 데이터베이스로 데이터를 계속 전송 가능
  - 완료 시간이 딱히 존재하지 않음

### 이기종 마이그레이션과 동종 마이그레이션

- 이기종 마이그레이션
  - 다른 기술간의 마이그레이션을 말한다.
  - MySQL -> Oracle 마이그레이션과 같다.
  - PostgreSQL과 같은 자체 호스팅 DBMS에서 Cloud SQL과 같은 관리형 버전으로의 마이그레이션도 포함된다.
- 동종 마이그레이션
  - 동일한 기술간의 마이그레이션을 말한다.
  - 두 데이터베이스의 스키마가 동일할 가능성이 높다
  - 스키마가 다를 경우 소스데이터베이스의 데이터를 변환해야한다.

반드시 DBMS기준이 아닌, 모델 기반으로 동종과 이기종이 분류될 수 있다. MySQL에서 Oracle로 이전하는 같은 관계형 모델의 전환의 경우에 동종 마이그레이션이고, MySQL에서 MongoDB로 전환하는 경우 이기종 마이그레이션이다.

### 전환 프로세스

소스 데이터베이스에서 대상 데이터베이스로 클라이언트를 전환하는 프로세스

- 클라이언트가 기존 소스 데이터베이스 연결을 종료하고 대상 데이터베이스에 새 연결을 만들어야한다.
- 소스 데이터베이스에서 연결을 종료한 후에는 모든 변경사항이 소스데이터베이스에서 대상 데이터베이스로 마이그레이션 해야한다.(드레이닝)
- 대상 데이터베이스가 제대로 작동하고 클라이언트가 정의된 수준 목표 내에 작동하는지 테스트할 수 있다.

### 마이그레이션 방법

#### 데이터베이스 마이그레이션 시스템 사용

- Striim, tcVision, Cloud Data Fusion등이 있다.

- 데이터 마이그레이션 프로세스

  - 개발자가 직접 소스 데이터베이스, 대상 데이터베이스, 마이그레이션 하는 데이터, 데이터 수정 로직을 정의한다.

- 데이터 추출 및 삽입

  - 시스템의 변경사항은 트랜잭션 로그를 기반으로 한 데이터베이스 지원 변경 데이터 캡쳐와 데이터베이스 관리 시스템의 쿼리 인터페이스를 사용하는 데이터 자체의 차등 쿼리를 사용하여 삭제할 수 있다.

  > 트랜잭션 로그 기반: 데이터 변경사항이 올바른 순서로 포함되어 있다. 모든 변경사항을 관찰 할 수 있다. 각 변경사항이 표시되고 이후에 손실없이 올바른 순서로 대상 데이터베이스로 마이그레이션되므로 로깅에 유용하다.

  > 차등쿼리: 모든 변경사항을 올바른 순서로 관찰할 수 없는 경우 사용한다. 각 데이터 항목에 타임 스탬프 또는 시퀀스 번호가 포함된 추가 속성이 있다. 폴링 알고리즘이 마지막 시퀀스 번호 이후의 모든 데이터 항목을 읽어 변경사항을 대상 데이터베이스에 전달한다.

- 데이터 변환

  - 데이터 유형 변환: 데이터 유형이 동일하지 않는 경우 유형 변환 규칙에 따라 소스 값을 대상 값으로 변환한다.
  - 데이터 구조 변환: 동일한 데이터베이스 모델의 구조 또는 다른 데이터베이스 모델간의 구조를 수정한다.
  - 데이터 값 변환: 값 변환은 데이터 유형을 변경하지 않고 값을 변경한다. 예를들어 시간대는 UTC로 변환하거나 문자열로 표시된 짧은 우편번호는 긴 우편번호(5자리 뒤에 4자리, ZIP+4)로 변환한다.

- 데이터베이스 관리 시스템 복제 기능 사용
  - 대상 데이터베이스가 소스 데이터베이스의 복사본인 경우
  - 1:1 매핑이 된다.
  - 데이터베이스 관리 시스템 내의 기능을 사용하여 복제하고, 데이터 수정은 하지 않는다.
- 커스텀 데이터베이스 마이그레이션 기능 사용
  - 커스텀을 쓰는 이유는 다음과 같다
    - 모든 세부정보를 완벽하게 제어
    - 기능을 재사용
    - 비용을 줄이위함
  - 커스텀 솔류션은 구현을 유연하게 할 수 있지만 마이그레이션 중 발생할 수 있는 버그, 제한, 기타 문제를 위해 지속적인 유지보수가 필요하다.

### 마이그레이션 고려사항

- 오류 처리
  - 데이터가 손실되거나 변경사항이 순서대로 처리되지 않는 일이 없어야한다.
- 확장성
  - 마이그레이션 시간은 중요하므로 가능한 빨리 마이그레이션이 진행되어야 한다.
- 고가용성 및 재해 복구
  - 기본 데이터베이스에는 장애가 발생하면 기본 데이터베이스로 승격되는 복제본이 존재한다.
  - 기본으로 승격된 복제본으로 다시 마이그레이션 연결을 하여 진행한다.

#### 피해야하는 문제

- 순서 위반
  - 소스 데이터베이스 변경사항은 커밋된 트랜잭션에 따라 정렬된다. 트랜잭션 로그에서 변경사항을 선택하는 경우 마이그레이션 중에 순서를 유지해야한다.
- 일관성 위반
  - 차등 쿼리를 사용하면 커밋 타임스탬프가 있다. 즉 동일한 타임스탬프가 있는 모든 변경사항이 동일한 트랜잭션에 있어야한다. 그렇지 않으면 데이터가 일관되지 않을 수가 있다.
- 누락되거나 중복된 데이터
  - 일부 데이터가 기본 복제본과 장애 조치 복제본 간에 복제되지 않는 경우 신중하게 복구해야한다.
- 로컬 트랜잭션
  - 소스 디비와 대상 디비가 동일한 변경사항을 수신하도록 하려면 소스 디비와 대상 디비에 모두 쓰기작업을 하면 좋다.
  - 그러나 위 사항은 다음과 같은 문제가 있다
  - 두 디비 모두 개별 트랜잭션이다. 한 디비에서 오류가 발생할 경우 데이터가 일치하지 않게된다.
  - 클라이언트가 순서를 변경하여 데이터 불일치가 발생할 수 있다. 클라이언트가 대상 트랜잭션 순서를 보장하지 않는한, 데이터 불일치가 발생할 수 있다.

## 데이터베이스 마이그레이션 실행

### 1단계: 초기 로드

첫 번째 데이터 항목이 추출되기 직전에 데이터 베이스 시스템 시간을 기록한다. 이 타임스탬프를 사용하여 해당 시점부터 트랜잭션 로그에서 모든 데이터베이스 변경사항을 가져올 수 있다. `SELECT *`과 같은 쿼리 또는 프로젝션이 있는 쿼리를 실행한다. 지정된 프로세스대로 데이터를 수정을 진행한다.

### 2단계: 마이그레이션 진행

두 가지 목적이 있다. 하나는 초기 로드 이후 변경사항을 읽는다. 둘째는 변경사항을 캡처하여 대상 데이터베이스로 전송한다. 마이그레이션 단계가 진행되면 소스 데이터베이스와 대상 데이터베이스가 거의 완전히 동기화가 된다. 보통 변경사항은 일괄작업으로 진행된다. 배치가 작으면 불일치가 더 적게 발생하지만 자주 변경되면 소스에서 많은 부하가 일어날 수가 있다.

### 3단계: 드레이닝

소스에서 대상으로 클라이언트를 전환할 준비를 하려면 완전히 동기화되었는지 확인해야한다.

소스 데이터베이스 시스템을 종료하여 수정이 일어나지 않게 한다.

몯느 변경사항이 대상 데이터베이스로 마이그레이션 될때까지 기다린다. 이 프로세스는 변경사항의 실제 드레이닝이다.

드레이닝이 완료되면 이후 증분 백업에 정의된 시작점을 확보하도록 대상 데이터베이스를 백업한다.

드레이닝이 완료되었는지 확인하려면 마지막 삽입 데이터항목을 소스 데이터베이스에 기록하고, 마지막 삽입 데이터 항목이 대상 데이터베이스에 표시되면 드레이닝 단계가 완료된 것이다.

### 4단계: 전환

드레이닝 완료후 클라이언트를 대상 데이터베이스로 전환한다. 다음 권장사항을 따른다.

데이터베이스에 대한 액세스를 사용 설정하기 전에 기본 기능을 테스트하여 클라이언트가 작동하는지 확인한다.

클라이언트 액세스를 사용 설정하기 전에 대상 데이터베이스를 백업한다. 대상 데이터베이스의 초기 상태를 복구할 수 있다.

### 5단계: 소스 데이터베이스 삭제

프로덕션 데이터베이스로의 전환을 완료하면 소스 데이터베이스를 삭제할 수 있다. 소스 데이터베이스는 최종 백업을 수행하는 것이 좋다.

### 6단계: 대체

6단계는 선택사항이다. 대상 데이터베이스에서 다시 소스 데이터베이스로 마이그레이션을 설정하는 것이다. 소스 데이터베이스를 드레이닝하고 모든 db를 백업한 후에는 대상 데이터베이스의 변경사항을 식별하고 소스 데이터베이스로 이전하는 마이그레이션 프로세스를 사용 설정할 수 있다. 이는 마이그레이션된 데이터베이스에 문제가 생겼을 때 다시 소스 데이터베이스로 액세스 할 수 있게 하기 위함이다.

### 마이그레이션 중 동적 변경사항

- 스키마 변경사항: 사전 정의된 스키마가 필요하거나 스키마가 없는 시스템으로 분류할 수 있다. 이 경우 변경 관리 프로세스를 통해 변경사항을 제어한다.
- 데이터 변경사항: 스키마가 없는 시스템에는 데이터값에 대한 제약이없다. 이전에 저장되지 않은 데이터 값이 표시될 수 있다. 예를 들어 json구조는 문자열로 저장되지만 액세스시 JSON 값으로 해석될 수 있다.
- 프로세스 변경사항: 데이터 일치가 안될 수가 있다. 드레이닝 단계가 끝날 때까지 변경사항이 반영이 지연되는 것이 바람직하다. 이때 변경사항은 대체가 실행되지 않는 한 대상 데이터베이스 및 해당 클라이언트로 제한된다.

## 원문

[데이터베이스 마이그레이션: 개념 및 원칙(1부)](https://cloud.google.com/architecture/database-migration-concepts-principles-part-1?hl=ko)

[데이터베이스 마이그레이션: 개념 및 원칙(2부)](https://cloud.google.com/architecture/database-migration-concepts-principles-part-2?hl=ko)

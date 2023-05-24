---
title: database lock
tags: database
layout: post
description: 데이터베이스의 락 종류
---

## Lock이란?

데이터베이스는 여러 사용자들이 같은 데이터를 동시에 접근하는 상황에서, 데이터의 무결성과 일관성을 지키기 위해 락을 사용한다.

### Lock의 종류

#### 공유 락(Shared Lock)

데이터를 변경하지 않는 읽기 명령에 대해 주어지는 락으로 Read Lock이라고도 불린다. 여러 사용자가 데이터를 동시에 읽어도 데이터의 일관성에는 아무런 영향을 주지 않는다.

#### 베타 락(Exclusive Lock)

데이터에 변경을 가하기 위해 쓰는 명령들에 대해 주어지는 락으로 Write Lock이라고도 불린다. 베타 락은 이름처럼 다른 세션이 해당 자원에 접근 하는 것을 막습니다. 이러한 점에서 베타 락은 멀티 스레딩 환경에서, 임계 영역을 안전하게 관리하기 위해 활용되는 뮤텍스와 유사하다. 베타 락은 트랜잭션 동안 유지된다.

#### 업데이트 락(Update Lock)

데이터를 수정하기 위해 베타락을 걸기 전 데드락을 방지하기 위해 사용되는 락이다. 일반적으로 업데이트 락은 쿼리의 필터(WHERE)가 실행되는 동안 적용된다.

서로 다른 트랜잭션에서 동일한 자원에 대해 읽기 쿼리 이후, 업데이트 쿼리를 적용하는 경우 컨버전 데드락이 발생하는데, 이를 막기위해 일부 SELECT 쿼리에서도 업데이트 락(WITH(UPDLOCK))을 적용하기도 한다.

####내재 락(Intent Lock)

내재락은 사용자가 요청한 범위에 대한 락을 걸 수 있는지 여부를 빠르게 파악하기 위해 사용되는 락이다. 내재락은 공유 락과 베타 락 앞에 I 기호를 붙인 IS, IX, SIX등이 있다.

사용자 A가 테이블의 하나의 로우에 대해 베타 락을 건 경우, 사용자 B가 테이블 전체에 대한 락을 걸기 위해서는 사용자A의 트랜잭션이 끝날 때까지 기다려야 한다. 사용자 B가 테이블에 락(DDL Lock)을 걸 수 있는지 여부를 파악하기 위해 테이블에 존재하는 모든 로우와 관련된 락을 찾아보는 것은 매우 비효율 적인 작업이다.

따라서 데이터베이스는 사용자 A가 로우에 베타 락(X)을 거는 시점에 해당 로우의 상위 객체들에 대한 내재 락을 걸어, 다른 사용자가 더 큰 범위의 자원들에 대해 락을 걸 수 있는지 여부를 빠르게 파악할 수 있도록 돕는다.

#### Lock Escalation

하나의 로우에 대해 락을 생성하면, 상위 객체들에 대한 내재 락들이 함께 생성된다. 그러나 락은 많은 메모리 자원을 필요로 한다. 따라서 많은 데이터베이스들은 테이블 내의 일정 비율 이상의 로우에 대한 락을 생성할 경우, 모든 로우에 대해 락을 생성하는 대신, 더 상위 객체인 테이블에만 락을 걸도록하여 메모리 사용을 최적화하는 기능을 지원한다. 이러한 데이터베이스의 기능을 Lock Escalation이라고 한다.

#### Lock의 호환성과 Conversation Deadlock

내재 공유락은 베타락을 제외한 모든 락과 함께 실행이 가능하다. 베타락은 다른 락과 호환이 불가능하며, 다른 트랜잭션의 모든 락이 해제될 때까지 실행될 수 없다. 동일하게 베타락을 얻은 트랜잭션이 커밋/롤백 할 때까지 동일 범위에 대한 모든 락은 실행 권한을 얻을 수 없다.



## 출처

[[Database\] 데이터베이스 락(Lock)의 종류와 역할 (velog.io)](https://velog.io/@koo8624/Database-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-%EB%9D%BDLock%EC%9D%98-%EC%A2%85%EB%A5%98%EC%99%80-%EC%97%AD%ED%95%A0)
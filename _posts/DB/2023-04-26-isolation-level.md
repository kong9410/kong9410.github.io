---
title: isolation level
tags: database
layout: post
description: isolation level에 대해 알아보자
---

## ACID

- 원자성(Atomicity): 하나의 트랜잭션이 작업이 전부 성공하거나 전부 실패해야 한다
- 일관성(Consistency): 작업이 성공적으로 완료 되더라도 작업 이전과 같은 상태를 유지해야 한다.
- 격리성(Isolation): 작업이 수행되고 있을 때 다른 작업이 끼어들지 못한다.
- 지속성(Durability): 성공적으로 수행된 트랜잭션에 대해 영구히 반영되어야 한다.

## Transaction

- Database 데이터를 조작하는 작업의 단위다

- 예)

  A가 B에게 1000원을 송금

  - A계좌에 1000원을 차감
  - B계좌에 1000월을 추가

- 위의 예시에서 A와 B의 계좌의 변경이 모두 반영되어야 하며 어느 하나 일부만 성공하면은 안된다

## Isolation Level

**ACID**원칙을 지키다보면 동시성(Concurrency)문제가 발생한다. Isolation Level별로 차등을 두어 동시성에 대한 이점을 가질 수 있게할 수 있다.

ANSI/ISO SQL Standard에서 정의한 Isolation Level은 다음과 같다.

- READ UNCOMMITTED
- READ COMMITTED
- REPEATABLE READ
- SERIALIZABLE

> ANSI: 미국 국립 표준 협회
> ISO: 국제 표준화 기구

### READ UNCOMMITTED

`SELECT` 실행시 `COMMIT`이 완료되지 않은 데이터를 읽어올 수 있다. `COMMIT`되지 않은 데이터를 읽는 현상을 **Dirty Read**라고 한다.

![No Image](https://nesoy.github.io/assets/posts/img/2019-05-08-21-09-02.png)

### READ COMMITTED

`commit`이 완료된 데이터만 `SELECT`시에 보인다. 대부분의 dbms의 기본 설정값이다. `Read Committed`에서는 **Dirty Read**가 발생하지 않는다.

`commit`이 수행하지 않더라도 DB에 이미 값이 반영이 되어있는 상태인데 `commit` 이전의 데이터를 보장받기 위해서는 `commit`되지 않은 쿼리를 복구하는 과정이 필요하다. 이 시점에서는 **Consistent Read**를 수행해야 함을 의미한다.

하나의 트랜잭션 안에서 `SELECT`를 수행 할 때마다 데이터가 동일하다는 보장을 해주지 않는다. 그 이유는 다른 트랜젝션에서 해당 데이터를 `COMMIT` 했을 경우 `COMMIT` 된 데이터를 반환해주는게 `Read Committed`의 특징이기 때문이다. 위와 같은 이유로 `Read Committed`를 `Non-repeatable Read`라고 한다.

> Consistent Read
> read(=SELECT) operation을 수행할 때 현재 DB의 값이 아닌 특정 시점의 DB snapshot을 읽어오는 것이다. 물론 이 snapshot은 commit 된 변화만이 적용된 상태를 의미한다.

![No Image](https://nesoy.github.io/assets/posts/img/2019-05-08-21-18-08.png)

![No Image](https://nesoy.github.io/assets/posts/img/2019-05-08-21-25-41.png)

- 트랜잭션 1이 Commit 한 이후 끝나지 않은 트랜잭션2가 다시 테이블 값을 읽으면 값이 변경됨을 알 수 있다.
- 하나의 트랜잭션 내에서 똑같은 SELECT 쿼리를 실행했을 때는 항상 같은 결과를 가져와야 하는 REPEATABLE READ의 정합성에 어긋난다.
- 이러한 문제는 주로 입금, 출금 처리가 진행되는 금전적인 처리에서 주로 발생한다.
  - 데이터의 정합성을 깨지고, 버그는 찾기 어려워진다.

### REPEATABLE READ

한 트랜잭션 안에서 반복해서 `SELECT`를 수행하더라도 읽어 들이는 값이 변화하지 않는다.

트랜잭션은 처음으로 `SELECT`를 수행한 뒤 그 이후에 모든 `SELECT`마다 해당 시점을 기준으로 **Consistent Read**를 수행하여 준다. 그러므로 트랜잭션 도중 다른 트랜잭션이 `commit`되더라도 새로 `commit`된 데이터는 보이지 않게 된다. 이유는 첫 `select`시에 생성된 `snapshot`을 읽기 때문이다.

![No Image](https://nesoy.github.io/assets/posts/img/2019-05-08-21-52-08.png)

![No Image](https://nesoy.github.io/assets/posts/img/2019-05-08-22-14-18.png)

Phantom Read

- 다른 트랜잭션에서 수행한 변경 작업에 의해 레코드가 보였다가 안보였다가 하는 현상
- 이를 방지하기 위해서는 쓰기 잠금을 걸어야한다.

### SERIALIZABLE

모든 작업을 하나의 트랜잭션에서 처리하는 것과 같은 가장 높은 고립수준을 제공한다. `ReadCommitted`나 `Repeatable Read` 두개의 공통적인 이슈는 `Phantom Read`가 발생한다는 점이다.

> Phantom Read
> 하나의 트랜잭션에서 update 명령이 유실되거나 덮어써질 수 있는 즉, update 후 commit 하고 다시 조회를 했을 때 예상과는 다른 값이 보이거나 데이터가 유실된 경우를 phantom read라고 한다.

그와 다르게 `SERIALIZABLE`에서는 `SELECT` 쿼리가 전부 `SELECT ... FOR SHARE`로 자동으로 변경되어 `Repeatable Read`에서 막을 수 없는 몇 가지 상황을 방지할 수 있게 된다.

> S Lock: (Shared Lock, 공유잠금)
> 읽기 잠금(Read Lock)이라고도 불린다. 어떤 트랜잭션에서 데이터를 읽고자 할 때 다른 shared lock은 허용되지만 exclusive lock은 불가하다. 읽을 수 있되 변경은 불가능하다.
>
> X Lock: (Exclusive Lock, 배타적 잠금)
>
> 쓰기 잠금이라고도 불린다. 어떤 트랜잭션에서 데이터를 변경하고자 할 때 해당 트랜잭션이 완료될 때까지 해당 테이블 혹은 레코드를 다른 트랜잭션에서 읽거나 쓰지 못하게 하기 위해 Exclusive Lock을 걸고 트랜잭션을 진행시키는 것이다.


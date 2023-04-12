---
title: table access
tags: database
layout: post
description: 오라클에서의 table access 방법에 대해 알아보자
---

# Table Access 개요

Row Sources에서 Row를 탐색하는 방법으로 Access Path와 같은 기술이 있다. Full Table Scan의 경우 단일 Row sources에서 row를 검색하는 것이다. join은 두개의 row sources에서 입력을 받는다.

데이터베이스는 다른 관계형 구조에 대해 다른 액세스 경로를 사용한다. 다음 표는 각 데이터 구조의 일반적인 액세스 경로이다.

Data Structrures And Access Paths

| Access Path               | Heap-Organized Tables | B-Tree Indexes and IOTs | Bitmap Indexes | Table Clusters |
| ------------------------- | --------------------- | ----------------------- | -------------- | -------------- |
| Full Table Scans          | X                     |                         |                |                |
| Table Access by Rowid     | X                     |                         |                |                |
| Sample Table Scans        | X                     |                         |                |                |
| Index Unique Scans        |                       | X                       |                |                |
| Index Range Scans         |                       | X                       |                |                |
| Index Full Scans          |                       | X                       |                |                |
| Index Fast Full Scans     |                       | X                       |                |                |
| Index Skip Scans          |                       | X                       |                |                |
| Index Join Scans          |                       | X                       |                |                |
| Bitmap Index Single Value |                       |                         | X              |                |
| Bitmap Index Range Scans  |                       |                         | X              |                |
| Bitmap Merge              |                       |                         | X              |                |
| Bitmap Index Range Scans  |                       |                         | X              |                |
| Cluster Scans             |                       |                         |                | X              |
| Hash Scans                |                       |                         |                | X              |

옵티마이저는 각기 다른 실행 가능한 실행계획을 고려하여  각 계획에 비용을 할당한다. 가장 낮은 비용의 계획을 고른다. 일반적으로 인덱스를 이용한 access path가 효과적이다.

## Heap-Organized Table Access

기본적으로 테이블은 힙으로 구성되어 있다. 이는 데이터베이스가 사용자가 지정된 순서가 아닌 적합한 위치에 행을 배치하기 때문이다. 사용자가 행을 추가하면 데이터 세그먼트에서 첫번째로 사용 가능한 빈 공간에 행이 배치된다. 즉 삽입된 순서대로 검색되는 것을 보장하지 않는다.

힙 테이블은 각 테이블행에 고유한 rowId를 가지고 이 rowId는 행 조각의 물리적 주소에 해당한다. rowId는 행의 10바이트 물리적 주소다. rowId는 특정 블록 및 행 번호를 가리킨다.

오라클은 인덱스 구성을 위해 내부적으로 rowId를 사용한다. B-Tree 인덱스의 각 키는 연관된 행의 주소를 가리키는 rowId와 연관되어 있다. 물리적 rowId는 테이블 행에 대한 가능한 가장 빠른 액세스를 제공하여 데이터베이스가 단일 IO만으로도 행을 검색할 수 있게 한다.

### Full Table Scans

풀 테이블 스캔은 테이블의 모든 row를 읽는 방식이다. 그런 다음 조건에 맞는 행을 필터링한다.

옵티마이저가 테이블을 풀스캔 하는 경우는 다음과 같다

- 인덱스가 존재하지 않음
- 인덱스 row 쿼리 조건에 함수를 적용
- nullable한 인덱스 컬럼이 존재할 때 `SELECT COUNT(*)`을 사용하는 경우
- 쿼리가 인덱스의 선행 컬럼을 사용하지 않을 때
- 대부분의 테이블 블록을 필요하는 경우
- 통계가 오래된 경우
- 테이블이 작은 경우
- 테이블에 대한 높은 병렬성
- 쿼리 힌트가 full scan인 경우

### Table Access by RowId

rowId는 스토리지에 있는 데이터의 위치를 나타낸다. rowid는 데이터파일과 데이터블록과 해당하는 블록에서 행의 위치를 결정한다. 행 ID를 지정하여 행을 찾는것은 데이터베이스에서 행의 정확한 위치를 지정하기 때문에 단일 행을 검색하는 가장 빠른 방법이다.

Table Access 를 하는방법은 다음과 같다

- 선택된 행의 rowid를 WHERE 절에서 얻거나 하나 이상의 인덱스 스캔을 통해서 가져온다
- 인덱스없는 컬럼이 WHERE 조건에 필요할 수도 있다
- rowid를 기반으로 테이블에서 각 선택된 행을 찾는다

### In-Memory Table Scans

오라클 12c 버전부터 In-Memory Scan 는 In-Memory Column Store에서 행을 검색한다. IM열 저장소는 빠른 검색에 최적화된 특수 열 형식으로 테이블 및 파티션 복사본을 저장하는 선택적 SGA 영역이다

// TODO

## B-Tree Index Access Paths

인덱스는 데이터를 빠르게 액세스하는데 필요한 테이블 혹은 테이블 클러스터와 관계된 구조다. 인덱스를 하나 이상의 컬럼을 만들어서 랜덤하게 분선되어 있는 row set을 검색할 수 있게된다. 인덱스는 disk io를 줄일 수 있는 많은 방법중 하나다

![Description of Figure 8-3 follows](https://docs.oracle.com/database/121/TGSQL/img/GUID-A4C15694-3DB6-43D6-B2C6-9A40905DF972-default.gif)

B tree의 블럭은 검색할 브랜치 블록과 데이터를 저장하는 리프 블록이 있다.

nonunique 인덱스는 데이터베이스가 extra column을 붙여서 rowid를 저장한다.

B tree 인덱스는 절대 null key를 가지지 않는다. 이것은 옵티마이저가 path를 선택할때 중요한 사항이다.

### Index Unique Scans

index unique scan은 해당 조건을 만족하는 하나의 row scan한다. 중복되지 않은 unique한 값을 =조건으로 검색하면 데이터 한 건을 찾는 순간 더이상 탐색하지 않음

다음은 prod_id가 19인 product ID 레코드를 찾는 과정이다.

![Description of Figure 8-4 follows](https://docs.oracle.com/database/121/TGSQL/img/GUID-EA766B71-8C8D-42CF-BCC3-240B1CBF43DE-default.png)



인덱스 유니크 스캔이 사용되려면 다음과 같은 조건이 필요하다

- 쿼리가 인덱스 유니크 키를 사용해야한다.
- 유니크 키 연산 조건은 equal(=) 연산자를 사용해야 한다

### Index Range Scans

스캔의 범위는 양쪽에 제한을 둘 수도 있고 한쪽에서 제한을 둘 수 있다. 옵티마이저는 일반적으로 선택적인 쿼리에 대해 범위 스캔을 선택한다.

데이터베이스는 인덱스를 오름차순으로 저장한다. 예를들어 department_id >= 20 조건을 가진 쿼리는 인덱스키가 20, 30, 40 등인 row를 반환하기 위해 range scan을 사용한다.

다음은 department_id 컬럼이 20인 값을 요청하고, 중복되는 인덱스가 있다. 이 예제에서는 department 20에대한 2개의 인덱스 항목이 존재한다.

![Description of Figure 8-5 follows](https://docs.oracle.com/database/121/TGSQL/img/GUID-22743B64-5FBF-498A-BCD4-4F73342BA0EE-default.png)

index range scans를 사용하려면 다음과같은 조건이 필요하다

- 하나 이상의 인덱스 컬럼이 조건에 포함되어야 한다
- 조건은 하나 이상의 표현식의 조합이고 논리 연산자 그리고 true, false 값이어야 한다.
  - department_id = :id
  - department_id < :id
  - department_id > :id
  - department_id >: low AND department_id < \:high
- 인덱스키에는 0, 1 이상의 값이 가능하다

### Index Full Scans

전체 인덱스를 순서대로 읽는다. 인덱스의 데이터는 인덱스 키를 기준으로 정렬되어 있기 때문에 인덱스 전체 스캔은 별도의 정렬 작업을 제거할 수 있다.

루트 블럭에서 인덱스의 왼쪽부터 해서 리프블럭까지 따라 내려간다. 리프블럭에 도달하면 정렬된 순서대로 인덱스의 하단을 횡단하는 방식으로 스캔이 시작된다. 

아래 그림은 department_id를 기준으로 정렬된 department record를 요청한다

![Description of Figure 8-6 follows](https://docs.oracle.com/database/121/TGSQL/img/GUID-2C7E0C86-A1B8-47E3-BF9E-91B9F768A667-default.png)

옵티마이저는 다음과 같을 때 full scans를 고려한다

- 쿼리에 인덱스 컬럼이 포함된다. 이 컬럼은 선행 컬럼이 아니어도 된다
- 조건을 지정하지 않았어도 다음 조건이 충족된다면 index full scan을 사용한다
  - 모든 테이블, 쿼리 컬럼이 인덱스에 있다
  - 적어도 하나의 인덱스 컬럼은 not null
- 쿼리가 인덱스화 된 null이 아닌 컬럼에 대해 order by를 포함

### Index Fast Full Scans

인덱스 패스트 풀 스캔은 디스크에 정렬되지 않은 인덱스 블럭을 읽는데 사용된다. 인덱스를 사용하여 테이블을 스캔하지 않고 테이블 대신 인덱스를 읽는다. 기본적으로 인덱스 자체를 테이블로 사용한다.

데이터베이스는 멀티블럭 IO를 사용해서 루트 블럭에서 모든 리프블럭을 읽는다. 

옵티마이저가 인덱스 풀 스캔을 선택하는 경우는 다음과 같다

- index_ffs 힌트를 사용한다

### Index Skip Scans

복합 인덱스의 선행 컬럼이 skip되어 있거나 쿼리에 지정되어있지 않은 경우에 수행된다.

스킵 스캔은 테이블 블럭을 스캔하는 것보다 빠르고 full index scan보다 빠르다.

복합 인덱스를 논리적으로 더 작은 하위 인덱스로 분할한다.

인덱스의 선행 컬럼에 있는 고유한 값의 수에 따라 논리적 하위 인덱스의 수가 결정된다. 숫자가 작을 수록 최적화 도구가 생성해야 하는 논리적 하위 인덱스의 수가 적어지고 검색의 효율성이 높아진다. scan은 각 논리 인덱스를 개별적으로 읽고 선행하지 않는 열의 필터 조건을 충족하지 않는 인덱스 블록을 건너뛴다.

root 도는 브랜치 블록에서 읽은 컬럼 값 정보를 이용해 조건에 부합하는 레코드를 포함할 가능성이 있는 리프블록만 골라서 액세스 하는 방식

첫번째 리프블록과 마지막 리프블록은 항상 스캔

버퍼 pinning을 이용한 skip 원리

- 상위블록 scan : 브랜치 블록 버퍼를 pinning 한채로 리프블록을 방문했다가 다시 브랜치 블록으로 돌아와 다음 방문할 리프 블록을 찾는 과정 반복
- 브랜치 블록들 간 연결할 수 있는 주소정보를 갖지 않기 때문에 하나의 브랜치 블록을 모두 처리하고 나면 다시 그 상위 노드를 재방문하는 식으로 진행
- root또는 브랜치 블록을 재방문 하더라도  pinning 상태이기 때문에 추가적인 블럭IO는 발생하지 않음

다음과 같은 경우에 스킵 스캔을 사용한다

- 선행컬럼이 쿼리 조건에 있지 않는다
  - 인덱스카 (a, b)일때 a가 조건에 없는 경우
- 복합인덱스의 선행 컬럼에 고유한 값이 거의 없고, 인덱스의 후행컬럼에 고유한 값이 많이 있는 경우
  - (a, b) 인덱스가 있고 a컬럼이 두개의 고유값이있고 b는 수천개가 있는 경우

### Index Join Scans

쿼리에서 요청한 모든 열을 함께 반환하는 여러 인덱스의 해시 조인이다. 모든 데이터가 인덱스에서 검색되기 때문에 데이터베이스는 테이블에 액세스 할 필요가 없다.

인덱스 조인에는 여러 인덱스를 검색한 다음 얻은 rowid에 해시 조인을 사용하여 행을 반환하는 작업이 포함된다. 인덱스 조인 검색에서는 테이블 액세스가 항상 금지된다.

예를들어 단일 테이블에서 두 인덱스를 결합하는 방법은 다음과 같다

- 첫번째 인덱스를 스캔하여 rowid를 찾는다
- 두번째 인덱스를 스캔하여 rowid를 찾는다
- row를 가져오기 위해 rowid별로 해시 조인을 수행한다

옵티마이저가 인덱스 조인 스캔을 선택하는 경우는 다음과 같다

- 여러 인덱스의 해시 조인은 테이블 액세스 없이 쿼리에서 요청한 모든 데이터를 검색할 수 있다
- 테이블에서 행을 검색하는 비용이 인덱스를 읽고 테이블에서 행을 검색하지 않는 비용보다 높은 경우에 인덱스 조인이 비용이 많이 들 수가 있다. 예를들어 두개의 인덱스를 스캔하고 조인하는 경우, 인덱스를 선택한 다음에 테이블을 탐색하는 것이 비용이 덜 든다.
- index_join(table_name) 힌트를 사용하여 인덱스 조인을 지정할 수 있다.

> ChatGPT
>
> - 조인 연산이 많은 경우
>
>   조인연산이 많아서 테이블에 대한 조인을 수행하는 것이 느릴 때, 인덱스 조인 스캔을 사용하여 조인 연산의 수를 줄일 수 있다
>
> - 큰 테이블에서 작은 데이터 집합을 검색해야 하는 경우
>
>   대용량의 테이블에서 작은 데이터 집합을 검색할 때, 인덱스를 사용하여 작은 데이터 집합만 검색하면 더 효율적이다
>
> - 질의 결과를 대량으로 처리해야 하는 경우
>
>   조인 연산으로 생성된 결과 집합이 대량으로 처리되어야 하는 경우, 인덱스를 사용하여 처리 속도를 높일 수 있다
>
> - 외래키 제약 조건이 있는 경우
>
>   외래 키 제약조건을 이용하여 조인하는 경우, 인덱스를 사용하여 외래키 제약 조건을 검색하는 것이 더욱 효율적이다

예)

```sql
SELECT /*+ index_join(employees) */ last_name, email
  FROM employees
 WHERE last_name LIKE 'A%';
```

위와 같은 쿼리가 주어지고 인덱스는 emp_name_ix = (last_name, first_name), emp_email_uk = (email) 이 존재한다.

이를 index join을 통해서 실행하게 되면, 우선 email을 index full scan을 하여 모든 email을 찾는다. 그리고 A로 시작하는 last_name을 emp_name_ix 인덱스따라 range scan한다. 그리고 두개의 rowid를 서로 join한다.

### Bitmap Index Access Paths

비트맵 인덱스는 인덱스된 데이터를 row id 범위와 결합한다.

일반적인 B트리 인덱스에서는 하나의 인덱스 항목이 하나의 행을 가리킨다. 비트맵 인덱스에서는 키는 인덱스 데이터와 rowid 범위의 조합이다.

데이터베이스는 각 인덱스 키에 하나 이상의 bitmap을 저장한다. 비트맵의 각 값은 rowId 범위 내의 row를 가리킨다. 비트맵 인덱스에서 하나의 인덱스 항목은 단일 행이 아닌 일련의 행을 가리킨다.

비트맵 인덱스와 B 트리 인덱스 차이점

비트맵 인덱스는 B 트리 인덱스와 다른 키를 사용하지만 B 트리 구조에 저장된다.

| Index Entry      | Key                      | Data   | Example                                  |
| ---------------- | ------------------------ | ------ | ---------------------------------------- |
| Unique B tree    | 인덱스된 데이터                 | RowId  | `101,AAAPvCAAFAAAAFaAAa`                 |
| Nonunique B tree | rowid와 결합된 인덱스된 데이터      | None   | `Smith,AAAPvCAAFAAAAFaAAa`               |
| Bitmap           | rowid range와 결합된 인덱스 데이터 | Bitmap | `M,low-rowid,high-rowid,1000101010101010` |

데이터베이스는 비트맵 인덱스를 B트리 구조로 저장한다. 데이터베이스는 인덱스가 정의된 컬럼 집합인 키의 첫번째 부분에서 B트리를 빠르게 검색하고 해당 rowid 범위와 비트맵을 얻을 수 있다.

#### 언제 사용하는가

비트맵 인덱스는 자주 수정되지 않는 고유하지 않은 카디널리티 데이터에 적합하다. 데이터의 카디널리티는 열의 고유 값 수가 전체 행 수와 관련하여 낮을 때 낮다.

일반적으로 B 트리 인덱스는 높은 카디널리티 데이터에 적합하다. 최적화 도구는 몇 개의 행을 반환하는 쿼리에 대해 B 트리 인덱스를 선택할 수 있다. 대조적으로 비트맵 인덱스는 고유하지 않은 카디널리티 데이터에 적합하다. 옵티마이저는 몇몇개의 행을 반환하는 쿼리에 대해서는 B tree index를 선택한다. 반대로 비트맵 인덱스는 적은 distinct 카디널리티 데이터에 대해 적합하다. 예를들어 gender 컬럼은 두개의 고유한 값은 M or F와 null만 포함하므로 비트맵 인덱스의 후보다. 테이블에 1억개의 row가 있는 경우 모든 여성 고객에 대한 쿼리는 선택적이지 않으므로 비트맵 인덱스 액세스 후보가된다. 압축 기술을 통해 비트맵 인덱스는 최소 IO로 많은 rowId를 생성할 수 있다.

비트맵 인덱스는 데이터 웨어하우스에서 임시 쿼리의 속도를 높이는데 유용한 방법이다. 

### Bitmap Conversion to Rowid

비트맵 변환은 비트맵의 entry와 테이블의 row 간에 변환된다. 이 변환은 entry에서 row로 또는 row에서 entry로 전환할 수 있다

### Bitmap Index Single Value

비트맵 인덱스를 사용하여 단일 키 값을 검색한다.

쿼리는 1 value를 포함하는 위치에 대해 단일 비트맵을 검색한다. 데이터베이스는 1 value를 row id로 변환한 다음 row id를 사용하여 row를 찾는다. 

### Bitmap Index Range Scans

비트맵 인덱스는 범위 값을 찾는 테이블 액세스 방법이다.

이 스캔은 B-tree range scan과 유사하다

### Bitmap Merge

여러 비트맵을 병합하고 결과적으로 단일 비트맵으로 변환한다. 실행계획에서 비트맵 병합은 BITMAP MERGE 작업으로 표시된다

### Table Cluster Access Paths

table cluster는 공통의 컬럼을 공유하고 관계된 데이터를 같은 블록내에 저장하는 테이블 그룹이다. 테이블이 클러스터되어있으면 하나의 데이터 블럭은 다수의 테이블에 포함된다

### Cluster Scans

인덱스 클러스터는 인덱스를 사용하여 데이터를 찾는 테이블 클러스터다.

클러스터 인덱스는 클러스터 키의 B-tree 인덱스다. 클러스터 검색은 인덱싱 된 클러스터에 저장된 테이블에서 클러스터 키 값이 동일한 모든 행을 검색한다.

#### 동작방법

인덱스 클러스터에서 데이터 베이스는 클러스터 키값이 동일한 모든 행을 동일한 데이터 블럭에 저장한다. 예를들어 hr.employee2 와 hr.departments2 테이블이 emp_department_cluster에 클러스터 키가 department_id인 경우, 데이터베이스는 department 10의 모든 employee를 같은 블럭에 저장하고 department 20의 모든 직원을 동일한 블럭에 저장한다.

B tree 클러스터 인덱스는 클러스터 키 값을 데이터를 포함하는 블록의 DBA(데이터베이스 블럭 주소)와 연결한다. 예를들어 key 30의 인덱스 항목은 department 30의 employee에 대한 row를 포함하는 블록의 주소를 표시한다.

사용자가 클러스터에 행을 요청하면 데이터베이스는 인덱스를 검색하여 row를 포함하는 블록의 DBA를 가져온다. 그런 다음 Oracle Database는 이러한 DBA를 기반으로 row를 찾는다

### Hash Scans

해시 클러스터는 인덱스 키가 해시 함수로 대체된다는 점을 제외하면 인덱스 클러스터와 같다. 별도의 클러스터 인덱스는 없다.

해시 클러스터에서 데이터는 인덱스다. 데이터베이스는 해시 검색을 사용하여 해시 값을 기준으로 해시 클러스터에서 행을 찾는다.

## 출처

[Optimizer Access Paths (oracle.com)](https://docs.oracle.com/database/121/TGSQL/tgsql_optop.htm#TGSQL228)
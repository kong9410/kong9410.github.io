---
title: GET과 POST 차이
tags: restfulapi web
---

# GET과 POST 차이

## GET

- 클라이언트에서 서버로 리소스를 요청하기 위해 사용하는 메소드
- 주소 끝에 파라미터가 포함됨(query string)
- 캐시가 가능하다. (한번 요청한 리소스를 다시 다운하지 않고 복사본을 반환)
- GET요청은 브라우저 히스토리에 남는다
- 요청 길이 제한이 있다 (브라우저마다 다르다)
- 데이터 요청할때만 사용

## POST

- 캐시되지 않는다
- 브라우저 히스토리에 남지 않는다
- 데이터 길이 제한이 없다



|         | GET                | POST                 |
| ------- | ------------------ | -------------------- |
| 사용목적    | 리소스 요청             | 생성 및 업데이트            |
| BODY 유무 | HTTP 메시지에 BODY가 없다 | HTTP 메시지에 BODY가 존재한다 |
| 멱등성     | 멱등                 | 멱등이 아니다              |

> 멱등: 여러 번 적용하더라도 결과가 달라지지 않는 성질을 말한다.


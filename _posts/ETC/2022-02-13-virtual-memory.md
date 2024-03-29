---
title: 가상메모리란
tags: os memory
layout: post
description: os에서 사용하는 가상 메모리가 무엇인지 알아보자
---

가상 메모리

- 가상 메모리가 필요한 이유
  1. 메모리 용량 부속 이슈
  2. 프로세스 메모리 영역간에 침범 이슈
- 가상 메모리: 메모리가 실제 메모리보다 많아 보이게 하는 기술
  - 프로세스간 공간 분리로 프로세스 이슈가 전체 시스템에 영향을 주지 않음
  - 프로세스는 가상 주소를 사용하고, 실제 해당 주소에서 데이터를 읽고/쓸때 물리 주소로 바꿔준다
  - virtual address: 프로세스가 참조하는 주소
  - physical address: 실제 메모리 주소
- MMU(Memory Management Unit)
  - CPU에 코드 실행시, 가상 주소 메모리 접근이 필요할 때, 해당 주소를 물리 주소값으로 변환해주는 하드웨어 장치
  - ![💿운영체제 (가상 메모리 개념)](https://media.vlpt.us/images/sdk1926/post/60cb06ba-1fc1-4c9e-a34f-2d5e9db5b3b9/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-05-03%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.47.49.png)
- 페이징
  - ![Operating System - Virtual Memory](https://www.tutorialspoint.com/operating_system/images/virtual_memory.jpg)
  - 크기가 동일한 페이지로 가상 주소 공간과 이에 매칭하는 물리 주소 공간을 관리
  - 하드웨어 지원이 필요
  - 페이지 번호를 기반으로 가상주소/ 물리주소 매핑정보를 기록/사용
  - 프로세스의 PCB에 Page Table 구조체를 가리키는 주소가 들어있음
  - Page Table에는 가상 주소와 물리 주소간 매핑 정보가 있음
- 페이징 시스템 구조
  - page: 고정된 크기의 block
  - 가상 주소 v = (p, d)
    - p: 가상 메모리 페이지
    - d: p안에서 참조하는 위치
  - 페이지 크기가 4KB 예
    - 가상 주소의 0비트에서 11비트가 변위(d)를 나타내고
    - 12비트 이상이 페이지 번호가 될 수 있음
- 페이지 폴트(page faults)
  - 실제 물리 메모리가 부재일 때 뜨는 인터럽트
  - 페이지 폴트가 발생하면 OS가 이를 해결한 뒤 다시 동일한 명령을 수행
  - 이러한 경우가 많이 일어나면 성능이 저하됨
  - 페이지 교체 정책
    - 메모리가 꽉 차있을 때 기존 페이지 중 하나를 물리 메모리에서 저장 매체로 내리고 새로운 페이지를 방금 비워진 해당 물리 메모리 공간에 올린다.
  - 반복적으로 페이지 폴트가 발생해서 과도하게 페이지 교체 작업이 일어나 실제로는 아무것도 못하는 상황을 스레싱(Thrashing)이라고 한다.
- TLB(Translation Lookaside Buffer, 페이지 정보 캐쉬)
  - 가상 메모리 주소를 물리적 주소로 변환하는 속도로 높이기 위해 사용하는 캐시
  - 최근에 일어난 가상 메모리와 물리 주소의 변환 테이블을 저장해둠
  - CPU가 가상 주소를 가지고 메모리에 접근할때 TLB에 먼저 접근하여 물리 주소를 찾는다.
  - TLB에 매핑이 존재하지 않으면 MMU가 페이지 테이블에서 해당되는 물리 주소로 변환 후 메모리에 접근
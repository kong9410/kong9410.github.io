---
title: 깃허브 액션 간단정리
tags: git
layout: post
description: 깃허브 액션에 대해 간단히 정리하고 무엇을 할 수 있는지 알아보자
---
# 깃허브 액션이란?

깃허브 액션을 사용하면 깃 레포지토리에 어떤 이벤트가 발생했을 때 특정작업이 일어나게 하거나 주기적으로 반복시키는 작업을 해줄수가 있다. 예를 들어 누군가가 feature 브랜치에 push를 한다면 깃허브 액션이 발생해 자동으로 테스트, 빌드, 태깅, 배포를 거치는 ci/cd 과정을 진행할 수가 있다.
깃허브 액션은 public에서 빌드 환경을 제공해주기 때문에 *jenkins* 같은 별도의 애플리케이션 없이 git 하나가지고 버전관리 및 ci/cd를 할 수 있다는 장점이 있다. 또한 public 서버가 아닌 자체적으로 로컬 혹은 원격 서버에 깃허브 액션이 실행될 수 있는 *runner* 서버를 구축하여 실행할 수 있다.

## workflows
Github Action의 작업 과정을 말한다. workflows는 깃허브 레포지토리내의 `.github/workflows`내에 있는 `yaml` 파일을 실행한다. `yaml`파일은 여러개 만들어 여러 action을 취할 수가 있다.

이벤트 발생요소는 다음과 같이 만들 수 있다.
```yaml
on:
  push:
    branches: [RB-*]
```
위 설정은 `RB-`로 시작하는 브랜치에 `push`가 발생하면 워크플로우가 실행된다. 브랜치는 여러개로 지정해 줄 수 있다.
단순히 `push`나 `pull-request`뿐만 아니라 `shceduler`에 의한 워크플로우도 정의할 수 있다.
```yaml
on:
  schedule:
  - cron: "0 0 * * *"
```

## jobs
job은 독립된 가상 머신(vm) 혹은 컨테이너 위에서 돌아가는 작업을 의미한다. 하나의 워크플로우는 적어도 하나 이상의 job을 가지고 있어야한다. 모든 job은 동시 실행되며 의존 관계에 따라 실행순서를 조절할 수 있다.
```yaml
jobs:
  job1:
    ...
  job2:
    ...
  job3:
    ...
```
job에 대한 세부 내용을 작성할 수 있는데 필수값인 실행환경을 설정하는 `runs-on` 옵션을 줄 수 있다.
```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
```
해당 잡은 우분투 최신버전에서 구동된다.

## steps
job에서의 작업 단계를 말한다. 작업은 command나 script 실행이 될수도 있으며, action이 될수도 있다. 커맨드나 스크립트를 쓸때는 `run`, 액션을 사용할때는 `uses`를 사용한다.
```yaml
jobs:
  job1:
    - uses: actions/checkout@v3
    - run: npm install
```

## actions
깃허브 액션에서 빈번하게 사용되는 액션을 일종의 모듈화한 것이라 볼 수 있다. 위의 예시에서 사용한 `actions/checkout@v3` 은 특정 깃 저장소를 내려받고 브랜치로 전환하는 작업을 진행한다. 이 것을 매번 step에서 정의하기에는 비용이 많이 들기 때문에 미리 만들어둔 것이라 보면된다. 이 액션 외에도 더 많은 액션이 존재한다. Github Marketplace에는 수많은 깃허브 액션이 존재하기 때문에 필요에 따라 사용할 수 있다.

## 실사용 예시
ci/cd 까지는 필요가 없고, 운영환경 배포 브랜치인 RB 브랜치에 push가 되었을 때 깃허브의 릴리즈가 추가되도록 관리하려고 했다. 이를 직접 수동으로 작업할 수는 있지만 자동화 기능이 있으면 더 편할 것 같아서 깃허브 액션을 적용했다.
```yaml
name: release

on:
  push:
    branches: RB-*

jobs:
  to-release:
    permissions: 
      contents: write
    stpes:
      - uses: actions/checkout@v3

      - name: create release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref_name }}
          release_name: ${{ github.ref_name }}
          draft: false
          prerelease: false
```

### 결과
![image1](https://user-images.githubusercontent.com/37204770/221402829-9ffad95e-eb03-486b-9091-8cb62094307e.png)

![image2](https://user-images.githubusercontent.com/37204770/221402912-8f97de36-2085-4461-9521-c8f64e9db985.png)

릴리즈 태그 및 릴리즈가 잘된 것을 확인했다.

### 실무에서
사내에서는 보안문제 때문에 runner를 따로 만들고 관리를 해줘야해서 runner를 만들고 설정하는 작업이 필요하다. 또한 actions의 경우에도 사내에서 사용할 수 있는 제한이 있기 때문에 사내에서 쓰는 marketplace가 따로 있다. 그래서 바로 실무에서 깃허브 액션은 적용하지 못하고 있고, runner를 만들기 위한 도커 및 쿠버네티스 학습이 필요하다.
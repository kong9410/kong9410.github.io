---
title: 정적 분석(Static Analysis)
tags: 정적분석 ci
---

# 정적 분석

![Stay Ahead of the Glitch with Static Code Analysis - Blog - Sevaa Group](https://sevaa.com/app/uploads/2018/09/ft-image-static-analysis.png)

정적 분석이란 프로그램을 실행 시키지 않고 코드 분석을 수행합니다. 정적 코드 분석은 코드내에서 발견될 수 있는 보안 취약점, 그리고 잠재적인 결함, 위험등을 찾아내어 개발자에게 알려주고 개발단계에서 여러 위험을 해결할 수 있습니다.

## 왜 사용하는가?

단순히 실행가능하고 동작하는 프로그램은 누구나 만들 수 있습니다. 하지만 개발자마다 제각기 다른 코딩 방식과 여러 보안 위협이나 오류를 야기시킬 수 있는 코드를 작성한채 방치한다면 이후 프로그램을 유지 보수하는데에 많은 어려움을 겪게될 것 입니다. 때문에 코드가 작성된 뒤에는 반드시 규칙에 맞는 올바른 코드가 작성되어 있는지 점검하고 수정해야 합니다.

매우 작은 프로젝트의 경우에는 직접 코드를 검수해가며 수정할 수 있겠지만 프로젝트 규모가 커지고, 사람이 모든 코드를 하나하나 정확히 읽어낼 수는 없기때문에 사람이 직접 코드를 분석하기란 쉽지 않습니다. 이때 개발자에게 도움이 될수 있는 것이 정적 분석도구 입니다.

## 정적 분석 vs 동적 분석

프로그램을 분석하는 방법에는 정적 분석뿐만이 아닌 동적으로 분석하는 동적 분석이라는 것도 있습니다.

정적 분석과 동적 분석모두 애플리케이션의 취약점을 찾아내는 목적을 가지고 있습니다. 두 분석의 가장 큰 차이점은 분석기가 개발 싸이클에서 어느 시점에서 수행되는지 입니다.

동적 분석은 애플리케이션 테스팅 및 수행 단계에서 취약점을 발견합니다. 유닛 테스트과 같은 방법에서 수행되며 빌드가 이루어지고 프로그램이 실행되는 과정에서 분석할 수 있습니다.

정적 분석은 실행 시점 이전에 수행됩니다. 때문에 지금 당장 실행할 수 있는 애플리케이션이 필요한 것은 아니라 개발 초기부터 사용할 수가 있습니다. 런타임에 사용자 혹은 해커의 입장에서 기능을 테스트하여 분석하지만, 정적 분석은 코드 자체 혹은 코드의 흐름에서 문제점을 발견할 수 있습니다.

| BASIS FOR COMPARISON |         STATIC TESTING         |             DYNAMIC TESTING             |
| :------------------: | :----------------------------: | :-------------------------------------: |
|        Basic         | Does not execute the software. | Execution of the software is necessary. |
|         Cost         |              Low               |                  High                   |
|  Statement coverage  |              100%              |                   50%                   |
|   Time consumption   |              Less              |                  More                   |
|       Uncovers       |     Large variety of bugs      |          Limited types of bugs          |
|      Performed       |       Before compilation       |   Only when executables are available   |

## 정적 분석 도구 사용으로 얻는 이점

모든 개발자들이 정적 분석을 직접 수동적으로 검수할 수는 있습니다. 하지만 코드가 단순히 하나만 있는 것이 아닌, 여러 클래스, 메소드 등이 얽힌 관계에서 새로운 기능을 추가하는 것 뿐만이 아닌, 유지보수를 할때마다 해당 코드를 직접 검수하기는 힘들것입니다.

***Speed***

개발자가 리뷰를 할 수 있는 속도엔 한계가 있습니다. 일정을 진행하기에 바쁜데 개발된 코드들을 전부 직접 검토한다면  소요되는 시간이 많을 것입니다. 이러한 문제를 자동화 도구로 빠르게 해결할 수 있습니다. 그리고 어디에 문제가 있는지 표시를 해주기 때문에, 에러를 빠르게 고칠 수 있습니다.

***Depth***

모든 실행 경로를 직접 확인하기는 힘들 것입니다. 분석 도구는 그러한 심층 분석을 손쉽게 할 수 있습니다.

***Accuracy***

코드 리뷰는 사람의 착오가 생길 수가 있습니다. 분석 도구를 이용하면 정해진 규칙과 기준에 따라 분석을 하기 때문에 착오가 발생하지는 않습니다.

***Security***

애플리케이션 보안을 강화하여 DevOps의 보안을 강화할 수 있습니다.



## 정적 코드 분석 소프트웨어 특징

- **IDE 통합**

  대부분의 정적 분석 도구는 개발자의 IDE와 통합되어있어서 개발환경 내에서 솔루션을 제공합니다. 이러한 통합은 개발자가 작업흐름을 중단하지 않고서 지속적으로 코드를 감시할 수 있습니다.

- **적절한 시기의 경고 알람**

  정적 코드 분석은 코드에서 버그와 취약성을 몇 초내로 검색할 수 있기 때문에, 작업 효율성을 향상시킵니다. 이러한 적절한 경고 알람은 개발자가 초기에 버그에 대응할 수 있게 합니다.

- **권장 해결 방안 제시**

  단순히 오류를 감지하고 보고하는 것에서 끝나는 것이 아닌, 개발자에게 변경 가능한 혹은 권장하는 방법으로 수정 방안에 대해서 알려주기도 합니다.



## 정적 분석 소프트웨어

- **CheckStyle**

  체크 스타일은 프로그래머가 코딩 표준 컨벤션을 지킬 수 있도록 도와주는 도구 입니다. 자동적으로 코드를 체크하고 규칙에 위배되는 코드를 보고서로 만듭니다. 여러 개발자가 통일된 코드 양식으로 개발할 수 있도록 도와줍니다.

- **CPD**

  Copy/Paste Detector, 복사 붙여넣기로 짜여진 코드를 감지하는 도구 입니다. 중복된 코드를 검사하는 이유는 리팩토링을 할 때 코드가 짜여진 모든 곳을 개발자가 직접 찾아 수정해야합니다. 코드가 절대 변하지 않는다면 문제가 되진 않겠지만 대부분 그렇지 않습니다. 개발자가 직접 찾아 중복 코드를 수정할 수 있겠지만, 중복된 코드가 일부이고 변수명이나 매개변수 등이 약간만 변형된 형태라면 개발자가 직접찾기란 쉽지 않습니다. CPD는 이러한 중복코드들을 쉽게 찾아낼 수 있습니다.

- **SpotBugs(구, FindBugs)**

  코드내에서 발생될 수 있는 버그 취약 코드를 찾아내어 개발자에게  알려줍니다. 가변이 될수 있는 enum 클래스 변수라던지, 결과가 항상 false가 되는 if문 등을 잡아낼 수가 있습니다. 컴파일된 바이트코드로 작동하기 때문에 사전에 빌드가 필요합니다. 기존 FindBugs를 잇는 분석도구입니다.

- **PMD**

  Programming Mistake Detector, 다음과 같은 소스코드의 결함을 탐지할 수 있습니다.

  - try, catch, finally, switch의 빈 블럭 같은 것
  - 사용하지 않는 변수, 파라미터, 메소드
  - 불필요한 조건문, 루프에 루프가 더해지는 경우
  - String, StringBuffer의 낭비
  - 높은 복잡도
  - 중복 코드

  CPD와 같은 이미 위의 나열된 분석도구를 일부 포함합니다. 

  SpotBugs는 바이트 코드로 동작하기 때문에 소스코드로 작동하는 PMD와는 약간의 차이가 있습니다.

- **FindSecurityBugs**

  SpotBugs의 기능을 크게 향상시키는 SpotBugs용 보안 전용 도구입니다. 예를들어 MD5는 보안 용도에 적합하지 않은 약한 알고리즘으로 알려져있습니다. 지문과 같은 비보안 목적으로 사용할 수 있지만 다른 알고리즘 (SHA-2)가 선호됩니다. 이런 경우 MD5 사용을 바꿔야 하지만 어떤 경우에는 어렵습니다. 이렇게 코드 상의 문제를 찾는 것은 많이 어렵지는 않지만 새로운 코드로 개선하는데서 문제가 발생합니다. findsecuritybugs로 이러한 문제를 발견하고 고쳐나갈 수 있습니다.

- **IntelliJ Inspection**

  InjelliJ IDEA에는 프로젝트에서 비정상적인 코드를 컴파일하기 전에 이를 감지하고 수정하는 코드 검사 세트가 있습니다. 다양한 문제를 찾아내고 강조할 수 있고, Dead Code를 찾아내고, 발생 가능한 버그를 찾아내고, 철자 문제를 찾아내고, 전반적인 코드 구조를 개선할 수 있습니다.

- **ESLint**

  자바스크립트 문법에서 에러를 표시해주는 도구 입니다. ES와 Lint를 합친 것 입니다. ES는 Ecma Script로서, Ecma라는 기구에서 만든 Script, 표준 Javascript를 의미합니다. Lint는 에러가 있는 코드에 표시를 달아 놓는 것을 의미합니다.

- **Klockwork**

  정적 애플리케이션 보안 테스트(SAST) 표준 준수를 하기위해 사용합니다. SQL 주입, 오염된 데이터, 버퍼 오버플로, 취약한 코딩 등의 보안 취약점,  Null Pointer, 메모리/리소스 누출 등의 와 같은 버그, 품질 문제를 탐지합니다.
- **SonarQube**

  Jenkins는 CI안에 정적분석이 있는 반면, SonarQube는 정적분석에만 특화되어 있습니다. 

  SonarQube는 다음과 같은 방식으로 SCM과 CI와 연동하여 사용할 수 있습니다.

  ![img](https://docs.sonarqube.org/latest/images/architecture-integrate.png)

  1. 개발자가 IDE에서 코딩을 하고 SonarLint를 사용해 로컬 분석을 수행합니다
  2. 개발자가 개발 완료된 코드를 SCM에 Push 합니다.
  3. Github WebHook과 같은 트리거가 작동하여 CI에서 빌드가 이루어지고, SonarQube Scanner에 의해 코드 분석이 이루어집니다.
  4. 분석 결과를 SonarQube로 보냅니다.
  5. SonarQube 서버에서 분석 결과를 SonarQube 데이터베이스에 저장하고, SonarQube 웹 서버로 결과를 시각적으로 보여줍니다
  6. 개발자들이 결과를 확인하고 코드 수정 작업에 들어갑니다.

  

이 외에도 많은 정적 도구들이 있습니다. 필요에 따라 도구를 더 찾아서 쓰는 것이 개발시간을 단축시키는데 도움이 될 것 입니다.



## 정적 분석의 한계점

정적 코드 분석도 만능은 아니기 때문에 분명히 한계점이 존재합니다.



**1. 이해할 수 없는 개발자의 의도**

```c++
int calculate(int length, int width)
{
    return length + width;
}
```

이 경우에 계산 결과가 오버플로가 일어날 가능성이 있습니다. 하지만 이 기능이 근본적으로 무엇을 의도한건인지, length와 width값에는 최대 어느정도의 값이 들어올 수 있는지 등의 파악을 해야만 코드 수정이 가능해질 것 입니다.



**2. 네이밍의 의미까지는 파악하지 못함**

```c++
int divide()
{
    int x;
  	if(foo())
    {
        x = 0;
    }
    else
    {
        x = 5;
    }
    return (10/x);
}
```

여기서 정적 분석 도구는 어떠한 기능적인 결함이나 체크스타일의 오류는 찾을 수 없을 것 입니다. 하지만 개발자가 `foo()`의 역할이 무엇인지 또 `x`의 역할이 무엇인지 알지 못한다면 코드를 전혀 이해할 수 없을 것 입니다.



## 그 외

[TOP 40 Static Code Analysis Tools (Best Source Code Analysis Tools)](https://www.softwaretestinghelp.com/tools/top-40-static-code-analysis-tools/)

Software Testing Help라는 홈페이지에 게시된 장적 분석도구 탑 40입니다. 자바 외에도 다양한 언어에 쓸수있는 도구들이 소개되어 있습니다.



## 참고

[What Is Static Analysis? And What Is Static Code Analysis](https://www.perforce.com/blog/sca/what-static-analysis)

[SonarQube Architecture and Integration](https://docs.sonarqube.org/latest/architecture/architecture-integration/)

[What You Should Know About Static Code Analysis Software](https://www.g2.com/categories/static-code-analysis)

[소스 정적 분석도구 SonarQube 리서칭](https://joypinkgom.tistory.com/20)

[How correlation (hybrid analysis) works](https://help.hcltechsw.com/appscan/Enterprise/9.0.3/topics/c_how_correlation_works.html)

[Warnings-ng-plugins Supported Report Formats](https://github.com/jenkinsci/warnings-ng-plugin/blob/master/SUPPORTED-FORMATS.md)

[Findsecbugs for Developers](https://www.jenkins.io/blog/2020/03/02/findsecbugs/)

[Source Code Analysis Tools](https://owasp.org/www-community/Source_Code_Analysis_Tools)
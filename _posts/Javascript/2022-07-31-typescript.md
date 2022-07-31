---
title: 타입 스크립트
tags: javascript typescript
layout: post
---

## 특징

### 컴파일 언어, 정적 타입 언어

자바스크립트는 동적 타입의 인터프리터 언어로 런타임에 오류를 발견할 수 있다.

타입스크립트는 정적 타입의 컴파일 언어이며 타입스크립트 컴파일러 또는 Babel을 통해 자바스크립트 코드로 변환된다. 코드 작성 단계에서 타입을 체크해 오류를 확인할 수 있고 실행 속도가 빠르다. 하지만 매번 타입을 결정해야하기 때문에 코드량이 증가하며 컴파일 시간이 오래걸린다.

### 자바스크립트 슈퍼셋(Superset)

자바스크립트의 슈퍼셋, 즉 자바스크립트 기본 문법에 타입스크립트 문법을 추가했다. .js에서 .ts로 변경하고 타입스크립트로 컴파일 해 변환할 수 있다.

### 객체 지향 프로그래밍 지원

ES6(ECMAScript 6)에서 새롭게 사용된 문법을 포함하고 있으며 클래스, 인터페이스, 상속, 모듈 등과 같은 객체지향 프로그래밍 패턴을 제공한다.

## 언제 사용하는 가

### 높은 수준의 코드 탐색과 디버깅

코드에 목적을 명시하고 목적에 맞지 않는 타입의 변수나 함수들에서 에러를 발생시켜 버그를 사전에 방지한다. 코드 자동완성이나 실행 전 피드백을 제공하여 작업과 동시에 디버깅이 가능해 생산성을 높일 수 있다.

### 자바스크립트 호환

자바스크립트와 100% 호환된다. 프론트엔드 백엔드 어디든 자바스크립트가 사용된다면 타입스크립트도 사용될 수 있다.

### 강력한 생태계

대부분의 라이브러리들이 타입스크립트를 지원하며 VSCODE를 비롯해 각종 에디터가 관련 기능과 플러그인을 지원한다.

### 점진적 전환 가능

추가 기능이나 특정 기능에만 타입스크립트를 도입함으로써 프로젝트를 점진적으로 전환할 수 있다.

## 프레임워크와 조화

### 리액트

호환성은 좋은 편이다. 리액트 공식 홈페이지에서 타입스크립트를 사용하기 위한 가이드를 제공하고 있다. 페이스북에서 제공하는 리액트 웹 개발용 보일러 플레이트(Boilerplate)인 Create React App은 간단한 옵션 추가만으로 타입스크립트를 사용할 수 있도록 지원한다.

### 뷰

Vue 2.0에서는 사용할 수 있지만 몇몇 라이브러리의 도움을 받아야 한다. 뷰3.0에서는 공식적으로 지원한다.

### 앵귤러

버전 2부터 타입스크립트 기반으로 만들어졌고 타입스크립트를 권장하고 있다.

## 전환 방법

### VSCode로 자바스크립트 페어링

비주얼 스튜디오 코드에서 편집기 또는 특정 작업 영역에 대해 자바스크립트에서 타입스크립트 검사를 활성화 할 수 있는 설정이 포함되어 있다. `.vscode/setting.json`에 다음 코드를 추가할 수 있다.

```json
{
  "Javascript.implicitProjectConfig.checkJs": true
}
```

타입이 안전하지 않은 줄에는 에디터에서 해당 코드를 에러로 표시한다.

### 자바스크립트용 타입스크립트 컴파일러 사용

타입스크립트 컴파일러를 설치하고 스크립트를 실행하거나 설정한다. tsconfig.json 파일을 사용하여 컴파일러 설정을 세팅한다.

```json
{
  "compilerOptions": {
    "allowJs": true // 타입스크립트 컴파일러를 통해 실행할 자바스크립트 파일을 선택
    "checkJS": false // 자바스크립트 파일에 대한 타입 체크
    "outDir": "./dist"
    "rootDir": "./src"
    "strict": false
  }
}
```

### 자바스크립트 파일을 타입 스크립트 파일로 변환

자바스크립트에서 파일이 안전한 파일인지 확인 후 하나씩 타입스크립트 파일로 변환한다. (.js -> .ts) allowJS를 활성화 했으므로 한번에 모두 이동할 필요 없이 점진적으로 이동할 수 있다.

### 엄격한 타입 체크

타입 체크를 더 엄격하게 하는 몇 가지 옵션이 있다.

```json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true
  }
}
```

## 문법

### 기본 타입

```typescript
let str: string = 'hi';
let num: number = 100;
let arr: Array = [1, 2, 3];
let arr2: number[] = [1, 2, 3];
let obj: object = {};
let obj2: { name: string, age: number } = {
  name: 'hoho',
  age: 22
}
```

```typescript
function add(a: number, b: number): number {
  return a + b;
}

// 옵셔널 파라미터
function log(a: string, b?: string, c?: string) {
  console.log(a);
}
```

```typescript
// 튜플
var arr: [string, number] = ['aa', 100];
```

```typescript
// ENUM
// 기본적으로 0부터 시작하고 1씩 증가한다.
enum Shoes {
  Nike = '나이키',
  Adidas = '아디다스'
}
```

`Any`: 모든 데이터 타입을 허용한다

`Void`: 변수에는 undefined와 null만 할당하고 함수에는 리턴 값을 설정할 수 없는 타입이다.

`Never`: 특정 값이 절대 발생할 수 없을 때 사용한다.

### 인터페이스

```typescript
interface User {
  age: number;
  name: string;
}
```

```typescript
var person: User = {
  age: 30,
  name: 'aa'
}

function getUser(user: User) {
  console.log(user);
}
```

```typescript
// 인덱싱
interface StringArray {
  [index: number]: string;
}

var arr2: StringArray = ['a', 'b', 'c'];
arr2[0] = 10 // Error;
```

``` typescript
// 딕셔너리 패턴
interface StringRegexDictionary {
  [key: string]: RegExp
}

var obj: StringRegexDictionary = {
  cssFile: /\.css$/;
  jsFile: 'a' // Error
}

obj['cssFile'] = /\.css$/;
obj['jsFile'] = 'a' // Error
```

```typescript
// 인터페이스 확장
interface Person {
  name: string;
  age: number;
}

interface User extends Person {
  language: string;
}
```

### 오퍼레이터

```typescript
// Union 타입: 자바스크립트의 Or 연산자와 같은 의미다. Union 타입으로 지정하면 각 타입의 공통된 속성에만 접근 가능하다.
function askSomeone(someone: Developer2 | Person) {
  console.log(someone);
}
```

```typescript
// Intersection 타입: 자바스크립트의 AND 연산자와 같다. 각각의 모든 타입이 포함된 객체를 넘기지 않으면 에러가 발생한다.
function askSomeone(someone: Developer & Person) {
  console.log(someone);
}
```

### 제네릭

한 가지 타입보다 여러가지 타입에서 동작하는 컴포넌트를 사용하는데 사용된다.

```typescript
function logTest<T> (text: T): T {
  return text;
}

logText<string>('aa');
logText<number>(100);
```

### 타입 추론

타입스크립트가 코드를 해석하는 과정을 뜻한다. 한번 데이터 타입이 정해지면 이후 변경된다면 에러가 발생한다.

```typescript
var a = true;
a = 100 // Error;
```

```typescript
// 가장 적절한 타입(Best Common Type) 배열에 담긴 값들을 추론하며 Union 타입으로 묶어 나가는 것을 말한다.
var arr = [1, 2, true];
//해당 코드 타입을 Number | Boolean으로 정의한다.
```

```typescript
// 인터페이스와 제네릭을 이용한 타입 추론 방식
interface Dropdown<T> {
  value: T;
  text: string;
}

var items: Dropdown<boolean> {
  value: true,
  text: 'aa'
}
```

### 타입 단언

타입스크립트가 해석하는 것보다 더 확실한 목적을 가지고 개발자가 해당 코드에 타입을 직접 지정하는 것을 의미한다.

```typescript
var a;
a = 10;
a = 'string';
vab b = a as string;
```

```typescript
var div = document.querySelector('div') as HTMLDivElement;
div.innerText;
// 타입 단언을 함으로써 null을 대비한 분기문을 작성하지 않아도 된다.
```

### 타입 호환

특정 타입이 다른 타입에 잘 호환되는지를 의미한다.

- 구조적 타이핑: 코드 구조 관점에서 타입이 서로 호환되는지를 판단하는 것이다. 구조적으로 더 큰 타입은 작은 타입을 호환할 수 없다.

```typescript
interface Developer {
  name: string;
  age: string;
}

interface Person {
  name: string;
}

var developer: Developer;
var person: Person;

developer = person; // error
person = developer;
```


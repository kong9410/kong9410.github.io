---
layout: post
title: Promise, Async, Await
tags: javascript
---

## Promise

`Promise`는 객체로 비동기 작업의 완료와 실패 작업을 나타낸다. Promise가 가질 수 있는 상태는 다음과 같다.

- 대기(pending): 이행하거나 거부되지 않은 초기 상태
- 이행(fulfilled): 연산이 성공적으로 완료됨
- 거부(rejected): 연산이 실패함

이행이나 거부될 때 then 메서드에 의해 대기열에 오른다

![img](https://mdn.mozillademos.org/files/8633/promises.png)

### Method

`Promise.all(iterable)` : `iterable`내의 모든 프로미스가 이행한 뒤 이행하고, 거부하면 즉시 거부하는 프로미스를 반환한다.

`Promise.race(iterable)`: `iterable`내의 어떤 프로미스가 이행하거나 거부하는 즉시 스스로 이행하거나 거부하는 프로미스를 반환한다.

`Promise.reject()`: 거부하는 `Promise`객체 반환

`Promise.resolve()`:  주어진 값으로 이행하는 `Promise`객체 반환

```javascript
const myMethod = new Promise((resolve, reject) => {
    setTimeout(function() {
        resolve("yami");
	}, 250);
});

myMethod.then((success) => {
   console.log("YAMI " + success); 
});
```

## Async/Await

`async function`은 `AsyncFunction` 객체를 반환하는 비동기 함수를 정의한다. 암시적으로 `Promise`를 사용하여 결과를 반환한다.

`await`는 `async function`에서만 유효하다. 여러 `promise`의 동작을 동기스럽게 사용할 수 있게하려는데 목적이있다.

```javascript
async function foo() {
    return 1;
}
function foo() {
    return Promise.resolve(1);
}
```

위의 두개의 함수는 서로 같다. `await`가 없다면 `async`는 동기적으로 실행된다. `await`가 있다면 `async`는 항상 비동기적으로 완료가 된다.

```javascript
async function foo() {
    await 1;
}
function foo() {
    return Promise.resolve(1).then(() => undefined);
}
```

`await`는 비동기 호출이 fulfilled 될때까지 다음 명령을 실행하지 않는다.

```javascript
const helloWorld = function() {
    return new Promise(resolve => {
        setTimeout(function() {
            resolve("helloWorld");  
        }, 2000);
    });
};

const helloPromise = function() {
    return new Promise(resolve => {
        setTimeout(function() {
            resolve("helloPromise");
        }, 1000);
    });
};

const worldAndPromise = async () => {
    const twoSeconds = await helloWorld();
    console.log(twoSeconds);
    const oneSeconds = await helloPromise();
    console.log(oneSeconds);
};

worldAndPromise();
```

`helloWorld`는 2초 뒤에 실행이 되고, `helloPromise`는 1초뒤에 실행이되는 코드다. `Promise`를 반환해서 `async`가 될것같지만 `await`를 사용함으로 `async function`이 실행되는 동안 pending이 되는 것을 알 수 있다. 때문에 `oneSeconds`는 `twoSeconds`이후 1초 뒤에 출력이 된다.


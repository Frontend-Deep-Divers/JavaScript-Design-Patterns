# 9장 비동기 프로그래밍 패턴

# 비동기 프로그래밍

자바스크립트에서는 비동기(`async`, `await`), 프로미스를 통해 비동기 코드를 쉽게 작성할 수 있다.

- 콜백을 통한 비동기 프로그래밍

```js
// 콜백 사용 예시
function makeRequest(url, callback) {
  fetch(url)
    .then((response) => response.json())
    .then((data) => callback(null, data))
    .catch((error) => callback(error));
}

makeRequest("http://example.com/", (error, data) => {
  if (error) {
    console.error(error);
  } else {
    console.log(data);
  }
});
```

# 프로미스 패턴

프로미스는 작업이 성공적으로 완료되거나, 거부되었을 때 결과를 제공하는 일종의 계약서 같은 존재다.

```js
// promise 사용 예시
function makeRequest(url) {
  return new Promise((resolve, reject) => {
    fetch(url)
      .then((response) => response.json())
      .then((data) => resolve(data))
      .catch((error) => reject(error));
  });
}
makeRequest("http://example.com/")
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

## 프로미스 체이닝

여러 개의 프로미스를 연결해 복잡한 비동기 로직을 처리하는 패턴

## 프로미스 에러 처리

`catch` 메서드를 사용해 프로미스 체인의 실행 중에 발생할 수 있는 에러를 처리하는 패턴

## 프로미스 병렬 처리

`Promise.all` 메서드를 사용해 여러 프로미스를 동시에 실행하는 패턴

```js
Promise.all([makeRequest(url1), makeRequest(url2)]).then(([data1, data2]) => {
  console.log(data1, data2);
});
```

## 프로미스 순차 실행

`Promise.resolve` 메서드를 사용해 여러 프로미스를 순차적으로 실행하는 패턴

## 프로미스 메모이제이션

캐시를 사용하여 프로미스 함수 호출의 결과 값을 저장해 중복된 요청을 방지하는 패턴

```js
const cache = new Map();

function memoizedMakeRequest(url) {
  if (cache.has(url)) {
    return cache.get(url);
  }

  return new Promise((resolve, reject) => {
    fetch(url)
      .then((response) => response.json())
      .then((data) => {
        cache.set(url, data);
        resolve(data);
      })
      .catch((error) => reject(error));
  });
}
```

## 프로미스 파이프라인

함수형 프로그래밍을 사용해 비동기 처리의 파이프라인을 생성하는 패턴

## 프로미스 재시도

프로미스가 실패했을 때, 특정 조건에 따라 재시도하는 패턴

## 프로미스 데코레이터

고차 함수를 사용해 프로미스에 적용할 수 있는 데코레이터를 생성하고 프로미스에 추가 기능을 부여하는 패턴

```js
function logger(fn) {
  return function (...args) {
    console.log("Starting function...");
    return fn(...args).then((result) => {
      console.log("Function completed.");
      return result;
    });
  };
}

const makeRequestWithLogger = logger(makeRequest);

makeRequestWithLogger("http://example.com/")
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

## 프로미스 경쟁

`Promise.race` 메서드를 사용해 여러 프로미스를 동시에 실행해 가장 먼저 완료되는 프로미스 결과를 반환하는 패턴

# async/await 패턴

```js
// async/await 사용 예시
async function makeRequest(url) {
  try {
    const response = await fetch(url);
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error(error);
  }
}

makeRequest("http://example.com/");
```

## 비동기 함수 조합 패턴

여러 비동기 함수를 하나의 함수로 조합하여 복잡한 비동기 로직을 만드는 패턴

## 비동기 반복

`for-await-of` 반복문을 사용해 비동기 반복 가능 객체를 순회하는 패턴

```js
async function* createAsyncIterable() {
  yield 1;
  yield 2;
  yield 3;
}

async function main() {
  for await (const value of createAsyncIterable()) {
    console.log(value);
  }
}
```

## 비동기 병렬

`Promise.all` 메서드를 함께 사용해 여러 비동기 작업을 동시에 실행하는 패턴

## 비동기 순차 실행

`Promise.resolve` 메서드를 함께 사용해 비동기 작업을 순차적으로 실행하는 패턴

## 비동기 재시도

비동기 작업 실패 시 자동으로 재시도하는 패턴

```js
async function makeRequestWithRetry(url) {
  let attempts = 0;

  while (attempts < 3) {
    try {
      const response = await fetch(url);
      const data = await response.json();
      return data;
    } catch (error) {
      attempts++;
      console.log(`Retrying request: attempt ${attempts}`);
    }
  }

  throw new Error("Request failed after 3 attempts.");
}
```

## 비동기 데코레이터

고차 함수를 사용해 데코레이터를

```js
function asyncLogger(fn) {
  return async function (...args) {
    console.log("Starting async function...");
    const result = await fn(...args);
    console.log("Async function completed.");
    return result;
  };
}

// 자동으로 고차 함수로 감싸서 실행
@asyncLogger
async function main() {
  const data = await makeRequest("http://example.com/");
  console.log(data);
}
```

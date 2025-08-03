## 프로미스

### 프로미스 체이닝

여러 프로미스를 연결

### 프로미스 에러 처리

catch를 사용하여 에러 처리

### 프로미스 병렬 처리

Promise.all을 사용해 여러 프로미스를 동시에 실행

### 프로미스 순차 처리

Promise.resolve를 사용해 프로미스를 순차적으로 실행

순차 실행 체인의 안전한 시작점을 만들 때 유용

### 프로미스 메모이제이션

```jsx
const cache = new Map;
function memoizedMakeRequest(url) {
	if (cache.has(url)) {
		return cache.get(url);
	}
	
	return new Promise((resolve, reject) => {
		fetch(url)
		.then(response => response.json())
		.then(data => {
			cache. set(url, data);
			resolve(data);
		})
		.catch(error => reject(error));
	});
}
```

### 프로미스 재시도

```jsx
function makeRequestWithRetry(url) {
	let attempts = 0;
	
	const makeRequest = () => new Promise((resolve, reject) »> {
		fetch(url)
		.then(response => response.json())
		.then(data => resolve(data))
		.catch(error => reject(error));
	});
	
	const retry = error => {
		attempts++;
		if (attempts >= 3) {
			throw new Error('Request failed after 3 attempts.');
		}
		
		console.log(`Retrying request: attempt ${attempts}`);
		return makeRequest ();
	};
	
	return makeRequest().catch(retry);
}
```

### 프로미스 데코레이터

프로미스에 적용할 수 있는 데코레이터

```jsx
function logger(fn) {
	return function(...args) {
		console.log('Starting function...');
		
		return fn(...args).then(result => {
			console.log('Function completed.');
			
			return result;
		})
	}
}

const makeRequestWithLogger = logger(makeRequest);

makeRequesetWithLogger(url)
	.then(data => console.log(data));
```

### 프로미스 경쟁

race를 사용해 여러 프로미스를 동시에 실행하고 먼저 완료되는 프로미스의 결과를 반환

## async/await 데코레이터

```jsx
function asyncLogger(fn) {
	return async function(...args) {
		console.log('Starting async function...');
		const result = await fn(...args);
		console.log('Async Function completed');
		return result;
	};
}

@asyncLogger
async function main() {
	const data = await makeRequest(url);
	console.log(data);
}
```

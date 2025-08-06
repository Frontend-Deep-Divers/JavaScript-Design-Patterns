네임스페이스란 코드 단위를 고유한 식별자로 그룹화한 것

자바스크립트는 네임스페이스를 기본적으로 지원하지는 않지만, 객체와 클로저를 통해 효과를 얻을 수 있음

## 단일 전역 변수 패턴

하나의 전역 변수를 주요 참조 객체로 사용

```jsx
const myApplication = (() => {
	function method() {}
	
	return {
		method,
	}
})();

myApplication.method();
```

충돌 가능성 있음

## 접두사 네임스페이스 패턴

식별자에 접두사_ 를 붙여서 정의

많은 전역 객체가 생성됨 + 충돌 가능성 있음

## 객체 리터럴 표기법 패턴

전역 네임스페이스 오염시키지 않음

깊은 중첩까지 지원

객체 네임스페이스가 존재하는지 확인하고, 존재하지 않으면 새로 정의하는 방법 사용

```jsx
var myApplication = myApplication || {};
if(!myApplication) { myApplication = {} };
window.myApplication || ( window.myApplication = {} );
var myApplication = $.fn.myApplication = function() {}; // jQuery
var myApplication = myApplication === undefined ? {} : myApplication
```

## 중첩 네임스페이스 패턴

객체 리터럴 패턴의 발전된 형태

충돌 가능성이 더 줄어듬

참조 작업이 많아짐 → 실제 성능 차이는 크지 않음

## 즉시 실행 함수 표현식 패턴

```jsx
((namespace) => {
	const a = 1;
	const b = 2;
	
	namespace.foo = 'foo';
	namespace.bar = () => {
		speak('hello');
	}
	
	function speak(msg) {
		console.log(msg, a, b);
	}
})((window.namespace = window.namespace || {}));
```

## 네임스페이스 주입 패턴

this를 네임스페이스의 프록시로 활용하여 특정 네임스페이스에 메서드와 속성을 주입

직접 접근이 불가능한 상황에서만 사용하는 것을 추천

```jsx
const myApp = myApp || {};
myApp.utils = {};

(() => {
	let val = 5;
	
	this.getValue = () => val;
	this.setValude = (newVal) => {
		val = newVal;
	}
}).apply(myApp.utils);

myApp.utils.getValue();
```

## 중첩 네임스페이스 자동화 패턴

중첩 네임스페이스 패턴은 추가하고자 하는 계층이 늘어날수록 많은 하위 객체가 정의되어야 함

```jsx
function extend(namespace, namespaceString) {
	const parts = namespaceString.split('.');
	let parent = namespace;
	
	for(let i=0; i<parts.length; i++) {
		if(typeof parent[parts[i]] === 'undefined') {
			parent[parts[i]] = {};
		}
		
		parent = parent[part[i]];
	}
	
	return parent;
}
```

extend 호출 한 줄로 하위 객체를 자동 정의할 수 있음

## 의존성 선언 패턴

로컬 변수를 사용하는 것이 전역 변수를 사용하는 것보다 빠름

사용할 로컬 네임스페이스를 함수 영역의 상단에 선언해두고 사용

## 심층 객체 확장 패턴

jQuery나 lodash의 extend 등을 활용하여 객체를 확장

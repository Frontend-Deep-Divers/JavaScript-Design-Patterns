# 11장 네임스페이스 패턴

네임스페이스는 **코드 단위를 고유한 식별자로 그룹화**한 것을 의미한다.

네임스페이스는

- 전역 네임스페이스 내 다른 객체나 변수와의 충돌을 방지
- 기능을 체계적으로 구성해 재사용성, 편의성을 높임

자바스크립트에서는 객체와 클로저를 이용해 구현할 수 있다.

# 네임스페이스의 기초

네임스페이스 패턴의 종류는 다음과 같다.

- 단일 전역 변수
- 접두사 네임스페이스
- 객체 리터럴 표기법
- 중첩 네임스페이스
- 즉시 실행 함수 표현식
- 네임스페이스 주입

## 단일 전역 변수 패턴

하나의 전역 변수를 주요 참조 객체로 사용하는 방식이다.

```js
const myUniqueApplication = (() => {
  function myMethod() {
    // 코드
    return;
  }

  return {
    myMethod,
  };
})();

myUniqueApplication.myMethod();
```

⚠️ 변수명을 다른 개발자가 사용하고 있지는 않은지 조심해서 사용해 충돌에 유의해야 한다.

## 접두사 네임스페이스 패턴

고유한 접두사를 선정해 동일 네임스페이스에 해당하는 객체나 변수를 해당 접두사를 붙여 선언하는 방식이다.

```js
const myApplication_propertyA = {};
const myApplication_propertyB = {};
function myApplication_myMethod() {
  // ...
}
```

⚠️ 애플리케이션의 규모가 커지면 많은 전역 객체가 생성될 가능성이 있다.

## 객체 리터럴 표기법 패턴

객체 리터럴 표기법을 통해 객체를 생성하고, 키와 값으로 이뤄진 집합을 통해 중첩 네임스페이스를 생성할 수도 있다.

```js
const myApplication = {
  // 앞서 살펴봤듯이, 이 객체 리터럴에 대한 함수를 정의할 수 있습니다.
  getInfo() {
    // ...
  },

  // 또한 객체 안에 추가로 객체 네임스페이스를 만들어 필요한 요소들을 담을 수도 있습니다.
  models: {},
  views: {
    pages: {},
  },
  collections: {},
};
```

⚠️ 동일한 식별자의 객체 네임스페이스가 존재하는지 확인하고 정의하여 충돌 가능성을 낮출 수 있다.

```js
var myApp = myApp || {};
```

## 중첩 네임스페이스 패턴

객체 리터럴 패턴의 발전형이다.

```js
const myApp = myApp || {};

// 중첩된 하위 속성을 정의할 때에도 비슷한 방법으로 객체 존재 여부를 확인합니다.
myApp.routers = myApp.routers || {};
myApp.model = myApp.model || {};
myApp.model.special = myApp.model.special || {};

// 필요에 따라 중첩 네임스페이스를 복잡하게 만들 수도 있습니다.
// myApp.utilities.charting.html5.plotGraph(/*...*/);
// myApp.modules.financePlanner.getSummary();
// myApp.services.social.facebook.realtimeStream.getLatest();
```

## 즉시 실행 함수 표현식 패턴

```js
// 네임스페이스의 이름과 undefined를 인자로 전달합니다.
// 1. 네임스페이스를 지역적으로 변경할 수 있고,
// 2. 함수 컨텍스트 밖에서 덮어쓰여지지 않습니다.
// 3. undefined 값이 정말로 undefined임을 보장합니다.
//    ES5 이전 버전에선 undefined는 변경될 수 있었기 때문에
//    필요한 과정입니다.

((namespace, undefined) => {
  // 비공개 속성들
  const foo = "foo";
  const bar = "bar";

  // 공개 메서드와 속성
  namespace.foobar = "foobar";
  namespace.sayHello = () => {
    speak("hello world");
  };

  // 비공개 메서드
  function speak(msg) {
    console.log(`You said: ${msg}`);
  }
})((window.namespace = window.namespace || {}));

// 공개된 속성과 메서드를 테스트합니다.
// 출력: foobar
console.log(namespace.foobar);

// 출력: hello world
namespace.sayHello();

// 새로운 속성을 할당합니다.
namespace.foobar2 = "foobar";

// 출력: foobar2
console.log(namespace.foobar2);
```

## 네임 스페이스 주입 패턴

즉시 실행 함수 패턴의 변형으로, 함수 내에서 `this`를 네임스페이스의 프록시로 활용하여 특정 네임스페이스에 메서드와 속성을 주입한다.

여러 모듈이나 네임스페이스에 비슷한 기본 기능을 할당할 때 유용한 패턴이다.

```js
const myApp = myApp || {};
myApp.utils = {};

(function () {
  let val = 5;

  this.getValue = () => val;

  this.setValue = (newVal) => {
    val = newVal;
  };

  // utils 하위에 새로운 하위 네임스페이스인 tools를 생성합니다.
  this.tools = {};
}).apply(myApp.utils);

// 위에서 utils를 통해 정의한 tools 네임스페이스에 새로운 동작을 추가합니다.
(function () {
  this.diagnose = () => "diagnosis";
}).apply(myApp.utils.tools);

// 생성된 네임스페이스 구조를 출력
console.log(myApp);

// 출력: 5
console.log(myApp.utils.getValue()); // 5 출력

// 'val' 값을 변경하고 반환합니다.
myApp.utils.setValue(25);
console.log(myApp.utils.getValue()); // 25 출력

// 심층 단계를 테스트합니다.
console.log(myApp.utils.tools.diagnose()); // "diagnosis" 출력
```

`call` 메서드를 사용해서도 비슷하게 구현할 수 있다.

⚠️ `apply`를 활용한 네임스페이스 확장 기법은 클로저나 모듈 구조를 직접 만들 수 없는 상황에서만 쓰는 것이 좋고,
일반적인 경우에는 명시적으로 객체 속성이나 메서드를 선언하는 방식이 더 안전하고 명확하다.

# 고급 네임스페이스 패턴

대규모 애플리케이션에 유용한 패턴과 유틸리티들이다.

## 중첩 네임스페이스 자동화 패턴

중첩해서 계속 네임스페이스를 선언적으로 생성하다 보면, 계층이 늘어날수록 깊이가 커져 매우 번거로워질 수 있다.

함수를 통해 중첩 네임스페이스 생성을 자동화하는 패턴이다.

```js
// 최상위 네임스페이스에 객체 리터럴을 할당합니다.
const myApp = {};

// 문자열 형식의 네임스페이스를 파싱하고
// 자동으로 중첩 네임스페이스를 생성해주는 간편한 함수입니다.
function extend(ns, ns_string) {
  const parts = ns_string.split(".");
  let parent = ns;
  let pl;

  pl = parts.length;

  for (let i = 0; i < pl; i++) {
    // 프로퍼티가 존재하지 않을 경우에만 생성합니다.
    if (typeof parent[parts[i]] === "undefined") {
      parent[parts[i]] = {};
    }

    parent = parent[parts[i]];
  }

  return parent;
}

const mod = extend(myApp, "modules.module2");
extend(myApp, "moduleA.moduleB.moduleC.moduleD");
extend(myApp, "longer.version.looks.like.this");
```

## 의존성 선언 패턴

중첩 네임스페이스 패턴을 약간 변형한 형태다. 함수나 모듈에서 사용할 로컬 네임스페이스를 함수 영역에 선언(단일 변수 패턴)하는 패턴이다.

```js
// 중첩된 네임스페이스에 접근하는 일반적인 방법입니다.
myApp.utilities.math.fibonacci(25);
myApp.utilities.math.sin(56);
myApp.utilities.drawing.plot(98, 50, 60);

// 로컬 변수에 개칭한 참조를 사용합니다.
const utils = myApp.utilities;

const maths = utils.math;
const drawing = utils.drawing;

// 이렇게 하면 네임스페이스에 더 쉽게 접근할 수 있습니다.
maths.fibonacci(25);
maths.sin(56);
drawing.plot(98, 50, 60);

// 로컬 변수를 사용하는 이 방식은 중첩 네임스페이스에
// 수백, 수천번 호출이 발생하는 경우에만 성능이 향상됩니다.
```

개인적으로 필자가 추천하는 객체 리터럴 패턴을 사용한 중첩 네임스페이스 방법이 매우 편하게 느껴진다. 거기에 자동화 패턴까지 구현한다면 가독성이 높고, 간편하게 네임스페이스를 생성할 수 있을 것 같다.

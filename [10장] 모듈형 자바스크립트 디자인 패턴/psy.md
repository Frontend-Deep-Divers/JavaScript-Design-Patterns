모듈형이란 서로 의존성이 낮은 기능들이 모듈로써 저장된 형태

스크립트 로더: 모듈형 자바스크립트를 구현하기 위한 핵심적인 도구. RequireJS, curl.js

## AMD

모듈과 의존성 모두를 비동기적으로 로드할 수 있도록 설계된 모듈 정의 방식

CJS에 대한 사양 초안으로 시작했지만, 개발은 amdjs 그룹이 직접 수행 ⇒ CJS ≠ AMD

define 메서드로 모듈 정의, require 메서드로 모듈 로딩

### AMD가 모듈형 자바스크립트 작성에 좋은 이유

- 유연한 모듈 정의 방식에 대한 명확한 제안
    - 명확하게 모듈을 정의
    - 어떤 모듈을 의존하는지도 명확함
- 기존에 많이 사용되던 전역 네임스페이스나 <script> 태그 방식에 비해 훨씬 구조화됨
    - <script> 태그 방식은 로드 순서를 지켜야 함
- 전역 네임스페이스 오염 방지
- CJS와 달리 전송 방식을 제공
    - CJS는 동기적으로 모듈을 불러오는데, 이는 파일 시스템이 있는 Node.js 환경에 최적화되어 있음
    - AMD는 브라우저 환경을 고려하여 만들어짐
- 지연 로딩 지원
    - require 코드가 실행되는 시점에 로딩됨

## CJS

서버 사이드에서 모듈을 선언하는 간단한 API를 지정하는 모듈 제안

AMD와 달리 I/O, 파일 시스템, 프로미스 등 광범위한 부분을 다룸 → Node.js

재사용 가능한 자바스크립트 코드로써 외부 의존 코드에 공개할 특정 객체를 내보냄

## UMD

브라우저와 서버 환경 모두에서 작동할 수 있는 모듈

```jsx
(function (root, factory) {
  if (typeof define === "function" && define.amd) {
    // AMD 환경 (예: RequireJS)
    define([], factory);
  } else if (typeof module === "object" && module.exports) {
    // CommonJS 환경 (예: Node.js)
    module.exports = factory();
  } else {
    // 브라우저 전역 객체에 등록
    root.myLib = factory();
  }
}(this, function () {
  // 모듈 정의 (공통 부분)
  function add(a, b) {
    return a + b;
  }

  function subtract(a, b) {
    return a - b;
  }

  // 모듈이 내보내는 API
  return {
    add: add,
    subtract: subtract
  };
}));
```

# 7장 자바스크립트 디자인 패턴

## 💡 어떤 패턴을 선택해야 하는가?

최고의 패턴은 무엇인가에 대한 정답은 없다. 각 코드와 애플리케이션이 요구하는 부분은 다르기 때문이다. 대신 **패턴이 실질적인 구현에 도움이 되는지 고려**해야 한다.

때문에 디자인 패턴에 대해 정확히 알고 사용할 때를 잘 파악한다면 올바른 디자인 패턴을 적용할 수 있다!

---

# 7.1 생성 패턴

- [생성자 패턴](#생성자-패턴)
- [모듈 패턴](#모듈-패턴)
- [노출(Revealing) 모듈 패턴](#노출revealing-모듈-패턴)
- [싱글톤 패턴](#싱글톤-패턴)
- [프로토타입 패턴](#프로토타입-패턴)
- [팩토리 패턴](#팩토리-패턴)

# 생성자 패턴

여기서 '생성자'는 클래스의 생성자 메서드를 의미한다.

### 객체 생성

자바스크립트의 기본적인 객체 생성 방법이다.

```js
const newObject = {};
const newObject = Object.create(Object.prototype);
const newObject = new Object();
```

이렇게 생성한 객체에는 `.` 문법, 대괄호 문법, `Object.defineProperty`를 통해 속성을 할당할 수 있다.

```js
const driver = Object.create(person);
```

이렇게 상속하는 객체를 생성할 수도 있다.

클래스 내부에 함수를 정의하면, 객체가 생성될 때마다 새로 함수가 생성된다.

프로토타입 객체를 사용하면 동일한 프로토타입 객체를 사용하는 여러 개의 객체를 생성할 수 있고, 모든 인스턴스가 동일한 함수를 공유할 수 있다.

# 모듈 패턴

ES 모듈이 존재하지 않던 초기 자바스크립트에서는 객체 리터럴 표기법, 모듈 패턴, AMD 모듈, CJS 모듈을 통해 모듈을 구현했었다.

캡슐화를 위한 패턴이다.

### 비공개

클로저를 활용해 '비공개' 상태와 구성을 캡슐화한다. 공개 API만을 노출하고 나머지는 클로저 내부에 비공개로 유지한다.

애플리케이션이 사용하는 부분만 노출시키고, 핵심 작업은 보호한다. IIFE를 사용해 객체를 반환한다.

반환된 객체에 포함된 변수를 비공개하려면 `WeakMap`을 사용할 수 있다. `WeakMap`은 객체만 키로 설정할 수 있으며, 순회가 불가능하다. 해당 객체를 참조를 통해서만 모듈 내부에 접근할 수 있다.

### 예제

```js
let counter = 0;

const testModule = {
  incrementCounter() {
    return counter++;
  },
  resetCounter() {
    counter = 0;
  },
};

export default testModule;
```

`counter` 변수는 비공개 변수로 작동해 오직 메서드를 통해서만 접근할 수 있게 된다.

장바구니 로직의 예시:

```js
// 비공개 변수 및 함수
const basket = [];
const doSomethingPrivate = () => {};

const basketModule = {
    // 공개 API
    addItem(values) {
        basket.push(values);
    }
    getItemCount() {
        return basket.length;
    },
    doSomething() {
        doSomethingPrivate();
    }
    getTotal() {
        return basket.reduce((sum, item) => item.price + sum, 0);
    }
}

export default basketModule;
```

`basketModule`로 네임스페이스를 지정해 모듈을 생성했다.

- 비공개 자유성: 모듈 내부에서만 사용 가능한 비공개 함수를 자유롭게 생성할 수 있다.
- 디버깅 용이성: 어떤 함수가 예외를 발생시켰는지 디버거에서 콜 스택을 찾기 쉬워진다.

## 모듈 패턴의 변형

### 믹스인(Mixin) 가져오기 변형

외부 라이브러리 같은 전역 스코프의 요소를 모듈 내부의 고차 함수에 인자로 전달할 수 있게 한다.

### 내보내기 변형

따로 이름을 지정하지 않고 전역 스코프로 변수를 내보낸다.

```js
// module.js
const privateVar = 0;
const privateMethod = () => { ... }

const module = {
  publicProperty: "foo",
  publicMethod: () => {
    console.log(privateMethod);
  }
}

export default module;
```

### 장점

- 초보 개발자가 이해하기 쉬움
- 노출하고 싶은 변수나 함수만 노출할 수 있음

### 단점

- 공개, 비공개 멤버를 서로 다르게 접근해야 해서 번거로움
- 나중에 추가한 메서드에서는 비공개 멤버에 접근 불가
- 자동화 단위 테스트에서 비공개 멤버는 제외됨
- 비공개 멤버는 쉽게 수정하기 어려움

> 자바스크립트에서의 모듈 패턴 참고: https://adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html

### WeakMap을 사용하는 최신 모듈 패턴

WeakMap은 약한 참조를 가지는 Map이다. 객체만을 키로 받을 수 있다. 참조되지 않는 키는 GC의 대상이 된다. WeakMap은 DOM 요소에 추가 정보를 저장하거나, 모듈 패턴에서 비공개 데이터를 저장하는데 사용된다.

```js
const privateData = new WeakMap();

class Person {
  constructor(name, age) {
    this.name = name; // 공개 데이터
    // 'this'라는 인스턴스 객체를 키로 사용해 비공개 데이터 저장
    privateData.set(this, { age: age });
  }
  // 비공개 데이터에 접근하는 메서드
  getAge() {
    const data = privateData.get(this);
    return data.age;
  }
}

const john = new Person("John", 30);
console.log(john.name); // "John"
console.log(john.age); // undefined (직접 접근 불가)
console.log(john.getAge()); // 30 (메서드를 통해서만 접근 가능)

// 나중에 john 객체에 대한 참조가 모두 사라지면,
// privateData에 저장된 { age: 30 } 데이터도 자동으로 메모리에서 해제됩니다.
```

# 노출(Revealing) 모듈 패턴

모듈 패턴의 개선 버전이다. 공개 변수나 메서드에 접근하기 위해 메인 객체 이름을 반복해서 작성해야 하는 번거로움에 탄생했다.

모든 함수, 변수를 비공개 스코프에 정의하고, 공개하고 싶은 부분만 포인터를 통해 비공개 요소에 접근할 수 있게 하는 익명 객체를 반환하는 패턴이다. `import`, `export`로 공개 여부를 결정한다.

```js
let privateCounter = 0;
const privateFunction = () => {
  privateCounter++;
};

const publicFunction = () => {
  publicIncrement();
};
const publicIncrement = () => {
  privateFunction();
};
const publicGetCount = () => privateCounter;

// 비공개 함수와 속성에 접근하는 공개 포인터
const myRevealingModule = {
  start: publicFunction,
  increment: publicIncrement,
  count: publicGetCount,
};

export default myRevealingModule;

// 사용법:
import myRevealingModule from "./myRevealingModule";

myRevealingModule.start();
```

### 장점

- 코드의 일관성이 유지됨
- 공개 객체를 알아보기 쉽게 해 가독성이 좋아짐

### 단점

- 비공개 함수를 참조하는 공개 함수, 비공개 변수를 참조하는 공개 객체 멤버를 수정할 수 없음(만들어진 모듈의 내부 동작 방식을 수정하기 어려워 유연하지 않지 않은 패턴임 -> 이는 코드가 안정적이고 예측 가능하게 동작하게 하므로 트레이드오프라고 볼 수 있을 것.)

# 7.5 싱글톤 패턴

클래스의 인스턴스가 오직 하나만 존재하도록 제한하는 패턴이다. 시스템의 전역에서 접근하고 공유해야 하는 단 하나의 객체가 필요할 때 유용하다.

```js
let instance;

const privateMethod = () => { ... }
const privateVar = 'foo';
const randomNumber = Math.random();

class MySingleton {
  constructor() {
    if (!instance) {
      this.privateProperty = 'bar';
      instance = this;
    }
  }

  publicMethod() { ... }

  getRandomNumber() { return randomNumber; }
}

export default MySingleton;
```

💡 싱글톤 패턴의 적합성 검증

- 클래스의 인스턴스는 **정확히 하나만 존재**해야 하며 접근이 쉬워야 함
- 인스턴스는 서브클래싱을 통해서만 확장할 수 있어야 하고, 코드의 수정 없이 확장된 인스턴스를 사용할 수 있어야 함

두 번째는 특정 조건에서 확장된 인스턴스를 별도의 코드 수정 없이 생성할 수 있어야 한다는 의미이다.

```js
constructor() {
  // 1. 팀장이 아직 없으면 새로 뽑아야 하는데...
  if (this._instance == null) {
    // 2. 어떤 조건(isFoo)에 따라 A 팀장 뽑을지,
    if (isFoo()) {
      this._instance = new FooSingleton();
    } else {
    // 3. B 팀장을 뽑을지 결정한다.
      this._instance = new BasicSingleton();
    }
  }

  return this._instance;
}
```

싱글톤 패턴은 필요 시에 객체를 생성하는 "지연 실행" 방식을 택하고 있다. 이는 프로그램 실행 시 즉시 생성되는 정적 객체와 달리 초기화 시점을 제어할 수 있어 더 안전하다.

### 예제

```js
// options: 싱글톤의 구성을 담고 있는 객체를 뜻합니다.
// 예시: const options = { name: "test", pointX: 5 };
class Singleton {
  constructor(options = {}) {
    // 싱글톤에 속성을 할당합니다.
    this.name = "SingletonTester";
    this.pointX = options.pointX || 6;
    this.pointY = options.pointY || 10;
  }
}

// 인스턴스를 담을 변수
let instance;

// 정적 변수와 메서드의 구현
const SingletonTester = {
  name: "SingletonTester",
  // 인스턴스를 가져오는 메서드
  // 싱글톤 객체의 싱글톤 인스턴스를 반환합니다.
  getInstance(options) {
    if (instance === undefined) {
      instance = new Singleton(options);
    }
    return instance;
  },
};

const singletonTest = SingletonTester.getInstance({
  pointX: 5,
});

// 값을 확인하기 위해 pointX를 출력합니다.
console.log(singletonTest.pointX);
```

하지만, 객체를 생성하기 위해 클래스를 정의해야 하는 C++나 자바와 달리, 자바스크립트는 객체를 직접적으로 생성할 수 있다. 싱글톤 클래스 대신 객체 하나만 생성해주면 된다는 의미이다. 자바스크립트에서의 싱글톤 클래스 사용이 적합하지 않을 수도 있는 이유는 다음과 같다.

- 큰 모듈의 경우, 싱글톤임을 파악하기 어려움
- 싱글톤이 참조하는 의존성 파악이 어렵고, 복수의 인스턴스 생성이 어려우며, 의존성 대체가 어려운 등 테스트하기 어려움
- 전역 범위에 걸쳐 데이터를 관리하므로 유효하게 사용될 수 있도록 로직을 꼼꼼히 작성해야 함

**유용하지만, 사용에 어려움이 많은 패턴이므로 주의해야 한다.**

# 프로토타입 패턴

이미 존재하는 객체를 복제해 만든 템플릿을 기반으로 새 객체를 생성하는 패턴이다.

프로토타입 패턴은 프로토타입 상속을 기반으로 한다. 우선 프로토타입 역할을 할 전용 객체를 생성한다. 만들어진 `prototype` 객체는 생성자를 통해 만들어진 객체의 설계도가 된다.

클래스와는 다른 개념으로, 프로토타입 상속은 클래스처럼 따로 정의되는 것이 아닌, 이미 존재하는 다른 객체를 복제해 새로운 객체를 만들어 낸다.

자바스크립트에서는 `Object.create(prototype, optionalDescriptorObjects)`으로 인자로 들어온 프로토타입을 기반으로 객체를 생성할 수 있다.

⚠️ 객체의 속성을 나열할 때 부모로부터 물려 받은, 즉 프로토타입 속성까지 나열된다. 이는 `hasOwnProperty` 메서드로 객체가 직접 소유한 속성만 가져올 수 있다.

# 팩토리 패턴

다른 패턴과 달리 생성자를 필요로 하지 않고, 필요한 타입의 팩토리 객체를 생성하는 다른 방법을 제공한다.

필요한 타입과 속성을 념겨 원하는 UI 컴포넌트를 만드는 UI 팩토리가 대표적인 예시이다.

팩토리 패턴은 동적인 요소나 애플리케이션 구조에 깊게 의지하는 등의 **객체 생성 과정이 복잡할 때 유용**하다.

```js
// 1. 생산할 제품(객체)들
class Latte {
  constructor() {
    this.name = "라떼";
  }
  brew() {
    console.log(`${this.name}: 우유를 섞어 부드럽게 만듭니다.`);
  }
}

class Americano {
  constructor() {
    this.name = "아메리카노";
  }
  brew() {
    console.log(`${this.name}: 뜨거운 물을 추가하여 만듭니다.`);
  }
}

// 2. 주문에 따라 커피를 만드는 '공장'
function coffeeFactory(type) {
  switch (type) {
    case "latte":
      return new Latte();
    case "americano":
      return new Americano();
    default:
      throw new Error("알 수 없는 커피 종류입니다.");
  }
}

// 3. 사용자(클라이언트)는 공장에 주문만 하면 됨
// 사용자는 Latte나 Americano 클래스를 직접 알 필요가 없습니다.
const myCoffee = coffeeFactory("latte");
const yourCoffee = coffeeFactory("americano");

myCoffee.brew(); // 출력: 라떼: 우유를 섞어 부드럽게 만듭니다.
yourCoffee.brew(); // 출력: 아메리카노: 뜨거운 물을 추가하여 만듭니다.
```

### 사용하면 좋은 상황

- 객체나 컴포넌트의 생성 과정이 복잡할 때
- 상황에 맞는 다양한 객체 인스턴스를 편하게 생성하고 싶을 때
- 같은 속성을 공유하는 여러 개의 작은 객체 또는 컴포넌트를 다룰 때
- 덕 타이핑 같은 API 규칙만 충족하면 되는 다른 객체의 인스턴스와 함께 객체를 구성할 때

### 사용하면 안 되는 상황

단순히 `new MyObject()`로도 해결 가능하다면 굳이 팩토리 패턴을 적용하는 것은 좋지 않다.

객체 생성 과정을 인터페이스 뒤에 추상화하기 때문에 객체 생성 과정이 복잡할 경우 단위 테스트의 복잡성 또한 증가한다.

### 추상 팩토리 패턴

같은 목표를 가진 여러 팩토리를 하나의 그룹으로 캡슐화하는 패턴이다.

객체의 생성 과정에 영향을 받지 않아야 하거나 여러 타입의 객체로 작업해야 하는 경우 유용하다.

# 구조 패턴

- [퍼사드 패턴](#퍼사드-패턴)
- [믹스인 패턴](#믹스인-패턴)
- [데코레이터 패턴](#데코레이터-패턴)
- [플라이웨이트 패턴](#플라이웨이트-패턴)

# 퍼사드 패턴

심층적인 복잡성을 숨기고, 사용하기 편리한 high-level의 인터페이스를 제공하는 패턴이다.

자바스크립트 라이브러리에서 흔히 볼 수 있는 구조이다. 리액트의 컴포넌트 자체가 일종의 퍼사드라고 볼 수 있다. 컴포넌트 내부는 자체적인 상태 관리, 생명주기 로직, 복잡한 렌더링 조건, 스타일링 등 수많은 로직이 캡슐화되어 있다. 사용자는 이러한 복잡한 내부를 알 필요없이, 정의된 `props`라는 간단한 창구로 컴포넌트와 상호작용한다.

# 믹스인 패턴

전통적인 프로그래밍 언어에서 믹스인은 서브클래스가 쉽게 상속받아 공통 기능을 재사용할 수 있도록 하는 클래스다.

서브클래싱은 부모 클래스 객체에서 속성을 상속받아 새로운 객체를 생성하는 것을 말한다.

믹스인을 통해 최소한의 복잡성으로 객체의 기능을 빌리거나 상속받을 수 있다. 다른 여러 클래스와 속성과 메서드를 공유할 수도 있다.

자바스크립트의 `extends` 절은 클래스나 생성자를 반환하는 표현식을 허용한다. 이를 통해 동적으로 부모 클래스를 받아 확장하는 믹스인 함수를 정의할 수 있다.

```js
const MyMixins = superclass =>
  class extends superclass {
    moveUp() { ... }
    moveDown() { ... }
    stop() { ... }
  }

class CarAnimator() {
  contructor() {
    moveLeft() { ... }
  }
}

class MyAnimator extends MyMixins(CarAnimator) {};

const myAnimator = new MyAnimator();
myAnimator.moveLeft();
myAnimator.stop();
```

### 장점과 단점

- 함수의 중복을 줄이고 재사용성을 높여줌
- 하지만, 리액트 개발 팀을 포함한 몇몇 개발자는 클래스나 객체의 프로토타입에 기능을 주입하는 것이 프로토타입의 오염과 함수의 출처에 대한 불확실성을 초래하기 때문에 선호하지 않음.

# 데코레이터 패턴

데코레이터 패턴은 코드 재사용을 목표로 하는 패턴이다. 믹스인과 마찬가지로 객체 서브클래싱의 다른 방법이다.

이미 만들어진 객체에 데코레이터(추가 속성, 메서드)를 붙여서 기능을 확장한다.

자바스크립트에서는 인스턴스에 `.`으로 속성, 메서드를 추가하거나 부모의 클래스를 받아 기능을 확장하는 식으로 구현할 수 있다.

# 플라이웨이트 패턴

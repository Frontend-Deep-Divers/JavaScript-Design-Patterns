# 7장 자바스크립트 디자인 패턴

## 💡 어떤 패턴을 선택해야 하는가?

최고의 패턴은 무엇인가에 대한 정답은 없다. 각 코드와 애플리케이션이 요구하는 부분은 다르기 때문이다. 대신 **패턴이 실질적인 구현에 도움이 되는지 고려**해야 한다.

때문에 디자인 패턴에 대해 정확히 알고 사용할 때를 잘 파악한다면 올바른 디자인 패턴을 적용할 수 있다!

---

# 생성 패턴

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

## 의사 클래스 데코레이터

데코레이터 패턴을 같은 인터페이스를 가지는 서로 다른 객체 내부에 새 객체를 넣어서 사용하는 방법으로 설명하는 방식이다.

자바스크립트는 인터페이스를 지원하지 않기에 직접 구현하거나, 타입스크립트를 사용할 수 있다.

> 자바스크립트에서의 인터페이스 구현: https://cryingnavi.github.io/javascript/2021/06/05/JS.html

## 추상 데코레이터

필요한 기본 메서드를 정의하고, 나머지 옵션은 서브클래스가 된다. 자바의 추상 클래스와 같은 방식이다.

### 장점과 단점

- 베이스 객체가 변경될 걱정없이 기능을 확장할 수 있고, 서브클래스에 의존하지도 않음.
- 하지만, 네임 스페이스에 작고 비슷한 객체를 지속적으로 추가하기에, 잘 관리하지 않으면 애플리케이션의 구조가 복잡해질 수 있음.
- 패턴에 익숙하지 않은 다른 개발자가 패턴의 사용 목적을 파악하기 어려울 수 있음. (문서화로 해결 가능)

# 플라이웨이트 패턴

반복되고 느리며 비효율적으로 데이터 공유가 이루어지는 코드를 최적화하기 위해 만들어졌다.

연관된 객체끼리 데이터를 공유하게 하면서 애플리케이션의 메모리를 최소화하기 위한 패턴이다. **객체의 내부 상태와 외부 상태를 분리**하는 것이 패턴의 핵심이다.

## 사용법

데이터 레이어에서 메모리 저장된 비슷한 객체 사이로 데이터를 공유하는 방법이 있다. 또한 DOM을 사용할 때, 각각의 요소에 이벤트 핸들러를 붙이지 않고 부모 요소에 등록하는 방법이 이에 해당한다.

- 내재적 상태: 객체의 내부 메서드에 필요한 것으로, 없으면 동작하지 않는 것을 의미한다. 같은 내재적 정보를 지닌 객체는 팩토리 메서드를 사용해 만들어진 하나의 공유된 객체로 대체할 수 있다.
- 외재적 상태: 제거되어 외부에 저장될 수 있다. 이를 다룰 땐 관리자를 사용하여 데이터를 공유한다. 플라이웨이트 객체와 내재적 상태를 보관하는 중앙 DB를 관리자로 사용하는 방법이 있다.

## 전통적인 플라이웨이트 구현 예제

- 플라이웨이트: 외부의 상태를 받아 작동하게 하는 인터페이스
- 구체적 플라이웨이트: 인터페이스를 실제로 구현하고 내부 상태를 저장한다. 다양한 컨텍스트 사이에서 공유할 수 있어야 하며, 외부 상태를 조작할 수 있어야 함
- 플라이웨이트 팩토리: 플라이웨이트 객체를 생성하고 관리

```js
// CoffeeOrder 인터페이스
const CoffeeOrder = {
  serveCoffee(context) {},
  getFlavor() {},
};

class CoffeeFlavor extends InterfaceImplementation {
  constructor(newFlavor) {
    super();
    this.flavor = newFlavor;
  }

  getFlavor() {
    return this.flavor;
  }

  serveCoffee(context) {
    console.log(
      `Serving Coffee flavor ${this.flavor} to table ${context.getTable()}`
    ); // 커피 제공 로그
  }
}

// CoffeeOrder 인터페이스 구현
CoffeeFlavor.implementsFor(CoffeeOrder); // 자바의 implements에 해당

const CoffeeOrderContext = (tableNumber) => ({
  getTable() {
    return tableNumber;
  },
});

class CoffeeFlavorFactory {
  constructor() {
    this.flavors = {};
    this.length = 0;
  }

  getCoffeeFlavor(flavorName) {
    let flavor = this.flavors[flavorName];
    if (!flavor) {
      flavor = new CoffeeFlavor(flavorName);
      this.flavors[flavorName] = flavor;
      this.length++;
    }
    return flavor;
  }

  getTotalCoffeeFlavorsMade() {
    return this.length;
  }
}

// 사용 예시:
const testFlyweight = () => {
  const flavors = [];
  const tables = [];
  let ordersMade = 0;
  const flavorFactory = new CoffeeFlavorFactory();

  function takeOrder(flavorIn, table) {
    flavors.push(flavorFactory.getCoffeeFlavor(flavorIn));
    tables.push(new CoffeeOrderContext(table));
    ordersMade++;
  }

  // 주문 처리
  takeOrder("Cappuccino", 2);
  // ...

  // 주문 제공
  for (let i = 0; i < ordersMade; ++i) {
    flavors[i].serveCoffee(tables[i]);
  }

  console.log("-");
  console.log(
    `total CoffeeFlavor objects made: ${flavorFactory.getTotalCoffeeFlavorsMade()}`
  );
};

testFlyweight();
```

## 플라이웨이트 패턴과 DOM 객체

이벤트 버블링 과정을 조정하는 데 사용할 수 있다. 중첩된 요소의 부모 요소에 이벤트를 바인딩해 메모리를 절약한다.

```html
<div id="container">
  <div class="toggle">
    More Info (Address)
    <span class="info"> This is more information </span>
  </div>
  <div class="toggle">
    Even More Info (Map)
    <span class="info">
      <iframe src="MAPS_URL"></iframe>
    </span>
  </div>
</div>

<script>
  (function () {
    const stateManager = {
      fly() {
        const self = this;
        $("#container")
          .off()
          .on("click", "div.toggle", function () {
            self.handleClick(this);
          });
      },
      handleClick(elem) {
        $(elem).find("span").toggle("slow");
      },
    };

    // 이벤트 리스너 초기화
    stateManager.fly();
  })();
</script>
```

# 행위 패턴

- 관찰자 패턴
- 중재자 패턴
- 커맨드 패턴

# 관찰자 패턴

한 객체가 변경될 때 다른 객체들에 변경되었음을 알리는 패턴이다. 변경된 객체는 누가 자신을 구독하는지 알 필요 없이 알림을 보낼 수 있다.

한 객체(주체)를 관찰하는 여러 객체들(관찰자)이 존재하고, 주체가 변경되면 관찰자들에게 자동으로 알림을 보낸다.

리액트에서는 상태 변화를 컴포넌트에 알리기 위해 관찰자 패턴을 사용한다.

주체가 `notify` 메서드를 실행하면, 구독 중인 모든 관찰자에 알림을 보내고, 관찰자 내부에서는 `update` 메서드가 실행된다.

```js
// 주체가 가질 수 있는 관찰자 목록
class ObserverList {
  constructor() {
    this.observerList = [];
  }

  add(obj) {
    return this.observerList.push(obj);
  }

  count() {
    return this.observerList.length;
  }

  get(index) {
    if (index > -1 && index < this.observerList.length) {
      return this.observerList[index];
    }
  }

  indexOf(obj, startIndex) {
    let i = startIndex;
    while (i < this.observerList.length) {
      if (this.observerList[i] === obj) {
        return i;
      }
      i++;
    }
    return -1;
  }

  removeAt(index) {
    this.observerList.splice(index, 1);
  }
}

class Subject {
  constructor() {
    this.observers = new ObserverList();
  }

  addObserver(observer) {
    this.observers.add(observer);
  }

  removeObserver(observer) {
    this.observers.removeAt(this.observers.indexOf(observer, 0));
  }

  notify(context) {
    const observerCount = this.observers.count();
    for (let i = 0; i < observerCount; i++) {
      this.observers.get(i).update(context);
    }
  }
}

class Observer {
  constructor() {}
  update() { ... }
}
```

## 관찰자 패턴과 발행/구독 패턴의 차이점

실제 자바스크립트 환경에서는 발행/구독 패턴이라는 변형된 형태의 구현이 더 널리 사용된다. 그 이유는 ECMAScript 구현체는 본질적으로 이벤트 기반이기 때문이다. 관찰자 패턴보다 느슨한 결합을 가지는 것이 특징이다.

관찰자 패턴에서는 관찰자 객체가 주체 객체에 알림 대상으로서 등록된다. 하지만, 발행/구독 패턴에서는 이벤트 구독자와 이벤트를 발생시키는 발행자 사이에 토픽/이벤트 채널을 둔다. 발행자와 구독자를 각자 독립적으로 유지시킨다. 이로써 애플리케이션에 특화된 이벤트를 정의할 수 있고, 구독자에게 필요한 값이 포함된 커스텀 인자를 전달할 수 있게 된다.

발행자가 구독자의 메서드를 직접 호출하는 대신, 구독자는 특정 작업이나 활동을 구독하고 해당 작업이나 활동이 발생하면 알림을 받게 된다.

```js
<div class="messageSender"></div>
<div class="messagePreview"></div>
<div class="messageCounter"></div>

// 간단한 발행/구독 패턴 구현
const events = (function () {
    const topics = {};
    const hOP = topics.hasOwnProperty;

    return {
        subscribe: function (topic, listener) {
            if (!hOP.call(topics, topic)) topics[topic] = [];

            const index = topics[topic].push(listener) - 1;

            return {
                remove: function () {
                    delete topics[topic][index];
                },
            };
        },
        publish: function (topic, info) {
            if (!hOP.call(topics, topic)) return;

            topics[topic].forEach(function (item) {
                item(info !== undefined ? info : {});
            });
        }
    };
})();

// 매우 간단한 새 메일 핸들러
// 받은 메시지 수를 세는 카운터 변수
let mailCounter = 0;

// "inbox/newMessage" 라는 이름의 토픽을 구독하는 구독자를 초기화

// 새 메시지의 미리보기를 렌더링
const subscriber1 = events.subscribe('inbox/newMessage', (data) => {
    // 디버깅을 위해 토픽을 출력
    console.log('A new message was received:', data);

    // 전달받은 데이터를 사용해 사용자에게 메시지 미리보기를 보여줌.
    document.querySelector('.messageSender').innerHTML = data.sender;
    document.querySelector('.messagePreview').innerHTML = data.body;
});

// 발행자를 통해 새 메시지를 표시하는 카운터 업데이트
const subscriber2 = events.subscribe('inbox/newMessage', (data) => {
    document.querySelector('.messageCounter').innerHTML = ++mailCounter;
});

events.publish('inbox/newMessage', {
    sender: 'hello@google.com',
    body: 'Hey there! How are you doing today?',
});

// 나중에 구독자가 새 토픽에 대한 알림을 받고 싶지 않으면
// 다음과 같이 구독을 취소
// subscriber1.remove();
// subscriber2.remove();
```

### 장점과 단점

- 애플리케이션의 요소들의 관계를 고민해보는 데 도움이 되고, 구성 요소들이 느슨하게 결합할 수 있도록 도와줌. 또한, 클래스를 강하게 결합시키지 않으면서 관련 객체들 사이의 일관성을 유지할 수 있음.
- 하지만, 그로 인해 애플리케이션의 각 부분들이 제대로 동작하는 지 파악하기 어려울 수 있음. 또한, 구독자와 발행자의 관계가 동적으로 결졍되므로 의존 관계를 추적하기가 까다로울 수 있음.

## 발행/구독 패턴 예제

```js
class PubSub {
  constructor() {
    // 알림을 보내거나 받을 수 있는 토픽을 저장
    this.topics = {};
    // 토픽 식별자
    this.subUid = -1;
  }

  publish(topic, args) {
    if (!this.topics[topic]) {
      return false;
    }

    const subscribers = this.topics[topic];
    let len = subscribers ? subscribers.length : 0;

    while (len--) {
      subscribers[len].func(topic, args);
    }

    return this;
  }

  subscribe(topic, func) {
    if (!this.topics[topic]) {
      this.topics[topic] = [];
    }

    const token = (++this.subUid).toString();
    this.topics[topic].push({
      token: token,
      func: func,
    });
    return token;
  }

  unsubscribe(token) {
    for (const m in this.topics) {
      if (this.topics[m]) {
        for (let i = 0, j = this.topics[m].length; i < j; i++) {
          if (this.topics[m][i].token === token) {
            this.topics[m].splice(i, 1);
            return token;
          }
        }
      }
    }
    return this;
  }
}

const pubsub = new PubSub();

pubsub.publish("/addFavorite", ["test"]);
pubsub.subscribe("/addFavorite", (topic, args) => {
  console.log("test", topic, args);
});
```

# 중재자 패턴

하나의 객체가 이벤트 발생 시 다른 여러 객체들에게 알림을 보낼 수 있는 패턴이다.

관찰자 패턴이 하나의 객체가 다른 객체에서 발생하는 이벤트를 구독할 수 있도록 하는 반면, 이 패턴은 하나의 객체가 다른 객체에서 발생한 특정 유형의 이벤트에 대해 알림을 받을 수 있다.

중재자 패턴에서 중재자는 **항공 교통 관제 시스템에서의 관제탑**으로 생각할 수 있다. 항공기의 모든 통신(이벤트)가 관제탑(중재자)을 거쳐 이루어지고, 항공기끼리는 직접 통신하지 않고 항공기의 이착륙이 관리된다.

## 예제

간단한 중재자는 한 줄의 코드로도 구현할 수 있다.

```js
const mediator = {};
```

의미적으로 봤을 때, 중재자는 객체 간의 워크플로우를 제어하는데, 이를 위해 객체 리터럴 이상의 복잡한 구조를 필요로 하지 않는다.

아래는 이벤트를 발생시키고 구독할 수 있는 몇 가지 유틸리티 메서드를 가진 기본적인 중재자 객체의 구현이다. 이 예제는 '신규 직원 등록'이라는 워크플로우를 `orgChart`라는 중재자를 통해 처리한다.

```js
const orgChart = {
  addNewEmployee() {
    // getEmployeeDetail이 사용자가 상호작용하는 뷰를 제공
    const employeeDetail = this.getEmployeeDetail();

    // 직원 정보 입력이 완료되면,
    // 중재자('orgchart' 객체)가 다음 행동을 결정
    employeeDetail.on("complete", (employee) => {
      // 추가할 이벤트를 가진 객체를 추가하고,
      // 중재자가 추가적인 작업을 하도록 설정
      const managerSelector = this.selectManager(employee);
      managerSelector.on("save", (employee) => {
        employee.save();
      });
    });
  },

  // ...
};
```

이 과정에서 `employeeDetail` 객체는 `managerSelector` 객체의 존재를 전혀 알지 못합니다. 모든 흐름 제어와 객체 간의 상호작용은 오직 `orgChart` 중재자를 통해서만 이루어진다.

## 중재자 패턴과 이벤트 집합(발행/구독) 패턴 비교

둘 다 '이벤트'와 '서드 파티 객체'를 유사점으로 가진다.

### 이벤트

중재자 패턴은 자바스크립트에서 구현을 단순히 하기 위해 이벤트를 사용할 뿐, 반드시 이벤트를 다룰 필요는 없다. 이벤트 집합 패턴은 그 자체로 이벤트를 처리하기 위한 목적을 가지고 있다.

### 서드 파티 객체

객체 간 상호작용을 간소화하기 위해 모두 서드 파티 객체를 사용한다.

이벤트 집합 패턴에서 서드 파티 객체(이벤트 채널)는 두 객체(소스와 핸들러) 사이에서 이벤트가 연결되도록 지원하는 역할만 수행한다.

하지만, 중재자 패턴에서는 비즈니스 로직과 워크플로우가 중재자 내부에 집중된다. 중재자는 직접 각 객체의 메서드 호출 시점과 속성 업데이트의 필요성을 판단해 워크플로우와 프로세스를 캡슐화하고 여러 객체를 조율해 시스템을 동작시킨다.

또, 이벤트 집합 패턴은 구독자가 존재하든, 존재하지 않든 이벤트를 발행시킨 후 처리를 위임한다. 반면 중재자 패턴은 미리 설정해 둔 특정 입력 또는 활동에 주목해 객체 사이의 행동을 직접 조율한다.

## 이벤트 집합 패턴의 활용

직접적인 구독 관계가 많아질 경우 또는 전혀 관련 없는 객체들 간의 소통이 필요할 때 사용한다.

부모 뷰와 자식 뷰가 있다고 가정했을 때, 자식 뷰가 발생시킨 이벤트에 부모 뷰의 이벤트가 처리되는 경우, 이벤트 집합 패턴을 활용하면 이점이 있다.

구체적인 예시: jQuery의 `on` 메서드, 리액트의 Context API

## 중재자 패턴의 활용

두 개 이상의 객체가 간접적인 관계를 가지고 있고 비즈니스 로직이나 워크플로우에 따라 상호작용 및 조정이 필요한 경우 유용하다.

## 이벤트 집합 패턴 + 중재자 패턴

```js
const MenuItem = MyFrameworkView.extend({
  events: {
    "click .thatThing": "clickedIt",
  },

  clickedIt(e) {
    e.preventDefault();

    // "menu:click:foo"를 실행한다고 가정
    MyFramework.trigger("menu:click:foo", { name: this.model.get("name") });
  },
});

// 애플리케이션의 다른 곳에서 구현
class MyWorkflow {
  constructor() {
    MyFramework.on("menu:click:foo", this.doStuff, this);
  }

  static doStuff() {
    // 이곳에 여러 객체를 인스턴스화
    // 객체의 이벤트 핸들러 설정
    // 모든 객체를 의미 있는 워크플로로 조정
  }
}
```

이벤트 집합 패턴: 전역적인 알림을 담당한다. `MenuItem` 같은 컴포넌트는 자신의 구체적인 행동(`'menu:click:foo'` 이벤트 발행)이 어떤 결과를 낳을지 전혀 알 필요가 없다. 그저 시스템 전체에 "메뉴가 클릭되었다"고 알리기만 한다.

중재자 패턴: 복잡한 워크플로우를 관리한다. `MyWorkflow` 같은 중재자 객체는 특정 전역 이벤트를 구독하고 있다가, 해당 이벤트가 발생하면 여러 객체 간의 상호작용을 순서대로 지휘하고 조정하는 복잡한 작업을 처리한다.

# 커맨드 패턴

메서드 호출, 요청 또는 작업을 단일 객체로 캡슐화하여 추후에 실행할 수 있도록 해주는 패턴이다. 명령을 실행하는 객체와 명령을 호출하는 객체 간의 결합을 느슨하게 해 구체적인 클래스(객체)의 변경에 대한 유연성을 향상시킨다.

명령을 내리는 객체와 명령을 실행하는 객체의 책임을 분리하는 것이 핵심이다.

## 예제

'실행할 동작'과 '해당 동작을 호출할 객체'를 연결하여 구현한다. 실행을 위한 동작(`run()` 또는 `execute()`)를 포함한다.

```js
const CarManager = {
  // 정보 조회
  requestInfo(model, id) {
    return `The information for ${model} with ID ${id} is foobar`;
  },

  // 자동차 구매
  buyVehicle(model, id) {
    return `You have successfully purchased Item ${id}, a ${model}`;
  },

  // 시승 신청
  arrangeViewing(model, id) {
    return `You have booked a viewing of ${model} ( ${id} )`;
  },
};
```

`CarManager` 객체는 각종 명령을 실행하는 커맨드 객체다. 객체 내부의 핵심 API가 변경되면, 메서드를 직접 호출하는 애플리케이션 내 모든 객체를 수정해야 하므로 `CarManager`와 이를 사용하는 클라이언트 코드들 간의 강한 결합이 있다. 이 문제는 API를 추상화함으로써 해결할 수 있다.

```js
carManager.execute = function (name) {
  // name = 'buyVehicle'
  return (
    carManager[name] &&
    carManager[name].apply(carManager, [].slice.call(arguments, 1))
  );
};

carManager.excute("buyVehicle", "Ferrari", "14523");
```

객체를 확장하여 실행할 수 있는 메서드의 이름과 데이터를 매개변수로 받아 처리하는 구조로 개선했다.

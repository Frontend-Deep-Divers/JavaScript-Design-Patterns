# 생성 패턴

## 생성자 패턴

객체가 새로 만들어진 뒤 초기화하는 특별한 메서드

ES2015 이후로 생성자를 가진 클래스를 만들 수 있음

### ~~프로토타입을 가진 생성자~~

```jsx
~~class Car {
	constructor() {}
	
	// 이 메서드는 Car 인스턴스가 생성될 때 마다 생성됨
	toString() {}
}

// 프로토타입으로 메서드를 생성해주면 효율적임
Car.prototype.toString = function() {}~~
```

## 모듈 패턴

초기 자바스크립트는 다음과 같은 방법으로 모듈을 구현함

- 객체 리터럴 표기법
    - 객체 리터럴(`{ … }`) 을 활용해 모듈처럼 활용
- 모듈 패턴
- AMD
- CJS

### **모듈 패턴**

- 다른 애플리케이션이 사용해야 하는 부분만 노출하고, 핵심 작업은 보호(클로저)
- 객체를 내보내서 네임스페이스로 활용
- 믹스인 가져오기 변형: 전역 스코프에 있는 요소를 고차 함수에 인자로 전달하여 마음대로 이름을 지정할 수 있음
    
    ```jsx
    // util.js
    export const privateMethod = () => { ... };
    
    // Module.js
    import { privateMethod } from './util.js';
    
    const myModule = () => ({
    	publicMethod() {
    		privateMethod();
    	}
    });
    
    export default myModule;
    
    // main.js
    import myModule from './Module.js'
    
    const moduleInstance = myModule();
    moduleInstance.publicMethod();
    ```
    
- 단점
    - 나중에 추가한 메서드에서는 비공개 멤버에 접근할 수 없음
    - 비공개 멤버를 수정하는 것은 어려움
- WeakMap을 활용한 모듈 패턴도 존재함 ⇒ ES2022 이후부터 #private 필드로 거의 대체됨
    
    ```jsx
    const privateData = new WeakMap();
    
    class Counter {
      constructor() {
        privateData.set(this, { count: 0 });
      }
    
      increment() {
        const data = privateData.get(this);
        data.count += 1;
      }
    
      decrement() {
        const data = privateData.get(this);
        data.count -= 1;
      }
    
      getCount() {
        return privateData.get(this).count;
      }
    }
    
    export default Counter;
    ```
    

## 노출 모듈 패턴

메인 객체의 이름을 반복해서 사용해야 하는 불편함 때문에 등장

모든 함수와 변수를 비공개 스코프에 정의하고 공개하고 싶은 부분만 포인터를 통해 공개

```jsx
const privateVar = 1;
const publicVar = 2;

const privateGetter = () => privateVar;

const publicGetter = () => { privateGetter() };

const myRevealingModule = {
	publicVar,
	publicGetter,
};

export default myRevealingModule;
```

비공개 함수를 참조하는 공개 함수를 수정할 수 없음

## 싱글톤 패턴

```jsx
let instance;

class MySingleton {
	constructor() {
		if(!instance) {
			this.publicProperty = 'public';
			instance = this;
		}
		
		return instance;
	}
	
	publicMethod() { ... }
}

export default MySingleton;
```

싱글톤 패턴은 지연된 실행(필요할 때 인스턴스 생성)이 중요함

자바스크립트는 객체를 리터럴로 생성할 수 있기 때문에 싱글톤 적용이 정말 필요한 지 생각해봐야 함

## 프로토타입 패턴

이미 존재하는 객체를 복제해 만든 템플릿을 기반으로 새 객체를 생성

자바스크립트만이 가진 고유의 방식으로 작업 가능

`Object.create`를 활용해 구현 가능

## 팩토리 패턴

생성자가 아닌 객체를 생성하는 방법을 제공

```jsx
class Dog {
  speak() {
    console.log('Woof!');
  }
}

class Cat {
  speak() {
    console.log('Meow!');
  }
}

class AnimalFactory {
	constructor() {
		this.animalClass = Dog;
	}
	
	createAnimal(options) {
		const { animalType, ...rest } = options;
		
		switch(animalType) {
			case 'dog':
				this.animalClass = Dog;
				break;
			case 'cat':
				this.animalClass = Cat;
				break;
		}
		
		return new this.animalClass(rest);
	}
}
```

서브 클래스를 통해 수정 가능

다음과 같은 경우에 유용

- 생성 과정이 복잡할 때
- 상황에 맞춰 다양한 객체 인스턴스를 편리하게 생성해야할 때

객체 생성 인터페이스 제공이 목표가 아니라면 사용하지 않는 것이 좋음 → 테스트의 복잡성이 증가함

추상 팩토리 패턴: 같은 목표를 가진 각각의 팩토리들을 하나의 그룹으로 캡슐화

# 구조 패턴

클래스와 객체를 체계적으로 구성하는 방법

## 퍼사드 패턴

실제 모습을 숨기고 편리한 인터페이스를 제공

![스크린샷 2025-07-27 오전 11.51.18.png](attachment:f887c6d7-1071-4f59-a2b2-eca1e409ca1a:스크린샷_2025-07-27_오전_11.51.18.png)

jQuery가 대표적 예시

## 믹스인 패턴

서브클래스가 상속받아 기능을 재사용할 수 있도록 하는 클래스

- 서브 클래스
- 믹스인
    
    ```jsx
    const MyMixin = superClass => (
    	class extends superClass {
    		moveUp() {}
    		moveDown() {}
    	}
    );
    
    class OldAnimator {
    	moveLeft() {}
    }
    
    class MyAnimator extends MyMixin(OldAnimator) {}
    
    const myAnimator = new MyAnimator();
    myAnimator.moveUp();
    myAnimator.moveLeft();
    ```
    
    불확실성이 생기기 때문에 반대하는 경우가 많음
    

## 데코레이터 패턴

믹스인과 마찬가지로 서브클래싱의 다른 방법

기존 클래스에 동적으로 기능을 추가하기 위해 사용

```jsx
class MacBook {
	constructor() {
		this.cost = 500;
		this.screenSize = 11.6;
	}
	
	getCost() {
		return this.cost;
	}
}

class Memory extends MacBook {
	constructor(mb) {
		super();
		this.mb = mb;
	}
	
	getCost() {
		return this.mb.getCost() + 50;
	}
}

class Insurance extends MacBook {
	constructor(mb) {
		super();
		this.mb = mb;
	}
	
	getCost() {
		return this.mb.getCost() + 100;
	}
}

let mb = new MacBook();

mb = new Memory(mb);
mb = new Insuracne(mb);

console.log(mb.getCost());
// 650
```

## 의사 클래스 데코레이터

데코레이터의 변형

인터페이스를 활용

```jsx
// 1. Interface 생성자 정의 (사전 정의된 클래스)
class Interface {
  constructor(name, methods) {
    this.name = name;
    this.methods = methods;
  }

  // 메서드 구현 확인용 정적 메서드
  static ensureImplements(object, interfaceInstance) {
    for (let method of interfaceInstance.methods) {
      if (!object[method] || typeof object[method] !== 'function') {
        throw new Error(`The method ${method} is not implemented.`);
      }
    }
  }
}

// 2. 지원해야 할 메서드를 정의한 Interface 인스턴스 생성
const reminder = new Interface('List', ['summary', 'placeOrder']);

// 3. 이 인터페이스를 구현하는 객체를 정의
const properties = {
  name: 'Remember to buy the milk',
  date: '05/06/2040',
  actions: {
    summary() {
      return 'Remember to buy the milk, we are almost out!';
    },
    placeOrder() {
      return 'Ordering milk from your local grocery store';
    }
  }
};

// 4. Todo 클래스 정의
class Todo {
  constructor({ actions, name }) {
    // 인터페이스 검증
    Interface.ensureImplements(actions, reminder);

    this.name = name;
    this.methods = actions;
  }
}

// 5. Todo 인스턴스 생성 및 테스트
const todoItem = new Todo(properties);

// 6. 동작 확인
console.log(todoItem.methods.summary());
// 출력: Remember to buy the milk, we are almost out!

console.log(todoItem.methods.placeOrder());
// 출력: Ordering milk from your local grocery store
```

### 추상 데코레이터

하나의 데코레이터 클래스를 만들어 두고 이를 상속받는 구체적인 데코레이터 클래스를 정의해서 사용

### 데코레이터 패턴의 장단점

장점: 베이스 객체가 수정될 걱정 없이 사용 가능, 수 많은 서브 클래스를 만들 필요 없음

단점: 앱의 구조가 복잡해질 수 있음, 다른 개발자가 파악하기 힘듦

## 플라이웨이트 패턴

반복되고 느리고 비효율적으로 데이터를 공유하는 코드를 최적화

연관된 객체끼리 데이터를 공유

### 사용법

1. 데이터 레이어에서 메모리에 저장된 비슷한 객체 사이로 데이터를 공유

### 데이터 공유

- 내재적 상태: 객체의 내부 메서드에 필요한 것, 없으면 동작하지 않음
- 외재적 상태: 외부에 저장될 수 있음

같은 내재적 데이터를 가진 객체를 팩토리 메서드를 사용해 하나의 공유된 객체로 대체 가능

외재적 데이터는 관리자를 사용

```jsx
class Circle {
  constructor(color) {
    this.color = color; // 내부 상태
  }

  draw({ id, x, y, radius }) {
    console.log(`(${id}) Draw a ${this.color} circle at (${x}, ${y}) with radius ${radius}`);
  }
}

class CircleFactory {
  constructor() {
    this.circles = {}; // color별로 공유된 Circle 저장
  }

  getCircle(color) {
    if (!this.circles[color]) {
      this.circles[color] = new Circle(color);
    }
    return this.circles[color];
  }
}

const factory = new CircleFactory();

// 고유한 외부 상태는 따로 관리
const circleData = [
  { id: 'A1', color: 'red', x: 10, y: 20, radius: 5 },
  { id: 'A2', color: 'red', x: 30, y: 40, radius: 10 },
  { id: 'B1', color: 'blue', x: 50, y: 60, radius: 8 },
];

for (const data of circleData) {
  const sharedCircle = factory.getCircle(data.color); // 내부 상태 공유
  sharedCircle.draw(data); // 외부 상태는 개별 전달
}
```

### 예시: 중앙 집중식 이벤트 핸들링

모든 하위 요소에 이벤트를 추가하지 않고, 상위 요소에만 이벤트 핸들러를 추가해 어떤 하위 요소에서 부터 이벤트가 버블링되었는지 체크한 뒤 처리하는 방식 (이벤트 위임)

# 행위 패턴

## 관찰자 패턴

한 객체가 변경될 때 다른 객체들에 변경되었음을 알림

Subject를 관찰하는 Objects

### 발행/구독 패턴

실제 자바스크립트 환경에서는 발행/구독 패턴이라는 변형된 형태의 구현이 더 널리 사용됨

이벤트 알림을 원하는 Subscriber와 이벤트를 발생시키는 Publisher 사이에 이벤트 채널을 둠

Subscriber와 Publisher를 독립적으로 유지

직접적인 구독 관계가 많아질 경우 또는 전혀 관련 없는 객체들 간의 소통이 필요할 때 사용

### 장단점

클래스를 강하게 결합시키지 않으면서 객체들 사이의 일관성을 유지할 수 있음 → 결합도 낮춤

Subject와 Object 사이에 동적인 관계 형성 → 유연성 확보

Subscriber(Object)의 기능이 중단되어도 Publisher(Subject)가 알 방법이 없음

## 중재자 패턴

하나의 객체가 이벤트 발생 시 다른 객체들에게 알림을 보냄

클래스 간의 관계를 관리함으로써 직접 참조를 없애고 느슨한 결합을 가능하게 함

### 발행/구독 패턴과 차이점

발행/구독 패턴은 이벤트를 사용하지만, 중재자 패턴은 필수가 아님

발행/구독 패턴의 서드 파티 객체는 연결시키는 역할만 함, 로직은 발행자와 구독자에 직접 구현됨 → 발행 후 망각

중재자 패턴에서 로직은 서드 파티 객체 내부에 집중됨

두 개 이상의 객체가 간접적인 관계를 가지고 있고 로직에 따라 상호작용 및 조정이 필요한 경우에 사용

## 커맨트 패턴

메서드 호출, 요청 또는 작업을 단일 객체로 캡슐화하여 실행

클래스의 변경에 대한 유연성 향상

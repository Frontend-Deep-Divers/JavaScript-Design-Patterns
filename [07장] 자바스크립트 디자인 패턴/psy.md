## 생성 패턴

### 생성자 패턴

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

### 모듈 패턴

초기 자바스크립트는 다음과 같은 방법으로 모듈을 구현함

- 객체 리터럴 표기법
    - 객체 리터럴(`{ … }`) 을 활용해 모듈처럼 활용
- 모듈 패턴
- AMD
- CJS

**모듈 패턴**

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
    

### 노출 모듈 패턴

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

### 싱글톤 패턴

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

### 프로토타입 패턴

이미 존재하는 객체를 복제해 만든 템플릿을 기반으로 새 객체를 생성

자바스크립트만이 가진 고유의 방식으로 작업 가능

`Object.create`를 활용해 구현 가능

### 팩토리 패턴

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

## 구조 패턴

클래스와 객체를 체계적으로 구성하는 방법

### 퍼사드 패턴

실제 모습을 숨기고 편리한 인터페이스를 제공

<img width="535" height="412" alt="스크린샷 2025-07-27 오전 11 51 18" src="https://github.com/user-attachments/assets/f5e1448f-baae-410b-bfb5-77168423b623" />


jQuery가 대표적 예시

### 믹스인 패턴

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
    

### 데코레이터 패턴

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

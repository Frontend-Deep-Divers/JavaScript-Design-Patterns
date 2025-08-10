## 고차 컴포넌트

여러 컴포넌트에서 동일한 로직을 사용하고 싶을 때 (로더 구현 시)

다른 컴포넌트를 인자로 받아 새로운 컴포넌트 반환

```jsx
function withStyles(Component) {
	return props => {
		const style = { padding: '0.2rem' };
		
		return <Component style={style} {...props} >
	}
}
```

재사용하고자 하는 로직을 한 곳에 모아 관리 → 관심사 분리

고차 컴포넌트와 대상 컴포넌트의 prop 이름 충돌 가능 → 디버깅, 확장 어려움

### 고차 컴포넌트 조합

withB로 withA를 감싸서 사용

## 렌더링 Props 패턴

컴포넌트를 재사용하는 또 다른 방법

JSX를 반환하는 함수를 값으로 가지는 prop, 을 렌더링

```jsx
<Title render={() => <h1>render prop</h1>} />

// Title.jsx
const Title = props => props.render();
```

한 컴포넌트를 재사용하면서 매번 다른 값을 렌더링 prop에 전달할 수 있음

```jsx
// 상태 끌어올리기 해결
// Input 리렌더링으로 Kelvin, Fahrenheit 리렌더링
function Input(props) {
	const [value, setValue] = useState("");
	
	return (
		<>
			<input
				type="text"
				value={value}
				onChage={...}
			/>
			{props.render(value)}
		</>
	);
}

function App() {
	return (
		<div>
			<Input
				render={(value) => (
					<>
						<Kelvin value={value} />
						<Fahrenheit value={value} />
					</>
				)}
			/>
		</div>
	);
}

// children을 통해 함수 전달 가능
function App() {
	return (
		<div>
			<Input>
				{(value) => (
					<>
						<Kelvin value={value} />
						<Fahrenheit value={value} />
					</>
				)}
			</Input>
		</div>
	);
}

function Input(props) {
	const [value, setValue] = useState("");
	
	return (
		<>
			<input
				type="text"
				value={value}
				onChage={...}
			/>
			{props.children(value)}
		</>
	);
}
```

여러 컴포넌트 사이에서 로직과 데이터를 쉽게 공유할 수 있음

고차 컴포넌트 패턴에서 발생할 수 있는 이름 충돌 문제 해결

애플리케이션 로직과 렌덜이 컴포넌트 분리 가능

Hooks가 렌더링 props 패턴 대체 가능

## 리액트 Hooks 패턴

### 클래스 컴포넌트

클래스 문법을 이해해야 함

로직이 많을 수록 컴포넌트 크기가 증가

코드가 복잡해져 디버깅이 어려워짐

### Hooks

컴포넌트의 상태와 라이프사이클 메서드를 관리할 때, 그리고 여러 컴포넌트 간에 동일한 상태 관련 로직 재사용을 위해 사용

## Hook의 장단점

- 더 적은 코드 라인 수
- 복잡한 컴포넌트의 단순화
- 핫로딩이 어려움
- 상태 관련 로직 재사용
- UI와 분리된 로직 공유

## 정적 가져오기

정적 가져오기와 번들링을 통해 크기가 큰 번들은 성능에 이슈가 될 수 있음

## 동적 가져오기

특정 컴포넌트를 필요할 때 로드할 수 있음

리액트 18부터는 서버사이드에서도 가능

상호작용 시, 화면에 보일 때 로드할 수 있음

## PRPL 패턴

저사양 기기나 느린 네트워크에서도 원활하게 작동하기 위해 효율적으로 로드하는 패턴

- Push: 중요한 리소스를 효율적으로 푸시, 서버 왕복 횟수를 최소화
- Render: 초기 경로를 최대한 빠르게 렌더링
- Pre-cache: 자주 방문하는 경로의 에셋을 백그라운드에서 미리 캐싱
- Lazy-load: 자주 요청되지 않는 데이터는 지연 로딩

HTTP/2 의 서버 푸시 → 클라이언트 캐싱이 불가능 → 웹워커를 활용 직접 캐싱

## 로딩 우선순위

<link rel=”preload”> 는 브라우저의 최적화 기능

중요한 리소스를 일찍 요청할 수 있도록 함

Time To Interactive, First Input Delay와 같은 지표를 최적화할 때 상호작용에 필요한 번들을 로드하는 데 유용

과도하게 사용하면 FCP, LCP에 필요한 리소스의 로딩이 지연됨

### SPA의 Preload

특수한 웹팩 주석을 통해 preload할 모듈을 지정할 수 있음

초기 렌더링 후 약 1초 이내에 표시되어야 하는 리소스에만 선별하여 적용

### Preload + async

preload된 스크립트를 기다리는 동안에도 파싱이 멈추지 않게 하려면 script 태그에 async를 적용해야 함

이 경우 다른 리소스의 다운로드를 지연시킬 수 있음

### 크롬 95+ 버전에서의 preload

- HTTP 헤더에 preload를 넣으면 우선적으로 로드됨
- 미리 로드되는 폰트는 <head> 태그 끝 부분이나 <body> 태그 시작 부분에 넣어야 함
- 이미지 preload는 우선순위가 낮음

## 리스트 가상화

전체 목록을 렌더링하는 대신 현재 화면에 보이는 행만 동적으로 렌더링

react-window(react-virtualized) 라이브러리를 활용해 가능

동적인 콘텐츠 목록을 렌더링하는 경우 권장

### content-visibility

최신 브라우저 중 일부는 CSS의 content-visibility 속성을 지원

화면 밖 콘텐츠의 렌더링과 페인팅을 필요한 시점까지 지연

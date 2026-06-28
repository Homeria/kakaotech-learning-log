# Day 3-1 · Day 3-2 · Day 4-1 React 핵심 정리

> 원본 자료:  
> - `Day3-1 - React 소개 & JSX & 컴포넌트.md`  
> - `Day3-2 - useState 상태 관리.md`  
> - `Day4-1 - useEffect & 정리.md`  
>
> 정리 순서: **3-1 React 소개 & JSX & 컴포넌트 → 3-2 useState 상태 관리 → 4-1 useEffect & 정리**

---

# 1. Day 3-1 — React 소개, JSX, 컴포넌트

## 1.1 왜 React를 사용하는가?

바닐라 JavaScript만으로 To-do 리스트 같은 화면을 만들면, 상태가 바뀔 때마다 개발자가 직접 DOM을 찾아 수정해야 한다.

예를 들어 항목 추가만 있을 때는 단순하지만, 삭제, 완료 표시, 필터링 기능이 추가되면 다음과 같은 작업을 모두 직접 처리해야 한다.

```javascript
const list = document.querySelector("#todo-list");
const addBtn = document.querySelector("#add-btn");
const input = document.querySelector("#todo-input");

addBtn.addEventListener("click", () => {
  const li = document.createElement("li");
  li.innerText = input.value;
  list.appendChild(li);
  input.value = "";
});
```

이 방식은 기능이 많아질수록 다음 문제가 생긴다.

| 문제 | 설명 |
|---|---|
| DOM 추적이 복잡함 | 어떤 요소를 언제 수정해야 하는지 직접 관리해야 한다. |
| 코드가 길어짐 | 추가, 삭제, 수정, 필터링마다 DOM 조작 코드가 늘어난다. |
| 실수 가능성 증가 | 화면과 데이터가 서로 어긋나기 쉽다. |
| 유지보수 어려움 | 기능이 늘어날수록 수정 범위가 커진다. |

---

## 1.2 React의 접근 방식

React는 개발자가 DOM을 직접 조작하는 방식보다, **상태(state)를 바꾸면 화면은 React가 자동으로 업데이트하는 방식**을 사용한다.

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState("");

  const addTodo = () => {
    setTodos([...todos, input]);
    setInput("");
  };

  return (
    <div>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={addTodo}>추가</button>
      <ul>{todos.map((t, i) => <li key={i}>{t}</li>)}</ul>
    </div>
  );
}
```

React에서는 개발자가 직접 DOM을 찾아 수정하는 대신, **데이터 상태를 변경**한다.  
그러면 React가 변경된 상태를 기준으로 화면을 다시 그린다.

---

## 1.3 React의 핵심 관점

```text
화면 = 상태의 결과
```

React에서는 화면을 직접 수정하는 것이 아니라, 화면을 결정하는 데이터를 바꾼다.

```text
상태 변경
→ React가 변경 감지
→ 필요한 UI 다시 렌더링
→ 화면 업데이트
```

---

## 1.4 Virtual DOM

React는 실제 DOM을 바로 수정하지 않고, 메모리 상의 **Virtual DOM**을 활용한다.

상태가 변경되면 React는 새로운 Virtual DOM을 만들고, 이전 Virtual DOM과 비교한다.  
그 후 실제로 변경된 부분만 실제 DOM에 반영한다.

| 개념 | 설명 |
|---|---|
| Virtual DOM | 메모리 상에 존재하는 가상의 DOM |
| Diffing | 이전 Virtual DOM과 새로운 Virtual DOM을 비교하는 과정 |
| Re-rendering | 상태 변경 후 화면을 다시 그리는 과정 |
| 장점 | 불필요한 DOM 조작을 줄여 성능과 유지보수성을 높임 |

---

# 2. JSX

## 2.1 JSX란?

JSX는 JavaScript 안에 HTML과 유사한 문법을 작성할 수 있게 해주는 **문법 확장**이다.

브라우저는 JSX를 직접 이해하지 못한다.  
따라서 Babel 같은 도구가 JSX를 일반 JavaScript 코드로 변환한다.

```jsx
const name = "김민준";

const element = (
  <div className="chat-message">
    <p>안녕하세요, {name}!</p>
    <p>현재 시각: {new Date().toLocaleTimeString()}</p>
  </div>
);
```

---

## 2.2 JSX에서 JavaScript 표현식 사용하기

JSX 안에서는 `{}`를 사용해 JavaScript 표현식을 넣을 수 있다.

```jsx
const name = "Elice";

return <p>Hello, {name}</p>;
```

가능한 것:

```jsx
{name}
{1 + 2}
{isLoggedIn ? "환영합니다" : "로그인해주세요"}
{todos.map((todo) => <li>{todo}</li>)}
```

불가능하거나 부적절한 것:

```jsx
{if (isLoggedIn) { ... }} // JSX 안에서 if문 직접 사용 불가
{for (...) { ... }}       // JSX 안에서 for문 직접 사용 불가
```

조건문이 필요하면 보통 삼항 연산자나 조건부 렌더링을 사용한다.

---

## 2.3 JSX 핵심 규칙

| 규칙 | 설명 | 예시 |
|---|---|---|
| 최상위 요소는 하나 | 여러 요소를 반환하려면 하나로 감싸야 함 | `<div>...</div>` 또는 `<>...</>` |
| `class` 대신 `className` | JSX는 JavaScript 문법이므로 `class` 예약어와 충돌 방지 | `<div className="container">` |
| 닫는 태그 필수 | 일반 태그와 self-closing 태그 모두 닫아야 함 | `<img src="a.jpg" />` |
| `{}` 안에는 표현식만 | if문, for문 같은 문은 직접 넣을 수 없음 | `{isOpen ? <A /> : <B />}` |

---

## 2.4 Fragment

여러 요소를 반환해야 하지만 불필요한 `<div>`를 만들고 싶지 않을 때 Fragment를 사용한다.

```jsx
return (
  <>
    <h1>제목</h1>
    <p>내용</p>
  </>
);
```

Fragment는 실제 DOM에 별도의 태그를 만들지 않고 요소들을 묶어준다.

---

## 2.5 `className`을 사용하는 이유

HTML에서는 `class` 속성을 사용하지만 JSX에서는 `className`을 사용한다.

JavaScript에서 `class`는 예약어이기 때문에 JSX에서는 충돌을 피하기 위해 `className`을 사용한다.

```jsx
<div className="container">내용</div>
```

---

# 3. React 프로젝트 환경 설정

## 3.1 Vite + React 프로젝트 생성

React 프로젝트는 Vite를 사용해 빠르게 생성할 수 있다.

```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

---

## 3.2 생성된 기본 구조

```text
my-app/
├── src/
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
├── index.html
└── package.json
```

| 파일 | 역할 |
|---|---|
| `src/App.jsx` | 루트 컴포넌트. 주로 여기서 화면 구조를 작성 |
| `src/main.jsx` | React 앱을 실제 DOM에 마운트하는 진입점 |
| `src/index.css` | 전역 스타일 파일 |
| `index.html` | `id="root"`를 가진 기본 HTML |
| `package.json` | 프로젝트 의존성, 스크립트 관리 |

---

# 4. 컴포넌트

## 4.1 컴포넌트란?

컴포넌트는 **독립적이고 재사용 가능한 UI 조각**이다.  
React 앱은 여러 컴포넌트를 조합하여 만들어진다.

```text
페이지
└── Header
└── Main
    └── Card
    └── Button
└── Footer
```

---

## 4.2 함수형 컴포넌트 기본 구조

```jsx
function Greeting() {
  return (
    <div>
      <h1>안녕하세요! 👋</h1>
      <p>React 라이브 교안입니다.</p>
    </div>
  );
}

export default function App() {
  return (
    <div>
      <Greeting />
      <Greeting />
    </div>
  );
}
```

컴포넌트는 HTML 태그처럼 사용할 수 있고, 여러 번 재사용할 수 있다.

---

## 4.3 컴포넌트 이름은 대문자로 시작

React 컴포넌트의 이름은 반드시 대문자로 시작해야 한다.

| 구분 | React의 해석 |
|---|---|
| `<div>` | 기본 HTML 태그 |
| `<button>` | 기본 HTML 태그 |
| `<Greeting />` | 사용자가 만든 React 컴포넌트 |
| `<MyComponent />` | 사용자가 만든 React 컴포넌트 |

React는 JSX를 해석할 때 첫 글자가 소문자인지 대문자인지로 HTML 태그와 사용자 컴포넌트를 구분한다.

```jsx
<mycomponent />  // HTML 태그처럼 인식
<MyComponent />  // React 컴포넌트로 인식
```

---

# 5. Props

## 5.1 Props란?

Props는 부모 컴포넌트가 자식 컴포넌트에게 전달하는 데이터이다.

```jsx
function App() {
  return (
    <div>
      <UserCard name="김민준" role="학생" />
      <UserCard name="이서아" role="조교" />
    </div>
  );
}
```

자식 컴포넌트는 props를 받아 화면에 사용할 수 있다.

```jsx
function UserCard({ name, role }) {
  return (
    <div className="card">
      <h3>{name}</h3>
      <p>{role}</p>
    </div>
  );
}
```

---

## 5.2 Props의 특징

| 특징 | 설명 |
|---|---|
| 부모 → 자식 | 데이터는 부모에서 자식 방향으로 전달된다. |
| 읽기 전용 | 자식은 props를 직접 수정할 수 없다. |
| 재사용성 향상 | 같은 컴포넌트에 다른 props를 전달해 여러 화면을 만들 수 있다. |
| 구조 분해 할당 가능 | `{ name, role }`처럼 필요한 값만 바로 꺼내 쓸 수 있다. |

---

## 5.3 Props 실습 구조

부모님의 재산을 자녀가 상속받는 예제를 통해 props를 이해할 수 있다.

```jsx
function App() {
  return (
    <main className="app">
      <h1>Props 실습 — 가문의 재산</h1>
      <FamilyBanner familyName="React 가문" />
      <ParentComponent name="아버지" assets={500} />
      <ChildComponent
        name="자녀"
        assets={0}
        onReceive={() => alert("재산을 받기로 했습니다!")}
      />
    </main>
  );
}
```

---

## 5.4 컴포넌트별 역할

| 컴포넌트 | 역할 |
|---|---|
| `FamilyBanner` | 가문 이름을 화면에 표시 |
| `ParentComponent` | 부모 이름과 재산 표시 |
| `ChildComponent` | 자녀 이름과 재산 표시, 버튼 클릭 시 함수 실행 |
| `App` | 전체 상태와 컴포넌트 조합 담당 |

---

## 5.5 정답 예시

```jsx
export default function FamilyBanner({ familyName }) {
  return <p className="family-banner">🏠 {familyName}</p>;
}
```

```jsx
export default function ParentComponent({ name, assets }) {
  return (
    <div className="parent-card">
      <p>{name} — {assets}원</p>
    </div>
  );
}
```

```jsx
export default function ChildComponent({ name, assets, onReceive }) {
  return (
    <div className="child-card">
      <p>{name} — {assets}원</p>
      <button type="button" onClick={onReceive}>재산 받기</button>
    </div>
  );
}
```

---

# 6. Day 3-1 핵심 정리

| 개념 | 핵심 포인트 |
|---|---|
| React | 상태가 바뀌면 화면을 자동으로 재렌더링한다. |
| Virtual DOM | 변경 전후를 비교해 필요한 부분만 실제 DOM에 반영한다. |
| JSX | JavaScript 안에 HTML 유사 문법을 작성한다. |
| `{}` | JSX 안에서 JavaScript 표현식을 넣을 때 사용한다. |
| 컴포넌트 | 재사용 가능한 UI 부품이다. |
| Props | 부모가 자식에게 전달하는 읽기 전용 데이터이다. |

---

# 7. Day 3-2 — useState 상태 관리

## 7.1 상태(State)란?

State는 컴포넌트에서 시간이 지나면서 바뀔 수 있는 데이터이다.

예를 들어 채팅 앱에서는 다음 값들이 상태가 될 수 있다.

| 예시 | 상태인 이유 |
|---|---|
| 입력창에 타이핑 중인 텍스트 | 사용자가 입력할 때마다 바뀜 |
| 전송된 메시지 목록 | 새 메시지가 전송될 때 바뀜 |
| 로그인 여부 | 로그인/로그아웃에 따라 바뀜 |
| 카운터 숫자 | 버튼 클릭에 따라 바뀜 |
| 모달 열림 여부 | 열기/닫기에 따라 바뀜 |

---

## 7.2 일반 변수와 상태의 차이

일반 변수는 값이 바뀌어도 React가 화면을 다시 그리지 않는다.

```jsx
function Counter() {
  let count = 0;

  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={() => { count++; console.log(count); }}>
        +1
      </button>
    </div>
  );
}
```

위 코드에서 `count` 값은 콘솔에서 증가할 수 있지만 화면은 자동으로 업데이트되지 않는다.

---

## 7.3 useState를 사용하는 이유

`useState`를 사용하면 값이 바뀔 때 React가 컴포넌트를 다시 렌더링한다.

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

버튼을 클릭하면 `count`가 증가하고, 화면도 함께 업데이트된다.

---

# 8. useState 기본 사용법

## 8.1 기본 구조

```jsx
const [상태값, 상태변경함수] = useState(초기값);
```

| 구성 | 설명 |
|---|---|
| 상태값 | 현재 데이터 값 |
| 상태변경함수 | 상태를 변경하는 함수 |
| 초기값 | 컴포넌트가 처음 렌더링될 때 사용할 값 |
| 반환값 | `[데이터, 변경함수]` 형태의 배열 |
| 사용 방식 | 구조 분해 할당으로 이름을 붙여 사용 |

---

## 8.2 다양한 상태 예시

```jsx
import { useState } from "react";

function Example() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("");
  const [isOpen, setIsOpen] = useState(false);
  const [items, setItems] = useState([]);
}
```

| 상태 | 초기값 | 의미 |
|---|---|---|
| `count` | `0` | 숫자 상태 |
| `name` | `""` | 문자열 상태 |
| `isOpen` | `false` | 불리언 상태 |
| `items` | `[]` | 배열 상태 |

---

## 8.3 상태 변경 예시

```jsx
setCount(count + 1);
setName("김민준");
setIsOpen(!isOpen);
setItems([...items, newItem]);
```

상태는 반드시 setter 함수를 통해 변경해야 한다.

---

## 8.4 상태 직접 수정 금지

상태를 직접 수정하면 React가 변경을 감지하지 못할 수 있다.

```jsx
count = count + 1; // 잘못된 방식
setCount(count + 1); // 올바른 방식
```

배열이나 객체도 마찬가지로 직접 수정하면 안 된다.

```jsx
items.push(newItem); // 잘못된 방식
setItems([...items, newItem]); // 올바른 방식
```

---

## 8.5 setter 함수의 특징

React의 상태 변경 함수는 상태 업데이트를 예약하고, 컴포넌트 리렌더링을 유발한다.

중요한 점은 상태 변경이 즉시 반영되는 것처럼 보이지 않을 수 있다는 것이다.

```jsx
setCount(count + 1);
console.log(count); // 변경 전 값이 출력될 수 있음
```

React는 성능 최적화를 위해 여러 상태 변경 요청을 모았다가 한 번에 처리할 수 있다.  
따라서 setter 호출 직후에 상태값을 읽으면 이전 값이 나올 수 있다.

---

## 8.6 setter에 await를 사용하면 안 되는 이유

React의 setter 함수는 Promise를 반환하지 않는다.  
따라서 `await setCount(...)`처럼 작성해도 원하는 방식으로 기다릴 수 없다.

```jsx
await setCount(count + 1); // 의미 없음
```

상태 변경 후 특정 작업이 필요하다면 `useEffect`를 사용해 해당 상태의 변화를 감지하는 방식이 더 적절하다.

---

# 9. 배열 상태 관리와 불변성

## 9.1 불변성 원칙

React에서 배열이나 객체 상태를 관리할 때는 기존 값을 직접 수정하지 않고, 새 배열이나 새 객체를 만들어 전달해야 한다.

이를 **불변성(Immutability)** 원칙이라고 한다.

---

## 9.2 왜 직접 수정하면 안 되는가?

React는 이전 state와 새로운 state의 참조를 비교해 변경 여부를 판단한다.  
즉, 원본 배열을 직접 수정하고 같은 배열을 다시 전달하면 React가 변경을 감지하지 못할 수 있다.

```jsx
const [fruits, setFruits] = useState(["사과", "바나나"]);

fruits.push("포도");
setFruits(fruits); // 잘못된 방식
```

---

## 9.3 배열 추가

```jsx
setFruits([...fruits, "포도"]);
```

기존 배열을 펼치고 새 값을 추가하여 새로운 배열을 만든다.

```text
["사과", "바나나"] → ["사과", "바나나", "포도"]
```

---

## 9.4 배열 삭제

```jsx
setFruits(fruits.filter((fruit) => fruit !== "바나나"));
```

`filter`를 사용하면 조건에 맞는 요소만 남긴 새 배열을 만들 수 있다.

```text
["사과", "바나나"] → ["사과"]
```

---

## 9.5 배열 수정

```jsx
setFruits(
  fruits.map((fruit) => fruit === "바나나" ? "딸기" : fruit)
);
```

`map`을 사용하면 각 요소를 검사하며 필요한 값만 바꾼 새 배열을 만들 수 있다.

```text
["사과", "바나나"] → ["사과", "딸기"]
```

---

## 9.6 배열 상태 관리 요약

| 작업 | 사용 메서드 | 예시 |
|---|---|---|
| 추가 | Spread | `setItems([...items, newItem])` |
| 삭제 | `filter` | `setItems(items.filter((item) => item.id !== id))` |
| 수정 | `map` | `setItems(items.map((item) => item.id === id ? updated : item))` |

---

# 10. 제어 컴포넌트와 비제어 컴포넌트

## 10.1 제어 컴포넌트란?

제어 컴포넌트는 입력값을 React state로 관리하는 방식이다.  
사용자가 입력할 때마다 state가 업데이트되므로 React가 현재 입력값을 항상 알고 있다.

```jsx
import { useState } from "react";

function PasswordInput() {
  const [password, setPassword] = useState("");

  const isValid = password.length >= 8;

  return (
    <div>
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="비밀번호를 입력하세요"
      />
      <p style={{ color: isValid ? "green" : "red" }}>
        {isValid ? "✅ 사용 가능한 비밀번호입니다" : "❌ 8자 이상 입력해주세요"}
      </p>
    </div>
  );
}
```

---

## 10.2 제어 컴포넌트가 필요한 경우

| 상황 | 이유 |
|---|---|
| 실시간 유효성 검사 | 입력값이 바뀔 때마다 검사해야 함 |
| 글자 수 제한 | 입력 상태를 React가 직접 관리해야 함 |
| 검색어 자동완성 | 타이핑 중인 값을 계속 알아야 함 |
| 입력값에 따른 UI 변경 | state를 기준으로 화면을 즉시 바꿔야 함 |

---

## 10.3 비제어 컴포넌트란?

비제어 컴포넌트는 입력값을 React state가 아니라 브라우저 DOM이 관리하는 방식이다.  
React는 타이핑 중에는 관여하지 않고, 필요한 순간에만 값을 읽는다.

```jsx
import { useRef } from "react";

function LoginForm() {
  const idRef = useRef(null);
  const passwordRef = useRef(null);

  const handleLogin = () => {
    const id = idRef.current.value;
    const password = passwordRef.current.value;
    console.log("로그인 시도:", id, password);
  };

  return (
    <div>
      <input ref={idRef} placeholder="아이디" />
      <input ref={passwordRef} type="password" placeholder="비밀번호" />
      <button onClick={handleLogin}>로그인</button>
    </div>
  );
}
```

---

## 10.4 제어 컴포넌트와 비제어 컴포넌트 비교

| 특징 | 제어 컴포넌트 | 비제어 컴포넌트 |
|---|---|---|
| 데이터 관리 주체 | React, `useState` | 브라우저 DOM |
| 값을 아는 시점 | 타이핑하는 실시간 | 제출 버튼을 누를 때 한 번 |
| 사용하는 Hook | `useState` | `useRef` |
| 적합한 상황 | 실시간 검사, 글자 수 제한, 자동완성 | 단순 폼 제출, 파일 업로드 |
| 코드 특징 | `value`와 `onChange`를 사용 | `ref`를 사용 |

---

# 11. Props와 useState 실습 — 재산 상태 관리

## 11.1 실습 목표

Props만 사용하면 부모와 자식에게 값을 전달할 수는 있지만, 버튼 클릭 후 실제 값이 바뀌지는 않는다.

`useState`를 사용하면 부모 재산과 자녀 재산을 상태로 관리하고, 버튼 클릭 시 값을 변경할 수 있다.

---

## 11.2 구현해야 할 상태

| 상태 | 초기값 | 설명 |
|---|---|---|
| `parentAssets` | `500` | 부모님의 재산 |
| `childAssets` | `0` | 자녀의 재산 |

---

## 11.3 구현해야 할 함수

`handleInherit` 함수는 다음 동작을 수행한다.

```text
1. 자녀 재산에 부모 재산을 더한다.
2. 부모 재산을 0으로 만든다.
3. 화면이 자동으로 업데이트된다.
```

---

## 11.4 정답 예시

```jsx
import { useState } from "react";
import FamilyBanner from "./components/FamilyBanner";
import ParentComponent from "./components/ParentComponent";
import ChildComponent from "./components/ChildComponent";
import "./App.css";

export default function Practice02() {
  const [parentAssets, setParentAssets] = useState(500);
  const [childAssets, setChildAssets] = useState(0);

  function handleInherit() {
    alert("재산을 받았습니다!");
    setChildAssets(childAssets + parentAssets);
    setParentAssets(0);
  }

  return (
    <main className="app">
      <h1>useState 실습 — 가문의 재산</h1>
      <FamilyBanner familyName="React 가문" />
      <ParentComponent name="부모님" assets={parentAssets} />
      <ChildComponent
        name="자녀"
        assets={childAssets}
        onReceive={handleInherit}
      />
    </main>
  );
}
```

---

# 12. Day 3-2 핵심 정리

| 개념 | 핵심 포인트 |
|---|---|
| State | 컴포넌트에서 시간이 지나며 바뀔 수 있는 데이터 |
| useState | `const [값, setter] = useState(초기값)` |
| setter 함수 | 상태를 변경하고 리렌더링을 유발 |
| 직접 수정 금지 | 상태는 반드시 setter로 변경 |
| 불변성 | 배열/객체는 원본을 수정하지 않고 새 값 생성 |
| Controlled Input | `value={state}`와 `onChange`로 입력값 제어 |
| Uncontrolled Input | `useRef`로 필요할 때 DOM 값을 읽음 |

---

# 13. Day 4-1 — useEffect

## 13.1 useEffect란?

`useEffect`는 컴포넌트가 화면에 렌더링된 이후에 실행되는 React Hook이다.

주로 다음 두 가지 목적을 위해 사용한다.

| 목적 | 설명 |
|---|---|
| 외부 시스템과의 연결 | API 호출, 타이머 설정, 구독 등 React 바깥의 일을 처리 |
| 실행 타이밍 통제 | 특정 코드가 매번 반복 실행되지 않도록 제어 |

---

## 13.2 왜 useEffect가 필요한가?

`useState`는 사용자가 버튼을 누르거나 입력할 때 상태를 바꾸고 화면을 업데이트할 때 사용한다.

하지만 실제 웹 서비스에서는 사용자의 행동이 없어도 코드가 실행되어야 할 때가 있다.

예시:

| 상황 | 필요한 동작 |
|---|---|
| 채팅방 입장 | 서버에서 기존 채팅 내역 불러오기 |
| 페이지 첫 진입 | 초기 데이터 요청 |
| 타이머 화면 | 화면이 켜지자마자 타이머 시작 |
| 브라우저 탭 제목 변경 | 특정 상태에 따라 `document.title` 수정 |
| 외부 구독 | 채팅방, 소켓, 이벤트 리스너 연결 |

이런 코드는 렌더링 자체와는 별개로 외부 시스템과 연결되므로 **부수효과(Side Effect)**라고 부른다.

---

# 14. 컴포넌트 생명주기와 useEffect

## 14.1 컴포넌트 생명주기

React 컴포넌트는 크게 세 단계를 거친다.

```text
Mount → Update → Unmount
```

| 단계 | 의미 |
|---|---|
| Mount | 컴포넌트가 화면에 처음 등장 |
| Update | state나 props가 바뀌어 다시 렌더링 |
| Unmount | 컴포넌트가 화면에서 사라짐 |

---

## 14.2 useEffect의 역할

`useEffect`를 사용하면 이 생명주기의 특정 시점에 코드를 실행할 수 있다.

예를 들어 다음처럼 사용할 수 있다.

| 실행 시점 | 예시 |
|---|---|
| 처음 화면에 나타났을 때 | API 호출, 초기 데이터 로드 |
| 특정 state가 바뀌었을 때 | 검색어 변경 시 결과 요청 |
| 화면에서 사라질 때 | 타이머 제거, 구독 해제 |

---

# 15. useEffect 기본 구조

## 15.1 기본 문법

```jsx
useEffect(실행할_함수, [의존성_배열]);
```

| 구성 | 설명 |
|---|---|
| 실행할 함수 | 화면이 그려진 후 실행할 실제 작업 코드 |
| 의존성 배열 | effect를 언제 다시 실행할지 결정하는 값 목록 |
| 실행 순서 | 렌더링 → 화면 업데이트 → effect 실행 |

---

## 15.2 useEffect 사용 예시

```jsx
import { useState, useEffect } from "react";

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("렌더링되었습니다");
  });

  useEffect(() => {
    console.log("컴포넌트가 화면에 나타났습니다!");
  }, []);

  useEffect(() => {
    console.log("카운트가 변경되었습니다:", count);
  }, [count]);

  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

---

## 15.3 의존성 배열 패턴

| 패턴 | 실행 시점 | 주요 사용처 |
|---|---|---|
| `useEffect(() => {})` | 매 렌더링마다 실행 | 거의 사용하지 않음 |
| `useEffect(() => {}, [])` | 마운트 시 1회 실행 | 초기 데이터 로드, 구독 |
| `useEffect(() => {}, [dep])` | `dep` 변경 시 실행 | 특정 상태 변화에 따른 부수효과 |

---

## 15.4 빈 배열과 의존성 배열 없음의 차이

| 코드 | 의미 |
|---|---|
| `useEffect(() => {}, [])` | 컴포넌트가 처음 마운트될 때 한 번만 실행 |
| `useEffect(() => {})` | 컴포넌트가 렌더링될 때마다 실행 |
| `useEffect(() => {}, [count])` | `count`가 바뀔 때마다 실행 |

빈 배열 `[]`은 매번 실행이 아니라 **처음 한 번만 실행**이다.  
의존성 배열 자체를 생략해야 매 렌더링마다 실행된다.

---

## 15.5 Strict Mode에서 두 번 실행되는 이유

개발 모드에서는 React Strict Mode 때문에 마운트 시 effect가 두 번 실행되는 것처럼 보일 수 있다.  
이는 버그가 아니라, 부수효과 코드가 안전하게 작성되었는지 확인하기 위한 개발 모드 동작이다.

---

# 16. Cleanup 함수

## 16.1 Cleanup 함수란?

Cleanup 함수는 effect에서 시작한 작업을 정리하는 함수이다.

예를 들어 `setInterval`로 타이머를 만들었다면, 컴포넌트가 사라질 때 `clearInterval`로 제거해야 한다.

```jsx
useEffect(() => {
  const intervalId = setInterval(() => {
    console.log("주기적으로 실행");
  }, 1000);

  return () => {
    clearInterval(intervalId);
  };
}, []);
```

---

## 16.2 Cleanup이 필요한 이유

Cleanup을 하지 않으면 컴포넌트가 화면에서 사라진 후에도 타이머나 구독이 계속 실행될 수 있다.

| 문제 | 설명 |
|---|---|
| 메모리 누수 | 필요 없는 작업이 계속 살아 있음 |
| 중복 실행 | 컴포넌트가 다시 생길 때 기존 작업과 새 작업이 동시에 실행 |
| 버그 발생 | 사라진 컴포넌트와 관련된 작업이 계속 동작 |
| 성능 저하 | 불필요한 타이머, 이벤트 리스너, 구독 유지 |

---

## 16.3 Cleanup 함수 실행 타이밍

Cleanup 함수는 두 가지 상황에서 실행된다.

| 타이밍 | 설명 |
|---|---|
| Unmount | 컴포넌트가 화면에서 완전히 사라질 때 |
| 다음 Effect 실행 직전 | 의존성 배열 값이 바뀌어 새로운 effect가 실행되기 바로 전 |

---

## 16.4 채팅방 예시

```jsx
useEffect(() => {
  console.log(`${roomId}번 방에 입장했습니다.`);

  return () => {
    console.log(`${roomId}번 방에서 퇴장했습니다.`);
  };
}, [roomId]);
```

`roomId`가 1에서 2로 바뀌면 실행 순서는 다음과 같다.

```text
1. 1번 방에 입장했습니다.
2. 1번 방에서 퇴장했습니다.
3. 2번 방에 입장했습니다.
```

즉, 새로운 effect가 실행되기 전에 이전 effect의 cleanup이 먼저 실행된다.

---

# 17. useEffect 실습 — 타이머와 탭 타이틀

## 17.1 실습 목표

`useEffect`를 활용해 다음 기능을 구현한다.

| 기능 | 설명 |
|---|---|
| 타이머 실행 | 1초마다 남은 시간을 줄인다. |
| 루프 카운트 증가 | 시간이 0이 되면 루프 횟수를 증가시키고 시간을 초기화한다. |
| 일시정지 | `isPaused`가 true이면 타이머를 멈춘다. |
| 미션 완료 | `isSolved`가 true이면 타이머를 멈춘다. |
| Cleanup | interval을 정리해 중복 실행을 막는다. |
| 탭 타이틀 변경 | loopCount에 따라 `document.title`을 변경한다. |

---

## 17.2 타이머 effect 구조

```jsx
useEffect(() => {
  if (isSolved || isPaused) return;

  const interval = setInterval(() => {
    setTimeLeft((prev) => {
      if (prev <= 1) {
        setLoopCount((c) => c + 1);
        return 15;
      }
      return prev - 1;
    });
  }, 1000);

  return () => {
    clearInterval(interval);
  };
}, [isSolved, isPaused]);
```

---

## 17.3 함수형 업데이트

타이머처럼 이전 상태를 기준으로 다음 상태를 계산할 때는 함수형 업데이트를 사용하는 것이 안전하다.

```jsx
setTimeLeft((prev) => prev - 1);
setLoopCount((c) => c + 1);
```

| 방식 | 설명 |
|---|---|
| `setCount(count + 1)` | 현재 렌더링 시점의 count 값을 사용 |
| `setCount((prev) => prev + 1)` | React가 보장하는 최신 이전 값을 사용 |

반복 실행, 타이머, 연속 클릭처럼 이전 값을 기준으로 계산할 때는 함수형 업데이트가 더 안전하다.

---

## 17.4 탭 타이틀 effect 구조

```jsx
useEffect(() => {
  document.title =
    loopCount > 0 ? `[루프 ${loopCount}회] 타임루프 작전` : "타임루프 작전";

  return () => {
    document.title = "실습";
  };
}, [loopCount]);
```

`loopCount`가 바뀔 때마다 탭 제목을 업데이트하고, cleanup에서 기본 제목으로 되돌릴 수 있다.

---

# 18. useEffect 의존성 배열 주의점

## 18.1 객체나 함수를 의존성 배열에 넣을 때

의존성 배열은 값을 비교해 effect 재실행 여부를 결정한다.

숫자나 문자열은 값 자체를 비교하지만, 객체나 함수는 메모리 주소를 비교한다.

```jsx
useEffect(() => {
  // effect
}, [{ name: "Elice" }]);
```

위와 같이 객체를 직접 넣으면 렌더링될 때마다 새로운 객체가 만들어진다.  
React는 매번 다른 값으로 인식하여 effect를 계속 실행할 수 있다.

---

## 18.2 무한 루프가 발생할 수 있는 이유

컴포넌트가 리렌더링될 때마다 객체나 함수가 새로 생성되면, 의존성 배열의 값이 바뀐 것으로 판단된다.

```text
렌더링
→ 새 객체 생성
→ 의존성 변경으로 판단
→ useEffect 실행
→ 상태 변경
→ 다시 렌더링
→ 새 객체 생성
→ 반복
```

이런 구조는 무한 루프를 만들 수 있다.

---

## 18.3 의존성 배열 관리 원칙

| 원칙 | 설명 |
|---|---|
| effect 내부에서 사용하는 state와 props는 의존성 배열에 포함 | 누락하면 오래된 값을 사용할 수 있음 |
| 객체/함수 직접 생성 후 의존성에 넣는 것 주의 | 매 렌더링마다 새 참조가 만들어짐 |
| 빈 배열 `[]`은 최초 1회 실행 | 매번 실행이 아님 |
| cleanup 필요한 작업은 반드시 정리 | 타이머, 이벤트 리스너, 구독 등 |

---

# 19. React 활용 3단계 요약

```text
1. UI는 컴포넌트를 조합해서 만든다.
2. 데이터(state)가 변하면 화면은 React가 자동 업데이트한다.
3. 부수효과, 외부 요청, 타이머는 useEffect로 관리한다.
```

---

# 20. 자주 나오는 질문

## 20.1 useState와 일반 변수는 어떻게 다른가?

일반 변수 변경은 화면 재렌더링을 트리거하지 않는다.  
`useState`의 setter 함수로 변경해야 React가 화면을 다시 그린다.

| 구분 | 일반 변수 | useState |
|---|---|---|
| 값 변경 가능 | 가능 | 가능 |
| 화면 업데이트 | 자동으로 안 됨 | setter 호출 시 리렌더링 |
| React가 변경 감지 | 어려움 | 가능 |
| 사용 목적 | 렌더링과 무관한 임시 값 | 화면에 반영되어야 하는 값 |

---

## 20.2 props와 state는 뭐가 다른가?

| 구분 | props | state |
|---|---|---|
| 관리 주체 | 부모 컴포넌트 | 컴포넌트 자신 |
| 전달 방향 | 부모 → 자식 | 컴포넌트 내부 |
| 수정 가능 여부 | 자식이 직접 수정 불가 | setter로 수정 가능 |
| 용도 | 컴포넌트에 데이터 전달 | 변화하는 데이터 관리 |

---

## 20.3 useEffect 의존성 배열을 비우면 매번 실행되는가?

아니다.

```jsx
useEffect(() => {}, []);
```

빈 배열은 마운트 시 한 번만 실행된다.

매번 실행되는 것은 다음처럼 의존성 배열을 생략한 경우이다.

```jsx
useEffect(() => {});
```

---

# 21. 전체 핵심 요약

## 21.1 React 전체 흐름

React는 상태를 중심으로 화면을 구성한다.

```text
컴포넌트 작성
→ Props로 데이터 전달
→ useState로 변하는 데이터 관리
→ 상태 변경 시 자동 리렌더링
→ useEffect로 외부 작업과 실행 타이밍 관리
```

---

## 21.2 React 기본 개념 요약

| 개념 | 핵심 |
|---|---|
| React | 상태가 바뀌면 화면을 자동으로 업데이트하는 UI 라이브러리 |
| JSX | JavaScript 안에서 HTML처럼 UI를 작성하는 문법 |
| 컴포넌트 | 재사용 가능한 UI 단위 |
| Props | 부모가 자식에게 전달하는 읽기 전용 데이터 |
| State | 컴포넌트 내부에서 변할 수 있는 데이터 |
| useState | 상태를 만들고 변경하는 Hook |
| useEffect | 렌더링 이후 부수효과를 처리하는 Hook |
| Cleanup | effect에서 시작한 작업을 정리하는 함수 |

---

## 21.3 Hook별 역할 정리

| Hook | 역할 | 대표 사용 상황 |
|---|---|---|
| `useState` | 컴포넌트의 상태 관리 | 카운터, 입력값, 목록, 열림/닫힘 |
| `useEffect` | 렌더링 이후 부수효과 처리 | API 호출, 타이머, 구독, 제목 변경 |
| `useRef` | DOM 접근 또는 렌더링과 무관한 값 보관 | 비제어 입력, 특정 DOM 요소 접근 |

---

## 21.4 최종 정리

React는 DOM을 직접 조작하는 방식의 복잡함을 줄이고, 상태를 기준으로 화면을 자동 업데이트하는 방식으로 UI를 만든다.  
JSX는 JavaScript 안에서 HTML과 비슷한 문법으로 화면을 작성하게 해주며, 컴포넌트는 UI를 작은 부품으로 나누어 재사용할 수 있게 한다.  
Props는 부모 컴포넌트가 자식 컴포넌트에게 데이터를 전달하는 방법이고, 자식은 props를 직접 수정할 수 없다.

상태가 필요한 경우에는 `useState`를 사용한다.  
일반 변수와 달리 `useState`의 setter 함수로 값을 변경하면 React가 컴포넌트를 다시 렌더링한다.  
배열이나 객체 상태를 다룰 때는 원본을 직접 수정하지 않고 새 배열이나 새 객체를 만들어 전달해야 한다.

입력값을 React가 실시간으로 관리해야 하면 제어 컴포넌트를 사용하고, 제출 시점에만 값을 읽으면 충분할 때는 비제어 컴포넌트를 사용할 수 있다.

`useEffect`는 렌더링 이후 실행되어야 하는 부수효과를 관리한다.  
API 호출, 타이머, 브라우저 탭 제목 변경, 외부 구독처럼 React 외부 시스템과 연결되는 작업에 사용한다.  
의존성 배열을 통해 effect 실행 시점을 제어할 수 있으며, 타이머나 구독처럼 정리가 필요한 작업은 cleanup 함수를 반드시 작성해야 한다.

결국 React 학습의 핵심은 다음 세 가지로 정리할 수 있다.

```text
1. UI는 컴포넌트로 쪼개서 만든다.
2. 변하는 데이터는 state로 관리한다.
3. 외부 시스템과 연결되는 부수효과는 useEffect로 관리한다.
```

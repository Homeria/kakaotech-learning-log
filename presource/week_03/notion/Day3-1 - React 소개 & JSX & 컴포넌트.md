## 1. 왜 React인가? — DOM 조작의 한계

바닐라 JS로 To-do 리스트를 만든다고 상상해 보세요.

```jsx
// ❌ 바닐라 JS 방식: 상태 변화마다 DOM을 직접 추적
const list = document.querySelector("#todo-list");
const addBtn = document.querySelector("#add-btn");
const input = document.querySelector("#todo-input");

addBtn.addEventListener("click", () => {
  const li = document.createElement("li");
  li.innerText = input.value;
  list.appendChild(li);
  input.value = "";
  // → 항목 삭제, 완료 표시, 필터링 기능이 추가되면, 각 기능에 따라 어떤 DOM을 언제 어떻게 찾아가서 수정/삭제/추가해야 할지 개발자가 일일이 추적하고 코드로 구현해야 하므로 복잡해짐
});
```

항목이 10개, 100개가 되거나 삭제·완료 표시·필터 기능이 추가되면, "어떤 DOM을 언제 업데이트해야 하는가"를 개발자가 일일이 추적해야 합니다.

**React의 접근법: 상태(state)만 바꾸면 화면은 React가 처리한다**

```jsx
// ✅ React 방식 (미리보기 — 아래에서 자세히 배웁니다)
function TodoApp() {
  const [todos, setTodos] = useState([]); // 상태 선언
  const [input, setInput] = useState("");

  const addTodo = () => {
    setTodos([...todos, input]); // 상태만 바꾸면 → 화면이 자동으로 재렌더링
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

<aside>
🎓

**리액트를 쓰면 좋은 점 또 한가지: 가상 DOM (Virtual DOM) 기반의 최적화**

React는 실제 DOM 대신 메모리 상의 **가상 DOM(Virtual DOM)**을 유지합니다. 상태가 변경되면 새로운 Virtual DOM을 만들고, 이전 Virtual DOM과 **비교(diffing)**하여 `실제로 변경된 부분만` 실제 DOM에 반영합니다. 덕분에 불필요한 연산이 줄어들고 성능이 올라갑니다.

</aside>

#### JSX — JS 안에서 HTML 쓰기

<aside>
📌

JSX는 JavaScript 안에 HTML과 유사한 문법을 작성할 수 있게 해주는 **문법 확장**입니다. 브라우저가 직접 이해하지는 못하고, Babel이 순수 JS로 변환합니다.

* Babel: 최신 자바스크립트나 JSX를 모든 브라우저가 알아들을 수 있는 '옛날 자바스크립트'로 바꿔주는 컴파일러(번역기)

</aside>

### JSX 기본 규칙

```jsx
// ✅ JSX: HTML처럼 보이지만 JS 표현식이 가능
const name = "김민준";
const element = (
  <div className="chat-message">
    <p>안녕하세요, {name}!</p>        {/* {} 안에 JS 표현식 */}
    <p>현재 시각: {new Date().toLocaleTimeString()}</p>
  </div>
);
```

**JSX 핵심 규칙 4가지**

```jsx
// 1. 최상위 요소는 반드시 하나 (또는 Fragment <> 사용)
return (
  <>         {/* Fragment: 불필요한 div 없이 여러 요소 묶기 */}
    <h1>제목</h1>
    <p>내용</p>
  </>
);

// 2. class 대신 className
<div className="container">...</div>

// 3. 닫는 태그 필수
<img src="photo.jpg" />   // self-closing도 반드시 /> 로 닫기

// 4. {} 안에는 JS 표현식만 (문, if/for 불가 → 삼항 연산자/map 사용)
const isLoggedIn = true;
return (
  <div>
    {isLoggedIn ? <p>환영합니다!</p> : <p>로그인해주세요.</p>}
  </div>
);
```

<aside>
💡

**💡 JS/React 비교**

HTML에서는 'class' 속성을 사용하지만, JSX는 JavaScript 문법 확장이라 JS 예약어인 'class'와 겹치지 않도록 'className'이라는 다른 이름을 사용합니다.

</aside>

---

## 2. 프로젝트 환경 설정

https://ko.react.dev/learn/build-a-react-app-from-scratch

**📂 Vite + React 프로젝트 생성 방법**

```bash
# 1. Vite로 React 프로젝트 생성
npm create vite@latest my-app -- --template react

# 2. 프로젝트 폴더로 이동
cd my-app

# 3. 의존성 설치
npm install

# 4. 개발 서버 실행
npm run dev
# → 브라우저에서 localhost(ex. http://localhost:5173) 접속
```

**생성된 파일 구조**

```
my-app/
├── src/
│   ├── App.jsx      ← 루트 컴포넌트 (여기서 작업)
│   ├── main.jsx     ← React 앱을 DOM에 마운트하는 진입점
│   └── index.css
...
├── index.html       ← id="root" div가 있는 HTML
└── package.json
```

## 3. 컴포넌트 — UI를 부품으로 쪼개기

<aside>
📌

컴포넌트는 **독립적이고 재사용 가능한 UI 조각**입니다. React 앱은 컴포넌트들의 조합으로 이루어집니다.

</aside>

### 함수형 컴포넌트 기본 구조

```jsx
// src/App.jsx

// 1. 함수 이름은 반드시 대문자로 시작
function Greeting() {
  // 2. JSX를 반환
  return (
    <div>
      <h1>안녕하세요! 👋</h1>
      <p>React 라이브 교안입니다.</p>
    </div>
  );
}

// 3. 다른 컴포넌트에서 HTML 태그처럼 사용
export default function App() {
  return (
    <div>
      <Greeting />     {/* 컴포넌트 사용 */}
      <Greeting />     {/* 재사용 가능! */}
    </div>
  );
}
```

- 함수 이름을 대문자로 써야 하는 이유
    
    **HTML 태그와 헷갈리지 않게 하기 위해서**입니다. React는 JSX 코드를 해석할 때, 첫 글자의 대소문자를 보고 서로 다르게 변환합니다.
    
    - **소문자(`<div>`, `<button>`)**: 원래 있는 HTML 태그로 인식
    - **대문자(`<MyComponent />`)**: 내가 직접 만든 React 컴포넌트로 인식
    
    만약 소문자로 쓰면 React가 일반 HTML 태그인 줄 알고 무시하거나 에러를 냅니다.
    
    우리가 작성한 JSX는 빌드 시 다음과 같이 JavaScript 코드로 바뀝니다.
    
    ```jsx
    // 소문자: 문자열 "mycomponent"로 변환 (HTML 태그로 취급)
    <mycomponent />  ->  React.createElement("mycomponent")
    
    // 대문자: 변수(함수 레퍼런스) MyComponent로 변환 (컴포넌트로 취급)
    <MyComponent />  ->  React.createElement(MyComponent)
    ```
    

### Props — 부모 → 자식으로 데이터 전달

```jsx
// 부모가 데이터를 "속성"으로 전달
function App() {
  return (
    <div>
      <UserCard name="김민준" role="학생" />
      <UserCard name="이서아" role="조교" />
    </div>
  );
}

// 자식은 props 객체로 데이터를 받음
function UserCard({ name, role }) {
  return (
    <div className="card">
      <h3>{name}</h3>
      <p>{role}</p>
    </div>
  );
}
```

```
출력:
┌─────────────┐  ┌─────────────┐
│ 김민준       │   │ 이서아       │
│ 학생         │  │ 조교         │
└─────────────┘  └─────────────┘
```

## ✏️ [실습] 컴포넌트 만들고 Prop 전달해보기

부모님의 재산을 자녀가 상속 받는 예제입니다. prop을 활용한 단방향 데이터 흐름을 위한 예시고, 돈 벌면 종종 부모님께 용돈도 드려보시기를 권장합니다.

```jsx
import FamilyBanner from "./components/FamilyBanner";
import ParentComponent from "./components/ParentComponent";
import ChildComponent from "./components/ChildComponent";
import "./App.css";

// ================================================================
// 실습 1 — components/ 폴더 안의 컴포넌트를 완성하세요.
//
//   ParentComponent.jsx
//     - name, assets prop을 받아 화면에 표시
//     - 예시: "아버지 — 500원"
//
//   ChildComponent.jsx
//     - name, assets, onReceive prop을 받아 화면에 표시
//     - 버튼 클릭 시 onReceive() 호출
//     - 예시: "자녀 — 0원" + [재산 받기] 버튼
// ================================================================

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

export default App;

```

```jsx
export default function FamilyBanner({ familyName }) {
  return <p className="family-banner">🏠 {familyName}</p>;
}
```

```jsx
// [실습 1] name, assets prop을 받아 표시하세요.
// 예시: "아버지 — 500원"

export default function ParentComponent() {
  return (
    <div className="parent-card">
      {/* TODO */}
    </div>
  );
}
```

```jsx
// ================================================================
// 실습 1 — name, assets, onReceive prop을 받아 화면에 표시하세요.
//   - 예시: "자녀 — 0원" + [재산 받기] 버튼
//   - 버튼 클릭 시 onReceive() 호출
//
// ================================================================

export default function ChildComponent() {
  return (
    <div className="child-card">
      {/* TODO */}
    </div>
  );
}

```

- **✅ 정답**
    
    !image.png
    
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
    

## 🔖 Part 1 핵심 정리

| 개념 | 핵심 포인트 |
| --- | --- |
| React | 상태(state)가 바뀌면 화면을 자동으로 재렌더링 |
| JSX | JS 안에 HTML 유사 문법. `{}` 안에 JS 표현식 |
| 컴포넌트 | 재사용 가능한 UI 부품 |
| Props | 부모 → 자식으로 데이터 전달. 구조 분해 할당으로 받기 |

> ➡️ **다음 파트**: `useState`로 상태를 관리하고, 실제 상호작용하는 컴포넌트 만들기
>
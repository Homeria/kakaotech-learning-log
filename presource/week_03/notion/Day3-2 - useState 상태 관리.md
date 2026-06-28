## 1. 상태(State)란 무엇인가?

> "컴포넌트에서 시간이 지남에 따라 변할 수 있는 것은 무엇일까요?"
> 
- 예를 들어 채팅 앱에는:
    - 입력창에 타이핑 중인 텍스트
    - 전송된 메시지 목록
    - 로그인 여부

이런 **"바뀔 수 있는 데이터"**를 React에서는 **상태(state)**라고 부릅니다.

**일반 변수 vs 상태**

```jsx
// ❌ 일반 변수: 값이 바뀌어도 화면이 다시 그려지지 않음
function Counter() {
  let count = 0;
  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={() => { count++; console.log(count); }}>
        +1 (화면 변화 없음)
      </button>
    </div>
  );
}
```

```jsx
// ✅ useState: 값이 바뀌면 React가 자동으로 재렌더링
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0); // 초기값 0

  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
// 버튼을 클릭할 때마다 count가 증가하고 화면도 업데이트됨
```

---

## 2. useState 사용법

<aside>
📌

`const [상태값, 상태변경함수] = useState(초기값);`

- 위 형태가 useState의 기본 구조
- 상태값은 현재 데이터 값, 상태변경함수는 그 데이터를 바꿀 수 있는 변경 함수
- **useState(초기값)** 호출 ➔ **[데이터, 변경함수]** 반환 ➔ **구조분해 할당**으로 각각 이름 붙여서 사용
</aside>

```jsx
import { useState } from 'react';

function Example() {
  const [count, setCount] = useState(0);    // 숫자 상태
  const [name, setName] = useState("");     // 문자열 상태
  const [isOpen, setIsOpen] = useState(false); // 불리언 상태
  const [items, setItems] = useState([]);   // 배열 상태

  // 상태 변경 (항상 setter 함수 사용!)
  // setCount(count + 1);   → count를 1 증가
  // setName("김민준");      → name을 "김민준"으로 변경
  // setIsOpen(!isOpen);    → 토글
  // setItems([...items, newItem]); → 배열에 항목 추가
}
```

<aside>
⛔

**직접 수정 x**

`count = count + 1;` 처럼 상태를 직접 수정하면 화면이 업데이트되지 않습니다. 반드시 `setCount(count + 1)` 처럼 **setter 함수**를 통해서만 변경해야 합니다.

* React의 setter 함수(`useState`의 두 번째 반환값)는 **상태(state)를 업데이트하고 컴포넌트 리렌더링을 유발하는 비동기 함수**

- 참고
    - **비동기 함수라고 표현했지만 `await`를 사용하면 안 됩니다**
        - `await`는 `Promise`를 반환하는 함수에만 쓸 수 있습니다. 하지만 React의 상태 변경 함수는 아무것도 반환하지 않는(`undefined`) 일반 함수이기 때문에 `await`를 붙여도 무시됩니다.
    - **호출 직후 상태 값을 읽으면 이전 값이 나온다?**
        - React는 성능 최적화를 위해 상태 변경을 즉시 처리하지 않고, 함수가 끝날 때까지 요청을 모았다가 한 번에 실행합니다. 즉, 호출 직후 다음 줄은 아직 값이 바뀌기 전인 '예약 상태'이므로 이전 값이 출력됩니다.
</aside>

<aside>
🎓

**비유로 이해하는 useState: "유튜버와 구독자"**

- 🎥 **유튜버 (State):** 최신 영상(데이터)을 가지고 있는 주체
- 🔘 **업로드 버튼 (set 함수):** 상태를 변경하고 알림을 보내는 유일한 스위치
- 📱 **구독자 (Component):** 영상을 기다리는 화면(UI)
- 🔄 **알림 & 시청 (Re-rendering):** 유튜버가 업로드 버튼(set 함수)을 누르는 순간, 구독자에게 알림이 가고, 화면이 최신 상태로 **자동 업데이트** 됩니다.
</aside>

## 3. 배열 상태 관리 — 불변성(Immutability) 원칙

리스트를 상태로 관리할 때, React에서 중요한 규칙이 있습니다.

**기존 배열을 직접 수정하지 말고, 새 배열을 만들어서 전달하세요.**

```jsx
// 🍎 초기 과일 바구니 상태 (기존 데이터가 들어있다고 가정)
const [fruits, setFruits] = useState(["사과", "바나나"]);

// ❌ 잘못된 방법 (원본 훼손)
// push를 쓰면 React가 바구니 자체는 똑같다고 생각해서 화면을 안 바꿔줍니다.
fruits.push("포도"); 
setFruits(fruits); 

// ========================================================

// ✅ 1. 과일 추가하기 (스프레드 연산자 활용)
// 스프레드로 기존 과일들을 새 바구니에 풀고, "포도"를 추가로 담습니다.
setFruits([...fruits, "포도"]); 
// 📝 출력 예시: ["사과", "바나나", "포도"]

// ✅ 2. 과일 삭제하기 (filter 활용)
// "바나나"가 아닌 녀석들만 쏙쏙 골라내어 새 바구니를 만듭니다.
setFruits(fruits.filter((fruit) => fruit !== "바나나"));
// 📝 출력 예시: ["사과"]

// ✅ 3. 과일 수정하기 (map 활용)
// 바구니를 하나씩 검사하다가 "바나나"를 발견하면 "딸기"로 바꿔서 새 바구니를 만듭니다.
setFruits(fruits.map((fruit) => 
  fruit === "바나나" ? "딸기" : fruit
));
// 📝 출력 예시: ["사과", "딸기"]
```

<aside>
🎓

**불변성(Immutability)**

함수형 프로그래밍에서 불변성이란 원본 데이터를 수정하지 않고 새로운 복사본을 만드는 원칙입니다. React는 이전 state 참조와 새 state 참조를 비교(얕은 비교)하여 변경 여부를 판단하기 때문에, 원본을 직접 수정하면 React가 변경을 감지하지 못합니다.

</aside>

## (심화) 제어 컴포넌트와 비제어 컴포넌트

<aside>
👀

**학습 이유**

지금까지는 버튼 클릭, 항목 추가처럼 **우리가 직접 만든 데이터**를 useState로 관리했습니다.

그런데 이런 화면을 생각해보세요.

> 🔐 비밀번호 입력창에서 8자 이상이 되는 순간, 아래에 "✅ 사용 가능" 메시지가 실시간으로 나타납니다.
> 

이 기능을 만들려면 **사용자가 키를 누를 때마다** React가 입력값을 즉시 알아야 합니다. 입력창의 값을 state와 연결해두면, 타이핑할 때마다 React가 바로 감지하고 화면을 업데이트할 수 있습니다. 이것이 **제어 컴포넌트(Controlled Component)**입니다.

반대로 이런 화면도 있습니다.

> 📝 로그인 화면에서 아이디와 비밀번호를 입력하고 "로그인" 버튼을 눌렀을 때 한 번만 값을 읽습니다.
> 

이 경우 타이핑할 때마다 state를 업데이트할 이유가 없습니다. 버튼을 누르는 순간에만 값을 읽으면 충분하니까요. 이것이 **비제어 컴포넌트(Uncontrolled Component)**입니다.

실무에서는 구현할 기능에 따라 두 가지 방식을 선택해서 사용합니다.

</aside>

```jsx
import { useState } from 'react';

function PasswordInput() {
  const [password, setPassword] = useState("");

  // 타이핑할 때마다 state가 업데이트되므로, 실시간으로 조건 계산 가능
  const isValid = password.length >= 8;

  return (
    <div>
      {/* value를 state에 묶어두어 React가 입력창을 통제함 */}
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="비밀번호를 입력하세요"
      />
      {/* React가 password 값을 알고 있으므로 즉시 피드백 가능 */}
      <p style={{ color: isValid ? "green" : "red" }}>
        {isValid ? "✅ 사용 가능한 비밀번호입니다" : "❌ 8자 이상 입력해주세요"}
      </p>
    </div>
  );
}
```

```jsx
import { useRef } from 'react';

function LoginForm() {
	// useRef는 React에서 특정 DOM 요소에 접근하거나, 렌더링과 관계없는 값을 컴포넌트 생명주기 내내 유지하고 싶을 때 사용하는 React Hook
  // DOM 요소에 직접 접근하기 위한 '이름표' 생성
  const idRef = useRef(null);
  const passwordRef = useRef(null);

  const handleLogin = () => {
    // 타이핑 중에는 React가 전혀 관여하지 않다가,
    // 버튼을 누르는 순간에만 값을 꺼내서 읽음
    const id = idRef.current.value;
    const password = passwordRef.current.value;
    console.log("로그인 시도:", id, password);
  };

  return (
    <div>
      {/* value, onChange 없음 — 브라우저가 값을 직접 기억 */}
      <input ref={idRef} placeholder="아이디" />
      <input ref={passwordRef} type="password" placeholder="비밀번호" />
      <button onClick={handleLogin}>로그인</button>
    </div>
  );
}
```

https://ko.react.dev/reference/react/useRef

https://ko.react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components

<aside>
🎓

**비유로 이해하는 제어 컴포넌트와 비제어 컴포넌트**

- **제어 컴포넌트 (Controlled): "실시간 동기화 문서"**
    
    사용자가 키보드를 칠 때마다 React가 즉시 알아채고 State에 저장합니다. (글자 수 제한, 실시간 비밀번호 검사 등에 사용)
    
- **비제어 컴포넌트 (Uncontrolled): "우체통"**
    
    사용자가 뭘 적든 React는 신경 쓰지 않습니다. 오직 '전송(Submit)' 버튼을 누르는 순간에만 우체통을 열어서 값을 확인합니다.
    
</aside>

### 📊 한눈에 비교하기

| **특징** | **제어 컴포넌트 (Controlled)** | **비제어 컴포넌트 (Uncontrolled)** |
| --- | --- | --- |
| **데이터 관리 주체** | React (useState) | 브라우저 DOM |
| **값을 아는 시점** | 타이핑하는 **실시간**으로 | **제출(Submit)** 버튼 누를 때 한 번 |
| **사용하는 Hook** | useState | useRef |
| **언제 쓰나요?** |   • 실시간 유효성 검사 (비밀번호 확인)
  • 글자 수 제한, 검색어 자동완성 |   • 단순한 폼 제출
  • 파일 업로드 (<input type="file">) |

---

## ✏️ [실습] Props & useState로 재산 상태 관리하기

아래 내용을 완성하면 **"재산 받기" 버튼을 누를 때 부모님의 재산이 자녀에게 이전**되는 기능이 완성됩니다.

---

**App.jsx: useState로 재산 상태 관리하기**

`App.jsx`를 열고 아래 TODO를 완성하세요.

```jsx
import FamilyBanner from "./components/FamilyBanner";
import ParentComponent from "./components/ParentComponent";
import ChildComponent from "./components/ChildComponent";
import "./App.css";

// ================================================================
// 실습 2 — App.jsx에 useState를 추가하여 재산 이전 기능 구현
//
//   [2-1] parentAssets(초기값 500), childAssets(초기값 0) 상태 선언
//   [2-2] handleInherit 함수 작성
//         → childAssets에 parentAssets를 더하고, parentAssets를 0으로
//   [2-3] ParentComponent의 assets prop → parentAssets
//   [2-4] ChildComponent의 assets → childAssets, onReceive → handleInherit
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

- ✅ 정답
    
    ```jsx
    import { useState } from "react";
    ...
    
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

## 🔖 Part 2 핵심 정리

| 개념 | 핵심 포인트 |
| --- | --- |
| useState | `const [값, setter] = useState(초기값)` |
| 상태 변경 | 반드시 setter 함수를 통해 변경 (직접 수정 금지) |
| 배열 상태 | Spread 연산자로 새 배열 생성 (불변성 원칙) |
| Controlled Input | `value={state}`  • `onChange` 로 입력값 제어 |

---

**🤔 화면이 처음 켜질 때 데이터를 가져오려면?**

<aside>
➡️

지금까지 우리는 useState를 사용해 **사용자가 버튼을 누르거나 타이핑을 할 때** 상태를 바꾸고 화면을 업데이트하는 방법을 배웠습니다.

하지만 실제 웹 서비스에서는 사용자의 행동이 없어도 코드가 실행되어야 할 때가 있습니다.

- 채팅방에 처음 들어왔을 때, **서버에서 기존 채팅 내역을 불러와야** 합니다.
- 화면이 켜지자마자 **타이머가 작동**해야 합니다.

React에게 *"이 데이터 요청 코드는 렌더링될 때마다 실행하지 말고, 화면이 처음 켜질 때 딱 한 번만 / 어떤 상태가 변경될 때마다 실행해!"* 라고 지시할 수 있는 도구가 필요합니다.

이처럼 컴포넌트의 렌더링과 무관하게 특정 시점(생명주기)에 코드를 실행하거나, 외부 시스템(서버, 타이머 등)과 연결할 때 사용하는 것이 바로 다음에 배울 **useEffect**입니다.

</aside>
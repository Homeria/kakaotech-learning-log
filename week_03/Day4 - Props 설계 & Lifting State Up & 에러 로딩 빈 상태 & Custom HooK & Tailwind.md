# Day 4-2 · Day 4-3 · Day 4-4 React 심화 정리

> 원본 자료:  
> - `Day4-2 - 컴포넌트 분리 & Props 설계.md`  
> - `Day4-3 - Lifting State Up & 에러 로딩 빈 상태 & Custom Hook.md`  
> - `Day4-4 - Tailwind CSS 스타일링.md`  
>
> 정리 순서: **4-2 컴포넌트 분리 & Props 설계 → 4-3 Lifting State Up & 로딩·에러·빈 상태 & Custom Hook → 4-4 Tailwind CSS 스타일링**

---

# 1. Day 4-2 — 컴포넌트 분리와 Props 설계

## 1.1 컴포넌트 분리 전략

React에서 컴포넌트를 분리하는 이유는 단순히 파일을 나누기 위해서가 아니다.  
핵심은 **각 컴포넌트가 하나의 역할에 집중하도록 만드는 것**이다.

하나의 컴포넌트가 너무 많은 일을 하면 코드가 복잡해지고, 버그를 찾기도 어려워진다.

---

## 1.2 단일 책임 원칙, SRP

**SRP(Single Responsibility Principle)**는 하나의 컴포넌트가 하나의 책임만 가지도록 설계하는 원칙이다.

안 좋은 구조는 다음과 같다.

```jsx
function App() {
  // 1. 전체 상태 관리
  // 2. 메시지 목록 화면 그리기
  // 3. 개별 메시지 디자인
  // 4. 입력창 UI 처리
  // 5. 전송 버튼 이벤트 처리
}
```

위 구조는 `App.jsx` 하나가 너무 많은 역할을 맡는다.  
기능이 늘어날수록 어디를 고쳐야 하는지 찾기 어려워지고, 협업 시 충돌 가능성도 커진다.

---

## 1.3 좋은 컴포넌트 분리 예시

역할별로 컴포넌트를 나누면 다음처럼 구조가 명확해진다.

```jsx
function ChatApp() {
  return (
    <main>
      <Header />
      <MessageList />
      <MessageInput />
    </main>
  );
}
```

| 컴포넌트 | 책임 |
|---|---|
| `App` | 전체 흐름과 조립 담당 |
| `Header` | 상단 제목이나 상태 표시 |
| `MessageList` | 메시지 목록 렌더링 |
| `MessageItem` | 메시지 하나의 UI 담당 |
| `MessageInput` | 입력창과 전송 이벤트 담당 |

---

## 1.4 컴포넌트를 나누면 좋은 경우

컴포넌트는 무조건 작게 쪼개는 것이 아니라, 분리할 이유가 있을 때 나누는 것이 좋다.

| 분리 기준 | 설명 |
|---|---|
| 같은 UI가 여러 곳에서 반복될 때 | 재사용 가능한 컴포넌트로 만들면 중복을 줄일 수 있다. |
| 한 컴포넌트가 너무 많은 역할을 맡을 때 | 단일 책임 원칙에 맞게 역할별로 나눈다. |
| 특정 부분만 상태가 자주 변할 때 | 자주 변하는 부분을 독립 컴포넌트로 분리하면 관리가 쉬워진다. |
| 코드가 너무 길어질 때 | JSX와 로직이 길어져 읽기 어려우면 분리한다. |
| 협업자가 동시에 작업해야 할 때 | 파일을 나누면 충돌 가능성을 줄일 수 있다. |

---

## 1.5 재사용성을 위한 분리

같은 UI를 여러 곳에서 반복한다면 컴포넌트로 분리하는 것이 좋다.

```jsx
// 나쁜 예: 같은 스타일을 반복 작성
<button style={{ background: "blue", borderRadius: "8px" }}>로그인</button>
<button style={{ background: "blue", borderRadius: "8px" }}>결제하기</button>
```

```jsx
// 좋은 예: 공통 버튼 컴포넌트로 재사용
function PrimaryButton({ text }) {
  return <button className="primary-button">{text}</button>;
}

<PrimaryButton text="로그인" />
<PrimaryButton text="결제하기" />
```

---

## 1.6 역할 분리를 위한 분리

하나의 컴포넌트 안에 여러 기능이 섞여 있으면 각 역할을 담당하는 컴포넌트로 분리한다.

```jsx
function InstagramFeed() {
  return (
    <article>
      <UserProfile />
      <FeedContent />
      <LikeButton />
      <CommentSection />
    </article>
  );
}
```

| 컴포넌트 | 역할 |
|---|---|
| `UserProfile` | 사용자 프로필 표시 |
| `FeedContent` | 본문 텍스트 표시 |
| `LikeButton` | 좋아요 기능 담당 |
| `CommentSection` | 댓글 입력과 목록 담당 |

---

## 1.7 렌더링 최적화를 위한 분리

상태가 자주 바뀌는 부분을 독립된 컴포넌트로 분리하면, 변경 범위를 좁게 만들 수 있다.

```jsx
function QuantitySelector() {
  const [count, setCount] = useState(1);

  return (
    <button onClick={() => setCount(count + 1)}>
      수량: {count}
    </button>
  );
}
```

수량이 바뀔 때마다 전체 쇼핑 페이지가 아니라 `QuantitySelector` 중심으로 관리할 수 있다.

---

# 2. 파일 분리 구조

## 2.1 React 프로젝트 폴더 구조 예시

```text
src/
├── App.jsx
├── main.jsx
│
├── api/
│   ├── axiosInstance.js
│   └── chatApi.js
│
├── components/
│   ├── common/
│   │   └── Button.jsx
│   ├── chat/
│   │   ├── MessageList.jsx
│   │   └── MessageInput.jsx
│   └── UserProfile.jsx
│
├── hooks/
│   └── useChat.js
│
├── utils/
│   └── formatDate.js
│
└── styles/
    └── global.css
```

---

## 2.2 폴더별 역할

| 폴더/파일 | 역할 |
|---|---|
| `App.jsx` | 앱의 최상위 컴포넌트. 라우터나 전역 Provider 조립 |
| `main.jsx` | React 앱 시작점. DOM에 앱을 마운트 |
| `api/` | 서버와의 데이터 통신 담당 |
| `components/` | 화면을 구성하는 UI 컴포넌트 |
| `components/common/` | 여러 곳에서 재사용하는 공통 컴포넌트 |
| `components/chat/` | 특정 도메인 기능 단위 컴포넌트 |
| `hooks/` | 복잡한 상태 로직을 분리한 Custom Hook |
| `utils/` | React와 무관한 순수 JavaScript 함수 |
| `styles/` | 전역 스타일이나 공통 CSS |

---

## 2.3 파일 분리의 효과

| 효과 | 설명 |
|---|---|
| 협업 효율 증가 | 팀원들이 서로 다른 파일을 작업해 충돌 가능성을 줄인다. |
| 유지보수 편리 | 특정 기능 수정 시 관련 파일만 확인하면 된다. |
| 재사용성 증가 | 공통 UI를 여러 곳에서 재사용할 수 있다. |
| 가독성 향상 | 컴포넌트의 역할이 명확해진다. |
| 테스트 용이 | 작은 단위의 컴포넌트와 함수는 테스트하기 쉽다. |

---

# 3. Props와 단방향 데이터 흐름

## 3.1 부모와 자식 컴포넌트

React에서는 컴포넌트 안에 다른 컴포넌트를 포함할 수 있다.  
이때 포함하는 쪽을 **부모 컴포넌트**, 포함되는 쪽을 **자식 컴포넌트**라고 한다.

```jsx
function App() {
  return (
    <div>
      <UserProfile />
    </div>
  );
}
```

위 예시에서 `App`은 부모이고, `UserProfile`은 자식이다.

---

## 3.2 Props란?

Props는 부모가 자식에게 데이터를 전달할 때 사용하는 값이다.  
비유하면 부모가 자식에게 보내는 **택배 상자**와 같다.

```jsx
function App() {
  return <UserProfile userName="김민준" />;
}

function UserProfile({ userName }) {
  return <p>현재 사용자: {userName}</p>;
}
```

---

## 3.3 Props의 핵심 특징

| 특징 | 설명 |
|---|---|
| 부모 → 자식 방향 | React의 데이터는 위에서 아래로 흐른다. |
| 읽기 전용 | 자식은 props를 직접 수정하면 안 된다. |
| 전달 데이터 다양 | 숫자, 문자열, 불리언, 배열, 객체, 함수 전달 가능 |
| 구조 분해 할당 가능 | `{ userName }`처럼 필요한 값만 꺼내 쓴다. |

---

## 3.4 자식이 Props를 직접 수정하면 안 되는 이유

자식 컴포넌트 안에서 props 값을 강제로 바꿔도 부모의 state가 바뀌는 것은 아니다.  
즉, 화면이 원하는 방식으로 업데이트되지 않는다.

```jsx
function UserProfile({ userName }) {
  const handleClick = () => {
    userName = "새로운 이름";
    console.log(userName);
  };

  return (
    <div>
      <p>현재 사용자: {userName}</p>
      <button onClick={handleClick}>이름 바꾸기</button>
    </div>
  );
}
```

props는 부모가 내려주는 읽기 전용 데이터로 생각해야 한다.

---

## 3.5 단방향 데이터 흐름

React는 의도적으로 단방향 데이터 흐름을 사용한다.

```text
부모 state
→ props
→ 자식 컴포넌트
→ 화면 렌더링
```

자식이 부모의 데이터를 바꾸고 싶다면, 부모가 함수를 만들어 자식에게 props로 전달해야 한다.

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return <Child onIncrease={() => setCount((c) => c + 1)} />;
}

function Child({ onIncrease }) {
  return <button onClick={onIncrease}>증가</button>;
}
```

---

## 3.6 단방향 데이터 흐름의 장단점

| 장점 | 단점 |
|---|---|
| 데이터가 어디서 내려오는지 추적하기 쉽다. | 깊은 자식에게 데이터를 넘기려면 Props Drilling이 발생할 수 있다. |
| 자식이 부모 데이터를 몰래 바꾸지 않아 예측 가능하다. | 형제 컴포넌트끼리 직접 데이터를 주고받을 수 없다. |
| 버그 발생 시 부모 컴포넌트부터 확인하면 된다. | 형제끼리 상태 공유가 필요하면 공통 부모로 상태를 끌어올려야 한다. |

---

## 3.7 Props로 전달할 수 있는 것들

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <Child
      number={42}
      text="안녕하세요"
      isActive={true}
      list={[1, 2, 3]}
      obj={{ name: "김민준" }}
      onClick={() => setCount((c) => c + 1)}
    />
  );
}
```

| 전달값 | 예시 |
|---|---|
| 숫자 | `number={42}` |
| 문자열 | `text="안녕하세요"` |
| 불리언 | `isActive={true}` |
| 배열 | `list={[1, 2, 3]}` |
| 객체 | `obj={{ name: "김민준" }}` |
| 함수 | `onClick={() => ...}` |

---

## 3.8 이벤트 핸들러 Props 네이밍

React에서는 부모가 자식에게 함수를 넘겨 이벤트를 처리하게 하는 경우가 많다.

이때 함수 props 이름은 보통 `on + 동사` 형태로 작성한다.

| 이름 | 의미 |
|---|---|
| `onClick` | 클릭했을 때 실행 |
| `onSend` | 전송할 때 실행 |
| `onChange` | 값이 바뀔 때 실행 |
| `onRetry` | 다시 시도할 때 실행 |
| `onAdd` | 추가할 때 실행 |

---

# 4. 실습 — 컴포넌트 분리와 Props 설계

## 4.1 기존 App.jsx의 문제

아래처럼 헤더, 게시글 목록, 게시글 카드, 푸터가 모두 `App.jsx`에 들어 있으면 App이 너무 많은 일을 하게 된다.

```jsx
export default function App() {
  return (
    <div className="app">
      <header className="header">
        <h1>📚 React 피드</h1>
        <span className="online-badge">● 3명 접속 중</span>
      </header>

      <ul className="post-list">
        {POSTS.map(post => (
          <li key={post.id} className="post-card">
            <div className="post-author">
              <span className="avatar">{post.avatar}</span>
              <strong>{post.author}</strong>
            </div>
            <p className="post-content">{post.content}</p>
          </li>
        ))}
      </ul>

      <footer className="stats-footer">
        게시글 {POSTS.length}개
      </footer>
    </div>
  );
}
```

---

## 4.2 책임 분석

| 코드 영역 | 하는 일 | 분리할 컴포넌트/파일 |
|---|---|---|
| `const POSTS = [...]` | 게시글 목록 데이터 정의 | `data/posts.js` |
| `<header>` | 앱 제목과 접속자 수 표시 | `Header` |
| `<li className="post-card">` | 게시글 하나의 작성자와 본문 표시 | `PostCard` |
| `<footer>` | 전체 또는 검색 결과 게시글 수 요약 | `StatsFooter` |

---

## 4.3 PostCard 분리 예시

```jsx
export default function PostCard({ avatar, author, content }) {
  return (
    <li className="post-card">
      <div className="post-author">
        <span className="avatar">{avatar}</span>
        <strong>{author}</strong>
      </div>
      <p className="post-content">{content}</p>
    </li>
  );
}
```

---

## 4.4 분리 후 App.jsx 예시

```jsx
import { POSTS } from "@/data/posts";
import Header from "@/components/Header";
import PostCard from "@/components/PostCard";
import StatsFooter from "@/components/StatsFooter";
import "@/App.css";

export default function App() {
  return (
    <div className="app">
      <Header onlineCount={3} />

      <ul className="post-list">
        {POSTS.map((post) => (
          <PostCard
            key={post.id}
            avatar={post.avatar}
            author={post.author}
            content={post.content}
          />
        ))}
      </ul>

      <StatsFooter count={POSTS.length} />
    </div>
  );
}
```

---

# 5. Day 4-2 핵심 정리

| 개념 | 핵심 포인트 |
|---|---|
| 컴포넌트 분리 | 역할별로 UI를 나누어 관리한다. |
| 단일 책임 원칙 | 하나의 컴포넌트는 하나의 책임에 집중한다. |
| Props | 부모가 자식에게 전달하는 데이터이다. |
| 단방향 데이터 흐름 | 데이터는 부모에서 자식으로 흐른다. |
| 이벤트 핸들러 Props | 자식이 부모의 함수를 호출해 부모 상태 변경을 요청한다. |
| Props Drilling | 깊은 자식에게 props를 계속 전달해야 하는 문제이다. |

---

# 6. Day 4-3 — Lifting State Up

## 6.1 Lifting State Up이 필요한 이유

컴포넌트를 분리하다 보면 서로 다른 자식 컴포넌트가 같은 데이터를 공유해야 할 때가 있다.

예를 들어 쇼핑몰 화면을 다음처럼 나누었다고 하자.

| 컴포넌트 | 역할 |
|---|---|
| `Header` | 우측 상단에 총 장바구니 개수를 표시 |
| `ProductList` | 상품 목록과 장바구니 담기 버튼 표시 |

사용자가 `ProductList`에서 담기 버튼을 누르면 `Header`의 장바구니 개수도 증가해야 한다.  
즉, 두 컴포넌트가 같은 `cartCount` 상태를 공유해야 한다.

---

## 6.2 React에서 형제끼리 직접 데이터를 주고받을 수 없는 이유

React의 데이터 흐름은 부모에서 자식 방향으로만 흐른다.  
따라서 형제 컴포넌트끼리는 직접 상태를 주고받을 수 없다.

```text
부모
├── Header
└── ProductList
```

`Header`와 `ProductList`는 형제 관계이므로, `ProductList`가 `Header`의 상태를 직접 바꿀 수 없다.

---

## 6.3 상태 끌어올리기 핵심

두 컴포넌트가 같은 데이터를 공유해야 한다면, 그들의 **공통 부모가 상태를 소유**해야 한다.

```text
ShoppingApp
├── ShoppingHeader ← count를 props로 받음
└── ProductList ← onAdd 함수를 props로 받음
```

| 대상 | 필요한 props |
|---|---|
| `ShoppingHeader` | 현재 장바구니 개수 `count` |
| `ProductList` | 장바구니 개수를 증가시키는 함수 `onAdd` |
| `ShoppingApp` | 실제 `cartCount` 상태와 `setCartCount` 소유 |

---

## 6.4 Lifting State Up 예시

```jsx
import { useState } from "react";
import ShoppingHeader from "../shared/components/ShoppingHeader";
import ProductList from "../shared/components/ProductList";
import { PRODUCTS } from "../shared/data/products";
import "@/App.css";

export default function ShoppingApp() {
  const [cartCount, setCartCount] = useState(0);

  const handleAddToCart = () => {
    setCartCount((c) => c + 1);
  };

  return (
    <div className="demo-app">
      <ShoppingHeader count={cartCount} />
      <ProductList products={PRODUCTS} onAdd={handleAddToCart} />
    </div>
  );
}
```

---

## 6.5 자식 컴포넌트 예시

```jsx
function ShoppingHeader({ count }) {
  return (
    <header className="shop-header">
      <h1>🛍 쇼핑몰</h1>
      <span className="cart-badge">🛒 {count}개</span>
    </header>
  );
}
```

```jsx
function ProductList({ products, onAdd }) {
  return (
    <ul className="product-list">
      {products.map((product) => (
        <li key={product.id} className="product-item">
          <span>{product.name} — {product.price.toLocaleString()}원</span>
          <button className="add-btn" onClick={onAdd}>담기</button>
        </li>
      ))}
    </ul>
  );
}
```

---

## 6.6 상태 끌어올리기 정리

| 상황 | 해결 |
|---|---|
| 두 자식이 같은 state를 봐야 함 | 공통 부모가 state를 가진다. |
| 한 자식이 상태를 보여줘야 함 | state 값을 props로 내려준다. |
| 다른 자식이 상태를 바꿔야 함 | 상태 변경 함수를 props로 내려준다. |
| 형제끼리 직접 통신하고 싶음 | 직접 통신하지 않고 부모를 통해 연결한다. |

---

# 7. 로딩·에러·빈 상태 처리

## 7.1 실제 서비스에서 필요한 상태

실제 서비스에서는 데이터를 서버에서 불러오기 때문에 항상 성공적인 데이터만 존재하지 않는다.  
다음 세 가지 상황을 반드시 처리해야 한다.

| 상태 | 상황 | 필요한 UI |
|---|---|---|
| 로딩 상태 | 데이터를 불러오는 중 | 로딩 스피너, 안내 문구 |
| 에러 상태 | 네트워크 오류나 서버 오류 | 에러 메시지, 재시도 버튼 |
| 빈 상태 | 데이터가 없음 | 빈 상태 안내 문구 |

---

## 7.2 왜 상태 처리가 중요한가?

사용자는 화면이 비어 있으면 앱이 고장났다고 생각할 수 있다.  
따라서 데이터가 아직 오지 않은 것인지, 실패한 것인지, 데이터가 없는 것인지를 명확히 보여줘야 한다.

| 처리하지 않은 경우 | 사용자 인식 |
|---|---|
| 로딩 표시 없음 | 버튼을 눌렀는데 반응이 없는 것처럼 보임 |
| 에러 메시지 없음 | 네트워크 문제인지 앱 문제인지 알 수 없음 |
| 빈 상태 안내 없음 | 데이터가 없는 건지 로딩 중인지 구분 불가 |

---

## 7.3 데이터 로딩 상태 구성

```jsx
const [products, setProducts] = useState([]);
const [isLoading, setIsLoading] = useState(true);
const [error, setError] = useState(null);
const [fetchIndex, setFetchIndex] = useState(0);
```

| 상태 | 의미 |
|---|---|
| `products` | 서버에서 받아온 상품 목록 |
| `isLoading` | 현재 로딩 중인지 여부 |
| `error` | 에러 메시지 |
| `fetchIndex` | 재요청을 위한 트리거 state |

---

## 7.4 try / catch / finally 패턴

```jsx
useEffect(() => {
  const load = async () => {
    try {
      await new Promise((r) => setTimeout(r, 1000));
      if (Math.random() < 0.3) {
        throw new Error("서버 응답 오류 (시뮬레이션)");
      }
      setProducts(PRODUCTS);
    } catch (err) {
      setError(err.message || "상품을 불러오지 못했습니다. 다시 시도해 주세요.");
    } finally {
      setIsLoading(false);
    }
  };

  load();
}, [fetchIndex]);
```

| 구문 | 역할 |
|---|---|
| `try` | 성공할 수 있는 비동기 작업 실행 |
| `catch` | 에러 발생 시 에러 상태 저장 |
| `finally` | 성공/실패 여부와 관계없이 로딩 종료 처리 |

---

## 7.5 useEffect 내부에서 async 함수를 직접 쓰지 않는 이유

`useEffect`의 콜백 함수는 cleanup 함수를 반환할 수 있어야 한다.  
그런데 `async` 함수는 항상 Promise를 반환한다.

따라서 다음처럼 직접 작성하지 않는다.

```jsx
useEffect(async () => {
  // 권장되지 않음
}, []);
```

대신 effect 내부에서 별도의 async 함수를 만들고 호출한다.

```jsx
useEffect(() => {
  const load = async () => {
    // 비동기 작업
  };

  load();
}, []);
```

---

## 7.6 재시도 기능

에러가 발생했을 때 다시 데이터를 요청하려면, 의존성 배열에 걸린 state를 변경해 effect를 다시 실행시킬 수 있다.

```jsx
const handleRetry = () => {
  setIsLoading(true);
  setError(null);
  setFetchIndex((i) => i + 1);
};
```

`fetchIndex`가 바뀌면 `[fetchIndex]`를 의존성으로 가진 `useEffect`가 다시 실행된다.

---

## 7.7 Early Return 패턴

로딩, 에러, 빈 상태를 JSX 안에서 복잡한 삼항 연산자로 처리하면 가독성이 떨어질 수 있다.  
대신 조건을 순서대로 검사하는 함수를 만들 수 있다.

```jsx
const renderContent = () => {
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} onRetry={handleRetry} />;
  if (products.length === 0) return <EmptyState />;
  return <ProductList products={products} onAdd={handleAddToCart} />;
};
```

---

## 7.8 상태별 렌더링 순서

보통 다음 순서로 처리한다.

```text
1. 로딩 중인가?
2. 에러가 있는가?
3. 데이터가 비어 있는가?
4. 정상 데이터가 있는가?
```

| 우선순위 | 조건 | 렌더링 |
|---|---|---|
| 1 | `isLoading` | `<LoadingSpinner />` |
| 2 | `error` | `<ErrorMessage />` |
| 3 | `products.length === 0` | `<EmptyState />` |
| 4 | 정상 데이터 | `<ProductList />` |

---

## 7.9 TanStack Query와 Suspense 참고

`isLoading`, `error`, `data`를 컴포넌트마다 직접 관리하면 코드가 반복된다.  
실무에서는 다음 도구를 사용하기도 한다.

| 도구 | 역할 |
|---|---|
| TanStack Query | 서버 상태, 캐싱, 재시도, 백그라운드 갱신 관리 |
| Suspense | 로딩 상태를 컴포넌트 트리 단위로 처리 |
| ErrorBoundary | 에러 상태를 컴포넌트 트리 단위로 처리 |

---

# 8. Custom Hook

## 8.1 Custom Hook이 필요한 이유

컴포넌트 안에 상태 선언, 데이터 로딩 로직, UI 렌더링이 모두 섞이면 코드가 길고 복잡해진다.

상품 목록을 불러오는 기능이 여러 페이지에 필요하다면 같은 코드를 복사해야 하는 문제가 생긴다.

이때 **Custom Hook**을 사용하면 상태 로직을 재사용 가능한 함수로 분리할 수 있다.

---

## 8.2 Custom Hook이란?

Custom Hook은 `useState`, `useEffect` 같은 React Hook을 사용한 로직을 별도의 함수로 추출한 것이다.  
함수 이름은 반드시 `use`로 시작한다.

```jsx
function useSomething() {
  const [state, setState] = useState(null);

  useEffect(() => {
    // effect
  }, []);

  return state;
}
```

---

## 8.3 Custom Hook을 쓰는 경우

| 상황 | 설명 |
|---|---|
| 상태 로직이 중복될 때 | 여러 컴포넌트에서 같은 API 통신, 로딩, 에러 처리가 반복 |
| 컴포넌트가 비대해질 때 | JSX보다 데이터 처리 로직이 길어져 가독성이 떨어짐 |
| UI와 로직을 분리하고 싶을 때 | 컴포넌트는 화면에 집중하고 Hook은 데이터 처리 담당 |
| React Hook을 포함한 로직을 재사용할 때 | 일반 함수에서는 Hook을 호출할 수 없으므로 Custom Hook으로 분리 |

---

## 8.4 Custom Hook의 장점

| 장점 | 설명 |
|---|---|
| 재사용성 | 같은 로직을 여러 컴포넌트에서 import하여 사용 가능 |
| 관심사 분리 | UI는 컴포넌트에, 데이터 로직은 Hook에 배치 |
| 유지보수성 | 로직 수정 시 Hook만 고치면 됨 |
| 상태 독립성 | 여러 컴포넌트가 같은 Hook을 사용해도 각자의 상태는 독립적 |

---

## 8.5 Custom Hook 규칙

| 규칙 | 설명 |
|---|---|
| 이름은 `use`로 시작 | 예: `useProducts`, `useFetch`, `useLocalStorage` |
| 내부에서 Hook 사용 가능 | `useState`, `useEffect` 등을 호출할 수 있음 |
| 일반 함수처럼 import 가능 | 필요한 컴포넌트에서 가져와 사용 |
| 모든 로직을 빼지는 않음 | 복잡하거나 중복되는 관심사만 분리 |

---

## 8.6 useProducts 예시

```jsx
import { useState, useEffect } from "react";
import { PRODUCTS } from "../../shared/data/products";

export function useProducts() {
  const [products, setProducts] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  const [fetchIndex, setFetchIndex] = useState(0);

  useEffect(() => {
    const load = async () => {
      try {
        await new Promise((r) => setTimeout(r, 1000));
        if (Math.random() < 0.3) {
          throw new Error("서버 응답 오류 (시뮬레이션)");
        }
        setProducts(PRODUCTS);
      } catch (err) {
        setError(err.message || "상품을 불러오지 못했습니다. 다시 시도해 주세요.");
      } finally {
        setIsLoading(false);
      }
    };

    load();
  }, [fetchIndex]);

  const reload = () => {
    setIsLoading(true);
    setError(null);
    setFetchIndex((i) => i + 1);
  };

  return { products, isLoading, error, reload };
}
```

---

## 8.7 Hook 적용 후 App.jsx

Custom Hook으로 데이터 로딩 로직을 분리하면 App 컴포넌트는 UI 조립에 더 집중할 수 있다.

```jsx
import { useState } from "react";
import { useProducts } from "./hooks/useProducts";
import ShoppingHeader from "../shared/components/ShoppingHeader";
import ProductList from "../shared/components/ProductList";
import Loading from "../shared/components/Loading";
import ErrorMessage from "../shared/components/ErrorMessage";
import EmptyState from "../shared/components/EmptyState";
import "@/App.css";

export default function ShoppingApp() {
  const { products, isLoading, error, reload } = useProducts();
  const [cartCount, setCartCount] = useState(0);

  const handleAddToCart = () => setCartCount((c) => c + 1);

  const renderContent = () => {
    if (isLoading) return <Loading />;
    if (error) return <ErrorMessage message={error} onRetry={reload} />;
    if (products.length === 0) return <EmptyState />;
    return <ProductList products={products} onAdd={handleAddToCart} />;
  };

  return (
    <div className="demo-app">
      <ShoppingHeader count={cartCount} />
      <div className="demo-content">{renderContent()}</div>
    </div>
  );
}
```

---

# 9. Day 4-3 핵심 정리

| 개념 | 핵심 포인트 |
|---|---|
| Lifting State Up | 여러 자식이 공유하는 상태는 공통 부모가 소유한다. |
| 함수 props | 자식이 부모 상태 변경을 요청할 때 함수를 내려준다. |
| 로딩 상태 | 데이터를 불러오는 중임을 사용자에게 알려준다. |
| 에러 상태 | 실패 이유와 재시도 방법을 제공한다. |
| 빈 상태 | 데이터가 없다는 사실을 명확히 보여준다. |
| Early Return | 조건별 UI를 순서대로 반환해 가독성을 높인다. |
| Custom Hook | 반복되는 상태 로직을 재사용 가능한 Hook으로 분리한다. |

---

# 10. Day 4-4 — Tailwind CSS 스타일링

## 10.1 Tailwind CSS란?

Tailwind CSS는 웹사이트나 앱의 디자인을 빠르게 꾸밀 수 있게 해주는 CSS 프레임워크이다.

별도의 CSS 파일에 클래스명을 직접 만들지 않고, JSX의 `className`에 미리 정의된 유틸리티 클래스를 조합해 스타일을 적용한다.

---

## 10.2 기존 CSS 방식과 Tailwind 방식

기존 CSS 방식:

```tsx
<button className="submit-btn">제출</button>
```

```css
.submit-btn {
  background-color: #2563eb;
  color: white;
  padding: 8px 16px;
  border-radius: 8px;
}
```

Tailwind 방식:

```tsx
<button className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors">
  제출
</button>
```

---

## 10.3 Tailwind의 핵심 아이디어

```text
클래스명을 직접 만들지 않고
미리 정의된 작은 스타일 클래스를 조합한다.
```

예를 들어 다음 클래스들은 각각 하나의 CSS 속성을 의미한다.

| Tailwind 클래스 | 의미 |
|---|---|
| `bg-blue-600` | 배경색을 파란색 계열로 지정 |
| `text-white` | 글자색 흰색 |
| `px-4` | 좌우 padding |
| `py-2` | 상하 padding |
| `rounded-lg` | 둥근 모서리 |
| `hover:bg-blue-700` | hover 시 배경색 변경 |
| `transition-colors` | 색상 변화에 transition 적용 |

---

# 11. Tailwind CSS 장단점

## 11.1 장점

| 장점 | 설명 |
|---|---|
| 클래스명 고민 불필요 | `.submit-btn`, `.card-wrapper` 같은 이름을 직접 짓지 않아도 된다. |
| CSS 파일 관리 감소 | 스타일이 JSX 안에 있어 파일 전환이 줄어든다. |
| 번들 최적화 | 빌드 시 실제 사용된 클래스만 포함해 CSS 크기를 줄인다. |
| 디자인 시스템 내장 | 색상, 간격, 폰트 크기 등이 일관된 스케일로 제공된다. |
| 반응형 처리 간편 | `md:`, `lg:` 같은 접두사로 미디어 쿼리 없이 처리 가능 |
| 상태 처리 간편 | `hover:`, `focus:` 등으로 상태별 스타일 적용 가능 |

---

## 11.2 단점

| 단점 | 설명 |
|---|---|
| `className`이 길어짐 | 스타일이 많을수록 JSX가 길어 보일 수 있다. |
| 초기 러닝커브 | `p-4`가 `padding: 16px`이라는 식의 매핑을 익혀야 한다. |
| 복잡한 커스텀 스타일 처리 | 기본 스케일 밖의 세밀한 값은 설정이나 임의값 문법이 필요하다. |
| JSX 가독성 저하 가능 | 스타일 클래스가 너무 많으면 구조 파악이 어려워질 수 있다. |

---

# 12. Tailwind CSS 설치와 적용

## 12.1 Tailwind CSS v4 특징

Tailwind CSS v4는 기본적으로 `tailwind.config.js` 설정 파일이 필요하지 않다.

---

## 12.2 설치 명령어

```bash
npm install -D tailwindcss @tailwindcss/postcss postcss
```

---

## 12.3 PostCSS 설정

```jsx
// postcss.config.mjs
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};

export default config;
```

---

## 12.4 글로벌 CSS에 Tailwind 불러오기

```css
/* app/globals.css */
@import "tailwindcss";
```

---

## 12.5 최상위 레이아웃에 CSS 로드

```tsx
import "./globals.css";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>{children}</body>
    </html>
  );
}
```

---

# 13. Tailwind 기본 문법

## 13.1 기본 적용

HTML이나 JSX의 `className`에 Tailwind 클래스를 띄어쓰기로 나열하면 된다.

```tsx
<div className="bg-white p-6 rounded-xl shadow-sm border border-gray-200">
  <h2 className="text-xl font-bold text-gray-900">제목</h2>
  <p className="text-sm text-gray-500 mt-2">설명 텍스트</p>
</div>
```

---

## 13.2 자주 쓰는 기본 클래스

| 분류 | 예시 | 의미 |
|---|---|---|
| 배경색 | `bg-white`, `bg-blue-600` | 배경색 지정 |
| 글자색 | `text-gray-900`, `text-white` | 글자색 지정 |
| 글자 크기 | `text-sm`, `text-xl`, `text-3xl` | 폰트 크기 지정 |
| 굵기 | `font-medium`, `font-bold` | 폰트 굵기 지정 |
| 여백 | `p-4`, `px-6`, `mt-2` | padding, margin 지정 |
| 테두리 | `border`, `border-gray-200` | 테두리 지정 |
| 둥근 모서리 | `rounded`, `rounded-lg`, `rounded-xl` | border-radius 지정 |
| 그림자 | `shadow-sm`, `shadow` | box-shadow 지정 |
| 레이아웃 | `flex`, `grid` | 레이아웃 방식 지정 |

---

# 14. 반응형 스타일

## 14.1 반응형 접두사

Tailwind에서는 `sm:`, `md:`, `lg:`, `xl:` 접두사로 화면 크기에 따라 다른 스타일을 적용할 수 있다.

```tsx
<div className="flex flex-col md:flex-row gap-4">
  <div>왼쪽</div>
  <div>오른쪽</div>
</div>
```

위 코드는 기본적으로 세로 배치이고, `md` 크기 이상에서는 가로 배치가 된다.

---

## 14.2 반응형 접두사 기준

| 접두사 | 최소 너비 | 대상 기기 |
|---|---|---|
| 없음 | 0px 이상 | 모든 크기, 기본값 |
| `sm:` | 640px 이상 | 작은 태블릿 |
| `md:` | 768px 이상 | 태블릿 |
| `lg:` | 1024px 이상 | 데스크톱 |
| `xl:` | 1280px 이상 | 큰 데스크톱 |

---

## 14.3 반응형 예시

```tsx
<h1 className="text-xl lg:text-3xl font-bold">
  타이틀
</h1>
```

기본 화면에서는 `text-xl`, `lg` 이상에서는 `text-3xl`이 적용된다.

---

# 15. 상태 변화 스타일

## 15.1 hover와 focus

Tailwind에서는 `hover:`, `focus:` 접두사를 사용해 상태에 따른 스타일을 적용할 수 있다.

```tsx
<button className="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg transition-colors">
  버튼
</button>
```

---

## 15.2 focus 예시

```tsx
<input className="border border-gray-300 focus:border-blue-500 focus:ring-2 focus:ring-blue-100 outline-none px-3 py-2 rounded-lg" />
```

| 접두사 | 의미 |
|---|---|
| `hover:` | 마우스를 올렸을 때 |
| `focus:` | 입력창이나 버튼에 포커스가 갔을 때 |
| `active:` | 클릭 중일 때 |
| `disabled:` | 비활성화 상태일 때 |

---

# 16. 임의값, Arbitrary Values

## 16.1 임의값이란?

Tailwind 기본 스케일에 없는 정확한 값이 필요할 때 `[값]` 형태로 작성할 수 있다.

```tsx
<div className="w-[380px] text-[13px] bg-[#f0f4ff] mt-[30px]">
  정확한 수치가 필요할 때
</div>
```

---

## 16.2 임의값 사용 시 주의점

임의값은 디자이너 시안의 정확한 수치를 반영할 때 유용하다.  
하지만 너무 많이 사용하면 Tailwind의 일관성 있는 디자인 시스템이라는 장점이 약해진다.

| 권장 | 비권장 |
|---|---|
| 기본 스케일을 먼저 사용 | 모든 값을 임의값으로 작성 |
| 꼭 필요한 경우에만 `[값]` 사용 | `p-[17px]`, `mt-[13px]`처럼 지나치게 세밀한 값 남발 |
| 디자인 시스템 유지 | 프로젝트 전체의 간격과 크기가 제각각이 됨 |

---

# 17. Flexbox와 Tailwind

## 17.1 Flexbox란?

Flexbox는 웹 페이지의 레이아웃을 유연하고 효율적으로 배치하기 위한 CSS 레이아웃 방식이다.

Tailwind에서는 Flexbox 관련 속성을 짧은 클래스로 적용할 수 있다.

```tsx
<div className="flex items-center justify-between gap-4">
  <span>프로필</span>
  <button>설정</button>
</div>
```

---

## 17.2 Flexbox 핵심 개념

| 개념 | 설명 |
|---|---|
| Container | 정렬할 요소들을 감싸는 부모 태그 |
| Item | 컨테이너 안에 나열되는 자식 태그 |
| Main Axis | 요소들이 배치되는 주축 |
| Cross Axis | 주축에 수직인 교차축 |

---

## 17.3 방향, Direction

| 클래스 | 의미 |
|---|---|
| `flex-row` | 가로 배치, 기본값 |
| `flex-col` | 세로 배치 |

```tsx
<div className="flex flex-col">
  <div>위</div>
  <div>아래</div>
</div>
```

---

## 17.4 주축 정렬, justify

| 클래스 | 의미 |
|---|---|
| `justify-start` | 왼쪽 또는 위쪽으로 붙임 |
| `justify-center` | 가운데 정렬 |
| `justify-between` | 양 끝으로 벌림 |
| `justify-end` | 오른쪽 또는 아래쪽으로 붙임 |

---

## 17.5 교차축 정렬, items

| 클래스 | 의미 |
|---|---|
| `items-start` | 위쪽 또는 왼쪽 정렬 |
| `items-center` | 교차축 가운데 정렬 |
| `items-end` | 아래쪽 또는 오른쪽 정렬 |

---

## 17.6 간격, gap

`gap`은 요소 사이의 간격을 일정하게 띄울 때 사용한다.

| 클래스 | 의미 |
|---|---|
| `gap-2` | 작은 간격 |
| `gap-4` | 보통 간격, 16px |
| `gap-6` | 더 큰 간격 |

```tsx
<div className="flex gap-4">
  <button>버튼 1</button>
  <button>버튼 2</button>
</div>
```

---

# 18. Tailwind CSS 실습

## 18.1 실습 1 — 텍스트 크기와 굵기

문제: 제목 `Tailwind 연습`의 글자 크기를 `3xl`, 굵기를 `bold`로 맞추기

```tsx
<h3 className="text-3xl font-bold">Tailwind 연습</h3>
```

---

## 18.2 실습 2 — 반응형

문제: 제목을 모바일에서는 `text-xl`, `md` 이상에서는 `text-4xl`로 적용하기

```tsx
<h3 className="text-xl md:text-4xl">
  화면 크기에 따라 달라지는 제목
</h3>
```

---

## 18.3 실습 3 — Hover 상태 변화

문제: 버튼 배경은 `indigo-600`, 글씨는 흰색, hover 시 배경은 `indigo-800`로 만들기

```tsx
<button
  type="button"
  className="inline-flex items-center rounded-lg bg-indigo-600 px-6 py-3 text-sm font-medium text-white hover:bg-indigo-800"
>
  호버해보세요
</button>
```

---

## 18.4 실습 4 — Flex, Margin, Padding

문제: 프로필 바를 가로 배치, 양끝 정렬, 세로 가운데 정렬, padding, 흰 배경, 둥근 모서리로 만들기

```tsx
<div className="flex items-center justify-between rounded-lg bg-white p-4">
  <span className="font-medium text-slate-800">프로필</span>
  <button
    type="button"
    className="rounded-md px-4 py-2 text-sm text-indigo-600 underline"
  >
    설정
  </button>
</div>
```

---

## 18.5 Tailwind 개발 도구

| 도구 | 역할 |
|---|---|
| Tailwind CSS IntelliSense | VS Code에서 Tailwind 클래스 자동 완성과 힌트 제공 |
| prettier-plugin-tailwindcss | 복잡한 클래스명을 규칙에 맞게 자동 정렬 |

---

# 19. Day 4-4 핵심 정리

| 개념 | 핵심 포인트 |
|---|---|
| Tailwind CSS | 유틸리티 클래스를 조합해 스타일링하는 CSS 프레임워크 |
| className | JSX 안에서 Tailwind 클래스를 작성하는 위치 |
| 반응형 접두사 | `sm:`, `md:`, `lg:` 등으로 화면 크기별 스타일 적용 |
| 상태 접두사 | `hover:`, `focus:` 등으로 상태별 스타일 적용 |
| 임의값 | `w-[380px]`처럼 기본 스케일에 없는 값을 직접 지정 |
| Flexbox | `flex`, `justify-*`, `items-*`, `gap-*`으로 레이아웃 구성 |
| IntelliSense | Tailwind 클래스 자동 완성 도구 |
| prettier plugin | Tailwind 클래스 정렬 도구 |

---

# 20. 전체 핵심 요약

## 20.1 Day 4-2 ~ Day 4-4 흐름

이번 내용은 React 앱을 실제 프로젝트처럼 구조화하고 다듬는 과정이다.

```text
컴포넌트 분리
→ Props 설계
→ 상태 공유가 필요하면 Lifting State Up
→ 서버 데이터는 로딩·에러·빈 상태 처리
→ 반복되는 로직은 Custom Hook으로 분리
→ Tailwind CSS로 빠르게 스타일링
```

---

## 20.2 React 구조 설계 핵심

| 개념 | 질문 | 판단 기준 |
|---|---|---|
| 컴포넌트 분리 | 이 컴포넌트가 너무 많은 일을 하나? | 역할이 2개 이상이면 분리 고려 |
| Props 설계 | 자식에게 무엇이 필요한가? | 필요한 값과 함수만 내려주기 |
| Lifting State Up | 여러 컴포넌트가 같은 state를 쓰나? | 공통 부모가 state 소유 |
| 로딩/에러/빈 상태 | 서버 요청 결과가 항상 성공하는가? | 실패와 빈 결과까지 UI로 표현 |
| Custom Hook | 같은 상태 로직이 반복되는가? | 중복되는 로직을 `use*` 함수로 추출 |
| Tailwind CSS | 빠르게 일관된 스타일을 적용할 것인가? | 유틸리티 클래스로 JSX에서 직접 스타일링 |

---

## 20.3 최종 정리

React 프로젝트가 커질수록 중요한 것은 단순히 컴포넌트를 많이 만드는 것이 아니라, **역할과 데이터 흐름을 기준으로 구조를 설계하는 것**이다.  
하나의 컴포넌트가 너무 많은 일을 하면 유지보수가 어려워지므로, 단일 책임 원칙에 따라 UI를 적절히 분리해야 한다.

Props는 부모가 자식에게 데이터를 전달하는 방법이며, React의 데이터 흐름은 기본적으로 부모에서 자식 방향으로 흐른다.  
자식이 부모의 상태를 직접 바꾸는 것이 아니라, 부모가 상태 변경 함수를 props로 내려주고 자식이 그 함수를 호출하는 방식으로 동작한다.

형제 컴포넌트가 같은 상태를 공유해야 할 때는 상태를 공통 부모로 끌어올린다.  
이것이 Lifting State Up이며, React에서 상태 공유를 해결하는 기본 패턴이다.

서버 데이터를 다룰 때는 정상 데이터만 고려하면 안 된다.  
로딩 중, 에러 발생, 데이터 없음 상태를 모두 UI로 표현해야 사용자가 앱의 현재 상태를 이해할 수 있다.  
이런 데이터 로딩 로직이 반복되면 Custom Hook으로 분리하여 UI와 로직을 나누는 것이 좋다.

마지막으로 Tailwind CSS는 CSS 파일을 따로 작성하기보다 JSX의 `className`에 유틸리티 클래스를 조합해 빠르게 스타일링하는 방식이다.  
반응형, hover/focus 상태, Flexbox 레이아웃 등을 짧은 클래스 조합으로 처리할 수 있어 React 프로젝트에서 빠르게 UI를 구성할 때 유용하다.

전체적으로 이번 파트의 핵심은 다음과 같다.

```text
1. 컴포넌트는 역할별로 분리한다.
2. 데이터는 부모에서 자식으로 흐른다.
3. 공유 상태는 공통 부모로 끌어올린다.
4. 서버 요청은 로딩·에러·빈 상태까지 처리한다.
5. 반복되는 로직은 Custom Hook으로 분리한다.
6. 스타일은 Tailwind CSS로 빠르게 일관성 있게 적용할 수 있다.
```

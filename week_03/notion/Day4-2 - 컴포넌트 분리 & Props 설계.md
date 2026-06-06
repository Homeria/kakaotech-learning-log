## 1. 컴포넌트 분리 전략

### 왜 분리하는가 — 단일 책임 원칙(SRP)

하나의 컴포넌트가 "너무 많은 일"을 하면 코드가 꼬이고 버그를 찾기 힘들어집니다.

```jsx
// ❌ 안 좋은 구조: App.jsx 한 파일에 너무 많은 로직을 넣음
function App() { 
  // 1. 전체 상태(메시지 목록, 입력값) 관리
  // 2. 메시지 목록 화면 그리기 (스크롤 처리)
  // 3. 개별 메시지 말풍선 디자인 (색상, 정렬)
  // 4. 입력창 UI 및 전송 버튼 클릭 이벤트 처리
}
```

<aside>
👀

**비유로 알아보는 책임 분리의 중요성**

- **안 좋은 구조 (혼자 다 하기):** 사장님 혼자서 주문 받고, 패티 굽고, 감자튀김 튀기고, 서빙하고, 청소까지 다 합니다. 손님이 몰리면 가게가 마비됩니다.
- **좋은 구조 (책임 분리):**
    - App (매니저): 전체 흐름 관리 (주문서 전달)
    - 컴포넌트 1 (캐셔): 손님 주문(입력)만 받음
    - 컴포넌트 2 (주방장): 주문서(데이터)를 보고 요리(목록)만 만듦
    - 컴포넌트 3 (포장직원): 요리 하나(메시지 하나)를 예쁘게 포장만 함

만약 감자튀김 만드는 로직에서 버그가 발생했다면(감자튀김이 탔다면)? 주방장 역할하는 컴포넌트 2 로직만 검토하고 수정하면 됩니다.

</aside>

- **[TIP] 컴포넌트를 언제 나누면 좋을까요?**
    
    **1. 같은 UI가 여러 곳에서 반복될 때 (재사용성)**
    
    > "어? 이 버튼 디자인, 아까 로그인 페이지에서도 썼는데?"
    > 
    
    ```jsx
    // ❌ 안 좋은 예: 똑같은 디자인의 버튼을 매번 새로 만듦
    <button style={{ background: 'blue', borderRadius: '8px' }}>로그인</button>
    <button style={{ background: 'blue', borderRadius: '8px' }}>결제하기</button>
    
    // ✅ 좋은 예: <PrimaryButton> 이라는 부품으로 만들어 재사용
    <PrimaryButton text="로그인" />
    <PrimaryButton text="결제하기" />
    ```
    
    **2. 한 컴포넌트가 너무 많은 역할을 맡을 때 (단일 책임 원칙)**
    
    > "인스타그램 피드 하나를 그리는데, 로직 4개가 한 함수에 섞여 있네?"
    > 
    
    ```jsx
    // ❌ 안 좋은 예: 피드 하나에 모든 로직이 다 섞여 있음
    function InstagramFeed() {
      // 유저 프로필 사진 가져오기
      // 본문 텍스트 더보기/접기 로직
      // 좋아요 하트 애니메이션 로직
      // 댓글 입력 및 전송 로직 ... (너무 복잡함!)
    }
    
    // ✅ 좋은 예: 역할별로 잘게 쪼개서 조립함
    function InstagramFeed() {
      return (
        <article>
          <UserProfile />      {/* 프로필만 담당 */}
          <FeedContent />      {/* 본문 텍스트만 담당 */}
          <LikeButton />       {/* 좋아요 기능만 담당 */}
          <CommentSection />   {/* 댓글 기능만 담당 */}
        </article>
      );
    }
    ```
    
    **3. 특정 부분만 상태(State)가 자주 변할 때 (렌더링 최적화)**
    
    > "쇼핑몰에서 상품 수량을 올렸는데, 왜 전체 페이지가 다시 깜빡거리지?"
    > 
    
    ```jsx
    // ❌ 안 좋은 예: 부모가 모든 상태를 다 가지고 있음
    function ShoppingPage() {
      const [count, setCount] = useState(1); // 수량 상태
      // 수량을 올릴 때마다 이 거대한 ShoppingPage 전체가 다시 그려짐
      return (
        <div>
          <Header />
          <ProductImage />
          <button onClick={() => setCount(count + 1)}>수량: {count}</button>
          <Footer />
        </div>
      );
    }
    
    // ✅ 좋은 예: 자주 변하는 상태를 독립된 자식 컴포넌트로 분리
    function QuantitySelector() {
      const [count, setCount] = useState(1);
      // 이제 수량을 올려도 이 작은 컴포넌트만 다시 그려짐 (성능 향상)
      return <button onClick={() => setCount(count + 1)}>수량: {count}</button>;
    }
    ```
    

### 파일 분리 구조 예시

```jsx
src/
├── App.jsx              // 앱의 최상위 컴포넌트 (라우터나 전역 Provider 조립)
├── main.jsx             // 앱의 시작점 (ReactDOM.createRoot가 있는 곳)
│
├── api/                 // [API 통신] 서버와의 데이터 통신을 전담
│   ├── axiosInstance.js // - 공통 설정 (Base URL, 헤더, 토큰 가로채기 등)
│   └── chatApi.js       // - 챗봇/메시지 관련 API 함수들 (fetch, axios 등)
│
├── components/          // [UI 컴포넌트] 화면을 구성하는 부품들의 방
│   ├── common/          // (선택) 서비스 전체에서 돌려쓰는 공통 컴포넌트
│   │   └── Button.jsx   //   - 스타일과 형태만 잡힌 공통 버튼
│   ├── chat/            // (선택) 특정 도메인(기능) 단위로 묶은 컴포넌트
│   │   ├── MessageList.jsx
│   │   └── MessageInput.jsx
│   └── UserProfile.jsx  // 유저 정보 표시 (역할 분리)
│
├── hooks/               // [커스텀 훅] 복잡한 비즈니스 로직(State 등)을 분리
│   └── useChat.js       // - 메시지 읽기/보내기 로직만 쏙 빼서 만든 커스텀 훅
│
├── utils/               // [유틸리티] React와 상관없는 순수 자바스크립트 함수
│   └── formatDate.js    // - "2026-06-02T14:30..." -> "오후 2시 30분" 변환 함수
│
└── styles/              // [스타일] CSS 파일들을 모아두는 방
    └── global.css       // - 전체 리셋 CSS 또는 공통 스타일
```

**파일 분리의 이유와 효과**

1. **협업의 필수 조건:** 5명의 팀원이 App.jsx 한 파일만 동시에 수정하면 코드 충돌(Conflict)이 발생합니다. 파일을 나누면 각자 맡은 파일만 안전하게 작업할 수 있습니다.
2. **유지보수의 편리함:** "버튼 색깔을 바꿔주세요"라는 요청이 왔을 때, 수천 줄의 코드 속을 헤맬 필요 없이 PrimaryButton.jsx 파일만 열어서 수정하면 끝납니다.

---

## 2. Props — 단방향 데이터 흐름

### 부모-자식 관계란?

React에서는 컴포넌트 안에 다른 컴포넌트를 품을 수 있습니다. 이때 **품는 쪽을 '부모', 품어지는 쪽을 '자식'**이라고 부릅니다.

```jsx
function App() { // 부모 컴포넌트
  return (
    <div>
      <UserProfile /> {/* 자식 컴포넌트 */}
    </div>
  );
}
```

### **🧑‍🧑‍🧒‍🧒 Props(Properties)란?**

부모가 자식에게 데이터를 전달할 때 사용하는 **'택배 상자'** 같은 거에요. (React는 데이터가 **부모 → 자식 방향으로만** 흐릅니다. 자식이 부모의 상태를 직접 바꿀 수 없습니다.)

- **자식의 의무:** 자식은 부모가 준 **Props를 직접 수정하는 방식**으로는 부모의 상태를 바꿀 수 없습니다.
- **부모의 권한:** 부모는 자식에게 어떤 데이터를 줄지 결정합니다.

```jsx
import { useState } from "react";

// 자식 컴포넌트
function UserProfile({ userName }) {
  const handleClick = () => {
    // 1. 자식이 전달받은 props 변수를 강제로 바꿉니다.
    userName = "새로운 이름"; 
    
    // 2. 콘솔을 찍어보면 변수 값 자체는 잘 바뀌어 있지만 화면에는 변화가 없습니다.
    console.log("자식 안에서의 userName 변수 값:", userName); 
  };

  return (
    <div>
      <p>현재 사용자: {userName}</p>
      <button onClick={handleClick}>이름 바꾸기 버튼</button>
    </div>
  );
}

// 부모 컴포넌트
export default function App() {
  const [name, setName] = useState("김민준");
  return <UserProfile userName={name} />;
}
```

<aside>
🎓

**단방향 바인딩 vs 양방향 바인딩**

Vue나 Angular는 양방향 바인딩(two-way binding)을 지원합니다. 데이터가 어디서 변경되어도 자동으로 동기화됩니다. React는 의도적으로 **단방향**을 선택했습니다. 데이터 흐름이 항상 위에서 아래로 흐르기 때문에, 버그가 발생했을 때 "어디서 상태가 바뀌었는지"를 추적하기 쉽습니다.

- **단방향 데이터 흐름의 장단점**
    
    
    | **장점** | **단점** |
    | --- | --- |
    | **버그 추적이 쉽다:** 데이터가 무조건 부모에서 내려오므로, 에러가 나면 부모 컴포넌트만 찾아가서 고치면 됩니다. | **Props Drilling (프롭스 드릴링):** 부모에서 까마득히 먼 증손자 컴포넌트에게 데이터를 주려면, 중간에 있는 자식들이 쓰지도 않을 데이터를 계속 전달(전달->전달->전달)해야 하는 귀찮음이 발생합니다. |
    | **코드의 예측 가능성:** 자식이 부모 데이터를 몰래 바꾸는 일이 없으므로, 코드가 어떻게 동작할지 예측하기 쉽습니다. | **상태 관리의 복잡성:** 형제 컴포넌트끼리 데이터를 주고받으려면 반드시 공통 부모까지 상태를 끌어올려야 합니다. |
</aside>

- **(참고) Props로 전달할 수 있는 것들**
    
    ```jsx
    function Parent() {
      const [count, setCount] = useState(0);
    
      return (
        <Child
          number={42}                           // 숫자
          text="안녕하세요"                      // 문자열
          isActive={true}                       // 불리언
          list={[1, 2, 3]}                      // 배열
          obj={{ name: "김민준" }}              // 객체
          onClick={() => setCount(c => c + 1)}  // 함수 (콜백 props)
        />
      );
    }
    
    function Child({ number, text, isActive, list, obj, onClick }) {
      return (
        <div>
          <p>{text} — {number}</p>
          {isActive && <p>활성 상태</p>}
          <button onClick={onClick}>클릭</button>
        </div>
      );
    }
    ```
    
    <aside>
    💡
    
    **💡 JS/React 비교 — 콜백 함수와 이벤트 핸들러**
    
    바닐라 JS에서 `addEventListener("click", handler)` 로 콜백을 등록했다면, React에서는 props로 함수를 전달합니다. `onSend`, `onChange`, `onRetry` 처럼 `on + 동사` 네이밍 관례를 따릅니다. 이것이 React의 **이벤트 핸들러 패턴**입니다.
    
    </aside>
    

---

## ✏️ [실습] 컴포넌트 분리와 Props 설계

현재 헤더, 게시글 3개, 푸터가 **모두 한 파일**에 들어있습니다. 아래 문제를 순서대로 풀어보세요.

---

**문제 1 — 책임 분석하기**

아래 표의 빈칸을 채워보세요. 코딩 없이 분석만 합니다.

| 코드 영역 | 이 코드가 하는 일 | 분리할 컴포넌트 이름 (제안) |
| --- | --- | --- |
| `<header className="header">` |  |  |
| `<li className="post-card">` |  |  |
| `<footer className="stats-footer">` |  |  |
- ✅ 정답
    
    
    | 코드 영역 | 하는 일 | 컴포넌트 이름 |
    | --- | --- | --- |
    | `const POSTS = [...]` | 게시글 목록 데이터 정의 | `data/posts.js` |
    | `<header>` | 앱 제목 + 접속자 수 표시 | `Header` |
    | `<li className="post-card">` | 게시글 하나의 작성자 / 본문 표시 | `PostCard` |
    | `<footer>` | 전체/검색 결과 게시글 수 요약 | `StatsFooter` |

---

**문제 2 — Header / PostCard / StatsFooter 컴포넌트**

```jsx
// 🚨 이 App.jsx는 너무 많은 일을 하고 있습니다.
// 각 부분이 어떤 역할을 하는지 파악하고 분리해보세요.

import { POSTS } from "@/data/posts";
import "@/App.css";

export default function App() {
  return (
    <div className="app">

      {/* ── 헤더 ─────────────────────────────────── */}
      <header className="header">
        <h1>📚 React 피드</h1>
        <span className="online-badge">● 3명 접속 중</span>
      </header>

      {/* ── 게시글 목록 ──────────────────────────── */}
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

      {/* ── 하단 통계 ─────────────────────────────── */}
      <footer className="stats-footer">
        게시글 {POSTS.length}개
      </footer>

    </div>
  );
}

```

```jsx
// components/PostCard.jsx
// TODO: avatar / author / content 중 필요한 것을 props로 받으세요.

export default function PostCard({ /* ??? */ }) {
  return (
    <li className="post-card">
      {/* TODO */}
    </li>
  );
}
```

- ✅ PostCard 정답 예시
    
    ```jsx
    // components/PostCard.jsx
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

<aside>
🤔

## **두 자식이 같은 데이터를 써야 한다면?**

지금까지 우리는 부모가 자식에게 데이터를 내려주는 법을 배웠습니다. 그런데 만약, **형제 관계인 두 컴포넌트가 똑같은 데이터를 공유해야 하는 상황**이 오면 어떻게 될까요?

예를 들어, 쇼핑몰에서 `<장바구니_버튼>` 컴포넌트와 화면 상단의 `<총_결제금액>` 컴포넌트가 있다고 해봅시다.

장바구니 버튼을 누르면 결제 금액도 같이 올라가야 합니다. 즉, 두 컴포넌트가 '장바구니에 담긴 아이템 수'라는 **같은 상태(State)를 공유**해야 합니다.

하지만 React는 데이터가 위에서 아래로만 흐릅니다. 형제끼리 데이터를 옆으로 던져줄 수는 없습니다. 이럴 때는 어떻게 해야 할까요?

이 문제를 해결하기 위한 React의 핵심 패턴, **'상태 끌어올리기(Lifting State Up)'**를 다음 파트에서 바로 알아보겠습니다.

</aside>

> ➡️ **다음 파트**: 여러 컴포넌트가 같은 상태를 공유해야 할 때 — Lifting State Up & useRef
>
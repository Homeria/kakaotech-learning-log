## 1️⃣ Next.js 기초와 렌더링 복습

<aside>
👾

### Framework vs Library

React는 **라이브러리**입니다. 라우팅, 데이터 페칭, 폴더 구조 등 모든 것을 개발자가 직접 결정합니다.

Next.js는 **프레임워크**입니다. "이 폴더에 이 파일을 두면 이렇게 동작한다"는 규칙이 이미 정해져 있습니다. 제어권이 프레임워크에 있고, 개발자는 그 규칙 안에서 코드를 채웁니다.

비유하자면 React는 식재료만 주는 것이고, Next.js는 레시피와 주방 도구까지 세팅해주는 것입니다.

</aside>

---

### App Router & File-based Routing

Next.js에서는 **폴더와 파일의 위치가 곧 URL 경로**가 됩니다.

https://nextjs-ko.org/docs/app/building-your-application/routing

| 파일 경로 | URL |
| --- | --- |
| `app/page.tsx` | `/` |
| `app/posts/page.tsx` | `/posts` |
| `app/posts/[postId]/page.tsx` | `/posts/1`, `/posts/2` |

!image.png

- `page.tsx` — 해당 경로에서 보여줄 화면
- `layout.tsx` — 공통 레이아웃으로, 하위 페이지들이 children prop으로 전달되어 내부에 렌더링됩니다.
- `loading.tsx` — 페이지 로딩 중에 보여줄 UI

---

## 2️⃣ 데이터 처리와 최적화 복습

### 서버 컴포넌트에서 직접 데이터 가져오기

Next.js의 Server Component는 서버에서 실행됩니다. 덕분에 컴포넌트 자체를 `async` 함수로 만들어 직접 데이터를 가져올 수 있습니다.

```tsx
// app/posts/page.tsx — Server Component
export default async function PostsPage() {
  const res = await fetch("http://localhost:8000/posts");
  const posts = await res.json();

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

이 컴포넌트는 브라우저가 아니라 서버에서 실행됩니다. 서버 컴포넌트는 모든 계산과 화면 조립을 서버에서 끝냅니다. 브라우저는 무거운 소스코드 대신 '다 만들어진 화면(HTML)'만 받기 때문에, 다운로드할 용량이 줄어들어 첫 화면이 뜨는 속도가 압도적으로 빨라집니다.

---

### Streaming과 loading.tsx

**SSR(Server Side Rendering)의 한계:** 서버에서 데이터를 모두 가져올 때까지 브라우저는 빈 화면을 보게 됩니다.

**Streaming으로 해결:** `loading.tsx`를 추가하면 Next.js는 HTML을 조각조각 나눠 전송합니다. 레이아웃과 로딩 UI를 먼저 보내고, 데이터 준비가 끝나면 나머지를 추가로 전송합니다.

> (참고) 스트리밍은 SSR의 단점을 해결하기 위해 페이지의 HTML을 더 작은 청크로 나누고 서버에서 클라이언트로 점진적으로 전송하는 방식입니다.
> 

https://nextjs-ko.org/docs/app/building-your-application/routing/loading-ui-and-streaming

```
loading.tsx가 있을 때:
1. 브라우저가 요청
2. 서버: 레이아웃 + 로딩 UI 즉시 전송 ← 사용자가 빈 화면 대신 스켈레톤을 봅니다
3. 서버: 데이터 페칭 진행 중...
4. 서버: 완성된 컴포넌트 추가 전송
5. 브라우저: 화면 업데이트
```

---

### Promise.all 로 여러 개의 Data Fetching

데이터를 순차적으로 가져오면 대기 시간이 합산됩니다.

```tsx
// ❌ 순차 실행: 100ms + 200ms = 300ms 대기
const user = await fetchUser();    // 100ms
const posts = await fetchPosts();  // 200ms

// ✅ 병렬 실행: max(100ms, 200ms) = 200ms 대기
const [user, posts] = await Promise.all([
  fetchUser(),
  fetchPosts(),
]);
```

두 요청이 서로 의존하지 않는다면, `Promise.all`로 동시에 보내는 것이 훨씬 효율적입니다.

---

## 3️⃣ Server Component vs Client Component, 그리고 Server Actions

### Server Component vs Client Component

Next.js의 모든 컴포넌트는 기본적으로 **Server Component**입니다. 서버에서 실행되며, 완성된 HTML 등이 클라이언트에 전달됩니다. 반면 **Client Component**는 파일 최상단에 `"use client"` 지시어를 선언하며, 서버에서 초기 렌더링 후 브라우저에서 실행됩니다.

| 구분 | Server Component | Client Component |
| --- | --- | --- |
| 선언 방법 | 기본값 (별도 선언 불필요) | 파일 상단에 `"use client"` |
| 실행 위치 | 서버 | 서버 (사전 렌더링) + 브라우저 (Hydration) |
| async/await | ✅ 컴포넌트 함수 자체에 사용 가능 | ❌ 컴포넌트 함수 자체에는 사용 불가
(내부 함수나 이벤트 핸들러에서는 사용 가능) |
| useState / useEffect | ❌ 사용 불가 | ✅ 사용 가능 |
| onClick 등 이벤트 핸들러 | ❌ 사용 불가 | ✅ 사용 가능 |
| DB / 환경변수 직접 접근 | ✅ 가능 (서버에서 실행) | ❌ 불가 (브라우저 노출 위험, `NEXT_PUBLIC_` 환경변수는 예외) |
| 클라이언트 번들 크기 | 영향 없음 | 번들에 포함됨 |

```tsx
// Server Component — 기본값, 컴포넌트 함수 자체에 async 사용 가능
// app/posts/page.tsx
export default async function PostsPage() {
  // 서버에서 직접 DB/API 호출 — 클라이언트에 URL 노출 없음
  const res = await fetch(`${process.env.FASTAPI_URL}/posts`);
  const posts = await res.json();
  return <ul>{posts.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

```tsx
// Client Component — "use client" 선언 필수
// app/search/page.tsx
"use client";
import { useState } from "react";

export default function SearchPage() {
  const [query, setQuery] = useState(""); // Server Component에서는 사용 불가
  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="검색어 입력..."
    />
  );
}
```

<aside>
📌

인터랙션(클릭, 입력 등 사용자 행동)이 필요하면 **Client Component**, 데이터 페칭·DB 접근만 필요하면 **Server Component**를 선택하시는 걸 권장드립니다. 하나의 페이지 안에서 두 종류를 자유롭게 섞어 쓸 수 있습니다.

</aside>

---

### Server Actions

**Server Actions**는 `"use server"` 지시어를 선언한 **서버 전용 함수**입니다. Client Component나 `<form>`에서 직접 호출할 수 있으며, 별도의 API Route를 만들지 않아도 됩니다. 함수 본문은 항상 서버에서 실행되므로 `FASTAPI_URL` 같은 환경변수를 안전하게 사용할 수 있습니다.

- 사용자의 입력(Form 제출)이나 상호작용으로 인해 서버의 데이터가 변경(CUD)되어야 할 때 사용

```tsx
// app/actions.ts
"use server";
import { revalidateTag } from "next/cache";
import { redirect } from "next/navigation";

export async function createPost(formData: FormData) {
  const title = formData.get("title") as string;
  const content = formData.get("content") as string;

  // 서버에서 직접 FastAPI 호출 — FASTAPI_URL이 클라이언트에 노출되지 않음
  await fetch(`${process.env.FASTAPI_URL}/posts`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ title, content }),
  });

  revalidateTag("posts-list"); // 게시글 목록 캐시 즉시 무효화
  redirect("/posts");          // 완료 후 목록 페이지로 이동
}
```

```tsx
// app/posts/new/page.tsx — <form action={서버액션}> 패턴
import { createPost } from "@/app/actions";

export default function NewPostPage() {
  return (
    // HTML <form>의 action에 서버 함수를 직접 연결
    // 제출 시 formData가 createPost로 자동 전달됨
    <form action={createPost}>
      <input name="title" placeholder="제목" />
      <textarea name="content" placeholder="내용" />
      <button type="submit">작성</button>
    </form>
  );
}
```

<aside>
💡

## SSR의 한계를 최소화하는 전략

**SSR의 기본 한계:** 서버가 HTML을 완성해 보내므로 초기 로딩은 빠르지만, 사용자 인터랙션(클릭, 입력 등)과 실시간 상태 변화를 서버 컴포넌트 단독으로는 처리하기는 어렵습니다.

**Server Component + Client Component 분리**로 해결합니다. 데이터 표시처럼 정적인 부분은 Server Component가 담당해 JavaScript 번들을 줄이고, 검색창·버튼처럼 인터랙션이 필요한 부분만 Client Component로 교체합니다. 필요한 곳에만 선택적으로 브라우저 코드를 씁니다.

**Server Actions**는 API Route Handler를 만들지 않고도 클라이언트에서 서버 로직을 실행할 수 있게 합니다. 폼 제출이나 데이터 변경 후 `revalidateTag()`로 캐시를 즉시 갱신하면, 사용자는 페이지 새로고침 없이 최신 데이터를 바로 볼 수 있습니다.

**결과:** 초기 로딩 속도라는 SSR의 장점을 유지하면서, 인터랙션 반응성이라는 CSR의 장점까지 확보합니다.

</aside>

---

<aside>
✏️

## 오늘 배울 것

이제 프론트엔드의 데이터 요청을 수신하고 처리하는 백엔드 저장 로직을 배워볼 건데요.

이전 주차에는 `sqlite3`로 직접 SQL을 작성했습니다. 이번에는 **SQLAlchemy**를 사용해 Python 코드로 데이터베이스를 다루는 방법을 배웁니다.

마지막에는 프론트엔드와 백엔드를 함께 실행해서, 실제로 데이터가 오가는 전체 흐름을 직접 확인할 예정입니다.

</aside>
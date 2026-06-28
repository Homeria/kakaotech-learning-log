# Day 2. Next.js 데이터 처리와 최적화 정리

> 원본 PDF: `Day2 - Next.js 데이터 처리와 최적화.pdf`  
> 주제: Data Fetching, Streaming, Server Actions, Caching, Optimization, Image, Metadata

---

# 1. Data Fetching

## 1.1 Next.js의 Data Fetching

Next.js는 렌더링 방식과 함께 데이터 처리 방식도 중요하다.  
App Router에서는 기본적으로 Server Component를 사용하므로, 서버에서 직접 데이터를 가져와 컴포넌트를 구성할 수 있다.

```text
Routing과 Rendering
→ Data Fetching
→ Caching
→ Optimizing
```

---

## 1.2 GET 요청 기본 패턴

Server Component에서는 `fetch`를 그대로 사용할 수 있다.  
서버에서 데이터를 가져오고, 그 결과를 기반으로 컴포넌트를 렌더링한다.

```tsx
interface Post {
  id: number;
}

async function getPosts() {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  const fetchedPosts: Post[] = await response.json();
  return fetchedPosts;
}

export default async function PostsPage() {
  const fetchedPosts = await getPosts();

  return (
    <>
      <h2>게시글 목록</h2>
      <ul>
        {fetchedPosts.map((post) => (
          <li key={post.id}>{JSON.stringify(post)}</li>
        ))}
      </ul>
    </>
  );
}
```

---

## 1.3 Server Component에서 fetch할 때의 특징

서버에서 `fetch`가 실행되므로, 브라우저 개발자 도구의 Network 탭에서 해당 외부 API 요청이 보이지 않을 수 있다.

```text
브라우저 → Next.js 서버에 페이지 요청
Next.js 서버 → 외부 API fetch
Next.js 서버 → 렌더링된 HTML 응답
```

| 특징 | 설명 |
|---|---|
| 실행 위치 | 서버 |
| Network 탭 | 외부 API fetch가 브라우저 Network에 직접 보이지 않을 수 있음 |
| 장점 | API 호출 로직을 클라이언트에 노출하지 않을 수 있음 |
| 단점 | 서버 작업이 오래 걸리면 페이지 응답도 늦어질 수 있음 |

---

# 2. SSR의 단점과 `loading.tsx`

## 2.1 SSR 방식의 단점

Server Component에서 데이터 요청이 오래 걸리면 서버 렌더링이 끝날 때까지 페이지가 보이지 않을 수 있다.

```tsx
async function getPosts() {
  await new Promise((resolve) => setTimeout(resolve, 3000));

  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  const fetchedPosts: Post[] = await response.json();
  return fetchedPosts;
}
```

위처럼 3초 지연이 있다면, 서버 작업이 끝나기 전까지 사용자는 완성된 화면을 보지 못한다.

---

## 2.2 `loading.tsx`

이 문제를 보완하기 위해 Next.js는 약속된 파일인 `loading.tsx`를 제공한다.

```tsx
// app/loading.tsx
export default function Loading() {
  return <h2>Loading...</h2>;
}
```

`loading.tsx`를 사용하면 아직 서버 작업이 끝나지 않은 컴포넌트가 있을 때 로딩 UI를 먼저 보여줄 수 있다.

---

## 2.3 `loading.tsx`의 역할

| 파일 | 역할 |
|---|---|
| `loading.tsx` | 해당 route segment가 로딩 중일 때 보여줄 UI |
| `page.tsx` | 로딩 완료 후 실제 화면 |
| `layout.tsx` | 로딩 상태와 관계없이 공유되는 UI |

---

## 2.4 `loading.tsx`를 사용했을 때의 흐름

```text
1. 렌더링 가능한 Layout과 UI를 먼저 보여준다.
2. 서버 작업이 진행 중인 영역에는 loading.tsx UI를 보여준다.
3. 서버 작업이 끝나면 실제 컴포넌트로 대체된다.
```

이 동작은 Next.js의 **Streaming** 기능과 연결된다.

---

# 3. Streaming

## 3.1 Streaming이란?

Streaming은 HTML 또는 데이터를 작은 조각, 즉 **chunk**로 나누어 준비가 완료된 조각부터 클라이언트에 점진적으로 전송하는 기술이다.

```text
전체 HTML을 한 번에 기다림 X
준비된 HTML 조각부터 먼저 전송 O
```

---

## 3.2 Streaming의 기반

Next.js의 Streaming은 HTTP의 **Chunked Transfer Encoding** 기술을 활용한다.

서버가 모든 렌더링을 끝낸 후 한 번에 HTML을 보내는 것이 아니라, 준비된 부분부터 나누어 보낼 수 있다.

---

## 3.3 React와 Streaming이 잘 맞는 이유

React 앱은 컴포넌트 단위로 구성된다.  
따라서 각각의 컴포넌트를 하나의 chunk처럼 생각할 수 있다.

```text
Root Layout
→ Header
→ PostsList
→ UsersList
```

각 컴포넌트의 준비가 완료되는 시점이 다르다면, 먼저 준비된 컴포넌트부터 클라이언트로 보낼 수 있다.

---

## 3.4 `loading.tsx`와 Streaming

`loading.tsx` 파일을 추가하면 Next.js는 자동으로 Streaming 기능을 적용한다.

```text
loading.tsx 생성
→ 로딩 UI 표시
→ 서버 컴포넌트 렌더링 진행
→ 완료된 chunk부터 전송
```

---

# 4. 여러 개의 Data Fetching

## 4.1 순차적 Fetching의 문제

한 페이지에서 여러 데이터 요청을 할 수 있다.

```tsx
export default async function PostsPage() {
  const fetchedPosts = await getPosts();
  const fetchedUsers = await getUsers();

  return (
    <>
      <h2>게시글 목록</h2>
      {/* posts와 users 렌더링 */}
    </>
  );
}
```

만약 `getPosts()`가 3초, `getUsers()`가 5초 걸린다면 총 8초 정도가 걸린다.

```text
getPosts 3초
→ getUsers 5초
= 총 8초
```

---

## 4.2 순차 실행이 발생하는 이유

`await`를 순서대로 작성하면 첫 번째 비동기 작업이 끝난 뒤 두 번째 작업이 시작된다.

```tsx
const fetchedPosts = await getPosts(); // 3초 대기
const fetchedUsers = await getUsers(); // 그 다음 5초 대기
```

---

## 4.3 `Promise.all`로 병렬 Fetching

서로 의존하지 않는 비동기 작업은 `Promise.all`로 병렬 실행할 수 있다.

```tsx
export default async function PostsPage() {
  const [fetchedPosts, fetchedUsers] = await Promise.all([
    getPosts(),
    getUsers(),
  ]);

  return (
    <>
      <h2>게시글 목록</h2>
      {/* posts와 users 렌더링 */}
    </>
  );
}
```

이 경우 전체 소요 시간은 가장 오래 걸리는 작업의 시간에 가까워진다.

```text
getPosts 3초
getUsers 5초
→ 병렬 실행
= 약 5초
```

---

## 4.4 `Promise.all`의 한계

`Promise.all`은 전체 작업이 모두 끝나야 결과를 반환한다.  
따라서 먼저 완료된 데이터가 있어도 전체 fetch가 끝나기 전까지 화면에 렌더링되지 않는다.

예를 들어 Posts는 3초 만에 준비되었지만 Users가 5초 걸리면, Posts도 5초 후에야 함께 렌더링된다.

---

# 5. Parallel Fetching과 Suspense

## 5.1 가장 최적의 방법

Next.js가 제시하는 더 나은 방법은 React의 `Suspense`를 활용하는 것이다.

핵심은 각 fetch 작업을 독립된 Server Component로 분리하고, 각각을 `Suspense`로 감싸는 것이다.

---

## 5.2 컴포넌트 분리

```tsx
// app/posts/PostsList.tsx
interface Post {
  id: number;
}

export default async function PostsList() {
  await new Promise((resolve) => setTimeout(resolve, 3000));

  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  const fetchedPosts: Post[] = await response.json();

  return (
    <ul>
      {fetchedPosts.map((post) => (
        <li key={post.id}>{JSON.stringify(post)}</li>
      ))}
    </ul>
  );
}
```

```tsx
// app/posts/UsersList.tsx
interface User {
  id: number;
}

export default async function UsersList() {
  await new Promise((resolve) => setTimeout(resolve, 5000));

  const response = await fetch("https://jsonplaceholder.typicode.com/users");
  const fetchedUsers: User[] = await response.json();

  return (
    <ul>
      {fetchedUsers.map((user) => (
        <li key={user.id}>{JSON.stringify(user)}</li>
      ))}
    </ul>
  );
}
```

---

## 5.3 Suspense 적용

```tsx
import { Suspense } from "react";
import PostsList from "./PostsList";
import UsersList from "./UsersList";

export default function PostsPage() {
  return (
    <>
      <h2>게시글 목록</h2>
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr" }}>
        <Suspense fallback={<h2>Loading Posts...</h2>}>
          <PostsList />
        </Suspense>

        <Suspense fallback={<h2>Loading Users...</h2>}>
          <UsersList />
        </Suspense>
      </div>
    </>
  );
}
```

---

## 5.4 Suspense의 장점

각 컴포넌트가 준비되는 대로 따로 렌더링된다.

```text
0초: Loading Posts..., Loading Users...
3초: PostsList 렌더링, Loading Users...
5초: UsersList 렌더링
```

| 방식 | 특징 |
|---|---|
| 순차 await | 작업 시간이 모두 더해짐 |
| Promise.all | 병렬 실행되지만 모두 끝나야 렌더링 |
| Suspense | 각 작업이 완료되는 대로 점진적 렌더링 |

---

# 6. POST, PUT, DELETE Data Fetching

## 6.1 상태 변경 요청의 문제

GET 요청은 서버에서 데이터를 읽어오는 작업이다.  
반면 POST, PUT, DELETE는 서버나 DB의 상태를 변경하는 작업이다.

| Method | 의미 |
|---|---|
| GET | 데이터 조회 |
| POST | 데이터 생성 |
| PUT/PATCH | 데이터 수정 |
| DELETE | 데이터 삭제 |

---

## 6.2 Client Component에서 처리하는 방식

가장 단순한 방식은 `"use client"`와 함께 React에서 하던 방식으로 처리하는 것이다.

```tsx
"use client";

import { ChangeEventHandler, FormEventHandler, useState } from "react";

export default function CreatePostPage() {
  const [inputs, setInputs] = useState({
    title: "",
    body: "",
    userId: 0,
  });

  const handleSubmit: FormEventHandler<HTMLFormElement> = (e) => {
    e.preventDefault();

    fetch("https://jsonplaceholder.typicode.com/posts", {
      method: "POST",
      body: JSON.stringify(inputs),
      headers: {
        "Content-type": "application/json; charset=UTF-8",
      },
    })
      .then((response) => response.json())
      .then((json) => alert(JSON.stringify(json)));
  };

  const handleChange: ChangeEventHandler<HTMLInputElement | HTMLTextAreaElement> =
    (e) => {
      setInputs({ ...inputs, [e.target.name]: e.target.value });
    };

  return (
    <form onSubmit={handleSubmit}>
      <input name="title" value={inputs.title} onChange={handleChange} />
      <textarea name="body" value={inputs.body} onChange={handleChange} />
      <input name="userId" type="number" value={inputs.userId} onChange={handleChange} />
      <button>생성</button>
    </form>
  );
}
```

---

## 6.3 Client Component 방식의 한계

이 방식은 순수 React 방식과 비슷하지만, Next.js 서버 기능의 장점을 충분히 활용하지 못한다.

| 한계 | 설명 |
|---|---|
| Client Component 필요 | `"use client"`를 사용해야 함 |
| Hydration 필요 | 이벤트 리스너와 state 연결을 위해 hydration 발생 |
| 비즈니스 로직 노출 | 일부 로직이 클라이언트 코드에 포함될 수 있음 |
| 서버 기능 활용 부족 | DB 작업, 인증/인가, redirect 등을 서버에서 자연스럽게 처리하기 어려움 |

---

## 6.4 Next.js 개발 방향성

Next.js에서는 가능한 한 서버 측에서 로직을 처리하는 것이 좋다.

```text
클라이언트
→ 이벤트 처리 최소화

서버
→ 비즈니스 로직
→ DB CRUD
→ 인증/인가
→ Redirect
```

이를 위해 Next.js는 **Server Actions**를 제공한다.

---

# 7. Server Actions

## 7.1 Server Action이란?

Server Action은 무조건 서버에서 실행되는 비동기 함수이다.  
클라이언트 번들에 포함되지 않으며, 서버/DB 상태가 변경되는 작업을 처리하는 데 사용한다.

```tsx
"use server";
```

---

## 7.2 Server Component에서 인라인 선언

Server Component에서는 Server Action을 인라인으로 선언할 수 있다.

```tsx
export default function Page() {
  const mutateSomething = async () => {
    "use server";
    // Mutating Something...
  };

  return <form action={mutateSomething}>...</form>;
}
```

---

## 7.3 Client Component에서는 인라인 선언 불가

`"use client"`가 사용된 파일 내부에서는 인라인 Server Action을 작성할 수 없다.

```tsx
"use client";

export default function Page() {
  const mutateSomething = async () => {
    "use server"; // 불가
  };
}
```

Client Component에서 Server Action을 사용하려면 별도 파일로 분리해야 한다.

---

## 7.4 별도 actions 파일 생성

파일 상단에 `"use server"`를 작성하면, 해당 파일 내부의 모든 함수는 Server Action이 된다.

```tsx
// app/actions.ts
"use server";

export const mutateSomething = async () => {
  // Mutating Something...
};
```

---

## 7.5 Client Component에서 import하여 사용

```tsx
"use client";

import { createPost } from "@/app/actions";
import { FormEventHandler, useState } from "react";

export default function CreatePostPage() {
  const [inputs, setInputs] = useState({
    title: "",
    body: "",
    userId: 0,
  });

  const handleSubmit: FormEventHandler<HTMLFormElement> = (e) => {
    e.preventDefault();
    createPost(inputs).then((json) => alert(JSON.stringify(json)));
  };

  return <form onSubmit={handleSubmit}>...</form>;
}
```

```tsx
// app/actions.ts
"use server";

export const createPost = async (inputs: {
  title: string;
  body: string;
  userId: number;
}) => {
  console.log("Server Action is executed in server");

  return fetch("https://jsonplaceholder.typicode.com/posts", {
    method: "POST",
    body: JSON.stringify(inputs),
    headers: {
      "Content-type": "application/json; charset=UTF-8",
    },
  }).then((response) => response.json());
};
```

---

## 7.6 Server Action의 장점

| 장점 | 설명 |
|---|---|
| 서버에서 실행 | 클라이언트 번들에 포함되지 않음 |
| 비즈니스 로직 캡슐화 | DB CRUD, 인증/인가 등을 서버에서 처리 가능 |
| endpoint 은닉 | 실제 fetch endpoint를 클라이언트에서 감출 수 있음 |
| 풀스택 개발 가능 | 별도 백엔드 없이 Next.js 내부에서 처리 가능 |
| redirect 연계 | 작업 완료 후 서버에서 페이지 이동 처리 가능 |

---

# 8. Server Actions와 Form

## 8.1 이벤트 리스너 없이 Server Action 사용하기

Client Component에서 `onSubmit` 이벤트와 state를 사용하면 hydration과 이벤트 처리 오버헤드가 생긴다.

이를 줄이기 위해 HTML `form` 자체의 제출 기능과 Server Action을 연결할 수 있다.

---

## 8.2 Form 기본 구조

```tsx
export default function CreatePostPage() {
  return (
    <form style={{ display: "flex", flexDirection: "column", width: "50%" }}>
      <input type="text" name="title" placeholder="제목" />
      <textarea name="body" placeholder="본문"></textarea>
      <input type="number" name="userId" placeholder="유저 번호" />
      <button>생성</button>
    </form>
  );
}
```

중요한 점은 각 input의 `name` 속성을 정확히 작성해야 한다는 것이다.  
`name`은 나중에 `FormData`에서 값을 꺼낼 때 사용된다.

---

## 8.3 form action에 Server Action 전달

React에서는 `form`의 `action` 속성에 endpoint 문자열이 아니라 함수를 전달할 수 있다.

```tsx
import { createPost } from "@/app/actions";

export default function CreatePostPage() {
  return (
    <form action={createPost}>
      <input type="text" name="title" placeholder="제목" />
      <textarea name="body" placeholder="본문"></textarea>
      <input type="number" name="userId" placeholder="유저 번호" />
      <button>생성</button>
    </form>
  );
}
```

---

## 8.4 FormData로 제출 데이터 받기

Server Action에는 제출된 데이터가 담긴 `FormData` 객체가 전달된다.

```tsx
"use server";

import { redirect } from "next/navigation";

export const createPost = async (formData: FormData) => {
  const inputs = {
    title: formData.get("title"),
    body: formData.get("body"),
    userId: formData.get("userId"),
  };

  await fetch("https://jsonplaceholder.typicode.com/posts", {
    method: "POST",
    body: JSON.stringify(inputs),
    headers: {
      "Content-type": "application/json; charset=UTF-8",
    },
  });

  redirect("/posts");
};
```

---

## 8.5 redirect

Server Action 작업 이후 `redirect` 함수를 사용해 특정 경로로 이동할 수 있다.

```tsx
import { redirect } from "next/navigation";

redirect("/posts");
```

---

## 8.6 bind로 추가 매개변수 전달

Form에 포함되지 않은 데이터가 필요하면 `bind`를 사용해 추가 인수를 전달할 수 있다.

```tsx
// app/actions.ts
"use server";

export const createPost = async (userId: number, formData: FormData) => {
  const inputs = {
    title: formData.get("title"),
    body: formData.get("body"),
    userId,
  };

  // 서버 작업
};
```

```tsx
// app/posts/create/page.tsx
import { createPost } from "@/app/actions";

export default function CreatePostPage({
  searchParams: { userId = 1 },
}: {
  searchParams: { userId?: number };
}) {
  const createPostWithUserId = createPost.bind(null, userId);

  return (
    <form action={createPostWithUserId}>
      <input type="text" name="title" placeholder="제목" />
      <textarea name="body" placeholder="본문"></textarea>
      <button>생성</button>
    </form>
  );
}
```

---

# 9. `useFormStatus`

## 9.1 `useFormStatus`란?

`useFormStatus`는 Server Action이 연결된 form의 제출 상태를 가져올 수 있는 Hook이다.

```tsx
import { useFormStatus } from "react-dom";
```

---

## 9.2 사용 조건

`useFormStatus`는 Hook이므로 Client Component에서 사용해야 한다.  
또한 form 태그 하위의 컴포넌트에서만 해당 form의 상태를 가져올 수 있다.

따라서 보통 제출 버튼을 별도 컴포넌트로 분리한다.

---

## 9.3 SubmitButton 예시

```tsx
"use client";

import { useFormStatus } from "react-dom";

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button disabled={pending}>
      {pending ? "생성 중..." : "생성"}
    </button>
  );
}
```

---

# 10. 렌더링과 Data Fetching 총정리

## 10.1 기본 SSR 렌더링 과정

```text
1. 클라이언트가 특정 경로에 GET 요청을 보낸다.
2. 서버에서 Root Layout을 렌더링한다.
3. 하위 Layout을 렌더링한다.
4. endpoint 경로에 맞는 Page 컴포넌트를 렌더링한다.
5. 렌더링이 완료되면 클라이언트에 HTML을 응답한다.
6. Client Component가 있다면 Hydration을 진행한다.
```

---

## 10.2 Streaming 과정

```text
1. 먼저 렌더링 가능한 컴포넌트부터 렌더링한다.
2. 서버 작업이 필요한 영역에는 loading.tsx 또는 Suspense fallback을 렌더링한다.
3. 준비된 HTML chunk를 클라이언트에 먼저 전달한다.
4. 서버 작업이 완료되는 대로 해당 컴포넌트를 렌더링해 추가 전달한다.
```

---

## 10.3 loading.tsx와 Suspense 비교

| 구분 | loading.tsx | Suspense |
|---|---|---|
| 적용 단위 | route segment 단위 | 컴포넌트 단위 |
| 사용 방식 | 약속된 파일 생성 | `<Suspense fallback={...}>` |
| 목적 | 페이지/구간 로딩 UI | 특정 컴포넌트별 로딩 UI |
| 장점 | 간단하게 자동 적용 | 세밀한 병렬 렌더링 제어 가능 |

---

# 11. Caching

## 11.1 Next.js와 Caching

Next.js는 렌더링 성능을 개선하기 위해 여러 캐싱 메커니즘을 제공한다.

특정 경로의 페이지에 처음 접속한 이후 다시 접속하면 로딩 속도가 빨라지는 것은 캐싱 덕분이다.

---

## 11.2 Next.js의 4가지 캐싱 메커니즘

| 캐시 | 위치 | 저장 대상 |
|---|---|---|
| Request Memoization | 서버 렌더링 과정 | 같은 렌더링 중 중복 fetch 결과 |
| Data Cache | 서버 | fetch 응답 데이터 |
| Full Route Cache | 서버 | 렌더링된 route 결과 |
| Router Cache | 클라이언트 | Hydration 완료된 렌더링 결과 |

---

# 12. Request Memoization

## 12.1 Request Memoization이란?

Request Memoization은 `fetch` API로 보낸 요청의 응답 데이터를 렌더링 중에 기억하는 캐싱 메커니즘이다.

GET 요청에 대해서만 적용되며, **페이지 요청부터 렌더링 종료까지의 시간 동안만** 지속된다.

---

## 12.2 동작 방식

하나의 route가 렌더링되는 과정에서 같은 endpoint로 fetch가 두 번 이상 발생하면 실제 fetch는 한 번만 실행된다.

```tsx
// app/posts/page.tsx
export default async function PostsPage() {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  const fetchedPosts = await response.json();

  return (
    <>
      <h2>게시글 목록 {fetchedPosts.length}개</h2>
      <PostsList />
    </>
  );
}
```

```tsx
// app/posts/PostsList.tsx
export default async function PostsList() {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  const fetchedPosts = await response.json();

  return (
    <ul>
      {fetchedPosts.map((post) => (
        <li key={post.id}>{JSON.stringify(post)}</li>
      ))}
    </ul>
  );
}
```

위처럼 같은 endpoint를 두 번 fetch해도 렌더링 중에는 한 번만 실제 요청이 발생하고 나머지는 캐시된 결과를 사용한다.

---

## 12.3 Request Memoization의 장점

| 장점 | 설명 |
|---|---|
| 중복 fetch 제거 | 같은 렌더링 과정에서 동일 요청을 한 번만 수행 |
| 성능 저하 방지 | fetch 코드가 중복되어도 큰 부담이 줄어듦 |
| Props Drilling 완화 | 상위에서 받은 데이터를 계속 props로 넘기지 않고 필요한 컴포넌트에서 직접 fetch 가능 |
| 전역 상태처럼 활용 | 같은 렌더링 중에는 동일 fetch 결과를 공유하는 효과 |

---

# 13. Data Cache

## 13.1 Data Cache란?

Data Cache도 `fetch` API 요청에 대한 응답 데이터를 기억하는 캐싱 메커니즘이다.

Request Memoization과 달리, 캐시를 삭제하거나 업데이트하기 전까지 모든 요청에 대해 지속될 수 있다.

---

## 13.2 Request Memoization과 Data Cache 차이

| 구분 | Request Memoization | Data Cache |
|---|---|---|
| 지속 범위 | 하나의 렌더링 과정 동안 | 캐시 삭제/업데이트 전까지 지속 |
| 적용 대상 | 같은 렌더링 중 동일 GET fetch | fetch 응답 데이터 |
| 위치 | 서버 렌더링 과정 | 서버 캐시 |
| 목적 | 중복 요청 제거 | 요청 간 데이터 재사용 |

---

## 13.3 Data Cache 동작 흐름

```text
1. 최초 fetch는 실제 request가 진행된다.
2. 성공하면 결과가 Data Cache에 저장된다.
3. 이후 같은 fetch가 수행되면 캐시된 데이터를 사용한다.
4. 다른 클라이언트가 접속해도 저장된 캐시를 사용할 수 있다.
```

---

# 14. Data Cache Revalidation

## 14.1 Revalidation이란?

Revalidation은 캐싱된 데이터를 다시 fetch하여 최신 버전으로 업데이트하는 것을 의미한다.

Data Cache는 오래된 데이터를 계속 보여줄 수 있으므로, 적절한 시점에 Revalidation을 수행해야 한다.

---

## 14.2 Revalidation 방식

| 방식 | 설명 |
|---|---|
| Time-based Revalidation | 특정 시간마다 자동으로 revalidate |
| On-demand Revalidation | 특정 이벤트 발생 시 수동으로 revalidate |

---

## 14.3 Time-based Revalidation

fetch 실행 시 `next.revalidate` 옵션을 설정한다.

```tsx
const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
  next: { revalidate: 3600 },
});
```

위 코드는 한 시간마다 revalidate되도록 설정한 것이다.

---

## 14.4 Time-based Revalidation 주의점

설정한 시간이 지난 후 최초 fetch가 발생하면, 먼저 기존 캐시 데이터를 반환한다.  
그 다음 백그라운드에서 revalidate가 진행되고, 이후 요청부터 업데이트된 캐시 데이터가 사용된다.

```text
1시간 경과
→ 첫 요청: 기존 캐시 데이터 사용
→ 백그라운드에서 캐시 업데이트
→ 다음 요청: 업데이트된 데이터 사용
```

---

## 14.5 On-demand Revalidation

On-demand Revalidation은 특정 이벤트가 발생했을 때 revalidate를 실행하는 방식이다.

주로 POST, PUT, DELETE 같은 데이터 변경 작업 직후에 수행한다.

Server Action에서 사용할 수 있다.

---

## 14.6 `revalidatePath`

`revalidatePath`는 특정 경로를 기준으로 revalidate를 수행한다.

```tsx
"use server";

import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

export const createPost = async (formData: FormData) => {
  // Post 생성
  revalidatePath("/posts");
  redirect("/posts");
};
```

---

## 14.7 `revalidatePath` 예시

```tsx
revalidatePath("/posts"); // /posts 경로 전체 revalidate
revalidatePath("/posts/[postId]", "page"); // 해당 page revalidate
revalidatePath("/posts/[postId]", "layout"); // 해당 layout revalidate
revalidatePath("/"); // 전체 revalidate
```

---

## 14.8 `revalidateTag`

`revalidateTag`는 tag 기반으로 캐시를 revalidate한다.  
먼저 fetch 함수 실행 시 `tags` 옵션을 설정해야 한다.

```tsx
export default async function PostsList() {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
    next: { tags: ["posts-list"] },
  });

  const fetchedPosts = await response.json();

  return (
    <ul>
      {fetchedPosts.map((post) => (
        <li key={post.id}>{JSON.stringify(post)}</li>
      ))}
    </ul>
  );
}
```

이후 Server Action에서 같은 tag를 인수로 전달한다.

```tsx
"use server";

import { revalidateTag } from "next/cache";
import { redirect } from "next/navigation";

export const createPost = async (formData: FormData) => {
  // Post 생성
  revalidateTag("posts-list");
  redirect("/posts");
};
```

---

## 14.9 Next.js 16 변경점 — `revalidateTag`

Next.js 16부터는 `revalidateTag`의 두 번째 인수로 cacheLife 프로필을 전달해야 한다.

```tsx
revalidateTag("posts-list", "max");
```

| 프로필 | 설명 |
|---|---|
| `"max"` | 대부분의 경우 적절한 기본 선택 |
| `"hours"` | 시간 단위 캐시 생명주기 |
| `"days"` | 일 단위 캐시 생명주기 |
| `{ expire: 3600 }` | 직접 지정하는 커스텀 프로필 |

---

## 14.10 Next.js 16 변경점 — `updateTag`

Next.js 16부터 `updateTag`가 추가되었다.  
Server Action 전용 API로, 동일한 요청 내에서 데이터를 즉시 만료시키고 새 데이터를 읽어올 수 있다.

```tsx
"use server";

import { updateTag } from "next/cache";
import { redirect } from "next/navigation";

export const createPost = async (formData: FormData) => {
  // Post 생성
  updateTag("posts-list");
  redirect("/posts");
};
```

---

## 14.11 Next.js 16 변경점 — `refresh`

Next.js 16부터 `refresh`도 추가되었다.  
캐시 저장소는 건드리지 않고, 상태 표시나 알림 같은 실시간 정보만 선별적으로 업데이트할 때 사용한다.

```tsx
"use server";

import { refresh } from "next/cache";

export const createPost = async (formData: FormData) => {
  // Post 생성
  refresh();
};
```

---

# 15. Full Route Cache와 Router Cache

## 15.1 두 캐시의 공통점

Full Route Cache와 Router Cache는 둘 다 **렌더링된 결과**를 저장하는 캐시이다.

| 캐시 | 위치 | 저장 대상 |
|---|---|---|
| Full Route Cache | 서버 | 서버에서 수행된 렌더링 결과 |
| Router Cache | 클라이언트 | Hydration이 완료된 렌더링 결과 |

---

## 15.2 Full Route Cache

Full Route Cache는 서버 단의 캐시이다.  
특정 경로로 최초 요청이 들어오면 SSR을 수행하고, 그 렌더링 결과를 Full Route Cache에 저장한다.

```text
최초 GET /posts
→ 서버에서 SSR 수행
→ 렌더링 결과 저장
→ 이후 요청에서 재사용 가능
```

---

## 15.3 Router Cache

Router Cache는 클라이언트 단의 캐시이다.  
서버로부터 받은 정적 렌더링 결과에 Hydration을 통해 JS 요소가 추가된 뒤, 그 결과가 Router Cache에 저장된다.

```text
서버 HTML 응답
→ Hydration
→ 클라이언트에서 렌더링 결과 저장
→ 이후 같은 페이지 이동 시 빠르게 사용
```

---

## 15.4 캐시 상호작용

Next.js의 빠른 사용자 경험은 여러 캐시가 함께 작동하기 때문에 가능하다.

```text
Request Memoization
→ 렌더링 중 중복 fetch 제거

Data Cache
→ fetch 결과 저장

Full Route Cache
→ 서버 렌더링 결과 저장

Router Cache
→ 클라이언트 렌더링 결과 저장
```

---

# 16. Caching 끄기와 버전 변경점

## 16.1 fetch 단위로 캐싱 끄기

각 fetch에 대해 캐시를 끄려면 `cache` 옵션을 설정한다.

```tsx
fetch("https://...", {
  cache: "no-store",
});
```

---

## 16.2 라우트 단위로 캐싱 끄기

특정 route에 대한 캐싱 기능을 전부 끄고 싶다면 `layout` 또는 `page`에 다음 코드를 추가한다.

```tsx
export const dynamic = "force-dynamic";
export const revalidate = 0;
```

---

## 16.3 Next.js 15 변경점 — fetch 기본 캐싱 OFF

Next.js 14에서는 `fetch()`를 호출하면 기본적으로 Data Cache에 저장되었다.  
하지만 Next.js 15부터는 기본값이 변경되어, 별도 설정이 없으면 캐시하지 않는다.

| 항목 | Next.js 14 | Next.js 15+ |
|---|---|---|
| 기본 `fetch()` | 자동 캐시됨 | 캐시 안 됨 |
| 캐시 끄기 | `{ cache: "no-store" }` | 기본값이므로 불필요 |
| 캐시 켜기 | 기본값이므로 불필요 | `{ cache: "force-cache" }` 명시 |
| 시간 기반 캐시 | `{ next: { revalidate: 60 } }` | 동일 |
| 태그 기반 캐시 | `{ next: { tags: ["..."] } }` | 동일 |
| Request Memoization | 동일하게 동작 | 동일하게 동작 |

---

## 16.4 Next.js 15+에서 캐시를 명시하는 방법

```tsx
// 영구 캐시
const res = await fetch("https://...", {
  cache: "force-cache",
});

// 시간 기반 캐시
const res = await fetch("https://...", {
  next: { revalidate: 3600 },
});

// 태그 기반 캐시
const res = await fetch("https://...", {
  next: { tags: ["posts-list"] },
});
```

---

# 17. Optimization

## 17.1 Next.js 최적화 기능

Next.js의 최적화 기능은 Core Web Vitals 개선에 초점이 맞추어져 있다.  
Core Web Vitals는 사용자에게 제공되는 웹 환경의 품질을 측정하는 주요 지표이다.

---

## 17.2 Core Web Vitals가 중요한 이유

Google은 Core Web Vitals를 검색 엔진 순위 결정에 반영한다고 발표했다.  
즉, Core Web Vitals는 SEO 개선을 위해 최적화해야 할 중요한 요소이다.

---

## 17.3 Core Web Vitals 3가지 지표

| 지표 | 의미 | 핵심 |
|---|---|---|
| LCP | Largest Contentful Paint | 주요 콘텐츠가 얼마나 빨리 로드되는지 |
| INP | Interaction to Next Paint | 사용자 상호작용에 얼마나 빠르게 응답하는지 |
| CLS | Cumulative Layout Shift | 예기치 않은 레이아웃 이동이 없는지 |

---

## 17.4 LCP

LCP는 웹 페이지의 주요 콘텐츠가 얼마나 빨리 로드되는지를 측정한다.  
실제 측정 시에는 가장 큰 이미지, 텍스트 블록, 동영상 등의 렌더링 시간을 확인한다.

---

## 17.5 INP

INP는 사용자의 상호작용에 대해 페이지가 얼마나 빠르게 응답하는지를 측정한다.  
오래 걸리는 상호작용이 있다면 사용자가 무슨 일이 일어나고 있는지 알 수 있도록 시각적 피드백을 빠르게 제공해야 한다.

---

## 17.6 CLS

CLS는 콘텐츠가 예기치 못하게 이동하는 현상이 없는지를 측정한다.  
이미지가 늦게 로드되면서 아래 콘텐츠가 밀리는 현상은 CLS에 나쁜 영향을 준다.

---

# 18. Image 컴포넌트

## 18.1 `next/image`

Next.js의 `Image` 컴포넌트는 HTML `img` 태그를 확장한 컴포넌트이다.  
이미지를 자동으로 최적화해준다.

```tsx
import Image from "next/image";
import eliceGame from "@/app/assets/elice-game.png";

<Image src={eliceGame} alt="elice-game" />;
```

---

## 18.2 Image 컴포넌트의 핵심 목적

Image 컴포넌트의 핵심은 Core Web Vitals 중 특히 **CLS와 LCP를 개선**하는 것이다.

| 개선 지표 | 설명 |
|---|---|
| CLS | 이미지 크기 공간을 미리 확보해 레이아웃 밀림 방지 |
| LCP | 이미지 최적화와 우선 로딩으로 주요 콘텐츠 로딩 개선 |

---

## 18.3 일반 img 태그의 문제

일반적인 `<img>` 태그는 이미지가 로드되기 전까지 정확한 공간을 확보하지 못할 수 있다.  
그 결과 이미지 로드 후 다른 요소들의 위치가 밀려 CLS에 나쁜 영향을 준다.

---

## 18.4 Image 컴포넌트의 CLS 개선

`Image` 컴포넌트는 이미지 크기를 미리 계산하여 해당 공간을 확보한다.  
따라서 이미지가 나중에 로드되어도 주변 레이아웃이 흔들리지 않는다.

---

## 18.5 placeholder

이미지가 로딩되는 동안 대신 표시할 placeholder를 지정할 수 있다.

```tsx
<Image
  src={eliceGame}
  alt="elice-game"
  placeholder="blur"
/>
```

`placeholder="blur"`를 설정하면 로딩 중 블러 처리된 이미지를 보여준다.

---

## 18.6 외부 이미지 사용 설정

외부 URL 이미지를 사용하려면 `next.config.mjs`에서 허용할 원격 이미지 패턴을 설정해야 한다.

```tsx
// next.config.mjs
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "elice.io",
        port: "",
      },
    ],
  },
};

export default nextConfig;
```

---

## 18.7 외부 이미지는 width와 height 필수

외부 URL 이미지는 Next.js가 이미지 크기를 미리 알 수 없다.  
따라서 `width`와 `height`를 필수적으로 설정해야 한다.

```tsx
<Image
  src="https://elice.io/_next/static/media/elice-brand.131cc564.png"
  alt="elice-logo"
  width={297}
  height={93}
/>
```

---

## 18.8 quality 속성

Next.js는 이미지 퀄리티를 자동으로 낮춰 로딩 속도를 개선한다.  
`quality` 속성으로 품질을 조절할 수 있다.

```tsx
<Image
  src={eliceGame}
  alt="elice-game"
  placeholder="blur"
  quality={30}
/>
```

| 속성 | 설명 |
|---|---|
| `quality` | 이미지 품질 설정 |
| 범위 | 1 ~ 100 |
| 기본값 | 75 |

---

## 18.9 priority 속성

LCP 개선을 위해 페이지에서 가장 중요한 이미지를 먼저 로드하도록 설정할 수 있다.

```tsx
<Image
  src={eliceGame}
  alt="elice-game"
  placeholder="blur"
  quality={100}
  priority
/>
```

`priority`는 메인 비주얼 이미지처럼 페이지의 핵심 이미지에 사용한다.

---

# 19. Metadata

## 19.1 Metadata 기능

Next.js에는 SEO를 위한 Metadata 기능이 있다.  
직접 `head` 태그를 작성하지 않아도 페이지의 title, description, Open Graph 등의 정보를 설정할 수 있다.

---

## 19.2 File-based Metadata

정해진 파일명으로 파일을 생성하면 그에 맞는 metadata가 자동으로 추가된다.

| 파일 | 역할 |
|---|---|
| `favicon.ico` | 브라우저 탭 아이콘 |
| `manifest.json` | PWA 관련 메타 정보 |
| `robots.txt` | 검색 엔진 크롤러 접근 규칙 |

---

## 19.3 Config-based Metadata

`page` 또는 `layout` 파일에서 `metadata` 객체를 선언하고 export하면 사용할 수 있다.

```tsx
import { Metadata } from "next";

export const metadata: Metadata = {
  title: "Elice",
  description: "Hello Elice",
  openGraph: {
    title: "Elice & AI",
    description: "Elice serves AI Education",
  },
};
```

---

## 19.4 `generateMetadata`

정적인 metadata가 아니라 데이터에 따라 동적으로 metadata를 설정하려면 `generateMetadata` 함수를 export한다.

```tsx
import { ResolvingMetadata } from "next";

export async function generateMetadata(
  { params: { postId } }: Props,
  parent: ResolvingMetadata
) {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${postId}`
  );

  const fetchedPost: Post = await response.json();

  return {
    title: fetchedPost.title,
  };
}
```

---

## 19.5 `generateMetadata`의 특징

| 특징 | 설명 |
|---|---|
| 동적 metadata | 데이터 fetch 결과에 따라 title 등을 변경 가능 |
| props 전달 | page/layout처럼 params와 searchParams를 받을 수 있음 |
| parent 인자 | 상위 라우트의 Metadata 정보를 받을 수 있음 |
| SEO 개선 | 페이지별 제목, 설명, OG 태그를 동적으로 구성 가능 |

---

# 20. 전체 핵심 요약

## 20.1 Data Fetching 핵심

| 개념 | 핵심 |
|---|---|
| Server Component fetch | 서버에서 직접 데이터를 가져와 렌더링 |
| loading.tsx | route 단위 로딩 UI |
| Streaming | 준비된 HTML chunk부터 점진적으로 전송 |
| Promise.all | 여러 fetch를 병렬 실행 |
| Suspense | 컴포넌트 단위로 병렬 fetch와 fallback 처리 |
| Server Actions | 서버에서 실행되는 상태 변경 함수 |
| FormData | form 제출 데이터를 Server Action에서 받는 객체 |
| useFormStatus | form 제출 상태를 확인하는 Hook |

---

## 20.2 Caching 핵심

| 캐시 | 핵심 |
|---|---|
| Request Memoization | 같은 렌더링 중 동일 GET fetch 중복 제거 |
| Data Cache | fetch 응답 데이터를 지속적으로 저장 |
| Full Route Cache | 서버에서 렌더링된 route 결과 저장 |
| Router Cache | 클라이언트에서 hydration 완료된 결과 저장 |
| revalidatePath | 경로 기반 On-demand Revalidation |
| revalidateTag | 태그 기반 On-demand Revalidation |
| cache: no-store | fetch 단위 캐싱 끄기 |
| dynamic = force-dynamic | route 단위 캐싱 끄기 |

---

## 20.3 Next.js 버전 변경점 핵심

| 항목 | 변경 |
|---|---|
| Next.js 15 | fetch 기본 캐싱 OFF |
| Next.js 15 | Data Cache를 쓰려면 `cache: "force-cache"` 명시 필요 |
| Next.js 15 | `revalidate`, `tags` 전략은 그대로 유효 |
| Next.js 16 | `revalidateTag`에 두 번째 cacheLife 인수 필요 |
| Next.js 16 | `updateTag` 추가 |
| Next.js 16 | `refresh` 추가 |

---

## 20.4 Optimization 핵심

| 기능 | 개선 대상 | 핵심 |
|---|---|---|
| Image 컴포넌트 | CLS, LCP | 이미지 크기 확보, 최적화, priority |
| placeholder blur | UX | 이미지 로딩 중 블러 이미지 표시 |
| quality | LCP | 이미지 품질 조절 |
| Metadata | SEO | title, description, Open Graph 설정 |
| generateMetadata | SEO | 데이터 기반 동적 metadata 생성 |

---

# 21. 최종 정리

Next.js의 데이터 처리는 Server Component를 중심으로 이루어진다.  
GET 요청은 서버에서 직접 `fetch`하여 컴포넌트를 렌더링할 수 있고, 이때 브라우저 Network 탭에는 외부 API 요청이 직접 보이지 않을 수 있다.  
하지만 서버 작업이 길어지면 화면이 늦게 보이는 문제가 생기므로, `loading.tsx`와 Streaming을 활용해 준비된 UI부터 점진적으로 보여줄 수 있다.

여러 데이터 요청이 있을 때 순차적으로 `await`하면 전체 시간이 길어진다.  
`Promise.all`을 사용하면 병렬 실행할 수 있지만, 모든 작업이 끝나야 렌더링된다는 한계가 있다.  
더 좋은 방식은 각각의 데이터 요청을 별도 Server Component로 분리하고 `Suspense`로 감싸는 것이다.  
그러면 각 데이터가 준비되는 대로 독립적으로 화면에 표시할 수 있다.

POST, PUT, DELETE처럼 서버나 DB 상태를 변경하는 작업은 Server Actions로 처리할 수 있다.  
Server Action은 서버에서만 실행되는 비동기 함수이며, 클라이언트 번들에 포함되지 않는다.  
form의 `action` 속성에 Server Action을 직접 연결하면 이벤트 리스너나 불필요한 client state 없이도 서버 작업을 수행할 수 있다.  
제출 상태가 필요하면 `useFormStatus`를 사용해 pending 상태를 확인할 수 있다.

Next.js의 빠른 성능은 다양한 캐싱 메커니즘에 기반한다.  
Request Memoization은 렌더링 중 중복 fetch를 제거하고, Data Cache는 fetch 응답을 저장한다.  
Full Route Cache는 서버 렌더링 결과를 저장하고, Router Cache는 클라이언트에서 hydration된 결과를 저장한다.  
데이터 변경 이후에는 `revalidatePath`나 `revalidateTag`를 통해 캐시를 갱신해야 한다.

최적화 측면에서는 Core Web Vitals가 중요하다.  
LCP는 주요 콘텐츠 로딩 속도, INP는 상호작용 응답성, CLS는 레이아웃 이동 안정성을 측정한다.  
Next.js의 `Image` 컴포넌트는 이미지 크기 확보, placeholder, quality, priority 등을 통해 CLS와 LCP 개선에 도움을 준다.  
또한 Metadata 기능을 통해 SEO에 필요한 title, description, Open Graph 정보를 쉽게 설정할 수 있고, `generateMetadata`를 사용하면 데이터에 따라 동적으로 metadata를 구성할 수 있다.

결국 이번 파트의 핵심은 다음과 같다.

```text
1. GET 데이터는 Server Component에서 fetch한다.
2. 느린 서버 작업은 loading.tsx와 Streaming으로 보완한다.
3. 여러 fetch는 Promise.all보다 Suspense 분리가 더 세밀하다.
4. 상태 변경 작업은 Server Actions로 서버에서 처리한다.
5. 캐시는 성능의 핵심이며, 변경 후에는 Revalidation이 필요하다.
6. Image와 Metadata는 CWV와 SEO 최적화의 핵심 도구이다.
```

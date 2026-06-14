# Day 4. Next.js 풀스택 통합 실습 정리

> 원본 PDF: `Day4 - 풀스택 통합 실습.pdf`  
> 주제: Next.js 핵심 기능 복습, 프로젝트 구조, FastAPI 연동, Fetch 기반 검색, Route Handler, axios 리팩토링, URL 쿼리 기반 서버 검색

---

# 1. 전체 학습 흐름

## 1.1 실습 목표

이번 자료는 Next.js로 검색 기능을 구현하면서 **프론트엔드와 백엔드가 연결되는 풀스택 흐름**을 이해하는 것이 핵심이다.

단순히 화면만 만드는 것이 아니라, 사용자의 입력이 Next.js를 거쳐 FastAPI 백엔드와 DB까지 전달되고, 다시 결과가 화면에 표시되는 과정을 실습한다.

```text
사용자 입력
→ Next.js 프론트엔드
→ Route Handler 또는 Direct Fetch
→ FastAPI 백엔드
→ SQLite DB
→ JSON 응답
→ 화면 업데이트
```

---

## 1.2 목차

| 순서 | 내용 |
|---|---|
| 1 | Next.js 핵심 기능 살펴보기 |
| 2 | 프로젝트 구조 및 기능 이해하기 |
| 3 | Fetch 기반 검색 기능 구현하기 |
| 4 | axios로 검색 기능 리팩토링하기 |
| 5 | 검색 기능 고도화하기 |

---

# 2. Next.js 핵심 기능 복습

## 2.1 Next.js란?

Next.js는 높은 퀄리티의 풀스택 웹 애플리케이션을 제작할 수 있는 **React 기반 프레임워크**이다.

React가 UI를 만들기 위한 라이브러리라면, Next.js는 여기에 라우팅, 렌더링, 데이터 처리, 캐싱, 최적화 기능을 더해 실제 서비스 수준의 웹 애플리케이션을 만들 수 있게 해준다.

---

## 2.2 Next.js 핵심 개념 4가지

Next.js를 잘 사용하려면 프레임워크의 흐름을 관통하는 핵심 개념을 이해해야 한다.

| 핵심 개념 | 설명 |
|---|---|
| Routing과 Rendering | 폴더 구조가 URL 구조가 되는 파일 기반 라우팅, Server Component 중심 렌더링 |
| Data Fetching | 서버에서 직접 fetch, Route Handler, Server Action 등 다양한 데이터 처리 방식 |
| Caching | Data Cache, Full Route Cache, Router Cache 등 다층 캐싱 |
| Optimizing | Image, Link, Font, Script, Metadata 등 내장 최적화 기능 |

---

## 2.3 Next.js 풀스택 흐름

이번 실습에서는 검색 기능을 만들면서 다음 흐름을 확인한다.

```text
1. Next.js 핵심 개념 복습
2. 프로젝트 구조와 기능 이해
3. Fetch 기반 검색 기능 구현
4. Route Handler로 API 경유 구조 구현
5. axios로 HTTP 요청 리팩토링
6. URL 쿼리 파라미터 기반 서버 검색으로 고도화
```

---

# 3. 프로젝트 구성 살펴보기

## 3.1 프로젝트 스택

이번 실습 프로젝트는 백엔드와 프론트엔드가 하나의 저장소 안에 분리되어 있는 구조이다.

| 구분 | 기술 |
|---|---|
| 백엔드 | FastAPI, SQLAlchemy ORM, SQLite, Pydantic v2 |
| 프론트엔드 | Next.js v15+, TypeScript, Tailwind CSS v4 |
| HTTP 클라이언트 | fetch, axios |
| 저장소 구조 | `backend/`, `frontend/` 분리 |

---

## 3.2 저장소 클론

로컬에서 실행하려면 다음 명령어로 저장소를 클론한다.

```bash
git clone https://github.com/elice-contents/next.js_fullstack_boilertemplate.git
```

---

## 3.3 폴더 구조

```text
backend/
├── __pycache__/
├── blog.db              # SQLite DB 파일
├── main.py              # FastAPI 엔트리포인트
└── pyproject.toml       # Python 패키지 설정

frontend/
├── app/
│   ├── posts/
│   │   ├── [postId]/
│   │   │   ├── edit/
│   │   │   │   └── page.tsx      # 수정 페이지
│   │   │   ├── DeleteButton.tsx  # 삭제 버튼, Client Component
│   │   │   └── page.tsx          # 상세 페이지
│   │   ├── new/
│   │   │   └── page.tsx          # 작성 페이지
│   │   ├── error.tsx             # 에러 UI
│   │   ├── loading.tsx           # 로딩 UI
│   │   └── page.tsx              # 목록 페이지
│   ├── actions.ts                # Server Actions
│   ├── globals.css
│   ├── layout.tsx                # 공통 레이아웃
│   └── page.tsx                  # 홈
├── next-env.d.ts
├── next.config.mjs
├── package.json
├── postcss.config.mjs
└── tsconfig.json
```

---

## 3.4 App Router URL 매핑

Next.js App Router에서는 `app/` 폴더 구조가 URL과 연결된다.

| 파일 경로 | URL |
|---|---|
| `app/page.tsx` | `/` |
| `app/posts/page.tsx` | `/posts` |
| `app/posts/new/page.tsx` | `/posts/new` |
| `app/posts/[postId]/page.tsx` | `/posts/123` |
| `app/posts/[postId]/edit/page.tsx` | `/posts/123/edit` |

---

# 4. 백엔드 API 기능 이해

## 4.1 제공되는 API

백엔드는 FastAPI로 구성되어 있고, 게시글 CRUD 기능을 제공한다.

| Method | Endpoint | 설명 |
|---|---|---|
| GET | `/posts` | 전체 글 목록 조회 |
| GET | `/posts/{id}` | 특정 글 상세 조회 |
| POST | `/posts` | 새 글 작성 |
| PUT | `/posts/{id}` | 글 수정 |
| DELETE | `/posts/{id}` | 글 삭제 |

---

## 4.2 주요 파일 역할

| 위치 | 역할 |
|---|---|
| `backend/main.py` | 실제 FastAPI 기능 구현 |
| `frontend/app/actions.ts` | Server Actions를 모아서 관리 |
| `frontend/app/posts/page.tsx` | 게시글 목록 페이지 |
| `frontend/app/posts/[postId]/page.tsx` | 게시글 상세 페이지 |
| `frontend/app/posts/[postId]/DeleteButton.tsx` | 삭제 이벤트 처리 |

---

# 5. 프로젝트 실행하기

## 5.1 프론트엔드 실행

프론트엔드 실행 순서는 다음과 같다.

```bash
cd frontend
npm install
touch .env.local
echo FASTAPI_URL='http://localhost:8000' >> .env.local
echo BASE_PATH='/proxy/3000' >> .env.local
echo ASSET_PREFIX='/proxy/3000' >> .env.local
npm run dev
```

---

## 5.2 백엔드 실행

백엔드 실행 순서는 다음과 같다.

```bash
cd backend
uv sync
uv run fastapi dev main.py
```

---

## 5.3 실행 방식

프론트엔드와 백엔드는 각각 별도의 서버로 실행된다.  
따라서 터미널을 두 개 열고 각각 실행하는 방식이 적절하다.

```text
터미널 1: frontend 실행
터미널 2: backend 실행
```

---

# 6. frontend/app/actions.ts 이해

## 6.1 actions.ts의 역할

`actions.ts`는 Client Component에서 직접 호출할 수 있는 **Server Actions**를 정의하는 파일이다.

API 엔드포인트를 별도로 만들지 않고도 서버 측 데이터 변경 로직을 처리할 수 있다.

---

## 6.2 actions.ts의 핵심 기능

| 기능 | 설명 |
|---|---|
| Server Action 정의 | 서버에서만 실행되는 함수 작성 |
| 데이터 변경 처리 | 생성, 수정, 삭제 같은 서버 상태 변경 작업 수행 |
| 캐시 무효화 | `revalidateTag` 등을 사용해 오래된 캐시 갱신 |
| 페이지 이동 | 작업 성공 후 `redirect`로 특정 페이지 이동 |

---

## 6.3 캐시와 revalidateTag

Next.js 15 이전 버전에서는 `fetch` 결과가 캐시에 저장될 수 있다.  
이 경우 글을 생성하거나 삭제해도 기존 캐시가 남아 이전 데이터가 화면에 보일 수 있다.

이를 해결하기 위해 요청 성공 후 `revalidateTag`를 사용해 특정 캐시 태그를 무효화하고, `redirect`로 페이지를 이동한다.

```tsx
"use server";

import { revalidateTag } from "next/cache";
import { redirect } from "next/navigation";

export async function createPost(formData: FormData) {
  // 게시글 생성 로직
  revalidateTag("posts-list");
  redirect("/posts");
}
```

---

## 6.4 Next.js 15+ fetch 캐싱 업데이트

Next.js 15부터는 `fetch` 기본 캐싱 정책이 바뀌었다.  
기본적으로 fetch 결과가 자동 캐싱되지 않는다.

| 버전 | fetch 캐싱 방식 |
|---|---|
| Next.js 14 | 기본적으로 fetch 결과가 캐싱됨 |
| Next.js 15 | 기본적으로 fetch 결과가 캐싱되지 않음 |
| Next.js 16 | `use cache` 등 명시적으로 캐시를 선언하고 제어하는 방향 강화 |

---

## 6.5 Next.js 16 revalidateTag 변경점

Next.js 16부터는 `revalidateTag`의 두 번째 인수로 cacheLife 프로필을 전달해야 한다.

```tsx
revalidateTag("posts-list", "max");
```

| 버전 | 사용 예시 |
|---|---|
| Next.js 14/15 | `revalidateTag("posts-list")` |
| Next.js 16 | `revalidateTag("posts-list", "max")` |

---

# 7. FastAPI 요청과 응답 흐름

## 7.1 데이터 구조

백엔드의 `main.py`에는 DB 테이블 구조를 정의하는 SQLAlchemy 모델과, 요청/응답 데이터의 형태와 유효성을 정의하는 Pydantic 스키마가 선언되어 있다.

| 구조 | 역할 |
|---|---|
| SQLAlchemy 모델 | DB 테이블 구조 정의 |
| Pydantic 스키마 | 요청/응답 데이터의 형태와 유효성 정의 |
| SQLite DB | 실제 데이터 저장 |
| FastAPI 라우트 | HTTP 요청 처리 |

---

## 7.2 주요 데이터 모델

| 모델 | 역할 |
|---|---|
| `Post` | DB 테이블 구조를 나타내는 SQLAlchemy 모델 |
| `PostCreate` | 게시글 생성 요청 데이터 검증 |
| `PostResponse` | 게시글 응답 데이터 직렬화 |

---

## 7.3 게시물 생성 성공 흐름

게시글 생성 요청이 성공하면 다음 순서로 처리된다.

```text
1. 클라이언트가 유효한 요청 데이터를 전송한다.
2. 서버가 Pydantic으로 유효성 검사를 수행한다.
3. 유효성 검사를 통과하면 DB에 저장한다.
4. 서버가 생성된 객체를 응답한다.
5. 브라우저가 응답을 받고 상태를 업데이트한다.
```

---

## 7.4 게시물 생성 실패 흐름

실패는 크게 두 가지로 나눌 수 있다.

| 실패 상황 | 처리 |
|---|---|
| 유효성 검사 실패 | 즉시 422 에러 반환 |
| DB 오류 발생 | rollback 후 500 에러 반환 |

```text
유효하지 않은 데이터 전송
→ Pydantic 유효성 검사 실패
→ 422 반환
→ 데이터 저장 안 됨
```

```text
유효한 데이터 전송
→ Pydantic 유효성 검사 통과
→ DB 오류 발생
→ rollback
→ 500 반환
→ 데이터 저장 안 됨
```

---

# 8. Server Component와 Client Component 선택

## 8.1 게시글 목록 페이지

`frontend/app/posts/page.tsx`는 `"use client"`를 쓰지 않으므로 기본적으로 Server Component이다.

브라우저 요청 시 서버에서 `PostsPage` 함수가 실행되고, 완성된 HTML을 브라우저로 전송한다.

Server Component에서는 브라우저에서 값의 변화를 추적하는 `state`가 구조적으로 필요하지 않다.

---

## 8.2 Server Component와 Client Component 비교

| 구분 | Server Component | Client Component |
|---|---|---|
| 선언 | 별도 선언 없음 | `"use client"` 추가 |
| 함수 형태 | `async function` 가능 | 일반 `function` 중심 |
| 데이터 요청 | `await fetch` | `useEffect` 안에서 fetch |
| 값 저장 | 변수 사용 | `useState` 사용 |
| 로딩 처리 | 서버에서 완료 후 HTML 전달 | Loading State 필요 |
| 에러 처리 | `error.tsx` 등 특수 파일 활용 | State 조건부 렌더링 |
| CORS | 서버 간 통신이므로 불필요 | 브라우저 요청이므로 필요할 수 있음 |

---

## 8.3 컴포넌트 선택 기준

핵심 질문은 다음과 같다.

```text
이 컴포넌트에 사용자 인터랙션이 필요한가?
```

| 화면/기능 | 적절한 컴포넌트 |
|---|---|
| 게시글 목록 | Server Component |
| 게시글 상세 | Server Component |
| 게시글 삭제 클릭 이벤트 | Client Component |
| 게시글 작성 폼 | Server Component 또는 Client Component |
| 실시간 유효성 검사 | Client Component |
| 로딩/에러 상태 변화 | Client Component |

게시글 작성 폼은 `useState`, `onChange`, `onSubmit` 없이 HTML form과 Server Action 조합만으로 제출할 수 있기 때문에 Server Component로도 구현할 수 있다.  
다만 실시간 유효성 검사나 로딩 상태 표시가 필요하면 Client Component가 필요하다.

---

# 9. Fetch 기반 검색 기능 구현하기

## 9.1 실습 목표

검색 페이지를 만들고, Client Component를 활용해 게시물 검색 기능을 구현한다.

실습에서는 두 가지 방식을 모두 다룬다.

| 방식 | 설명 |
|---|---|
| Direct Fetch | Client Component에서 FastAPI로 직접 fetch |
| Route Handler | Client Component에서 Next.js `/api/search`를 호출하고, Route Handler가 FastAPI 호출 |

---

## 9.2 Direct Fetch 방식 흐름

```text
브라우저
→ FastAPI 직접 호출
→ 전체 게시물 조회
→ useState에 게시물 저장
→ 검색어 입력
→ 클라이언트에서 filter + includes
→ 결과 렌더링
```

---

## 9.3 Direct Fetch에서 필요한 상태

`frontend/app/search/page.tsx`를 생성하고 Client Component로 선언한다.

```tsx
"use client";
```

검색 페이지에서는 다음 상태를 관리한다.

| 상태 | 설명 |
|---|---|
| `query` | 검색어 입력값 |
| `results` | 서버에서 가져온 전체 게시물 목록 |
| `loading` | 로딩 여부 |
| `error` | 에러 메시지 |

---

## 9.4 useEffect 안에서 fetch

Client Component에서는 브라우저가 직접 FastAPI를 호출한다.  
`useEffect` 안에 async 함수를 별도로 정의하고 즉시 호출하는 패턴을 사용한다.

```tsx
useEffect(() => {
  const loadPosts = async () => {
    try {
      setLoading(true);
      setError(null);

      const res = await fetch(`${process.env.NEXT_PUBLIC_FASTAPI_URL}/posts`);
      if (!res.ok) {
        throw new Error("게시글을 불러오지 못했습니다.");
      }

      const data = await res.json();
      setResults(data);
    } catch (err) {
      setError(err instanceof Error ? err.message : "알 수 없는 에러");
    } finally {
      setLoading(false);
    }
  };

  loadPosts();
}, []);
```

---

## 9.5 catch와 finally

| 블록 | 역할 |
|---|---|
| `try` | 요청 실행과 성공 처리 |
| `catch` | 에러 상태 업데이트 |
| `finally` | 성공/실패 여부와 관계없이 로딩 종료 |

---

## 9.6 실시간 검색 필터링

`results`에는 전체 게시물 목록이 저장된다.  
`filtered`에는 검색어가 바뀔 때마다 계산되는 필터링된 목록이 저장된다.

```tsx
const filtered = results.filter((post) => {
  const keyword = query.toLowerCase();

  return (
    post.title.toLowerCase().includes(keyword) ||
    post.content.toLowerCase().includes(keyword)
  );
});
```

`toLowerCase()`를 사용하면 대소문자 구분 없이 검색할 수 있다.

---

## 9.7 Client Component용 환경 변수

Client Component에서 환경 변수를 사용하려면 `NEXT_PUBLIC_` 접두사가 필요하다.

```env
NEXT_PUBLIC_FASTAPI_URL=/proxy/8000
```

Next.js에서 `NEXT_PUBLIC_`으로 시작하는 환경 변수는 브라우저 코드에 노출될 수 있는 값이라는 의미이다.

---

# 10. CORS

## 10.1 CORS란?

CORS는 **Cross-Origin Resource Sharing**의 약자이다.  
브라우저가 다른 출처의 서버에 요청할 때 적용되는 보안 정책이다.

서버가 명시적으로 허용하지 않으면 응답이 와도 브라우저가 읽지 못하게 막는다.

---

## 10.2 Origin의 구성

Origin은 다음 세 가지로 구성된다.

```text
Protocol + Host + Port
```

예시:

| 서버 | Origin |
|---|---|
| Next.js | `http://localhost:3000` |
| FastAPI | `http://localhost:8000` |

포트가 다르면 서로 다른 Origin으로 취급된다.

---

## 10.3 Server Component와 Client Component의 CORS 차이

| 요청 위치 | CORS 필요 여부 | 이유 |
|---|---|---|
| Server Component → FastAPI | 불필요 | 서버 간 통신이므로 브라우저 CORS 정책 대상이 아님 |
| Client Component → FastAPI | 필요 | 브라우저가 다른 Origin으로 요청하기 때문 |
| Client Component → Route Handler | 일반적으로 불필요 | 같은 Next.js Origin으로 요청 |
| Route Handler → FastAPI | 불필요 | 서버 간 통신 |

---

## 10.4 FastAPI CORS 설정

FastAPI에서는 CORS Middleware를 추가해 허용할 Origin을 설정할 수 있다.

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

| 옵션 | 설명 |
|---|---|
| `allow_origins` | 허용할 출처 |
| `allow_credentials` | 쿠키/인증 헤더 포함 요청 허용 |
| `allow_methods` | 허용할 HTTP 메서드 |
| `allow_headers` | 허용할 헤더 |

---

# 11. Route Handler 기반 검색 기능

## 11.1 Route Handler 방식

Route Handler 방식에서는 브라우저가 FastAPI를 직접 호출하지 않는다.

```text
브라우저
→ Next.js Route Handler `/api/search`
→ FastAPI
→ DB
→ FastAPI 응답
→ Route Handler 응답
→ 브라우저
```

---

## 11.2 route.ts 생성

다음 파일을 생성한다.

```text
frontend/app/api/search/route.ts
```

이 파일에서 `GET` 함수를 구현하면 `/api/search` 요청을 처리할 수 있다.

---

## 11.3 Route Handler의 GET 함수

Route Handler는 서버에서 실행되므로 `NEXT_PUBLIC_` 접두사가 없는 환경 변수를 사용할 수 있다.

```tsx
export async function GET() {
  const res = await fetch(`${process.env.FASTAPI_URL}/posts`);

  if (!res.ok) {
    return Response.json(
      { message: "게시글을 불러오지 못했습니다." },
      { status: res.status }
    );
  }

  const data = await res.json();
  return Response.json(data);
}
```

---

## 11.4 page.tsx 호출 방식 변경

Direct Fetch에서는 백엔드 URL을 직접 호출했다.

```tsx
fetch(`${process.env.NEXT_PUBLIC_FASTAPI_URL}/posts`);
```

Route Handler 방식에서는 Next.js 내부 API를 호출한다.

```tsx
fetch("/api/search");
```

---

## 11.5 환경 변수 변경

Route Handler는 서버에서만 동작하므로 클라이언트에 백엔드 URL을 노출할 필요가 없다.

```env
FASTAPI_URL=http://localhost:8000
# NEXT_PUBLIC_FASTAPI_URL=/proxy/8000
```

---

## 11.6 Direct Fetch와 Route Handler 비교

| 항목 | Direct Fetch | Route Handler |
|---|---|---|
| 요청 흐름 | 브라우저가 FastAPI 직접 호출 | 브라우저 → Next.js → FastAPI |
| fetch URL | `http://localhost:8000/posts` | `/api/search` |
| 환경 변수 | `NEXT_PUBLIC_FASTAPI_URL` 필요 | `FASTAPI_URL`만 사용 가능 |
| 백엔드 URL 노출 | 브라우저 Network 탭에 노출 | 노출되지 않음 |
| CORS | FastAPI CORS 설정 필요 | 일반적으로 불필요 |
| 코드 복잡도 | 낮음 | 더 높음, `route.ts` 필요 |
| 적절한 상황 | 공개 API, 간단한 프로토타입 | API 키나 민감 정보 보호, 응답 가공 필요 |

---

# 12. 프론트엔드 ↔ 백엔드 전체 흐름

## 12.1 기본 요청/응답 흐름

프론트엔드와 백엔드는 HTTP 요청과 응답으로 연결된다.

```text
1. 사용자 액션: 버튼 클릭 또는 검색어 입력
2. 프론트엔드가 HTTP 요청 전송
3. 백엔드가 DB 쿼리 실행
4. DB가 데이터 반환
5. 백엔드가 JSON 응답
6. 프론트엔드가 UI 업데이트
```

---

## 12.2 역할 구분

| 주체 | 역할 |
|---|---|
| 사용자 Browser | 입력, 클릭 등 UI 이벤트 발생 |
| Next.js Frontend | 요청 생성, 응답 수신, 화면 렌더링 |
| FastAPI Backend | 요청 처리, 데이터 가공, DB 접근 |
| SQLite DB | 실제 데이터 저장과 조회 |

---

## 12.3 핵심 원리

모든 요청은 반드시 응답과 짝을 이룬다.

```text
Request: 사용자 → DB 방향
Response: DB → 사용자 방향
```

프론트엔드는 데이터를 요청하고, 백엔드는 그 데이터를 가공하여 전달하는 역할을 한다.

---

# 13. axios로 검색 기능 리팩토링하기

## 13.1 axios란?

axios는 브라우저와 Node.js에서 HTTP 요청을 보낼 수 있는 JavaScript 라이브러리이다.

```bash
npm install axios
```

---

## 13.2 axios의 장점

| 장점 | 설명 |
|---|---|
| 자동 JSON 변환 | `res.json()` 호출 없이 `res.data`로 접근 가능 |
| 에러 처리 편리 | 4xx, 5xx 응답이 자동으로 `catch` 블록으로 이동 |
| 타입 안정성 | 제네릭 타입을 지원 |
| 요청 설정 편리 | baseURL, timeout, headers 등을 객체로 설정 가능 |

---

## 13.3 axios 응답 스키마

axios는 응답을 객체 형태로 반환한다.  
실제 데이터는 `data` 프로퍼티에 들어 있다.

```tsx
const res = await axios.get<Post[]>("/api/search");
console.log(res.data);
```

---

## 13.4 axios 에러 스키마

axios 에러에는 여러 정보가 포함된다.

| property | 설명 |
|---|---|
| `message` | 에러 요약 메시지와 실패 status |
| `name` | axios 에러는 보통 `AxiosError` |
| `stack` | 에러 스택 트레이스 |
| `config` | 요청 당시 axios 설정 객체 |
| `code` | axios 내부 에러 코드 |
| `status` | HTTP 응답 상태 코드 |

---

## 13.5 자주 발생하는 axios 에러

| 코드 | 설명 |
|---|---|
| `ERR_NETWORK` | 네트워크 오류, 브라우저에서는 CORS 위반도 포함 가능 |
| `ERR_BAD_REQUEST` | 잘못된 요청 형식 또는 필수 파라미터 누락, 주로 4xx |
| `ERR_BAD_RESPONSE` | 응답 파싱 실패 또는 예상치 못한 형식, 주로 5xx |
| `ECONNABORTED` | 요청 timeout 또는 브라우저에 의해 중단 |
| `ETIMEDOUT` | axios 제한 시간 초과 |
| `ERR_CANCELED` | AbortSignal 등으로 요청이 명시적으로 취소됨 |
| `ERR_INVALID_URL` | 유효하지 않은 URL |

---

## 13.6 axios 에러 핸들링

axios 에러인지 확인할 때는 `axios.isAxiosError()`를 사용한다.

```tsx
try {
  const res = await axios.get<Post[]>("/api/search");
  setResults(res.data);
} catch (err) {
  if (axios.isAxiosError(err)) {
    setError(err.message);
  } else {
    setError("알 수 없는 에러가 발생했습니다.");
  }
}
```

---

## 13.7 axios 메서드별 문법

| Method | 용도 | 문법 |
|---|---|---|
| GET | 조회 | `axios.get<T>(url)` |
| POST | 생성 | `axios.post<T>(url, data)` |
| PUT | 전체 수정 | `axios.put<T>(url, data)` |
| PATCH | 일부 수정 | `axios.patch<T>(url, data)` |
| DELETE | 삭제 | `axios.delete(url)` |

예시:

```tsx
const res = await axios.get<Post[]>("/posts");
const created = await axios.post<Post>("/posts", {
  title: "새 글 제목",
  body: "내용입니다",
});
await axios.delete("/posts/1");
```

---

## 13.8 fetch에서 axios로 리팩토링

기존 fetch 방식:

```tsx
const res = await fetch("/api/search");

if (!res.ok) {
  throw new Error("검색에 실패했습니다.");
}

const data = await res.json();
setResults(data);
```

axios 방식:

```tsx
const res = await axios.get<Post[]>("/api/search");
setResults(res.data);
```

---

## 13.9 DeleteButton도 axios로 리팩토링

삭제 버튼처럼 클라이언트에서 요청을 보내는 컴포넌트도 axios로 리팩토링할 수 있다.

```tsx
await axios.delete(`/api/posts/${postId}`);
```

다만 자료에서는 Route Handler는 서버에서 실행되어 Node.js 내장 `fetch`로 충분하므로, axios는 주로 클라이언트에 적용하는 흐름을 사용한다.

---

## 13.10 Fetch와 axios 비교

| 항목 | fetch | axios |
|---|---|---|
| 설치 | 브라우저 내장, 별도 설치 불필요 | `npm install axios` 필요 |
| JSON 변환 | `res.json()` 직접 호출 | `res.data`로 자동 접근 |
| 에러 처리 | `res.ok` 직접 체크 필요 | 4xx, 5xx가 자동으로 catch로 이동 |
| 타입 지원 | 별도 타입 지정이 상대적으로 번거로움 | 제네릭 지원 |
| 서버 Route Handler | 내장 fetch로 충분 | 보통 불필요 |
| Client Component | 사용 가능 | 실무에서 자주 사용 |

---

# 14. 검색 기능 고도화하기

## 14.1 기존 방식의 한계

기존 검색 방식은 전체 게시물을 받아온 뒤 클라이언트에서 `filter()`로 검색했다.

```text
전체 데이터 요청
→ 클라이언트에 전체 저장
→ 검색어 입력
→ 브라우저에서 filter
```

이 방식은 데이터가 적을 때는 단순하지만, 데이터가 많아질수록 비효율적이다.

---

## 14.2 개선 방식

검색어를 URL 쿼리 파라미터로 보내고, 서버에서 필터링하도록 개선한다.

```text
브라우저: /search?q=리액트
→ axios.get("/api/search?q=리액트")
→ Route Handler가 q를 파싱
→ FastAPI에 /posts?q=리액트 요청
→ DB에서 필터링
→ 필터링된 결과만 반환
```

---

## 14.3 개선 효과

| 효과 | 설명 |
|---|---|
| 공유 가능 | URL이 검색 상태를 표현하므로 검색 결과를 공유하거나 북마크 가능 |
| 뒤로가기/앞으로가기 자연스러움 | URL 변경이 검색 상태 변화와 연결됨 |
| 성능 개선 | 필요한 결과만 전송하므로 네트워크와 렌더링 부담 감소 |
| 확장성 | 정렬, 페이지네이션, 필터를 같은 URL 파라미터 방식으로 확장 가능 |
| 서버 중심 처리 | 권한, 캐싱, 필터링을 서버에서 관리 가능 |

---

# 15. 백엔드 검색 기능 고도화

## 15.1 FastAPI에서 q 파라미터 받기

`backend/main.py`의 `get_posts` 함수에 `q` 파라미터를 추가한다.

```python
from typing import Optional

@app.get("/posts")
def get_posts(q: Optional[str] = None):
    # q가 없으면 전체 게시글 반환
    # q가 있으면 제목 또는 본문 기준 필터링
```

---

## 15.2 동작 방식

| q 값 | 동작 |
|---|---|
| 없음 | 기존과 동일하게 전체 게시글 반환 |
| 있음 | SQLAlchemy `or` 조건으로 제목 또는 본문 내용 필터링 |

---

## 15.3 서버 필터링 흐름

```text
GET /posts?q=리액트
→ FastAPI가 q 파라미터 수신
→ SQLAlchemy로 title 또는 content LIKE 검색
→ DB에서 조건에 맞는 결과만 조회
→ 필터링된 JSON 반환
```

---

# 16. Route Handler 검색 기능 고도화

## 16.1 route.ts에서 request 받기

`frontend/app/api/search/route.ts`의 GET 함수 시그니처를 변경해 브라우저의 요청 객체를 직접 받는다.

```tsx
export async function GET(request: Request) {
  // request.url 파싱
}
```

---

## 16.2 URL 파싱

`new URL(request.url)`로 URL을 파싱하고, `searchParams.get("q")`로 검색어를 추출한다.

```tsx
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const q = searchParams.get("q");

  const url = q
    ? `${process.env.FASTAPI_URL}/posts?q=${encodeURIComponent(q)}`
    : `${process.env.FASTAPI_URL}/posts`;

  const res = await fetch(url);
  const data = await res.json();

  return Response.json(data);
}
```

---

## 16.3 q 유무에 따른 FastAPI URL 분기

| q 상태 | FastAPI 호출 URL |
|---|---|
| q 없음 | `/posts` |
| q 있음 | `/posts?q=검색어` |

---

## 16.4 encodeURIComponent

검색어에 한글, 공백, 특수문자가 포함되면 URL이 깨질 수 있다.  
Route Handler에서 FastAPI URL을 직접 조합할 때는 `encodeURIComponent`로 검색어를 인코딩해야 한다.

```tsx
encodeURIComponent(q)
```

예시:

| 원본 검색어 | 인코딩 후 |
|---|---|
| `리액트` | `%EB%A6%AC%EC%95%A1%ED%8A%B8` |
| `react query` | `react%20query` |
| `a&b` | `a%26b` |

---

# 17. 프론트엔드 검색 페이지 고도화

## 17.1 URL 쿼리 파라미터 기반 검색

기존에는 검색어를 React state로만 관리하고 클라이언트에서 필터링했다.  
고도화 이후에는 URL의 `q` 값을 검색 상태로 사용한다.

---

## 17.2 useSearchParams

`useSearchParams()`로 현재 URL의 `q` 값을 읽는다.

```tsx
"use client";

import { useSearchParams } from "next/navigation";

const searchParams = useSearchParams();
const query = searchParams.get("q") ?? "";
```

---

## 17.3 router로 URL 갱신

입력창 값이 바뀌면 `router`를 활용해 URL을 갱신한다.

```tsx
import { useRouter } from "next/navigation";

const router = useRouter();

const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  const value = e.target.value;

  if (value) {
    router.push(`/search?q=${encodeURIComponent(value)}`);
  } else {
    router.push("/search");
  }
};
```

---

## 17.4 클라이언트 filter 제거

고도화 후에는 클라이언트에서 `filter()`를 사용하지 않는다.

```text
기존:
전체 데이터 수신
→ 클라이언트 filter

변경:
검색어를 서버로 전달
→ 서버/DB에서 필터링
→ 필터링된 결과만 수신
```

---

# 18. 고도화 후 전체 검색 흐름

## 18.1 최종 흐름

```text
1. 사용자가 검색창에 입력하고 Enter 또는 이벤트 발생
2. URL이 /search?q=리액트 형태로 변경
3. 프론트엔드가 /api/search?q=리액트 요청
4. Route Handler가 q를 읽고 FastAPI에 /posts?q=리액트 요청
5. FastAPI가 DB에서 WHERE 조건으로 필터링
6. 필터링된 JSON 데이터 반환
7. 검색 결과만 화면에 렌더링
```

---

## 18.2 요청과 응답 방향

```text
Request:
Browser → Next.js Route Handler → FastAPI → DB

Response:
DB → FastAPI → Next.js Route Handler → Browser
```

---

## 18.3 최종 구조의 장점

| 장점 | 설명 |
|---|---|
| URL이 검색 상태 | 공유, 북마크, 뒤로가기 가능 |
| 필요한 데이터만 수신 | 네트워크 비용 절감 |
| 브라우저 부담 감소 | 클라이언트 필터링 제거 |
| 서버 중심 확장 가능 | 정렬, 페이지네이션, 권한, 캐싱 확장 가능 |
| 백엔드 URL 보호 | Route Handler를 거치므로 FastAPI URL 직접 노출 감소 |

---

# 19. 전체 핵심 요약

## 19.1 프로젝트 구조 핵심

| 개념 | 핵심 |
|---|---|
| `backend/` | FastAPI, SQLAlchemy, SQLite, Pydantic 기반 백엔드 |
| `frontend/` | Next.js, TypeScript, Tailwind CSS 기반 프론트엔드 |
| `actions.ts` | Server Actions를 모아서 관리 |
| `app/api/search/route.ts` | Next.js Route Handler |
| `app/posts/` | 게시글 목록, 상세, 작성, 수정, 삭제 페이지 |

---

## 19.2 풀스택 데이터 흐름 핵심

| 단계 | 설명 |
|---|---|
| 사용자 액션 | 검색어 입력, 버튼 클릭 등 |
| 프론트엔드 요청 | fetch 또는 axios로 HTTP 요청 |
| Route Handler | 필요한 경우 Next.js 서버에서 백엔드 요청 중계 |
| 백엔드 처리 | FastAPI가 요청 검증, DB 조회/수정 |
| DB 응답 | SQLite에서 데이터 반환 |
| UI 업데이트 | 프론트엔드가 JSON 응답을 렌더링 |

---

## 19.3 Server Component와 Client Component 핵심

| 기준 | 선택 |
|---|---|
| 사용자 인터랙션이 필요 없음 | Server Component |
| 클릭, 입력, state, effect 필요 | Client Component |
| HTML form + Server Action으로 충분 | Server Component 가능 |
| 실시간 유효성 검사, pending 표시 필요 | Client Component 필요 |

---

## 19.4 Direct Fetch와 Route Handler 핵심

| 구분 | Direct Fetch | Route Handler |
|---|---|---|
| 단순성 | 더 단순함 | 파일 추가로 조금 복잡함 |
| 보안 | 백엔드 URL 노출 | 백엔드 URL 숨김 |
| CORS | 필요 | 보통 불필요 |
| 응답 가공 | 클라이언트에서 직접 처리 | Next.js 서버에서 가공 가능 |
| 추천 상황 | 공개 API, 빠른 프로토타입 | 민감 정보 보호, 서버 중계 필요 |

---

## 19.5 fetch와 axios 핵심

| 구분 | fetch | axios |
|---|---|---|
| 설치 | 내장 | 설치 필요 |
| JSON 처리 | `res.json()` 필요 | `res.data` |
| 에러 처리 | `res.ok` 직접 확인 | 4xx/5xx 자동 catch |
| 타입 | 직접 관리 | 제네릭 지원 |
| 적용 위치 | Server/Client 모두 가능 | 주로 Client Component 리팩토링에 활용 |

---

## 19.6 검색 기능 고도화 핵심

```text
기존 방식:
전체 데이터 조회
→ 클라이언트 filter
→ 검색 상태는 컴포넌트 내부 state

개선 방식:
URL q 파라미터 사용
→ Route Handler에서 q 파싱
→ FastAPI가 DB 필터링
→ 필요한 결과만 반환
```

---

# 20. 최종 정리

이번 실습은 Next.js와 FastAPI를 연결해 실제 풀스택 흐름을 이해하는 과정이다.  
프로젝트는 `backend/`와 `frontend/`로 나뉘며, 백엔드는 FastAPI, SQLAlchemy, SQLite, Pydantic으로 구성되고 프론트엔드는 Next.js, TypeScript, Tailwind CSS로 구성된다.

Next.js의 핵심은 Routing과 Rendering, Data Fetching, Caching, Optimizing이다.  
실습에서는 이 중 특히 Data Fetching과 Route Handler, Server Component와 Client Component의 역할 차이를 중심으로 검색 기능을 구현한다.

처음에는 Client Component에서 FastAPI를 직접 호출하는 Direct Fetch 방식으로 검색 기능을 만든다.  
이 방식은 단순하지만 백엔드 URL이 브라우저 Network 탭에 노출되고, CORS 설정이 필요하다.  
이를 보완하기 위해 Next.js의 Route Handler를 사용하면 브라우저는 `/api/search`만 호출하고, Next.js 서버가 FastAPI를 대신 호출한다.  
이 구조는 백엔드 URL을 숨기고 CORS 문제를 줄이며, 응답 데이터를 서버에서 가공할 수 있다는 장점이 있다.

이후 HTTP 요청 코드를 axios로 리팩토링한다.  
axios는 `res.data`로 자동 JSON 변환을 제공하고, 4xx와 5xx 응답을 자동으로 catch 블록으로 넘겨주며, 제네릭 타입을 통해 타입 안정성도 확보할 수 있다.  
다만 Route Handler처럼 서버에서 실행되는 코드는 Node.js 내장 fetch로 충분하므로 axios는 주로 클라이언트 컴포넌트 요청에 적용한다.

마지막으로 검색 기능을 URL 쿼리 파라미터 기반으로 고도화한다.  
기존에는 전체 게시글을 가져와 브라우저에서 `filter()`로 검색했지만, 개선 후에는 `/search?q=검색어`처럼 URL이 검색 상태를 표현하고, Route Handler가 이 값을 FastAPI로 전달한다.  
FastAPI는 DB에서 직접 필터링한 결과만 반환하므로 네트워크와 렌더링 부담이 줄어든다.  
또한 URL이 검색 상태가 되기 때문에 검색 결과 공유, 북마크, 뒤로가기와 앞으로가기까지 자연스럽게 동작한다.

결국 이번 실습의 핵심은 다음과 같다.

```text
1. Next.js는 프론트엔드와 서버 로직을 함께 다룰 수 있다.
2. Server Component와 Client Component는 사용자 인터랙션 여부로 구분한다.
3. Direct Fetch는 단순하지만 URL 노출과 CORS 문제가 있다.
4. Route Handler는 Next.js 서버를 중계 계층으로 사용해 보안성과 확장성을 높인다.
5. axios는 클라이언트 요청 코드를 더 간결하고 안정적으로 만든다.
6. 검색 상태는 URL 쿼리 파라미터로 표현하면 공유성과 확장성이 좋아진다.
7. 데이터가 많아질수록 클라이언트 필터링보다 서버/DB 필터링이 적절하다.
```

# Day 5-1 · Day 5-2 · Day 5-3 배포와 E2E 검증 정리

> 원본 자료:  
> - `Day5-1 - 풀스택 통합 실습 복습.md`  
> - `Day5-2 - Vercel & Railway 소개 및 배포.md`  
> - `Day5-3 - E2E 검증과 Docker AWS 개요.md`  
>
> 정리 순서: **5-1 풀스택 통합 실습 복습 → 5-2 Vercel & Railway 배포 → 5-3 E2E 검증과 Docker/AWS 개요**

---

# 1. Day 5-1 — 풀스택 통합 실습 복습

## 1.1 전체 복습 흐름

이번 파트는 이전에 구현했던 Next.js + FastAPI 풀스택 앱의 검색 기능을 다시 정리하는 내용이다.

핵심은 다음 두 가지 방식으로 검색 기능을 구현하고 비교하는 것이다.

```text
1. Direct Fetch
   브라우저 → FastAPI 직접 호출

2. Route Handler
   브라우저 → Next.js Route Handler → FastAPI
```

그리고 기존 `fetch` 기반 코드를 `axios`로 리팩토링한다.

---

# 2. 실습 1 — Fetch 기반 검색 기능 구현

## 2.1 실습 목표

클라이언트 컴포넌트 안에서 검색어를 입력하면, 실시간으로 글 목록이 필터링되는 화면을 구현한다.

기본 흐름은 다음과 같다.

```text
전체 게시글 데이터 조회
→ 검색어 입력
→ 검색어를 기준으로 게시글 필터링
→ 결과 목록 렌더링
```

---

## 2.2 Direct Fetch 방식

Direct Fetch는 브라우저가 FastAPI 서버를 직접 호출하는 방식이다.

```text
Browser
→ FastAPI /posts
→ 전체 게시글 데이터 반환
→ Client Component에서 필터링
```

---

## 2.3 Direct Fetch의 특징

| 항목 | 내용 |
|---|---|
| 호출 위치 | 브라우저 |
| 호출 대상 | FastAPI |
| 환경변수 | `NEXT_PUBLIC_FASTAPI_URL` 필요 |
| 장점 | 구조가 단순하고 직관적 |
| 단점 | 브라우저 Network 탭에 백엔드 주소 노출 |
| CORS | FastAPI에서 CORS 허용 설정 필요 |
| 적합한 경우 | 공개 API, 간단한 프로토타입 |

---

## 2.4 NEXT_PUBLIC_ 접두사

Next.js는 기본적으로 환경변수를 서버에서만 읽을 수 있게 제한한다.  
Client Component는 브라우저에서 실행되므로, 브라우저 코드에서 환경변수를 사용하려면 `NEXT_PUBLIC_` 접두사를 붙여야 한다.

| 환경변수 | 사용 위치 | 설명 |
|---|---|---|
| `FASTAPI_URL` | 서버 전용 | Server Component, Server Action, Route Handler에서 사용 |
| `NEXT_PUBLIC_FASTAPI_URL` | 브라우저 공개용 | Client Component에서 사용 가능 |

주의할 점은 `NEXT_PUBLIC_`으로 시작하는 값은 브라우저에 노출될 수 있다는 것이다.  
따라서 API Key나 비밀 정보에는 사용하면 안 된다.

---

## 2.5 Route Handler 방식

Route Handler는 브라우저가 FastAPI를 직접 호출하지 않고, Next.js 서버의 API 라우트를 거쳐 백엔드에 요청하는 방식이다.

```text
Browser
→ Next.js /api/search
→ FastAPI /posts
→ Next.js가 응답 전달
→ Browser
```

---

## 2.6 Route Handler의 특징

| 항목 | 내용 |
|---|---|
| 호출 위치 | 브라우저는 Next.js 내부 API만 호출 |
| 실제 백엔드 호출 | Next.js 서버가 대신 수행 |
| 환경변수 | `FASTAPI_URL` 사용 가능 |
| 장점 | 백엔드 주소를 브라우저에 노출하지 않음 |
| CORS | 서버 간 통신이므로 CORS 문제 없음 |
| 단점 | `route.ts` 파일이 필요해 코드가 조금 더 복잡 |
| 적합한 경우 | 민감 정보 보호, 응답 가공, 실제 서비스 구조 |

---

# 3. 프로젝트 실행

## 3.1 백엔드 실행

```bash
cd fullstack-practice/backend
uv run fastapi dev main.py
```

---

## 3.2 프론트엔드 실행

```bash
cd fullstack-practice/frontend
npm install
npm run dev
```

---

## 3.3 환경변수 설정

`frontend/.env.local` 파일에 환경변수를 설정한다.

```env
FASTAPI_URL='http://localhost:8000'
```

Direct Fetch 방식을 사용할 경우에는 Client Component에서 접근할 수 있도록 다음 환경변수도 필요하다.

```env
NEXT_PUBLIC_FASTAPI_URL='http://localhost:8000'
```

런박스나 프록시 환경에서는 `BASE_PATH` 관련 환경변수도 사용할 수 있다.

```env
NEXT_PUBLIC_BASE_PATH='/proxy/3000'
```

---

# 4. 검색 페이지 구현 구조

## 4.1 search/page.tsx의 역할

`frontend/app/search/page.tsx`는 검색 화면을 담당한다.  
사용자 입력과 실시간 필터링이 필요하므로 Client Component로 구현한다.

```tsx
"use client";
```

---

## 4.2 필요한 상태

검색 페이지에서는 다음 상태를 관리한다.

| 상태 | 타입 | 설명 |
|---|---|---|
| `query` | `string` | 검색어 입력값 |
| `results` | `Post[]` | 서버에서 가져온 전체 게시글 목록 |
| `loading` | `boolean` | 데이터 로딩 여부 |
| `error` | `string \| null` | 에러 메시지 |

---

## 4.3 Post 타입

```tsx
type Post = {
  id: number;
  title: string;
  content: string;
  created_at: string;
};
```

---

## 4.4 Fetch 체이닝 패턴

React의 `useEffect` 안에서 데이터를 가져올 때 다음 패턴을 사용할 수 있다.

```tsx
fetch("요청할_주소")
  .then((response) => {
    if (!response.ok) throw new Error("에러 발생!");
    return response.json();
  })
  .then((data) => {
    setResults(data);
  })
  .catch((error) => {
    setError(error.message);
  })
  .finally(() => {
    setLoading(false);
  });
```

| 단계 | 역할 |
|---|---|
| `then(response)` | 요청 성공 여부 확인 및 JSON 변환 |
| `then(data)` | 변환된 데이터를 state에 저장 |
| `catch(error)` | 에러 상태 처리 |
| `finally()` | 성공/실패와 관계없이 로딩 종료 |

---

## 4.5 Direct Fetch 정답 구조

```tsx
useEffect(() => {
  setLoading(true);
  setError(null);

  fetch(`${process.env.NEXT_PUBLIC_FASTAPI_URL}/posts`)
    .then((res) => {
      if (!res.ok) throw new Error("데이터를 불러오는 데 실패했습니다");
      return res.json();
    })
    .then((data) => setResults(data))
    .catch((err) => setError(err.message))
    .finally(() => setLoading(false));
}, []);
```

---

## 4.6 Route Handler 정답 구조

```tsx
useEffect(() => {
  setLoading(true);
  setError(null);

  fetch(`${BASE_PATH}/api/search`)
    .then((res) => {
      if (!res.ok) throw new Error("데이터를 불러오는 데 실패했습니다");
      return res.json();
    })
    .then((data) => setResults(data))
    .catch((err) => setError(err.message))
    .finally(() => setLoading(false));
}, []);
```

---

## 4.7 검색어 기반 필터링

`results` 배열에서 제목이나 내용에 검색어가 포함된 게시글만 남긴다.

```tsx
const filtered: Post[] = results.filter(
  (post) =>
    post.title.includes(query) || post.content.includes(query)
);
```

---

# 5. Route Handler 구현

## 5.1 route.ts 파일 위치

Route Handler 방식에서는 다음 파일을 생성한다.

```text
frontend/app/api/search/route.ts
```

이 파일은 다음 URL에 대응된다.

```text
/api/search
```

---

## 5.2 Route Handler 역할

브라우저가 `/api/search`로 요청하면 이 서버 사이드 핸들러가 FastAPI를 대신 호출하고 결과를 브라우저에 반환한다.

```text
Browser
→ GET /api/search
→ route.ts 실행
→ FastAPI /posts 호출
→ NextResponse.json(data)
→ Browser
```

---

## 5.3 route.ts 정답 코드

```tsx
import { NextResponse } from "next/server";

export async function GET() {
  const fastapiUrl = process.env.FASTAPI_URL;

  if (!fastapiUrl) {
    return NextResponse.json(
      { detail: "FASTAPI_URL 환경 변수가 설정되지 않았습니다" },
      { status: 500 }
    );
  }

  const res = await fetch(`${fastapiUrl}/posts`);

  if (!res.ok) {
    return NextResponse.json(
      { detail: "게시글 목록을 불러오는 데 실패했습니다" },
      { status: res.status }
    );
  }

  const data = await res.json();
  return NextResponse.json(data);
}
```

---

## 5.4 Direct Fetch와 Route Handler 비교

| 구분 | Direct Fetch | Route Handler |
|---|---|---|
| 호출 주소 | `NEXT_PUBLIC_FASTAPI_URL/posts` | `/api/search` |
| 통신 흐름 | 브라우저 → FastAPI | 브라우저 → Next.js 서버 → FastAPI |
| 환경변수 노출 | `NEXT_PUBLIC_` 접두사 필요 | 브라우저에 노출할 필요 없음 |
| CORS | FastAPI CORS 설정 필요 | 서버 내부 호출이라 CORS 에러 없음 |
| 코드 복잡도 | 낮음 | 상대적으로 높음 |
| 보안성 | 백엔드 주소 노출 | 백엔드 주소 숨김 |

---

# 6. 실습 2 — axios로 리팩토링

## 6.1 axios란?

axios는 JavaScript 환경에서 서버와 HTTP 비동기 통신을 주고받기 위해 널리 사용되는 오픈소스 라이브러리이다.

```bash
npm install axios
```

---

## 6.2 fetch 대신 axios를 사용하는 이유

| 이유 | 설명 |
|---|---|
| 자동 JSON 변환 | `res.json()` 없이 `response.data`로 접근 가능 |
| 직관적인 에러 처리 | 2xx가 아닌 응답은 자동으로 `catch`로 이동 |
| 구조화된 에러 객체 | `axios.isAxiosError(err)`로 axios 에러 구분 |
| 다양한 기능 | Interceptor, 요청 취소, timeout 설정 등 지원 |

---

## 6.3 axios 기본 사용법

```tsx
import axios from "axios";

async function getData() {
  try {
    const response = await axios.get<DataType[]>("요청할_주소");
    console.log(response.data);
  } catch (err) {
    if (axios.isAxiosError(err)) {
      console.error(err.response?.data?.detail);
    } else {
      console.error("알 수 없는 오류 발생");
    }
  }
}
```

---

## 6.4 search/page.tsx 리팩토링

기존 `fetch` 기반 Route Handler 호출을 `axios`로 교체한다.

```tsx
"use client";

import { useState, useEffect } from "react";
import Link from "next/link";
import axios from "axios";

type Post = {
  id: number;
  title: string;
  content: string;
  created_at: string;
};

export default function SearchPage() {
  const [query, setQuery] = useState<string>("");
  const [results, setResults] = useState<Post[]>([]);
  const [loading, setLoading] = useState<boolean>(false);
  const [error, setError] = useState<string | null>(null);

  const BASE_PATH = process.env.NEXT_PUBLIC_BASE_PATH ?? "";

  useEffect(() => {
    async function fetchPosts() {
      setLoading(true);
      setError(null);

      try {
        const res = await axios.get<Post[]>(`${BASE_PATH}/api/search`);
        setResults(res.data);
      } catch (err) {
        if (axios.isAxiosError(err)) {
          setError(err.response?.data?.detail ?? "게시글을 불러오는 데 실패했습니다");
        } else {
          setError("알 수 없는 오류가 발생했습니다");
        }
      } finally {
        setLoading(false);
      }
    }

    fetchPosts();
  }, []);

  const filtered = results.filter(
    (post) => post.title.includes(query) || post.content.includes(query)
  );

  return (
    <main>{/* 검색 UI */}</main>
  );
}
```

---

## 6.5 DeleteButton.tsx 리팩토링

기존 삭제 요청도 `fetch`에서 `axios.delete()`로 교체할 수 있다.

```tsx
"use client";

import axios from "axios";

export default function DeleteButton({ postId }: { postId: number }) {
  const BASE_PATH = process.env.NEXT_PUBLIC_BASE_PATH ?? "";

  async function handleDelete() {
    if (!confirm("정말 삭제할까요?")) return;

    try {
      await axios.delete(`${BASE_PATH}/api/posts/${postId}`);
      window.location.href = `${BASE_PATH}/posts`;
    } catch (err) {
      if (axios.isAxiosError(err)) {
        alert(err.response?.data?.detail ?? "게시글 삭제에 실패했습니다");
      } else {
        alert("알 수 없는 오류가 발생했습니다");
      }
    }
  }

  return (
    <button onClick={handleDelete}>
      삭제하기
    </button>
  );
}
```

---

## 6.6 fetch와 axios 리팩토링 비교

| 비교 항목 | 기존 fetch 방식 | axios 방식 | 리팩토링 효과 |
|---|---|---|---|
| JSON 데이터 변환 | `const data = await res.json()` | `const data = res.data` | 코드가 간결하고 데이터 접근이 직관적 |
| HTTP 에러 감지 | `if (!res.ok) throw new Error(...)` | 2xx가 아니면 자동으로 `catch` 이동 | 수동 에러 체크 누락 방지 |
| 서버 에러 메시지 | 실패 응답 body를 다시 읽어야 함 | `err.response.data.detail` 접근 | 백엔드 detail 추출이 쉬움 |
| 메서드 사용 | `method: "DELETE"` | `axios.delete()` | 자동완성, 가독성, 타입 안정성 향상 |

---

# 7. Day 5-2 — Vercel & Railway 소개 및 배포

## 7.1 Vercel이란?

Vercel은 Next.js를 배포하는 가장 간단한 방법 중 하나이다.  
Next.js를 만든 회사가 직접 운영하는 서비스이므로 Next.js와의 호환성이 좋다.

배포 후 다음과 같은 URL이 생성된다.

```text
https://your-project.vercel.app
```

---

## 7.2 Vercel 특징

| 특징 | 설명 |
|---|---|
| 무료 플랜 | 개인 프로젝트를 무료로 배포 가능 |
| 자동 배포 | GitHub push 시 자동 재배포 |
| Preview 배포 | PR을 열면 미리보기 URL 자동 생성 |
| Next.js 친화적 | Next.js 제작사가 운영하여 호환성이 좋음 |

---

## 7.3 배포할 프로젝트 구조

이번 배포 대상 프로젝트는 `fullstack-practice`이다.

```text
fullstack-practice/
├── frontend/    # Vercel에 배포
└── backend/     # Railway에 배포
```

---

## 7.4 배포 흐름

Next.js 프론트엔드는 Vercel에 배포하고, FastAPI 백엔드는 Railway에 배포한다.

```text
1단계: Railway에 FastAPI 백엔드 배포
2단계: Vercel에 Next.js 프론트엔드 배포
3단계: Vercel 환경변수에 Railway URL 입력
4단계: Railway CORS에 Vercel URL 허용
```

---

# 8. GitHub 레포지토리 준비

## 8.1 GitHub에 코드 올리기

배포를 위해서는 코드가 GitHub에 올라가 있어야 한다.

```bash
cd fullstack-practice

git init
git add .
git commit -m "initial commit"
git branch -M main

git remote add origin https://github.com/{username}/fullstack-practice
git push -u origin main
```

---

## 8.2 배포에서 GitHub가 필요한 이유

| 이유 | 설명 |
|---|---|
| 서비스 연결 | Vercel과 Railway가 GitHub 레포지토리에서 코드를 가져옴 |
| 자동 배포 | push 시 변경사항이 자동으로 반영됨 |
| 배포 이력 관리 | 어떤 커밋이 배포되었는지 추적 가능 |
| 협업 | 팀원과 함께 코드와 배포 흐름을 관리 가능 |

---

# 9. Railway로 FastAPI 백엔드 배포

## 9.1 Railway란?

Railway는 서버가 항상 켜져 있는 **컨테이너 기반 호스팅 서비스**이다.  
Python, Node.js, Go 등 다양한 언어로 만든 서버를 GitHub 레포지토리만 연결해 배포할 수 있다.

---

## 9.2 FastAPI를 Vercel에 올리지 않는 이유

Vercel은 Serverless 환경이다.  
요청이 들어올 때만 잠깐 실행되고, 요청이 없으면 꺼지는 구조이다.

이 구조에서는 SQLite처럼 파일 시스템에 직접 저장하는 DB가 적합하지 않다.  
FastAPI를 Vercel에 올리면 서버는 뜰 수 있지만, `blog.db` 파일이 요청마다 초기화되어 데이터가 사라질 수 있다.

Railway는 컨테이너 기반이므로 서버가 계속 켜져 있고, 파일 시스템도 유지된다.  
따라서 SQLite 파일도 서버가 재시작되지 않는 한 유지될 수 있다.

---

## 9.3 로컬과 Railway 배포 차이

| 구분 | 로컬 개발 | Railway 배포 |
|---|---|---|
| 접근 가능 범위 | 내 컴퓨터에서만 | 전 세계 공개 URL |
| FastAPI 주소 | `http://localhost:8000` | `https://xxx.up.railway.app` |
| Vercel에서 호출 가능 여부 | 불가. localhost는 내 PC 기준 | 가능. 공개 URL |
| DB 유지 | 로컬 파일로 유지 | 서버 재시작 전까지 유지 |

---

## 9.4 Railway 무료 티어 주의

Railway는 Trial 플랜으로 일정 크레딧을 제공한다.  
실습 범위에서는 충분할 수 있지만, 크레딧이 소진되면 서버가 중단될 수 있다.

실습 후 사용하지 않는 서비스는 삭제해두는 것이 좋다.

---

## 9.5 Railway 배포 단계

```text
1. Railway 가입
2. GitHub 로그인 및 코드 접근 허용
3. Add Service에서 GitHub Repo 선택
4. Settings 탭에서 Root Directory를 backend로 설정
5. Deploy 탭에서 Start Command 입력
6. Variables 탭에서 환경변수 설정
7. Deploy 실행
8. Networking에서 Railway 도메인 생성
```

---

## 9.6 Railway Start Command

FastAPI 서버를 Railway에서 실행하려면 다음 Start Command를 사용한다.

```bash
fastapi run main.py --host 0.0.0.0 --port $PORT
```

| 옵션 | 의미 |
|---|---|
| `--host 0.0.0.0` | 외부에서 접근 가능하도록 모든 네트워크 인터페이스에서 대기 |
| `--port $PORT` | Railway가 제공하는 포트 환경변수 사용 |

---

## 9.7 Railway 도메인 생성

Railway에서 도메인을 생성하면 다음과 같은 공개 URL이 만들어진다.

```text
https://next-js-fullstack-production.up.railway.app
```

이 URL은 나중에 Vercel 환경변수 `FASTAPI_URL`에 입력한다.

---

# 10. CORS 설정

## 10.1 배포 시 CORS가 필요한 이유

로컬에서는 Next.js가 `http://localhost:3000`, FastAPI가 `http://localhost:8000`에서 실행된다.  
배포 후에는 Vercel과 Railway가 서로 다른 도메인에서 동작한다.

따라서 브라우저에서 직접 백엔드에 요청하는 경우 CORS 설정이 필요하다.

---

## 10.2 main.py CORS 수정 예시

```python
import os

app = FastAPI(title="Blog API")

FRONTEND_URL = os.getenv("FRONTEND_URL", "http://localhost:3000")

origins = [
    "http://localhost:3000",
    FRONTEND_URL,
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## 10.3 CORS 환경변수 운영

Railway Variables 탭에서 Vercel URL을 추가한다.

```text
https://your-project.vercel.app,http://localhost:3000
```

자료에서는 `CORS_ORIGINS`라는 변수 이름을 언급하지만, 실제 코드에서는 `FRONTEND_URL` 또는 팀에서 정한 변수명과 일치해야 한다.

---

## 10.4 임시 CORS 설정 주의

복잡하면 임시로 다음처럼 모든 Origin을 허용할 수도 있다.

```python
allow_origins=["*"]
```

하지만 실무에서는 크레덴셜 유출 위험이 있으므로 권장되지 않는다.  
실제 서비스에서는 허용할 프론트엔드 도메인을 명시하는 것이 안전하다.

---

# 11. Vercel로 Next.js 프론트엔드 배포

## 11.1 Vercel 배포 단계

```text
1. Vercel 가입
2. GitHub 레포지토리 가져오기
3. 프로젝트의 frontend 디렉토리를 배포 대상으로 설정
4. Environment Variables에 FASTAPI_URL 입력
5. Deploy 클릭
6. 빌드 로그 확인
7. Vercel URL 생성 확인
```

---

## 11.2 Vercel 환경변수

Vercel에는 Railway에서 생성한 FastAPI URL을 환경변수로 입력한다.

```env
FASTAPI_URL=https://your-app.up.railway.app
```

`FASTAPI_URL`은 Next.js 서버에서만 사용하는 환경변수이다.  
`NEXT_PUBLIC_` 접두사를 붙이지 않으면 브라우저에 노출되지 않는다.

---

## 11.3 배포 후 Railway CORS 업데이트

Vercel 배포가 끝나면 생성된 Vercel URL을 Railway 백엔드 CORS 설정에 추가해야 한다.

```text
Railway 대시보드
→ backend 서비스
→ Variables
→ CORS_ORIGINS 또는 FRONTEND_URL 업데이트
→ Redeploy
```

예시:

```text
https://your-project.vercel.app,http://localhost:3000
```

---

# 12. 배포 후 확인

## 12.1 확인 항목

| 확인 | 설명 |
|---|---|
| Vercel URL 접속 | 프론트엔드 페이지가 로드되는지 확인 |
| Railway URL/docs 접속 | Swagger UI가 열리는지 확인 |
| 게시글 목록 | Vercel 앱에서 게시글 목록을 불러오는지 확인 |
| 검색 기능 | 배포된 URL에서 검색이 동작하는지 확인 |
| 작성/수정/삭제 | CRUD 기능이 정상 동작하는지 확인 |

---

## 12.2 공통 트러블슈팅

| 문제 | 확인할 것 |
|---|---|
| 응답 없음, 500 에러 | Railway 로그 확인, 서버 시작 실패, requirements 누락, DB 경로 문제 |
| CORS 에러 | Railway Variables에 Vercel URL이 있는지 확인 |
| 게시글 목록이 비어 있음 | Vercel 환경변수 `FASTAPI_URL` 확인 |
| 환경변수 수정했는데 반영 안 됨 | 환경변수 변경 후 Redeploy 필요 |
| http/https 문제 | Vercel URL과 Railway URL의 프로토콜 정확히 확인 |

---

# 13. 직접 배포 실습 체크리스트

```text
1. fullstack-practice 폴더를 GitHub에 push
2. Railway에 backend 디렉토리로 FastAPI 배포
3. Railway 도메인 생성 후 URL 복사
4. main.py CORS 수정 또는 Variables 설정
5. Vercel에 frontend 디렉토리로 Next.js 배포
6. Vercel 환경변수 FASTAPI_URL 입력
7. Railway CORS에 Vercel URL 추가
8. 배포된 화면에서 검색 기능 확인
```

---

# 14. Day 5-3 — E2E 검증과 Docker/AWS 개요

## 14.1 E2E 검증이란?

E2E는 End-to-End의 약자이다.  
사용자의 실제 행동 흐름을 처음부터 끝까지 따라가면서 앱 전체가 문제없이 동작하는지 확인하는 과정이다.

로컬 환경에서는 프론트엔드, 백엔드, DB가 같은 컴퓨터에 있어 잘 동작할 수 있다.  
하지만 배포 후에는 프론트엔드가 Vercel에 있고, 백엔드가 Railway에 있으며, DB는 Railway 서버의 SQLite 파일로 존재한다.

따라서 배포 환경에서도 데이터가 정상적으로 오가는지 확인해야 한다.

---

## 14.2 E2E 검증의 의미

```text
사용자
→ Vercel 프론트엔드
→ Railway FastAPI 백엔드
→ SQLite DB
→ Railway 응답
→ Vercel 화면 업데이트
→ 사용자 확인
```

---

# 15. 사용자 시나리오 체크리스트

## 15.1 게시글 작성 — POST

```text
1. https://your-project.vercel.app/posts/new 접속
2. 제목과 내용 입력
3. 작성하기 클릭
4. 목록 페이지로 자동 이동 확인
5. 방금 작성한 게시글이 목록에 나타나는지 확인
```

---

## 15.2 게시글 조회 — GET

```text
1. 목록에서 방금 작성한 게시글 클릭
2. 상세 페이지 화면 확인
3. Railway URL/docs 접속
4. GET /posts 실행
5. Swagger UI에서 사용자가 입력한 게시글이 있는지 확인
```

---

## 15.3 검색 — Query Param

```text
1. https://your-project.vercel.app/search 접속
2. 검색어 입력
3. URL이 /search?q=Next 형태로 변경되는지 확인
4. 새로고침해도 검색 결과가 유지되는지 확인
5. URL 기반 서버 필터링이 작동하는지 확인
```

---

## 15.4 게시글 수정 — PUT

```text
1. 게시글 상세 페이지 접속
2. 수정하기 클릭
3. 제목 또는 내용 변경 후 저장
4. 변경된 내용이 화면에 반영되는지 확인
```

---

## 15.5 게시글 삭제 — DELETE

```text
1. 삭제 버튼 클릭
2. 확인 창 승인
3. 목록 페이지로 리다이렉트 확인
4. 삭제한 게시글이 목록에서 사라졌는지 확인
```

---

# 16. Railway Swagger UI 확인

## 16.1 Swagger UI에서 DB 확인

Railway 도메인의 `/docs`로 접속하면 FastAPI Swagger UI를 확인할 수 있다.

```text
https://your-railway-app.up.railway.app/docs
```

Swagger UI에서 `GET /posts`를 실행해 Vercel에서 작성한 게시글이 실제 DB에 저장되었는지 확인한다.

---

## 16.2 배포 환경 구성

| 구성요소 | 배포 위치 | 역할 |
|---|---|---|
| Browser | 사용자 기기 | Vercel URL 접속 |
| Next.js | Vercel | 화면 렌더링, 요청 전송 |
| FastAPI | Railway | API 처리 |
| SQLite | Railway 서버 파일 | 데이터 저장 |

---

# 17. Vercel + Railway vs Docker + AWS

## 17.1 배포 방식 비교

| 구분 | Vercel + Railway | Docker + AWS |
|---|---|---|
| 난이도 | 낮음, 클릭 중심 | 높음, Linux/Docker/AWS 필요 |
| 설정 시간 | 10~15분 | 1~3시간 |
| 컨트롤 | 제한적, 플랫폼 규칙 따름 | 거의 완전 제어 |
| 비용 | 무료 플랜 존재, 트래픽 제한 | 인프라 비용 발생 |
| 스케일링 | 자동, Vercel Edge Network | 직접 구성, ECS/ALB/ASG 등 |
| DB | SQLite 파일 유지 가능, 실습용 | RDS/PostgreSQL 주로 사용 |
| 모니터링 | Vercel Analytics | CloudWatch, Prometheus 등 |
| 적합한 상황 | 사이드 프로젝트, 빠른 MVP | 팀 시스템, 고트래픽 실제 서비스 |

---

## 17.2 Vercel의 한계

Vercel은 편리하지만 모든 상황에 적합하지는 않다.

| 한계 | 설명 |
|---|---|
| 실행 시간 제한 | Serverless Function은 시간 제한이 있어 오래 걸리는 작업에 부적합 |
| 실행 환경 제한 | Node.js, Python 지원이 제한적일 수 있음 |
| 인프라 접근 불가 | 자체 서버 수준의 세밀한 성능 튜닝 어려움 |
| 실시간 기능 한계 | WebSocket, 장시간 연결, 스트리밍 처리에 제약 가능 |
| 파일 기반 DB 한계 | Serverless 환경에서 SQLite 파일 유지가 어려움 |

---

## 17.3 Docker + AWS가 필요한 상황

다음과 같은 상황에서는 Docker와 AWS 같은 직접 인프라 구성이 필요해질 수 있다.

```text
- 매일 1만 명 이상 사용하는 B2B SaaS
- 기업 보안 정책 기준을 만족해야 하는 서비스
- 감사 로그가 필요한 시스템
- ML 모델 서빙
- WebSocket 기반 실시간 기능
- PostgreSQL 수백 GB 이상 데이터
- 세밀한 서버 성능 튜닝이 필요한 경우
```

---

# 18. SQLite의 한계

## 18.1 SQLite의 특징

SQLite는 파일 기반 데이터베이스이다.  
이번 실습에서는 Railway 서버의 파일 시스템에 `blog.db` 파일을 저장하는 방식으로 사용했다.

---

## 18.2 SQLite의 한계

| 한계 | 설명 |
|---|---|
| 파일 공유 문제 | 여러 서버 인스턴스가 같은 DB 파일을 공유하기 어려움 |
| 쓰기 충돌 | 트래픽이 많으면 동시에 쓰기 작업이 발생해 충돌 가능 |
| 확장성 제한 | 고트래픽 서비스나 대용량 데이터에 부적합 |
| 운영 안정성 | 프로덕션에서는 별도 DB 서버가 더 적절 |

---

## 18.3 프로덕션 DB

실제 서비스에서는 보통 네트워크로 접근하는 DB를 사용한다.

```text
PostgreSQL
MySQL
MariaDB
```

AWS 환경에서는 RDS에 PostgreSQL이나 MySQL을 올려 사용하는 경우가 많다.

---

# 19. 그동안 배운 내용 정리

## 19.1 주차별 흐름

```text
1주차~3주차
→ Python, FastAPI, SQLite, SQLAlchemy로 백엔드 API 구축

4주차
→ Next.js로 프론트엔드 구축 + 풀스택 통합

4주 5일차
→ Vercel + Railway로 실제 배포
```

---

## 19.2 다음 학습 방향

다음 단계에서는 동일한 앱을 Docker로 컨테이너화하고 AWS EC2에 직접 배포하는 흐름을 학습한다.

| 항목 | Vercel + Railway | Docker + AWS |
|---|---|---|
| 인프라 | 플랫폼이 관리 | 직접 EC2 서버 호스팅 |
| 컨테이너 | 없음 | Docker로 앱 패키징 |
| 배포 방식 | Git Push → 자동 배포 | 이미지 빌드 → 서버에서 실행 |
| 학습 목표 | 클라우드 플랫폼 활용법 이해 | DevOps 기본 실무 능력 확보 |

---

# 20. 전체 핵심 요약

## 20.1 Day 5-1 핵심

| 개념 | 핵심 |
|---|---|
| Direct Fetch | 브라우저가 FastAPI를 직접 호출 |
| Route Handler | Next.js 서버가 FastAPI를 대신 호출 |
| NEXT_PUBLIC_ | 브라우저 코드에 노출할 환경변수 접두사 |
| CORS | 브라우저가 다른 Origin으로 직접 요청할 때 필요 |
| axios | HTTP 요청을 더 간결하고 안정적으로 처리하는 라이브러리 |
| axios.isAxiosError | axios 에러를 안전하게 구분하는 함수 |

---

## 20.2 Day 5-2 핵심

| 개념 | 핵심 |
|---|---|
| Vercel | Next.js 프론트엔드 배포 플랫폼 |
| Railway | FastAPI 백엔드 배포 플랫폼 |
| GitHub | 배포 소스 저장소 |
| FASTAPI_URL | Vercel에서 Railway 백엔드 URL을 가리키는 서버 환경변수 |
| CORS 설정 | Vercel 도메인을 Railway 백엔드에서 허용 |
| 배포 확인 | Vercel 앱, Railway `/docs`, CRUD, 검색 기능 확인 |

---

## 20.3 Day 5-3 핵심

| 개념 | 핵심 |
|---|---|
| E2E 검증 | 사용자 흐름 전체가 배포 환경에서 정상 동작하는지 확인 |
| POST 검증 | 게시글 작성 후 목록 반영 확인 |
| GET 검증 | 목록/상세 조회와 Swagger UI 확인 |
| Query Param 검색 | URL이 검색 상태를 유지하는지 확인 |
| PUT 검증 | 게시글 수정 반영 확인 |
| DELETE 검증 | 삭제 후 목록에서 사라지는지 확인 |
| Docker/AWS | 더 높은 제어권이 필요한 운영 환경에 적합 |
| SQLite 한계 | 파일 기반 DB라 확장성과 동시 쓰기에 약함 |

---

# 21. 최종 정리

Day 5에서는 로컬에서 구현했던 Next.js + FastAPI 풀스택 앱을 실제 배포와 검증 흐름까지 확장했다.

먼저 검색 기능을 복습했다.  
Direct Fetch는 브라우저가 FastAPI를 직접 호출하는 방식으로 단순하지만, 백엔드 URL이 노출되고 CORS 설정이 필요하다.  
반면 Route Handler 방식은 브라우저가 Next.js의 `/api/search`만 호출하고, Next.js 서버가 FastAPI를 대신 호출한다.  
이 방식은 백엔드 URL을 숨길 수 있고, 서버 간 통신이므로 CORS 문제를 줄일 수 있다.

그다음 기존 `fetch` 기반 코드를 `axios`로 리팩토링했다.  
axios는 `response.data`로 자동 JSON 변환을 제공하고, 2xx가 아닌 응답을 자동으로 `catch`로 보내며, `axios.isAxiosError()`를 통해 서버가 보낸 에러 메시지도 쉽게 꺼낼 수 있다.  
검색 페이지뿐 아니라 삭제 버튼처럼 클라이언트에서 HTTP 요청을 보내는 컴포넌트도 axios로 바꿀 수 있다.

배포 단계에서는 프론트엔드는 Vercel에, 백엔드는 Railway에 배포한다.  
Vercel은 Next.js 배포에 최적화된 플랫폼이고, Railway는 FastAPI처럼 계속 실행되어야 하는 백엔드 서버를 컨테이너 기반으로 배포하기 적합하다.  
배포 후 Vercel에는 `FASTAPI_URL` 환경변수로 Railway 백엔드 URL을 등록하고, Railway 백엔드에는 Vercel 도메인을 CORS 허용 Origin으로 추가해야 한다.

배포가 끝나면 E2E 검증을 수행한다.  
사용자가 실제로 하는 흐름대로 게시글 작성, 조회, 검색, 수정, 삭제를 하나씩 확인한다.  
특히 검색 기능은 URL이 `/search?q=검색어` 형태로 바뀌는지, 새로고침 후에도 검색 결과가 유지되는지 확인해야 한다.  
Railway의 `/docs` Swagger UI를 통해 FastAPI와 SQLite DB에 실제 데이터가 저장되었는지도 확인한다.

마지막으로 Vercel + Railway와 Docker + AWS의 차이를 비교했다.  
Vercel + Railway는 빠르게 배포하고 MVP나 사이드 프로젝트를 만들기에 좋지만, 실행 시간, 인프라 제어, 대규모 트래픽, 실시간 기능, 파일 기반 DB 유지 측면에서 한계가 있다.  
고트래픽 서비스, 기업 보안 요구, ML 모델 서빙, WebSocket, 대용량 DB가 필요한 경우에는 Docker로 앱을 컨테이너화하고 AWS EC2, RDS, ECS 등으로 직접 운영하는 방식이 더 적합하다.

결국 이 파트의 핵심은 다음과 같다.

```text
1. Direct Fetch는 단순하지만 백엔드 주소 노출과 CORS 문제가 있다.
2. Route Handler는 Next.js 서버를 프록시처럼 사용해 보안성과 구조를 개선한다.
3. axios는 클라이언트 HTTP 요청 코드를 간결하고 안정적으로 만든다.
4. Next.js 프론트엔드는 Vercel에, FastAPI 백엔드는 Railway에 배포한다.
5. 배포 후에는 환경변수와 CORS 설정을 반드시 확인해야 한다.
6. E2E 검증은 실제 사용자 흐름으로 앱 전체가 동작하는지 확인하는 과정이다.
7. Vercel + Railway는 빠른 배포에 적합하고, Docker + AWS는 높은 제어권과 운영 확장성이 필요한 경우에 적합하다.
```

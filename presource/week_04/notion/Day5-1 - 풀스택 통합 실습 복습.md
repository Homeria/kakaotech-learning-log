## 실습 1 — Fetch 기반 검색 기능 구현

<aside>
👉🏻

**클라이언트 컴포넌트 안에서 검색어 입력 시, 실시간으로 글 목록이 필터링되는 화면 구현(전체 데이터 조회 → 검색어 기반 필터링)**

1. **Direct Fetch**: 브라우저 → FastAPI(`NEXT_PUBLIC_FASTAPI_URL/posts`) 직접 호출
    - 단순하고 직관적인 구조
    - 브라우저 네트워크 탭에 백엔드 주소가 노출되며, 브라우저가 직접 호출하므로 FastAPI 측에 CORS 허용 세팅 필요
    - NEXT_PUBLIC_ 접두사의 역할과 보안
        
        Next.js는 데이터 노출(보안 사고)을 막기 위해 환경변수를 기본적으로 **서버에서만** 읽을 수 있게 제한합니다.
        
        하지만 **Client Component**는 브라우저에서 직접 실행되므로, 브라우저가 직접 FastAPI 주소로 요청을 보내려면 주소 값을 알고 있어야 합니다.
        
        이때 환경변수 이름 앞에 **`NEXT_PUBLIC_`**을 붙여주면, Next.js가 해당 변수를 브라우저(클라이언트) 코드에도 안전하게 노출해 줍니다.
        
        - FASTAPI_URL 👉 **서버 전용**
        - NEXT_PUBLIC_FASTAPI_URL 👉 **브라우저 공개용**
            - Client Component에서도 정상적으로 주소를 읽을 수 있습니다.
            - 공개해도 무방한 정보에만 활용
2. **Route Handler**: 브라우저 → `/api/search` (Route Handler) → FastAPI
    - 브라우저는 우리 서버(/api/search)를 호출하고, Next.js 서버가 백엔드를 대신 호출
        - Next.js 앱 내부의 `app/api/search/route.ts` 경로에 **GET** 요청을 처리하는 API 서버를 구축
        - 이 서버는 브라우저 대신 외부 백엔드(FastAPI) 서버와 통신하는 **프록시(Proxy, 대리인)** 역할을 수행
    - 브라우저에 백엔드 주소 노출 x, CORS 설정 x
</aside>

```tsx
cd fullstack-practice/backend
uv run fastapi dev main.py
```

```tsx
cd fullstack-practice/frontend
npm install
npm run dev
```

**`frontend/.env.local`**

```python
FASTAPI_URL='http://localhost:8000'
```

---

**`search/page.tsx` — 빈칸 코드 (Direct Fetch + Route Handler)**

- 힌트
    
    ### 비동기 Fetch 체이닝 예시
    
    React의 useEffect 안에서 데이터를 안전하게 가져오고 상태(State)에 저장할 때 사용하는 패턴
    
    ```tsx
    fetch("요청할_주소")
      .then((response) => {
        // 1단계: 요청이 성공했는지 검사하고 JSON 변환 준비
        if (!response.ok) throw new Error("에러 발생!");
        return response.json(); 
      })
      .then((data) => {
        // 2단계: 변환된 진짜 데이터를 상태(state)에 저장
        setResults(data); 
      })
      .catch((error) => {
        // 3단계: 에러 발생 시 처리
        setError(error.message); 
      })
      .finally(() => {
        // 4단계: 성공/실패 여부와 상관없이 로딩 완료 처리
        setLoading(false); 
      });
    ```
    

```tsx
// app/search/page.tsx — 검색 페이지 (Client Component)
"use client";

import { useState, useEffect } from "react";
import Link from "next/link";

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

  // ===========================================================================
  // [실습 1] Direct Fetch 방식
  //   흐름: 브라우저 → FastAPI (NEXT_PUBLIC_FASTAPI_URL/posts) 직접 호출
  //   TODO: 아래 useEffect 블록을 완성해보세요. 완성 후 Route Handler 방식은 주석 처리하세요.
  // ===========================================================================
  /*
    useEffect(() => {
      setLoading(true);
      setError(null);

      // TODO: process.env.NEXT_PUBLIC_FASTAPI_URL 을 사용해 /posts 를 fetch 하세요.
      //       성공 시 setResults, 실패 시 setError, 완료 시 setLoading(false) 처리.

    }, []);
  */

  // ===========================================================================
  // [실습 1] Route Handler 방식
  //   흐름: 브라우저 → /api/search (Route Handler) → FastAPI
  //   TODO: 아래 useEffect 블록을 완성해보세요.
  // ===========================================================================
  useEffect(() => {
    setLoading(true);
    setError(null);

    // TODO: `${BASE_PATH}/api/search` 를 fetch 하세요.
    //       성공 시 setResults, 실패 시 setError, 완료 시 setLoading(false) 처리.

  }, []);

  // ===========================================================================
  // TODO: results 배열을 query 로 필터링하는 로직을 구현해보세요.
  //       post.title 또는 post.content 에 query 가 포함된 게시글만 남기세요.
  // ===========================================================================
  const filtered: Post[] = results;

  return (
    <main>
      <div className="flex items-center gap-3 mb-6">
        <Link href={`${BASE_PATH}/posts`} className="text-gray-400 hover:text-gray-600 text-sm">
          ← 목록으로
        </Link>
        <h1 className="text-2xl font-bold text-gray-900">검색</h1>
      </div>

      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="제목 또는 내용으로 검색하세요"
        className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm mb-6 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
      />

      {loading && <p className="text-center text-gray-400 py-10">불러오는 중...</p>}
      {error && <p className="text-center text-red-500 py-10">{error}</p>}
      {!loading && !error && filtered.length === 0 && (
        <p className="text-center text-gray-400 py-10">
          {query ? "검색 결과가 없습니다." : "게시글이 없습니다."}
        </p>
      )}
      {!loading && !error && filtered.length > 0 && (
        <ul className="space-y-3">
          {filtered.map((post) => (
            <li key={post.id}>
              <Link
                href={`${BASE_PATH}/posts/${post.id}`}
                className="block bg-white border border-gray-200 rounded-xl p-5 hover:border-blue-400 hover:shadow-sm transition-all"
              >
                <p className="font-medium text-gray-900">{post.title}</p>
                <p className="text-sm text-gray-500 mt-1 line-clamp-2">{post.content}</p>
                <p className="text-xs text-gray-400 mt-2">
                  {new Date(post.created_at).toLocaleDateString("ko-KR")}
                </p>
              </Link>
            </li>
          ))}
        </ul>
      )}
    </main>
  );
}
```

**`app/api/search/route.ts` — 빈칸 코드**

- 힌트
    
    ### Route Handler 예시
    
    ```tsx
    import { NextResponse } from "next/server";
    
    // HTTP GET 요청이 들어오면 실행되는 함수
    export async function GET() {
      // 1. 서버 전용 환경 변수(비밀 주소)를 읽어옵니다.
      const targetUrl = process.env.TARGET_URL_KEY;
    
      // 2. [예외 처리] 환경 변수가 설정되지 않았다면 Next.js 에러(500)를 반환합니다.
      if (!targetUrl) {
        return NextResponse.json(
          { detail: "환경 변수가 설정되지 않았습니다." },
          { status: 500 }
        );
      }
    
      // 3. 외부 API 서버를 호출합니다.
      const res = await fetch(`${targetUrl}/endpoint-path`);
    
      // 4. [예외 처리] 외부 서버 호출이 실패했다면 에러를 반환합니다.
      if (!res.ok) {
        return NextResponse.json(
          { detail: "데이터를 가져오는 데 실패했습니다." },
          { status: res.status }
        );
      }
    
      // 5. 성공 시 데이터를 JSON으로 변환하여 클라이언트에게 최종 전달합니다.
      const data = await res.json();
      return NextResponse.json(data);
    }
    ```
    

```tsx
// app/api/search/route.ts — 검색용 Route Handler
//
// [실습 1] Route Handler 방식
//   흐름: 브라우저 → GET /api/search (이 파일) → FastAPI
//
//   브라우저가 /api/search 로 요청하면 이 서버 사이드 핸들러가
//   FastAPI 를 대신 호출하고 결과를 브라우저에 반환합니다.
//   → 브라우저 입장에서는 같은 서버(Next.js)와만 통신하므로 CORS 불필요

import { NextResponse } from "next/server";

export async function GET() {
  // TODO: 아래 단계를 구현해보세요.
  //
  //   1. process.env.FASTAPI_URL 로 환경 변수를 읽어오세요.
  //      (서버에서 실행되므로 NEXT_PUBLIC_ 접두사 불필요)
  //
  //   2. FASTAPI_URL 이 없다면 500 에러를 반환하세요.
  //
  //   3. `${fastapiUrl}/posts` 를 fetch 하여 게시글 목록을 가져오세요.
  //
  //   4. FastAPI 응답이 실패라면 동일한 상태 코드로 에러를 반환하세요.
  //
  //   5. 성공 시 FastAPI 응답 데이터를 NextResponse.json() 으로 반환하세요.
}
```

- 정답
    
    **`frontend/app/search/page.tsx` — Direct Fetch 정답**
    
    ```tsx
    // app/search/page.tsx — 검색 페이지 (Client Component)
    "use client";
    
    import { useState, useEffect } from "react";
    import Link from "next/link";
    
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
    
      // BASE_PATH: 런박스 환경에서 Client Component가 Route Handler 경로를 올바르게 구성하기 위해 필요합니다. 
      // .env.local 파일에 NEXT_PUBLIC_BASE_PATH='/proxy/3000'이 포함되어 있는지 확인해주세요.
      const BASE_PATH = process.env.NEXT_PUBLIC_BASE_PATH ?? "";
    
      // ===========================================================================
      // [실습 1] Direct Fetch 방식
      //   흐름: 브라우저 → FastAPI (NEXT_PUBLIC_FASTAPI_URL/posts) 직접 호출
      //   TODO: 아래 useEffect 블록을 완성해보세요.완성 후 Route Handler 방식(아래)은 주석 처리하세요.
      // ===========================================================================
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
    
      // ===========================================================================
      // TODO: results 배열을 query 로 필터링하는 로직을 구현해보세요.
      //       post.title 또는 post.content 에 query 가 포함된 게시글만 남기세요.
      // ===========================================================================
      const filtered: Post[] = results.filter(
        (post) =>
          post.title.includes(query) || post.content.includes(query)
      );
    ```
    
    **`frontend/app/search/page.tsx` — Route Handler 정답**
    
    ```tsx
    // app/search/page.tsx — 검색 페이지 (Client Component)
    "use client";
    
    import { useState, useEffect } from "react";
    import Link from "next/link";
    
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
    
      // BASE_PATH: 런박스 환경에서 Client Component가 Route Handler 경로를 올바르게 구성하기 위해 필요합니다. 
      // .env.local 파일에 NEXT_PUBLIC_BASE_PATH='/proxy/3000'이 포함되어 있는지 확인해주세요.
      const BASE_PATH = process.env.NEXT_PUBLIC_BASE_PATH ?? "";
    
      // ===========================================================================
      // [실습 1] Route Handler 방식
      //   흐름: 브라우저 → /api/search (Route Handler) → FastAPI
      //   TODO: 아래 useEffect 블록을 완성해보세요.
      // ===========================================================================
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
    
      // ===========================================================================
      // TODO: results 배열을 query 로 필터링하는 로직을 구현해보세요.
      //       post.title 또는 post.content 에 query 가 포함된 게시글만 남기세요.
      // ===========================================================================
      const filtered: Post[] = results.filter(
        (post) =>
          post.title.includes(query) || post.content.includes(query)
      );
    ```
    
    | **구분** | **Direct Fetch** | **Route Handler** |
    | --- | --- | --- |
    | **호출하는 주소** | `NEXT_PUBLIC_FASTAPI_URL` (예: `https://api.example.com/posts`) | `/api/search` (Next.js 백엔드 주소) |
    | **통신 흐름** | 브라우저(Client) → **FastAPI(외부 서버)** | 브라우저 → **Next.js 서버(Route Handler)** → **FastAPI** |
    | **환경 변수 노출** | `NEXT_PUBLIC_` 접두사 필요 (브라우저에 노출됨) | 브라우저에 노출할 필요 없음 |
    | **CORS 문제** | FastAPI 서버에서 브라우저 접근을 허용(CORS 설정)해야 함 | Next.js 서버 내부에서 호출하므로 **CORS 에러가 발생하지 않음** |
    
    **`frontend/app/api/search/route.ts`**
    
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

## 실습 2 — axios로 리팩토링

<aside>
👀

### Axios 란?

: 자바스크립트(JavaScript) 환경에서 서버와 HTTP 비동기 통신을 주고받기 위해 가장 널리 사용되는 오픈소스 라이브러리

기본 브라우저 내장 기능인 fetch가 있음에도 불구하고 Axios를 설치해 사용하는 이유는 뭘까요?

1. **자동 JSON 변환:** fetch를 쓸 때는 귀찮게 res.json()을 한 번 더 호출해야 했지만, Axios는 서버가 준 데이터를 알아서 객체로 변환하여 `response.data`에 넣어줍니다.
2. **직관적인 에러 처리:** fetch는 서버가 에러 상태 코드(404, 500 등)를 반환해도 에러를 발생시키지 않아 일일이 if (!res.ok)로 체크해야 했습니다. 반면 **Axios는 2xx 범위가 아닌 모든 응답에 대해 자동으로 에러(throw)를 발생**시켜 줍니다.
3. **구조화된 에러 객체:** 백엔드가 에러 이유를 담아 보낸 세부 메시지를 axios.isAxiosError(err)를 통해 매우 손쉽게 꺼내 쓸 수 있습니다.
4. **다양한 기능:** 요청 및 응답 가로채기(Interceptor), 요청 취소, 타임아웃 설정, 자동 에러 처리 등의 고급 기능을 제공
</aside>

실습 1에서 활용한 `fetch`를 `axios`로 리팩토링합니다. `search/page.tsx`와 `posts/[postId]/DeleteButton.tsx`에서 사용되는 `fetch`를 `axios`로 교체합니다.

**1. axios 설치**

```bash
cd fullstack-practice/frontend
npm install axios
```

---

**`search/page.tsx` — 빈칸 코드 (fetch → axios 교체)**

- 힌트
    
    ```tsx
    import axios from "axios";
    
    async function getData() {
      try {
        const response = await axios.get<DataType[]>("요청할_주소");
        
        console.log(response.data); 
      } catch (err) {
        // 에러 발생 시 Axios 에러인지 일반 에러인지 안전하게 분류합니다.
        if (axios.isAxiosError(err)) {
          // 서버가 보낸 세부 메시지(detail)를 꺼내 씁니다.
          console.error(err.response?.data?.detail); 
        } else {
          console.error("알 수 없는 오류 발생");
        }
      }
    }
    ```
    

```tsx
// app/search/page.tsx — 검색 페이지 (Client Component)
"use client";

import { useState, useEffect } from "react";
import Link from "next/link";
// TODO: axios 를 import 하세요. (npm install axios 먼저 실행)
// import axios from "axios";

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

  // ===========================================================================
  // [실습 1 완성] fetch 기반 Route Handler 방식
  //   흐름: 브라우저 → /api/search (Route Handler) → FastAPI
  // ===========================================================================
  //
  // [실습 2] TODO: 아래 fetch 구현을 axios 로 리팩토링해보세요.
  //
  //   1. useEffect 내부에 async function fetchPosts() {} 를 선언하세요.
  //   2. try / catch / finally 구조로 에러 처리를 구현하세요.
  //   3. axios.get<Post[]>(`${BASE_PATH}/api/search`) 로 데이터를 가져오세요.
  //      → 성공 시: setResults(res.data)
  //   4. catch 블록에서 axios.isAxiosError(err) 로 HTTP 에러를 구분하세요.
  //      → AxiosError 라면: setError(err.response?.data?.detail ?? "...")
  //      → 그 외라면:      setError("알 수 없는 오류가 발생했습니다")
  //   5. finally 블록에서 setLoading(false) 를 호출하세요.
  //   6. 완성 후 아래 fetch 블록은 주석 처리하세요.
  // ===========================================================================
  useEffect(() => {
    setLoading(true);
    setError(null);

    fetch(`${BASE_PATH}/api/search`)
      .then((res) => {
        if (!res.ok) throw new Error("게시글을 불러오는 데 실패했습니다");
        return res.json();
      })
      .then((data: Post[]) => setResults(data))
      .catch((err: Error) => setError(err.message))
      .finally(() => setLoading(false));
  }, []);

  const filtered = results.filter(
    (post) => post.title.includes(query) || post.content.includes(query)
  );

  return (
    <main>
      <div className="flex items-center gap-3 mb-6">
        <Link href={`${BASE_PATH}/posts`} className="text-gray-400 hover:text-gray-600 text-sm">
          ← 목록으로
        </Link>
        <h1 className="text-2xl font-bold text-gray-900">검색</h1>
      </div>

      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="제목 또는 내용으로 검색하세요"
        className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm mb-6 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
      />

      {loading && (
        <p className="text-center text-gray-400 py-10">불러오는 중...</p>
      )}

      {error && (
        <p className="text-center text-red-500 py-10">{error}</p>
      )}

      {!loading && !error && filtered.length === 0 && (
        <p className="text-center text-gray-400 py-10">
          {query ? "검색 결과가 없습니다." : "게시글이 없습니다."}
        </p>
      )}

      {!loading && !error && filtered.length > 0 && (
        <ul className="space-y-3">
          {filtered.map((post) => (
            <li key={post.id}>
              <Link
                href={`${BASE_PATH}/posts/${post.id}`}
                className="block bg-white border border-gray-200 rounded-xl p-5 hover:border-blue-400 hover:shadow-sm transition-all"
              >
                <p className="font-medium text-gray-900">{post.title}</p>
                <p className="text-sm text-gray-500 mt-1 line-clamp-2">
                  {post.content}
                </p>
                <p className="text-xs text-gray-400 mt-2">
                  {new Date(post.created_at).toLocaleDateString("ko-KR")}
                </p>
              </Link>
            </li>
          ))}
        </ul>
      )}
    </main>
  );
}

```

---

**`DeleteButton.tsx` — 빈칸 코드 (fetch → axios 교체)**

- 힌트
    
    ```tsx
    async function deleteData(id: number) {
      try {
        // 삭제 요청은 바디 필요 x
        await axios.delete(`요청할_주소/${id}`);
        console.log("삭제 성공");
      } catch (err) {
        if (axios.isAxiosError(err)) {
          alert(err.response?.data?.detail);
        }
      }
    }
    ```
    

```tsx
"use client";

// TODO: axios 를 import 하세요.
// import axios from "axios";

export default function DeleteButton({ postId }: { postId: number }) {
  const BASE_PATH = process.env.NEXT_PUBLIC_BASE_PATH ?? "";

  // ===========================================================================
  // [실습 1 완성] fetch 기반 삭제 요청
  // ===========================================================================
  //
  // [실습 2] TODO: 아래 fetch 구현을 axios 로 리팩토링해보세요.
  //
  //   1. try / catch 구조로 교체하세요.
  //   2. axios.delete(`${BASE_PATH}/api/posts/${postId}`) 로 요청하세요.
  //   3. catch 블록에서 axios.isAxiosError(err) 로 에러를 구분하세요.
  //      → AxiosError 라면: alert(err.response?.data?.detail ?? "...")
  //   4. 완성 후 아래 fetch 블록은 주석 처리하세요.
  // ===========================================================================
  async function handleDelete() {
    if (!confirm("정말 삭제할까요?")) return;

    const res = await fetch(`${BASE_PATH}/api/posts/${postId}`, {
      method: "DELETE",
    });

    if (res.ok) {
      window.location.href = `${BASE_PATH}/posts`;
    }
  }

  return (
    <button
      onClick={handleDelete}
      className="bg-red-500 text-white text-sm px-4 py-2 rounded-lg hover:bg-red-600 transition-colors"
    >
      삭제하기
    </button>
  );
}

```

- 정답
    
    ### 📊 fetch vs axios 리팩토링 비교 분석표
    
    | **비교 항목** | **기존 fetch 방식** | **변경된 axios 방식** | **리팩토링 효과 및 이점** |
    | --- | --- | --- | --- |
    | **JSON 데이터 변환** | const res = await fetch(...)
    
    const data = await res.json() | const res = await axios.get(...)
    
    const data = res.data | **[코드 간결화]**
    불필요한 .json() 호출 단계가 생략되어 코드 한 줄이 줄어들고 데이터 접근이 직관적으로 변합니다. |
    | **HTTP 에러 감지** | if (!res.ok) { throw new Error(...) }
    *(404, 500 에러도 성공으로 인식하므로 수동 예외 처리 필수)* | 2xx 범위를 벗어난 응답 코드는 **자동으로 에러를 발생**시켜 즉시 catch 블록으로 이동 | **[안정성 향상]**
    수동으로 에러 체크를 누락하여 발생할 수 있는 버그를 원천 차단하고, 성공과 실패 흐름이 명확히 양분됩니다. |
    | **서버 에러 메시지(FastAPI detail)** | 실패 응답 바디를 다시 읽기 위해 내부에서 await res.json()을 또 실행해 꺼내야 함 | **err.response.data.detail**
    로 즉시 접근 가능 | **[디버깅 편의성]**
    FastAPI 백엔드가 보낸 세부 에러 메시지(detail)를 비동기 작업 없이 즉시 추출해 경고창이나 상태에 반영할 수 있습니다. |
    | **제공하는 메서드** | method: "DELETE", method: "POST" 등 옵션 객체에 문자열로 직접 지정 | axios.post(), axios.delete() 등 **전용 메서드** 제공 | **[타입 안정성]**
    메서드명을 오타 내서 발생하는 실수를 방지하고 자동완성 지원을 받을 수 있습니다. |
    
    **`frontend/app/search/page.tsx` 정답**
    
    ```tsx
    // app/search/page.tsx — 검색 페이지 (Client Component)
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
    
      // ===========================================================================
      // [실습 2 완성] axios 기반 Route Handler 방식
      //   흐름: 브라우저 → /api/search (Route Handler) → FastAPI
      // ===========================================================================
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
    
      // ===========================================================================
      // [실습 1 완성] fetch 기반 Route Handler 방식 (주석 처리)
      // ===========================================================================
      // useEffect(() => {
      //   setLoading(true);
      //   setError(null);
      //
      //   fetch(`${BASE_PATH}/api/search`)
      //     .then((res) => {
      //       if (!res.ok) throw new Error("게시글을 불러오는 데 실패했습니다");
      //       return res.json();
      //     })
      //     .then((data: Post[]) => setResults(data))
      //     .catch((err: Error) => setError(err.message))
      //     .finally(() => setLoading(false));
      // }, []);
    
      const filtered = results.filter(
        (post) => post.title.includes(query) || post.content.includes(query)
      );
    
      return (
        <main>
          <div className="flex items-center gap-3 mb-6">
            <Link href={`${BASE_PATH}/posts`} className="text-gray-400 hover:text-gray-600 text-sm">
              ← 목록으로
            </Link>
            <h1 className="text-2xl font-bold text-gray-900">검색</h1>
          </div>
    
          <input
            type="text"
            value={query}
            onChange={(e) => setQuery(e.target.value)}
            placeholder="제목 또는 내용으로 검색하세요"
            className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm mb-6 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          />
    
          {loading && (
            <p className="text-center text-gray-400 py-10">불러오는 중...</p>
          )}
    
          {error && (
            <p className="text-center text-red-500 py-10">{error}</p>
          )}
    
          {!loading && !error && filtered.length === 0 && (
            <p className="text-center text-gray-400 py-10">
              {query ? "검색 결과가 없습니다." : "게시글이 없습니다."}
            </p>
          )}
    
          {!loading && !error && filtered.length > 0 && (
            <ul className="space-y-3">
              {filtered.map((post) => (
                <li key={post.id}>
                  <Link
                    href={`${BASE_PATH}/posts/${post.id}`}
                    className="block bg-white border border-gray-200 rounded-xl p-5 hover:border-blue-400 hover:shadow-sm transition-all"
                  >
                    <p className="font-medium text-gray-900">{post.title}</p>
                    <p className="text-sm text-gray-500 mt-1 line-clamp-2">
                      {post.content}
                    </p>
                    <p className="text-xs text-gray-400 mt-2">
                      {new Date(post.created_at).toLocaleDateString("ko-KR")}
                    </p>
                  </Link>
                </li>
              ))}
            </ul>
          )}
        </main>
      );
    }
    ```
    
    **`frontend/app/posts/[postId]/DeleteButton.tsx` 정답**
    
    ```tsx
    "use client";
    
    import axios from "axios";
    
    export default function DeleteButton({ postId }: { postId: number }) {
      const BASE_PATH = process.env.NEXT_PUBLIC_BASE_PATH ?? "";
    
      // ===========================================================================
      // [실습 2 완성] axios 기반 삭제 요청
      // ===========================================================================
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
    
      // ===========================================================================
      // [실습 1 완성] fetch 기반 삭제 요청 (주석 처리)
      // ===========================================================================
      // async function handleDelete() {
      //   if (!confirm("정말 삭제할까요?")) return;
      //
      //   const res = await fetch(`${BASE_PATH}/api/posts/${postId}`, {
      //     method: "DELETE",
      //   });
      //
      //   if (res.ok) {
      //     window.location.href = `${BASE_PATH}/posts`;
      //   }
      // }
    
      return (
        <button
          onClick={handleDelete}
          className="bg-red-500 text-white text-sm px-4 py-2 rounded-lg hover:bg-red-600 transition-colors"
        >
          삭제하기
        </button>
      );
    }
    ```
    

---

<aside>
✏️

## 오늘 배울 것

어제까지 로컬에서만 작동하던 앱을 **Vercel에 배포**합니다.

배포가 되면 누구나 URL을 통해 접근할 수 있는 실제 서비스가 됩니다. 오늘은 로컬호스트 대신 여러분이 만든 URL로 검색을 할 수 있습니다.

</aside>
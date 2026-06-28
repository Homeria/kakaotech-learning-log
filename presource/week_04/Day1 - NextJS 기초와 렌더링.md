# Next.js 기초와 렌더링 정리

> 원본 PDF: `[수업 자료] Next.js 기초와 렌더링.pdf`  
> 주제: Next.js 개요, 프레임워크와 라이브러리, App Router 기반 Routing, Route Handler, Rendering, Server Component와 Client Component

---

# 1. Next.js 개요

## 1.1 Next.js란?

Next.js는 **React 기반의 프레임워크**이다.  
높은 퀄리티의 풀스택 웹 애플리케이션을 만들 수 있도록 라우팅, 렌더링, 데이터 패칭, 최적화, 캐싱 등 여러 기능을 제공한다.

```text
React 기반
+ Routing
+ Rendering
+ Data Fetching
+ Optimizing
+ Caching
= Next.js
```

---

## 1.2 Next.js를 배우는 이유

React는 웹 프론트엔드 개발에서 매우 널리 사용되는 기반 기술이 되었다.  
하지만 React만으로 실제 서비스를 만들려면 라우팅, 서버 렌더링, 데이터 요청, 성능 최적화, 캐싱 등을 직접 구성해야 한다.

Next.js는 이런 기능들을 프레임워크 차원에서 제공하여 React 애플리케이션을 더 체계적으로 개발할 수 있게 해준다.

---

# 2. 프레임워크와 라이브러리

## 2.1 프레임워크와 라이브러리를 사용하는 이유

현업에서는 대규모 프로그램을 개발해야 한다.  
모든 기능과 전체 흐름을 개발자가 한 줄씩 직접 구현하는 데는 한계가 있다.

이 문제를 해결하기 위해 개발자는 이미 만들어진 코드의 모음인 **프레임워크**와 **라이브러리**를 사용한다.

---

## 2.2 공통점

프레임워크와 라이브러리는 모두 이미 만들어진 코드의 모음이다.  
Node.js 환경에서는 둘 다 npm을 통해 설치할 수 있다.

```bash
npm i next
npm i react react-dom
```

| 구분 | 공통점 |
|---|---|
| 프레임워크 | 이미 만들어진 코드와 규칙을 제공 |
| 라이브러리 | 이미 만들어진 코드와 기능을 제공 |

---

## 2.3 핵심 차이: 코드 흐름 제어권

프레임워크와 라이브러리의 가장 큰 차이는 **코드 흐름 제어의 주도권**이다.

| 구분 | 코드 흐름 제어권 | 설명 |
|---|---|---|
| 라이브러리 | 개발자 | 개발자가 필요한 순간에 라이브러리의 함수나 클래스를 호출한다. |
| 프레임워크 | 프레임워크 | 프레임워크가 정한 규칙 안에 개발자가 코드를 작성하면 프레임워크가 실행 흐름을 관리한다. |

---

## 2.4 라이브러리

라이브러리는 코드의 흐름을 개발자가 주로 제어한다.  
라이브러리는 개발에 도움이 되는 함수, 클래스, 기능을 제공하고, 이를 언제 어떻게 사용할지는 개발자가 결정한다.

```text
개발자가 흐름 설계
→ 필요한 기능을 라이브러리에서 가져와 사용
→ 개발자가 실행 시점과 구조를 제어
```

---

## 2.5 프레임워크

프레임워크는 애플리케이션의 전체 흐름이 이미 정해져 있다.  
개발자는 프레임워크의 규칙에 맞게 파일과 코드를 작성하고, 프레임워크가 전체 실행 흐름을 관리한다.

Next.js에서는 `app/` 디렉토리 아래에 `page.tsx`, `layout.tsx`, `route.ts` 같은 약속된 파일을 작성하면 라우팅과 렌더링이 자동으로 연결된다.

```text
프레임워크가 흐름 제공
→ 개발자는 규칙에 맞게 파일 작성
→ 프레임워크가 실행과 연결을 처리
```

---

## 2.6 프레임워크와 라이브러리 비교

| 구분 | 라이브러리 | 프레임워크 |
|---|---|---|
| 흐름 제어 | 개발자가 제어 | 프레임워크가 제어 |
| 사용 방식 | 필요한 기능을 직접 호출 | 정해진 규칙에 맞춰 코드 작성 |
| 자유도 | 높음 | 상대적으로 낮음 |
| 구조 제공 | 적음 | 많음 |
| 예시 | React, Axios 등 | Next.js, Remix, Gatsby 등 |

---

# 3. React와 프레임워크

## 3.1 React 기반 프레임워크가 필요한 이유

React는 현재 웹 애플리케이션의 기본 기술처럼 사용되고 있다.  
React를 더 잘 활용하기 위해 React 기반 프레임워크를 사용하는 흐름이 생겼다.

즉, 과거에는 HTML, CSS, JavaScript를 더 잘 사용하기 위해 React를 사용했다면, 이제는 React를 더 잘 사용하기 위해 Next.js 같은 React 기반 프레임워크를 사용한다고 볼 수 있다.

---

## 3.2 React 기반 프레임워크 예시

| 프레임워크 | 설명 |
|---|---|
| Next.js | React 기반 풀스택 웹 프레임워크 |
| Remix | 웹 표준과 서버 중심 데이터 흐름을 강조하는 React 프레임워크 |
| Gatsby | 정적 사이트 생성에 강점을 가진 React 프레임워크 |

---

## 3.3 Next.js 버전 기준

자료에서는 **Next.js 14**를 기준으로 설명한다.  
다만 Next.js는 업데이트가 자주 이루어지므로, Next.js 15와 16에서 바뀐 부분도 함께 언급된다.

| 버전 | 기준 |
|---|---|
| Next.js 14 | 2023년 10월 27일 기준 |
| Next.js 15 | 2024년 10월 22일 기준 변경점 언급 |
| Next.js 16 | 2025년 10월 21일 기준 변경점 언급 |

---

# 4. Next.js 프로젝트 시작

## 4.1 create-next-app

React의 CRA처럼 Next.js도 프로젝트를 빠르게 시작할 수 있는 명령어를 제공한다.

```bash
npx create-next-app@14
```

명령어 실행 후 여러 설정을 선택하면 Next.js 프로젝트가 생성된다.

---

## 4.2 개발 서버 실행

프로젝트 생성 후 개발 서버는 다음 명령어로 실행한다.

```bash
npm run dev
```

개발 서버가 실행되면 로컬 주소로 접속하여 Next.js 앱을 확인할 수 있다.

---

## 4.3 Next.js 핵심 개념

| 핵심 개념 | 설명 |
|---|---|
| Routing과 Rendering | 파일 기반 라우팅과 서버 중심 렌더링 방식 |
| Data Fetching | 서버나 외부 API에서 데이터 가져오기 |
| Optimizing | 이미지, 폰트, 번들 등 성능 최적화 |
| Caching | 데이터와 페이지를 효율적으로 캐싱 |

---

# 5. Routing

## 5.1 기존 React의 Routing 방식

React는 라우팅 기능을 기본으로 내장하지 않는다.  
따라서 React에서 페이지 전환을 구현하려면 직접 구현하거나 `react-router-dom` 같은 외부 라이브러리를 사용해야 했다.

```bash
npm i react-router-dom
```

---

## 5.2 Next.js의 Routing 방식

Next.js는 라우팅 기능을 기본으로 제공한다.  
특히 App Router에서는 **파일 시스템 기반 라우팅**을 사용한다.

```text
파일/디렉토리 구조
→ Next.js가 자동으로 URL path 생성
→ 해당 page.tsx 렌더링
```

---

# 6. App Router의 Routing 규칙

## 6.1 규칙 1 — app 디렉토리 하위에 위치

Routing과 관련된 디렉토리와 파일은 모두 `app/` 디렉토리 하위에 위치한다.

```text
app/
├── page.tsx
├── about/
│   └── page.tsx
└── products/
    └── list/
        └── page.tsx
```

---

## 6.2 규칙 2 — 디렉토리 이름이 URL path가 된다

각 디렉토리의 이름은 URL path에 대응된다.  
최상위 `app/` 디렉토리는 `/`에 대응된다.

| 디렉토리 구조 | URL |
|---|---|
| `app/page.tsx` | `/` |
| `app/about/page.tsx` | `/about` |
| `app/about/contact/page.tsx` | `/about/contact` |
| `app/products/list/page.tsx` | `/products/list` |
| `app/api/products/route.ts` | `/api/products` |

---

## 6.3 규칙 3 — page.tsx 또는 route.ts가 있어야 의미 있는 경로가 된다

디렉토리가 URL path에 대응되더라도, 실제로 의미 있는 경로가 되려면 약속된 파일이 필요하다.

| 파일 | 역할 |
|---|---|
| `page.tsx` | 해당 경로에 접속했을 때 렌더링할 UI 정의 |
| `route.ts` | 해당 경로에 대한 백엔드 API 정의 |

`.js`, `.jsx` 파일도 가능하지만, TypeScript 기준으로는 보통 `.tsx`, `.ts`를 사용한다.

---

# 7. page.tsx

## 7.1 page.tsx의 역할

`page.tsx`는 해당 path에 렌더링될 컴포넌트를 정의하는 파일이다.  
기본 export로 컴포넌트를 내보내야 한다.

```tsx
export default function Page() {
  return <h1>About Page</h1>;
}
```

파일 위치:

```text
app/about/page.tsx
```

접속 경로:

```text
/about
```

---

## 7.2 page.tsx 예시

```tsx
// app/about/page.tsx
export default function Page() {
  return <h1>About Page</h1>;
}
```

```tsx
// app/about/contact/page.tsx
export default function Page() {
  return <h1>Contact Page</h1>;
}
```

```tsx
// app/products/list/page.tsx
export default function Page() {
  return <h1>Products List Page</h1>;
}
```

---

# 8. layout.tsx

## 8.1 layout.tsx의 역할

`layout.tsx`는 하위 라우트와 공유하는 UI를 작성하는 파일이다.  
예를 들어 모든 페이지에 공통으로 보이는 헤더, 네비게이션, 사이드바 등을 작성할 수 있다.

---

## 8.2 Root Layout

Next.js 애플리케이션에서 최상위에 위치한 Root Layout은 필수이다.  
Root Layout에는 반드시 `html`과 `body` 태그가 포함되어야 한다.

```tsx
export default function RootLayout({
  children,
}: Readonly<{ children: React.ReactNode }>) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

파일 위치:

```text
app/layout.tsx
```

---

## 8.3 children props

Layout의 `children` props에는 현재 path에 해당하는 `page.tsx`에서 내보낸 컴포넌트가 들어온다.

```tsx
export default function RootLayout({
  children,
}: Readonly<{ children: React.ReactNode }>) {
  return (
    <html lang="en">
      <body>
        <h1>Elice with Next.js!</h1>
        {children}
      </body>
    </html>
  );
}
```

```text
RootLayout
└── children = 현재 route의 page.tsx
```

---

# 9. Navigation과 Link

## 9.1 Link 컴포넌트

Next.js에서 페이지 간 이동을 할 때는 `next/link`에서 제공하는 `Link` 컴포넌트를 사용한다.

```tsx
import Link from "next/link";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <h1>Elice with Next.js!</h1>
        <ul>
          <li><Link href="/">홈</Link></li>
          <li><Link href="/about">소개</Link></li>
          <li><Link href="/about/contact">연락처</Link></li>
          <li><Link href="/products/list">제품 리스트</Link></li>
        </ul>
        {children}
      </body>
    </html>
  );
}
```

---

## 9.2 왜 a 태그 대신 Link를 사용하는가?

`<a>` 태그는 클릭 시 브라우저가 서버에 HTML을 새로 요청한다.  
즉, 전체 페이지가 다시 로드된다.

반면 Next.js의 `Link`는 클릭 동작을 가로채서 클라이언트 사이드 라우팅을 수행한다.

| 구분 | `<a href="/about">` | `<Link href="/about">` |
|---|---|---|
| 이동 방식 | Full Page Reload | Client-Side Navigation |
| 화면 전환 | 깜빡임 발생 가능 | 부드러운 전환 |
| 상태 유지 | useState 등 상태 초기화 | 필요한 경우 상태 유지 가능 |
| 로딩 방식 | HTML, JS, CSS 재요청 | 필요한 컴포넌트 중심 전환 |
| Prefetch | 없음 | 보이는 경로를 미리 로드 가능 |
| 사용 권장 | 외부 링크 또는 일반 HTML 이동 | Next.js 내부 페이지 이동 |

---

# 10. 중첩 Layout

## 10.1 중첩 Layout이란?

하위 라우트에 `layout.tsx`를 추가하면 중첩 Layout을 구현할 수 있다.

```text
app/
├── layout.tsx
└── about/
    ├── layout.tsx
    ├── page.tsx
    ├── contact/
    │   └── page.tsx
    └── history/
        └── page.tsx
```

---

## 10.2 중첩 Layout의 동작

하위 Layout은 상위 Layout의 `children`으로 들어간다.

```text
Root Layout
└── About Layout
    └── About 하위 page.tsx
```

---

## 10.3 Root Layout과 하위 Layout의 차이

| 구분 | Root Layout | 하위 Layout |
|---|---|---|
| 위치 | `app/layout.tsx` | `app/about/layout.tsx` 등 |
| 필수 여부 | 필수 | 선택 |
| html/body 태그 | 반드시 포함 | 포함하지 않음 |
| 적용 범위 | 전체 앱 | 해당 경로와 하위 경로 |

---

# 11. 정의하지 않은 경로와 not-found.tsx

## 11.1 기본 404 페이지

Next.js의 라우팅 방식으로 정의하지 않은 경로에 접속하면 Next.js가 제공하는 기본 404 페이지가 나타난다.

---

## 11.2 not-found.tsx

`not-found.tsx` 파일을 생성하고 컴포넌트를 export하면 정의되지 않은 경로에서 보여줄 UI를 직접 지정할 수 있다.

```tsx
import Link from "next/link";

export default function NotFound() {
  return (
    <>
      <h2>404 Not Found</h2>
      <Link href="/">Return Home</Link>
    </>
  );
}
```

파일 위치:

```text
app/not-found.tsx
```

---

# 12. 동적 라우팅

## 12.1 Path Parameter

Next.js에서는 Path Parameter를 사용해 동적 라우팅을 구현할 수 있다.

```text
/posts/1
/posts/13
/posts/256
```

여기서 `1`, `13`, `256`처럼 경로의 일부가 바뀌는 값을 Path Parameter라고 한다.

---

## 12.2 대괄호 디렉토리

Path Parameter를 사용하려면 디렉토리 이름을 대괄호로 감싼다.

```text
app/products/[productId]/page.tsx
```

이 경로는 다음 URL들과 매칭된다.

```text
/products/1
/products/abc
/products/256
```

---

## 12.3 Next.js 14에서 params 사용

Next.js 14 기준으로 parameter는 page 컴포넌트의 props 중 `params`로 전달된다.

```tsx
interface Props {
  params: {
    productId: string;
  };
}

export default function ProductPage({ params: { productId } }: Props) {
  return <h4>Details of Product {productId}</h4>;
}
```

---

## 12.4 Next.js 15 변경점 — params는 Promise

Next.js 15부터 `params`는 동기 객체가 아니라 Promise로 사용하도록 변경되었다.  
따라서 컴포넌트를 `async function`으로 선언하고 `await`으로 값을 꺼내야 한다.

```tsx
interface Props {
  params: Promise<{
    productId: string;
  }>;
}

export default async function ProductPage({ params }: Props) {
  const { productId } = await params;
  return <h4>Product {productId}</h4>;
}
```

---

## 12.5 params 변경 정리

| 함수/파일 | Next.js 15+에서 params 타입 | 처리 방식 |
|---|---|---|
| `page.tsx` | `Promise<{ ... }>` | `await` 필요 |
| `layout.tsx` | `Promise<{ ... }>` | `await` 필요 |
| `generateMetadata` | `Promise<{ ... }>` | `await` 필요 |
| `route.ts` | `Promise<{ ... }>` | `await` 필요 |
| `generateStaticParams` | 동기, Promise 아님 | 그대로 사용 |

---

# 13. Catch-all Route

## 13.1 Catch-all이란?

디렉토리 이름을 `[...변수명]` 형식으로 만들면 여러 개의 path parameter를 배열로 받을 수 있다.

```text
app/products/[...productIds]/page.tsx
```

매칭 예시:

```text
/products/1
/products/1/2
/products/a/b/c
```

---

## 13.2 Catch-all 예시

```tsx
interface Props {
  params: {
    productIds: string[];
  };
}

export default function ProductsPage({ params: { productIds } }: Props) {
  return (
    <>
      <h4>Details of Products</h4>
      <ul>
        {productIds.map((productId, i) => (
          <li key={i}>{productId}</li>
        ))}
      </ul>
    </>
  );
}
```

---

## 13.3 Catch-all 주의점

`[...productIds]`는 parameter가 없는 경우에는 매칭되지 않는다.

즉, 다음 경로는 매칭되지 않는다.

```text
/products
```

---

# 14. Optional Catch-all Route

## 14.1 Optional Catch-all이란?

parameter가 없는 경우도 포함하려면 디렉토리 이름을 `[[...변수명]]` 형식으로 설정한다.

```text
app/products/[[...productIds]]/page.tsx
```

---

## 14.2 Optional Catch-all 매칭 예시

| URL | 매칭 여부 |
|---|---|
| `/products` | 매칭 |
| `/products/1` | 매칭 |
| `/products/1/2` | 매칭 |
| `/products/a/b/c` | 매칭 |

---

# 15. Query String

## 15.1 Query String이란?

Query String은 URL의 `?` 뒤에 붙는 key-value 형태의 값이다.

```text
/posts?page=2&size=5
/products/list?keyword=phone&page=1&size=10
```

---

## 15.2 Next.js 14에서 searchParams 사용

Query String 값은 page 컴포넌트의 props 중 `searchParams`로 전달된다.

```tsx
interface Props {
  searchParams: {
    keyword?: string;
    page?: number;
    size?: number;
  };
}

export default function ProductListPage({
  searchParams: { keyword, page = 1, size = 10 },
}: Props) {
  return (
    <>
      <h2>Product List</h2>
      {keyword && <h4>searched by: {keyword}</h4>}
    </>
  );
}
```

---

## 15.3 Next.js 15 변경점 — searchParams도 Promise

Next.js 15부터 `searchParams`도 `params`처럼 Promise로 변경되었다.  
따라서 `await`으로 값을 꺼내야 한다.

```tsx
interface Props {
  searchParams: Promise<{
    keyword?: string;
    page?: number;
    size?: number;
  }>;
}

export default async function ProductListPage({ searchParams }: Props) {
  const { keyword, page = 1, size = 10 } = await searchParams;

  return (
    <>
      <h2>Product List</h2>
      {keyword && <h4>searched by: {keyword}</h4>}
    </>
  );
}
```

---

## 15.4 params와 searchParams를 함께 사용하는 경우

Next.js 15 이상에서는 둘 다 Promise이므로 각각 `await`하거나 `Promise.all`로 병렬 처리할 수 있다.

```tsx
export default async function Page({ params, searchParams }: Props) {
  const [{ productId }, { keyword }] = await Promise.all([
    params,
    searchParams,
  ]);

  return <div>{productId} / {keyword}</div>;
}
```

---

# 16. Route Handler

## 16.1 Route Handler란?

Next.js는 React 기반의 풀스택 프레임워크이다.  
따라서 프론트엔드 UI뿐 아니라 일반적인 백엔드 API 개발도 가능하다.

`route.ts` 파일에서 특정 경로에 대한 HTTP Request Handler를 정의할 수 있는데, 이를 **Route Handler**라고 한다.

---

## 16.2 route.ts의 위치와 URL

```text
app/api/products/route.ts
```

위 파일은 다음 API 경로에 대응된다.

```text
/api/products
```

---

## 16.3 HTTP Method 함수 export

HTTP Method 이름을 가진 함수를 export하면 해당 Method에 대한 Request Handler가 등록된다.

```tsx
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
  try {
    const products = [];
    return NextResponse.json(products);
  } catch (e) {
    if (e instanceof Error) {
      return NextResponse.json({ error: e.message }, { status: 400 });
    }
  }
}

export async function POST(request: NextRequest) {
  // Something Created
}

export async function PUT(request: NextRequest) {
  // Something Updated
}

export async function DELETE(request: NextRequest) {
  // Something Deleted
}
```

---

## 16.4 Method와 API 매칭

| export 함수 | 요청 |
|---|---|
| `GET` | `GET /api/products` |
| `POST` | `POST /api/products` |
| `PUT` | `PUT /api/products` |
| `DELETE` | `DELETE /api/products` |

---

## 16.5 NextRequest와 NextResponse

| 객체 | 역할 |
|---|---|
| `NextRequest` | 요청에 대한 정보를 얻기 위한 객체 |
| `NextResponse` | 응답을 생성하기 위한 객체 |

```tsx
return NextResponse.json(products);
return NextResponse.json({ error: e.message }, { status: 400 });
```

---

# 17. Rendering

## 17.1 Next.js를 사용하는 중요한 이유

Next.js의 가장 큰 특징 중 하나는 **최적화된 Rendering 방식**이다.  
React의 기존 Client Side Rendering 방식의 한계를 보완하기 위해 Next.js는 서버 중심 렌더링 방식을 제공한다.

---

## 17.2 React의 기본 렌더링 방식: CSR

React는 기본적으로 **Client Side Rendering, CSR** 방식을 따른다.

```text
브라우저가 빈 HTML과 JS 파일을 받음
→ JS가 로드됨
→ React가 컴포넌트를 실행
→ 화면 요소가 동적으로 생성됨
```

---

## 17.3 CSR의 장점

| 장점 | 설명 |
|---|---|
| 새로고침 없는 동적 화면 | 클라이언트에서 상태를 바꾸며 부드러운 UI 구현 가능 |
| 빠른 페이지 전환 가능 | 한 번 앱이 로드되면 내부 전환이 빠를 수 있음 |
| 풍부한 상호작용 | 이벤트, 상태 관리, 동적 UI 구현이 쉬움 |

---

## 17.4 CSR의 한계

| 한계 | 설명 |
|---|---|
| SEO 어려움 | 초기 HTML에 실제 콘텐츠가 거의 없어 검색 엔진이 내용을 파악하기 어려울 수 있음 |
| 초기 로딩 속도 문제 | JS가 로드되고 실행되어야 화면이 완성됨 |
| JavaScript 비활성화/오류 시 렌더링 불가 | JS가 실행되지 않으면 화면이 제대로 보이지 않을 수 있음 |

---

# 18. Next.js의 기본 렌더링 방식: SSR

## 18.1 SSR이란?

SSR은 **Server Side Rendering**의 약자이다.  
백엔드에서 페이지 대부분의 영역을 미리 처리하여 프론트엔드로 전달하는 방식이다.

```text
서버에서 HTML 생성
→ 브라우저에 완성된 HTML 전달
→ 브라우저가 즉시 콘텐츠 표시
```

---

## 18.2 Next.js가 SSR을 사용하는 이유

Next.js는 CSR의 한계를 해결하기 위해 SSR을 기본 방식으로 채택했다.

| CSR 한계 | SSR로 보완되는 점 |
|---|---|
| 초기 HTML이 비어 있음 | 서버에서 콘텐츠가 포함된 HTML을 내려줌 |
| SEO가 어려움 | 검색 엔진이 HTML 콘텐츠를 읽기 쉬움 |
| JS 오류 시 렌더링 불가 | 기본 HTML은 이미 렌더링되어 있음 |
| 초기 로딩 체감 문제 | 사용자가 콘텐츠를 더 빨리 볼 수 있음 |

---

## 18.3 전통적인 SSR의 단점

| 단점 | 설명 |
|---|---|
| 페이지 전환 시 매번 로드 | 페이지 이동마다 서버에서 HTML을 다시 받아야 함 |
| UX 저하 가능 | 매번 로딩이 발생하면 사용자 경험이 나빠질 수 있음 |
| 상호작용 처리 한계 | 클라이언트 상태와 동적 UI 구현이 제한될 수 있음 |

Next.js는 SSR의 장점과 React의 CSR 방식 장점을 함께 활용할 수 있는 구조를 제공한다.

---

# 19. Server Component와 Client Component

## 19.1 핵심 개념

Next.js의 렌더링 방식을 이해하려면 **Server Component**와 **Client Component**의 차이를 이해해야 한다.

```text
Server Component
vs
Client Component
```

---

# 20. Server Component

## 20.1 Server Component란?

Next.js에서 작성하는 모든 컴포넌트는 기본적으로 **Server Component**이다.

즉, 별도로 `"use client"`를 선언하지 않으면 Server Component로 동작한다.

```tsx
export default async function Page() {
  await new Promise((resolve) => setTimeout(resolve, 2000));

  return (
    <>
      <h4>Characters</h4>
      <ul>
        <li>Hellobit</li>
        <li>Cheshire</li>
        <li>Caterpillar</li>
      </ul>
    </>
  );
}
```

---

## 20.2 Server Component의 특징

| 특징 | 설명 |
|---|---|
| 기본값 | Next.js 컴포넌트는 기본적으로 Server Component |
| 실행 위치 | 서버에서 실행 및 렌더링 |
| async 가능 | `async function`으로 선언 가능 |
| 데이터 패칭에 적합 | 서버에서 직접 데이터를 가져오기 좋음 |
| 클라이언트 기능 제한 | 이벤트 리스너, Hook, state 관리 사용 불가 |
| 최적화 기반 | Next.js의 여러 최적화 기능의 기반이 됨 |

---

## 20.3 Server Component에서 사용할 수 없는 것

Server Component에서는 클라이언트 단에서 실행되어야 하는 기능을 사용할 수 없다.

```tsx
import { useState } from "react";

export default async function Page() {
  const [characterList, setCharacterList] = useState(); // error

  return (
    <>
      <h4 onClick={() => alert("Characters")}>Characters</h4> {/* error */}
    </>
  );
}
```

| 기능 | 이유 |
|---|---|
| `useState` | 클라이언트 상태 관리 Hook |
| `useEffect` | 브라우저 렌더링 이후 실행되는 Hook |
| 이벤트 리스너 | 클릭, 입력 등 브라우저 이벤트 필요 |
| 브라우저 API | `window`, `document`, `localStorage` 등은 서버에 없음 |

---

# 21. Client Component

## 21.1 Client Component란?

Client Component는 브라우저에서 동적인 기능이 필요한 컴포넌트이다.  
파일 상단에 `"use client"`를 추가하여 지정한다.

```tsx
"use client";

import { useState } from "react";

export default function CharactersPage() {
  const [characterList, setCharacterList] = useState([]);

  return (
    <>
      <h4 onClick={() => alert("Characters")}>Characters</h4>
      <ul>
        <li>Hellobit</li>
        <li>Cheshire</li>
        <li>Caterpillar</li>
      </ul>
    </>
  );
}
```

---

## 21.2 Client Component도 SSR 기반이다

주의할 점은 `Client Component`라고 해서 CSR만 의미하는 것은 아니라는 것이다.

Client Component 역시 먼저 서버에서 HTML이 렌더링된다.  
그 후 브라우저에서 Hydration이 진행되어 이벤트 리스너와 Hook이 연결된다.

```text
서버에서 HTML 렌더링
→ 브라우저로 HTML 전달
→ Hydration
→ 이벤트 리스너와 Hook 연결
→ 상호작용 가능
```

---

# 22. Hydration

## 22.1 Hydration이란?

Hydration은 서버에서 렌더링된 HTML에 JavaScript 동작을 부착하는 과정이다.

즉, 이미 화면에 보이는 HTML을 React 컴포넌트처럼 동작하게 만드는 과정이라고 이해할 수 있다.

---

## 22.2 Hydration 과정

```text
1. 서버가 HTML을 렌더링한다.
2. 브라우저가 HTML을 받아 화면에 표시한다.
3. JavaScript가 로드된다.
4. React가 기존 HTML에 이벤트 리스너와 Hook 설정을 부착한다.
5. 사용자는 클릭, 입력 등 상호작용을 할 수 있다.
```

---

# 23. Server Component와 Client Component 비교

| 구분 | Server Component | Client Component |
|---|---|---|
| 기본 여부 | 기본값 | `"use client"` 필요 |
| 실행 위치 | 서버 | 서버 렌더링 후 클라이언트에서 Hydration |
| HTML 렌더링 | 서버에서 수행 | 서버에서 수행 |
| Hydration | 필요 없음 | 필요함 |
| `async function` | 가능 | 일반적으로 서버 컴포넌트 방식과 구분 필요 |
| `useState` | 사용 불가 | 사용 가능 |
| `useEffect` | 사용 불가 | 사용 가능 |
| 이벤트 핸들러 | 사용 불가 | 사용 가능 |
| 브라우저 API | 사용 불가 | 사용 가능 |
| 적합한 상황 | 정적 UI, 데이터 패칭, 서버 작업 | 클릭, 입력, 상태 변화, 브라우저 API 사용 |

---

# 24. 언제 Server Component를 사용할까?

Server Component는 다음 상황에 적합하다.

| 상황 | 이유 |
|---|---|
| 동적인 브라우저 상호작용이 필요 없음 | 이벤트 리스너와 Hook이 필요 없기 때문 |
| 서버에서 데이터를 가져와야 함 | 서버에서 직접 fetch 가능 |
| 민감한 로직을 클라이언트에 보내고 싶지 않음 | 서버에서만 실행 가능 |
| 번들 크기를 줄이고 싶음 | 클라이언트 JS로 내려갈 필요가 적음 |
| SEO와 초기 렌더링이 중요함 | 서버에서 HTML을 생성하기 좋음 |

자료에서는 데이터 fetching 작업은 클라이언트에서도 가능하지만, Server Component에서 하는 것을 권장한다고 설명한다.

---

# 25. 언제 Client Component를 사용할까?

Client Component는 브라우저에서만 가능한 작업이 필요할 때 사용한다.

| 상황 | 예시 |
|---|---|
| 이벤트 핸들러 필요 | `onClick`, `onChange`, `onSubmit` |
| 상태 관리 필요 | `useState`, `useReducer` |
| 생명주기 effect 필요 | `useEffect` |
| 브라우저 API 사용 | `window`, `document`, `localStorage` |
| 사용자 입력 처리 | input, form, modal, dropdown |
| 동적인 UI 상호작용 | 탭, 아코디언, 토글, 드래그 |

---

# 26. 전체 핵심 요약

## 26.1 Next.js 핵심 흐름

```text
React는 UI 라이브러리
→ Next.js는 React 기반 프레임워크
→ 프레임워크가 라우팅, 렌더링, 데이터 처리 흐름 제공
→ 개발자는 app 디렉토리 규칙에 맞춰 파일 작성
→ Next.js가 URL, 렌더링, 서버/클라이언트 역할을 연결
```

---

## 26.2 Routing 핵심

| 개념 | 핵심 |
|---|---|
| App Router | `app/` 디렉토리를 기반으로 라우팅 |
| `page.tsx` | 해당 경로의 UI 렌더링 |
| `layout.tsx` | 하위 라우트와 공유하는 UI |
| `Link` | 클라이언트 사이드 내비게이션 |
| `not-found.tsx` | 404 페이지 커스터마이징 |
| `[id]` | 동적 라우팅 |
| `[...ids]` | Catch-all Route |
| `[[...ids]]` | Optional Catch-all Route |
| `searchParams` | Query String 값 |
| `route.ts` | 백엔드 API Route Handler |

---

## 26.3 Next.js 14와 15+ 차이 핵심

| 항목 | Next.js 14 | Next.js 15+ |
|---|---|---|
| `params` | 동기 객체 | Promise |
| `searchParams` | 동기 객체 | Promise |
| 사용 방식 | 바로 구조분해 가능 | `await` 필요 |
| page/layout/route | 동기 props 방식 | async function 사용 필요 |
| `generateStaticParams` | 동기 | 그대로 동기 사용 |

---

## 26.4 Rendering 핵심

| 렌더링 방식 | 설명 | 장점 | 한계 |
|---|---|---|---|
| CSR | 클라이언트에서 JS로 화면 생성 | 동적 UI, 빠른 내부 전환 | SEO, 초기 로딩, JS 의존 문제 |
| SSR | 서버에서 HTML 생성 후 전달 | SEO, 초기 HTML 제공, JS 비활성화 대응 | 전통 방식은 페이지 전환 UX 한계 |
| Next.js 방식 | SSR 기반 + React 상호작용 | SSR과 React 장점을 함께 활용 | Server/Client Component 구분 필요 |

---

## 26.5 Server Component와 Client Component 핵심

| 구분 | 요약 |
|---|---|
| Server Component | 기본값. 서버에서 실행 및 렌더링. 데이터 패칭과 정적 UI에 적합 |
| Client Component | `"use client"` 필요. 서버 렌더링 후 Hydration. 이벤트, Hook, 브라우저 API 사용 가능 |
| Hydration | 서버에서 만들어진 HTML에 JS 동작을 부착하는 과정 |

---

# 27. 최종 정리

Next.js는 React 기반의 풀스택 프레임워크이다.  
React가 UI를 만들기 위한 라이브러리라면, Next.js는 React 애플리케이션을 실제 서비스 수준으로 만들기 위해 필요한 라우팅, 렌더링, 데이터 처리, 최적화, 캐싱 같은 구조를 제공한다.

프레임워크와 라이브러리의 차이는 코드 흐름 제어권에 있다.  
라이브러리는 개발자가 필요한 기능을 호출하는 방식이고, 프레임워크는 정해진 규칙 안에서 개발자가 코드를 작성하면 프레임워크가 전체 흐름을 제어한다.  
Next.js는 `app/` 디렉토리, `page.tsx`, `layout.tsx`, `route.ts` 같은 파일 규칙을 통해 애플리케이션 구조를 자동으로 연결한다.

Next.js의 Routing은 파일 시스템 기반으로 동작한다.  
`app/about/page.tsx`는 `/about` 경로가 되고, `app/products/[productId]/page.tsx`는 동적 경로가 된다.  
`Link` 컴포넌트를 사용하면 전체 페이지 새로고침 없이 부드러운 클라이언트 사이드 내비게이션을 구현할 수 있다.

Rendering 측면에서 React의 기본 CSR은 동적 UI에는 강하지만, SEO와 초기 로딩, JavaScript 의존성 문제를 가진다.  
Next.js는 SSR을 기본으로 하여 서버에서 HTML을 먼저 생성하고, 필요한 경우 클라이언트에서 Hydration을 통해 상호작용을 연결한다.

Next.js App Router에서는 컴포넌트가 기본적으로 Server Component이다.  
Server Component는 서버에서 실행되며 데이터 패칭과 정적 UI에 적합하지만, `useState`, `useEffect`, 이벤트 핸들러, 브라우저 API를 사용할 수 없다.  
이런 기능이 필요하면 파일 상단에 `"use client"`를 작성하여 Client Component로 만들어야 한다.

결국 Next.js 학습의 핵심은 다음과 같다.

```text
1. Next.js는 React 기반 프레임워크이다.
2. app 디렉토리 구조가 URL 라우팅이 된다.
3. page.tsx는 화면, layout.tsx는 공통 UI, route.ts는 API를 담당한다.
4. Next.js는 SSR을 기본으로 하여 CSR의 한계를 보완한다.
5. 기본 컴포넌트는 Server Component이다.
6. 이벤트, Hook, 브라우저 API가 필요하면 Client Component로 만든다.
7. Client Component도 먼저 서버에서 렌더링되고, 이후 Hydration으로 동작이 연결된다.
```

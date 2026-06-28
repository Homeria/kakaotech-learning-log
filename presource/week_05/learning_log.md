## 🗓 이번 주 개요
- 주차: Week 05
- 주제
    - Docker 컨테이너 기초
    - Docker Compose
    - AWS 인프라 개요
    - 클라우드 DB 연동
- 키워드: 

## 학습 내용

### 1. Docker 컨테이너 기초

- React는 "우리 이런 기능이 있어~ 통제권은 너희에게 있어~"하는 느낌의 **라이브러리**이다. 그래서 React만으로 실제 서비스를 만들려면 여러 기능들을 직접 구성해야 한다.
- Next.js는 **React 기반의 프레임워크**이이며, 라우팅, 렌더링, 데이터 패칭, 최적화, 캐싱 등의 여러 기능을 제공한다. 즉, 코드 흐름 제어의 주도권을 쥐고 있어 개발자는 해당 기능들을 규칙에 맞게 사용하기만 하면 된다.

- Next.js의 Routing
    - 방식 : **파일 시스템 기반 라우팅** 사용
    - 규칙
        - app 디렉터리 하위에 위치
            - Routing과 관련된 디렉터리와 파일은 모두 `app/` 디렉터리 하위에 위치
        - 디렉터리 이름이 URL path가 됨
            - 각 디렉터리의 이름은 URL path에 대응
            - 최상위 `app/` 디렉터리는 `/`에 대응
            - 예시
                | 디렉토리 구조 | URL |
                |---|---|
                | `app/page.tsx` | `/` |
                | `app/about/page.tsx` | `/about` |
                | `app/about/contact/page.tsx` | `/about/contact` |
                | `app/products/list/page.tsx` | `/products/list` |
                | `app/api/products/route.ts` | `/api/products` |
        - page.tsx 또는 route.ts가 있어야 의미 있는 경로가 됨
            - 디렉터리가 URL path에 대응되더라도, 실제로 의미 있는 경로가 되기 위해 다음의 파일이 필요함.
                | 파일 | 역할 |
                |---|---|
                | `page.tsx` | 해당 경로에 접속했을 때 렌더링할 UI 정의 |
                | `route.ts` | 해당 경로에 대한 백엔드 API 정의 |

- tsx의 역할
    - page.tsx
        - 해당 path에 렌더링될 컴포넌트를 정의하는 파일
        - 기본 export로 컴포넌트를 내보내야 함
    - layout.tsx
        - 하위 라우트와 공유하는 UI를 작성하는 파일
        - 모든 페이지의 공통 컴포넌트(헤더, 네비게이션, 사이드바 등)를 작성
        - Root Layout
            - 최상위에 위치한 Layout, `html`과 `body` 태그가 포함되어야 함
        - Children props
            - `children` props에는 path에 해당하는 `page.tsx`에서 내보낸 컴포넌트가 들어옴
    - Navigation과 Link
        - Link 
            - `<a>`와 `<Link>`의 차이
                | 구분 | `<a href="/about">` | `<Link href="/about">` |
                |---|---|---|
                | 이동 방식 | Full Page Reload | Client-Side Navigation |
                | 화면 전환 | 깜빡임 발생 가능 | 부드러운 전환 |
                | 상태 유지 | useState 등 상태 초기화 | 필요한 경우 상태 유지 가능 |
                | 로딩 방식 | HTML, JS, CSS 재요청 | 필요한 컴포넌트 중심 전환 |
                | Prefetch | 없음 | 보이는 경로를 미리 로드 가능 |
                | 사용 권장 | 외부 링크 또는 일반 HTML 이동 | Next.js 내부 페이지 이동 |

- 동적 라우팅
    - Path Parameter
        - `1`, `13`처럼 경로의 일부가 바뀌는 값을 Path Parameter라고 함
        - 해당 라우팅으로 `/posts/1`, `/posts/13`처럼 라우팅 가능
        - 해당 방법을 위해 디렉터리 이름을 대괄호로 감쌈
            - `app/products/[productId]/page.tsx`
    - Next.js 15 - `params`는 동기 객체가 아닌 Promise를 사용하도록 변경, 컴포넌트를 `async function`으로 선언하고 `await`으로 값을 꺼내야 함
        ```tsx
        interface Props { params: Promise<{productId: string;}>; }

        export default async function ProductPage({ params }: Props) {
        const { productId } = await params;
        return <h4>Product {productId}</h4>;
        }
        ```
    - params 변경 정리
        | 함수/파일 | Next.js 15+에서 params 타입 | 처리 방식 |
        |---|---|---|
        | `page.tsx` | `Promise<{ ... }>` | `await` 필요 |
        | `layout.tsx` | `Promise<{ ... }>` | `await` 필요 |
        | `generateMetadata` | `Promise<{ ... }>` | `await` 필요 |
        | `route.ts` | `Promise<{ ... }>` | `await` 필요 |
        | `generateStaticParams` | 동기, Promise 아님 | 그대로 사용 |

- Catch-all Route
    - 디렉터리 이름을 `[...변수명]` 형식으로 만들어 여러 개의 path parameter를 배열로 받음
        - `app/products/[...productIds]/page.tsx` -> `/products/1`, `/products/1/2`, ...
    - 코드 예시
        ```tsx
        interface Props { params: { productIds: string[]; }; }

        export default function ProductsPage({ params: { productIds } }: Props) {
            return (
                ...
                {productIds.map((productId, i) => (
                    <li key={i}>{productId}</li>
                ))}
                ...);
        }
        ```
    - 주의점
        - `[...productIds]`는 parameter가 없는 경우에는 매칭되지 않음
            - `/products`는 매칭되지 않음

- Optional Catch-all Route
    - parameter가 없는 경우도 포함하려면 디렉터리 이름을 `[[...변수명]]` 형식으로 설정
        - `app/products/[[...productIds]]/page.tsx`
        - `/products`, `/products/1`, `/products/1/2`, ...

- Query String
    - URL의 `?` 뒤에 붙는 key-value 형태의 값
        - `/products/list?keyword=phone&page=18&size=10
        - list [keyword=phone, page=18, size=10]
    - Next.js 15부터도 `Promise`를 사용하는 것으로 변경되어 `await`으로 값을 꺼내와야 함
    - 각각 `await`하는 경우
        ```tsx
        interface Props {
            searchParams: Promise<{ keyword?: string; page?: number; size?: number; }>;
        }
        export default async function ProductListPage({ searchParams }: Props) {
            const { keyword, page = 1, size = 10 } = await searchParams;
            return ( {keyword && <h4>searched by: {keyword}</h4>} );
        }
        ```   
    - `Promise.all`로 병렬 처리하는 경우
    ```tsx
    export default async function Page({ params, searchParams }: Props) {
        const [{ productId }, { keyword }] = await Promise.all([ params, searchParams, ]);
        return <div>{productId} / {keyword}</div>;
    }
    ```

- Route Handler
    - Next.js는 React 기반의 **플스택** 프레임워크며, 일반적인 백엔드 API 개발도 가능
    - `route.ts`에서 HTTP Req handler 정의 가능 -> **Route Handler**
    - HTTP Method function export
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
        export async function POST(request: NextRequest) { Something Created }
        export async function PUT(request: NextRequest) { Something Updated }
        export async function DELETE(request: NextRequest) { Something Deleted }

- Rendering 
    - React는 **Client Side Rendering, CSR** 방식을 따름.

        | 장점 | 설명 |
        |---|---|
        | 새로고침 없는 동적 화면 | 클라이언트에서 상태를 바꾸며 부드러운 UI 구현 가능 |
        | 빠른 페이지 전환 가능 | 한 번 앱이 로드되면 내부 전환이 빠를 수 있음 |
        | 풍부한 상호작용 | 이벤트, 상태 관리, 동적 UI 구현이 쉬움 |

        | 한계 | 설명 |
        |---|---|
        | SEO 어려움 | 초기 HTML에 실제 콘텐츠가 거의 없어 검색 엔진이 내용을 파악하기 어려울 수 있음 |
        | 초기 로딩 속도 문제 | JS가 로드되고 실행되어야 화면이 완성됨 |
        | JavaScript 비활성화/오류 시 렌더링 불가 | JS가 실행되지 않으면 화면이 제대로 보이지 않을 수 있음 |

    - Next.js는 **Server Side Rendering, SSR** 방식을 따르며, **SSR**의 장점과 **CSR** 방식 장점을 함께 활용할 수 있는 구조를 제공

        | CSR 한계 | SSR로 보완되는 점 |
        |---|---|
        | 초기 HTML이 비어 있음 | 서버에서 콘텐츠가 포함된 HTML을 내려줌 |
        | SEO가 어려움 | 검색 엔진이 HTML 콘텐츠를 읽기 쉬움 |
        | JS 오류 시 렌더링 불가 | 기본 HTML은 이미 렌더링되어 있음 |
        | 초기 로딩 체감 문제 | 사용자가 콘텐츠를 더 빨리 볼 수 있음 |

    - Server Component
        - Next.js에서 작성하는 모든 컴포넌트는 기본적으로 **Server Component**이며, 별도로 최상단에 `"use client"`를 선언하지 않으면 Server Component로 동작
        - 특징
            | 특징 | 설명 |
            |---|---|
            | 기본값 | Next.js 컴포넌트는 기본적으로 Server Component |
            | 실행 위치 | 서버에서 실행 및 렌더링 |
            | async 가능 | `async function`으로 선언 가능 |
            | 데이터 패칭에 적합 | 서버에서 직접 데이터를 가져오기 좋음 |
            | 클라이언트 기능 제한 | 이벤트 리스너, Hook, state 관리 사용 불가 |
            | 최적화 기반 | Next.js의 여러 최적화 기능의 기반이 됨 |
        - Client 단에서 실행되어야 하는 기능을 사용할 수 없음
            | 기능 | 이유 |
            |---|---|
            | `useState` | 클라이언트 상태 관리 Hook |
            | `useEffect` | 브라우저 렌더링 이후 실행되는 Hook |
            | 이벤트 리스너 | 클릭, 입력 등 브라우저 이벤트 필요 |
            | 브라우저 API | `window`, `document`, `localStorage` 등은 서버에 없음 |

    - Client Component
        - 브라우저에서 동적인 기능이 필요한 컴포넌트, 최상단에 `"use client"`를 명시하여 지정
        - **CSR**만 의미하는 것은 아니며, 먼저 서버에서 HTML이 렌더링 된 후 Hydration이 진행되어 이벤트 리스너와 Hook 연결

    - Server Component vs Client Component
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

- Hydration
    - 서버에서 렌더링된 HTML에 JavaScript 동작을 부착하는 과정, 즉 HTML을 React 컴포넌트처럼 동작하게 만든느 과정
    - 과정
        - 서버가 HTML을 렌더링
        - 브라우저가 HTML을 받아 화면에 표시
        - JavaScript 로드
        - React가 기존 HTML에 이벤트 리스너와 Hook 설정 부착
        - 사용자는 클릭, 입력 등 상호작용 가능


- React에서 `npm run dev` 하면 HTML 렌더링을 해주니까 React 자체가 프레임워크인 줄 알고 있었으나, 사실 Next.js가 프레임워크인 걸 처음 알게 됐다.
- HTML/CSS/JS 파트에서는 사실 "우리 코딩한다!"라고 하는 느낌보다는 디자인적인 느낌이 강했다. React는 전 파트보다는 로직 관련한 기능이 많이 있었지만, 여전히 디자인적인 느낌이 강했다. Next.js를 통해서 드디어 웹 페이지를 제공하는 '서버'로써의 역할을 다하게 되는 느낌이 들었다.

---

### 2. NextJS 데이터 처리와 최적화

- Data Fetching
    - Next.js는 기본적으로 Server Component를 사용하므로, 서버에서 직접 데이터를 가져와 컴포넌트를 구성할 수 있음
    - 즉, FE-BE가 구성돼있을 때 BE가 사용자에게 노출되지 않고 FE가 먼저 BE에게 데이터를 받아서 **SSR**을 진행하여 제공할 수 있으므로 외부에 API 요청이 보이지 않을 수 있음
    - 서버 작업이 오래 걸리면 페이지 응답이 늦어질 수 있음. 즉, Data Fetching이 끝나기 전까지 **SSR** 방식은 웹 페이지를 제공할 수 없음.
    - 위와 같은 단점을 해결하고자 Server Component에서의 Data Fetching을 하는 과정에서는 `loading.tsx`를 통해 로딩 화면을 먼저 보여주어 Data Fetching 중임을 표시할 수 있음.
- Streaming
    - HTML 또는 데이터를 작은 조각(**chunk**)로 나누어 준비가 완료도니 조각부터 클라이언트에 점진적으로 전송하는 기술
    - 즉, 서버가 모든 렌더링을 끝낸 후에 한 번에 보내는 게 아닌 준비된 부분부터 나누어 보낼 수 있음
    - React는 component 단위로 구성하기 때문에 chunk 방식과의 조합이 좋음.
    - `loading.tsx` 파일 추가 시 자동으로 Streaming 기능을 적용함
- 여러 개의 Data Fetching
    - 요청 B를 위해 요청 A의 결과값이 필요하는 등의 귀속적 관계가 아니라면, Fetching을 병렬로 진행할 수 있음.
    - 코드 예시
        ```tsx
        export default async function PostsPage() {
        const [fetchedPosts, fetchedUsers] = await Promise.all([
            getPosts(),
            getUsers(),
        ]);
        return ( {/* posts와 users 렌더링 */} );
        }
        ```
    - `Promise.all`은 전체 작업이 모두 끝나야 결과를 반환하기 때문에, 완료된 데이터가 있어도 전체 Fetch가 끝나기 전까지 렌더링되지 않는 닩머이 있음
- Parallel Fetching과 Suspense
    - `Suspense`
        - 각 fetch 작업을 독립된 Server Component로 분리하고, 각각을 `Suspense`로 감싸는 방법

        ```tsx
        // app/posts/PostsList.tsx
        interface Post { id: number; }

        export default async function PostsList() {
            const response = await fetch(URL);
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
        import { Suspense } from "react";
        import PostsList from "./PostsList";

        export default function PostsPage() {
        return (
            <>
            <h2>게시글 목록</h2>
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr" }}>
                <Suspense fallback={<h2>Loading Posts...</h2>}>
                <PostsList />
                </Suspense>
            </div>
            </>
        );
        }
        ```

- POST, PUT, DELETE Data Fetching
    - POST(생성), PUT/PATCH(수정), DELETE(삭제)는 서버나 DB의 상태를 변경하는 작업
    - Client Component에서 처리하는 방식
        - `"use client"`와 함께 React에서 하던 방식으로 처리하는 방식

        | 한계 | 설명 |
        |---|---|
        | Client Component 필요 | `"use client"`를 사용해야 함 |
        | Hydration 필요 | 이벤트 리스너와 state 연결을 위해 hydration 발생 |
        | 비즈니스 로직 노출 | 일부 로직이 클라이언트 코드에 포함될 수 있음 |
        | 서버 기능 활용 부족 | DB 작업, 인증/인가, redirect 등을 서버에서 자연스럽게 처리하기 어려움 |

    - Next.js에서는 서버 측에서 로직을 처리하는 것이 좋아, Server Actions를 제공함

- Server Actions
    - 무조건 서버에서 실행되는 비동기 함수
    - 클라이언트 번들에 포함되지 않으며, 서버/DB 상태가 변경되는 작업을 처리하는 데 사용

- Caching
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

- Optimization
    | 기능 | 개선 대상 | 핵심 |
    |---|---|---|
    | Image 컴포넌트 | CLS, LCP | 이미지 크기 확보, 최적화, priority |
    | placeholder blur | UX | 이미지 로딩 중 블러 이미지 표시 |
    | quality | LCP | 이미지 품질 조절 |
    | Metadata | SEO | title, description, Open Graph 설정 |
    | generateMetadata | SEO | 데이터 기반 동적 metadata 생성 |

- 내용을 정리하다보니, 현재 방법으로는 너무 자세하게 남기는 듯 하다. 어차피 해당 레포지토리에 정리본이 있으니, 내용이 생각나지 않을 때 학습 일지를 보는 대신 정리본을 보는 게 나을 듯 하다. 정리할 때 한 번씩 1주의 내용을 리마인드할 수 있는 게 좋지만 말이다...

---

### 3. FE-BE API 연동

- 핵심 내용
    1. Next.js는 화면과 서버 로직을 함께 다룰 수 있는 **풀스택** 프레임워크이다.
    2. 데이터 조회는 Server Component에서 처리하기 좋다
    3. 데이터 변겨은 Server Action과 form action 패턴으로 처리할 수 있다.
    4. FastAPI는 요청을 받고 Pydantic으로 검증한 뒤 비즈니스 로직을 수행한다.
    5. SQLAlchemy는 Python 객체와 DB 테이블을 연결하는 **ORM**이다.
    6. SQLite는 실제 데이터를 DB에 저장한다.
    7. 전체 흐름은 Next.js -> Server Action -> FastAPI -> SQLAlchemy -> SQLite -> 응답 -> 화면 업데이트
- 아무래도 실시간 강의인 만큼 리마인드 성격의 내용이 많았다. Next.js의 구성과 코드를 복습하고, 새롭게 SQLAlchemy **ORM**이 적용된 FastAPI 백엔드 서버와의 연동을 실습해보는 것에 의의를 두었다.
- SQLite의 Raw Query로는 많이 접해보고 서버를 구성해보았지만, 들어보기만 했던 ORM으로 서버를 구성해보니 왜 사용하는지는 알 것 같다.
- 개인적으로는 Query문이 더 명확해 보이긴 하지만, 이건 내가 익숙해서 그런 것 같다. 아마 ORM에 익숙해지면 ORM이 더 편할 거 같기도 하다.
- 다만, ORM에서는 N+1과 같은 이슈들이 있다고는 들었는데, 그건 FE-BE 연동 심화 과정을 겪어보며 조사해봐야겠다.

---

### 4. 풀스택 통합 실습

- 핵심 내용
    1. Next.js는 프론트엔드와 서버 로직을 함께 다룰 수 있다.
    2. Server Component와 Client Component는 사용자 인터랙션 여부로 구분한다.
    3. Direct Fetch는 단순하지만 URL 노출과 CORS 문제가 있다.
    4. Route Handler는 Next.js 서버를 중계 계층으로 사용해 보안성과 확장성을 높인다.
    5. axios는 클라이언트 요청 코드를 더 간결하고 안정적으로 만든다.
    6. 검색 상태는 URL 쿼리 파라미터로 표현하면 공유성과 확장성이 좋아진다.
    7. 데이터가 많아질수록 클라이언트 필터링보다 서버/DB 필터링이 적절하다.
- Next.js와 FastAPI를 연결해 실제 풀스택 **흐름을 이해**하는데 중점을 둔 내용이었다.
- Next.js의 핵심은 Routing, Rendering, Data Fetching, Caching, Optimizing이었고, 특히 Data Fetching, Route Handler, Server Component, Client Component의 역할 차이를 중심의 내용이었다.
- Client Component에서 FastAPI를 직접 호출하는 Direct Fetch, Route Handler를 통한 axios 방식의 Fetch를 실습해보면서, 각각의 장단점을 코드로 보고 실행해보며 느낄 수 있었다.
- 보안이 중요시되는 등의 위치에서는 FastAPI를 Next.js에서 호출하는 방식이 더 어울릴 것이고... 그렇다고 트래픽이 중요한 환경에서 백엔드 URL이 노출되는 걸 감수하고 응답 속도를 높이는 건... 아닌 듯 하다.
- HTTP Req 코드를 axios로 리팩토링해보며, 더 명확해보인다는 느낌을 받았다.

---

### 5. 실전 배포 스프린트
- 핵심 내용
    1. Direct Fetch는 단순하지만 백엔드 주소 노출과 CORS 문제가 있다.
    2. Route Handler는 Next.js 서버를 프록시처럼 사용해 보안성과 구조를 개선한다.
    3. axios는 클라이언트 HTTP 요청 코드를 간결하고 안정적으로 만든다.
    4. Next.js 프론트엔드는 Vercel에, FastAPI 백엔드는 Railway에 배포한다.
    5. 배포 후에는 환경변수와 CORS 설정을 반드시 확인해야 한다.
    6. E2E 검증은 실제 사용자 흐름으로 앱 전체가 동작하는지 확인하는 과정이다.
    7. Vercel + Railway는 빠른 배포에 적합하고, Docker + AWS는 높은 제어권과 운영 확장성이 필요한 경우에 적합하다.
- 해당 강의도 3일차와 마찬가지로 실시간 강의인 만큼, 전 강의 내용을 복습하는 느낌이 강했다.
- 복습을 하고나서, Vercel(Next.js), Railway(FastAPI) 조합으로 배포 및 호스팅을 진행해보며, 내가 만든 게 직접 웹 페이지로 제공된다는 것이 신기했다. 다만, 처음으로 진행해본 배포 및 호스팅인 만큼 환경 변수 관련 오류가 발생해서 완전한 호스팅은 불가했던 게 아쉬웠다.
- Docker + AWS도 사실 실습해보고 싶었으나 시간이 되지 않아 진행해보지 못했다. 사실 모든 개발자들이 다뤄봐야 하는 조합이긴 한데, 다음 코스를 밟아가면서 언젠가 진행해보지 않을까 싶다. 지금은 지금의 일에 집중하는 편이 좋아보인다.

## 🧱 막혔던 지점 & 해결 과정
- 문제: Next.js에서 Fetching과 HTTP Method를 사용하는 데 어려움이 있었다.
- 시도: fetching과 HTTP Method를 사용하는데 사용하는 틀(보일러플레이트?)을 직접 적용해보기, (3, 5일차에 했던 실습 중 try-catch-finally 세트를 기억해보기)
- 해결: 세트를 가져다 써보면서 이해해보려고 노력하고, 어느 정도는 사용하는 빙법을 익힐 수 있었다. 
- 배운 점: fetching을 하는 다양한 방법을 써보면서 각각의 장단점을 떠올릴 수 있었다.
- 아쉬운 점: 이 세트를 자세히 써봐야 사용할 떄마다 즉각적으로 떠오를 거 같다는 느낌이 드는 것

## 🔁 이번 주 회고 (KPT)
- Keep: 강의 내용 정리는 PDf 및 실습을 다시 돌려보는 것이 아닌 AI에게 마크다운 형식으로 정리해달라고 하여 시간을 효율적으로 사용하는 것
- Problem: 학습 일지에 모든 것을 다 적으려고 하는 것
- Try: 학습 일지에는 내용 조금과 내가 느낀 점을 위주로 적는 걸로 바꾸고, AI에게 강의 내용 외에 궁금한 걸 물어보며 기억해보기

## 🎯 다음 주 목표
- [ ] 학습 일지에 모든 걸 다 적으려 하지 말고, 초압축 요약본과 이떄 내가 가졌던 궁금증 및 이해가 가지 않았던 이유 등을 자세히 서술하기
- [ ] 학습 일지 작성 밀리지 않게 노력하기...
- [ ] 기말고사 잘 보기...
- [ ] 종강하고 프리코스 과제3를 시작하기 전 복습하기
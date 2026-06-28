# Day 3-1 · Day 3-2 · Day 3-3 Next.js 풀스택 흐름 정리

> 원본 자료:  
> - `Day3-1 - Next.js 기초 & 데이터 처리 복습.md`  
> - `Day3-2 - SQLAlchemy 소개 및 CRUD 실습.md`  
> - `Day3-3 - End-to_End 흐름 확인.md`  
>
> 정리 순서: **3-1 Next.js 기초와 데이터 처리 복습 → 3-2 SQLAlchemy 소개 및 CRUD 실습 → 3-3 End-to-End 흐름 확인**

---

# 1. Day 3-1 — Next.js 기초와 데이터 처리 복습

## 1.1 Framework vs Library

React는 **라이브러리**이고, Next.js는 **프레임워크**이다.

React는 UI를 만들기 위한 기능을 제공하지만, 라우팅, 데이터 페칭, 폴더 구조, 렌더링 방식 등은 개발자가 직접 선택하고 구성해야 한다.  
반면 Next.js는 이미 정해진 규칙을 제공한다. 개발자는 그 규칙에 맞게 파일과 코드를 작성하면 된다.

| 구분 | React | Next.js |
|---|---|---|
| 종류 | 라이브러리 | 프레임워크 |
| 제어권 | 개발자에게 있음 | 프레임워크가 큰 흐름을 제어 |
| 라우팅 | 직접 구성 필요 | 파일 기반 라우팅 제공 |
| 데이터 처리 | 직접 패턴 설계 | Server Component, Route Handler, Server Action 등 제공 |
| 비유 | 식재료만 제공 | 레시피와 주방 도구까지 제공 |

---

## 1.2 App Router와 File-based Routing

Next.js에서는 **폴더와 파일의 위치가 URL 경로**가 된다.

즉, `app/` 폴더 안에 어떤 폴더와 파일을 만들었는지가 곧 라우팅 구조가 된다.

| 파일 경로 | URL |
|---|---|
| `app/page.tsx` | `/` |
| `app/posts/page.tsx` | `/posts` |
| `app/posts/[postId]/page.tsx` | `/posts/1`, `/posts/2` |

---

## 1.3 App Router 주요 파일

| 파일 | 역할 |
|---|---|
| `page.tsx` | 해당 경로에서 보여줄 화면 |
| `layout.tsx` | 공통 레이아웃. 하위 페이지가 `children` prop으로 전달됨 |
| `loading.tsx` | 페이지 로딩 중 보여줄 UI |
| `error.tsx` | 에러 발생 시 보여줄 UI |
| `[postId]` | 동적 라우트. URL의 일부를 변수처럼 사용 |

---

# 2. Server Component에서 데이터 가져오기

## 2.1 Server Component의 특징

Next.js의 Server Component는 서버에서 실행된다.  
따라서 컴포넌트 자체를 `async` 함수로 만들고, 그 안에서 직접 데이터를 가져올 수 있다.

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

---

## 2.2 Server Component의 장점

Server Component는 브라우저가 아니라 서버에서 실행된다.  
서버에서 데이터 요청과 화면 조립을 끝낸 뒤, 브라우저에는 완성된 HTML이 전달된다.

| 장점 | 설명 |
|---|---|
| 빠른 초기 화면 | 서버에서 만들어진 HTML을 받으므로 첫 화면 표시가 빠름 |
| 번들 크기 감소 | 불필요한 클라이언트 JavaScript를 줄일 수 있음 |
| 환경변수 보호 | 서버 환경변수를 클라이언트에 노출하지 않고 사용 가능 |
| DB/API 직접 접근 | 서버에서 백엔드 API나 DB에 접근하기 좋음 |

---

# 3. Streaming과 loading.tsx

## 3.1 SSR의 한계

SSR은 서버에서 HTML을 완성해 브라우저로 보내는 방식이다.  
하지만 서버에서 데이터를 모두 가져올 때까지 브라우저가 빈 화면을 보게 될 수 있다.

```text
브라우저 요청
→ 서버 데이터 페칭
→ 서버 렌더링 완료
→ 브라우저에 HTML 전달
```

데이터 페칭이 오래 걸리면 사용자 입장에서는 화면이 늦게 뜬다고 느낄 수 있다.

---

## 3.2 Streaming으로 해결

Streaming은 HTML을 한 번에 완성해서 보내는 것이 아니라, 작은 조각으로 나누어 준비된 부분부터 브라우저에 보내는 방식이다.

`loading.tsx`를 추가하면 Next.js는 레이아웃과 로딩 UI를 먼저 보내고, 데이터 준비가 끝나면 나머지 컴포넌트를 추가로 전송한다.

```text
loading.tsx가 있을 때:

1. 브라우저가 요청
2. 서버가 레이아웃 + 로딩 UI 즉시 전송
3. 서버에서 데이터 페칭 진행
4. 데이터 준비 완료
5. 완성된 컴포넌트 추가 전송
6. 브라우저 화면 업데이트
```

---

## 3.3 loading.tsx의 역할

| 파일 | 역할 |
|---|---|
| `loading.tsx` | 로딩 중 빈 화면 대신 보여줄 UI |
| Streaming | 준비된 HTML chunk를 점진적으로 전송 |
| Suspense | 컴포넌트 단위 로딩 UI 처리 |

---

# 4. 여러 개의 Data Fetching과 Promise.all

## 4.1 순차 실행의 문제

여러 데이터를 순차적으로 가져오면 대기 시간이 합산된다.

```tsx
// 순차 실행
const user = await fetchUser();    // 100ms
const posts = await fetchPosts();  // 200ms
```

이 경우 총 대기 시간은 약 300ms가 된다.

---

## 4.2 Promise.all 병렬 실행

두 요청이 서로 의존하지 않는다면 `Promise.all`로 동시에 실행하는 것이 효율적이다.

```tsx
const [user, posts] = await Promise.all([
  fetchUser(),
  fetchPosts(),
]);
```

| 방식 | 소요 시간 |
|---|---|
| 순차 실행 | 각 요청 시간의 합 |
| 병렬 실행 | 가장 오래 걸리는 요청 시간 |

---

# 5. Server Component vs Client Component

## 5.1 기본 구분

Next.js의 모든 컴포넌트는 기본적으로 **Server Component**이다.  
브라우저에서 실행되어야 하는 기능이 필요할 때만 파일 상단에 `"use client"`를 작성하여 Client Component로 만든다.

---

## 5.2 비교표

| 구분 | Server Component | Client Component |
|---|---|---|
| 선언 방법 | 기본값, 별도 선언 불필요 | 파일 상단에 `"use client"` |
| 실행 위치 | 서버 | 서버 사전 렌더링 + 브라우저 Hydration |
| 컴포넌트 함수 async | 가능 | 컴포넌트 함수 자체에는 사용 불가 |
| `useState`, `useEffect` | 사용 불가 | 사용 가능 |
| `onClick`, `onChange` | 사용 불가 | 사용 가능 |
| DB/환경변수 직접 접근 | 가능 | 불가, 브라우저 노출 위험 |
| 클라이언트 번들 크기 | 영향 없음 | 번들에 포함됨 |

---

## 5.3 Server Component 예시

```tsx
// app/posts/page.tsx
export default async function PostsPage() {
  const res = await fetch(`${process.env.FASTAPI_URL}/posts`);
  const posts = await res.json();

  return (
    <ul>
      {posts.map((p) => (
        <li key={p.id}>{p.title}</li>
      ))}
    </ul>
  );
}
```

서버에서 직접 FastAPI를 호출하므로 `FASTAPI_URL`이 클라이언트에 노출되지 않는다.

---

## 5.4 Client Component 예시

```tsx
"use client";

import { useState } from "react";

export default function SearchPage() {
  const [query, setQuery] = useState("");

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="검색어 입력..."
    />
  );
}
```

입력값을 상태로 관리하고, 사용자의 입력 이벤트를 처리해야 하므로 Client Component가 필요하다.

---

## 5.5 선택 기준

핵심 질문은 다음과 같다.

```text
이 컴포넌트에 클릭, 입력, 상태 변화 같은 사용자 인터랙션이 필요한가?
```

| 상황 | 선택 |
|---|---|
| 데이터 조회와 화면 표시만 필요 | Server Component |
| 클릭, 입력, 상태 변화 필요 | Client Component |
| DB/API 접근, 환경변수 사용 | Server Component |
| 브라우저 API 사용 | Client Component |
| 검색창, 버튼, 모달, 폼 실시간 제어 | Client Component |

---

# 6. Server Actions

## 6.1 Server Action이란?

Server Action은 `"use server"` 지시어를 선언한 **서버 전용 함수**이다.  
Client Component나 `<form>`에서 직접 호출할 수 있고, 별도의 API Route를 만들지 않아도 서버 로직을 실행할 수 있다.

주로 사용자의 입력이나 상호작용으로 인해 서버 데이터가 변경되어야 할 때 사용한다.

```text
Create
Update
Delete
```

---

## 6.2 actions.ts 예시

```tsx
// app/actions.ts
"use server";

import { revalidateTag } from "next/cache";
import { redirect } from "next/navigation";

export async function createPost(formData: FormData) {
  const title = formData.get("title") as string;
  const content = formData.get("content") as string;

  await fetch(`${process.env.FASTAPI_URL}/posts`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ title, content }),
  });

  revalidateTag("posts-list");
  redirect("/posts");
}
```

---

## 6.3 form action 패턴

```tsx
// app/posts/new/page.tsx
import { createPost } from "@/app/actions";

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="제목" />
      <textarea name="content" placeholder="내용" />
      <button type="submit">작성</button>
    </form>
  );
}
```

HTML `<form>`의 `action`에 Server Action을 연결하면 제출 시 `FormData`가 자동으로 전달된다.

---

## 6.4 Server Action의 장점

| 장점 | 설명 |
|---|---|
| 서버에서 실행 | 클라이언트 번들에 포함되지 않음 |
| 환경변수 보호 | `FASTAPI_URL` 같은 서버 환경변수를 안전하게 사용 |
| API Route 생략 가능 | 별도 Route Handler 없이 서버 로직 호출 |
| 캐시 갱신 가능 | `revalidateTag()`로 최신 데이터 반영 |
| 리다이렉트 가능 | 작업 완료 후 `redirect()`로 이동 |

---

## 6.5 SSR의 한계를 줄이는 전략

SSR은 초기 로딩이 빠르지만, 서버 컴포넌트만으로는 사용자 인터랙션을 처리하기 어렵다.  
따라서 다음처럼 역할을 나누는 것이 좋다.

```text
정적 데이터 표시
→ Server Component

검색창, 버튼, 입력, 모달
→ Client Component

데이터 변경 작업
→ Server Action
```

이렇게 하면 SSR의 장점인 빠른 초기 화면을 유지하면서, 필요한 곳에만 클라이언트 상호작용을 추가할 수 있다.

---

# 7. Day 3-2 — SQLAlchemy 소개 및 CRUD 실습

## 7.1 SQLAlchemy란?

SQLAlchemy는 Python 클래스와 데이터베이스 테이블을 연결해 주는 **ORM(Object-Relational Mapping)** 라이브러리이다.

SQL 문장을 직접 작성하지 않고도 Python 코드로 데이터베이스의 데이터를 관리할 수 있게 도와준다.

```text
Python 객체
↔ SQLAlchemy ORM
↔ SQL 쿼리
↔ Database Table
```

---

## 7.2 ORM이 필요한 이유

이전에는 `sqlite3` 모듈로 SQL 문자열을 직접 작성했다.

```python
cursor.execute(
    "INSERT INTO messages (user_id, content) VALUES (?, ?)",
    (1, "안녕")
)
```

이 방식은 테이블이 많아지고 프로젝트 규모가 커질수록 복잡해진다.  
문자열 안의 SQL 오타는 실행 전까지 확인하기 어렵고, IDE 자동완성이나 타입 지원을 받기도 어렵다.

ORM은 Python 코드를 SQL로 변환해주는 번역기 역할을 한다.

---

## 7.3 sqlite3 방식과 SQLAlchemy 방식 비교

| 작업 | sqlite3 방식 | SQLAlchemy 방식 |
|---|---|---|
| 등록 | `cursor.execute("INSERT INTO ...")` | `db.add(post)` |
| 조회 | `cursor.execute("SELECT * FROM ...")` | `db.execute(select(Post)).scalars().all()` |
| 삭제 | `cursor.execute("DELETE FROM ...")` | `db.delete(post)` |
| 안정성 | 실행 전 오류 확인 어려움 | IDE 자동완성, 타입 지원 가능 |
| 코드 형태 | SQL 문자열 중심 | Python 객체 중심 |

---

# 8. SQLAlchemy 핵심 개념

## 8.1 설치 및 실행 준비

```bash
pip install SQLAlchemy
```

백엔드 실행 흐름은 다음과 같다.

```bash
cd fullstack-practice/backend

uv venv
uv pip install -r requirements.txt
uv run fastapi dev main.py
```

에러가 발생하면 잘못 생성된 `.venv` 폴더를 삭제하고 다시 시작할 수 있다.

```bash
rm -rf .venv
```

프론트엔드는 다음 명령어로 실행한다.

```bash
cd fullstack-practice/frontend

npm install
npm run dev
```

---

## 8.2 데이터베이스 연결 설정

SQLAlchemy를 사용하기 위해서는 `engine`, `SessionLocal`, `Base`를 설정한다.

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase

DATABASE_URL = "sqlite:///./blog.db"

engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False}
)

SessionLocal = sessionmaker(
    bind=engine,
    autoflush=False,
)

class Base(DeclarativeBase):
    pass
```

---

## 8.3 핵심 객체 역할

| 개념 | 역할 |
|---|---|
| `engine` | 실제 데이터베이스 파일과의 연결 경로를 열고 관리 |
| `SessionLocal` | 데이터 조회, 추가, 수정, 삭제 작업을 수행할 세션 생성 |
| `Base` | Python 클래스가 DB 테이블과 연결될 수 있도록 규칙 제공 |
| `Model` | DB 테이블에 저장될 구체적인 항목을 선언한 클래스 |
| `Session` | 하나의 요청에서 DB 작업을 수행하는 작업 단위 |

---

# 9. Model — Python 클래스와 DB 테이블 매핑

## 9.1 Post 모델

`Base`를 상속받아 데이터베이스에 생성할 `posts` 테이블 구조를 설계한다.

```python
from datetime import datetime, timezone
from sqlalchemy import Integer, String, Text, DateTime
from sqlalchemy.orm import Mapped, mapped_column

class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(Integer, primary_key=True, index=True)

    title: Mapped[str] = mapped_column(String(200), nullable=False)

    content: Mapped[str] = mapped_column(Text, nullable=False)

    created_at: Mapped[datetime] = mapped_column(
        DateTime,
        default=lambda: datetime.now(timezone.utc)
    )

Base.metadata.create_all(bind=engine)
```

---

## 9.2 Post 모델 필드 정리

| 필드 | 타입 | 설명 |
|---|---|---|
| `id` | Integer | 각 게시글을 식별하는 고유 번호, Primary Key |
| `title` | String(200) | 게시글 제목, 최대 200자, 빈 값 불가 |
| `content` | Text | 게시글 본문, 빈 값 불가 |
| `created_at` | DateTime | 글 작성 시간, 기본값은 현재 UTC 시간 |

---

## 9.3 create_all

```python
Base.metadata.create_all(bind=engine)
```

이 코드는 서버가 실행될 때 설계한 테이블이 실제 DB 파일에 없으면 자동으로 생성한다.

---

# 10. DB 세션 관리

## 10.1 get_db 함수

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

`get_db()`는 요청에서 사용할 DB 세션을 빌려주고, 요청이 끝나면 세션을 닫는 역할을 한다.

---

## 10.2 get_db가 필요한 이유

| 이유 | 설명 |
|---|---|
| 코드 중복 방지 | 매 라우트마다 세션 열고 닫는 코드를 반복하지 않아도 됨 |
| 안정성 | 요청 종료 후 세션을 확실하게 닫을 수 있음 |
| FastAPI 의존성 주입 | `Depends(get_db)`로 각 엔드포인트에 세션 전달 가능 |

---

# 11. 데이터 조회 API

## 11.1 전체 조회

전체 게시글을 조회할 때는 `select(Post)`와 `db.scalars(...).all()`을 사용한다.

```python
@app.get("/posts", response_model=list[PostResponse])
def get_posts(db: Session = Depends(get_db)):
    return db.scalars(select(Post)).all()
```

| 코드 | 의미 |
|---|---|
| `select(Post)` | `SELECT * FROM posts`에 해당하는 쿼리 생성 |
| `db.scalars(...)` | 쿼리를 실행하고 ORM 객체 중심으로 결과 추출 |
| `.all()` | 전체 결과를 리스트로 반환 |

---

## 11.2 단건 조회

특정 조건에 맞는 데이터 한 건만 조회할 때는 `where`와 `db.scalar()`를 사용한다.

```python
stmt = select(Post).where(Post.id == post_id)
post = db.scalar(stmt)
```

| 코드 | 의미 |
|---|---|
| `.where(Post.id == post_id)` | 특정 ID와 일치하는 조건 |
| `db.scalar(stmt)` | 조건에 맞는 단일 객체 반환, 없으면 None |

---

## 11.3 단건 조회 헬퍼 함수

반복되는 단건 조회를 별도 함수로 분리하면 가독성이 좋아진다.

```python
def _get_post_or_none(db: Session, post_id: int) -> Post | None:
    return db.scalar(select(Post).where(Post.id == post_id))
```

---

## 11.4 실습 1 — GET /posts/{post_id}

특정 ID의 게시글 하나를 조회하는 API이다.

```python
@app.get("/posts/{post_id}", response_model=PostResponse)
def get_post(post_id: int, db: Session = Depends(get_db)):
    stmt = select(Post).where(Post.id == post_id)
    post = db.scalar(stmt)

    if not post:
        raise HTTPException(status_code=404, detail="게시글을 찾을 수 없습니다")

    return post
```

---

# 12. 트랜잭션과 데이터 생성 API

## 12.1 트랜잭션이란?

트랜잭션은 더 이상 쪼갤 수 없는 하나의 논리적인 작업 단위이다.

여러 DB 작업 중 하나라도 실패하면, 전체 작업을 실행 전 상태로 되돌려 데이터가 잘못 저장되는 것을 막는다.

---

## 12.2 트랜잭션 예시

```text
송금 거래

1. A 계좌에서 10만원 인출
2. B 계좌로 10만원 입금

만약 2단계에서 실패하면
1단계 인출도 취소되어야 한다.
```

이때 전체 작업을 되돌리는 것을 **rollback**이라고 한다.

---

## 12.3 데이터 생성 흐름

새 데이터를 DB에 안전하게 저장하는 과정은 다음과 같다.

| 단계 | 코드 | 설명 |
|---|---|---|
| 1 | `db.add(post)` | 새 객체를 세션에 임시 추가 |
| 2 | `db.commit()` | DB에 영구 저장 |
| 3 | `db.refresh(post)` | DB에서 생성된 id, created_at 등을 객체에 동기화 |
| 4 | `db.rollback()` | 에러 발생 시 임시 변경 사항 취소 |

---

## 12.4 try-except 패턴

```python
try:
    post = Post(title="제목", content="내용")
    db.add(post)
    db.commit()
    db.refresh(post)
except Exception as e:
    db.rollback()
```

---

## 12.5 실습 2 — POST /posts

게시글을 생성하는 API이다.

```python
@app.post("/posts", response_model=PostResponse, status_code=201)
def create_post(data: PostCreate, db: Session = Depends(get_db)):
    try:
        post = Post(title=data.title, content=data.content)

        db.add(post)
        db.commit()
        db.refresh(post)

        return post
    except Exception as e:
        db.rollback()
        raise HTTPException(status_code=500, detail=f"게시글 생성 실패: {str(e)}")
```

---

# 13. 데이터 수정 API

## 13.1 ORM 방식의 수정

SQLAlchemy에서는 직접 SQL `UPDATE`문을 작성하지 않아도 된다.

먼저 DB에서 객체를 조회한 뒤, Python 객체의 속성을 수정하고 `commit()`하면 SQLAlchemy가 변경을 감지해 자동으로 `UPDATE`를 실행한다.

```python
post = db.scalar(select(Post).where(Post.id == 1))

post.title = "새로운 제목"

db.commit()
```

---

## 13.2 실습 3 — PUT /posts/{post_id}

```python
@app.put("/posts/{post_id}", response_model=PostResponse)
def update_post(post_id: int, data: PostUpdate, db: Session = Depends(get_db)):
    post = _get_post_or_none(db, post_id)

    if not post:
        raise HTTPException(status_code=404, detail="게시글을 찾을 수 없습니다")

    try:
        if data.title is not None:
            post.title = data.title
        if data.content is not None:
            post.content = data.content

        db.commit()
        db.refresh(post)

        return post
    except Exception as e:
        db.rollback()
        raise HTTPException(status_code=500, detail=f"게시글 수정 실패: {str(e)}")
```

---

## 13.3 수정 흐름

```text
1. 수정 대상 게시글 조회
2. 없으면 404 반환
3. title 또는 content 값이 있으면 객체 속성 수정
4. commit으로 DB 반영
5. refresh로 최신 데이터 동기화
6. 실패 시 rollback
```

---

# 14. 데이터 삭제 API

## 14.1 ORM 방식의 삭제

삭제할 대상을 먼저 조회한 뒤, `db.delete(post)`로 삭제 대상으로 표시하고 `db.commit()`으로 DB에 반영한다.

```python
post = db.scalar(select(Post).where(Post.id == 1))

db.delete(post)
db.commit()
```

---

## 14.2 실습 4 — DELETE /posts/{post_id}

```python
@app.delete("/posts/{post_id}", status_code=204)
def delete_post(post_id: int, db: Session = Depends(get_db)):
    post = _get_post_or_none(db, post_id)

    if not post:
        raise HTTPException(status_code=404, detail="게시글을 찾을 수 없습니다")

    try:
        db.delete(post)
        db.commit()
    except Exception as e:
        db.rollback()
        raise HTTPException(status_code=500, detail=f"게시글 삭제 실패: {str(e)}")
```

---

## 14.3 삭제 흐름

```text
1. 삭제 대상 게시글 조회
2. 없으면 404 반환
3. delete로 삭제 대상으로 표시
4. commit으로 실제 DB에서 삭제
5. 실패 시 rollback
```

---

# 15. CRUD API 전체 요약

| 기능 | Method | Endpoint | 핵심 코드 |
|---|---|---|---|
| 전체 조회 | GET | `/posts` | `db.scalars(select(Post)).all()` |
| 단건 조회 | GET | `/posts/{post_id}` | `db.scalar(select(Post).where(Post.id == post_id))` |
| 생성 | POST | `/posts` | `db.add(post)`, `db.commit()`, `db.refresh(post)` |
| 수정 | PUT | `/posts/{post_id}` | 객체 속성 수정 후 `db.commit()` |
| 삭제 | DELETE | `/posts/{post_id}` | `db.delete(post)`, `db.commit()` |

---

# 16. Day 3-3 — End-to-End 흐름 확인

## 16.1 프로젝트 구조

전체 프로젝트는 `frontend/`와 `backend/`로 분리된다.

```text
fullstack-practice/
├── frontend/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── actions.ts
│   │   └── posts/
│   │       ├── page.tsx
│   │       ├── new/page.tsx
│   │       └── [postId]/
│   │           ├── page.tsx
│   │           └── edit/page.tsx
│   └── .env.local
│
└── backend/
    ├── main.py
    └── blog.db
```

---

## 16.2 프론트엔드 역할

| 파일 | 역할 |
|---|---|
| `layout.tsx` | 공통 레이아웃, 네비게이션 |
| `page.tsx` | `/` 경로. 보통 `/posts`로 리다이렉트 |
| `actions.ts` | Server Action. 서버에서 FastAPI 호출 |
| `posts/page.tsx` | 게시글 목록 |
| `posts/new/page.tsx` | 새 글 작성 폼 |
| `posts/[postId]/page.tsx` | 게시글 상세 |
| `posts/[postId]/edit/page.tsx` | 게시글 수정 폼 |
| `.env.local` | `FASTAPI_URL=http://localhost:8000` 환경변수 |

---

## 16.3 백엔드 역할

| 파일 | 역할 |
|---|---|
| `main.py` | FastAPI 엔드포인트와 SQLAlchemy 모델 정의 |
| `blog.db` | SQLite DB 파일. 실행 중 자동 생성 가능 |

---

# 17. 새 게시글 작성 End-to-End 흐름

## 17.1 전체 요청 흐름

새 게시글 작성 시 일어나는 일은 다음과 같다.

```text
1. 브라우저
   사용자가 /posts/new 폼에 제목과 내용을 입력하고 추가 버튼 클릭

2. Next.js Server Action
   actions.ts의 createPost(formData)가 서버에서 실행

3. FastAPI
   POST /posts 엔드포인트 실행
   Pydantic으로 요청 데이터 유효성 검증

4. SQLAlchemy
   Post 객체 생성
   db.add()
   db.commit()
   db.refresh()

5. SQLite DB
   INSERT INTO posts 실행

6. 응답 반환
   PostResponse JSON 반환
   revalidateTag("posts-list")
   redirect("/posts")

7. 브라우저
   목록 페이지에서 새 게시글 확인
```

---

## 17.2 Server Action에서 FastAPI 호출

```tsx
await fetch("http://localhost:8000/posts", {
  method: "POST",
  body: JSON.stringify({
    title,
    content,
  }),
});
```

실제로는 환경변수를 사용해 URL을 관리한다.

```tsx
await fetch(`${process.env.FASTAPI_URL}/posts`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ title, content }),
});
```

---

## 17.3 FastAPI 처리 흐름

```text
POST /posts
→ Pydantic 유효성 검사
→ Post 객체 생성
→ db.add(post)
→ db.commit()
→ db.refresh(post)
→ PostResponse 반환
```

---

## 17.4 DB 저장 흐름

```sql
INSERT INTO posts (title, content, created_at)
VALUES (...);
```

SQLAlchemy를 사용하면 개발자가 직접 SQL 문자열을 쓰지 않아도 Python 객체 조작이 SQL로 변환된다.

---

## 17.5 응답 이후 Next.js 처리

게시글 생성이 끝나면 Next.js는 다음 작업을 수행한다.

| 작업 | 설명 |
|---|---|
| `revalidateTag("posts-list")` | 게시글 목록 캐시를 무효화해 최신 데이터 반영 |
| `redirect("/posts")` | 작성 완료 후 목록 페이지로 이동 |
| 목록 페이지 재렌더링 | 새 게시글이 포함된 목록 확인 |

---

# 18. Server Action을 쓰면 CORS 에러가 없는 이유

## 18.1 CORS의 의미

CORS는 브라우저가 다른 출처의 서버에 요청할 때 적용되는 보안 정책이다.

예를 들어 브라우저가 `localhost:3000`에서 `localhost:8000`으로 직접 요청하면 서로 다른 출처이기 때문에 CORS 정책의 영향을 받는다.

---

## 18.2 Server Action의 요청 위치

Server Action을 사용하면 브라우저가 FastAPI에 직접 요청하지 않는다.

```text
브라우저
→ Next.js Server Action
→ FastAPI
```

FastAPI 호출은 브라우저가 아니라 Next.js 서버에서 실행된다.  
즉, 서버 간 통신이므로 브라우저의 CORS 정책 대상이 아니다.

---

## 18.3 요청 방식 비교

| 방식 | 요청 주체 | CORS 영향 |
|---|---|---|
| Client Component 직접 fetch | 브라우저 | CORS 설정 필요 |
| Server Component fetch | Next.js 서버 | CORS 영향 없음 |
| Server Action fetch | Next.js 서버 | CORS 영향 없음 |

---

# 19. 프로젝트 실행하기

## 19.1 백엔드 실행

```bash
cd fullstack-practice/backend

uv pip install -r requirements.txt

uv run fastapi dev main.py
```

실행 후 다음 주소를 확인할 수 있다.

| 주소 | 설명 |
|---|---|
| `http://localhost:8000` | FastAPI 서버 |
| `http://localhost:8000/docs` | Swagger UI, API 명세서 |

---

## 19.2 프론트엔드 실행

새 터미널을 열고 실행한다.

```bash
cd fullstack-practice/frontend

npm install

npm run dev
```

실행 후 다음 주소에서 확인한다.

```text
http://localhost:3000
```

---

## 19.3 환경 변수 설정

`frontend/.env.local` 파일을 만들고 다음 내용을 작성한다.

```env
FASTAPI_URL=http://localhost:8000
```

---

## 19.4 직접 확인할 것

```text
1. 백엔드와 프론트엔드를 모두 실행한다.
2. http://localhost:3000 접속
3. 새 게시글 작성
4. 게시글 목록에서 새 게시글 확인
5. 게시글 수정
6. 게시글 삭제
7. Swagger UI에서 DB 변경 확인
```

---

# 20. 4주차까지의 전체 흐름 정리

## 20.1 핵심 흐름

```text
Next.js 프론트
→ Server Action
→ FastAPI 백엔드
→ SQLAlchemy ORM
→ SQLite DB
```

---

## 20.2 계층별 역할

| 계층 | 역할 | 기술 |
|---|---|---|
| 프론트엔드 | 사용자 화면, 폼 입력 | Next.js |
| Server Action | 서버에서 API 호출, 캐시 관리 | Next.js `actions.ts` |
| 백엔드 API | 요청 처리, 비즈니스 로직 | FastAPI |
| ORM | Python 객체와 DB 테이블 연결 | SQLAlchemy |
| 데이터베이스 | 데이터 영구 저장 | SQLite |

---

# 21. 전체 핵심 요약

## 21.1 Next.js 핵심

| 개념 | 핵심 |
|---|---|
| Framework | 정해진 규칙에 따라 앱 구조를 구성 |
| App Router | 폴더와 파일 구조가 URL이 됨 |
| Server Component | 서버에서 실행되어 HTML을 만들어 전달 |
| Client Component | 브라우저에서 인터랙션 처리 |
| Streaming | 준비된 HTML 조각부터 점진적으로 전송 |
| Server Action | 서버에서 실행되는 함수, form과 연결 가능 |
| revalidateTag | 데이터 변경 후 캐시 무효화 |
| redirect | 작업 후 특정 경로로 이동 |

---

## 21.2 SQLAlchemy 핵심

| 개념 | 핵심 |
|---|---|
| ORM | Python 객체와 DB 테이블을 연결 |
| engine | DB 연결 관리 |
| SessionLocal | DB 작업 세션 생성 |
| Base | 모델 선언의 기준 클래스 |
| Model | DB 테이블 구조를 나타내는 Python 클래스 |
| select | 조회 쿼리 생성 |
| db.scalar | 단건 조회 |
| db.scalars(...).all() | 여러 건 조회 |
| db.add | 생성할 객체 등록 |
| db.commit | DB에 변경사항 반영 |
| db.refresh | DB 자동 생성 값을 객체에 반영 |
| db.delete | 삭제할 객체 표시 |
| db.rollback | 실패 시 변경사항 취소 |

---

## 21.3 CRUD 핵심

| 작업 | 흐름 |
|---|---|
| Create | 객체 생성 → `db.add()` → `db.commit()` → `db.refresh()` |
| Read All | `select(Post)` → `db.scalars(...).all()` |
| Read One | `select(Post).where(...)` → `db.scalar()` |
| Update | 객체 조회 → 속성 수정 → `db.commit()` → `db.refresh()` |
| Delete | 객체 조회 → `db.delete()` → `db.commit()` |

---

## 21.4 End-to-End 핵심

```text
사용자가 폼 제출
→ Next.js Server Action 실행
→ FastAPI POST /posts 호출
→ Pydantic 유효성 검사
→ SQLAlchemy가 Post 객체를 DB에 저장
→ SQLite에 실제 INSERT
→ 응답 반환
→ revalidateTag로 캐시 갱신
→ redirect로 목록 페이지 이동
→ 새 게시글 확인
```

---

# 22. 최종 정리

이번 파트는 Next.js와 FastAPI, SQLAlchemy, SQLite가 하나의 흐름으로 연결되는 과정을 이해하는 내용이다.

먼저 Next.js의 기본 구조를 복습했다.  
Next.js는 React 기반 프레임워크이며, App Router에서는 폴더와 파일 구조가 URL이 된다.  
`page.tsx`는 화면, `layout.tsx`는 공통 레이아웃, `loading.tsx`는 로딩 UI를 담당한다.  
Server Component는 서버에서 실행되어 데이터를 가져오고 HTML을 만들어 브라우저에 전달하며, Client Component는 클릭, 입력, 상태 변화처럼 브라우저 인터랙션이 필요한 경우 사용한다.

데이터 조회는 Server Component에서 직접 `fetch`할 수 있다.  
서버에서 실행되기 때문에 환경변수를 안전하게 사용할 수 있고, 브라우저에 백엔드 URL을 노출하지 않을 수 있다.  
데이터 변경 작업은 Server Action으로 처리할 수 있다.  
Server Action은 `"use server"`로 선언한 서버 전용 함수이며, form의 `action`에 연결하면 사용자가 제출한 `FormData`를 받아 서버에서 FastAPI를 호출하고, 작업 후 `revalidateTag`와 `redirect`까지 처리할 수 있다.

다음으로 SQLAlchemy를 배웠다.  
SQLAlchemy는 Python 클래스와 DB 테이블을 연결하는 ORM이다.  
직접 SQL 문자열을 작성하는 대신 Python 객체를 만들고 수정하면, SQLAlchemy가 이를 SQL로 변환해 DB에 반영한다.  
이를 위해 `engine`, `SessionLocal`, `Base`, `Model`이 필요하고, FastAPI에서는 `get_db()`를 통해 요청마다 DB 세션을 주입받는다.

CRUD 실습에서는 전체 조회, 단건 조회, 생성, 수정, 삭제를 구현했다.  
조회는 `select(Post)`, `db.scalar()`, `db.scalars(...).all()`을 사용하고, 생성은 `db.add()`, `db.commit()`, `db.refresh()` 순서로 처리한다.  
수정은 조회한 객체의 속성을 바꾼 뒤 `commit()`하고, 삭제는 `db.delete()` 후 `commit()`한다.  
에러가 발생하면 `rollback()`으로 변경 사항을 되돌려 데이터 안정성을 유지한다.

마지막으로 End-to-End 흐름을 확인했다.  
사용자가 Next.js 화면에서 게시글 작성 폼을 제출하면 Server Action이 실행되고, Next.js 서버가 FastAPI의 `POST /posts`를 호출한다.  
FastAPI는 Pydantic으로 요청 데이터를 검증한 뒤 SQLAlchemy로 `Post` 객체를 생성하고 SQLite DB에 저장한다.  
저장이 끝나면 응답이 반환되고, Next.js는 `revalidateTag("posts-list")`로 캐시를 갱신한 뒤 `/posts`로 이동한다.  
브라우저에서는 목록 페이지에서 새 게시글을 확인할 수 있다.

결국 이 파트의 핵심은 다음과 같다.

```text
1. Next.js는 화면과 서버 로직을 함께 다룰 수 있다.
2. 데이터 조회는 Server Component에서 처리하기 좋다.
3. 데이터 변경은 Server Action과 form action 패턴으로 처리할 수 있다.
4. FastAPI는 요청을 받고 Pydantic으로 검증한 뒤 비즈니스 로직을 수행한다.
5. SQLAlchemy는 Python 객체와 DB 테이블을 연결하는 ORM이다.
6. SQLite는 실제 데이터를 저장한다.
7. 전체 흐름은 Next.js → Server Action → FastAPI → SQLAlchemy → SQLite → 응답 → 화면 업데이트이다.
```

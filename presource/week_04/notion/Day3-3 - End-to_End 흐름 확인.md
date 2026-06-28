## 1️⃣ 프로젝트 구조

```
fullstack-practice/
├── frontend/                    # Next.js 앱
│   ├── app/
│   │   ├── layout.tsx           # 공통 레이아웃 (네비게이션 등)
│   │   ├── page.tsx             # / → /posts 리다이렉트
│   │   ├── actions.ts           # Server Action (서버에서 API 호출)
│   │   └── posts/
│   │       ├── page.tsx             # 게시글 목록
│   │       ├── new/page.tsx         # 새 글 작성 폼
│   │       └── [postId]/
│   │           ├── page.tsx         # 게시글 상세
│   │           └── edit/page.tsx    # 수정 폼
│   └── .env.local           # FASTAPI_URL=http://localhost:8000
│
└── backend/                     # FastAPI 앱
    ├── main.py              # API 엔드포인트 + SQLAlchemy 모델
    └── blog.db              # SQLite DB 파일 (자동 생성)
```

---

## 2️⃣ 전체 요청 흐름

새 게시글을 작성할 때 일어나는 일을 단계별로 살펴봅니다.

```
[1] 브라우저
     └─ 사용자가 /posts/new 폼에 제목과 내용을 입력하고 추가 버튼 클릭

[2] Next.js Server Action (actions.ts)
     └─ createPost(formData) 함수가 서버에서 실행
     └─ fetch("http://localhost:8000/posts", { method: "POST", body: JSON.stringify({title, content}) })

[3] FastAPI (main.py)
     └─ POST /posts 엔드포인트 실행
     └─ Pydantic으로 요청 데이터 유효성 검증

[4] SQLAlchemy
     └─ Post 객체 생성 → db.add() → db.commit() → db.refresh()

[5] SQLite DB (blog.db)
     └─ INSERT INTO posts (title, content, created_at) VALUES (...)

[6] 응답 반환
     └─ PostResponse JSON 반환
     └─ revalidateTag("posts-list") → /posts 리다이렉트

[7] 브라우저
     └─ 목록 페이지에서 새 게시글 확인 ✅
```

#### (참고) Server Action을 쓰면 CORS 에러가 발생하지 않는 이유

브라우저는 보안상 **다른 출처(도메인/포트)로의 요청**을 기본으로 차단합니다. 이를 CORS(Cross-Origin Resource Sharing) 정책이라고 합니다.

현재 구조에서는 브라우저가 아니라 Next.js 서버가 직접 FastAPI 서버(8000번)로 데이터를 요청합니다. 즉, 브라우저가 개입하지 않는 '서버 간 통신'이기 때문에 CORS 설정을 백엔드에 하지 않아도 에러 없이 잘 작동합니다.

---

## **3️⃣** 프로젝트 실행하기

**백엔드 실행**

```bash
cd fullstack-practice/backend

# 1. 필요한 라이브러리 설치
uv pip install -r requirements.txt

# 2. 백엔드 서버 개발 모드로 구동하기
uv run fastapi dev main.py

# → http://localhost:8000 에서 실행됩니다.
# → http://localhost:8000/docs 에서 API 명세서(Swagger UI)를 볼 수 있습니다.
```

**프론트엔드 실행 (새 터미널)**

```bash
cd fullstack-practice/frontend
npm install
npm run dev
# → http://localhost:3000 에서 실행
```

**환경 변수 확인**

`frontend/.env.local` 파일을 만들고 아래 내용을 기입합니다.

```
FASTAPI_URL=http://localhost:8000
```

**직접 확인하기**

1. 백엔드와 프론트엔드를 모두 실행합니다.
2. `http://localhost:3000`에서 새 게시글을 작성하고 확인합니다.
3. 게시글을 수정하고 삭제해봅니다.
4. Swagger UI에서 DB가 실제로 변경되는 것을 확인합니다.

---

## ✅ 4주차 지금까지 배운 내용 정리

> **Next.js(프론트) → Server Action → FastAPI(백엔드) → SQLAlchemy → SQLite(DB)**
> 

각 계층이 무엇을 담당하는지 기억합니다.

| 계층 | 역할 | 기술 |
| --- | --- | --- |
| 프론트엔드 | 사용자 화면, 폼 입력 | Next.js |
| Server Action | 서버에서 API 호출, 캐시 관리 | Next.js (actions.ts) |
| 백엔드 API | 요청 처리, 비즈니스 로직 | FastAPI |
| ORM | Python ↔ DB 변환 | SQLAlchemy |
| 데이터베이스 | 데이터 영구 저장 | SQLite |
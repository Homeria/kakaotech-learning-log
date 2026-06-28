## 1️⃣ Vercel이란?

https://vercel.com/

Vercel은 Next.js를 배포하는 가장 간단한 방법입니다. Next.js를 만든 회사가 직접 운영하는 서비스라 호환성이 좋습니다.

| 특징 | 내용 |
| --- | --- |
| 무료 플랜 | 개인 프로젝트 무료로 배포 가능 |
| 자동 배포 | GitHub Push 시 자동으로 재배포 |
| Preview 배포 | PR을 열면 미리보기 URL 자동 생성 |

배포 후 `https://your-project.vercel.app` 같은 URL이 생성됩니다.

---

## 2️⃣ 실습 프로젝트 구조 파악

지금부터 배포할 프로젝트는 `fullstack-practice`입니다.

```
fullstack-practice/
├── frontend/    ← Vercel에 배포
└── backend/     ← Railway에 배포
```

Vercel은 Next.js 프론트엔드만 배포합니다. FastAPI 백엔드는 **Railway** 별도 서비스로 배포합니다.

```
배포 흐름:
1단계: Railway에 FastAPI 백엔드 배포
2단계: Vercel에 Next.js 프론트엔드 배포
         (환경변수에 Railway URL 입력)
```

---

## 3️⃣ 사전 준비: GitHub 레포지토리 생성

배포를 위해 코드가 GitHub에 올라가 있어야 합니다.

```bash
# 1. 프로젝트 폴더에서
cd fullstack-practice

# 2. Git 초기화 (이미 되어 있으면 스킵)
git init

# 3. 변경사항 스테이징 & 코밋
git add .
git commit -m "initial commit"
git branch -M main

# 4. GitHub에서 새 레포지토리를 만들기
git remote add origin https://github.com/{username}/fullstack-practice
git push -u origin main
```

---

## 4️⃣ Railway란?

<aside>
🚂

Railway는 서버가 항상 켜져 있는 **컨테이너 기반** 호스팅 서비스입니다. Python, Node.js, Go 등 어떤 언어로 만든 서버도 GitHub 레포지토리만 연결하면 배포할 수 있습니다.

</aside>

### 왜 FastAPI 백엔드를 Vercel에 올릴 수 없나요?

Vercel은 **서버리스(Serverless)** 환경입니다. 요청이 들어올 때만 잠깐 실행되고, 요청이 없으면 꺼집니다. 이 구조에서는 SQLite처럼 파일 시스템에 직접 저장하는 DB가 동작하지 않습니다. FastAPI를 Vercel에 올리면 서버는 뜨지만, `blog.db` 파일이 요청마다 초기화되어 데이터가 사라집니다.

Railway는 컨테이너로 서버를 올립니다. 서버가 계속 켜져 있고, 파일 시스템도 유지됩니다. SQLite 파일도 서버가 재시작되지 않는 한 살아있습니다.

### 로컬과 Railway 배포의 차이

| 구분 | 로컬 개발 | Railway 배포 |
| --- | --- | --- |
| 접근 가능 범위 | 내 컴퓨터에서만 | 전 세계 공개 URL |
| FastAPI 주소 | http://localhost:8000 | https://xxx.up.railway.app |
| Vercel에서 호출 가능? | 불가 (localhost는 내 PC) | 가능 (공개 URL) |
| DB(SQLite) 유지 | 항상 유지 | 서버 재시작 전까지 유지 |

### 왜 Railway를 사용하나요?

이번 강의에서 Python과 FastAPI를 배운 이유가 있습니다. 프론트엔드(Next.js)와 백엔드(FastAPI)가 **분리된 실제 서비스 구조**를 직접 경험하는 것이 목표였습니다. Railway에 FastAPI를 올리면 `localhost`에서 벗어나 실제로 전 세계 어디서나 접근 가능한 API 서버가 됩니다. Vercel의 Next.js가 그 Railway URL을 환경변수로 받아 호출하는 것이 오늘 배포 실습의 핵심 흐름입니다.

> **Railway 무료 티어**: Trial 플랜으로 $5 크레딧이 제공됩니다. 이번 실습 범위에서는 충분하지만, 크레딧 소진 후에는 서버가 중단됩니다. 실습 후 사용하지 않는 서비스는 삭제해두는 것을 권장합니다.
> 

---

## 5️⃣ Railway로 FastAPI 백엔드 배포

**단계 1. Railway 가입 & 프로젝트 생성**

https://railway.com/

```
1. 가입, GitHub 로그인 및 코드 접근 허용
2. 레포지토리 선택
- Configure ~ 등을 눌러 GitHub 레포지토리 접근 권한 허용
```

!image.png

!image.png

!image.png

**단계 2. 백엔드 서비스 설정**

```jsx
1. Add Service → GitHub Repo 선택
2. Settings 탭 → Root Directory: backend 입력
3. Deploy 탭 → Start Command 입력:
   fastapi run main.py --host 0.0.0.0 --port $PORT
4. Variables 탭 → CORS_ORIGINS 변수 추가 (나중에 Vercel URL로 업데이트)
5. Deploy 버튼 클릭
```

!image.png

!image.png

!image.png

**단계 3. Railway 도메인 생성**

```jsx
Settings → Networking → Generate Domain 클릭
→ https://next-js-fullstack-production.up.railway.app 같은 URL 생성(아무것도 입력 안 하고 그대로 생성 버튼만 눌러주셔도 됩니다)
→ 이 URL을 복사해 둡니다 (프론트엔드 환경변수에 사용)
```

**CORS 업데이트 필요** — 현재 `main.py`의 CORS는 localhost만 허용합니다. 배포 전에 수정합니다.

```python
import os  # 1. 상단에 os 임포트 추가

# ... 기존 코드 동일 ...

app = FastAPI(title="Blog API")

# 2. 환경 변수에서 프론트엔드 주소를 가져오고, 없으면 로컬 주소를 씁니다.
FRONTEND_URL = os.getenv("FRONTEND_URL", "http://localhost:3000")

# 안전하게 허용할 출처 리스트 생성
origins = [
    "http://localhost:3000",  # 로컬 테스트용은 상시 허용
    FRONTEND_URL,             # 배포된 진짜 프론트엔드 주소 허용
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,    # 3. origins 리스트 주입
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

> Railway Variables 탭에서 `CORS_ORIGINS`에 Vercel URL을 나중에 추가하면 됩니다. 복잡하면 `allow_origins=["*"]`로 임시 설정하는 것도 가능하지만, 실무에서는 크레덴셜 유출 위험이 있습니다.
> 

---

## 6️⃣ Vercel로 Next.js 프론트엔드 배포

- **Vercel 가입 & 프로젝트 가져오기**

!image.png

!image.png

!image.png

- **환경 변수 설정**

```jsx
Environment Variables 섹션에서 아래 변수 3개를 추가합니다:

FASTAPI_URL  = https://your-app.up.railway.app
                ↑ Railway에서 복사한 URL
```

> `FASTAPI_URL`은 Next.js **서버에서만** 사용하는 환경변수입니다. `NEXT_PUBLIC_` 접두사 없이 선언하면 브라우저에 노출되지 않아 안전합니다.
> 

**단계 4. 배포**

```jsx
Deploy 버튼 클릭
→ 빌드 로그 확인 (약 2~3분)
→ 배포 완료: https://your-project.vercel.app URL 생성!
```

**단계 5. Railway CORS 업데이트**

```
Railway 대시보드 → backend 서비스 → Variables
CORS_ORIGINS 업데이트:
  https://your-project.vercel.app,http://localhost:3000
→ Redeploy 클릭
```

---

## 7️⃣ 배포 후 확인

배포가 완료되면 아래를 확인합니다.

```jsx
1. Vercel URL 접속 → 페이지가 로드되는지 확인
2. Railway URL/docs 접속 → Swagger UI가 나오면 백엔드 정상
3. Vercel 앱에서 게시글 목록이 맞게 불러오는지 확인
4. 실제 검색 기능 테스트
```

---

## 8️⃣ 공통 트러블슈팅

배포 후 문제가 생기면 다음을 확인합니다.

**응답이 없습니다 / 500 에러**

```
→ Railway 로그 확인
   - uvicorn 시작 실패? requirements.txt 누락 여부 확인
   - DB 경로 문제? blog.db 파일 생성됨
```

**CORS 에러 (브라우저 콘솔에 CORS 에러)**

```jsx
→ Railway Variables에서 CORS_ORIGINS에 Vercel URL이 있는지 확인
   http와 https 구분 주의
```

**Vercel에서 게시글 목록이 비어 있습니다**

```jsx
→ Vercel 환경변수 FASTAPI_URL 다시 확인
   배포 설정 → Environment Variables에 FASTAPI_URL 입력 여부
   환경변수 변경 후 Redeploy 필요
```

---

## 9️⃣ 실습: 직접 배포해보기

1. `fullstack-practice` 폴더를 GitHub에 코드 Push
2. Railway에 backend 디렉토리로 FastAPI Deploy
3. Railway 도메인 생성 → URL 복사
4. `main.py` CORS 수정 (Railway에 코드 Push 또는 Variables 통해 설정)
5. Vercel에 frontend 디렉토리로 Next.js 배포
6. Vercel 환경변수에 FASTAPI_URL 입력
7. Railway CORS에 Vercel URL 추가
8. 화면에서 검색해보기 확인 ✅
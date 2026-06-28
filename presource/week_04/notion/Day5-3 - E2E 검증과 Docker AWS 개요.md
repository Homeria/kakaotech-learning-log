## 1️⃣ E2E(End-to-End) 검증이란?

E2E 검증은 사용자의 실제 행동 흐름을 처음부터 끝까지 따라가면서 앱 전체가 문제없이 동작하는지 확인하는 과정입니다.

로컬 환경에서는 서버와 DB가 같은 컴퓨터에 있어서 동작하더라도, 배포 후에는 프론트(Vercel)과 백엔드(Railway)가 서로 다른 서버에 있습니다. 이 상태에서 데이터가 정상적으로 오가는지 확인하는 것이 E2E 검증입니다.

---

## 2️⃣ 사용자 시나리오 체크리스트

배포된 Vercel URL에서 아래를 순서대로 수행합니다.

**① 게시글 작성 (POST)**

```
1. https://your-project.vercel.app/posts/new 접속
2. 제목과 내용 입력 후 "작성하기" 클릭
3. 목록 페이지로 자동 이동됨
✓ 확인할 것: 방금 작성한 게시글이 목록에 나타나는지
```

**② 게시글 조회 (GET)**

```
1. 목록에서 방금 작성한 게시글 클릭
2. 상세 페이지 화면 확인
3. Railway URL/docs에서 GET /posts로 DB 저장 여부 직접 확인
✓ 확인할 것: Swagger UI에서 사용자가 입력한 게시글이 있는지
```

**③ 검색 (Query Param)**

```jsx
1. https://your-project.vercel.app/search 접속
2. 검색어 입력 (예: "Next")
3. URL이 /search?q=Next로 변경되는지 확인
4. 브라우저를 새로고침해도 검색 결과가 유지되는지 확인
✓ 확인할 것: URL에 의한 서버 필터링이 작동하는지
```

**④ 게시글 수정 (PUT)**

```
1. 게시글 상세 페이지 → "수정하기" 클릭
2. 제목 또는 내용 변경 후 저장
3. 변경된 내용이 화면에 반영되는지 확인
```

**⑤ 게시글 삭제 (DELETE)**

```
1. 삭제 버튼 클릭 → 승인
2. 목록 페이지로 리다이렉트 확인
3. 삭제한 게시글이 목록에서 사라졌는지 확인
```

---

## 3️⃣ 상태 확인: Railway Swagger UI

실제 DB에 데이터가 실제로 저장되는지 직접 확인합니다.

```jsx
Railway 도메인/docs 접속
  → GET /posts 실행 → Vercel에서 작성한 게시글들이 보이는지 확인

브라우저에서는 Next.js가 동작하고,
Railway URL에서는 FastAPI가 동작하고,
SQLite blog.db에 데이터가 저장됩니다.
```

---

## 4️⃣ Vercel vs Docker/AWS 비교

5주차에서 사용할 Docker/AWS 방식의 배포가 언제 필요한지 Vercel과 비교해봅니다.

| 구분 | Vercel + Railway | Docker + AWS |
| --- | --- | --- |
| **난이도** | 난이도 하 (클릭 몇 번) | 난이도 상 (Linux, Docker, AWS 필요) |
| **설정 시간** | 10~15분 | 1~3시간 |
| **컨트롤** | 제한적 (Vercel 규칙 따름) | 완전 제어 |
| **비용** | 무료플랜 존재 (트래픽 제한) | 인프라 비용 발생 |
| **스케일링** | 자동 (Vercel Edge Network) | 직접 구성 (ECS, ALB, ASG) |
| **DB** | SQLite 파일 유지 가능 (Railway) | RDS/PostgreSQL 주로 사용 |
| **모니터링** | Vercel Analytics | CloudWatch, Prometheus |
| **적합한 상황** | 사이드 프로젝트, 빠른 MVP | 팀 시스템, 고트래픽 실제 서비스 |

---

## 5️⃣ Vercel의 한계 (언제 Docker/AWS가 필요한가)

Vercel은 편리하지만 다음과 같은 상황에서는 한계가 있습니다.

**기능 제약**

```jsx
- 실행 시간 제한: Serverless Function은 10초 제한
  → 실시간 처리, 스트리밍에 부적합

- 실행 환경: Node.js / Python 제한적 지원
  → ML 모델, 대용량 연산 등에 부적합

- 인프라 접근 불가: 자체 서버 없음
  → 특수 환경이나 성능 튜닝 불가
```

**Docker + AWS가 필요한 실제 상황**

```jsx
- 매일 1만 명이상 사용하는 B2B SaaS
- 기업 보안 정책 기준(감사 로그 의무)
- ML 모델 서빙, WebSocket 실시간 기능
- DB가 PostgreSQL이고 수백 GB 이상 데이터
```

---

## 6️⃣ SQLite의 한계

이번 배포에는 SQLite를 사용했습니다. 파일 기반 DB라서 Railway의 지속 스토리지에 저장됩니다.

```jsx
SQLite 사용 시 충돌 리스크:
- 동시에 여러 서버를 늘려도 DB 파일을 공유할 수 없음
- 트래픽이 많으면 쓰기 충돌 발생 가능

프로덕션에서는 PostgreSQL, MySQL 같은
네트워크로 접근하는 DB를 사용합니다.
```

---

## 7️⃣ 그동안 배운 내용 정리

```jsx
1주차~3주차: Python, FastAPI, SQLite, SQLAlchemy로 백엔드 API 구축
4주차:    Next.js로 프론트엔드 구축 + 풀스택 통합
4주 5일차:  Vercel + Railway로 실제 배포 → URL 하나로 버튼 클릭
```

다음주에는 동일한 앱을 **Docker로 컨테이너화**하고 **AWS EC2에 직접 배포**하는 방법을 배웁니다.

| 항목 | Vercel (오늘) | Docker + AWS (다음) |
| --- | --- | --- |
| 인프라 | Vercel이 관리 | 직접 EC2 서버 호스팅 |
| 컨테이너 | 없음 | Docker로 앱 패키징 |
| 배포 방식 | Git Push → 자동 | 이미지 빌드 → EC2에 실행 |
| 학습 목표 | 클라우드 플랫폼 활용법 이해 | DevOps 기본 실무 능력 확보 |
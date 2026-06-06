# Day 5. GitHub 협업 정리

> 원본 PDF: `Day 5. Github 협업.pdf`  
> 주제: GitHub 협업의 중요성, 저장소 생성, Collaborator, Branch 전략, Pull Request, Git LFS, 고급 Git 기능, 충돌 해결

---

# 1. GitHub 협업의 중요성

## 1.1 왜 GitHub 협업이 필요한가?

로컬 컴퓨터 안에서만 Git을 사용할 때는 혼자서 커밋, 브랜치, 병합을 관리하면 된다.  
하지만 실무에서는 여러 개발자가 동시에 같은 프로젝트를 수정한다.

이때 코드를 한 곳에 모으고, 팀원 간의 변경 사항과 충돌을 관리할 수 있는 **원격 협업 플랫폼**이 필요하다.  
GitHub는 이러한 협업을 가능하게 해주는 대표적인 플랫폼이다.

---

## 1.2 Git의 분산 구조와 협업의 장점

| 구분 | 설명 |
|---|---|
| 분산 버전 관리 시스템 | Git은 중앙 서버가 아니라 로컬 저장소 중심으로 작업한다. |
| 전체 이력 보유 | 모든 사용자가 전체 프로젝트 이력을 가지고 있어 오프라인 작업이 가능하다. |
| 백업과 복구 | 여러 로컬 저장소가 존재하므로 백업과 복구가 쉽다. |
| 병렬 개발 | 여러 개발자가 각자의 브랜치에서 동시에 작업할 수 있다. |
| 코드 리뷰 | Pull Request를 통해 팀원 간 코드 리뷰 문화를 만들 수 있다. |
| 충돌 최소화 | 작업을 분리하고 병합 시점에 충돌을 관리할 수 있다. |
| 커밋 이력 관리 | 누가 언제 무엇을 바꿨는지 명확하게 추적할 수 있다. |

---

## 1.3 Git과 GitHub의 차이

Git은 **나만의 일기장**, GitHub는 **모두의 클라우드**라고 이해할 수 있다.

| 구분 | Git | GitHub |
|---|---|---|
| 설치 및 위치 | 내 컴퓨터, Local | 웹 플랫폼, Cloud |
| 주요 목적 | 버전 관리 | 협업 및 공유 |
| 작업 형태 | 터미널 명령어, CLI | 시각적 웹 인터페이스, GUI |
| 핵심 가치 | 코드를 안전하게 저장하자 | 팀원과 코드를 함께 만들자 |

---

## 1.4 Branch로 평행하게 작업하기

브랜치를 사용하면 같은 프로젝트 안에서도 각자의 작업을 분리하여 진행할 수 있다.

예를 들어 프론트엔드 개발자는 `로그인 UI`를 개발하고, 백엔드 개발자는 `로그인 API`를 개발할 수 있다.  
두 사람은 서로의 작업에 영향을 주지 않고 각자의 브랜치에서 평행하게 작업한 뒤, 작업이 끝나면 병합한다.

```text
main
├── feature/login-ui
└── feature/login-api
```

---

## 1.5 GitHub 협업 도구 전체 맵

| 도구 | 역할 |
|---|---|
| Repository | 프로젝트의 집. 코드와 커밋, 브랜치, 히스토리를 중앙에서 관리 |
| Branch | 독립적인 작업 공간. 기능별, 버그별 작업을 분리 |
| Pull Request | 코드 리뷰와 병합을 위한 핵심 절차 |
| Issue | 작업 관리, 버그 추적, 논의 기록 |
| Wiki | 프로젝트 문서화, 셋업 가이드, 코딩 컨벤션 정리 |

---

## 1.6 GitHub 협업 사이클

GitHub의 기능들은 따로 떨어져 있는 것이 아니라 하나의 협업 사이클로 연결된다.

```text
Wiki 읽기
→ Issue 확인
→ Branch 생성
→ 코드 작성
→ Pull Request 생성
→ 코드 리뷰
→ Merge
→ Repository 업데이트
```

즉, GitHub 협업은 단순히 코드를 올리는 과정이 아니라,  
문서 확인, 작업 분배, 브랜치 개발, 리뷰, 병합까지 포함하는 전체 개발 흐름이다.

---

# 2. GitHub 저장소 생성

## 2.1 GitHub 접속과 로그인

GitHub 저장소를 만들기 위해서는 먼저 GitHub에 접속하고 로그인해야 한다.

```text
1. GitHub 접속
2. Sign in 클릭
3. 계정으로 로그인
```

---

## 2.2 Repository란?

Repository는 하나의 코드 저장소이다.  
줄여서 **레포(repo)**라고도 부른다.

| 항목 | 설명 |
|---|---|
| Repository | 하나의 코드 저장소 |
| 로컬 기준 | 하나의 폴더 |
| 일반적인 기준 | 프로젝트 하나 = Repository 하나 |
| 예외 | 기술 스택이나 서비스 구조에 따라 한 프로젝트 안에 여러 Repository를 만들 수도 있음 |

---

## 2.3 Repository 생성 시 설정 항목

GitHub에서 Repository를 생성할 때 주로 다음 항목을 설정한다.

| 항목 | 설명 |
|---|---|
| Template | 기존 템플릿을 기반으로 만들지 선택 |
| Owner | Repository 소유자 또는 조직 |
| Repository name | 저장소 이름 |
| Description | 저장소 설명 |
| Public / Private | 공개 범위 설정 |
| README file initialize | README 파일 자동 생성 여부 |
| `.gitignore` template | Git이 무시할 파일 템플릿 선택 |
| License | 코드 사용, 배포, 수정 조건 지정 |

---

## 2.4 Public과 Private

| 구분 | Public | Private |
|---|---|---|
| 조회 가능 범위 | 누구나 조회 가능 | 특정 사용자만 조회 가능 |
| 장점 | 포트폴리오로 활용 가능, 외부 서비스 연동이 비교적 간편 | 비공개 프로젝트 가능, 외부 노출 방지 |
| 주의점 | 패스워드, API Key 등 민감 정보 절대 업로드 금지 | 협업자는 초대 필요 |
| 사용 예 | 개인 프로젝트, 오픈소스 | 사내 업무, 비공개 팀 프로젝트 |

---

## 2.5 README란?

README는 Repository에 대해 설명하는 문서이다.  
보통 Markdown 문법으로 작성한다.

README에는 다음과 같은 내용을 작성할 수 있다.

| 항목 | 설명 |
|---|---|
| 프로젝트 소개 | 어떤 프로젝트인지 설명 |
| 설치 방법 | 프로젝트를 실행하기 위한 설치 과정 |
| 실행 방법 | 실제 실행 명령어 |
| 사용법 | 주요 기능과 사용 방법 |
| 기여 방법 | 협업자가 참여하는 방식 |
| License | 사용 및 배포 조건 |
| 저작권 정보 | 소유권과 권리 표시 |

README가 잘 작성되어 있으면 사용자와 개발자가 프로젝트를 더 쉽게 이해할 수 있고, 프로젝트의 신뢰도와 완성도도 높아진다.

---

## 2.6 `.gitignore`란?

`.gitignore`는 Git이 추적하지 않아야 하는 파일이나 폴더를 지정하는 파일이다.

| 목적 | 설명 |
|---|---|
| 불필요한 파일 제외 | 빌드 결과물, 캐시, 로그 파일 등을 제외 |
| 보안 유지 | 비밀번호, API Key 등 민감 정보 업로드 방지 |
| 저장소 관리 | Repository를 가볍고 깔끔하게 유지 |
| 템플릿 활용 | 언어와 프레임워크별 `.gitignore` 템플릿 사용 가능 |

예시:

```gitignore
node_modules/
.env
dist/
.DS_Store
```

---

## 2.7 License란?

License는 소프트웨어의 사용, 배포, 수정 권한을 정의한다.

| 항목 | 설명 |
|---|---|
| 사용 권한 | 다른 사람이 코드를 사용할 수 있는지 결정 |
| 수정 권한 | 코드를 변경할 수 있는지 결정 |
| 배포 조건 | 변경한 코드를 어떻게 배포할 수 있는지 규정 |
| 주의 사항 | 타인의 코드를 사용할 때는 반드시 License를 확인해야 함 |

개인 공부나 간단한 프로젝트에서는 지정하지 않아도 되지만, 공개 프로젝트나 협업 프로젝트에서는 License 확인이 중요하다.

---

# 3. Git 활용 워크플로우

## 3.1 로컬 저장소와 원격 저장소의 동기화

GitHub 협업에서는 로컬 저장소와 원격 저장소가 동기화를 통해 상호작용한다.

일반적인 흐름은 다음과 같다.

```text
1. 로컬 저장소에서 작업
2. git add와 git commit으로 변경 사항 저장
3. git push로 원격 저장소에 업로드
4. git pull로 원격 저장소의 최신 변경 사항을 로컬에 반영
5. git fetch로 원격 저장소의 상태 확인
```

---

## 3.2 주요 명령어

| 명령어 | 설명 |
|---|---|
| `git add` | 변경 사항을 스테이징 영역에 올림 |
| `git commit` | 변경 사항을 로컬 저장소에 기록 |
| `git push` | 로컬 커밋을 원격 저장소로 전송 |
| `git pull` | 원격 저장소의 최신 변경 사항을 가져와 병합 |
| `git fetch` | 원격 저장소의 상태를 확인하고 가져오지만 자동 병합은 하지 않음 |

---

# 4. GitHub 저장소 복제와 연결

## 4.1 Clone repository

`git clone`은 원격 저장소에 있는 Git Repository를 로컬 컴퓨터로 복사하는 명령어이다.

```bash
git clone <repository-url>
```

GitHub에서 Clone할 때는 다음 순서로 진행한다.

```text
1. GitHub Repository에서 Code 버튼 클릭
2. HTTPS 주소 복사
3. 터미널에서 git clone <주소> 실행
```

---

## 4.2 현재 작업 폴더를 GitHub Repository에 연결하기

이미 로컬에서 작업하던 폴더를 원격 Repository에 연결하려면 다음 명령어를 사용할 수 있다.

```bash
echo "# test" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/elice/test.git
git push -u origin main
```

---

## 4.3 `git remote add origin`

```bash
git remote add origin https://github.com/elice/test.git
```

이 명령어는 로컬 저장소와 원격 저장소를 연결한다.  
`origin`은 원격 저장소의 단축 이름이다.

| 요소 | 의미 |
|---|---|
| `git remote add` | 원격 저장소 추가 |
| `origin` | 원격 저장소의 별칭 |
| URL | 연결할 GitHub Repository 주소 |

---

# 5. Collaborator

## 5.1 Collaborator란?

Collaborator는 Repository에 함께 작업할 수 있도록 초대된 팀원을 의미한다.

Collaborator로 등록되면 Repository를 직접 수정할 수 있고, GitHub 웹에서 바로 commit과 push도 가능하다.

---

## 5.2 Collaborator 등록 방법

```text
Repository
→ Settings
→ Collaborators
→ 팀원 초대
→ 초대 받은 사람이 수락
```

---

## 5.3 Collaborator 협업 시 주의점

팀원이 같은 원격 저장소를 직접 수정할 수 있기 때문에, 항상 최신 상태를 유지해야 한다.

```bash
git pull
```

최신 버전이 유지되지 않으면 충돌이 발생하여 원활한 협업이 어려워진다.

| 주의 사항 | 설명 |
|---|---|
| 작업 전 pull | 원격 저장소의 최신 변경 사항을 먼저 반영 |
| 작업 후 push | 내 변경 사항을 원격 저장소에 업로드 |
| 충돌 확인 | 같은 파일을 수정한 경우 충돌 가능 |
| 소통 필요 | 같은 영역을 동시에 수정하지 않도록 역할 분담 |

---

# 6. Branch 전략과 Merge

## 6.1 Git Flow란?

Git Flow는 중대형 프로젝트에 적합한 브랜치 전략이다.  
기능 개발과 유지보수를 위해 여러 종류의 브랜치를 나누어 사용하는 방식이다.

---

## 6.2 Git Flow 주요 브랜치

| 브랜치 이름 | 설명 |
|---|---|
| `main` 또는 `master` | 제품 출시 버전을 관리하는 브랜치 |
| `develop` | 다음 출시 버전을 개발하는 브랜치 |
| `feature` | 새로운 기능을 개발할 때 사용하는 브랜치 |
| `release` | 다음 출시 버전을 준비하는 브랜치 |
| `hotfix` | 긴급한 버그 수정이 필요할 때 사용하는 브랜치 |

---

## 6.3 Git Flow 브랜치 흐름

| 단계 | 브랜치 | 핵심 역할 | 흐름 |
|---|---|---|---|
| 1 | `develop` | 개발 최신 코드 시작 | 개발 기준 브랜치 |
| 2 | `feature` | 기능 개발 | `develop` → `feature` |
| 3 | `feature` 병합 | 기능 완료 | `feature` → `develop` |
| 4 | `release` | 릴리즈 준비 | `develop` → `release` |
| 5 | `release` 병합 | 안정화 | `release` → `develop` |
| 6 | `main` | 배포 | `develop` → `main` |
| 7 | `main` | 안정 코드 | 배포 버전 유지 |
| 8 | `hotfix` | 긴급 수정 | `main` → `hotfix` → `main` |

---

## 6.4 Feature 개발 흐름

기능 개발은 보통 `develop` 브랜치에서 `feature` 브랜치를 분기하여 진행한다.

```bash
git clone <repo-url>
git checkout -b feature/login develop
git add .
git commit -m "feat: add login feature"
npm test
git push origin feature/login
```

이후 GitHub에서 Pull Request를 생성하고, 리뷰 후 `develop` 브랜치에 병합한다.

---

## 6.5 Release 흐름

Release는 계획된 배포를 준비하는 과정이다.

```bash
git checkout -b release/1.2.0 develop
# QA / 스테이징 테스트
git commit -m "fix: release bug"
git checkout main
git merge release/1.2.0
git tag v1.2.0
git checkout develop
git merge release/1.2.0
```

| 단계 | 설명 |
|---|---|
| release 분기 | `develop`에서 `release` 브랜치 생성 |
| 베타 테스트 | QA, 스테이징 배포 검증 |
| 버그 픽스 | release 브랜치에서 수정 |
| main 머지와 태그 | 운영 반영 및 버전 태그 생성 |
| develop 백머지 | 수정 사항을 develop에도 반영 |

---

## 6.6 Hotfix 흐름

Hotfix는 운영 중 긴급한 버그를 수정하는 과정이다.  
`develop`이 아니라 `main`에서 출발한다.

```bash
git checkout -b hotfix/login-crash main
git commit -m "fix: login crash"
git checkout main
git merge hotfix/login-crash
git tag v1.2.1
git checkout develop
git merge hotfix/login-crash
```

| 단계 | 설명 |
|---|---|
| 운영 버그 발견 | Sentry, 고객 리포트 등으로 발견 |
| hotfix 분기 | `main`에서 긴급 수정 브랜치 생성 |
| 수정 및 검증 | 최소 변경으로 문제 해결 |
| main 머지와 태그 | 즉시 배포 |
| develop 백머지 | 동일 버그 재발 방지 |

---

# 7. 브랜치 네이밍 가이드

## 7.1 기본 규칙

브랜치 이름은 목적과 작업 범위를 쉽게 파악할 수 있도록 작성해야 한다.

| 규칙 | 설명 |
|---|---|
| 소문자 사용 | 브랜치 이름은 소문자로 작성 |
| 하이픈 사용 | 단어 구분은 하이픈 사용 |
| 공백 금지 | 공백과 특수문자 사용하지 않음 |
| 짧고 명확하게 | 작업 내용을 간결하게 표현 |

---

## 7.2 브랜치 유형과 접두사

| 유형 | 접두사 | 설명 |
|---|---|---|
| 기능 개발 | `feature/` | 새로운 기능 개발 |
| 버그 수정 | `bugfix/` | 일반 버그 수정 |
| 긴급 수정 | `hotfix/` | 프로덕션 환경 긴급 버그 또는 보안 수정 |
| 릴리즈 준비 | `release/` | 릴리즈 버전 준비 및 QA |
| 실험 / 테스트 | `experiment/` | 실험적 기능 개발 |
| 문서 작업 | `docs/` | 문서 수정 및 추가 |

---

## 7.3 네이밍 패턴

```text
type/short-description-issue-number
```

| 구성 | 설명 |
|---|---|
| `type` | 브랜치 유형 접두사 |
| `short-description` | 작업 내용을 간결하게 설명 |
| `issue-number` | 선택 사항. GitHub나 Jira 이슈 번호 |

---

## 7.4 브랜치 이름 예시

| 작업 | 브랜치 이름 예시 |
|---|---|
| 로그인 기능 추가 | `feature/login-api-45` |
| 장바구니 수량 오류 수정 | `bugfix/cart-quantity-102` |
| 서버 응답 지연 긴급 수정 | `hotfix/server-latency-210` |
| 1.2.0 버전 릴리즈 준비 | `release/1.2.0` |
| API 문서 업데이트 | `docs/api-update-88` |

---

# 8. Merge 전략

## 8.1 Merge란?

Merge는 한 브랜치의 변경 사항을 다른 브랜치로 통합하는 과정이다.  
Git에서는 두 개의 브랜치를 하나로 합칠 때 merge를 사용한다.

---

## 8.2 왜 브랜치 병합 전략이 필요한가?

| 이유 | 설명 |
|---|---|
| 동시 작업 효율성 | 독립 브랜치에서 개발하고 병합 시점에 충돌 해결 |
| 브랜치 목적 명확화 | main, develop, feature 등 역할 분리 |
| 환경별 브랜치 분리 | 배포용, 테스트용, 기능 개발용 브랜치 구분 |
| 버전 관리 | 안정 브랜치를 기준으로 롤백 가능 |
| 비상 대응 | hotfix 등을 통해 신속한 긴급 수정 가능 |

---

## 8.3 Branch Merge Strategy

GitHub에서 브랜치를 메인 브랜치로 병합할 때 여러 전략을 선택할 수 있다.  
프로젝트 특성과 팀의 워크플로우에 맞게 선택해야 한다.

| 병합 전략 | 설명 | 커밋 히스토리 | 단점 |
|---|---|---|---|
| Merge Commit | 모든 커밋 히스토리를 보존하면서 병합 | 유지됨 | 히스토리가 복잡해질 수 있음 |
| Squash and Merge | 여러 커밋을 하나의 커밋으로 압축하여 병합 | 압축됨 | 작업 세부 기록이 사라짐 |
| Rebase and Merge | 커밋들을 베이스 브랜치 위로 재배치 | 변형됨 | 충돌 발생 시 복잡하고 커밋 해시가 변경됨 |

---

# 9. Branch 생성과 Pull Request 실습 흐름

## 9.1 Branch 생성하기

```bash
# 현재 브랜치 확인
git branch

# 새 브랜치 만들고 바로 이동
git switch -c feature/readme

# 기존 방식
git checkout -b feature/readme

# 브랜치 목록 확인
git branch
```

| 명령어 | 설명 |
|---|---|
| `git branch` | 브랜치 목록 확인 |
| `git switch -c <브랜치명>` | 새 브랜치 생성 후 이동 |
| `git checkout -b <브랜치명>` | 새 브랜치 생성 후 이동하는 기존 방식 |

`git switch -c`는 Git 2.23 이후 도입된 권장 명령어이고, `git checkout -b`는 기존 방식이다.

---

## 9.2 README 편집하기

```bash
code README.md
```

README 파일을 열고 내용을 수정한 뒤 저장한다.

Markdown 빠른 문법:

```markdown
# 제목
**굵게**
- 목록
[링크](url)
```

---

## 9.3 변경사항 확인과 Commit

```bash
git status
git diff
git add README.md
git commit -m "docs: update README title"
```

| 명령어 | 설명 |
|---|---|
| `git status` | 현재 변경 상태 확인 |
| `git diff` | 라인 단위 변경 내용 확인 |
| `git add <파일>` | 변경 사항을 스테이징 |
| `git commit -m "메시지"` | 변경 사항을 커밋으로 기록 |

커밋은 로컬 브랜치에만 저장되며, 아직 원격 GitHub에는 반영되지 않는다.

---

## 9.4 원격 저장소에 Push

```bash
git remote -v
git push -u origin feature/readme
```

| 명령어 | 설명 |
|---|---|
| `git remote -v` | 연결된 원격 저장소 확인 |
| `git push -u origin <브랜치명>` | 로컬 브랜치를 원격 저장소로 업로드하고 추적 관계 설정 |

`-u` 또는 `--set-upstream` 옵션은 로컬 브랜치와 원격 브랜치를 연결한다.  
한 번 설정하면 이후에는 `git push`, `git pull`만으로 동기화할 수 있다.

---

# 10. Fork와 Pull Request

## 10.1 Fork란?

Fork는 내용이 동일한 Repository를 내 계정으로 복사하는 것이다.

Fork한 Repository에서 수정해도 원래 Repository에는 바로 반영되지 않는다.  
원본 Repository에 변경 사항을 반영하려면 Pull Request를 생성해야 한다.

---

## 10.2 Fork 기반 협업 흐름

```text
원본 Repository
→ Fork
→ 내 계정의 복사본 Repository
→ 수정 및 Commit
→ Push
→ Pull Request 생성
→ 원본 관리자가 Review
→ Merge
→ 원본 Repository 변경
```

---

# 11. Pull Request

## 11.1 Pull Request란?

Pull Request, 줄여서 PR은 브랜치에서 작업한 코드 변경 사항을 팀에 공유하고, 병합 전에 검토를 받는 협업 절차이다.

PR은 코드 리뷰의 핵심 도구이며, 팀원에게 내 코드를 설명하고 피드백을 주고받는 소통의 장이다.

---

## 11.2 Pull Request가 필요한 이유

| 이유 | 설명 |
|---|---|
| 코드 품질 향상 | 리뷰를 통해 버그, 스타일, 로직 오류를 사전에 발견 |
| 협업 안전성 확보 | 직접 main 브랜치에 병합하지 않아 안정성 보장 |
| 기록 추적 | PR에 남는 토론과 리뷰 기록으로 변경 이유와 결정 과정 추적 |
| 소통 강화 | 코드 변경 내용을 팀원과 공유하고 의견을 주고받음 |

---

## 11.3 Pull Request 단계

| 단계 | 설명 |
|---|---|
| 1단계 | 작업 브랜치 생성 |
| 2단계 | 코드 작업 완료 후 로컬에서 커밋 및 푸시 |
| 3단계 | GitHub에서 Pull Request 생성 |
| 4단계 | 팀원들이 변경 내용을 확인하고 코멘트와 피드백 제공 |
| 5단계 | 필요 시 리뷰를 반영해 다시 커밋 및 푸시 |
| 6단계 | 리뷰어 승인 후 main 또는 develop 브랜치에 병합 |

---

## 11.4 Pull Request 생성 시 확인할 항목

| 항목 | 설명 |
|---|---|
| base | 최종적으로 변경 사항이 들어가야 하는 브랜치 |
| compare | 병합하려는 작업 브랜치 |
| description | 어떤 변경이 있는지 설명 |
| reviewer | 필요한 경우 리뷰어 지정 |
| 관련 Issue | 연결된 작업이나 버그 이슈 |

---

## 11.5 Pull Request 만들기

GitHub에서 PR을 만드는 기본 순서는 다음과 같다.

```text
1. 상단바에서 Pull requests 클릭
2. Compare & pull request 클릭
3. base와 compare 브랜치 확인
4. 설명 작성
5. 필요 시 reviewer 지정
6. Create pull request 클릭
7. 충돌이 없다면 Merge pull request 클릭
```

---

# 12. PR 작성 매너

## 12.1 코드 리뷰 기본 원칙

| 원칙 | 설명 |
|---|---|
| 모든 코드는 리뷰 후 머지 | main/develop에 직접 반영하지 않음 |
| 리뷰 책임은 팀 전체 | 특정 사람만의 책임이 아님 |
| 최소 1명 승인 필수 | 병합 전 리뷰어 승인 필요 |
| 빠른 응답 | PR 요청 시 하루 안에 답변하는 것이 좋음 |
| 큰 PR은 오프라인 리뷰 | 변경량이 많으면 별도 리뷰 시간 확보 |
| 사실과 데이터로 의견 제시 | 감정이 아니라 근거 중심으로 리뷰 |

---

## 12.2 PR 요청 시 매너

| 항목 | 설명 |
|---|---|
| 변경 내용 설명 | 무엇을 왜 바꿨는지 충분히 작성 |
| PR 제목 | `(키워드) 작업 내용 #PR번호` 형식 등 일관성 있게 작성 |
| 본문 구성 | 작업 동기, 변경 내용, 주안점, 연관 티켓, 체크리스트 포함 |
| UI 변경 | 스크린샷 또는 GIF 첨부 |
| 작은 단위 | 너무 큰 PR보다 작은 단위로 생성 |
| 주석과 커밋 | 리뷰어가 이해할 수 있게 충실히 작성 |
| 관련 코드 포함 | 중복 없이 필요한 코드만 포함 |

---

# 13. Git LFS

## 13.1 기존 Git의 구조적 한계

Git은 커밋마다 전체 트리를 스냅샷으로 기록한다.  
변경되지 않은 파일은 기존 객체를 참조하므로 텍스트 파일은 효율적으로 관리된다.

하지만 이미지, 영상, 바이너리 파일처럼 diff 효율이 떨어지는 파일은 작은 수정에도 거의 전체가 새 객체로 저장된다.  
그 결과 저장소 용량이 급격히 커질 수 있다.

---

## 13.2 대용량 파일 관리 문제

| 문제 | 설명 |
|---|---|
| 저장소 용량 급증 | 단일 대용량 파일이 전체 Repository 크기를 크게 늘림 |
| 작업 속도 저하 | Commit, Push, Clone 시간이 파일 크기에 비례해 증가 |
| 불필요한 히스토리 공유 | 모든 팀원이 원하지 않는 대용량 파일 이력까지 다운로드해야 함 |

---

## 13.3 Git LFS란?

Git LFS는 **Large File Storage**의 약자이다.  
Git이 효율적으로 처리하지 못하는 대용량 파일을 다루기 위한 공식 확장 도구이다.

| 개념 | 설명 |
|---|---|
| 외부 격리 저장 | 실제 대용량 파일은 Git 저장소 외부에 별도로 보관 |
| 포인터 파일 | Git 내부 히스토리에는 원본 파일 위치와 해시값만 담긴 작은 파일 저장 |
| 목적 | 대용량 파일로 인한 저장소 비대화 방지 |

---

## 13.4 Git LFS 동작 구조

```text
로컬 저장소
→ 대용량 파일의 참조 정보만 Git에 커밋

원격 Git 서버
→ 참조 정보만 저장

LFS 서버
→ 실제 대용량 파일 저장
```

---

## 13.5 Git LFS 설치

| 운영체제 | 설치 명령어 | 초기화 |
|---|---|---|
| Mac OS | `brew install git-lfs` | `git lfs install` |
| Windows | `choco install git-lfs` 또는 `winget install git-lfs` | `git lfs install` |
| Linux, Debian/Ubuntu | `sudo apt-get update` 후 `sudo apt-get install git-lfs` | `git lfs install` |

---

## 13.6 Git LFS 특징

| 특징 | 설명 |
|---|---|
| 대용량 파일 버전 관리 | GB 단위 파일도 관리 가능 |
| 저장소 크기 감소 | Git에는 포인터만 저장되어 Repository 크기 감소 |
| Git 워크플로우 유지 | 기존 Git 흐름을 유지하면서 대용량 파일 관리 |
| 다양한 호스팅 지원 | GitHub, GitLab, Bitbucket 등에서 지원 |

---

## 13.7 Git LFS 대안 비교

| 기능 | Git LFS | Git Submodule | Git-annex |
|---|---|---|---|
| 대용량 파일 지원 | 가능 | 불가능 | 가능 |
| 히스토리 단순성 | 좋음, 포인터 방식 | 보통 | 보통 |
| 사용 난이도 | 쉬움 | 중간 | 어려움 |
| 협업 적합도 | 높음 | 중간 | 낮음, 복잡함 |

---

## 13.8 Git LFS 실무 활용 정리

| 활용 | 설명 |
|---|---|
| 대용량 파일 버전 관리 | `.mp4`, `.psd`, `.zip`, `.csv` 등 관리 |
| 포인터 기반 저장 | Git에는 포인터만 저장, 실제 파일은 LFS 서버 저장 |
| 협업 시 자동 동기화 | `git pull` 시 LFS 파일도 함께 내려받음 |
| 빠른 clone/push | 파일을 나중에 개별 다운로드해 속도 개선 가능 |
| CI/CD 연동 | `git lfs pull`로 LFS 파일을 받아 테스트/배포 활용 |
| 추적 상태 확인 | `git lfs ls-files`, `.gitattributes`로 확인 |
| 용량 관리 | `git lfs prune`으로 불필요한 캐시 정리 |

---

## 13.9 Git LFS 사용 시 고려사항

| 고려사항 | 설명 |
|---|---|
| 스토리지 제한 | 호스팅 서비스별 저장 용량 제한 확인 |
| 트래픽 제한 | 다운로드 트래픽 제한 확인 |
| 서버 지원 여부 | Git LFS 지원 서버인지 확인 |
| 팀원 환경 | 공동 작업자 모두 `git lfs install` 필요 |
| CI/CD 연동 | 배포 환경에서 LFS 파일을 내려받는 설정 필요 |

---

# 14. 고급 Git 기능

## 14.1 주요 고급 기능

| 기능 | 설명 |
|---|---|
| `git stash` | 작업 중인 변경 사항을 임시 저장하고 복원 |
| `git cherry-pick` | 특정 커밋만 골라 다른 브랜치에 적용 |
| `git rebase` | 커밋 히스토리를 깔끔하게 정리 |
| `git reflog` | 모든 이력 기록 보기. 삭제된 커밋도 복구 가능 |
| `git bisect` | 이진 탐색으로 버그 원인 커밋 추적 |
| `git tag` | 특정 커밋에 버전 식별자 부여 |

---

## 14.2 Git Stash

`git stash`는 작업 중이던 변경 사항을 임시로 저장하고 브랜치를 이동할 수 있게 해준다.

브랜치 전환 시 아직 커밋하지 않은 미완성 작업을 잠시 보관할 때 유용하다.

| 명령어 | 설명 |
|---|---|
| `git stash` | 변경 사항 임시 저장 |
| `git stash list` | 저장된 stash 목록 보기 |
| `git stash apply` | 마지막 stash 적용, 목록에서는 삭제되지 않음 |
| `git stash pop` | stash 적용 후 목록에서 제거 |

---

## 14.3 Git Tag

`git tag`는 커밋에 버전 태그를 지정하여 릴리즈 지점을 명확히 표시하는 기능이다.

주로 `main` 브랜치의 릴리즈 커밋에 사용한다.

| 명령어 | 설명 |
|---|---|
| `git tag v1.0.0` | 로컬에 태그 추가 |
| `git push origin v1.0.0` | 원격 저장소에 태그 푸시 |

---

## 14.4 Git Cherry-pick

`git cherry-pick`은 특정 커밋만 골라 다른 브랜치에 반영할 수 있는 기능이다.  
Hotfix나 긴급 수정 사항을 다른 브랜치에 빠르게 반영할 때 유용하다.

```bash
git cherry-pick <commit-hash>
```

---

## 14.5 Git Reflog

`git reflog`는 브랜치, HEAD, reset 등 Git 내부 이동 기록을 추적할 수 있게 한다.  
실수로 브랜치를 삭제하거나 reset했을 때 복구에 유용하다.

```bash
git reflog
```

---

## 14.6 Git Bisect

`git bisect`는 이진 탐색 방식으로 버그가 발생한 커밋을 추적한다.  
대규모 프로젝트에서 어떤 커밋이 문제를 만들었는지 찾을 때 유용하다.

```bash
git bisect start
```

---

# 15. 충돌 해결하기

## 15.1 Merge Conflict란?

Merge Conflict는 같은 파일의 같은 라인을 두 브랜치가 서로 다르게 수정했을 때 발생한다.  
Git이 자동으로 합칠 수 없기 때문에 사람이 직접 어떤 내용을 남길지 결정해야 한다.

---

## 15.2 충돌이 자주 발생하는 상황

| 상황 | 설명 |
|---|---|
| 같은 줄 동시 수정 | 두 사람이 같은 파일의 같은 줄을 서로 다르게 수정 |
| 같은 함수 리팩토링 | 같은 함수를 두 브랜치가 각자 다르게 정리 |
| 삭제 vs 수정 | 한쪽은 파일을 삭제하고 다른 쪽은 해당 파일을 수정 |
| 머지 작업 중 자동 병합 실패 | 가장 흔한 충돌 상황 |

---

## 15.3 충돌 마커

Git이 자동으로 충돌 파일에 삽입하는 마커는 다음과 같다.

```text
<<<<<<< HEAD
현재 브랜치의 코드
=======
병합하려는 브랜치의 코드
>>>>>>> branch-name
```

| 마커 | 의미 |
|---|---|
| `<<<<<<< HEAD` | 충돌 시작. 현재 브랜치의 코드 시작 |
| `=======` | 현재 브랜치 코드와 가져온 코드의 구분선 |
| `>>>>>>> branch-name` | 충돌 끝. 병합하려는 상대 브랜치 표시 |

---

## 15.4 VSCode 충돌 해결 옵션

| 옵션 | 설명 |
|---|---|
| Accept Current Change | 내 변경, 즉 HEAD만 살리고 가져온 변경은 버림 |
| Accept Incoming Change | 가져온 변경만 살리고 내 변경은 버림 |
| Accept Both Changes | 양쪽 변경을 모두 남김. 순서와 중복은 직접 정리 필요 |
| Compare Changes | 좌우 비교 화면에서 차이를 확인하며 직접 편집 |

---

## 15.5 3-way merge editor

VSCode의 3-way merge editor는 충돌을 세 영역으로 보여준다.

| 영역 | 설명 |
|---|---|
| Current | 내 변경 |
| Incoming | 가져온 변경 |
| Result | 최종 결과. 직접 편집하는 영역 |

---

# 16. 브랜치 전환 전 주의사항

## 16.1 자주 하는 실수

작업한 변경이 아직 커밋되지 않은 상태에서 다른 브랜치로 이동하려고 하면 문제가 생길 수 있다.

Git은 변경 사항이 덮어써질 위험이 있으면 브랜치 전환을 거부한다.

```bash
git switch other-branch

error: Your local changes to the following files would be overwritten by checkout:
 README.md
Please commit your changes or stash them before you switch branches.
```

---

## 16.2 해결 방법 1: 커밋하고 전환

```bash
git add .
git commit -m "wip: 작업 중"
git switch other-branch
```

이 방법은 작업 내용을 정식 커밋으로 기록한다.  
히스토리에 남아도 되는 작업이라면 적절하다.

---

## 16.3 해결 방법 2: stash하고 전환

```bash
git stash
git switch other-branch

# 다시 돌아왔을 때
git stash pop
```

이 방법은 커밋 없이 변경 사항을 임시 보관한다.  
아직 정식 커밋으로 남기기 애매한 작업일 때 적절하다.

---

## 16.4 브랜치 전환 전 체크

브랜치를 이동하기 전에는 항상 현재 작업 트리가 깔끔한지 확인해야 한다.

```bash
git status
```

VSCode에서는 좌측 Source Control 패널에서도 변경 상태를 확인할 수 있다.

---

# 17. 전체 핵심 요약

## 17.1 GitHub 협업 핵심

| 개념 | 핵심 |
|---|---|
| Git | 로컬에서 버전을 관리하는 도구 |
| GitHub | Git 저장소를 온라인에서 공유하고 협업하는 플랫폼 |
| Repository | 프로젝트의 코드 저장소 |
| Branch | 독립적으로 작업하기 위한 분기 |
| Pull Request | 코드 리뷰와 병합을 위한 절차 |
| Issue | 작업과 버그를 관리하는 기록 공간 |
| Wiki | 프로젝트 설명서와 지식 저장소 |

---

## 17.2 GitHub 협업 흐름

```text
Repository 생성
→ 팀원 Collaborator 초대
→ Issue로 작업 관리
→ Branch 생성
→ 코드 작성 및 Commit
→ Push
→ Pull Request 생성
→ Review
→ Merge
→ 최신 코드 Pull
```

---

## 17.3 Branch 전략 핵심

| 브랜치 | 역할 |
|---|---|
| `main` | 배포 가능한 안정 버전 |
| `develop` | 다음 출시를 위한 개발 기준 |
| `feature/*` | 기능 개발 |
| `bugfix/*` | 일반 버그 수정 |
| `hotfix/*` | 운영 긴급 수정 |
| `release/*` | 배포 준비 |
| `docs/*` | 문서 작업 |

---

## 17.4 PR 핵심

PR은 단순히 코드를 합치는 요청이 아니라,  
변경 내용을 설명하고, 리뷰를 받고, 프로젝트 품질을 높이는 협업 절차이다.

좋은 PR은 다음 조건을 갖춘다.

```text
- 변경 목적이 명확하다.
- 작업 범위가 작고 분명하다.
- 관련 Issue나 티켓이 연결되어 있다.
- 리뷰어가 이해할 수 있도록 설명이 충분하다.
- UI 변경이 있으면 스크린샷이나 GIF가 첨부되어 있다.
```

---

## 17.5 Git LFS 핵심

Git LFS는 대용량 파일을 Git 저장소에 직접 저장하지 않고,  
Git에는 포인터 파일만 저장하고 실제 파일은 LFS 서버에 보관하는 방식이다.

이미지, 영상, 디자인 파일, 압축 파일, 대용량 CSV 등 일반 Git으로 관리하기 비효율적인 파일을 다룰 때 유용하다.

---

## 17.6 충돌 해결 핵심

충돌은 협업에서 자연스럽게 발생할 수 있다.  
중요한 것은 충돌 자체를 두려워하는 것이 아니라, 충돌 마커를 이해하고 어떤 코드를 최종 결과로 남길지 판단하는 것이다.

브랜치 전환 전에는 항상 다음을 확인한다.

```bash
git status
```

작업 중인 변경 사항이 있다면 다음 중 하나를 선택한다.

```bash
git add .
git commit -m "wip: 작업 중"
```

또는

```bash
git stash
git switch other-branch
git stash pop
```

---

# 18. 최종 정리

GitHub 협업은 단순히 코드를 업로드하는 과정이 아니라, 팀이 같은 코드베이스를 안전하게 관리하고 개선하는 전체 흐름이다.  
Git은 로컬에서 버전을 관리하는 도구이고, GitHub는 그 Git 저장소를 온라인에 올려 팀원들과 함께 작업할 수 있게 해주는 협업 플랫폼이다.

협업에서는 Repository, Branch, Pull Request, Issue, Wiki가 서로 연결되어 사용된다.  
작업자는 Issue로 할 일을 확인하고, Branch에서 독립적으로 개발한 뒤, Pull Request를 통해 코드 리뷰를 받고 Merge한다.  
이 과정에서 README, `.gitignore`, License 같은 기본 문서와 설정도 프로젝트의 완성도와 협업 효율을 높이는 중요한 요소가 된다.

브랜치 전략은 프로젝트의 규모와 배포 방식에 따라 달라질 수 있다.  
Git Flow에서는 `main`, `develop`, `feature`, `release`, `hotfix` 브랜치를 사용하여 기능 개발, 배포 준비, 긴급 수정 흐름을 분리한다.  
또한 Merge Commit, Squash and Merge, Rebase and Merge 같은 병합 전략을 팀 규칙에 맞게 선택해야 한다.

대용량 파일은 일반 Git으로 관리하면 저장소가 무거워질 수 있으므로 Git LFS를 사용할 수 있다.  
Git LFS는 실제 파일 대신 포인터를 Git에 저장하여 대용량 파일을 효율적으로 관리한다.

마지막으로 협업 중 충돌은 충분히 발생할 수 있다.  
충돌이 발생하면 VSCode의 충돌 해결 기능이나 3-way merge editor를 활용해 내 변경, 가져온 변경, 최종 결과를 비교하며 정리하면 된다.  
브랜치를 전환하기 전에는 반드시 `git status`로 작업 트리가 깨끗한지 확인하는 습관이 중요하다.

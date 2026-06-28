# Day 5 - 클라우드 DB 연동 상세 정리

> 카카오테크캠퍼스 활동 자료 정리본  
> 원본 파일: `Day 5 - 클라우드 DB 연동.md`  
> 실습 기준 코드: Day 2 정답 코드 기반 `todo-fastapi-compose-student/`

---

## 0. 먼저 알아둘 점

원본 자료의 파일명은 **Day 5 - 클라우드 DB 연동**이지만, 본문에는 `Day 4`라는 표현이 여러 번 등장한다. 내용상 핵심은 다음과 같다.

> Day 2에서 Docker Compose로 함께 띄웠던 **로컬 MySQL 컨테이너** 대신, **AWS RDS MySQL**을 FastAPI 앱에 연결하는 실습이다.

즉, 이번 활동의 주제는 **애플리케이션 코드를 크게 바꾸는 것**이 아니라, 애플리케이션이 사용하는 **DB 연결 대상**을 로컬 컨테이너에서 클라우드 DB로 바꾸는 것이다.

---

## 1. 오늘의 전체 목표

이번 실습의 최종 목표는 다음 한 문장으로 정리할 수 있다.

> 로컬 Docker Compose 환경에서 실행되는 FastAPI 앱이, 로컬 MySQL 컨테이너가 아니라 AWS RDS MySQL에 데이터를 저장하도록 연결 설정을 바꾸고, SQL로 실제 저장 여부를 확인한다.

### 기존 구조와 변경 후 구조

| 구분 | Day 2 구조 | 이번 실습 구조 |
|---|---|---|
| 애플리케이션 | FastAPI 컨테이너 | FastAPI 컨테이너 |
| DB | 로컬 MySQL 컨테이너 | AWS RDS MySQL |
| DB 주소 | `db:3306` | `<rds-endpoint>:3306` |
| 연결 방식 | Docker Compose 내부 네트워크 | 인터넷/RDS 보안 그룹/DB 계정 |
| 확인 방식 | `docker exec todo-db ...` | MySQL client 컨테이너로 RDS 접속 |

원본 자료의 구조 비교는 다음과 같다.

```text
Day 2
[FastAPI container] ---> [MySQL container]
        web                    db

이번 실습
[FastAPI container] ---> [AWS RDS MySQL]
        web                    <rds-endpoint>:3306
```

여기서 중요한 변화는 FastAPI 컨테이너 자체가 바뀌는 것이 아니라, FastAPI가 바라보는 **DB 주소**가 바뀐다는 점이다.

---

## 2. 학습 목표 3가지

| 번호 | 목표 | 쉽게 말하면 |
|---|---|---|
| 1 | RDS endpoint, port, database name, username, password가 각각 무엇인지 설명할 수 있다. | DB에 접속하려면 어떤 정보가 필요한지 이해한다. |
| 2 | Day 2 FastAPI 앱의 `DATABASE_URL`을 바꿔 RDS MySQL에 연결할 수 있다. | 앱 코드가 아니라 환경변수를 바꿔 DB 연결 대상을 바꾼다. |
| 3 | MySQL client 컨테이너로 RDS에 접속해 `SELECT` 문으로 데이터를 확인할 수 있다. | 앱에서 생성한 데이터가 실제 RDS에 저장됐는지 SQL로 확인한다. |

---

## 3. 시작 전 복습 키워드

이번 실습은 Day 1~3에서 배운 내용을 이어서 사용하는 활동이다.

| 키워드 | 다시 떠올릴 내용 | 이번 실습에서 쓰이는 곳 |
|---|---|---|
| YAML | 들여쓰기로 구조를 표현하는 설정 파일 형식 | `docker-compose.yml`에서 환경변수와 서비스 설정 확인 |
| Docker | 앱 실행 환경을 컨테이너로 묶는 도구 | FastAPI 앱 컨테이너 실행, MySQL client 컨테이너 실행 |
| Docker Compose | 여러 컨테이너를 함께 실행하는 도구 | Day 2의 FastAPI + MySQL 구조 복습 및 앱 실행 |
| RDS | AWS가 운영을 도와주는 관리형 데이터베이스 | 로컬 MySQL 컨테이너 대신 사용할 클라우드 DB |

### 실습 폴더

이번 활동에서 사용하는 폴더는 다음과 같다.

```bash
todo-fastapi-compose-student/
```

이 폴더는 Day 2 정답 코드 기반이다. 따라서 이번 실습은 Compose 파일을 처음부터 완성하는 활동이 아니라, 이미 동작하던 앱에서 **DB 연결 문자열을 바꿔보는 활동**에 가깝다.

---

## 4. Day 2 복습: 왜 `db`로 접속했을까?

Day 2에서 사용한 DB 연결 문자열은 다음과 같은 형태였다.

```text
DATABASE_URL=mysql+pymysql://todo_user:todo_pass@db:3306/todo_db
```

여기서 `db`는 실제 인터넷 도메인이 아니라, `docker-compose.yml` 안에 정의된 **service 이름**이다.

```yaml
services:
  db:
    image: mysql:8.0

  web:
    environment:
      DATABASE_URL: mysql+pymysql://todo_user:todo_pass@db:3306/todo_db
```

Docker Compose는 실행될 때 자동으로 내부 네트워크를 만든다. 그리고 같은 Compose 네트워크 안에서는 service 이름을 hostname처럼 사용할 수 있다.

그래서 `web` 컨테이너 입장에서는 다음 주소로 MySQL에 접속할 수 있었다.

```text
db:3306
```

이를 풀어서 말하면 다음과 같다.

| 구성 요소 | 의미 |
|---|---|
| `web` | FastAPI 앱이 실행되는 컨테이너 |
| `db` | MySQL이 실행되는 컨테이너의 Compose service 이름 |
| `3306` | MySQL 기본 접속 포트 |
| `todo_db` | 접속할 MySQL 데이터베이스 이름 |
| `todo_user` | DB 사용자 이름 |
| `todo_pass` | DB 사용자 비밀번호 |

이번 실습에서는 이 `db` 자리에 **AWS RDS endpoint**가 들어간다.

---

## 5. RDS 연결에 필요한 6가지 값

AWS RDS MySQL에 연결하려면 다음 값들이 필요하다.

| 값 | 의미 | 예시 | 실습에서 확인할 것 |
|---|---|---|---|
| `endpoint` | RDS 접속 주소 | `ktc-day4-demo.xxxxxx.ap-northeast-2.rds.amazonaws.com` | 정확히 복사했는지 확인 |
| `port` | MySQL 접속 포트 | `3306` | `:3306`이 빠지지 않았는지 확인 |
| `database name` | 내가 사용할 DB 이름 | `ktc_001_db` | 본인에게 배정된 DB 이름인지 확인 |
| `username` | DB 사용자 이름 | `ktc_001` | 본인 계정인지 확인 |
| `password` | DB 비밀번호 | 제공된 임시 비밀번호 | 공백 없이 정확히 입력했는지 확인 |
| `security group` | 누가 DB에 접속할 수 있는지 정하는 AWS 방화벽 | 3306 허용 규칙 | 접속이 안 될 때 가장 먼저 의심 |

### RDS 연결 문자열 예시

```text
DATABASE_URL=mysql+pymysql://ktc_001:<password>@<rds-endpoint>:3306/ktc_001_db
```

각 부분을 분해하면 다음과 같다.

| 부분 | 예시 | 의미 |
|---|---|---|
| DB 드라이버 | `mysql+pymysql` | SQLAlchemy가 MySQL에 PyMySQL 드라이버로 접속한다는 의미 |
| 사용자 이름 | `ktc_001` | RDS MySQL 사용자 계정 |
| 비밀번호 | `<password>` | 해당 사용자 계정의 비밀번호 |
| 호스트 | `<rds-endpoint>` | RDS 인스턴스의 접속 주소 |
| 포트 | `3306` | MySQL 서버 포트 |
| DB 이름 | `ktc_001_db` | 사용할 데이터베이스 이름 |

### `DATABASE_URL` 형식 이해하기

일반적인 형식은 다음과 같다.

```text
DB종류+드라이버://사용자이름:비밀번호@호스트:포트/데이터베이스명
```

이번 실습에 맞추면 다음과 같다.

```text
mysql+pymysql://ktc_001:<password>@<rds-endpoint>:3306/ktc_001_db
```

### 주의할 점

| 주의사항 | 이유 |
|---|---|
| `<password>`는 본인에게 제공된 값으로 바꿔야 한다. | 그대로 두면 인증에 실패한다. |
| `<rds-endpoint>`도 실제 RDS endpoint로 바꿔야 한다. | placeholder는 실제 주소가 아니다. |
| 비밀번호가 들어간 `DATABASE_URL`을 GitHub에 올리면 안 된다. | DB 접속 정보가 유출될 수 있다. |
| 문자열 중간에 공백이 들어가면 안 된다. | 연결 문자열 파싱 또는 인증이 실패할 수 있다. |
| DB 이름 끝까지 정확히 입력해야 한다. | DB 이름이 틀리면 `Unknown database`가 발생할 수 있다. |

---

## 6. 실습 0: 준비 확인

### 필요한 것

| 준비물 | 설명 |
|---|---|
| Docker Desktop 실행 | 컨테이너 실행을 위해 필요 |
| `todo-fastapi-compose-student/` 폴더 | Day 2 정답 코드 기반 실습 자료 |
| 제공된 RDS 접속 정보 | endpoint, username, password, DB name 등 |
| 브라우저 접속 가능 환경 | FastAPI docs 또는 todo 화면 확인용 |

### 실습 폴더로 이동

```bash
cd todo-fastapi-compose-student
```

### 기존 컨테이너 상태 확인

```bash
docker compose ps
```

이 명령어는 현재 Compose 프로젝트 안에서 실행 중인 컨테이너를 보여준다.

| 상태 | 의미 |
|---|---|
| 아무것도 안 보임 | 아직 컨테이너가 실행되지 않았거나 다른 폴더에 있음 |
| `web`, `db` 등이 보임 | 기존 Day 2 실습 컨테이너가 실행 중일 수 있음 |
| `Exited` 상태 | 컨테이너가 실행됐다가 종료됨 |

---

## 7. 실습 1: Day 2 로컬 DB에서 `SELECT` 해보기

먼저 Day 2에서 했던 로컬 MySQL 컨테이너 구조를 복습한다.

### 컨테이너 실행

```bash
docker compose up -d
docker compose ps
```

`-d`는 detached mode, 즉 백그라운드 실행을 의미한다.

### MySQL 컨테이너 내부로 들어가기

```bash
docker exec -it todo-db mysql -u todo_user -ptodo_pass todo_db
```

이 명령어는 `todo-db` 컨테이너 안에서 MySQL 클라이언트를 실행해 로컬 DB에 접속한다.

| 옵션/값 | 의미 |
|---|---|
| `docker exec` | 실행 중인 컨테이너 안에서 명령 실행 |
| `-it` | 터미널에서 상호작용 가능하게 실행 |
| `todo-db` | MySQL 컨테이너 이름 |
| `mysql` | MySQL CLI 실행 |
| `-u todo_user` | DB 사용자 이름 지정 |
| `-ptodo_pass` | DB 비밀번호 지정. `-p`와 비밀번호 사이에 공백 없음 |
| `todo_db` | 접속할 DB 이름 |

### MySQL shell에서 실행할 SQL

```sql
SHOW TABLES;
SELECT * FROM todos;
SELECT COUNT(*) AS todo_count FROM todos;
EXIT;
```

각 SQL의 의미는 다음과 같다.

| SQL | 의미 |
|---|---|
| `SHOW TABLES;` | 현재 DB에 어떤 테이블이 있는지 확인 |
| `SELECT * FROM todos;` | `todos` 테이블의 모든 데이터 조회 |
| `SELECT COUNT(*) AS todo_count FROM todos;` | todo 데이터 개수 확인 |
| `EXIT;` | MySQL shell 종료 |

이 단계의 구조는 여전히 다음과 같다.

```text
web container ---> db container
```

즉, 아직 AWS RDS에 연결한 상태가 아니라 Day 2 로컬 DB 구조를 확인하는 단계다.

---

## 8. 실습 2: RDS 연결 문자열로 바꾸기

이번 실습의 핵심 단계다. FastAPI 앱이 사용하는 DB 연결 문자열을 로컬 MySQL 컨테이너에서 RDS MySQL로 바꾼다.

### `.env.example`을 `.env`로 복사

Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

macOS/Linux:

```bash
cp .env.example .env
```

`.env.example`은 예시 설정 파일이고, `.env`는 실제 실행에 사용할 설정 파일이다.

일반적으로 `.env`에는 비밀번호 같은 민감정보가 들어갈 수 있으므로 GitHub에 올리지 않는 것이 원칙이다.

### 기존 `DATABASE_URL`

```text
DATABASE_URL=mysql+pymysql://todo_user:todo_pass@db:3306/todo_db
```

이 값은 로컬 Docker Compose 내부의 `db` 서비스에 접속하는 설정이다.

### RDS용 `DATABASE_URL`

```text
DATABASE_URL=mysql+pymysql://ktc_001:<password>@<rds-endpoint>:3306/ktc_001_db
```

여기서 반드시 본인에게 배정된 값으로 바꿔야 한다.

### 변경 후 체크리스트

| 확인 항목 | 확인 내용 |
|---|---|
| 사용자 이름 | `ktc_001` 부분을 본인 계정으로 바꿨는가? |
| 비밀번호 | `<password>`를 본인 비밀번호로 바꿨는가? |
| endpoint | `<rds-endpoint>`를 정확히 복사했는가? |
| DB 이름 | 마지막의 `ktc_001_db`가 본인 DB 이름인가? |
| 포트 | `:3306`이 빠지지 않았는가? |
| 공백 | URL 중간에 불필요한 공백이 들어가지 않았는가? |
| Git 관리 | `.env`가 Git에 올라가지 않도록 되어 있는가? |

---

## 9. 실습 3: 앱 컨테이너 재실행

`.env` 파일을 수정했다고 해서 이미 실행 중인 컨테이너가 자동으로 새 값을 읽는 것은 아니다. 설정을 바꾼 뒤에는 컨테이너를 재실행해야 한다.

### 기본 재실행 명령

```bash
docker compose down
docker compose up -d --build web
docker compose logs -f web
```

각 명령의 의미는 다음과 같다.

| 명령 | 의미 |
|---|---|
| `docker compose down` | 기존 Compose 컨테이너와 네트워크를 내림 |
| `docker compose up -d --build web` | `web` 서비스를 다시 빌드하고 백그라운드 실행 |
| `docker compose logs -f web` | `web` 컨테이너 로그를 실시간으로 확인 |

### `web`만 실행하기 어렵다면

```bash
docker compose up -d --build
docker compose logs -f web
```

이 경우 Compose에 정의된 서비스 전체를 다시 실행한다.

### 로그 확인 포인트

| 로그 상태 | 해석 |
|---|---|
| DB 연결 에러 없이 서버가 뜸 | RDS 연결이 정상일 가능성이 높음 |
| `Access denied` | username/password 문제 가능성 |
| `Unknown database` | DB 이름 오타 가능성 |
| `timed out` | 보안 그룹 또는 네트워크 접근 문제 가능성 |
| 계속 `db:3306`에 접속하려고 함 | `.env` 변경이 적용되지 않았을 가능성 |

로그 확인을 종료하려면 다음 키를 누른다.

```text
Ctrl + C
```

---

## 10. 실습 4: FastAPI에서 todo 생성

앱 컨테이너가 정상적으로 실행되면 브라우저에서 FastAPI 문서 페이지에 접속한다.

```text
http://localhost:8000/docs
```

또는 수업 자료에 todo 화면이 따로 제공되어 있다면 그 화면을 사용한다.

### 진행 순서

| 순서 | 작업 | 확인할 것 |
|---|---|---|
| 1 | todo 1개 생성 | 요청이 에러 없이 성공하는가? |
| 2 | todo 목록 조회 | 방금 만든 todo가 조회되는가? |
| 3 | 앱 응답 확인 | FastAPI가 정상 응답하는가? |
| 4 | 다음 실습에서 SQL 확인 | 실제 RDS DB에 저장되었는지 검증 |

이 단계에서 브라우저 화면에서 todo가 보인다고 해서 끝이 아니다. 다음 단계에서 MySQL client 컨테이너로 RDS에 직접 접속해 실제 저장 여부를 확인한다.

---

## 11. 실습 5: MySQL client 컨테이너로 RDS 접속

RDS는 로컬 Docker Compose의 `todo-db` 컨테이너가 아니다.

따라서 RDS에 접속할 때는 다음 방식이 아니다.

```bash
docker exec -it todo-db mysql ...
```

대신 MySQL client가 들어있는 컨테이너를 **잠깐 실행**해서 RDS에 접속한다.

### Windows PowerShell 한 줄 명령

```powershell
docker run --rm -it mysql:8.0 mysql -h <rds-endpoint> -P 3306 -u ktc_001 -p ktc_001_db
```

PowerShell에서는 줄바꿈 문법 때문에 헷갈릴 수 있으므로 원본 자료에서는 한 줄 실행을 권장한다.

### macOS/Linux 여러 줄 명령

```bash
docker run --rm -it mysql:8.0 mysql \
  -h <rds-endpoint> \
  -P 3306 \
  -u ktc_001 \
  -p \
  ktc_001_db
```

### 명령어 구성 설명

| 부분 | 의미 |
|---|---|
| `docker run` | 새 컨테이너 실행 |
| `--rm` | 컨테이너 종료 후 자동 삭제 |
| `-it` | 터미널 상호작용 가능 |
| `mysql:8.0` | MySQL client가 포함된 이미지 |
| `mysql` | 컨테이너 안에서 실행할 MySQL CLI |
| `-h <rds-endpoint>` | 접속할 RDS 주소 |
| `-P 3306` | 접속 포트. 대문자 `P` 사용 |
| `-u ktc_001` | DB 사용자 이름 |
| `-p` | 비밀번호를 프롬프트에서 입력 |
| `ktc_001_db` | 접속할 DB 이름 |

비밀번호 입력창이 나오면 본인 비밀번호를 입력한다.

> 보안상 명령어에 비밀번호를 직접 적는 것보다 `-p`만 입력하고 프롬프트에서 입력하는 방식이 더 안전하다.

### MySQL shell에서 확인할 SQL

```sql
SELECT DATABASE();
SELECT CURRENT_USER();
SHOW TABLES;
SELECT * FROM todos;
SELECT COUNT(*) AS todo_count FROM todos;
EXIT;
```

각 SQL의 의미는 다음과 같다.

| SQL | 확인하는 것 |
|---|---|
| `SELECT DATABASE();` | 현재 접속 중인 DB 이름 |
| `SELECT CURRENT_USER();` | 현재 접속한 DB user |
| `SHOW TABLES;` | 현재 DB에 있는 테이블 목록 |
| `SELECT * FROM todos;` | FastAPI 앱에서 생성한 todo 데이터 |
| `SELECT COUNT(*) AS todo_count FROM todos;` | todo 개수 |
| `EXIT;` | MySQL shell 종료 |

성공하면 FastAPI에서 생성한 todo가 RDS MySQL의 `todos` 테이블에 저장된 것을 확인할 수 있다.

---

## 12. 실습 6: 출석 check-in 해보기

RDS에 접속한 상태에서 수업용 check-in 데이터를 남긴다.

### check-in INSERT

```sql
INSERT INTO classroom.checkins (student_id, nickname, db_user)
VALUES ('001', 'neo', CURRENT_USER());
```

각 컬럼의 의미는 다음과 같다.

| 컬럼 | 의미 | 예시 |
|---|---|---|
| `student_id` | 본인 식별용 번호 | `'001'` |
| `nickname` | 수업에서 보여져도 괜찮은 닉네임 | `'neo'` |
| `db_user` | 현재 접속한 DB 사용자 | `CURRENT_USER()` |

### 주의사항

| 항목 | 주의할 내용 |
|---|---|
| `student_id` | 본인 배정 번호로 바꾼다. 또는 겹치지 않을 목적으로 핸드폰 번호 뒷자리 4자리를 사용할 수 있다. |
| `nickname` | 수업에서 공개되어도 괜찮은 이름을 사용한다. |
| 개인정보 | email 전체나 민감한 개인정보는 넣지 않는다. |

### 최근 check-in 조회

```sql
SELECT student_id, nickname, db_user, created_at
FROM classroom.checkins
ORDER BY created_at DESC
LIMIT 20;
```

이 SQL은 가장 최근에 등록된 check-in 20개를 최신순으로 보여준다.

| 절 | 의미 |
|---|---|
| `SELECT student_id, nickname, db_user, created_at` | 보고 싶은 컬럼 선택 |
| `FROM classroom.checkins` | `classroom` DB/schema의 `checkins` 테이블에서 조회 |
| `ORDER BY created_at DESC` | 생성 시간이 최신인 것부터 정렬 |
| `LIMIT 20` | 최대 20개만 조회 |

---

## 13. 자주 만나는 에러 정리

| 에러 메시지/증상 | 가장 흔한 원인 | 먼저 확인할 것 | 쉽게 이해하기 |
|---|---|---|---|
| `connection timed out` | RDS 보안 그룹에서 3306이 열려 있지 않음 | RDS security group 상태 확인 | DB 서버까지 네트워크 연결 자체가 안 되는 상태 |
| `Access denied` | username/password 오류 | 본인 계정과 비밀번호 확인 | DB에 도착은 했지만 로그인 실패 |
| `Unknown database` | DB 이름 오타 | `ktc_001_db` 같은 본인 DB 이름 확인 | 로그인은 됐지만 지정한 DB가 없음 |
| `Unknown MySQL server host` | endpoint 오타 | RDS endpoint 다시 복사 | RDS 주소를 DNS가 찾지 못함 |
| 앱은 뜨지만 데이터 저장 실패 | 테이블이 아직 없음 | 로그 확인, 테이블 생성 방식 확인 | 서버는 켜졌지만 DB 테이블 준비가 안 됨 |
| 계속 로컬 DB에 저장됨 | `DATABASE_URL` 변경이 적용되지 않음 | 컨테이너 재실행, Compose 설정 확인 | 아직 `db:3306`을 보고 있을 가능성 |
| `mysql` 명령어가 없음 | web 컨테이너에는 MySQL CLI가 없음 | `docker run --rm -it mysql:8.0 mysql ...` 사용 | 앱 컨테이너 안에는 MySQL client가 없을 수 있음 |

### 에러를 구분하는 간단한 기준

| 상황 | 가능성이 큰 문제 |
|---|---|
| 접속이 오래 걸리다가 실패 | 보안 그룹, 네트워크, endpoint 문제 |
| 바로 `Access denied` | 계정/비밀번호 문제 |
| 바로 `Unknown database` | DB 이름 문제 |
| 앱이 계속 로컬 데이터만 보여줌 | `.env` 변경 미적용 또는 컨테이너 재실행 누락 |

---

## 14. Day 2와 이번 실습 핵심 비교

| 구분 | Day 2 | 이번 실습 |
|---|---|---|
| DB 위치 | 로컬 MySQL 컨테이너 | AWS RDS MySQL |
| DB 주소 | `db` | RDS endpoint |
| 포트 | `3306` | `3306` |
| 확인 방법 | `docker exec todo-db` | MySQL client 컨테이너 실행 |
| 접근 제어 | Docker Compose network | RDS security group + DB user |
| 운영 관점 | 개발 환경 | 클라우드 운영 환경에 가까움 |
| 설정 변경 핵심 | Compose service 이름 사용 | RDS endpoint 사용 |

가장 중요한 한 줄은 다음과 같다.

> 애플리케이션은 환경변수만 바꿔도 다른 DB에 연결할 수 있고, 운영 환경에서는 그 연결을 계정과 네트워크로 통제한다.

---

## 15. 이번 실습에서 꼭 이해해야 하는 흐름

### 1단계: 로컬 구조 확인

```text
FastAPI web container ---> MySQL db container
```

Day 2에서 사용한 구조다. `web`은 Compose 내부 hostname인 `db`로 접속한다.

### 2단계: `.env`의 `DATABASE_URL` 변경

```text
mysql+pymysql://todo_user:todo_pass@db:3306/todo_db
```

위 값을 다음과 같이 바꾼다.

```text
mysql+pymysql://ktc_001:<password>@<rds-endpoint>:3306/ktc_001_db
```

### 3단계: 컨테이너 재실행

```bash
docker compose down
docker compose up -d --build web
```

이미 실행 중인 컨테이너가 예전 환경변수를 들고 있을 수 있기 때문에 재실행이 필요하다.

### 4단계: FastAPI에서 todo 생성

```text
http://localhost:8000/docs
```

브라우저에서 todo를 생성하고 조회한다.

### 5단계: RDS에 직접 접속해서 SQL 확인

```bash
docker run --rm -it mysql:8.0 mysql -h <rds-endpoint> -P 3306 -u ktc_001 -p ktc_001_db
```

그 뒤 SQL로 확인한다.

```sql
SELECT * FROM todos;
```

---

## 16. 수업 후 정리 및 cleanup 체크리스트

### 로컬 컨테이너 정리

```bash
docker compose down
```

이 명령은 실행 중인 Compose 컨테이너를 내린다.

### 반드시 지킬 것

| 항목 | 이유 |
|---|---|
| `DATABASE_URL`을 GitHub에 올리지 않기 | DB endpoint, username, password가 포함될 수 있음 |
| RDS endpoint와 password를 공개 채팅방에 남기지 않기 | 외부인이 DB 접속을 시도할 수 있음 |
| 수업용 비밀번호는 수업 후 더 이상 사용하지 않기 | 임시 실습용 계정이므로 장기 사용에 적합하지 않음 |
| 실습 DB 접근 권한이 회수될 수 있음을 이해하기 | 수업용 RDS는 일정 기간 후 접근이 막힐 수 있음 |
| check-in에 민감한 개인정보를 넣지 않기 | 다른 사람에게 조회될 수 있는 데이터일 수 있음 |

---

## 17. 원본 이미지 자료가 설명하는 내용

원본 Markdown에는 이미지 파일명이 포함되어 있지만, 본문 파일만 제공된 상태에서는 이미지 자체가 함께 포함되어 있지 않다. 각 이미지가 가리키는 역할은 본문 문맥상 다음과 같다.

| 원본 이미지 파일명 | 설명하는 내용 |
|---|---|
| `day4-01-review-keywords.png` | Day 1~3 복습 키워드 맵: YAML, Docker, Docker Compose, RDS |
| `day4-02-local-vs-rds-architecture.png` | Day 2 로컬 DB 구조와 RDS 연결 구조 비교 |
| `day4-04-rds-connection-card.png` | RDS 연결에 필요한 6가지 값 정리 |
| `day4-03-database-url-breakdown.png` | `DATABASE_URL` 구성 요소 해부 |
| `day4-06-practice-flow.png` | RDS 실습 단계별 흐름도 |
| `day4-07-mysql-client-container.png` | MySQL client 컨테이너로 RDS에 접속하는 구조 |
| `day4-09-checkin-flow.png` | check-in 테이블 실시간 조회 흐름 |
| `day4-05-security-group-gate.png` | RDS 보안 그룹에서 3306 허용 여부에 따른 접속 가능/불가능 비교 |
| `day4-10-cleanup-checklist.png` | 실습 후 정리 체크리스트 |

---

## 18. 실습 명령어 모음

### 로컬 DB 컨테이너 실행 및 확인

```bash
docker compose up -d
docker compose ps
```

### 로컬 MySQL 컨테이너 접속

```bash
docker exec -it todo-db mysql -u todo_user -ptodo_pass todo_db
```

### 로컬 DB SQL 확인

```sql
SHOW TABLES;
SELECT * FROM todos;
SELECT COUNT(*) AS todo_count FROM todos;
EXIT;
```

### `.env` 복사

Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

macOS/Linux:

```bash
cp .env.example .env
```

### RDS 연결 문자열 예시

```text
DATABASE_URL=mysql+pymysql://ktc_001:<password>@<rds-endpoint>:3306/ktc_001_db
```

### 앱 컨테이너 재실행

```bash
docker compose down
docker compose up -d --build web
docker compose logs -f web
```

또는:

```bash
docker compose up -d --build
docker compose logs -f web
```

### FastAPI 접속

```text
http://localhost:8000/docs
```

### RDS 접속: Windows PowerShell

```powershell
docker run --rm -it mysql:8.0 mysql -h <rds-endpoint> -P 3306 -u ktc_001 -p ktc_001_db
```

### RDS 접속: macOS/Linux

```bash
docker run --rm -it mysql:8.0 mysql \
  -h <rds-endpoint> \
  -P 3306 \
  -u ktc_001 \
  -p \
  ktc_001_db
```

### RDS SQL 확인

```sql
SELECT DATABASE();
SELECT CURRENT_USER();
SHOW TABLES;
SELECT * FROM todos;
SELECT COUNT(*) AS todo_count FROM todos;
EXIT;
```

### check-in 입력

```sql
INSERT INTO classroom.checkins (student_id, nickname, db_user)
VALUES ('001', 'neo', CURRENT_USER());
```

### check-in 조회

```sql
SELECT student_id, nickname, db_user, created_at
FROM classroom.checkins
ORDER BY created_at DESC
LIMIT 20;
```

### 로컬 컨테이너 정리

```bash
docker compose down
```

---

## 19. 최종 요약

이번 활동은 **FastAPI 코드 구조를 새로 짜는 실습**이 아니라, 이미 Docker Compose로 동작하던 앱의 DB 연결 대상을 바꾸는 실습이다.

Day 2에서는 다음처럼 로컬 MySQL 컨테이너에 연결했다.

```text
web ---> db:3306
```

이번 실습에서는 다음처럼 AWS RDS MySQL에 연결한다.

```text
web ---> <rds-endpoint>:3306
```

핵심은 `DATABASE_URL`이다.

```text
DATABASE_URL=mysql+pymysql://사용자:비밀번호@DB주소:포트/DB이름
```

운영 환경에 가까워질수록 애플리케이션 코드는 그대로 두고, DB 주소, 계정, 비밀번호, 보안 그룹 같은 **환경 설정과 접근 제어**를 통해 인프라를 연결하고 보호한다.

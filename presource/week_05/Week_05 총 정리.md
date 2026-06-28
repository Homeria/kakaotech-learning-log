# 카카오테크캠퍼스 주간 총정리: Docker에서 AWS RDS 연동까지

> 이 문서는 Day 1, Day 2, Day 3, Day 5 정리본을 바탕으로 한 **한 주 전체 흐름 요약본**입니다.  
> 세부 명령어와 모든 예외 상황을 전부 나열하기보다는, 이번 주 활동에서 반드시 이어서 이해해야 하는 핵심 개념과 실습 흐름을 중심으로 정리했습니다.

---

## 1. 이번 주 전체 흐름 한눈에 보기

이번 주의 큰 흐름은 **로컬에서 컨테이너로 앱을 실행하는 방법을 배우고, 그 앱을 클라우드 인프라와 연결하는 방향으로 확장하는 것**이었다.

```text
Day 1: Docker 컨테이너 기초
  → 앱 실행 환경을 이미지/컨테이너로 포장하고 실행하는 법

Day 2: Docker Compose
  → FastAPI 앱 + MySQL DB처럼 여러 컨테이너를 함께 실행하는 법

Day 3: AWS 인프라 개요
  → EC2, S3, RDS 같은 클라우드 리소스와 VPC/보안 그룹 이해

Day 5: 클라우드 DB 연동
  → 로컬 MySQL 컨테이너 대신 AWS RDS MySQL에 FastAPI 앱 연결
```

전체 구조 변화는 다음처럼 볼 수 있다.

```text
[1단계] 단일 컨테이너 실행
Dockerfile → Image → Container

[2단계] 다중 컨테이너 로컬 개발 환경
FastAPI container ── Docker Compose network ── MySQL container

[3단계] 클라우드 인프라 이해
EC2 / S3 / RDS / VPC / IAM / Security Group

[4단계] 로컬 앱 + 클라우드 DB 연동
FastAPI container ── Internet/RDS Endpoint ── AWS RDS MySQL
```

---

## 2. Day 1 핵심: Docker 컨테이너 기초

Day 1의 핵심은 Docker 명령어 자체보다 **왜 컨테이너가 필요한지**를 이해하는 것이었다.

### 2.1 컨테이너가 해결하려는 문제

개발에서 자주 발생하는 문제는 다음과 같다.

> “내 컴퓨터에서는 잘 되는데, 다른 환경에서는 왜 안 되지?”

주된 원인은 OS, 라이브러리, 패키지 버전, 설정 파일, 실행 방식이 환경마다 다르기 때문이다. Docker는 애플리케이션뿐 아니라 실행 환경까지 함께 묶어 **어디서나 비슷하게 실행되도록 만드는 도구**다.

### 2.2 VM과 컨테이너 비교

| 구분 | 가상머신 VM | 컨테이너 |
|---|---|---|
| 가상화 방식 | 하드웨어 수준 가상화 | OS 수준 가상화 |
| OS | 각 VM마다 Guest OS 포함 | Host OS 커널 공유 |
| 무게 | 무겁고 부팅이 느림 | 가볍고 실행이 빠름 |
| 격리 수준 | 강함 | VM보다는 약하지만 충분히 격리 가능 |
| 사용 감각 | 컴퓨터 한 대를 새로 띄움 | 격리된 프로세스 실행 환경을 띄움 |

컨테이너는 `chroot`, `namespace`, `cgroup` 같은 리눅스 기능을 조합해 독립된 서버처럼 보이게 만든다.

| 기술 | 역할 |
|---|---|
| `chroot` | 특정 디렉터리를 루트(`/`)처럼 보이게 해 파일 시스템 격리 |
| `namespace` | 프로세스, 네트워크, hostname 등을 분리된 세계처럼 보이게 함 |
| `cgroup` | CPU, 메모리, 디스크 I/O 같은 자원 사용량 제한 |

### 2.3 Docker의 기본 구성 요소

| 개념 | 의미 | 비유 |
|---|---|---|
| Dockerfile | 이미지를 만들기 위한 빌드 스크립트 | 조리법 |
| Image | 컨테이너 실행을 위한 읽기 전용 템플릿 | 완성된 실행 패키지 |
| Container | 이미지를 실제로 실행한 상태 | 실행 중인 앱 |
| Registry | 이미지를 저장하고 공유하는 저장소 | 이미지 창고 |
| Docker Hub | 대표적인 공개 Registry | 공식 이미지 저장소 |

핵심 관계는 다음과 같다.

```text
Dockerfile --docker build--> Image --docker run--> Container
```

### 2.4 자주 쓰는 Docker 명령어

| 목적 | 명령어 | 의미 |
|---|---|---|
| 이미지 빌드 | `docker build -t my-app .` | 현재 디렉터리 Dockerfile로 이미지 생성 |
| 컨테이너 실행 | `docker run my-app` | 이미지로 새 컨테이너 실행 |
| 백그라운드 실행 | `docker run -d ...` | 터미널을 점유하지 않고 실행 |
| 포트 연결 | `docker run -p 8080:80 nginx` | 호스트 8080 → 컨테이너 80 연결 |
| 실행 중 컨테이너 확인 | `docker ps` | 현재 실행 중인 컨테이너 목록 |
| 모든 컨테이너 확인 | `docker ps -a` | 종료된 컨테이너까지 확인 |
| 컨테이너 내부 접속 | `docker exec -it <container> bash` | 실행 중인 컨테이너 안에서 명령 실행 |
| 컨테이너 삭제 | `docker rm <container>` | 중지된 컨테이너 삭제 |

특히 `docker run`과 `docker exec`의 차이가 중요하다.

| 명령어 | 차이 |
|---|---|
| `docker run` | 새 컨테이너를 만든 뒤 실행 |
| `docker exec` | 이미 실행 중인 컨테이너 안에서 추가 명령 실행 |

### 2.5 Dockerfile에서 중요한 명령어

| 명령어 | 역할 | 주의점 |
|---|---|---|
| `FROM` | 베이스 이미지 지정 | 보통 Dockerfile의 시작점 |
| `WORKDIR` | 컨테이너 내부 작업 디렉터리 지정 | 이후 명령어 기준 경로가 됨 |
| `COPY` | 파일/디렉터리를 이미지 안으로 복사 | 빌드 컨텍스트 안의 파일만 가능 |
| `RUN` | 이미지 빌드 시 실행할 명령 | 패키지 설치 등에 사용 |
| `EXPOSE` | 컨테이너가 사용할 포트 명세 | 실제 외부 공개는 아님 |
| `CMD` | 컨테이너 실행 시 기본 명령 | `docker run` 명령으로 덮어쓸 수 있음 |
| `ENTRYPOINT` | 컨테이너 실행 시 고정 실행 명령 | `CMD`와 조합 가능 |
| `USER` | 명령 실행 사용자 지정 | root 실행을 줄이는 보안 습관 |

가장 많이 헷갈리는 부분은 `EXPOSE`다.

```dockerfile
EXPOSE 80
```

이것은 “이 컨테이너는 80번 포트를 쓴다”는 명세일 뿐이다. 실제로 내 PC에서 접근하려면 다음처럼 `-p` 옵션이 필요하다.

```bash
docker run -p 8080:80 nginx
```

---

## 3. Day 2 핵심: Docker Compose

Day 2는 단일 컨테이너에서 벗어나 **여러 컨테이너를 하나의 애플리케이션처럼 관리하는 방법**을 배웠다.

예를 들어 FastAPI 앱과 MySQL DB는 보통 한 컨테이너에 같이 넣지 않고 분리한다.

```text
FastAPI container  →  MySQL container
```

이 구조를 `docker run` 명령 여러 개로 관리하면 환경변수, 포트, 네트워크, 볼륨, 실행 순서를 매번 직접 맞춰야 한다. Docker Compose는 이를 하나의 YAML 파일로 관리하게 해준다.

### 3.1 환경변수

환경변수는 코드에 직접 박아두면 곤란한 설정을 외부에서 주입하는 방식이다.

예를 들어 DB 주소, 포트, 사용자명, 비밀번호, 실행 모드 등을 환경변수로 둘 수 있다.

| 방식 | 사용 위치 | 예시 |
|---|---|---|
| Dockerfile `ENV` | 이미지에 기본 환경변수 포함 | `ENV APP_PORT=8000` |
| `docker run -e` | 컨테이너 실행 시 주입 | `docker run -e APP_ENV=local ...` |
| `--env-file` | `.env` 파일로 주입 | `docker run --env-file .env ...` |
| Compose `environment` | compose 파일에 직접 작성 | `DATABASE_URL=...` |
| Compose `env_file` | compose에서 `.env` 파일 읽기 | `env_file: .env` |

중요한 원칙은 **비밀번호가 들어간 `.env` 파일을 GitHub에 올리지 않는 것**이다.

### 3.2 볼륨과 데이터 영속화

컨테이너 내부 데이터는 컨테이너를 삭제하면 사라질 수 있다. DB 데이터처럼 남아야 하는 데이터는 컨테이너 밖에 저장해야 한다.

| 구분 | Bind Mount | Volume |
|---|---|---|
| 저장 위치 | 사용자가 지정한 호스트 경로 | Docker가 관리하는 저장 공간 |
| 주 용도 | 개발 중 소스 코드 연결 | DB 데이터, 로그, 운영 데이터 보존 |
| 장점 | 로컬 파일을 바로 수정 가능 | Docker가 관리해 안정적 |
| 예시 | `./app:/app` | `db-data:/var/lib/mysql` |

실습 감각으로는 다음처럼 정리하면 된다.

```text
개발 소스 공유 → Bind Mount
DB 데이터 보존 → Volume
```

### 3.3 Docker 네트워크

컨테이너끼리 통신하려면 Docker 네트워크가 필요하다.

기본 bridge 네트워크에서는 IP로 통신할 수 있지만 이름 통신은 제한적이다. 반면 사용자 정의 네트워크나 Compose 네트워크에서는 서비스 이름으로 통신할 수 있다.

Day 2에서 FastAPI 앱이 MySQL에 접속할 때 사용한 주소는 다음과 같다.

```text
DATABASE_URL=mysql+pymysql://todo_user:todo_pass@db:3306/todo_db
```

여기서 `db`는 인터넷 주소가 아니라 `docker-compose.yml`에 있는 **서비스 이름**이다.

```yaml
services:
  web:
    environment:
      DATABASE_URL: mysql+pymysql://todo_user:todo_pass@db:3306/todo_db

  db:
    image: mysql:8.0
```

즉, Compose 내부 네트워크에서는 `db`라는 이름이 MySQL 컨테이너의 hostname처럼 동작한다.

### 3.4 Compose 핵심 문법

| 항목 | 역할 |
|---|---|
| `services` | 실행할 컨테이너 서비스 목록 |
| `image` | 사용할 기존 이미지 지정 |
| `build` | Dockerfile로 직접 이미지 빌드 |
| `ports` | 호스트 포트와 컨테이너 포트 연결 |
| `expose` | 컨테이너 내부 포트 명세, 외부 공개 아님 |
| `environment` | 환경변수 직접 주입 |
| `env_file` | 환경변수 파일 사용 |
| `volumes` | 데이터 저장소 연결 |
| `networks` | 컨테이너 간 통신 네트워크 설정 |
| `healthcheck` | 컨테이너가 실제로 준비됐는지 검사 |
| `depends_on` | 서비스 시작 순서와 의존성 설정 |

DB와 앱을 함께 띄울 때는 `healthcheck`와 `depends_on`이 중요하다. DB 컨테이너가 “시작”됐다고 해서 곧바로 DB 접속이 가능한 상태는 아닐 수 있기 때문이다.

### 3.5 Compose 자주 쓰는 명령어

| 목적 | 명령어 |
|---|---|
| 실행 | `docker compose up` |
| 백그라운드 실행 | `docker compose up -d` |
| 빌드 후 실행 | `docker compose up -d --build` |
| 상태 확인 | `docker compose ps` |
| 로그 확인 | `docker compose logs -f` |
| 특정 서비스 로그 | `docker compose logs -f web` |
| 내부 명령 실행 | `docker compose exec db bash` |
| 중지/삭제 | `docker compose down` |
| 볼륨까지 삭제 | `docker compose down -v` |

`docker compose down -v`는 DB 데이터가 담긴 볼륨까지 삭제할 수 있으므로 실습 초기화가 아니라면 조심해야 한다.

---

## 4. Day 3 핵심: AWS 인프라 개요

Day 3는 Docker에서 잠시 벗어나, 실제 클라우드 배포를 이해하기 위해 필요한 **네트워크 기본기와 AWS 핵심 리소스**를 배웠다.

### 4.1 네트워크 핵심 개념

| 개념 | 의미 |
|---|---|
| MAC 주소 | 같은 LAN 안에서 장치를 식별하는 주소 |
| IP 주소 | 네트워크 상에서 호스트를 식별하는 주소 |
| Port | 하나의 호스트 안에서 특정 프로세스를 식별하는 번호 |
| Router | 서로 다른 네트워크 사이에서 패킷 경로를 결정 |
| NAT | 사설 IP와 공인 IP를 변환 |
| TCP | 연결 지향, 신뢰성 있는 전송 |
| UDP | 비연결형, 빠른 전송 |
| TLS | 암호화, 무결성, 인증 제공 |

실습과 가장 직접 연결되는 개념은 IP와 Port다.

```text
IP 주소  = 어느 서버인가?
Port     = 그 서버 안의 어떤 프로그램인가?
```

예를 들어 다음 주소에서:

```text
http://<EC2_PUBLIC_IP>:8000
```

- `<EC2_PUBLIC_IP>`는 EC2 서버를 가리킨다.
- `8000`은 EC2 안에서 실행 중인 FastAPI 프로세스를 가리킨다.

### 4.2 AWS 글로벌 인프라

| 개념 | 의미 |
|---|---|
| Region | AWS 데이터센터가 모여 있는 지리적 위치, 예: 서울 리전 |
| Availability Zone, AZ | 리전 안의 독립된 데이터센터 단위 |
| Edge Location | 사용자 가까이에서 콘텐츠 캐시/DNS 응답 제공 |

장애에 강한 구조를 만들려면 여러 AZ에 리소스를 분산하는 것이 중요하다.

### 4.3 VPC와 네트워크 구성

VPC는 AWS 안에 만드는 독립적인 논리 네트워크다.

| 구성 요소 | 역할 |
|---|---|
| VPC | AWS 안의 독립 네트워크 범위 |
| Subnet | VPC를 더 작게 나눈 네트워크, 하나의 AZ에 속함 |
| Public Subnet | 인터넷과 직접 통신 가능한 서브넷 |
| Private Subnet | 인터넷과 직접 통신하지 않는 내부용 서브넷 |
| Internet Gateway | VPC와 인터넷 연결 |
| NAT Gateway | Private Subnet에서 외부로 나가는 통신 지원 |
| Route Table | 목적지에 따라 트래픽을 어디로 보낼지 결정 |

퍼블릭 서브넷으로 동작하려면 보통 다음 조건이 필요하다.

1. VPC에 Internet Gateway가 연결되어 있어야 한다.
2. Route Table에 `0.0.0.0/0 → Internet Gateway` 경로가 있어야 한다.
3. 인스턴스에 Public IP가 있어야 한다.
4. 보안 그룹에서 필요한 포트를 허용해야 한다.

### 4.4 Security Group과 Network ACL

| 구분 | Security Group | Network ACL |
|---|---|---|
| 적용 범위 | 인스턴스 수준 | 서브넷 수준 |
| 상태 | Stateful | Stateless |
| 규칙 방식 | 허용 규칙 중심 | 허용/거부 규칙 가능 |
| 응답 트래픽 | 자동 허용 | 인바운드/아웃바운드 모두 별도 고려 |

실습에서는 주로 Security Group을 많이 다룬다. 예를 들어 FastAPI를 EC2에서 8000번 포트로 실행하면, Security Group에서 TCP 8000 인바운드를 허용해야 외부에서 접속할 수 있다.

### 4.5 IAM

IAM은 AWS 리소스 접근 권한을 관리하는 서비스다.

```text
누가, 어떤 리소스에 대해, 어떤 작업을 할 수 있는가?
```

| 개념 | 의미 |
|---|---|
| Root User | 계정 최고 권한 사용자, 일상 작업에 사용하지 않는 것이 좋음 |
| IAM User | 실제 작업용 사용자 |
| IAM Group | 여러 사용자에게 공통 권한 부여 |
| IAM Policy | 권한을 정의하는 JSON 문서 |
| IAM Role | 사용자나 서비스가 임시로 맡는 권한 묶음 |
| MFA | 비밀번호 외 추가 인증 수단 |

기본 원칙은 **최소 권한 원칙**이다.

### 4.6 EC2, S3, RDS

| 서비스 | 역할 | 이번 주 흐름에서의 위치 |
|---|---|---|
| EC2 | AWS에서 빌려 쓰는 가상 서버 | FastAPI 백엔드 서버 실행 가능 |
| S3 | 객체 스토리지 | 정적 프론트엔드 파일 호스팅 가능 |
| RDS | 관리형 관계형 데이터베이스 | MySQL 같은 DB를 AWS에서 운영 |

Day 3 실습 구조는 다음처럼 이해할 수 있다.

```text
사용자 브라우저
  → S3 정적 웹사이트
  → EC2 FastAPI 백엔드
  → RDS MySQL DB
```

### 4.7 RDS 고가용성/확장 개념

| 구분 | Multi-AZ | Read Replica |
|---|---|---|
| 목적 | 장애 복구, 고가용성 | 읽기 성능 향상 |
| 복제 방식 | 동기식 | 비동기식 |
| 장애 전환 | 자동 페일오버 | 수동 전환 성격 |
| 읽기 분산 | 주 목적 아님 | 가능 |

간단히 말하면:

```text
장애 대응 → Multi-AZ
읽기 부하 분산 → Read Replica
```

---

## 5. Day 5 핵심: 클라우드 DB 연동

Day 5의 핵심은 Day 2에서 만든 FastAPI + MySQL Compose 구조를 클라우드 DB로 확장하는 것이다.

### 5.1 구조 변화

| 구분 | Day 2 | Day 5 |
|---|---|---|
| DB 위치 | 로컬 MySQL 컨테이너 | AWS RDS MySQL |
| DB 주소 | `db` | `<rds-endpoint>` |
| 접근 제어 | Docker Compose 네트워크 | RDS Security Group + DB 계정 |
| 확인 방식 | `docker exec todo-db ...` | MySQL client 컨테이너로 RDS 접속 |
| 운영 감각 | 로컬 개발 환경 | 클라우드 운영 환경에 가까움 |

구조를 그림처럼 보면 다음과 같다.

```text
Day 2
FastAPI container ──> MySQL container
              DATABASE_URL의 host = db

Day 5
FastAPI container ──> AWS RDS MySQL
              DATABASE_URL의 host = <rds-endpoint>
```

### 5.2 핵심은 `DATABASE_URL`

Day 2의 로컬 DB 연결 문자열은 다음과 같았다.

```text
DATABASE_URL=mysql+pymysql://todo_user:todo_pass@db:3306/todo_db
```

Day 5에서는 `db` 자리에 RDS endpoint가 들어간다.

```text
DATABASE_URL=mysql+pymysql://ktc_001:<password>@<rds-endpoint>:3306/ktc_001_db
```

일반 형식은 다음과 같다.

```text
DB종류+드라이버://사용자이름:비밀번호@호스트:포트/데이터베이스명
```

| 구성 요소 | 예시 | 의미 |
|---|---|---|
| DB 종류/드라이버 | `mysql+pymysql` | MySQL에 PyMySQL 드라이버로 접속 |
| 사용자 | `ktc_001` | DB 사용자 이름 |
| 비밀번호 | `<password>` | DB 비밀번호 |
| 호스트 | `<rds-endpoint>` | RDS 접속 주소 |
| 포트 | `3306` | MySQL 기본 포트 |
| DB 이름 | `ktc_001_db` | 사용할 데이터베이스 |

### 5.3 RDS 연결에 필요한 값

| 값 | 의미 |
|---|---|
| endpoint | RDS 접속 주소 |
| port | MySQL 접속 포트, 보통 3306 |
| database name | 사용할 DB 이름 |
| username | DB 사용자 이름 |
| password | DB 비밀번호 |
| security group | 누가 RDS에 접속할 수 있는지 정하는 방화벽 |

RDS 연결 실패 시 가장 먼저 이 값들 중 하나가 틀렸다고 보고 점검하면 된다.

### 5.4 실습 흐름

```text
1. Day 2 로컬 DB 구조 확인
2. .env.example을 .env로 복사
3. DATABASE_URL을 RDS용으로 변경
4. docker compose down 후 web 재실행
5. FastAPI docs에서 todo 생성
6. MySQL client 컨테이너로 RDS 접속
7. SELECT 문으로 실제 저장 확인
```

주요 명령어는 다음과 같다.

```bash
# 설정 변경 후 컨테이너 재실행
docker compose down
docker compose up -d --build web
docker compose logs -f web
```

RDS에 직접 접속할 때는 로컬 `todo-db` 컨테이너가 아니라 MySQL client 컨테이너를 잠깐 실행한다.

```bash
docker run --rm -it mysql:8.0 mysql -h <rds-endpoint> -P 3306 -u ktc_001 -p ktc_001_db
```

접속 후 확인 SQL:

```sql
SELECT DATABASE();
SELECT CURRENT_USER();
SHOW TABLES;
SELECT * FROM todos;
SELECT COUNT(*) AS todo_count FROM todos;
EXIT;
```

### 5.5 자주 만나는 에러

| 에러/증상 | 가능성이 큰 원인 | 확인할 것 |
|---|---|---|
| `connection timed out` | RDS 보안 그룹에서 3306 미허용 | Security Group |
| `Access denied` | 사용자명/비밀번호 오류 | username, password |
| `Unknown database` | DB 이름 오타 | database name |
| `Unknown MySQL server host` | endpoint 오타 | RDS endpoint |
| 계속 로컬 DB에 저장됨 | `.env` 변경 미적용 | 컨테이너 재실행 여부 |
| `mysql` 명령어 없음 | 앱 컨테이너에 MySQL CLI 없음 | `mysql:8.0` client 컨테이너 사용 |

---

## 6. 이번 주를 관통하는 핵심 연결 관계

### 6.1 Docker 관점

```text
Dockerfile은 이미지를 만든다.
이미지는 컨테이너의 실행 템플릿이다.
컨테이너는 이미지를 실제로 실행한 상태다.
Compose는 여러 컨테이너와 네트워크, 볼륨, 환경변수를 한 번에 관리한다.
```

### 6.2 네트워크 관점

```text
컨테이너 안에서 앱이 떠도 외부에서 접근하려면 포트 연결이 필요하다.
Compose 내부에서는 service 이름으로 컨테이너끼리 통신한다.
AWS에서는 VPC, 서브넷, 라우팅, 보안 그룹이 통신 가능 여부를 결정한다.
RDS 연결은 endpoint, port, 계정, 보안 그룹이 모두 맞아야 성공한다.
```

### 6.3 설정 분리 관점

이번 주에서 가장 실무적인 감각은 **코드와 설정을 분리하는 것**이다.

| 나쁜 방식 | 좋은 방식 |
|---|---|
| DB 주소를 코드에 직접 작성 | `DATABASE_URL` 환경변수 사용 |
| 비밀번호를 코드/GitHub에 저장 | `.env`로 분리하고 Git 제외 |
| 로컬 DB 주소만 고정 | 환경변수 변경으로 로컬/RDS 전환 |
| 컨테이너 실행 명령을 매번 직접 입력 | Compose YAML로 재현 가능하게 관리 |

Day 5에서 앱 코드를 크게 바꾸지 않고 RDS로 연결할 수 있었던 이유도 DB 연결 정보가 `DATABASE_URL`로 분리되어 있었기 때문이다.

---

## 7. 실습할 때 기억할 체크리스트

### Docker / Compose 체크리스트

| 상황 | 확인할 것 |
|---|---|
| 컨테이너가 안 뜸 | `docker compose logs -f` 확인 |
| 브라우저 접속 안 됨 | `ports` 또는 `docker run -p` 설정 확인 |
| DB 연결 안 됨 | 서비스 이름, 포트, 환경변수 확인 |
| 설정 바꿨는데 반영 안 됨 | 컨테이너 재실행 여부 확인 |
| 데이터가 사라짐 | 볼륨을 썼는지, `down -v`를 실행했는지 확인 |

### AWS 체크리스트

| 상황 | 확인할 것 |
|---|---|
| EC2 접속 안 됨 | Public IP, SSH 키, Security Group 22번 |
| FastAPI 외부 접속 안 됨 | `--host 0.0.0.0`, Security Group 8000번 |
| S3 정적 페이지 안 보임 | 퍼블릭 접근 설정, 객체 권한, 정적 웹 호스팅 |
| RDS 접속 안 됨 | endpoint, 3306 포트, DB 계정, Security Group |
| 비용 걱정 | EC2, RDS, NAT Gateway, Elastic IP, S3 정리 |

---

## 8. 이번 주 최종 요약

이번 주 활동은 단순히 Docker 명령어 몇 개를 익히는 과정이 아니라, **로컬 개발 환경에서 클라우드 인프라로 이어지는 백엔드 실행 구조를 단계적으로 이해하는 과정**이었다.

처음에는 Dockerfile로 이미지를 만들고 단일 컨테이너를 실행했다. 그다음 Docker Compose로 FastAPI와 MySQL처럼 여러 컨테이너가 함께 동작하는 구조를 만들었다. 이후 AWS에서 EC2, S3, RDS, VPC, IAM, Security Group 같은 인프라 요소가 어떤 역할을 하는지 배웠고, 마지막으로 로컬 MySQL 컨테이너 대신 AWS RDS MySQL에 FastAPI 앱을 연결했다.

가장 중요한 한 줄은 다음이다.

> 애플리케이션 코드는 컨테이너로 실행 환경을 표준화하고, 환경변수와 네트워크 설정을 통해 로컬 DB든 클라우드 DB든 연결 대상을 바꿀 수 있다.

즉, 이번 주의 핵심은 다음 네 가지다.

| 핵심 | 의미 |
|---|---|
| 컨테이너화 | 앱 실행 환경을 이미지로 포장해 어디서나 비슷하게 실행 |
| Compose | 여러 컨테이너, 네트워크, 볼륨, 환경변수를 하나의 구성으로 관리 |
| AWS 인프라 | 서버, 스토리지, DB, 네트워크, 보안을 클라우드 리소스로 구성 |
| RDS 연동 | 로컬 DB에서 클라우드 DB로 연결 대상을 전환하는 실전 흐름 |

이 흐름을 이해하면 이후 배포 자동화, CI/CD, 운영 환경 구성으로 넘어갈 때도 “앱은 어떻게 실행되고, 어디에 저장하며, 어떤 네트워크와 권한으로 연결되는가?”를 기준으로 구조를 파악할 수 있다.

---

## 9. 참고한 정리본

- `Day 1 - Docker 컨테이너 기초.md`
- `Day 2 - Docker Compose.md`
- `Day 3 - AWS 인프라 개요.md`
- `Day 5 - 클라우드 DB 연동.md`

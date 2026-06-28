# Day 2 - Docker Compose 상세 정리

원본: `Day 2 - Docker Compose.pdf`  
주제: Dockerfile의 환경변수, 컨테이너 데이터 영속화, Docker 네트워크, 다중 컨테이너 구조, Docker Compose 문법, 배포 환경 분리

---

## 0. 전체 흐름

Day 2의 핵심은 **단일 컨테이너 실행에서 다중 컨테이너 애플리케이션 구성으로 넘어가는 것**이다.

Day 1에서 Dockerfile로 이미지를 만들고 단일 컨테이너를 실행하는 방법을 배웠다면, Day 2에서는 다음 내용을 다룬다.

1. 컨테이너 동작을 바꾸는 **환경변수**
2. 컨테이너 삭제 후에도 데이터를 남기는 **볼륨 / 바인드 마운트**
3. 컨테이너끼리 통신하게 만드는 **Docker 네트워크**
4. 여러 컨테이너를 하나의 서비스처럼 구성하는 **다중 컨테이너**
5. 이 모든 설정을 하나의 YAML 파일로 관리하는 **Docker Compose**
6. 로컬 / 개발 / 운영 환경에 따라 Compose 설정을 분리하는 **배포 환경 관리**

---

# 1. 환경변수

## 1.1 환경변수란?

환경변수는 **프로세스가 실행될 때 참조할 수 있는 외부 설정값**이다.

Docker에서 환경변수는 다음 시점에 사용될 수 있다.

- 이미지를 빌드할 때 각 레이어에서 참조
- 컨테이너가 실행되는 동안 애플리케이션이 참조
- 서버 포트, DB 주소, 컨텍스트 경로, 실행 모드 같은 설정을 동적으로 관리

예를 들어 FastAPI 서버가 DB에 접속할 때 DB 주소를 코드에 박아두지 않고 다음처럼 환경변수로 받을 수 있다.

```bash
MY_DB_HOST=my-db
```

코드에서는 `MY_DB_HOST`를 읽어서 DB 접속 대상을 결정한다.

---

## 1.2 Dockerfile의 ENV

Dockerfile에서는 `ENV` 명령어로 환경변수를 설정한다.

```dockerfile
ENV <key>=<value>
```

예시:

```dockerfile
ENV VAR_A=A
ENV APP_PORT=8000
```

여러 개를 한 줄에 정의할 수도 있다.

```dockerfile
ENV VAR_A=A VAR_B=B
```

여러 줄로 나눌 때는 백슬래시(`\`)를 쓴다.

```dockerfile
ENV VAR_A="Hello \
World"
```

주의할 점은 Dockerfile의 `ENV`는 **이미지에 남는다**는 것이다.  
즉, 해당 이미지로 만든 컨테이너는 기본적으로 그 환경변수를 갖게 된다.

또한 부모 이미지에 선언된 `ENV`는 자식 이미지에도 상속된다.

---

## 1.3 ENV 값을 다른 시점에 주입하기

환경변수는 반드시 Dockerfile에 고정할 필요가 없다. 목적에 따라 주입 시점을 나눌 수 있다.

### 빌드 시점: ARG + --build-arg

이미지를 빌드할 때만 필요한 값은 `ARG`를 쓴다.

```dockerfile
ARG VAR_A_FROM_ARG
ENV VAR_A=${VAR_A_FROM_ARG}
```

빌드할 때는 다음처럼 전달한다.

```bash
docker build --build-arg VAR_A_FROM_ARG=A .
```

빌드할 때만 필요한 값이면 `ENV`보다 `ARG`가 적합하다.

### 컨테이너 실행 시점: -e

컨테이너를 실행할 때 환경변수를 직접 줄 수 있다.

```bash
docker run -e VAR_A=A my-container
```

### 컨테이너 실행 시점: --env-file

환경변수가 많으면 `.env` 파일을 만들어 넣을 수 있다.

```bash
docker run --env-file .env my-container
```

`.env` 예시:

```env
VAR_A=A
VAR_B=B
```

---

## 1.4 `${VAR}`와 `$VAR`

환경변수를 참조할 때는 `$VAR`와 `${VAR}` 둘 다 가능하다.

```bash
$VAR
${VAR}
```

하지만 보통은 `${VAR}`가 더 안전하다.

이유는 다음과 같다.

### 문자열과 붙일 때 명확함

```bash
${VAR}_A
```

만약 `VAR=v`라면 위 값은 `v_A`가 된다.

반면 아래처럼 쓰면 `VAR_A`라는 변수로 해석될 수 있다.

```bash
$VAR_A
```

### 조건부 기본값 사용 가능

```bash
${VAR:-defaultValue}
```

`VAR`가 없으면 `defaultValue`를 사용한다.

```bash
${VAR:+defaultValue}
```

`VAR`가 있으면 `defaultValue`, 없으면 빈 문자열을 사용한다.

---

## 1.5 환경변수 사용 시 유의사항

환경변수 이름은 충돌을 피해야 한다.  
시스템이나 베이스 이미지에서 이미 사용하는 변수명을 덮어쓰면 예상치 못한 문제가 생길 수 있다.

권장 방식:

```dockerfile
ENV MYAPP_DB_HOST=my-db
ENV MYAPP_PORT=8000
```

즉, 프로젝트 접두사를 붙인다.

또한 Dockerfile에서 `ENV`를 쓸 때 `=`를 생략할 수는 있지만, 오류 가능성이 있고 추후 삭제될 수 있으므로 명시적으로 쓰는 것이 좋다.

권장:

```dockerfile
ENV VAR_A=A
```

비권장:

```dockerfile
ENV VAR_A A
```

---

# 2. 볼륨과 데이터 영속화

## 2.1 컨테이너의 휘발성

컨테이너 내부에 저장된 데이터는 기본적으로 **컨테이너가 삭제되면 같이 사라진다.**

예를 들어 MySQL 컨테이너를 띄우고 내부에 DB 데이터를 저장했다고 해도, 컨테이너를 삭제하면 DB 데이터도 사라질 수 있다.

따라서 DB 데이터, 로그, 설정 파일처럼 유지해야 하는 데이터는 컨테이너 밖에 저장해야 한다.  
이것을 **데이터 영속화**라고 한다.

---

## 2.2 Docker의 영속화 방식

Docker는 대표적으로 두 가지 영속화 방식을 제공한다.

1. **Volume**
2. **Bind mount**

`tmpfs mount`도 있지만 메모리 기반이라 영속화 방식은 아니다.

---

## 2.3 Bind mount

Bind mount는 **호스트 머신의 특정 파일 또는 디렉터리를 컨테이너 내부 경로에 직접 연결하는 방식**이다.

예시:

```bash
docker run -v /some/host/path:/some/container/path my-container
```

의미:

- 왼쪽: 호스트 머신 경로
- 오른쪽: 컨테이너 내부 경로

특징:

- 호스트 파일시스템에 강하게 의존한다.
- 탐색기나 터미널에서 바로 접근하고 수정하기 쉽다.
- 소스 코드 공유, 실시간 개발에 유리하다.
- 호스트 경로 관리 책임이 사용자에게 있다.

예를 들어 개발 중인 FastAPI 소스 코드를 컨테이너에 바로 연결하면, 코드를 수정할 때마다 컨테이너 내부에서도 변경 사항이 반영된다.

---

## 2.4 Volume

Volume은 **Docker가 관리하는 저장 공간을 컨테이너에 마운트하는 방식**이다.

예시:

```bash
docker run -v my-vol:/some/path my-container
```

또는 더 명시적인 `--mount` 형식:

```bash
docker run --mount type=volume,src=my-vol,dst=/some/path my-container
```

특징:

- 저장 위치를 Docker가 관리한다.
- Docker CLI/API로 관리할 수 있다.
- 호스트 파일시스템 영향이 상대적으로 적다.
- DB 파일, 로그, 설정값 보존에 적합하다.
- 특별한 이유가 없다면 공식적으로도 Volume 방식이 더 권장된다.

---

## 2.5 Dockerfile의 VOLUME

Dockerfile에서도 `VOLUME` 명령어를 사용할 수 있다.

```dockerfile
VOLUME /some/path /other/path
```

또는 JSON array 형식:

```dockerfile
VOLUME ["/some/path", "/other/path"]
```

이렇게 설정된 이미지로 컨테이너를 실행하면 해당 경로에 볼륨이 자동 생성된다.

단점은 자동 생성된 볼륨 이름이 해시값처럼 만들어져 관리하기 어려울 수 있다는 것이다.

---

## 2.6 `-v`와 `--mount`의 차이

Volume 기준으로는 `-v`와 `--mount`가 기능적으로 거의 같다.

```bash
docker run -v my-vol:/some/path my-container
```

```bash
docker run --mount type=volume,src=my-vol,dst=/some/path my-container
```

하지만 Bind mount 기준으로는 차이가 있다.

### Bind mount에서 `-v`

```bash
docker run -v /some/host/path:/some/container/path my-container
```

호스트 경로가 없으면 자동 생성될 수 있다.

### Bind mount에서 `--mount`

```bash
docker run --mount type=bind,src=/some/host/path,dst=/some/path my-container
```

호스트 경로가 없으면 에러가 난다.

따라서 정확하고 엄격한 설정이 필요하면 `--mount`가 더 명시적이다.

---

## 2.7 Volume vs Bind Mount 비교

| 구분 | Bind Mount | Volume |
|---|---|---|
| 저장 위치 | 사용자가 정한 호스트 디렉터리 | Docker가 정한 호스트 디렉터리 |
| 관리 주체 | 사용자 | Docker |
| 접근성 | 탐색기/터미널에서 바로 접근 가능 | Docker를 통해 접근하는 것이 원칙 |
| 주요 용도 | 소스 코드 공유, 실시간 개발 | DB 파일, 로그, 설정값 보존 |
| 안정성 | 호스트 환경 영향 큼 | 상대적으로 독립적 |
| 권장 상황 | 개발 편의성 중요 | 데이터 보존과 운영 안정성 중요 |

실무 감각으로 정리하면 다음과 같다.

- 개발 중 소스 코드를 컨테이너에 연결할 때: **Bind mount**
- DB 데이터, 서비스 로그, 운영 데이터 보존: **Volume**

---

# 3. Docker 네트워크

## 3.1 Docker 네트워크가 필요한 이유

컨테이너는 기본적으로 격리된 자원이다.  
따라서 컨테이너끼리 통신하려면 네트워크 설정이 필요하다.

예를 들어 다음과 같은 구조에서는 API 서버 컨테이너가 DB 컨테이너와 통신해야 한다.

```text
Frontend 컨테이너 -> Backend API 컨테이너 -> DB 컨테이너
```

이 통신을 제어하는 Docker의 기능이 **Docker Network**다.

---

## 3.2 Docker 네트워크 드라이버

Docker는 여러 네트워크 드라이버를 제공한다.

대표 드라이버:

- `bridge`
- `host`
- `overlay`
- `ipvlan`
- `macvlan`
- `none`

Docker 엔진에는 기본적으로 다음 네트워크가 생성되어 있다.

```bash
docker network ls
```

일반적으로 기본 항목은 다음과 비슷하다.

```text
bridge
host
none
```

---

## 3.3 bridge 네트워크

컨테이너를 별도 설정 없이 실행하면 기본적으로 `bridge` 네트워크에 연결된다.

```bash
docker run -d --name nginx1 nginx
docker run -d --name nginx2 nginx
```

기본 bridge 네트워크에서는 컨테이너끼리 **IP 주소로 통신 가능**하다.

컨테이너 IP 확인 예시:

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nginx1
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nginx2
```

nginx1 내부에서 nginx2 IP로 요청:

```bash
docker exec -it nginx1 bash
curl [nginx2_IP주소]
```

이 방식은 성공한다.

---

## 3.4 기본 bridge 네트워크의 한계

기본 bridge 네트워크에서는 컨테이너 이름으로 통신이 안 될 수 있다.

예시:

```bash
docker exec -it nginx1 bash
curl nginx2
```

기본 bridge 네트워크는 내장 DNS 서버가 제공되지 않아 컨테이너 이름을 해석하지 못한다.

정리:

| 통신 방식 | 기본 bridge |
|---|---|
| IP 주소 통신 | 가능 |
| 컨테이너 이름 통신 | 불가 |

---

## 3.5 사용자 정의 네트워크

컨테이너 이름으로 통신하려면 사용자 정의 네트워크를 만드는 것이 좋다.

```bash
docker network create elice-network
```

컨테이너 실행 시 해당 네트워크에 연결한다.

```bash
docker run -d --name web1 --network elice-network nginx
docker run -d --name web2 --network elice-network nginx
```

이제 같은 사용자 정의 네트워크 안에서는 컨테이너 이름으로 통신할 수 있다.

```bash
docker exec -it web1 bash
curl web2
```

정리:

| 통신 방식 | 사용자 정의 bridge 네트워크 |
|---|---|
| IP 주소 통신 | 가능 |
| 컨테이너 이름 DNS 통신 | 가능 |

실무에서는 DB 주소를 IP로 쓰기보다 서비스 이름으로 쓰는 방식이 훨씬 자연스럽다.

예:

```env
MY_DB_HOST=my-db
```

---

# 4. 다중 컨테이너

## 4.1 다중 컨테이너란?

다중 컨테이너란 **여러 개의 컨테이너가 하나의 애플리케이션을 구성하는 구조**다.

과거에는 한 서버나 한 컨테이너 안에 모든 기능을 넣는 방식도 많았지만, 현대 시스템에서는 기능을 분리하는 경우가 많다.

예시:

```text
Frontend 컨테이너
Backend API 컨테이너
Database 컨테이너
```

각 컨테이너는 독립적으로 실행되고, 네트워크로 서로 통신하며 하나의 서비스처럼 동작한다.

---

## 4.2 다중 컨테이너가 필요한 이유

### 기능별 분리

Frontend, Backend, DB를 분리하면 각 기능을 독립적으로 개발하고 배포할 수 있다.

### 실행 주기의 독립

단일 컨테이너 안에서 API 서버와 DB 서버를 동시에 실행하려면 `ENTRYPOINT`나 프로세스 관리가 복잡해진다.

반면 여러 컨테이너로 나누면 각 컨테이너가 하나의 주요 프로세스만 담당한다.

### 독립된 관리

API 개발팀과 DB 관리팀이 나뉘어 있다면 컨테이너도 분리되어 있는 편이 협업에 좋다.

각 팀이 버전 관리, 배포, 모니터링, 로그 관리를 독립적으로 할 수 있다.

### 효율적인 자원 할당

API 서버에만 부하가 높다면 API 컨테이너만 더 늘리거나 자원을 더 줄 수 있다.  
DB는 별도로 관리하면 된다.

### MSA와 잘 맞음

Microservice Architecture는 기능을 작은 서비스 단위로 분리하는 구조다.  
다중 컨테이너는 MSA 구조를 구성하는 기본 방식 중 하나다.

---

## 4.3 Dockerfile만으로 다중 컨테이너를 관리할 때의 불편함

다중 컨테이너를 `docker run` 명령어만으로 관리하면 설명할 것이 많아진다.

- 어떤 이미지로 컨테이너를 띄우는지
- 컨테이너 이름은 무엇인지
- 어떤 포트를 연결하는지
- 어떤 환경변수를 주는지
- 어떤 볼륨을 연결하는지
- 어떤 네트워크에 붙이는지
- 어떤 컨테이너가 먼저 떠야 하는지

이 모든 것을 팀원에게 명령어로 전달하면 실수가 생기기 쉽다.

이 문제를 해결하는 도구가 **Docker Compose**다.

---

# 5. Docker Compose

## 5.1 Docker Compose란?

Docker Compose는 **다중 컨테이너를 정의하고 관리할 수 있도록 도와주는 도구**다.

보통 `docker-compose.yml` 또는 `compose.yml` 파일을 사용한다.

Compose 파일 하나에 다음을 정의할 수 있다.

- 이미지 빌드
- 컨테이너 실행
- 환경변수
- 포트 바인딩
- 볼륨
- 네트워크
- 헬스체크
- 서비스 의존 관계

Dockerfile이 단일 이미지 빌드 규칙을 정의한다면, Docker Compose는 여러 컨테이너와 Docker 컴포넌트를 묶어 하나의 애플리케이션 실행 구성을 정의한다.

---

## 5.2 Compose 파일 기본 예시

```yaml
services:
  my-app:
    image: myapp:db
    container_name: my-app
    build:
      context: /some-path/my-app
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - MY_DB_HOST=my-db
    networks:
      - elice-network

  my-db:
    image: mysql:9.6.0
    container_name: my-db
    volumes:
      - /some-path/compose/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3306:3306"
    env_file:
      - .mysql-env

networks:
  elice-network:
```

참고로 Compose v2부터는 `version` 속성이 deprecated로 취급되므로 보통 생략한다.

---

# 6. Docker Compose CLI 기본 명령어

## 6.1 실행 및 생성

### 포그라운드 실행

```bash
docker compose up
```

터미널에 로그가 실시간으로 출력된다.  
터미널을 점유한다.

### 백그라운드 실행

```bash
docker compose up -d
```

컨테이너를 백그라운드에서 실행한다.  
실무와 실습 모두에서 자주 쓰는 방식이다.

### 이미지 재빌드 후 실행

```bash
docker compose up --build
```

소스코드나 Dockerfile이 변경되어 이미지를 다시 만들어야 할 때 쓴다.

보통은 다음 형태를 많이 쓴다.

```bash
docker compose up -d --build
```

---

## 6.2 상태 확인 및 로그

### 컨테이너 상태 확인

```bash
docker compose ps
```

### 로그 출력

```bash
docker compose logs
```

### 로그 실시간 확인

```bash
docker compose logs -f
```

특정 서비스 로그만 보고 싶으면 서비스명을 붙인다.

```bash
docker compose logs -f db
```

### 실행 중인 프로세스 확인

```bash
docker compose top
```

---

## 6.3 중지 및 삭제

### 컨테이너 중지

```bash
docker compose stop
```

컨테이너를 삭제하지 않고 중지만 한다.

특정 서비스만 중지:

```bash
docker compose stop db
```

### 중지된 컨테이너 다시 시작

```bash
docker compose start
```

단, `up`과 달리 설정 파일을 새로 읽어 재생성하지 않는다.

### 컨테이너와 네트워크 삭제

```bash
docker compose down
```

컨테이너를 중지하고 컨테이너와 네트워크를 삭제한다.

### 볼륨까지 삭제

```bash
docker compose down -v
```

주의: DB 볼륨까지 삭제되면 데이터가 사라질 수 있다.

### 이미지까지 삭제

```bash
docker compose down --rmi all
```

모든 이미지를 삭제한다.

```bash
docker compose down --rmi local
```

Compose가 직접 빌드한 로컬 이미지만 삭제한다.

```bash
docker compose down -v --rmi all
```

컨테이너, 네트워크, 볼륨, 이미지까지 전부 삭제한다.  
실습 초기화에는 유용하지만 운영 환경에서는 매우 조심해야 한다.

---

## 6.4 실행 중인 컨테이너 조작

### 컨테이너 내부 명령 실행

```bash
docker compose exec <서비스명> <명령어>
```

예:

```bash
docker compose exec db bash
```

### 서비스 재시작

```bash
docker compose restart
```

설정 변경 사항은 반영되지 않는다.  
설정 변경을 반영하려면 보통 `up -d` 또는 `up -d --build`를 사용한다.

---

# 7. docker-compose.yml 문법

## 7.1 services

`services`는 Compose 파일에서 가장 중요한 항목이다.  
실행할 컨테이너들을 정의한다.

```yaml
services:
  my-app:
    image: myapp:db
    container_name: my-app
```

구조:

- `services`: 서비스 목록
- `my-app`: 서비스 이름
- `image`: 컨테이너로 실행할 이미지
- `container_name`: 실제 컨테이너 이름

`container_name`은 `docker run --name`과 비슷하다.  
생략할 수도 있다.

---

## 7.2 image와 build

이미 존재하는 이미지를 쓸 때:

```yaml
services:
  db:
    image: mysql:8.0
```

직접 Dockerfile로 빌드할 때:

```yaml
services:
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
```

`context`는 빌드 컨텍스트다.  
`dockerfile`은 사용할 Dockerfile 이름이다.  
이는 `docker build -f` 옵션과 유사하다.

---

## 7.3 healthcheck

`healthcheck`는 컨테이너 내부에서 주기적으로 명령어를 실행해 서비스가 실제로 정상 동작 중인지 확인한다.

예시:

```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "todo_user", "-ptodo_pass"]
  interval: 5s
  timeout: 5s
  retries: 10
```

의미:

- `test`: 검사 명령어
- 종료 코드 0: healthy
- 종료 코드 0이 아님: unhealthy
- `interval`: 검사 주기
- `timeout`: 검사 제한 시간
- `retries`: 실패 허용 횟수

`docker ps`의 STATUS 칼럼에 `healthy` 또는 `unhealthy`가 표시될 수 있다.

중요한 점은 `healthcheck`가 `depends_on`의 `service_healthy` 조건과 연동된다는 것이다.

---

## 7.4 depends_on

`depends_on`은 서비스 간 시작 순서와 의존 관계를 정의한다.

예:

```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy
```

이 설정은 `db`가 먼저 실행되고, `db`의 healthcheck가 통과된 뒤 `app`이 시작되도록 한다.

서버와 DB 조합에서는 매우 중요하다.  
DB가 아직 준비되지 않았는데 API 서버가 먼저 DB 접속을 시도하면 오류가 날 수 있기 때문이다.

조건 종류:

| 조건 | 의미 |
|---|---|
| `service_started` | 서비스가 시작만 되면 진행 |
| `service_healthy` | healthcheck를 통과하면 진행 |
| `service_completed_successfully` | 정상 종료까지 대기 |

---

## 7.5 ports

`ports`는 호스트와 컨테이너 포트를 연결한다.  
`docker run -p` 옵션과 비슷하다.

```yaml
ports:
  - "3000:3000"
```

의미:

```text
호스트 포트:컨테이너 포트
```

예를 들어:

```yaml
ports:
  - "8000:8000"
```

호스트의 `localhost:8000`으로 접속하면 컨테이너 내부의 `8000`번 포트로 연결된다.

---

## 7.6 expose와 ports의 차이

`expose`는 문서화 성격이 강하다.  
컨테이너가 어떤 포트를 사용한다는 명세 역할을 한다.

```yaml
expose:
  - "3306"
```

특징:

- 외부 호스트에 직접 포트를 열지 않는다.
- 같은 Docker 네트워크 안의 다른 컨테이너가 참고할 수 있는 의미를 가진다.
- Dockerfile의 `EXPOSE`와 비슷하지만, 외부 노출을 권장하지 않는다는 의미도 포함한다.

반면 `ports`는 실제로 호스트에 포트를 바인딩한다.

```yaml
ports:
  - "3306:3306"
```

정리:

| 항목 | 역할 |
|---|---|
| `expose` | 컨테이너 내부 포트 명세, 외부 노출 없음 |
| `ports` | 호스트와 컨테이너 포트 실제 연결 |

DB는 보통 외부에 직접 노출하지 않고, 내부 네트워크에서만 접근시키는 편이 안전하다.

---

## 7.7 ports 짧은 구문

Compose의 `ports`는 여러 방식으로 쓸 수 있다.

```yaml
ports:
  - "3000"
```

컨테이너의 3000/tcp 포트를 호스트의 임시 포트에 바인딩한다.

```yaml
ports:
  - "9000:3000"
```

호스트 9000번 → 컨테이너 3000번

```yaml
ports:
  - "3000-3005"
```

컨테이너의 3000~3005 포트를 호스트 임시 포트와 연결한다.

```yaml
ports:
  - "9000-9005:3000-3005"
```

호스트 9000~9005 → 컨테이너 3000~3005

```yaml
ports:
  - "127.0.0.1:9001:3000"
```

호스트의 127.0.0.1 주소에서만 접근 가능하게 바인딩한다.

```yaml
ports:
  - "9000:3000/udp"
```

UDP 포트 바인딩이다.

---

## 7.8 ports 긴 구문

긴 구문은 각 항목을 명시적으로 쓸 수 있다.

```yaml
ports:
  - target: 3000
    published: 3000
    protocol: tcp
    mode: host
    host_ip: 127.0.0.1
```

의미:

- `target`: 컨테이너 내부 포트
- `published`: 호스트 또는 외부에 공개할 포트
- `protocol`: tcp 또는 udp
- `mode`: host 또는 ingress
- `host_ip`: 바인딩할 호스트 IP

설정이 복잡하거나 명확성이 중요할 때 긴 구문이 좋다.

---

## 7.9 environment와 env_file

환경변수를 직접 작성할 수 있다.

```yaml
environment:
  - MY_DB_HOST=my-db
```

또는 매핑 형태로도 쓸 수 있다.

```yaml
environment:
  MY_DB_HOST: my-db
  APP_PORT: 8000
```

환경변수가 많으면 파일로 분리한다.

```yaml
env_file:
  - .mysql-env
```

`.mysql-env` 예시:

```env
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_DATABASE=todo
MYSQL_USER=todo_user
MYSQL_PASSWORD=todo_pass
```

실무에서는 비밀번호 같은 민감정보를 Git에 올리지 않도록 `.gitignore`에 포함해야 한다.

---

## 7.10 volumes

`volumes`는 `docker run -v` 옵션과 비슷하게 볼륨 또는 바인드 마운트를 정의한다.

Bind mount 예시:

```yaml
volumes:
  - /some-path/compose/init.sql:/docker-entrypoint-initdb.d/init.sql
```

이 예시는 호스트의 `init.sql` 파일을 MySQL 컨테이너의 초기화 스크립트 경로에 연결한다.

Named volume 예시:

```yaml
services:
  db:
    image: mysql:8.0
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:
```

DB 데이터는 보통 named volume을 쓰는 편이 안전하다.

---

## 7.11 command

`command`는 Dockerfile의 `CMD`와 비슷하다.  
이미지 기본 명령을 덮어쓰거나, ENTRYPOINT에 넘길 파라미터 역할로 사용할 수 있다.

예:

```yaml
command:
  - --default-authentication-plugin=mysql_native_password
```

MySQL 같은 이미지에서는 실행 옵션을 넘기는 용도로 자주 쓴다.

---

## 7.12 networks

Compose 파일에서 Docker 네트워크를 정의할 수 있다.

```yaml
networks:
  elice-network:
```

서비스에 연결:

```yaml
services:
  app:
    networks:
      - elice-network

  db:
    networks:
      - elice-network
```

기본적으로 bridge 드라이버 타입의 네트워크가 생성된다.

같은 네트워크에 있는 서비스끼리는 서비스 이름으로 통신할 수 있다.

예:

```env
MY_DB_HOST=db
```

---

# 8. 배포 환경

## 8.1 배포란?

배포는 개발자가 만든 소프트웨어를 실제 사용자가 사용할 수 있도록 서버나 클라우드 환경에 올리는 과정이다.

내 컴퓨터에서는 문제가 없더라도 실제 서버 환경에서는 다음 문제가 생길 수 있다.

- 동시 접속자 증가
- 운영체제/라이브러리 버전 차이
- 네트워크 차이
- 자원 부족
- 로그/모니터링 문제
- 보안 설정 차이

Docker는 이런 환경 차이를 줄이고 **환경의 일관성**을 유지하는 데 도움이 된다.

---

## 8.2 개발 환경과 배포 환경의 차이

일반적으로 환경은 다음처럼 나눌 수 있다.

| 환경 | 의미 | 주요 목적 |
|---|---|---|
| local | 개별 개발자 PC | 즉각적인 실행, 자유로운 개발 |
| dev | 팀 개발 환경 | 코드 통합, 기능 검증, 테스트 |
| prod | 운영 환경 | 실제 사용자 서비스, 성능, 안정성, 모니터링 |

Compose 파일 관점에서는 환경마다 달라지는 값이 많다.

---

## 8.3 환경에 따라 달라질 수 있는 요소

환경별로 달라질 수 있는 요소:

- 환경변수
- DB 주소
- 포트 바인딩
- 볼륨 경로
- 네트워크 설정
- 로그 관리 방식
- 리소스 제한
- 디버그 모드 여부

예를 들어 로컬에서는 DB 호스트가 `my-db`일 수 있지만, 운영에서는 `my-db-prod`일 수 있다.

---

## 8.4 docker-compose.yml 파일 분리

하나의 Compose 파일에 모든 환경 설정을 넣으면 실수하기 쉽다.  
따라서 공통 설정과 환경별 설정을 분리하는 것이 좋다.

예:

```text
docker-compose.yml
production.yml
```

기본 파일:

```yaml
services:
  my-app:
    environment:
      - MY_DB_HOST=my-db
```

운영 파일:

```yaml
services:
  my-app:
    environment:
      - MY_DB_HOST=my-db-prod
```

실행:

```bash
docker compose -f docker-compose.yml -f production.yml up -d
```

뒤에 오는 파일이 앞의 설정을 덮어쓴다.

이 방식으로 로컬/개발/운영 환경을 분리할 수 있다.

---

# 9. FastAPI 웹 애플리케이션 Docker Compose 실습

## 9.1 실습 목표

앞 실습에서 단일 서비스 환경을 이해했다면, 이번 실습은 Docker Compose를 활용해 웹 서비스와 데이터베이스를 하나의 구성으로 통합하고 동시에 실행하는 것이다.

목표 구조:

```text
FastAPI 웹 애플리케이션 컨테이너
+
DB 컨테이너
+
네트워크
+
볼륨
+
환경변수
+
헬스체크 / 의존성
```

---

## 9.2 실습 절차 개요

1. VSCode에서 제공된 Docker 프로젝트 폴더 열기
2. 우측 상단 파일 아이콘 클릭
3. `docker-compose.yml` 파일 생성
4. Compose 파일 작성
5. `docker compose up -d --build`로 실행
6. `docker compose ps`로 상태 확인
7. `docker compose logs -f`로 로그 확인
8. 브라우저 또는 API 요청으로 서비스 동작 확인
9. 필요 시 `docker compose down`으로 정리

---

## 9.3 Compose 실습에서 쓰는 주요 속성

실습에서 중요한 속성은 다음과 같다.

| 속성 | 역할 |
|---|---|
| `services` | 실행할 서비스를 구분하여 작성 |
| `environment` | 각 컨테이너에서 사용할 환경변수 작성 |
| `ports` | 호스트와 컨테이너 포트를 연결해 외부 접근 허용 |
| `volumes` | 호스트 경로나 명명된 볼륨을 컨테이너 내부 경로에 마운트 |
| `healthcheck` | 컨테이너가 실제로 정상 동작하는지 주기적으로 확인 |
| `depends_on` | 서비스 간 시작 순서와 의존 관계 정의 |

특히 DB와 API 서버 조합에서는 `healthcheck`와 `depends_on` 조합이 중요하다.  
DB 컨테이너가 시작되었다고 해서 바로 DB 접속 가능 상태는 아닐 수 있기 때문이다.

---

## 9.4 FastAPI + DB Compose 예시

아래는 강의 내용을 바탕으로 정리한 예시 형태다.

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: todo-app
    ports:
      - "8000:8000"
    environment:
      - MY_DB_HOST=db
      - MY_DB_PORT=3306
      - MY_DB_USER=todo_user
      - MY_DB_PASSWORD=todo_pass
      - MY_DB_NAME=todo
    depends_on:
      db:
        condition: service_healthy
    networks:
      - todo-network

  db:
    image: mysql:8.0
    container_name: todo-db
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=todo
      - MYSQL_USER=todo_user
      - MYSQL_PASSWORD=todo_pass
    volumes:
      - db-data:/var/lib/mysql
    expose:
      - "3306"
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "todo_user", "-ptodo_pass"]
      interval: 5s
      timeout: 5s
      retries: 10
    networks:
      - todo-network

volumes:
  db-data:

networks:
  todo-network:
```

실행:

```bash
docker compose up -d --build
```

상태 확인:

```bash
docker compose ps
```

로그 확인:

```bash
docker compose logs -f
```

정리:

```bash
docker compose down
```

볼륨까지 삭제:

```bash
docker compose down -v
```

주의: `down -v`는 DB 데이터까지 삭제할 수 있다.

---

# 10. 핵심 암기 포인트

## 환경변수

- `ENV`는 Dockerfile에서 환경변수를 이미지에 설정한다.
- 빌드 시점 값은 `ARG`와 `--build-arg`.
- 실행 시점 값은 `docker run -e` 또는 `--env-file`.
- `${VAR}` 표기가 `$VAR`보다 안전하다.
- 프로젝트 접두사를 붙이면 시스템 환경변수 충돌을 줄일 수 있다.

## 볼륨

- 컨테이너 내부 데이터는 컨테이너 삭제 시 사라질 수 있다.
- 데이터를 보존하려면 Volume 또는 Bind mount를 사용한다.
- 개발 소스 공유는 Bind mount.
- DB 데이터 보존은 Volume.
- `--mount`는 `-v`보다 명시적이다.
- Bind mount에서 `--mount`는 호스트 경로가 없으면 에러를 낸다.

## 네트워크

- 기본 bridge 네트워크에서는 IP 통신은 가능하지만 이름 통신은 제한된다.
- 사용자 정의 bridge 네트워크에서는 컨테이너 이름으로 DNS 통신이 가능하다.
- Compose에서는 같은 네트워크의 서비스끼리 서비스 이름으로 통신한다.

## 다중 컨테이너

- Frontend, Backend, DB를 분리하면 관리와 확장이 쉬워진다.
- 컨테이너마다 실행 주기를 독립적으로 관리할 수 있다.
- MSA 구조와 잘 맞는다.
- `docker run` 명령어 여러 개를 공유하는 것은 복잡하므로 Compose를 쓴다.

## Docker Compose

- `services`는 필수 핵심 항목이다.
- `image`는 사용할 이미지, `build`는 직접 빌드 설정이다.
- `ports`는 외부 노출, `expose`는 내부 명세에 가깝다.
- `environment`와 `env_file`로 환경변수를 관리한다.
- `volumes`로 데이터 영속화를 설정한다.
- `healthcheck`와 `depends_on.condition: service_healthy` 조합은 DB 의존 서비스에서 중요하다.
- `docker compose up -d --build`는 가장 자주 쓰는 실행 명령이다.
- `docker compose down -v`는 볼륨 삭제까지 하므로 주의해야 한다.

---

# 11. 실무식으로 정리한 권장 패턴

## 개발 환경

```yaml
services:
  app:
    build: .
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    environment:
      - APP_ENV=local
```

개발 환경에서는 소스 코드를 bind mount로 연결해 수정 사항을 빠르게 반영한다.

## 운영 환경

```yaml
services:
  app:
    image: my-app:latest
    ports:
      - "80:8000"
    environment:
      - APP_ENV=prod
```

운영 환경에서는 소스코드를 bind mount하지 않고, 빌드된 이미지를 사용한다.

## DB

```yaml
services:
  db:
    image: mysql:8.0
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:
```

DB는 named volume을 사용해 데이터를 보존한다.

---

# 12. 이 강의에서 가장 중요한 한 줄

Docker Compose는 여러 컨테이너, 네트워크, 볼륨, 환경변수, 실행 순서를 하나의 YAML 파일로 정의해서 **다중 컨테이너 애플리케이션을 반복 가능하고 일관되게 실행하게 해주는 도구**다.

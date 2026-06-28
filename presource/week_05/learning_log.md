## 🗓 이번 주 개요
- 주차: Week 05
- 주제
    - Docker 컨테이너 기초
    - Docker Compose
    - AWS 인프라 개요
    - 카카오크루 특강
    - 클라우드 DB 연동
- 키워드: Docker, 컨테이너, 이미지, 볼륨, Compose, AWS, EC2, S3, RDS, VPC, IAM, Security Group, ...

## 학습 내용

### 1. Docker 컨테이너 기초

- 해당 챕터에서는 Docker가 무엇이고, 어떠한 원리로 이루어지고, 왜 사용해야 하는지에 대해 이해하는 것이었다.

- Docker는 가상머신(VM, Virtual Machine)과 비교하면 다음과 같다.

    | 구분 | 가상머신 VM | 컨테이너 |
    |---|---|---|
    | 가상화 방식 | 하드웨어 수준 가상화 | OS 수준 가상화 |
    | OS | 각 VM마다 Guest OS 포함 | Host OS 커널 공유 |
    | 무게 | 무겁고 부팅이 느림 | 가볍고 실행이 빠름 |
    | 격리 수준 | 강함 | VM보다는 약하지만 충분히 격리 가능 |
    | 사용 감각 | 컴퓨터 한 대를 새로 띄움 | 격리된 프로세스 실행 환경을 띄움 |

- 이러한 컨테이너는 다음의 리눅스 기술을 통해 독립 서버처럼 보이게 만든다.

    | 기술 | 역할 |
    |---|---|
    | `chroot` | 특정 디렉터리를 루트(`/`)처럼 보이게 해 파일 시스템 격리 |
    | `namespace` | 프로세스, 네트워크, hostname 등을 분리된 세계처럼 보이게 함 |
    | `cgroup` | CPU, 메모리, 디스크 I/O 같은 자원 사용량 제한 |

- Docker의 구성 요소는 다음과 같다.

    | 개념 | 의미 | 비유 |
    |---|---|---|
    | Dockerfile | 이미지를 만들기 위한 빌드 스크립트 | 조리법 |
    | Image | 컨테이너 실행을 위한 읽기 전용 템플릿 | 완성된 실행 패키지 |
    | Container | 이미지를 실제로 실행한 상태 | 실행 중인 앱 |
    | Registry | 이미지를 저장하고 공유하는 저장소 | 이미지 창고 |
    | Docker Hub | 대표적인 공개 Registry | 공식 이미지 저장소 |

- Docker 명령어로는 다음과 같음, 특히 `docker run`과 `docker exec`의 차이가 중요하다.

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

- Dockerfile에서 중요한 명령어들은 다음과 같다.

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

- `EXPOSE Port_num`은 “이 컨테이너는 Port_num번 포트를 쓴다”는 명세일 뿐이며, 실제로 내 PC에서 접근하려면 다음처럼 `-p` 옵션이 필요하다.

---

### 2. Docker Compose

- Dockerfile은 하나의 컨테이너를 만드는 데 쓰이는데, 실제 서비스는 구조에 따라 마이크로서비스(MSA)의 구조로 만들어 유지보수성을 강화한다.
- 다만, Dockerfile 별로 하나씩 Container를 수동으로 생성하게 되면 번거로우므로, 이를 Orechestration 할 수 있는 툴이 필요하며, 이는 Docker Compose를 통해 할 수 있다.
- 즉, Docker Compose는 **여러 컨테이너를 하나의 애플리케이션처럼 관리하는 방법**이다.

- 환경 변수를 통해 외부에서 값을 주입할 수 있으며, 방식은 다음과 같으며, 포인트는 `.env`같은 민감한 파일은 Github에 올리지 않아야 한다.

    | 방식 | 사용 위치 | 예시 |
    |---|---|---|
    | Dockerfile `ENV` | 이미지에 기본 환경변수 포함 | `ENV APP_PORT=8000` |
    | `docker run -e` | 컨테이너 실행 시 주입 | `docker run -e APP_ENV=local ...` |
    | `--env-file` | `.env` 파일로 주입 | `docker run --env-file .env ...` |
    | Compose `environment` | compose 파일에 직접 작성 | `DATABASE_URL=...` |
    | Compose `env_file` | compose에서 `.env` 파일 읽기 | `env_file: .env` |

- 컨테이너 내부 데이터는 컨테이너 삭제 시에 사라질 수 있는데, 이러한 데이터를 남기려면 컨테이너 밖인 볼륨(Volume)에 저장할 수 있다.

    | 구분 | Bind Mount | Volume |
    |---|---|---|
    | 저장 위치 | 사용자가 지정한 호스트 경로 | Docker가 관리하는 저장 공간 |
    | 주 용도 | 개발 중 소스 코드 연결 | DB 데이터, 로그, 운영 데이터 보존 |
    | 장점 | 로컬 파일을 바로 수정 가능 | Docker가 관리해 안정적 |
    | 예시 | `./app:/app` | `db-data:/var/lib/mysql` |

- 개발 소스 공유는 Bind Mount, DB 데이터 보존은 Volume을 사용한다.

- 컨테이너끼리 통신을 요구할 때는 Docker 네트워크를 사용할 수 있으며, Compose에서는 서비스 이름으로 통신할 수 있다.

- Compose 핵심 문법은 다음과 같다.

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

- DB와 앱을 함께 띄울 때는 `healthcheck`와 `depends_on`이 중요하다. 대부분 DB 컨테이너가 무작정 시작됐다고 사용할 수 있는 것이 아닌 워밍업이 필요하기 때문이다.

- 자주 쓰는 Compose 명령어는 다음과 같다.

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

---

### 3. AWS 인프라 개요

- 실제 서비스는 자체 서버로 운영하는 경우도 있으나 대부분의 경우에는 AWS와 같은 클라우드 인프라를 이용한다.
- 이는 컴퓨팅 파워를 동적으로 조절하거나 잠시 필요한 경우에만 유연성있게 사용하고 반납할 수 있기 때문이다.
- 그러한 클라우드 인프라 중 대표적인 것이 AWS 인프라다.
- 사용할 때 핵심은 `IP 주소(어느 서버인가)`와 `Port(그 서버 안의 어떤 프로그램인가)`이다.

- AWS 글로벌 인프라 개념은 다음과 같다.

    | 개념 | 의미 |
    |---|---|
    | Region | AWS 데이터센터가 모여 있는 지리적 위치, 예: 서울 리전 |
    | Availability Zone, AZ | 리전 안의 독립된 데이터센터 단위 |
    | Edge Location | 사용자 가까이에서 콘텐츠 캐시/DNS 응답 제공 |

- VPC(Virtual Private Cloud)는 AWS 안에 만드는 독립적인 논리 네트워크로, 구성 요소와 역할은 다음과 같다.

    | 구성 요소 | 역할 |
    |---|---|
    | VPC | AWS 안의 독립 네트워크 범위 |
    | Subnet | VPC를 더 작게 나눈 네트워크, 하나의 AZ에 속함 |
    | Public Subnet | 인터넷과 직접 통신 가능한 서브넷 |
    | Private Subnet | 인터넷과 직접 통신하지 않는 내부용 서브넷 |
    | Internet Gateway | VPC와 인터넷 연결 |
    | NAT Gateway | Private Subnet에서 외부로 나가는 통신 지원 |
    | Route Table | 목적지에 따라 트래픽을 어디로 보낼지 결정 |

- Public Subnet으로 동작하려면 다음의 조건이 필요하다.
    1. VPC에 Internet Gateway가 연결되어 있어야 한다.
    2. Route Table에 `0.0.0.0/0 → Internet Gateway` 경로가 있어야 한다.
    3. 인스턴스에 Public IP가 있어야 한다.
    4. 보안 그룹에서 필요한 포트를 허용해야 한다.

- 보안 그룹(Security Group)과 Network ACL은 다음과 같다.

    | 구분 | Security Group | Network ACL |
    |---|---|---|
    | 적용 범위 | 인스턴스 수준 | 서브넷 수준 |
    | 상태 | Stateful | Stateless |
    | 규칙 방식 | 허용 규칙 중심 | 허용/거부 규칙 가능 |
    | 응답 트래픽 | 자동 허용 | 인바운드/아웃바운드 모두 별도 고려 |

- IAM(Identity and Access Management)은 AWS 리소스 접근 권한을 관리하는 서비스이다.
- 누가, 어떤 리소스에 대해, 어떤 작업을 할 수 있는가를 나타낸다.
- 개념은 다음과 같으며, 기본 원칙은 최소 권한 원칙이다.

    | 개념 | 의미 |
    |---|---|
    | Root User | 계정 최고 권한 사용자, 일상 작업에 사용하지 않는 것이 좋음 |
    | IAM User | 실제 작업용 사용자 |
    | IAM Group | 여러 사용자에게 공통 권한 부여 |
    | IAM Policy | 권한을 정의하는 JSON 문서 |
    | IAM Role | 사용자나 서비스가 임시로 맡는 권한 묶음 |
    | MFA | 비밀번호 외 추가 인증 수단 |

- 상품인 EC2, S3, RDS는 다음과 같다.

    | 서비스 | 역할 | 이번 주 흐름에서의 위치 |
    |---|---|---|
    | EC2 | AWS에서 빌려 쓰는 가상 서버 | FastAPI 백엔드 서버 실행 가능 |
    | S3 | 객체 스토리지 | 정적 프론트엔드 파일 호스팅 가능 |
    | RDS | 관리형 관계형 데이터베이스 | MySQL 같은 DB를 AWS에서 운영 |

- RDS의 고가용성/확장 개념은 다음과 같다.

    | 구분 | Multi-AZ | Read Replica |
    |---|---|---|
    | 목적 | 장애 복구, 고가용성 | 읽기 성능 향상 |
    | 복제 방식 | 동기식 | 비동기식 |
    | 장애 전환 | 자동 페일오버 | 수동 전환 성격 |
    | 읽기 분산 | 주 목적 아님 | 가능 |

---

### 4. 클라우드 DB 연동

- 해당 챕터는 실시간 강의로, `FastAPI + MySQL Compose` 구조에서 DB 부분을 클라우드 DB로 확장하는 것이다.
- 주 차이점은 다음과 같다.

    | 구분 | Day 2 | Day 5 |
    |---|---|---|
    | DB 위치 | 로컬 MySQL 컨테이너 | AWS RDS MySQL |
    | DB 주소 | `db` | `<rds-endpoint>` |
    | 접근 제어 | Docker Compose 네트워크 | RDS Security Group + DB 계정 |
    | 확인 방식 | `docker exec todo-db ...` | MySQL client 컨테이너로 RDS 접속 |
    | 운영 감각 | 로컬 개발 환경 | 클라우드 운영 환경에 가까움 |

- 핵심 차이는 Dockerfile에 있는 `DATABASE_URL`에 있으며, 클라우드 DB의 경우에는 db 자리에 RDS endpoint가 들어간다.
- 일반 형식은 `DB종류+드라이버://사용자이름:비밀번호@호스트:포트/데이터베이스명`과 같다.

    | 구성 요소 | 예시 | 의미 |
    |---|---|---|
    | DB 종류/드라이버 | `mysql+pymysql` | MySQL에 PyMySQL 드라이버로 접속 |
    | 사용자 | `ktc_001` | DB 사용자 이름 |
    | 비밀번호 | `<password>` | DB 비밀번호 |
    | 호스트 | `<rds-endpoint>` | RDS 접속 주소 |
    | 포트 | `3306` | MySQL 기본 포트 |
    | DB 이름 | `ktc_001_db` | 사용할 데이터베이스 |

- RDS에 연결하기 위한 값으로는 다음과 같다.

    | 값 | 의미 |
    |---|---|
    | endpoint | RDS 접속 주소 |
    | port | MySQL 접속 포트, 보통 3306 |
    | database name | 사용할 DB 이름 |
    | username | DB 사용자 이름 |
    | password | DB 비밀번호 |
    | security group | 누가 RDS에 접속할 수 있는지 정하는 방화벽 |

- 해당 실습은 쿠키즈가 각자 다른 환경에서 FE와 BE를 가동하지만 DB는 같은 endpoint를 사용하는 경우이므로, 결국 웹 페이지와 백엔드 서버가 각자 독립적인 환경에 위치해있을지 몰라도 데이터 입출력은 같은 곳에서 발생하므로, 공유 클라우드와 같은 느낌을 받을 수 있었다.

---

## 🧱 막혔던 지점 & 해결 과정
- 문제: 강의 내용을 정리하는데 있어 기존 방식으로도 눈에 잘 들어오지 않는 점
- 시도
    - AI(ChatGPT)에게 프로젝트를 다시 생성하고 조건을 더욱 고도화함.
        - 특히 표로 표현 가능한 것들은 표로 표현하도록 개별적인 지시함.
    - Day 1, 2, 3, 5일차 강의 내용 모두를 주며 강의 내용을 일부 생략해도 좋으니 실무 입장으로 한 주에 대한 내용을 정리해달라고 요청함.
- 해결: 눈에 잘 들어오지 않던 내용들을 흐름(Docker를 통한 배포 용이성, Compose를 통한 Container Orchestration, AWS를 통한 배포)을 아주 쉽게 이해할 수 있었음.
- 아쉬운 점: 내용이 생각보다는 많이 생략된 느낌이 없지 않아 있음.
## 🔁 이번 주 회고 (KPT)
- Keep: AI에게 한 주에 대한 학습 흐름으로 정리본을 받아 효율적이면서 이해하기 쉬운 흐름으로 정리한 것.
- Problem: AI에게 시킨 강의 내용 중 일부가 생각보다 많이 생략된 점.
- Try: 진행해가면서 강의 내용마다 적절한 생략 가능 정도를 AI에게 프롬프트로 반영하기

## 🎯 다음 주 목표
- [ ] 강의 내용 정리에 대해서 조금 더 고도화해보기
- [ ] 개발해놨던 토이 프로젝트를 Docker를 사용하도록 바꿔보기
- [ ] Docker를 사용하도록 전환한 토이 프로젝트를 AWS Free-tier를 통해 작은 개인 서비스를 구축해보며 인프라 적응해보기
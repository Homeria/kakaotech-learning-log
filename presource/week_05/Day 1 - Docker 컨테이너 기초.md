# Day 1 - Docker 컨테이너 기초 상세 정리

> 카카오테크캠퍼스 활동 자료 `Day 1 - Docker 컨테이너 기초.pdf`를 이해용으로 정리한 문서입니다.  
> 시험 대비가 아니라, Docker/컨테이너 개념과 실습 흐름을 다시 따라갈 수 있도록 구성했습니다.

---

## 0. 전체 흐름

이 자료는 Docker를 바로 명령어부터 외우는 방식이 아니라, **왜 컨테이너가 등장했는지**부터 시작해서 Docker의 구조, 명령어, Dockerfile 작성법, 실습까지 이어지는 흐름이다.

| 큰 주제 | 내용 |
|---|---|
| 가상머신 vs 컨테이너 | 개발 환경 불일치 문제, 서버 운영 방식의 변화, Bare Metal/VM/Container 비교 |
| Docker 개념 및 장점 | Docker가 무엇인지, 컨테이너가 무엇을 담는지, Docker의 장점 |
| Docker 구성 요소 | Client, Docker Host, Docker daemon, Image, Container, Registry, Swarm, Stack |
| Docker 기본 명령어 | `docker build`, `docker run`, `docker ps`, `docker rm`, `docker exec` |
| Docker Desktop / Docker Hub 실습 | Docker Desktop 설치, Docker Hub에서 nginx 이미지 찾기, pull/run/확인 |
| Dockerfile 명령어 | `FROM`, `COPY`, `ADD`, `WORKDIR`, `EXPOSE`, `CMD`, `ENTRYPOINT`, `RUN`, `USER` |
| Dockerfile 실습 | nginx 기반 이미지 빌드, FastAPI 웹 애플리케이션 이미지 빌드 실습 |

---

## 1. 가상머신 vs 컨테이너

### 1.1 개발자의 영원한 숙제: “내 컴퓨터에서는 되는데?”

개발을 하다 보면 다음과 같은 문제가 자주 발생한다.

> 내 컴퓨터에서는 잘 되는데 왜 다른 사람의 컴퓨터에서는 안 되는 거지?

이 문제의 주된 원인은 **환경 불일치**다.

| 원인 | 설명 |
|---|---|
| 개발 서버, 테스트 서버, 운영 서버 환경 차이 | 로컬에서는 되지만 테스트/운영 서버에서는 실패할 수 있음 |
| OS 버전 차이 | Ubuntu 버전, Windows/macOS/Linux 차이 등 |
| 라이브러리 버전 차이 | Python 패키지, Node 패키지, 시스템 라이브러리 버전 차이 |
| 설치 순서 차이 | 같은 프로그램을 설치해도 순서가 다르면 의존성 상태가 달라질 수 있음 |
| 설정 파일 차이 | 환경변수, 포트, 경로, 권한 등이 다를 수 있음 |

Docker와 컨테이너는 이 문제를 줄이기 위해 등장한 기술이다. 핵심은 **애플리케이션 실행 환경까지 함께 포장해서 어디서든 비슷하게 실행되도록 만드는 것**이다.

---

### 1.2 서버 운영의 역사

자료는 서버 운영 방식을 크게 세 단계로 보여준다.

| 시대 | 구조 | 핵심 특징 |
|---|---|---|
| Bare Metal | 물리 서버 위에 OS와 애플리케이션을 직접 실행 | 성능은 좋지만 리소스 분할이 어려움 |
| Virtualized | 물리 서버 위에 Hypervisor를 두고 여러 VM 실행 | OS 단위 격리 가능, 하지만 무겁고 느림 |
| Containerized | Host OS 커널을 공유하고 실행 환경만 격리 | 가볍고 빠르며 많은 컨테이너를 실행 가능 |

자료의 그림은 시간이 지나면서 하나의 큰 물리 서버를 그대로 쓰는 방식에서, 여러 VM을 올리는 방식으로, 다시 더 작은 단위의 컨테이너를 많이 올리는 방식으로 변화했음을 보여준다.

---

### 1.3 베어메탈(Bare Metal)

베어메탈은 **물리 서버를 그대로 사용하는 형태**다.

| 항목 | 설명 |
|---|---|
| 실행 구조 | 운영체제(OS)가 하드웨어 위에서 직접 실행됨 |
| 가상화 계층 | 중간에 Hypervisor 같은 가상화 계층이 없음 |
| 장점 | 하드웨어에 직접 접근하므로 성능 손실이 적음 |
| 사용 예 | HPC, DB 등 매우 높은 성능이 필요한 경우 |
| 단점 | 하나의 물리 서버에 하나의 OS만 사용하는 구조가 일반적이라 CPU/메모리 같은 리소스를 쪼개 쓰기 어려움 |
| 결과 | 서버 활용률이 낮아질 수 있음 |

쉽게 말하면, 고성능 컴퓨터 한 대를 통째로 하나의 작업에 쓰는 방식이다. 성능은 좋지만, 여러 서비스를 유연하게 분리해서 운영하기에는 불편하다.

---

### 1.4 가상머신(Virtual Machine, VM)

가상머신은 **물리 서버 위에 가상화 소프트웨어가 올라가고, 그 위에서 여러 개의 가상 컴퓨터가 돌아가는 형태**다.

| 항목 | 설명 |
|---|---|
| 실행 구조 | 물리 서버 위에 Hypervisor를 두고 여러 VM 실행 |
| OS | 각 VM은 자기만의 Guest OS를 가짐 |
| 격리 수준 | OS를 여러 개 돌릴 수 있어 격리 수준이 높음 |
| 하드웨어 | 완전한 가상 하드웨어를 제공 |
| OS 혼합 | Windows와 Linux를 동시에 실행 가능 |
| 단점 | 각 VM이 자체 OS를 포함하므로 무겁고 부팅이 느림 |
| 오버헤드 | CPU/Memory 오버헤드 존재 |

VM은 실제 컴퓨터를 여러 대 흉내 내는 방식이다. 격리는 강하지만, 각 VM마다 OS가 들어가므로 용량도 크고 시작도 느리다.

---

### 1.5 컨테이너(Containerized)

컨테이너는 **Host OS의 커널을 공유하면서 독립된 실행 환경만 격리하는 형태**다.

| 항목 | 설명 |
|---|---|
| 실행 구조 | Host OS 위에 Container Engine을 두고 여러 컨테이너 실행 |
| OS | OS 전체를 복제하지 않고 Host OS 커널을 공유 |
| 장점 | 가볍고 빠름 |
| 리소스 | 리소스를 효율적으로 사용 |
| 규모 | 하나의 OS 위에 수십~수백 개의 컨테이너 실행 가능 |
| 활용 분야 | DevOps, 마이크로서비스의 기본 기술 |
| 단점 | 커널을 공유하므로 VM보다 격리 수준이 낮고 보안 측면에서는 VM보다 약할 수 있음. 다만 최근에는 많이 보완됨 |

컨테이너는 VM처럼 OS 전체를 새로 띄우는 것이 아니라, **프로세스 실행 환경을 격리해서 작은 가상 서버처럼 보이게 만드는 방식**이다.

---

### 1.6 컨테이너는 진짜 독립 서버인가?

자료의 핵심 설명은 다음과 같다.

> 컨테이너는 독립된 서버처럼 보이지만, 실제로는 리눅스 커널이 이미 갖고 있는 기능을 조합해서 “독립된 것처럼 보이도록” 만든 기술이다.

컨테이너의 핵심 기반 기술은 세 가지다.

| 기술 | 역할 | 쉽게 이해하기 |
|---|---|---|
| `chroot` | 파일 시스템 격리 | “이 디렉토리를 `/`라고 믿어라”라고 만드는 기술 |
| `namespace` | 프로세스 및 시스템 리소스 이름공간 격리 | 각 컨테이너에 자기만의 세계관을 제공 |
| `cgroup` | 자원 제어/제한 | CPU, 메모리 같은 사용량을 제한 |

---

### 1.7 chroot: 가짜 루트 만들기

`chroot`는 시스템에 다음과 같이 지시하는 기술이다.

> 이 디렉토리를 루트(`/`)라고 믿어라.

| 항목 | 설명 |
|---|---|
| 원래 루트 `/` | 리눅스 파일 시스템의 최상위 디렉토리 |
| `chroot` 실행 후 | 특정 디렉토리가 `/`처럼 보임 |
| 장점 | 파일 시스템을 어느 정도 격리할 수 있음 |
| 한계 | 프로세스 트리는 보호하지 못함 |
| 결론 | 컨테이너의 기반 기술이지만 단독으로는 부족함 |

즉, `chroot`만 쓰면 파일 시스템은 속일 수 있지만, 프로세스나 네트워크 같은 다른 영역까지 완전하게 격리되지는 않는다.

---

### 1.8 namespace: 세계관을 분리하기

`namespace`는 리눅스의 강력한 격리 기술로, 각 프로세스에게 분리된 “우주”를 제공한다.

| 격리 대상 | 설명 |
|---|---|
| PID | 컨테이너 A의 PID 1과 컨테이너 B의 PID 1은 실제로는 다른 프로세스지만, 각각 자신이 PID 1이라고 인식 |
| Network | 네트워크 인터페이스가 독립됨. 가상 NIC, 가상 라우팅 테이블 사용 |
| Hostname | 컨테이너 안에서 hostname을 바꿔도 호스트에는 영향 없음 |

즉, namespace는 **다른 컨테이너나 호스트의 자원을 보이지 않게 만드는 기능**에 가깝다.

---

### 1.9 cgroup: 리눅스 자원 관리하기

`namespace`가 “보이지 않게” 만드는 기능이라면, `cgroup`은 “사용량을 제한하는” 기능이다.

| 제한 가능한 자원 | 설명 |
|---|---|
| CPU | 컨테이너가 사용할 CPU 비율/시간 제한 |
| 메모리 | 컨테이너가 사용할 수 있는 메모리 제한 |
| 디스크 I/O | 디스크 읽기/쓰기 사용량 제한 |
| 네트워크 대역폭 | 네트워크 사용량 제한 |
| 프로세스 개수 | 생성 가능한 프로세스 수 제한 |

이 기능 덕분에 하나의 컨테이너가 폭주하더라도 호스트 전체가 다운되는 것을 어느 정도 방지할 수 있다.

---

### 1.10 컨테이너 엔진의 의미

자료에서는 다음 식으로 정리한다.

```text
chroot + namespace + cgroup = 컨테이너 엔진
```

이 조합을 통해 컨테이너는 다음과 같은 상태를 갖는다.

| 독립된 것처럼 보이는 요소 | 의미 |
|---|---|
| 독립된 PID 트리 | 컨테이너 안에서 프로세스 목록이 분리되어 보임 |
| 독립된 파일 시스템 | 컨테이너 내부 파일 구조가 따로 있는 것처럼 보임 |
| 독립된 네트워크 | 컨테이너마다 네트워크 환경이 분리됨 |
| 독립된 호스트명 | 컨테이너 내부 hostname을 별도로 가짐 |
| 제한된 리소스 | CPU/메모리 등 사용량 제한 가능 |

결과적으로 컨테이너는 **하나의 작은 가상 서버처럼 보이게 된다**.

---

### 1.11 VM vs Container 비교

| 구분 | VM | Container |
|---|---|---|
| 격리 수준 | 하드웨어 레벨 가상화 | OS 레벨 가상화 |
| OS 포함 여부 | Guest OS 포함, 무거움 | Host OS 커널 공유, 가벼움 |
| 구동 속도 | 느림, 부팅 필요 | 빠름, 프로세스 실행에 가까움 |
| 용량 | 큼, 보통 GB 단위 | 작음, 보통 MB 단위 |

정리하면, VM은 **격리가 강하지만 무겁고**, 컨테이너는 **가볍고 빠르지만 커널 공유 특성상 VM보다 격리가 약할 수 있다**.

---

## 2. Docker 개념 및 장점

### 2.1 Docker의 기본 개념

Docker는 다음을 가능하게 해주는 오픈소스 플랫폼이다.

> 애플리케이션과 실행에 필요한 모든 환경을 컨테이너라는 독립된 공간에 패키징하여, 인프라 환경에 구애받지 않고 소프트웨어를 신속하게 개발, 배포, 실행할 수 있게 해준다.

즉, Docker는 단순히 프로그램 실행 도구가 아니라 **애플리케이션 실행 환경을 표준화해서 개발-배포-운영을 편하게 만드는 플랫폼**이다.

---

### 2.2 컨테이너에 포함되는 것

컨테이너는 보통 다음을 포함한다.

| 포함 요소 | 설명 |
|---|---|
| 애플리케이션 | 실제 실행할 프로그램 |
| 라이브러리 | 애플리케이션 실행에 필요한 의존성 |
| 환경설정 파일 | 실행 옵션, 설정 파일 등 |

컨테이너는 다른 컨테이너와 독립적으로 실행된다. 자료에서는 컨테이너가 **어떠한 리눅스 시스템에서도 실행 가능**하다고 설명한다.

---

### 2.3 Docker와 리눅스 커널 기능

Docker 컨테이너는 가상머신과 달리 Host OS 커널을 공유한다.

| 기술 | Docker에서의 역할 |
|---|---|
| Host OS 커널 공유 | VM보다 경량화되어 오버헤드가 적음 |
| namespace | 애플리케이션 간 환경을 격리 |
| control group(cgroup) | 컨테이너별 CPU, 메모리 등 시스템 자원 제한 |

핵심은 **보이는 환경은 분리하고, 실제 커널은 공유한다**는 것이다.

---

### 2.4 Docker의 주요 장점

| 장점 | 영어 표현 | 설명 |
|---|---|---|
| 유연성 | Flexible | 복잡한 애플리케이션도 컨테이너로 구성 가능 |
| 경량성 | Lightweight | 운영체제 커널을 공유하여 오버헤드를 최소화 |
| 교체 가능성 | Interchangeable | 컨테이너가 실행 중에도 업데이트 및 업그레이드 가능 |
| 이식성 | Portable | 로컬에서 빌드하고 클라우드에 배포하여 어디서나 실행 가능 |
| 확장성 | Scalable | 컨테이너 복제를 통해 확장 가능 |

활동/실습 관점에서 가장 중요한 장점은 **로컬에서 만든 실행 환경을 그대로 배포 환경에 가져갈 수 있다는 점**이다.

---

## 3. Docker 구성 요소 및 주요 컴포넌트

### 3.1 Docker는 클라이언트-서버 모델

Docker는 클라이언트-서버 아키텍처를 기반으로 한다.

자료의 설명을 흐름으로 바꾸면 다음과 같다.

```text
사용자 명령 입력
→ Docker Client가 명령 전달
→ Docker daemon이 실제 작업 수행
→ 필요한 이미지는 Registry에서 가져옴
→ Image를 기반으로 Container 생성/실행
```

Docker의 기본 구성 요소는 크게 세 가지다.

| 구성 요소 | 역할 |
|---|---|
| Client | 사용자가 Docker와 대화하는 통로 |
| Docker Host | Docker Engine이 실제로 구동되는 환경 |
| Registry | Docker 이미지들이 저장된 창고 |

---

### 3.2 Docker Client

Docker Client는 사용자가 Docker와 대화하는 통로다.

| 항목 | 설명 |
|---|---|
| 역할 | 사용자가 터미널에서 입력한 명령어를 Docker daemon에 전달 |
| 통신 방식 | Docker CLI는 REST API를 사용해 Docker daemon과 통신 |
| 대표 명령어 | `docker build`, `docker pull`, `docker rm` |

예시 명령어:

| 명령어 | 의미 |
|---|---|
| `docker build` | 이미지를 생성하기 |
| `docker pull` | 레지스트리에서 이미지를 가져오기 |
| `docker rm` | 컨테이너 삭제하기 |

---

### 3.3 Docker Host

Docker Host는 Docker Engine이 실제로 구동되는 환경이다.

| 구성 | 설명 |
|---|---|
| Docker daemon | 클라이언트의 명령을 받아 실제 작업을 수행하는 “뇌” 역할 |
| Image | 컨테이너 실행을 위한 읽기 전용 템플릿 |
| Container | 이미지를 실행한 실제 상태 |

#### Docker daemon

Docker daemon은 다음을 직접 관리한다.

| 관리 대상 | 설명 |
|---|---|
| 이미지 | 빌드, 저장, 삭제, 태그 관리 |
| 컨테이너 | 생성, 실행, 중지, 삭제 |
| 네트워크 | 컨테이너 네트워크 관리 |
| 기타 데몬 | 다른 데몬들과 통신하여 Docker 서비스 관리 |

#### Image

이미지는 컨테이너를 실행하기 위한 **읽기 전용 템플릿**이다.

| 특징 | 설명 |
|---|---|
| 읽기 전용 | 원본 이미지 자체는 그대로 두고 컨테이너를 생성 |
| 실행 정보 포함 | 컨테이너 실행 명령어와 환경 설정 포함 |
| 재사용 가능 | 하나의 이미지로 여러 컨테이너 생성 가능 |
| Registry 등록 가능 | 이미지를 레지스트리에 올릴 수 있음 |

#### Container

컨테이너는 **이미지를 실행한 실제 상태**다. 이미지를 클래스라고 생각하면, 컨테이너는 그 클래스로 만든 객체/인스턴스에 가깝다.

---

### 3.4 Registry

Registry는 Docker 이미지들이 저장되어 있는 “창고”다.

| 항목 | 설명 |
|---|---|
| 역할 | Docker 이미지 저장 |
| 기본 Registry | Docker Hub |
| 사용 가능 작업 | 이미지 등록, 다운로드 |
| 추가 가능성 | 프라이빗 레지스트리 설정 가능 |

Docker Hub에는 nginx, Ubuntu, Python, PostgreSQL 같은 다양한 오픈소스 프로젝트 및 솔루션 이미지가 저장되어 있다.

---

### 3.5 Docker 주요 컴포넌트 계층

자료의 그림은 다음과 같은 계층 구조를 보여준다.

```text
Stack
 └─ Service #1, #2, #3 ...
     └─ Container
         └─ Image
Swarm
 └─ Node #1, Node #2, Node #3 ...
Docker
Host OS
Infra
```

| 구성 요소 | 설명 |
|---|---|
| 인프라스트럭처 | Docker가 실행될 수 있는 호스트 머신들의 집합. 물리 시스템, 가상 머신, 네트워크, 저장 시스템 포함 |
| 호스트 OS | 인프라스트럭처에 설치된 Linux 같은 운영체제 |
| Docker | Docker daemon으로, Docker 명령어에 따라 컨테이너 생성, 관리, 배포 수행 |
| Node | 컨테이너가 실행되는 시스템. 하나의 물리 시스템 또는 가상 머신으로 구성 |
| Docker Swarm | Docker Node들을 클러스터로 묶어 관리하는 오케스트레이션 도구 |
| Service | 같은 목적을 위해 구성된 애플리케이션 집합. 여러 Node에 분산 배치된 컨테이너들로 구성 |
| Stack | 여러 Service를 그룹화한 개념. 서비스 간 연계를 통해 종속성을 공유하고 확장/축소 가능 |

---

### 3.6 Docker 이론 정리

자료의 이론 정리는 다음 세 문장으로 압축할 수 있다.

1. Docker는 클라이언트-서버 아키텍처 기반이며, Docker daemon이 Docker object들을 관리한다.
2. Docker는 Image, Container, Registry를 통해 애플리케이션의 개발, 배포, 실행을 지원한다.
3. Docker Swarm과 Stack을 이용해 오케스트레이션 및 확장성 있는 서비스를 제공하며, 물리 시스템과 가상 머신 모두에서 유연하게 실행 가능하다.

---

## 4. Docker 기본 명령어

## 4.1 `docker build`

### 의미

`docker build`는 **Dockerfile을 기반으로 Docker Image를 생성하는 명령어**다.

```bash
docker build [옵션] Dockerfile_경로
```

예시:

```bash
docker build -t my-app .
```

| 요소 | 의미 |
|---|---|
| `docker build` | 이미지 빌드 명령 |
| `-t my-app` | 이미지 이름을 `my-app`으로 지정 |
| `.` | 현재 디렉토리의 Dockerfile을 빌드 컨텍스트로 사용 |

자료 설명: 현재 디렉토리(`.`)의 Dockerfile을 읽어 `my-app`이라는 이름으로 빌드한다.

### 자주 쓰는 옵션

| 옵션 | 설명 |
|---|---|
| `-t` | 이미지 이름과 태그 설정 |
| `-f` | 특정 Dockerfile 지정 |
| `--no-cache` | 이전에 빌드한 캐시 미사용 |
| `--build-arg` | Dockerfile의 `ARG` 지시어에 값을 전달 |
| `--target` | Multi-stage Dockerfile인 경우 빌드할 타겟 정의 |
| `-o` | 이미지를 로컬 파일로 export |

---

## 4.2 `docker run`

### 의미

`docker run`은 **Image를 기반으로 새 Container를 만들고 실행하는 명령어**다.

```bash
docker run [옵션] 이미지이름 [실행할_명령어]
```

예시:

```bash
docker run ubuntu:20.04
```

자료 설명: `ubuntu:20.04` 이미지를 이용해 새 컨테이너를 실행한다.

### 자주 쓰는 옵션

| 옵션 | 설명 |
|---|---|
| `-it` | 인터랙티브 모드 + 터미널 연결 |
| `--name` | 컨테이너에 이름 부여 |
| `-d` | 백그라운드(detached)로 실행 |
| `-p` | 호스트 포트와 컨테이너 포트 매핑 |
| `--rm` | 컨테이너 종료 시 자동 삭제 |
| `-v` | 호스트 디렉토리를 컨테이너에 마운트, 즉 볼륨 연결 |

---

### `docker run` 예제 1: Ubuntu 컨테이너에서 쉘 접속

```bash
docker run -it --name my-ubuntu ubuntu:20.04 bash
```

| 부분 | 의미 |
|---|---|
| `-it` | 터미널을 통해 컨테이너 안에 직접 명령 입력 |
| `--name my-ubuntu` | 컨테이너 이름을 `my-ubuntu`로 지정 |
| `ubuntu:20.04` | 사용할 이미지 |
| `bash` | 컨테이너 안에서 실행할 명령, 즉 Linux Shell |

---

### `docker run` 예제 2: nginx 웹 서버를 백그라운드로 실행

```bash
docker run -d --name web-server -p 8080:80 nginx
```

| 부분 | 의미 |
|---|---|
| `-d` | 컨테이너를 백그라운드에서 실행 |
| `--name web-server` | 컨테이너 이름을 `web-server`로 지정 |
| `-p 8080:80` | 호스트 포트 8080을 컨테이너 포트 80으로 연결 |
| `nginx` | nginx 웹 서버 이미지 |

브라우저에서 다음 주소로 접속할 수 있다.

```text
http://localhost:8080
```

---

### `docker run` 예제 3: 테스트용 컨테이너 자동 삭제

```bash
docker run --rm ubuntu:20.04 echo "Hello Docker"
```

| 부분 | 의미 |
|---|---|
| `--rm` | 컨테이너가 명령 실행을 마치고 종료되면 자동 삭제 |
| `echo "Hello Docker"` | 컨테이너 안에서 한 번 실행 후 종료되는 명령 |

이 방식은 테스트용으로 좋다. 실행 로그만 확인하고 컨테이너 목록에는 남기고 싶지 않을 때 사용한다.

---

### `docker run` 자주 하는 실수: 포트 매핑 누락

문제 예시:

```bash
docker run -d nginx
```

이렇게 실행하면 nginx 컨테이너는 실행되지만, 호스트의 포트와 연결하지 않았기 때문에 브라우저에서 접속이 안 될 수 있다.

해결:

```bash
docker run -d -p 8080:80 nginx
```

| 문제 | 해결 |
|---|---|
| 브라우저에서 접속 안 됨 | `-p 8080:80`처럼 포트 매핑 추가 |

---

## 4.3 `docker ps`

### 의미

`docker ps`는 **현재 실행 중인 컨테이너 목록을 보는 명령어**다.

```bash
docker ps [옵션]
```

서버에서 “지금 어떤 컨테이너가 돌고 있지?”를 확인할 때 필수다.

### 기본 출력 정보

| 컬럼 | 의미 |
|---|---|
| `CONTAINER ID` | 컨테이너 고유 ID |
| `NAMES` | 컨테이너 이름 |
| `IMAGE` | 사용한 이미지 |
| `STATUS` | 실행 상태. 예: `Up`, `Exited` |
| `PORTS` | 포트 매핑 정보 |

### 자주 쓰는 옵션

| 명령어 | 설명 |
|---|---|
| `docker ps -a` | 모든 컨테이너 보기. 종료된 컨테이너 포함 |
| `docker ps --all` | `-a`와 동일 |
| `docker ps -n 3` | 최근 생성된 컨테이너 3개만 보기 |
| `docker ps -q` | 컨테이너 ID만 출력. 삭제 같은 다른 명령과 조합하기 좋음 |

---

## 4.4 `docker rm`

### 의미

`docker rm`은 **더 이상 필요 없는 컨테이너를 삭제하는 명령어**다.

```bash
docker rm [옵션] 컨테이너이름(or ID) ...
```

디스크와 컨테이너 목록을 깨끗하게 유지하기 위해 필요하다.

### 주의사항

| 상황 | 설명 |
|---|---|
| 실행 중인 컨테이너 | 바로 삭제할 수 없음 |
| 삭제 순서 | 먼저 `docker stop`으로 멈춘 뒤 `docker rm` 실행 |

### 예제 1: 단일 컨테이너 삭제

```bash
docker rm my-ubuntu
```

| 내용 | 설명 |
|---|---|
| 대상 | `my-ubuntu` 컨테이너 삭제 |
| 주의 | 실행 중이면 에러 발생. 먼저 정지 필요 |

### 예제 2: 여러 컨테이너 한 번에 삭제

```bash
docker rm my-ubuntu web-server
```

공백으로 나열하면 여러 컨테이너를 한 번에 삭제할 수 있다.

---

## 4.5 `docker exec`

### 의미

`docker exec`는 **이미 실행 중인 컨테이너 내부에 접속하거나 특정 명령어를 추가로 실행할 때 사용하는 명령어**다.

```bash
docker exec [옵션] CONTAINER COMMAND [ARGS...]
```

예시:

```bash
docker exec -it my-nginx /bin/bash
```

자료 설명: 실행 중인 `my-nginx` 컨테이너에 인터랙티브 모드(`-it`)로 접속하여 bash 셸을 실행한다.

### `run`과 `exec` 차이

| 명령어 | 의미 |
|---|---|
| `docker run` | 새로운 컨테이너를 만들어서 실행 |
| `docker exec` | 이미 돌아가고 있는 컨테이너에 들어가서 작업 |

### 자주 쓰는 옵션

| 옵션 | 설명 |
|---|---|
| `-i` | 표준 입출력 유지 |
| `-t` | pseudo-TTY 할당 |
| `-e` | 컨테이너 접속 간 임시로 사용할 환경변수 정의 |

---

## 5. Docker Desktop 설치 및 Docker Hub 실습

### 5.1 Docker Desktop 라이선스 정책

자료에서는 Docker Desktop 라이선스 정책과 관련해 다음을 설명한다.

| 항목 | 설명 |
|---|---|
| 개인 목적 사용 | 무료로 사용 가능 |

단, 라이선스 정책은 변경될 수 있으므로 실제 업무/회사 사용 시에는 Docker 공식 정책을 확인해야 한다.

---

### 5.2 Docker Desktop 설치 과정

Docker Desktop은 GUI 기반의 관리 도구를 제공한다. 자료에서는 다음 장점을 설명한다.

> 복잡한 설정 없이 Linux, Windows, macOS 환경 어디서나 동일한 컨테이너 개발 환경을 즉시 구축할 수 있다.

설치 흐름은 다음과 같다.

| 단계 | 내용 |
|---|---|
| 1 | Docker Desktop 설치 |
| 2 | 설치 완료 후 재부팅 필요 |
| 3 | Docker Subscription Service Agreement 화면에서 `Accept` 클릭 |
| 4 | Finish 설정 화면에서 `Finish` 클릭 |
| 5 | Personal 목적으로 선택 후 회원가입 또는 로그인 |
| 6 | Docker Desktop 실행 화면 확인 |

---

### 5.3 Docker 설치 확인

터미널 또는 CMD에서 다음 명령어로 Docker 설치를 확인한다.

```bash
docker --version
```

설치가 정상이라면 Docker 버전이 출력된다. 이 시점부터 CLI 환경에서 Docker를 사용할 수 있다.

---

### 5.4 Docker Hub 접속과 이미지 검색

Docker Hub에는 다양한 오픈소스 프로젝트 및 솔루션 이미지가 **Docker Repository** 형태로 저장되어 있다.

자료의 nginx 실습 흐름은 다음과 같다.

| 단계 | 내용 |
|---|---|
| 1 | Docker Desktop에서 Docker Hub 접속 |
| 2 | Docker Hub 내에서 `nginx` 검색 |
| 3 | nginx의 다양한 버전 확인 |
| 4 | `Pull` 버튼 클릭 |
| 5 | 로컬에 이미지가 다운로드되었는지 확인 |

---

### 5.5 nginx 이미지 확인

로컬에 보존되어 있는 Docker 이미지 목록은 다음 명령어로 확인한다.

```bash
docker images
```

이 명령을 실행하면 로컬에 다운로드된 `nginx:latest` 같은 이미지 목록을 볼 수 있다.

---

### 5.6 nginx 이미지 실행

자료에서는 다음 명령어로 nginx를 실행한다.

```bash
docker run -p 80:80 nginx
```

| 부분 | 의미 |
|---|---|
| `docker run` | 컨테이너 실행 |
| `-p 80:80` | 호스트 80번 포트와 컨테이너 80번 포트 연결 |
| `nginx` | 실행할 이미지 |

---

### 5.7 포트 포워딩

자료에서 포트 포워딩은 다음처럼 설명된다.

> 외부에서 호스트의 특정 포트로 들어온 접속 요청을 컨테이너 내부의 지정된 포트로 넘겨주는 역할을 한다.

nginx 실습에서는 다음 연결을 사용한다.

```text
호스트 80번 포트 → 컨테이너 80번 포트
```

즉, 브라우저가 `localhost:80`으로 요청하면 Docker가 이를 nginx 컨테이너의 80번 포트로 전달한다.

---

### 5.8 nginx 컨테이너 실행 확인

컨테이너 실행 확인 방법은 두 가지가 제시된다.

| 방법 | 설명 |
|---|---|
| Docker Desktop | 컨테이너 목록에서 nginx가 가동 중인지 확인 |
| CLI | `docker container ls` 명령어로 가동 중인 nginx 확인 |

명령어:

```bash
docker container ls
```

브라우저에서는 다음 주소로 접속한다.

```text
http://localhost
```

HTTP 기본 포트는 80번이므로 포트를 생략해도 80번 포트로 인식한다.

---

### 5.9 Docker 명령어 간단 정리

| 목적 | 명령어 |
|---|---|
| Docker 버전 확인 | `docker --version` |
| 이미지 검색 및 다운로드 | `docker pull` |
| 이미지 목록 확인 | `docker images` |
| 컨테이너 목록 확인 | `docker container ls` |
| 컨테이너 실행 | `docker run -p 80:80 nginx` |

---

## 6. Dockerfile 명령어

## 6.1 Dockerfile이란?

Dockerfile은 **사용자가 원하는 컨테이너 환경을 구축하기 위해 필요한 이미지 빌드 과정을 자동화 스크립트 형식으로 작성한 설정 파일**이다.

쉽게 말하면, Dockerfile은 다음 질문에 답하는 파일이다.

> 이 이미지는 어떤 OS/런타임에서 시작하고, 어떤 파일을 넣고, 어떤 패키지를 설치하고, 어떤 명령으로 실행할 것인가?

---

## 6.2 `FROM`

### 역할

`FROM`은 이미지의 출발점, 즉 베이스 이미지를 정하는 명령어다.

자료에서는 이를 “우리가 그리고자 하는 그림(이미지)에 대한 캔버스를 정의하는 명령어”라고 설명한다.

| 항목 | 설명 |
|---|---|
| 핵심 역할 | 베이스 이미지 지정 |
| 작성 위치 | 특별한 경우가 아니라면 Dockerfile에서 가장 먼저 작성 |
| 권장 | 좋은 품질의 Official Image 활용 |
| 기술적 역할 | 독립된 범위인 Build Stage 초기화 |

예시:

```dockerfile
FROM nginx
```

### 플랫폼 지정

이미지는 다양한 플랫폼별로 작성될 수 있다.

| 플랫폼 예시 | 설명 |
|---|---|
| `linux/amd64` | 일반적인 x86_64 Linux 환경 |
| `linux/arm64` | ARM64 Linux 환경 |
| `windows/amd64` | Windows amd64 환경 |

다중 플랫폼 이미지를 참조하는 경우 `--platform` 옵션으로 선택할 수 있다. 기본값은 빌드를 수행하는 플랫폼이다.

예시:

```dockerfile
FROM --platform=linux/amd64 nginx
```

### `AS` 별칭

`AS` 구문을 통해 Build Stage에 별칭을 부여할 수 있다.

```dockerfile
FROM node AS builder
```

Multi-stage build에서 이 별칭을 참조할 수 있다.

### tag와 digest

이미지 버전 지정 방식은 `tag`와 `digest`가 있다.

| 방식 | 특징 | 예시 |
|---|---|---|
| tag | 덮어쓰기가 가능 | `nginx:latest`, `ubuntu:20.04` |
| digest | 덮어쓰기가 불가능 | `nginx@sha256:...` |

| 표기 | 의미 |
|---|---|
| `:` | tag 명시 |
| `@` | digest 명시 |
| 기본값 | `:latest` |

### `FROM`보다 먼저 올 수 있는 유일한 명령어: `ARG`

자료에서는 `ARG`가 `FROM`보다 먼저 사용될 수 있는 유일한 명령어라고 설명한다.

사용 목적:

| 목적 | 설명 |
|---|---|
| 빌드 시점 버전 정의 | Dockerfile 작성 시점이 아니라 빌드 시점에 이미지 버전을 정하고 싶을 때 |
| tag 변수화 | 이미지 tag 정보를 변수화하고 `ARG`를 통해 전달 |

예시:

```dockerfile
ARG NODE_VERSION=20
FROM node:${NODE_VERSION}
```

### Multi-stage build에서 `FROM AS` 활용

자료의 예시는 하나의 Dockerfile에 두 가지 프로세스가 있는 상황이다.

| 단계 | 역할 |
|---|---|
| Stage #1 | Node를 통해 빌드 |
| Stage #2 | nginx를 통해 배포 |

`COPY --from`을 통해 이전 Stage의 결과물을 복사할 수 있고, `docker build --target` 옵션으로 빌드 대상을 지정할 수 있다.

---

## 6.3 `COPY`

### 역할과 사용법

`COPY`는 빌드할 때 파일이나 디렉토리를 컨테이너 내부로 복사한다.

```dockerfile
COPY <src> <dest>
```

| 항목 | 설명 |
|---|---|
| `<src>` | 복사할 원본 파일/디렉토리 |
| `<dest>` | 컨테이너 내부 목적지 경로 |
| 여러 개의 `<src>` | 2개 이상 기술 가능 |
| 목적지 자동 생성 | `<dest>` 경로를 미리 만들지 않아도 자동 생성 가능 |

옵션:

| 옵션 | 설명 | 조건 |
|---|---|---|
| `--chown` | 파일 소유권 수정 | Linux 기반 컨테이너에서 유효 |
| `--chmod` | 파일 허가권 수정 | Linux 기반 컨테이너에서 유효 |

예시:

```dockerfile
COPY ./index.html /usr/share/nginx/html/
```

### COPY 더 알아보기

| 내용 | 설명 |
|---|---|
| 정규식 사용 | `<src>` 정보를 정규식으로 기술 가능 |
| 정규식 규칙 | Go 언어의 `filepath.Match` 룰을 따름 |
| 빌드 컨텍스트 | `<src>`는 반드시 빌드 컨텍스트 내부 경로여야 함 |
| 디렉토리 복사 | `<src>`가 디렉토리라면 하위 파일/디렉토리가 복사됨 |
| 디렉토리 자체 | `<src>` 자체는 복사되지 않음 |
| 여러 `<src>` | `<src>`가 여러 개라면 `<dest>`는 반드시 디렉토리여야 함 |
| 캐시 영향 | `<src>` 대상이 변경되면 그 이후 캐시가 사라지므로 순서에 주의 |

캐시 관점에서 자주 바뀌는 파일을 너무 앞에서 `COPY`하면 이후 명령어의 캐시가 모두 깨질 수 있다.

### `COPY`와 `ADD` 비교

| 명령어 | 기능 |
|---|---|
| `COPY` | 파일/디렉토리 복사 |
| `ADD` | `COPY` 기능 + URL source 가능 + tarball 자동 압축 해제 |

자료의 권장 방향:

> 특별한 경우가 아니라면 `COPY`를 통해 명시적으로 기술하는 것이 좋다.

`ADD`는 편리하지만 동작이 더 많기 때문에 의도가 불분명해질 수 있다.

---

## 6.4 `WORKDIR`

### 역할과 사용법

`WORKDIR`은 컨테이너 내부 작업 디렉토리를 설정한다.

```dockerfile
WORKDIR /app
```

| 역할 | 설명 |
|---|---|
| 디렉토리 생성 | 해당 디렉토리가 없으면 생성 |
| 이동 | 해당 디렉토리로 이동 |
| 다음 명령어 기준 경로 | 이후 `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, `ADD` 등이 실행될 경로가 됨 |
| 기본값 | 기술하지 않으면 `/` |
| 권장 | 상대경로도 지원하지만 가독성을 위해 절대경로 사용 권장 |

`WORKDIR`을 지정하면 의도치 않은 경로에서 명령어가 실행되는 일을 줄일 수 있다.

---

## 6.5 `EXPOSE`

### 역할과 사용법

`EXPOSE`는 해당 이미지가 실행될 때 컨테이너가 사용할 네트워크 포트를 문서화하는 명령어다.

```dockerfile
EXPOSE 80
```

| 항목 | 설명 |
|---|---|
| 작성 값 | 컨테이너가 내부적으로 사용할 포트 번호 |
| 프로토콜 | `tcp`, `udp` 지정 가능 |
| 기본 프로토콜 | `tcp` |

### 중요한 주의사항

`EXPOSE`는 실제로 컨테이너 포트를 외부에 노출하지 않는다.

| 오해 | 실제 |
|---|---|
| `EXPOSE 80`을 쓰면 호스트에서 80번 포트로 접근 가능하다 | 접근 불가능할 수 있음 |
| `EXPOSE`가 포트 공개 기능이다 | 실제 기능이 아니라 “명세” 역할이다 |

실제로 포트를 노출하려면 `docker run`에서 `-p` 또는 `-P`를 사용해야 한다.

| 옵션 | 설명 |
|---|---|
| `-P` | `EXPOSE`로 선언된 컨테이너 포트를 호스트의 임의 포트에 매핑 |
| `-p` | `EXPOSE`와 무관하게 직접 지정한 `호스트:컨테이너` 포트를 매핑 |

예시:

```bash
docker run -p 8080:80 nginx
```

추가로, 같은 Docker 네트워크 안의 컨테이너끼리는 `-p`/`-P` 없이도 상대 컨테이너가 listen 중인 포트로 통신할 수 있다.

여러 포트를 한 번에 선언할 수도 있다.

```dockerfile
EXPOSE 80 443
```

---

## 6.6 `CMD`

### 역할

`CMD`는 컨테이너 실행 시 수행할 **기본 명령어**를 정의한다.

자료에서는 `RUN`처럼 `exec` 형태와 `shell` 형태로 기술할 수 있다고 설명한다.

| 항목 | 설명 |
|---|---|
| 실행 시점 | 컨테이너 실행 시 |
| 개수 | 하나의 Stage에 하나의 `CMD`만 유효 |
| 여러 개 작성 시 | 가장 마지막 `CMD`만 유효 |
| 덮어쓰기 | `docker run` 실행 시 명령어를 주면 `CMD`를 덮어씀 |

예시:

```dockerfile
CMD ["python", "app.py"]
```

### `docker run` 명령어가 `CMD`를 덮어쓰는 경우

```bash
docker run my-img python script.py
```

이 경우 Dockerfile의 `CMD`는 통째로 교체되고, `python script.py`가 실행된다.

주의할 점은 **추가되는 것이 아니라 교체되는 것**이다.

---

### `CMD`를 `ENTRYPOINT`와 함께 사용하는 경우

자료 예시:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

이 경우 최종 실행 명령은 다음 형태다.

```text
ENTRYPOINT + CMD
```

| 실행 명령 | 실제 실행 |
|---|---|
| `docker run my-img` | `python app.py` |
| `docker run my-img script.py` | `python script.py` |

즉, `ENTRYPOINT`는 실행 파일을 고정하고, `CMD`는 기본 인자를 제공한다.

### ENTRYPOINT 재정의

`ENTRYPOINT`는 `--entrypoint` 옵션으로 재정의할 수 있다.

```bash
docker run --entrypoint <새로운_실행파일> <이미지> [명령어와_인자]
```

예시:

```bash
docker run -it --entrypoint bash my-img
```

이 명령은 기존 ENTRYPOINT를 덮어쓰고 bash 셸로 진입한다.

### CMD의 일반적인 활용

자료에서는 `CMD`가 `ENTRYPOINT`의 파라미터 용도로 사용할 수 있지만, 실무에서는 자주 사용하지 않는다고 설명한다. 대신 Docker 이미지가 기본적으로 제공하는 실행 기능이 `CMD`에 정의되어 있는 경우가 많다.

예를 들어 Python 공식 이미지의 경우 별도 명령이 없을 때 Python 코드를 즉시 입력하고 실행할 수 있는 대화형 환경(REPL)을 기본값으로 제공할 수 있다.

---

## 6.7 `RUN`

### 역할과 사용법

`RUN`은 **이미지가 빌드될 때 실행될 명령어**를 정의한다.

```dockerfile
RUN pip install -r requirements.txt
```

| 항목 | 설명 |
|---|---|
| 실행 시점 | 이미지 빌드 시 |
| 목적 | 패키지 설치, 파일 생성, 시스템 설정 등 |
| 형태 | shell form, exec form 두 가지 |

### shell form

플랫폼에 따라 기본 shell이 설정된다.

| 플랫폼 | 기본 shell |
|---|---|
| Linux | `/bin/sh -c` |
| Windows | `cmd /S /C` |

여러 줄 작성은 backslash(`\`)를 통해 가능하다.

```dockerfile
RUN apt-get update && \
    apt-get install -y nginx
```

`SHELL` 지시어를 통해 기본 shell을 변경할 수도 있다.

### exec form

exec form은 JSON array로 작성한다.

```dockerfile
RUN ["echo", "hello"]
```

주의사항:

| 주의 | 설명 |
|---|---|
| 따옴표 | JSON array이므로 쌍따옴표 사용 |
| backslash | escape 처리 필요 |
| shell processing | shell 기능이 필요하면 `sh -c` 또는 shell form 고려 |

---

### `RUN`, `CMD`, `ENTRYPOINT` 비교

| 구분 | RUN | CMD | ENTRYPOINT |
|---|---|---|---|
| 실행 시점 | 이미지 빌드 시 | 컨테이너 실행 시 | 컨테이너 실행 시 |
| 용도 | 이미지에 필요한 설정/설치 작업 | 기본 실행 명령 또는 기본 인자 | 반드시 실행되어야 할 기본 실행 파일/명령 |
| 덮어쓰기 | 해당 없음 | 가능 | 기본적으로 어렵지만 `--entrypoint`로 가능 |

---

### 레이어와 커밋

Dockerfile의 각 명령어는 Docker 이미지에 새로운 레이어를 생성한다.

| 개념 | 설명 |
|---|---|
| Layer | 이미지가 쌓이는 단위 |
| Image | 여러 레이어가 쌓여 만들어진 최종 결과 |
| Commit | 레이어를 확정하는 것 |
| RUN | 이미지가 빌드될 때 이미지 레이어에서 수행할 커맨드 |

`RUN`이 실행되면 커밋을 통해 레이어가 확정된다.

---

### 레이어 캐시 이해하기

이미지의 각 레이어를 생성하는 작업은 빌드할 때마다 반복될 수 있다. Docker는 이전 빌드와 차이가 없다면 기존 레이어를 재사용한다. 이를 **레이어 캐싱**이라고 한다.

| 원리 | 설명 |
|---|---|
| 변경 없음 | 기존 레이어 캐시 사용 |
| 변경 발생 | 해당 레이어부터 다시 빌드 |
| 캐시 무효화 | 한 번 캐시가 깨지면 그 아래 모든 명령어의 캐시도 무효화 |
| 작성 순서 | 자주 변하는 명령어는 최대한 뒤로 배치하는 것이 좋음 |

`COPY` 명령어가 자주 바뀌는 파일을 너무 앞에서 복사하면 이후 모든 캐시가 깨질 수 있다.

---

### `apt-get` 사용 시 주의

패키지 설치는 `RUN` 명령어의 대표적인 사용 사례다.

| 명령어 | 설명 |
|---|---|
| `apt-get update` | 패키지 레포지토리의 최신 정보를 업데이트 |
| `apt-get install` | 원하는 패키지를 다운로드/설치 |

문제 상황:

```dockerfile
RUN apt-get update
RUN apt-get install -y nginx mysql
```

Dockerfile이 바뀌었을 때 `RUN apt-get update`는 캐시되어 유지되고, `RUN apt-get install ...`만 다시 실행될 수 있다. 이 경우 패키지 목록은 최신이 아닌데 설치 명령만 다시 실행되어 의도하지 않은 버전 문제가 생길 수 있다.

더 바람직한 방식:

```dockerfile
RUN apt-get update && \
    apt-get install -y nginx mysql
```

자료에서는 다음을 권장한다.

| 권장 | 이유 |
|---|---|
| 하나의 `RUN` 명령어로 묶기 | 캐싱 단위를 통일하기 위해 |
| 버전을 명시적으로 기술하기 | 의도하지 않은 버전 설치 방지 |

---

## 6.8 `USER`

### 역할

`USER`는 특정 사용자를 지정하여 명령어를 수행해야 할 때 사용한다.

영향을 받는 명령어:

| 명령어 | 설명 |
|---|---|
| `CMD` | 컨테이너 실행 시 기본 명령 |
| `RUN` | 이미지 빌드 시 실행 명령 |
| `ENTRYPOINT` | 컨테이너 실행 시 고정 실행 명령 |

`USER`를 지정하지 않으면 루트 권한으로 명령어가 수행된다.

### 왜 root가 위험한가?

| 위험 | 설명 |
|---|---|
| 개발자 실수 | 시스템에 영향을 미치는 명령어를 실수로 실행할 수 있음 |
| 악의적 사용자 | 취약점 악용 시 더 큰 피해 발생 가능 |
| 권한 과다 | 애플리케이션 실행에 불필요한 권한을 가짐 |

자료에서는 가능하면 **최소 필요한 권한만 가지는 유저로 실행**하도록 권장한다.

### 사용법

`USER`는 두 가지 형태로 작성할 수 있다.

```dockerfile
USER user
USER user:group
```

`group` 또는 GID가 생략되면 user의 primary group 또는 root group으로 설정된다. 이후 작성된 명령어는 해당 user로 실행된다.

### 유의사항

| 주의사항 | 설명 |
|---|---|
| USER는 사용자를 생성하지 않음 | 존재하지 않는 user라면 `RUN`으로 직접 생성해야 함 |
| USER만으로 root 접속을 완전히 막을 수 없음 | `docker exec -it -u root foo /bin/bash`처럼 root 접속 가능 |
| 추가 보안 필요 | root 접속을 제한하려면 별도 보안 조치 필요 |

---

## 7. Dockerfile 개념 및 실습

## 7.1 nginx 기반 Dockerfile 이미지 실행 실습

### 실습 준비

자료에서는 다음 파일을 생성한다.

| 파일 | 역할 |
|---|---|
| `Dockerfile` | nginx 기반 이미지를 만들기 위한 빌드 설정 |
| `index.html` | nginx가 제공할 HTML 파일 |

---

### Dockerfile 작성

자료 화면에 나온 Dockerfile은 다음과 같다.

```dockerfile
FROM nginx
COPY ./index.html /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

| 줄 | 설명 |
|---|---|
| `FROM nginx` | nginx 공식 이미지를 베이스 이미지로 사용 |
| `COPY ./index.html /usr/share/nginx/html/` | 현재 디렉토리의 `index.html`을 nginx 기본 HTML 경로로 복사 |
| `EXPOSE 80` | 컨테이너가 내부적으로 80번 포트를 사용한다는 명세 작성 |
| `CMD ["nginx", "-g", "daemon off;"]` | 컨테이너 실행 시 nginx를 foreground로 실행 |

`daemon off;`를 사용하는 이유는 컨테이너의 메인 프로세스가 종료되지 않도록 하기 위해서다. nginx가 백그라운드 데몬으로 빠져버리면 컨테이너가 종료될 수 있다.

---

### index.html 작성

자료 화면에 나온 `index.html` 내용은 다음과 같다.

```html
<h1> this is custom image from docker.</h1>
```

이 파일을 nginx 기본 정적 파일 경로에 복사하므로, 브라우저에서 접속하면 기본 nginx 페이지 대신 이 HTML 내용이 출력될 수 있다.

---

### 이미지 빌드

터미널에서 다음 명령어로 이미지를 빌드한다.

```bash
docker build -t mynginx:1.0 .
```

| 부분 | 의미 |
|---|---|
| `docker build` | Dockerfile을 사용해 이미지 빌드 |
| `-t mynginx:1.0` | 이미지 이름을 `mynginx`, 버전을 `1.0`으로 태그 지정 |
| `.` | 현재 디렉토리를 빌드 컨텍스트로 사용 |

---

### 컨테이너 실행

빌드된 이미지를 다음 명령어로 실행한다.

```bash
docker run -p 8080:80 mynginx:1.0
```

| 부분 | 의미 |
|---|---|
| `-p 8080:80` | 호스트 8080 포트를 컨테이너 80 포트로 매핑 |
| `mynginx:1.0` | 방금 빌드한 이미지 실행 |

---

### 브라우저에서 확인

브라우저에서 다음 주소로 접속한다.

```text
http://localhost:8080
```

정상 동작 시 다음 중 하나를 확인할 수 있다.

| 상황 | 출력 |
|---|---|
| 기본 nginx 파일이 유지된 경우 | nginx 기본 페이지 |
| `index.html` 복사가 정상 반영된 경우 | `this is custom image from docker.` 내용 |

---

## 7.2 FastAPI 웹 애플리케이션 이미지 빌드 실습

자료의 마지막 실습은 **간단한 FastAPI 웹 애플리케이션 이미지를 빌드하는 것**이다.

### 실습 목표

제공된 To-Do 웹사이트 프로젝트를 분석하여 Dockerfile을 작성하고, Docker Desktop을 활용해 이미지를 빌드한 뒤 컨테이너를 실행하여 웹사이트가 정상 구동되는지 확인한다.

### 실습 준비 단계

| 단계 | 내용 |
|---|---|
| 1 | VSCode 다운로드 |
| 2 | 사전에 제공된 TODO 파일 준비 |
| 3 | VSCode에서 `Open Folder` 클릭 |
| 4 | 제공된 Docker 관련 파일 또는 프로젝트 폴더 열기 |
| 5 | 우측 상단 파일 아이콘 클릭 |
| 6 | 파일명을 `Dockerfile`로 입력 |
| 7 | Dockerfile 작성 |

### FastAPI Dockerfile에 들어갈 주요 명령어

자료에서는 Dockerfile 작성 시 다음 지시어를 사용한다고 정리한다.

| 명령어 | 역할 |
|---|---|
| `FROM` | 베이스 이미지 설정 |
| `WORKDIR` | 컨테이너 안의 작업 디렉토리를 `/app`으로 설정 |
| `COPY` | 현재 디렉토리의 모든 파일을 컨테이너의 `WORKDIR`로 복사 |
| `RUN pip install -r requirements.txt` | 의존성 설치 |
| `EXPOSE` | 컨테이너가 사용할 포트 명시 |
| `CMD` | 컨테이너 실행 시 앱을 구동할 명령어 작성 |

자료에는 FastAPI 실습의 최종 Dockerfile 전체 코드는 제시되지 않고, 작성해야 할 구성 요소가 제시되어 있다. 일반적으로는 다음과 같은 형태가 된다.

```dockerfile
FROM python:3.10

WORKDIR /app

COPY . /app

RUN pip install -r requirements.txt

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

위 코드는 자료의 구성 요소를 바탕으로 한 이해용 예시다. 실제 프로젝트의 진입 파일이 `main.py`가 아니거나 FastAPI 객체 이름이 `app`이 아니면 `CMD` 부분은 프로젝트 구조에 맞게 수정해야 한다.

---

## 8. 실습 관점 핵심 정리

### 8.1 Image와 Container 구분

| 개념 | 비유 | 설명 |
|---|---|---|
| Image | 설계도/템플릿 | 컨테이너를 만들기 위한 읽기 전용 기준 |
| Container | 실행된 결과물 | 이미지를 실제로 실행한 상태 |

---

### 8.2 `build`, `run`, `exec` 구분

| 명령어 | 언제 쓰는가 | 결과 |
|---|---|---|
| `docker build` | Dockerfile로 이미지를 만들 때 | Image 생성 |
| `docker run` | Image로 새 컨테이너를 만들고 실행할 때 | Container 생성 및 실행 |
| `docker exec` | 이미 실행 중인 컨테이너 안에서 명령을 실행할 때 | 기존 Container 내부 작업 |

---

### 8.3 포트 관련 핵심

| 개념 | 설명 |
|---|---|
| 컨테이너 내부 포트 | 컨테이너 안의 애플리케이션이 listen하는 포트 |
| 호스트 포트 | 내 PC/서버에서 외부 요청을 받는 포트 |
| `-p 8080:80` | 호스트 8080 → 컨테이너 80 |
| `EXPOSE 80` | 문서화/명세 역할. 실제 공개는 아님 |

가장 흔한 실수는 `EXPOSE`를 적었기 때문에 자동으로 외부에서 접속 가능하다고 생각하는 것이다. 실제 외부 접속을 위해서는 `docker run -p`가 필요하다.

---

### 8.4 Dockerfile 작성 순서 감각

좋은 Dockerfile은 단순히 동작만 하는 파일이 아니라, 캐시와 보안을 고려한다.

| 고려사항 | 설명 |
|---|---|
| `FROM`은 처음에 | 베이스 이미지와 Build Stage를 정함 |
| 자주 안 바뀌는 작업은 앞쪽 | 캐시를 오래 활용 가능 |
| 자주 바뀌는 파일 복사는 뒤쪽 | 캐시 무효화를 줄임 |
| `RUN apt-get update && apt-get install`은 묶기 | 패키지 목록과 설치 캐시 문제 방지 |
| `USER` 사용 고려 | root 권한 실행을 줄여 보안 개선 |
| `EXPOSE`만 믿지 않기 | 실제 포트 공개는 `docker run -p` |

---

## 9. 페이지별 내용 반영표

| 페이지 | 반영 내용 |
|---|---|
| 1~2 | Docker Container 기초, 가상머신 vs 컨테이너 섹션 시작 |
| 3 | 개발자의 영원한 숙제, 환경 불일치 문제 |
| 4 | 서버 운영의 역사: Bare Metal → Virtualized → Containerized |
| 5 | Bare Metal 시대의 특징과 한계 |
| 6 | Virtual Machine 시대의 특징, 장점, 단점 |
| 7 | Containerized 시대의 특징, 장점, 보안상 한계 |
| 8 | 컨테이너가 독립 서버처럼 보이는 원리, chroot/namespace/cgroup |
| 9 | chroot의 역할과 한계 |
| 10 | namespace의 역할, PID/네트워크/호스트명 격리 |
| 11 | cgroup의 역할, CPU/메모리/디스크 I/O/네트워크/프로세스 제한 |
| 12 | chroot + namespace + cgroup = 컨테이너 엔진 |
| 13 | VM vs Container 비교표 |
| 14~15 | Docker 개념 및 장점 섹션 시작 |
| 16~18 | Docker 기본 개념, 컨테이너 구성 요소, 커널 공유, namespace/cgroup |
| 19 | Docker 주요 장점: 유연성, 경량성, 교체 가능성, 이식성, 확장성 |
| 20~21 | Docker 구성 요소, 클라이언트-서버 모델 |
| 22 | Docker Client 설명과 명령어 예시 |
| 23 | Docker Host, daemon, image, container 설명 |
| 24 | Registry와 Docker Hub 설명 |
| 25~26 | Docker 주요 컴포넌트: Infra, Host OS, Docker, Node, Swarm, Service, Stack |
| 27 | Docker 이론 정리 |
| 28~30 | `docker build` 의미와 옵션 |
| 31~36 | `docker run` 의미, 옵션, 예제, 포트 매핑 실수 |
| 37~38 | `docker ps` 의미와 옵션 |
| 39~41 | `docker rm` 의미와 예제 |
| 42~43 | `docker exec` 의미와 옵션 |
| 44~52 | Docker Desktop 설치, 라이선스, 설치 확인 |
| 53~63 | Docker Hub에서 nginx 찾기, 이미지 pull, run, 포트 포워딩, 실행 확인 |
| 64~65 | Dockerfile 명령어 섹션 시작 |
| 66~70 | `FROM`, platform, tag/digest, `ARG`, Multi-stage `AS` 활용 |
| 71~73 | `COPY`, `ADD` 비교, 빌드 컨텍스트와 캐시 주의 |
| 74 | `WORKDIR` 역할과 사용법 |
| 75~77 | `EXPOSE` 역할과 실제 포트 공개와의 차이 |
| 78~80 | `CMD`, `ENTRYPOINT`와 함께 쓰는 방식, 일반적인 활용 |
| 81~89 | `RUN`, shell/exec form, layer, commit, cache, apt-get 주의 |
| 90~92 | `USER` 역할, 사용법, 보안 유의사항 |
| 93~95 | Dockerfile 개념 및 실습 섹션, Dockerfile 소개 |
| 96~101 | nginx 기반 Dockerfile 실습: 파일 생성, Dockerfile/index.html 작성, build/run/browser 확인 |
| 102~107 | FastAPI 웹 애플리케이션 이미지 빌드 실습 흐름과 Dockerfile 구성 요소 |

---

## 10. 마지막 요약

Docker는 컨테이너 기술을 쉽게 사용할 수 있게 해주는 플랫폼이다. 컨테이너는 VM처럼 완전한 OS를 새로 띄우는 것이 아니라, Host OS 커널을 공유하면서 `chroot`, `namespace`, `cgroup` 같은 리눅스 기능을 조합해 독립된 실행 환경처럼 보이게 만든다.

Docker를 사용할 때는 Image와 Container의 차이를 이해해야 한다. Image는 실행 템플릿이고, Container는 그 이미지를 실제로 실행한 상태다. Dockerfile은 Image를 만드는 자동화 스크립트이며, `docker build`는 이미지를 만들고, `docker run`은 컨테이너를 실행하고, `docker exec`는 실행 중인 컨테이너 안에서 추가 작업을 수행한다.

실습에서 가장 중요한 감각은 다음이다.

```text
Dockerfile 작성
→ docker build로 이미지 생성
→ docker run으로 컨테이너 실행
→ -p 옵션으로 포트 연결
→ 브라우저/CLI/Docker Desktop으로 실행 확인
```

이 흐름을 이해하면 nginx뿐 아니라 FastAPI 같은 웹 애플리케이션도 컨테이너 이미지로 빌드하고 실행할 수 있다.

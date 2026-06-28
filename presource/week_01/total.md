## 🗓 이번 주 개요
- 주차: Week 01
- 주제
    - 언어 기초 (Python) & 함수 및 객체지향 설계 응용
    - 비동기(asyncio)와 테스트(pytest)
    - HTTP & API 기초
    - FastAPI & Pydantic(BaseModel 상속 처리)
    - DB 및 SQL 기초
    - 카카오크루 특강 (Zero - 카테캠 1기 수료생 및 카카오 선물하기 개발자)
- 키워드: Data Type, I/O, Type Casting, enumerate, Early Return, args, *kwargs, Magic Method, @dataclass, capsulation, sync, async, asyncio, pytest, HTTP, API, RESTful API, Pydantic, DB, SQL

## 학습 내용
### 1. 언어 기초 (Python) & 함수 및 객체지향 설계 응용
- Python에 대한 기본적인 문법은 알고 있었으나, 다음의 기능은 이번 강의를 통해 덩구 자세하게 알게 됐다.
    - strip() : 문자열의 양 옆 공백 제거
    - List Comprehension : for 한 줄로 리스트 구성.
        - Filtering, Type Casting 등을 할 수 있음
    - Early Return : 여러 체크가 필요한 로직에 대해서 만족하지 않는 것을 먼저 찾는 것
    - 가변 인자 (args, *kwargs) : 함수의 인자 개수가 정해지지 않았을 때 사용
    - Data Class : \__init__, \__repr__, \__eq__ 자동 생성
    - Capsulation : 데이터(상태)와 그 데이터를 다루는 기능(메서드)을 하나로 묶음
        - _name : "직접 접근 금지" 약속
        - @property : 안전하게 읽기만 허용 (검증된 통로를 만듦)
- 위 기능들을 배우고 나서 궁금한 점에 대해 추가 조사한 바는 다음과 같다.
    - 2개 이상의 Parent Class가 존재할 때
        - MRO() : Method Resolution Order
            - 상속받은 부모 class의 왼쪽부터 차례로 상속받게 되어 우선순위가 부여
        - super()를 통한 부모 클래스의 함수 실행
            - super() : MRO 순서로 실행
            - super(시작클래스, self) : 시작클래스의 다음 순서에 있는 클래스의 메서드를 찾아 실행
- 특히, 로직 작성 부분에서 nested if의 가독성이 굉장히 떨어진다고 생각하여 과거에는 'is_케이스'1, 'is_케이스2'와 같이 boolean 값으로 정리하여 && 또는 ||를 쓰는 형식을 적용해본 적이 있었다. 그런데 FastAPI 프레임워크를 활용해보면서 특별한 오류들을 Raise를 해야 하는 경우가 생기는데 이때 Early Return을 Gemini로부터 알게 됐고, 실제 서버에 적용해보면서 가독성이 많이 상승한 느낌을 받았었다. 별도의 강의로 배워본 적은 없었지만 카테캠 강의를 통해 이번 기회에 상기해보는 계기가 됐다.

### 2. 비동기(asyncio)와 테스트(pytest)
- 동기(sync)와 비동기(async)의 방식 차이는 숙지하고 있었으나, 이를 코드로 작성해보고 차이를 더욱 자세하게 알 수 있었다.
    - await을 통한 통제권 전이
    - run()을 통한 루프 시작
    - create_task()와 *task_list와 gather()를 통한 시작
    - sleep()을 통한 비동기 대기 방식
- 계층을 이해함으로써 비동기가 어떤식으로 통제권을 제어할 수 있는지를 알 수 있어 신기했다.
    - 코루틴: 비동기 함수 그 자체
    - 테스크: 루프에 예약되어 실행 대기 중인 작업
    - 루프: 모든 비동기 작업을 지휘하는 중앙 엔진
- 동기와 비동기 방식을 숙지하면서 비동기 방식이 항상 우수한 것으로 오인하고 있었고, 경우에 따라 골라 써야 함을 알게 됐다.
    - 비동기: I/O 등의 대기 시간이 긴 트래픽, 실시간 양방향 통신을 처리하는 데 우수
    - 동기: CPU 등의 대기 시간이 짧은 트래픽을 처리하는 데 우수
- 코드의 작동 테스트를 위해서 임의의 엣지 케이스를 생각해보고 직접 값을 넣거나 조작하여 테스트를 해 본적은 있으나, pytest와 같은 테스트 툴이 있다는 것을 새롭게 알게 됐다.
    - 크기에 따른 테스트 종류에 대해 더 자세하게 알게 됐다.
    - Rust에서 보던 assert 기능이 Python에도 있음을 알게 됐다.
    - 테스트 함수명(test_기능), with-raises, match, decotrator(fixture, mark(skip, skipif, xfail), Parametrize) 등의 문법 등을 통해 

### 3. HTTP & API 기초
- '데이터통신'과 '컴퓨터네트워크' 강의를 수강한 적이 있어 네트워크에 대해서는 어느 정도 숙지하고 있었고, 이번 강의를 통해 이를 상기하는 게기가 됐다.
    - 네트워크의 원리, IP 주소, Port 번호, 서브넷 마스크, DNS, 종류(WAN, MAN, LAN, PAN), Topology, TCP/IP, UDP
- FastAPI 프레임워크를 이용한 토이 프로젝트와 더불어 현재 수강중인 '소프트웨어공학'에서 배운 RESTful API에 대해 어느 정도 숙지하고 있었고, 이번 강의를 통해 고도화하는 계기가 됐다.
    - 브라우저에 링크를 입력하면 DNS로부터 IP 주소를 반환받아 TCP/IP 패킷으로 해당 IP 주소에 GET 요청을 하여 웹페이지를 받는다.
    - HTTP는 무상태성과 비연결성을 통해 연결 유지와 상태 보존을 하지 않아 요청마다 독립적인 처리가 가능하도록 설계됐다.
    - GET, POST, PUT, DELETE API 엔드포인트를 설계할 때 CRUD(Create, Read, Update, Delete)처럼 생각하면 쉽다는 생각이 들었다.
    - 패킷 자체(구성 요소, 속성 등)에 대해서도 자세히 알 수 있어 엔지니어링 관점에서의 지식을 얻을 수 있었다.
    - 더 나아가, 왜 TCP 뿐만 아니라 QUIC 등의 여러 프로토콜이 개발되고 HTTP 3.0으로 채택됐는지도 궁금한 점을 찾아보다가 알 수 있었다. 특히 TCP 고유의 Blocking 문제가 있음을 알게 됐다.

### 4. FastAPI & Pydantic(BaseModel 상속 처리)
- 과거 토이 프로젝트를 통해 MVC와 Layered Architecture 등 유지보수에 용이한 구조를 배웠었다.
- 그 다음 토이 프로젝트로 FastAPI로 API 엔드포인트를 구성해보고, 특정 엔드포인트에 대한 특정 Service-Repository 조합 또는 CRUD 함수를 만들어 유지보수가 좋은 코드를 작성해본 바가 있다. 그 과정에서 Pydantic이라는 것에 대해 알게 됐고 이를 적용함으로써 요청-응답 스키마 구성에 유용하게 써본 적이 있다.
- 이 강의를 통해서 과거에 배웠던 유지보수가 용이한 구조, 직전 강의에서 배웠던 네트워크 관련 지식, FastAPI를 통한 백엔드 서버 구축, Pydantic을 통한 요청-응답 스키마 구성을 다시금 상기해보는 계기가 됐다.

### 5. DB 및 SQL 기초
- 토이 프로젝트에서 로컬로 데이터를 저장하기 위해 sqlite3 라이브러리를 통해 ORM을 사용하지 않고 Raw SQL을 사용해본 경험이 있어 해당 내용을 숙지하는 데 어렵지 않았다.
- MySQL로 실습을 진행하던 '데이터베이스' 강의도 수강한 적이 있어 크게 어렵지 않았으나, 몰랐던 문법들이 있어 새롭게 느껴졌다.
    - SHOW TABLES: 테이블 목록 확인
    - DESC target_table: 테이블 구조 확인
    - ALTER TABLE target_table
        - ADD COLUMN name type constraint: 컬럼 추가
        - MODFIY COLUMN name type constraint: 컬럼 수정
        - CHANGE COLUMN target_column new_column type constraint: 컬럼 이름 변경
        - DROP COLUMN name: 컬럼 삭제
        - RENAME new_table_name: 테이블 이름 변경
        - ADD CONSTRAINT name constraint(속성): 제약조건 추가
- ERD에 대해서도 배운 적이 있어서 내용을 숙지하는 데 어렵지 않았고, 헷갈리던 부분을 다시 상기하는 계기가 됐다.

## 🧱 막혔던 지점 & 해결 과정
- 문제: Git에서 Conflict 해결하는 데 익숙치 않아 계속 오류 발생
- 시도: git status, branch 등 여러 git 명령어를 써보며 그 명령어가 어떤 역할을 할 지 숙지하고 차근차근 conflict를 찾아가기
- 해결: conflict가 발생한 부분을 발견하고 그 부분을 수정하기.
- 배운 점: conflict가 최대한 나지 않게 관리를 잘 하면 되지만, 협업의 특성 상 conflict 해결 능력이 필요하니 노력해서 기르자.

## 🔁 이번 주 회고 (KPT)
- Keep: 강의 내용을 숙지하고 궁금한 점을 AI에게 물어보며 지식 확장하기
- Problem: 평소에 Github Desktop(GUI)를 사용하다 보니 명령어에 익숙치 않아 conflict 해결 실습에서 영상 없이는 해결하기 어려웠음.
- Try: 앞으로 GUI보다는 CUI에서 Git을 익숙하게 쓰도록 노력하기, 추후에 conflict 최대한 나지 않게 branch 관리하기

## 🎯 다음 주 목표
- [ ] Git과 Github를 GUI 없이 꼭 명령어로만 사용해보기
- [ ] 다양한 Branch 구성과 PR, Merge를 통해 Conflict를 유발하고 이를 해결해보기
- [ ] 구상중인 토이 프로젝트를 FastAPI로 베이스 구성하기
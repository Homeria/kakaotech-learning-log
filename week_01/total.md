## 개요
- 주차: Week 01
- 주제: 언어 기초 (Python), 함수 및 객체지향 설계 응용
- 키워드: Data Type, I/O, Type Casting, enumerate, Early Return, args, *kwargs, Magic Method, @dataclass, capsulation

## 학습 내용
### 1. 언어 기초 (Python)
- Python에 대한 기본적인 문법은 알고 있었으나, 다음의 기능은 이번 강의를 통해 알게 됐다.
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
    




## 극복 지점 및 과정

## 회고

## 목표

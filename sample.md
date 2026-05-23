## 🗓 이번 주 개요
- 주차: Week 01 (5/18~5/22) — 언어 심화
- 키워드: #Python #OOP #async #FastAPI #SQL기초

## 📚 이번 주 학습한 것

### 1. 함수 및 객체지향 설계 응용
- 함수를 "하나의 기능 단위"로 작게 쪼개는 연습을 했다.
  특히 Early Return을 쓰니 if 중첩이 사라져서 코드 가독성이 확 달라졌다.
- Data Class는 "값을 담는 그릇"이라고 이해했다.
  일반 클래스로 쓰면 `__init__`을 매번 써야 하는데, `@dataclass` 하나면 끝나는 게 신기했다.
- 예제: 사용자 정보를 dict로 넘기던 걸 `User` Data Class로 바꾸니 어떤 필드가 있는지
	IDE가 알려줘서 실수가 줄었다.

### 2. FastAPI & Pydantic
- FastAPI는 "스키마를 먼저 정하고, 그걸로 입출력을 강제하는" 프레임워크라는 감이 왔다.
- Pydantic `BaseModel`로 요청 본문을 정의하니,
  잘못된 타입이 들어오면 알아서 422 에러를 돌려줘서 편했다.
- Swagger UI가 자동으로 생성되는 것도 인상 깊었다. API 문서를 따로 안 만들어도 된다니!

### 3. DB 및 SQL 기초
- ERD를 직접 그려 보며 "테이블은 결국 현실 세계의 관계를 쪼갠 것" 이라는 관점이 잡혔다.
- 아직 SQL은 SELECT 정도만 익숙하고, JOIN은 다음 주에 제대로 붙어볼 예정.

## 🧱 막혔던 지점 & 해결 과정
- 문제: FastAPI에서 `POST` 요청을 보냈는데 계속 `422 Unprocessable Entity`
- 시도: Swagger UI에서 직접 테스트 → 본문을 form으로 보냈던 게 원인이었음
- 해결: `Content-Type: application/json` 으로 바꾸고, Pydantic 모델 필드명과 요청 JSON 키를 정확히 맞춤. 이후 정상 동작.
- 배운 점: 에러 메시지를 읽으면 대부분 답이 나와 있다. 메시지를 끝까지 읽자.

## 🔁 이번 주 회고 (KPT)
- Keep: 배운 내용을 바로 코드로 쳐보고 실행 결과를 확인하는 습관
- Problem: 강의를 1.5배속으로 넘기다가 async 부분을 한 번에 이해 못 함. 결국 다시 봄
- Try: 어려운 주제는 처음엔 1배속 + 직접 따라 코딩으로 보기

## 🎯 다음 주 목표
- [ ] JOIN 3종류(INNER/LEFT/RIGHT)를 예제 문제로 막힘 없이 쓰기
- [ ] JavaScript 기초에서 `let/const/var` 차이를 내 언어로 설명해 보기
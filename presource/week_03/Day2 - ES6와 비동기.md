# Day 2. ES6 문법과 비동기 처리 정리

> 원본 PDF: `Day 2. ES6와 비동.pdf`  
> 주제: ES6 문법, 배열 메서드, 내장 객체, 비동기 처리, Promise, async/await

---

# 1. ES6 문법 개요

## 1.1 ECMAScript와 ES6

자바스크립트는 1997년부터 **ECMAScript**라는 사양으로 표준화되어 왔다.  
ES6는 **ECMAScript 2015**를 의미하며, 2015년에 업데이트되면서 자바스크립트 문법에 큰 변화가 도입되었다.

대표적으로 `let`, `const`, 화살표 함수, 구조 분해 할당, Spread 문법, 템플릿 리터럴 등이 있다.

---

## 1.2 ES6 이후 도입된 주요 문법

| 문법 | 예시 | 설명 |
|---|---|---|
| `let`, `const` | `let name = "Elice";` | 변수를 선언하는 새로운 방식 |
| 화살표 함수 | `() => { ... }` | 함수를 간결하게 작성하는 문법 |
| 구조 분해 할당 | `const { name, age } = user;` | 객체나 배열의 값을 쉽게 변수에 할당 |
| Spread 문법 | `const arr = [...arr1, ...arr2];` | 배열이나 객체를 펼쳐서 사용 |
| 템플릿 리터럴 | `` `Hello ${name}` `` | 문자열 안에 변수나 표현식을 쉽게 삽입 |
| 옵셔널 체이닝 | `user.contact?.email` | 존재하지 않는 값에 안전하게 접근 |
| 프로퍼티/메서드 축약 | `{ name, age }` | 객체 작성 시 중복을 줄이는 문법 |
| 배열 메서드 | `forEach`, `map`, `filter`, `reduce` | 배열 데이터를 효율적으로 처리 |

---

# 2. `let`과 `const`

## 2.1 `let`, `const`의 등장

ES6 이전에는 변수를 선언할 때 주로 `var`를 사용했다.  
ES6 이후에는 `var` 대신 `let`과 `const` 사용이 권장된다.

```javascript
let apple = "사과";
const banana = "바나나";
```

---

## 2.2 `let`

`let`은 **값 변경이 가능한 변수**를 선언할 때 사용한다.  
선언과 초기화를 동시에 하지 않아도 된다.

```javascript
let fruit = "오렌지";
fruit = "포도";

let elice;
elice = "hellobit";
```

### 핵심

| 항목 | 설명 |
|---|---|
| 값 변경 | 가능 |
| 선언과 초기화 | 동시에 하지 않아도 됨 |
| 중복 선언 | 불가능 |
| 스코프 | 블록 레벨 스코프 |

---

## 2.3 `const`

`const`는 **값 변경이 불가능한 변수**, 즉 상수를 선언할 때 사용한다.  
선언과 초기화를 반드시 동시에 해야 한다.

```javascript
const fruit = "오렌지";

fruit = "포도"; // error

const elice; // error
```

### 핵심

| 항목 | 설명 |
|---|---|
| 값 변경 | 불가능 |
| 선언과 초기화 | 반드시 동시에 해야 함 |
| 중복 선언 | 불가능 |
| 스코프 | 블록 레벨 스코프 |

---

## 2.4 `var` vs `let`, `const`

`var`는 변수 중복 선언이 허용된다.  
반면 `let`과 `const`는 변수 중복 선언 시 에러가 발생한다.

```javascript
var fruit = "오렌지";
var fruit = "포도"; // 가능

let food = "사과";
let food = "바나나"; // error

const name = "Elice";
const name = "Hellobit"; // error
```

---

## 2.5 변수 선언 권장 방식

| 상황 | 권장 키워드 |
|---|---|
| 값이 바뀌지 않는 경우 | `const` |
| 값이 바뀌어야 하는 경우 | `let` |
| 기존 코드 유지 외에는 사용 비추천 | `var` |

일반적으로는 먼저 `const`를 사용하고, 값 변경이 필요한 경우에만 `let`을 사용하는 방식이 좋다.

---

# 3. 변수와 스코프

## 3.1 스코프란?

**스코프(Scope)**란 변수를 참조할 수 있는 유효 범위를 의미한다.

변수는 어디서 선언되었는지에 따라 접근 가능한 범위가 달라진다.

---

## 3.2 전역 스코프와 지역 스코프

| 종류 | 설명 |
|---|---|
| 전역 스코프 | 가장 바깥의 코드 영역 |
| 지역 스코프 | 함수 및 코드 블록 내부 |

---

## 3.3 전역 스코프

전역 스코프는 코드의 가장 바깥 영역이다.  
전역 스코프에서 선언한 변수는 여러 위치에서 접근할 수 있다.

```javascript
let a = 1;

function func() {
  console.log(a);
}

func(); // 1
```

---

## 3.4 지역 스코프

지역 스코프는 함수 내부 또는 코드 블록 내부의 범위이다.  
지역 스코프에서 선언한 변수는 보통 해당 영역 안에서만 접근할 수 있다.

```javascript
function func() {
  let b = 2;
  console.log(b);
}

func();

console.log(b); // error
```

---

## 3.5 함수 레벨 스코프와 블록 레벨 스코프

지역 스코프를 어디까지 인정하는지에 따라 함수 레벨 스코프와 블록 레벨 스코프로 나눌 수 있다.

| 구분 | 설명 | 해당 키워드 |
|---|---|---|
| 함수 레벨 스코프 | 함수 내부만 지역 스코프로 인정 | `var` |
| 블록 레벨 스코프 | 함수 내부와 코드 블록 내부를 지역 스코프로 인정 | `let`, `const` |

---

## 3.6 함수 레벨 스코프 - `var`

`var`는 함수 레벨 스코프를 지원한다.  
따라서 `if`문 내부에서 `var`로 선언한 변수는 블록 밖에서도 접근될 수 있다.

```javascript
if (true) {
  var b = 2;
}

console.log(b); // 2
```

---

## 3.7 블록 레벨 스코프 - `let`, `const`

`let`과 `const`는 블록 레벨 스코프를 지원한다.  
따라서 `if`문 내부에서 선언한 변수는 블록 밖에서 접근할 수 없다.

```javascript
if (true) {
  let b = 2;
}

console.log(b); // error
```

---

# 4. 스코프 체인과 렉시컬 스코프

## 4.1 스코프 체인

**스코프 체인**은 모든 스코프가 계층적 구조로 연결된 것을 의미한다.

변수나 함수를 참조할 때 자바스크립트는 현재 스코프에서 먼저 찾고, 없으면 상위 스코프로 이동하면서 순차적으로 검색한다.

```javascript
let global = 10;

function outerFunc() {
  let local1 = 30;

  function innerFunc() {
    let local2 = 50;
    console.log(global);
    console.log(local1);
    console.log(local2);
  }

  innerFunc();
}
```

위 코드에서 `innerFunc`는 자기 자신의 지역 변수뿐만 아니라 상위 함수의 변수, 전역 변수에도 접근할 수 있다.

---

## 4.2 스코프 체인의 핵심

| 내용 | 설명 |
|---|---|
| 현재 스코프에서 먼저 검색 | 변수를 현재 위치에서 먼저 찾는다. |
| 없으면 상위 스코프로 이동 | 바깥 스코프로 올라가며 검색한다. |
| 최상위는 전역 스코프 | 모든 지역 스코프의 최상위는 전역 스코프이다. |
| 아래 방향 검색은 불가능 | 바깥 스코프에서 안쪽 지역 변수에 접근할 수 없다. |

---

## 4.3 렉시컬 스코프

**렉시컬 스코프(Lexical Scope)**란 함수를 **정의한 위치**에서 상위 스코프를 결정하는 방식이다.

자바스크립트는 렉시컬 스코프를 따른다.

```javascript
let x = 10;

function func1() {
  let x = 20;
  func2();
}

function func2() {
  console.log(x);
}

func1(); // 10
```

`func2`는 `func1` 안에서 호출되었지만, 정의된 위치는 전역 스코프이다.  
따라서 `func2`의 상위 스코프는 전역 스코프이고, 결과는 `10`이 된다.

---

## 4.4 동적 스코프

**동적 스코프(Dynamic Scope)**는 함수를 **호출한 위치**에서 상위 스코프를 결정하는 방식이다.  
자바스크립트는 동적 스코프가 아니라 렉시컬 스코프를 사용한다.

---

# 5. 화살표 함수

## 5.1 화살표 함수란?

화살표 함수는 ES6 이후 도입된 새로운 함수 선언 형태이다.  
일반 함수보다 간단하게 함수를 작성할 수 있다.

```javascript
const add = (a, b) => {
  return a + b;
};
```

---

## 5.2 일반 함수와 비교

```javascript
function add(a, b) {
  return a + b;
}

const addArrow = (a, b) => {
  return a + b;
};
```

---

## 5.3 더 간단한 표현

함수 본문이 한 줄이고 바로 값을 반환하는 경우 `return`과 중괄호를 생략할 수 있다.

```javascript
const add = (a, b) => a + b;
```

---

# 6. 옵셔널 체이닝

## 6.1 옵셔널 체이닝이 필요한 이유

객체에서 존재하지 않는 프로퍼티에 접근하거나, 배열에서 범위를 벗어난 인덱스를 조회하려고 하면 에러가 발생할 수 있다.

옵셔널 체이닝은 이러한 상황에서 에러 없이 안전하게 값을 조회할 수 있도록 도와준다.

---

## 6.2 `?.` 연산자

`?.` 연산자를 사용하면 존재하지 않는 프로퍼티나 인덱스에 접근해도 에러가 발생하지 않고 `undefined`를 반환한다.

```javascript
const user = {
  name: "Elice"
};

console.log(user.contact.email); // error
console.log(user.contact?.email); // undefined
```

---

## 6.3 사용 예시

```javascript
const user = {
  name: "Elice",
  contact: {
    email: "elice@example.com"
  }
};

console.log(user.contact?.email);
console.log(user.address?.city);
```

---

# 7. 구조 분해 할당

## 7.1 구조 분해 할당이란?

**구조 분해 할당(Destructuring Assignment)**은 객체나 배열의 값을 분해하여 변수에 쉽게 할당하는 문법이다.

---

## 7.2 객체 구조 분해 할당

ES6 이전에는 객체의 프로퍼티 값을 변수에 할당할 때 프로퍼티를 하나씩 참조해야 했다.

```javascript
const user = {
  name: "Elice",
  age: 20
};

const name = user.name;
const age = user.age;
```

ES6부터는 다음과 같이 간단하게 작성할 수 있다.

```javascript
const user = {
  name: "Elice",
  age: 20
};

const { name, age } = user;
```

---

## 7.3 변수 이름 변경

구조 분해 할당 시 변수 이름을 다른 이름으로 변경할 수도 있다.

```javascript
const user = {
  name: "Elice",
  age: 20
};

const { name: userName, age: userAge } = user;
```

---

## 7.4 기본값 설정

객체에 해당 프로퍼티가 없을 때 사용할 기본값을 설정할 수 있다.

```javascript
const user = {
  name: "Elice"
};

const { name, age = 20 } = user;
```

---

## 7.5 중첩 객체 구조 분해

객체 안의 객체도 구조 분해 할당할 수 있다.

```javascript
const user = {
  name: "Elice",
  contact: {
    email: "elice@example.com"
  }
};

const {
  contact: { email }
} = user;
```

---

## 7.6 Rest Syntax

`...`을 이용하면 할당되지 않은 나머지 프로퍼티를 객체 형태로 받을 수 있다.  
이 객체는 보통 `rest`라는 이름으로 설정한다.

```javascript
const user = {
  name: "Elice",
  age: 20,
  job: "developer"
};

const { name, ...rest } = user;

console.log(rest); // { age: 20, job: "developer" }
```

---

## 7.7 함수 인수에서 구조 분해 할당

함수의 인수로 전달되는 객체를 바로 구조 분해 할당할 수 있다.

```javascript
function printUser({ name, age }) {
  console.log(name);
  console.log(age);
}

printUser({ name: "Elice", age: 20 });
```

---

## 7.8 배열 구조 분해 할당

배열도 구조 분해 할당을 할 수 있다.  
이때 변수 이름은 원하는 대로 설정할 수 있다.

```javascript
const arr = [10, 20, 30];

const [a, b, c] = arr;
```

---

## 7.9 배열에서 Rest Syntax

배열에서도 `...`을 사용해 나머지 값을 배열 형태로 받을 수 있다.

```javascript
const arr = [10, 20, 30, 40];

const [first, second, ...rest] = arr;

console.log(rest); // [30, 40]
```

---

# 8. Spread 문법

## 8.1 Spread 문법이란?

**Spread 문법**은 배열이나 문자열처럼 순서대로 하나씩 접근할 수 있는 객체, 즉 이터러블에 해당하는 여러 값의 집합을 펼쳐서 개별 값의 목록으로 만드는 문법이다.

```javascript
const arr = [1, 2, 3];

console.log(...arr); // 1 2 3
```

---

## 8.2 배열 결합

Spread 문법은 여러 배열을 결합할 때 간편하게 사용할 수 있다.

```javascript
const arr1 = [1, 2];
const arr2 = [3, 4];

const combined = [...arr1, ...arr2];

console.log(combined); // [1, 2, 3, 4]
```

---

## 8.3 함수 인수로 전달

Spread 문법은 함수의 인수로 여러 값을 전달할 때 사용할 수 있다.

```javascript
const numbers = [1, 2, 3];

console.log(Math.max(...numbers));
```

---

## 8.4 객체에서 Spread 사용

객체도 Spread 문법을 사용할 수 있다.  
단, 객체 리터럴 내부에서만 사용할 수 있다.

```javascript
const user = {
  name: "Elice",
  age: 20
};

const newUser = {
  ...user,
  job: "developer"
};
```

---

## 8.5 Rest와 Spread의 차이

둘 다 `...`을 사용하지만 역할이 다르다.

| 구분 | 역할 | 예시 |
|---|---|---|
| Rest Syntax | 남은 값을 모은다. | `const { name, ...rest } = user;` |
| Spread Syntax | 값을 펼친다. | `const arr = [...arr1, ...arr2];` |

---

# 9. 템플릿 리터럴

## 9.1 템플릿 리터럴이란?

템플릿 리터럴은 문자열을 더 편리하게 작성하기 위한 문법이다.  
백틱 기호와 `${}`를 이용하여 문자열 내부에 변수나 표현식을 포함할 수 있다.

```javascript
const name = "Elice";

const text = `Hello ${name}`;
```

---

## 9.2 기존 문자열 연결과 비교

```javascript
const name = "Elice";
const age = 20;

const text1 = "이름은 " + name + "이고 나이는 " + age + "입니다.";
const text2 = `이름은 ${name}이고 나이는 ${age}입니다.`;
```

템플릿 리터럴을 사용하면 문자열 연결을 더 읽기 쉽게 작성할 수 있다.

---

# 10. 프로퍼티/메서드 축약

## 10.1 프로퍼티 축약

객체 리터럴에서 프로퍼티 이름과 변수 이름이 동일할 때, 프로퍼티 이름을 생략할 수 있다.

```javascript
const name = "Elice";
const age = 20;

const user = {
  name,
  age
};
```

위 코드는 다음과 같다.

```javascript
const user = {
  name: name,
  age: age
};
```

---

## 10.2 메서드 축약

객체 안의 메서드도 더 짧게 작성할 수 있다.

```javascript
const user = {
  name: "Elice",
  sayHello() {
    console.log("Hello");
  }
};
```

---

# 11. 배열 메서드

## 11.1 고차 함수

**고차 함수(Higher-Order Function)**는 함수를 인자로 받거나, 함수를 반환하는 함수이다.

```javascript
function repeat(callback) {
  callback();
}
```

---

## 11.2 콜백 함수

함수를 인자로 받는 고차 함수에서, 인자로 넘겨주는 함수를 **콜백 함수**라고 한다.

```javascript
repeat(function() {
  console.log("콜백 함수 실행");
});
```

`addEventListener`에서 등록하는 이벤트 핸들러 함수도 콜백 함수의 일종이다.

---

## 11.3 배열 메서드와 콜백 함수

배열에 내장된 배열 메서드에 콜백 함수를 전달하면 각 원소에 대해 콜백 함수를 호출한다.

대표적인 배열 메서드는 다음과 같다.

| 메서드 | 설명 |
|---|---|
| `forEach` | 각 요소에 대해 콜백 함수를 한 번씩 실행 |
| `map` | 각 요소를 변환하여 새로운 배열 반환 |
| `filter` | 조건을 만족하는 요소만 모아 새로운 배열 반환 |
| `reduce` | 각 요소를 누적 계산하여 하나의 결과 반환 |

---

## 11.4 `forEach`

`forEach`는 배열의 각 요소에 대해 콜백 함수를 한 번씩 순서대로 실행한다.  
반환 값은 없다.

```javascript
const arr = [1, 2, 3];

arr.forEach(function(item, index) {
  console.log(item, index);
});
```

| 특징 | 설명 |
|---|---|
| 실행 대상 | 배열의 각 요소 |
| 콜백 인수 | 요소 값, 인덱스 |
| 반환값 | 없음 |

---

## 11.5 `map`

`map`은 배열의 각 요소에 대해 콜백 함수를 실행하고, 그 반환 값을 모아 새로운 배열을 반환한다.  
원래 배열은 변경되지 않는다.

```javascript
const arr = [1, 2, 3];

const result = arr.map(function(item) {
  return item * 2;
});

console.log(result); // [2, 4, 6]
```

| 특징 | 설명 |
|---|---|
| 실행 대상 | 배열의 각 요소 |
| 반환값 | 새로운 배열 |
| 원본 배열 | 변경되지 않음 |
| 사용 목적 | 배열 값 변환 |

---

## 11.6 `filter`

`filter`는 배열의 각 요소에 대해 조건을 검사하고, 조건을 만족하는 요소만 모아 새로운 배열을 반환한다.  
콜백 함수는 조건의 결과를 boolean 값으로 반환해야 한다.

```javascript
const arr = [1, 2, 3, 4];

const result = arr.filter(function(item) {
  return item % 2 === 0;
});

console.log(result); // [2, 4]
```

| 특징 | 설명 |
|---|---|
| 실행 대상 | 배열의 각 요소 |
| 반환값 | 조건을 만족하는 요소들의 배열 |
| 원본 배열 | 변경되지 않음 |
| 콜백 반환값 | boolean |

---

## 11.7 `reduce`

`reduce`는 배열의 각 요소에 대해 콜백 함수를 호출하여 누적된 결과를 반환한다.  
두 번째 인수로 초깃값을 지정할 수 있다.

```javascript
const arr = [1, 2, 3, 4];

const sum = arr.reduce(function(acc, cur) {
  return acc + cur;
}, 0);

console.log(sum); // 10
```

| 특징 | 설명 |
|---|---|
| 실행 대상 | 배열의 각 요소 |
| 반환값 | 누적 결과 |
| 콜백 인수 | 이전까지의 누적값, 현재 요소 |
| 두 번째 인수 | 초깃값 |

---

# 12. 내장 객체

## 12.1 내장 객체란?

자바스크립트는 여러 용도에 활용할 수 있는 객체를 내장하고 있다.  
숫자, 문자, 날짜, JSON 데이터 등을 다룰 때 유용한 객체를 제공한다.

---

## 12.2 주요 내장 객체

| 객체 | 설명 |
|---|---|
| `window` | 브라우저 창을 나타내는 객체 |
| `document` | 브라우저에 로드된 웹 페이지 객체 |
| `Number` | 숫자 자료형을 다루기 위한 객체 |
| `Math` | 수학 연산과 상수를 제공하는 객체 |
| `Date` | 날짜와 시간을 다루기 위한 객체 |
| `String` | 문자열을 조작하기 위한 객체 |
| `JSON` | 객체를 직렬화/역직렬화하기 위한 객체 |

---

## 12.3 `window`

`window`는 DOM document를 포함하는 브라우저 창을 나타내는 객체이다.  
현재 창의 정보를 얻거나 창을 조작할 수 있다.

브라우저 환경에서는 `window` 객체가 전역 객체이다.

---

## 12.4 `document`

`document`는 브라우저에 로드된 웹 페이지에 대한 객체이다.  
문서의 title, URL 등의 정보를 얻을 수 있고, 요소 생성과 검색 등의 기능을 제공한다.

---

## 12.5 `Number`

`Number`는 자바스크립트의 number 자료형을 감싸는 객체이다.  
유의미한 상수값과 숫자를 변환하는 메서드 등을 제공한다.

```javascript
const num = 3.14159;

console.log(num.toFixed(2)); // "3.14"
```

`toFixed()`는 숫자의 소수점 자릿수를 제어하며, 반환값은 문자열이다.

---

## 12.6 `Math`

`Math`는 기본적인 수학 연산 메서드와 상수를 다루는 객체이다.

```javascript
Math.max(1, 2, 3); // 3
Math.min(1, 2, 3); // 1
```

배열과 함께 사용할 때는 Spread 문법을 활용할 수 있다.

```javascript
const arr = [1, 2, 3];

Math.max(...arr); // 3
```

---

## 12.7 `Math.random()`과 `Math.floor()`

`Math.random()`은 0 이상 1 미만의 난수를 반환한다.  
`Math.floor()`는 소수점 이하 숫자를 버린다.

```javascript
const random = Math.random();
const number = Math.floor(random * 10);
```

---

## 12.8 `Date`

`Date`는 특정 시점의 날짜를 표시하기 위한 객체이다.  
날짜와 관련된 작업을 하기 위한 여러 메서드를 포함한다.

```javascript
const today = new Date();
```

---

## 12.9 `Date` 주요 메서드

| 메서드 | 설명 |
|---|---|
| `getDay()` | 요일을 0부터 6까지 반환. 0은 일요일, 6은 토요일 |
| `setDate()` | 날짜를 설정 |
| `toDateString()` | 특정 포맷의 날짜 문자열을 반환 |
| `getTime()` | 1970년 1월 1일부터 흐른 시간을 밀리초 단위로 반환 |

```javascript
const date = new Date();

console.log(date.getDay());
console.log(date.getTime());
```

---

## 12.10 `String`

`String`은 문자열을 조작하기 위한 여러 메서드를 포함한다.

```javascript
const text = "Hello Elice";

console.log(text.length);
console.log(text.toUpperCase());
```

---

## 12.11 `JSON`

`JSON`은 자바스크립트 객체를 직렬화하거나 역직렬화할 때 사용되는 객체이다.  
데이터 통신 과정에서 자주 사용된다.

| 개념 | 설명 |
|---|---|
| 직렬화 | 객체 데이터를 다른 시스템에서도 사용할 수 있도록 연속적인 데이터 형태로 변환 |
| 역직렬화 | 직렬화된 데이터를 다시 원래 객체 형태로 되돌리는 것 |

```javascript
const user = {
  name: "Elice",
  age: 20
};

const json = JSON.stringify(user);
const parsed = JSON.parse(json);
```

---

# 13. 타이머 함수

## 13.1 타이머 함수란?

자바스크립트에는 특수한 역할을 하는 타이머 함수가 있다.

| 함수 | 설명 |
|---|---|
| `setTimeout` | 특정 시간 후에 콜백 함수 실행 |
| `setInterval` | 특정 시간마다 콜백 함수 반복 실행 |

일반 함수는 호출하면 즉시 실행되지만, 타이머 함수를 사용하면 함수 호출을 예약할 수 있다.

---

## 13.2 `setTimeout`

`setTimeout`은 특정 시간 후에 콜백 함수가 호출되도록 예약한다.  
두 번째 인수로 시간을 전달하며, 단위는 밀리초(ms)이다.

```javascript
setTimeout(function() {
  console.log("1초 후 실행");
}, 1000);
```

세 번째 이후의 인수에는 콜백 함수에 전달될 인수를 넣을 수 있다.

```javascript
setTimeout(function(name) {
  console.log(`Hello ${name}`);
}, 1000, "Elice");
```

---

## 13.3 `clearTimeout`

`setTimeout`은 숫자 형태의 timeout id를 반환한다.  
`clearTimeout`에 이 id를 전달하면 해당 timeout을 제거할 수 있다.

```javascript
const timeoutId = setTimeout(function() {
  console.log("실행되지 않음");
}, 1000);

clearTimeout(timeoutId);
```

---

## 13.4 `setInterval`

`setInterval`은 특정 시간마다 콜백 함수가 반복 호출되도록 예약한다.  
두 번째 인수로 시간을 전달하며, 단위는 밀리초(ms)이다.

```javascript
setInterval(function() {
  console.log("1초마다 실행");
}, 1000);
```

---

## 13.5 `clearInterval`

`setInterval`은 숫자 형태의 interval id를 반환한다.  
`clearInterval`에 이 id를 전달하면 해당 interval을 제거할 수 있다.

```javascript
const intervalId = setInterval(function() {
  console.log("반복 실행");
}, 1000);

clearInterval(intervalId);
```

---

# 14. 실행 컨텍스트와 콜 스택

## 14.1 자바스크립트 엔진

작성한 자바스크립트 코드는 자바스크립트 엔진을 통해 파싱되고 실행된다.  
Chrome 브라우저의 경우 V8 엔진을 사용한다.

---

## 14.2 실행 컨텍스트

자바스크립트 엔진은 코드 실행 전에 **실행 컨텍스트**를 생성한다.

실행 컨텍스트는 코드가 실행되는 환경을 정의하며, 변수, 함수 선언, this 바인딩 등 코드 실행에 필요한 정보를 관리하는 개념이다.

---

## 14.3 실행 컨텍스트의 단계

| 단계 | 설명 |
|---|---|
| 생성 단계 | 변수 선언문과 함수 선언 등을 먼저 저장 |
| 실행 단계 | 나머지 코드를 실행하고 변수 값 할당 |

---

## 14.4 실행 컨텍스트 스택

생성되는 실행 컨텍스트는 모두 **실행 컨텍스트 스택**, 즉 **콜 스택(Call Stack)**에서 관리된다.

스택은 후입선출 구조이다.

| 개념 | 설명 |
|---|---|
| Stack | 후입선출 구조 |
| LIFO | Last In First Out |
| 콜 스택 | 실행 컨텍스트를 관리하는 스택 |

---

## 14.5 싱글 스레드

자바스크립트는 오직 하나의 실행 컨텍스트 스택, 즉 콜 스택을 가진다.  
따라서 한 번에 하나의 작업만 수행할 수 있다.

이를 **싱글 스레드(Single Thread)** 기반 언어라고 한다.

---

# 15. 동기와 비동기

## 15.1 싱글 스레드의 문제점

싱글 스레드는 한 번에 하나의 작업만 수행하므로 작업 순서가 보장된다는 장점이 있다.  
하지만 앞선 작업이 종료되기 전까지 이후 작업을 시작하지 못한다는 문제가 있다.

이처럼 앞선 작업 때문에 다음 작업이 대기하는 현상을 **블로킹(Blocking)**이라고 한다.

---

## 15.2 동기 처리

**동기(Synchronous) 처리 방식**은 작업이 순차적으로 실행되며, 현재 작업이 완료될 때까지 다음 작업이 대기하는 방식이다.

```javascript
const data = fs.readFileSync("largeFile.txt", "utf8");

console.log("다음 작업");
```

파일 읽기가 끝날 때까지 다음 작업은 실행되지 않는다.

---

## 15.3 비동기 처리

**비동기(Asynchronous) 처리 방식**은 작업이 시작된 후 완료 여부와 상관없이 다음 작업을 즉시 실행하는 방식이다.  
작업 완료 시 콜백 함수나 Promise 등을 통해 결과를 처리한다.

```javascript
fs.readFile("largeFile.txt", "utf8", function(err, data) {
  console.log("파일 로드 완료");
});

console.log("다른 작업 실행");
```

---

## 15.4 동기와 비동기 비교

| 구분 | 동기 처리 | 비동기 처리 |
|---|---|---|
| 실행 방식 | 순차적으로 실행 | 작업 완료를 기다리지 않고 다음 작업 실행 |
| 대기 여부 | 현재 작업이 끝날 때까지 대기 | 대기하지 않음 |
| 문제점 | 블로킹 발생 가능 | 실행 순서를 이해하기 어려울 수 있음 |
| 결과 처리 | 바로 반환값 사용 | 콜백, Promise, async/await 사용 |

---

## 15.5 비동기로 동작하는 자바스크립트 기능

| 기능 | 예시 |
|---|---|
| 타이머 함수 | `setTimeout`, `setInterval` |
| Ajax | `XMLHttpRequest`, `Fetch API` |
| 이벤트 핸들러 | `addEventListener` |

---

# 16. 비동기 동작 원리

## 16.1 이벤트 루프와 태스크 큐

브라우저는 비동기 처리를 위해 **이벤트 루프(Event Loop)**와 **태스크 큐(Task Queue)**를 사용한다.

태스크 큐는 Event Queue, Callback Queue라고도 한다.

| 개념 | 설명 |
|---|---|
| Call Stack | 현재 실행 중인 작업이 쌓이는 공간 |
| Task Queue | 비동기 함수의 콜백 함수가 잠시 저장되는 큐 영역 |
| Event Loop | 콜 스택이 비었을 때 태스크 큐의 작업을 콜 스택으로 이동 |
| Queue | FIFO 방식의 자료구조 |

---

## 16.2 큐 구조

큐는 먼저 들어온 데이터가 먼저 나가는 구조이다.

| 자료구조 | 방식 |
|---|---|
| Queue | FIFO, First In First Out |
| Stack | LIFO, Last In First Out |

---

## 16.3 비동기 실행 순서 예시

```javascript
console.log("1. 스크립트 시작");

setTimeout(function() {
  console.log("3. setTimeout");
}, 0);

console.log("2. 스크립트 끝");
```

실행 결과는 다음과 같다.

```text
1. 스크립트 시작
2. 스크립트 끝
3. setTimeout
```

`setTimeout`의 시간이 0ms여도 콜백 함수는 바로 실행되지 않는다.  
콜백 함수는 태스크 큐에 들어가고, 콜 스택이 비어야 이벤트 루프에 의해 콜 스택으로 이동한다.

---

# 17. Promise

## 17.1 Promise가 필요한 이유

기본적인 비동기 처리는 처리 결과를 바로 활용하기 어렵다.  
또한 에러 처리도 복잡해질 수 있다.

`Promise`를 사용하면 비동기 처리의 성공과 실패를 명확하게 다룰 수 있다.

---

## 17.2 Promise 생성

Promise 객체는 `new Promise()`를 사용해 생성한다.  
이때 콜백 함수가 반드시 인수로 전달되어야 한다.

```javascript
const promise = new Promise(function(resolve, reject) {
  // 비동기 작업
});
```

---

## 17.3 `resolve`와 `reject`

Promise의 콜백 함수에는 `resolve`와 `reject` 함수가 인수로 전달된다.

| 함수 | 의미 |
|---|---|
| `resolve` | 비동기 처리 성공 |
| `reject` | 비동기 처리 실패 |

```javascript
const promise = new Promise(function(resolve, reject) {
  const success = true;

  if (success) {
    resolve("성공");
  } else {
    reject("실패");
  }
});
```

---

## 17.4 Promise 상태

Promise는 처리 상태에 따라 세 가지 중 하나의 상태를 가진다.

| 상태 | 설명 |
|---|---|
| `pending` | 비동기 처리가 완료되기 전, 대기 상태 |
| `fulfilled` | 비동기 처리가 완료되었고 성공한 상태 |
| `rejected` | 비동기 처리가 완료되었고 실패한 상태 |

---

## 17.5 상태 변화

| 실행 | 상태 변화 |
|---|---|
| Promise 생성 직후 | `pending` |
| `resolve()` 실행 | `pending` → `fulfilled` |
| `reject()` 실행 | `pending` → `rejected` |

---

# 18. Promise 처리

## 18.1 `then`, `catch`, `finally`

Promise 객체는 비동기 처리 완료 이후를 다루기 위해 세 가지 주요 메서드를 제공한다.

| 메서드 | 설명 |
|---|---|
| `.then()` | 비동기 성공 및 실패 후 처리 |
| `.catch()` | 비동기 실패 후 처리 |
| `.finally()` | 성공/실패 여부와 관계없이 이후 처리 |

각 메서드는 모두 Promise 객체를 반환하므로 체이닝이 가능하다.

---

## 18.2 `then`

`then` 메서드에는 두 가지 콜백 함수를 전달할 수 있다.

| 인수 | 설명 |
|---|---|
| 첫 번째 콜백 | 비동기 성공 시 실행. `resolve()`로 넘긴 값을 받음 |
| 두 번째 콜백 | 비동기 실패 시 실행. `reject()`로 넘긴 값을 받음 |

```javascript
promise.then(
  function(result) {
    console.log(result);
  },
  function(error) {
    console.log(error);
  }
);
```

---

## 18.3 `catch`

`catch` 메서드에는 비동기 실패 시 실행될 콜백 함수가 전달된다.  
`reject()`로 넘긴 값을 인수로 받는다.

```javascript
promise.catch(function(error) {
  console.log(error);
});
```

---

## 18.4 `finally`

`finally` 메서드는 비동기 성공/실패 여부와 관계없이 실행된다.

```javascript
promise.finally(function() {
  console.log("작업 종료");
});
```

---

## 18.5 Promise Chaining

Promise의 `then`, `catch`, `finally`는 Promise를 반환하므로 메서드 체이닝을 통해 이어서 사용할 수 있다.

```javascript
promise
  .then(function(result) {
    console.log(result);
  })
  .catch(function(error) {
    console.log(error);
  })
  .finally(function() {
    console.log("종료");
  });
```

---

# 19. Promise 메서드

## 19.1 `Promise.resolve`

`Promise.resolve()`는 인수로 전달받은 값을 resolve하는 Promise를 반환한다.

```javascript
const promise = Promise.resolve("성공");
```

---

## 19.2 `Promise.reject`

`Promise.reject()`는 인수로 전달받은 값을 reject하는 Promise를 반환한다.

```javascript
const promise = Promise.reject("실패");
```

---

## 19.3 `Promise.all`

`Promise.all()`은 여러 Promise를 병렬적으로 처리할 때 사용한다.  
인수로 여러 Promise 객체가 담긴 배열을 전달한다.

```javascript
Promise.all([promise1, promise2, promise3])
  .then(function(results) {
    console.log(results);
  });
```

모든 Promise가 resolve된 후에 `then`으로 넘어가며, 각 Promise의 결과는 배열 형태로 전달된다.

---

## 19.4 Promise.all의 Fail-Fast

`Promise.all()`은 인수로 전달된 Promise 중 하나라도 reject되면, 나머지 결과를 기다리지 않고 즉시 전체가 reject된다.

이를 **Fail-Fast**라고 한다.

```javascript
Promise.all([promise1, promise2, promise3])
  .catch(function(error) {
    console.log(error);
  });
```

`catch`에는 가장 먼저 실패한 Promise의 reject 값이 전달된다.

---

## 19.5 `Promise.allSettled`

`Promise.allSettled()`는 성공/실패 여부와 관계없이 모든 Promise가 완료될 때까지 기다린다.

```javascript
Promise.allSettled([promise1, promise2, promise3])
  .then(function(results) {
    console.log(results);
  });
```

각 결과는 다음 두 형태 중 하나로 반환된다.

```javascript
{ status: "fulfilled", value: ... }
{ status: "rejected", reason: ... }
```

---

## 19.6 `Promise.all` vs `Promise.allSettled`

| 구분 | `Promise.all` | `Promise.allSettled` |
|---|---|---|
| 실패 시 동작 | 하나라도 실패하면 즉시 전체 reject | 성공/실패와 관계없이 끝까지 기다림 |
| 반환값 | 성공 값들의 배열 | status 객체들의 배열 |
| 사용 시점 | 모두 성공해야 의미 있을 때 | 일부 실패를 허용할 때 |
| 동작 방식 | Fail-Fast | Wait-All |

---

# 20. 마이크로태스크 큐

## 20.1 마이크로태스크 큐란?

브라우저에는 태스크 큐 외에 **마이크로태스크 큐(Microtask Queue)**라는 공간이 있다.  
마이크로태스크 큐는 Job Queue라고도 한다.

마이크로태스크 큐는 태스크 큐보다 우선순위가 높다.

---

## 20.2 실행 우선순위

콜 스택이 비면 이벤트 루프는 먼저 마이크로태스크 큐의 작업을 콜 스택으로 이동시킨다.  
마이크로태스크 큐가 비면 그 다음 태스크 큐의 작업이 콜 스택으로 들어간다.

| 큐 | 저장되는 비동기 작업 |
|---|---|
| 마이크로태스크 큐 | Promise, Ajax |
| 태스크 큐 | 타이머 함수, 이벤트 핸들러 함수 |

---

## 20.3 Promise와 setTimeout의 실행 순서

```javascript
console.log(1);

setTimeout(function() {
  console.log(4);
}, 0);

Promise.resolve().then(function() {
  console.log(3);
});

console.log(2);
```

실행 결과는 다음과 같다.

```text
1
2
3
4
```

Promise는 마이크로태스크 큐에 저장되므로, 태스크 큐에 저장되는 `setTimeout` 콜백보다 먼저 실행된다.

---

# 21. async/await

## 21.1 Promise의 단점

Promise를 이용해 비동기를 처리할 수 있지만, 체이닝이 계속되면 코드의 가독성이 떨어질 수 있다.

```javascript
promise
  .then(...)
  .then(...)
  .then(...)
  .catch(...);
```

---

## 21.2 async/await의 등장

ES8부터 도입된 `async/await` 문법을 사용하면 비동기 처리를 훨씬 깔끔하게 작성할 수 있다.

`async/await`을 이용하면 비동기 처리를 동기 처리하듯이 작성할 수 있다.

---

## 21.3 `await`

`await` 키워드를 사용하면 해당 비동기 처리가 완료될 때까지 기다린 후, resolve된 값을 반환한다.

```javascript
const result = await promise;
```

단, `await`은 반드시 `async` 함수 내부에서 사용해야 한다.

---

## 21.4 `async`

`async` 함수는 항상 반환값을 resolve하는 Promise를 반환한다.

```javascript
async function getData() {
  return "데이터";
}

getData().then(function(result) {
  console.log(result);
});
```

---

## 21.5 async/await 예시

```javascript
function fetchData() {
  return new Promise(function(resolve) {
    setTimeout(function() {
      resolve("데이터 로드 완료");
    }, 1000);
  });
}

async function main() {
  const result = await fetchData();
  console.log(result);
}

main();
```

---

# 22. 예외 처리

## 22.1 예외 처리의 필요성

모든 프로그램은 에러가 발생할 가능성이 있다.  
에러가 발생하면 프로그램이 종료될 수 있으므로, 종료를 방지하기 위해 예외 상황에 대응하는 코드를 작성해야 한다.

---

## 22.2 try ... catch

기본적으로 `try ... catch` 문을 이용해 예외를 처리할 수 있다.  
`try`만 단독으로 사용할 수는 없다.

```javascript
try {
  // 에러가 발생할 수 있는 코드
} catch (error) {
  // 에러 처리 코드
}
```

---

## 22.3 try ... catch ... finally

`finally` 문을 추가하면 에러 여부와 상관없이 실행되는 코드를 작성할 수 있다.

```javascript
try {
  // 에러가 발생할 수 있는 코드
} catch (error) {
  console.log(error);
} finally {
  console.log("항상 실행");
}
```

---

## 22.4 async/await에서의 예외 처리

`async/await`을 사용할 때는 `try ... catch ... finally` 문을 사용하여 예외 처리를 진행한다.

```javascript
async function main() {
  try {
    const result = await fetchData();
    console.log(result);
  } catch (error) {
    console.log("에러 발생:", error);
  } finally {
    console.log("작업 종료");
  }
}
```

---

# 23. 전체 핵심 정리

## 23.1 ES6 문법 핵심

| 개념 | 핵심 |
|---|---|
| `let` | 값 변경 가능한 변수 선언 |
| `const` | 값 변경 불가능한 상수 선언 |
| 화살표 함수 | 함수를 간결하게 표현 |
| 옵셔널 체이닝 | 존재하지 않는 프로퍼티에 안전하게 접근 |
| 구조 분해 할당 | 객체/배열 값을 변수에 쉽게 할당 |
| Spread 문법 | 배열/객체 값을 펼쳐서 사용 |
| 템플릿 리터럴 | 문자열 안에 변수와 표현식을 쉽게 삽입 |
| 프로퍼티 축약 | 객체 작성 시 중복되는 이름 생략 |

---

## 23.2 배열 메서드 핵심

| 메서드 | 핵심 기능 | 반환값 |
|---|---|---|
| `forEach` | 각 요소를 순회하며 실행 | 없음 |
| `map` | 각 요소를 변환 | 새로운 배열 |
| `filter` | 조건에 맞는 요소만 추출 | 새로운 배열 |
| `reduce` | 요소를 누적 계산 | 누적 결과 |

---

## 23.3 비동기 핵심

| 개념 | 설명 |
|---|---|
| 동기 | 현재 작업이 끝날 때까지 다음 작업 대기 |
| 비동기 | 작업 완료를 기다리지 않고 다음 작업 실행 |
| 콜 스택 | 실행 컨텍스트를 관리하는 공간 |
| 태스크 큐 | 타이머, 이벤트 핸들러 콜백 등이 대기하는 공간 |
| 이벤트 루프 | 콜 스택이 비면 큐의 작업을 콜 스택으로 이동 |
| 마이크로태스크 큐 | Promise 등이 대기하는 우선순위 높은 큐 |

---

## 23.4 Promise 핵심

| 개념 | 설명 |
|---|---|
| `Promise` | 비동기 처리의 성공/실패를 다루는 객체 |
| `pending` | 대기 상태 |
| `fulfilled` | 성공 상태 |
| `rejected` | 실패 상태 |
| `resolve` | 성공 처리 |
| `reject` | 실패 처리 |
| `.then()` | 성공/실패 후 처리 |
| `.catch()` | 실패 후 처리 |
| `.finally()` | 성공/실패 관계없이 실행 |
| `Promise.all()` | 모두 성공해야 성공 |
| `Promise.allSettled()` | 모두 끝날 때까지 대기 |

---

## 23.5 async/await 핵심

| 개념 | 설명 |
|---|---|
| `async` | 항상 Promise를 반환하는 함수 |
| `await` | Promise가 완료될 때까지 기다리고 resolve 값을 반환 |
| `try ... catch` | async/await에서 예외 처리 |
| `finally` | 성공/실패와 관계없이 실행 |

---

# 24. 최종 요약

ES6는 자바스크립트 문법을 더 안전하고 편리하게 만든 중요한 업데이트이다.  
`let`과 `const`는 기존의 `var`보다 변수 중복 선언과 의도치 않은 변경을 줄일 수 있으며, 블록 레벨 스코프를 지원한다.  
화살표 함수, 구조 분해 할당, Spread 문법, 템플릿 리터럴, 옵셔널 체이닝은 코드를 더 간결하고 읽기 쉽게 만든다.

배열 메서드인 `forEach`, `map`, `filter`, `reduce`는 콜백 함수를 활용하여 배열 데이터를 효율적으로 처리한다.  
또한 자바스크립트는 `window`, `document`, `Number`, `Math`, `Date`, `String`, `JSON`과 같은 내장 객체를 제공하여 브라우저, 숫자, 날짜, 문자열, 데이터 변환 작업을 쉽게 수행할 수 있게 한다.

비동기 처리는 싱글 스레드 기반의 자바스크립트에서 블로킹 문제를 해결하기 위한 핵심 개념이다.  
브라우저는 콜 스택, 태스크 큐, 마이크로태스크 큐, 이벤트 루프를 이용해 비동기 작업을 처리한다.  
Promise는 비동기 작업의 성공과 실패를 명확하게 관리하며, `async/await`은 Promise 기반 비동기 코드를 동기 코드처럼 읽기 쉽게 작성할 수 있도록 도와준다.

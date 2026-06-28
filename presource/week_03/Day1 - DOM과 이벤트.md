# Day 1. DOM과 이벤트 정리

> 원본 PDF: `Day 1. DOM과 이벤트.pdf`  
> 주제: DOM 개념, HTML 요소 선택·조작·생성, 이벤트, 이벤트 객체, 이벤트 전파

---

# 1. DOM이란?

## 1.1 DOM의 의미

DOM은 **Document Object Model**의 약자로, 한국어로는 **문서 객체 모델**이라고 한다.

DOM은 HTML이나 XML처럼 구조화된 문서를 자바스크립트가 다룰 수 있도록 **객체 형태로 표현한 방식**이다.

브라우저는 HTML 코드를 단순한 문자열로만 보는 것이 아니라, 각 태그와 내용을 객체로 변환한다.  
이렇게 변환된 객체 구조를 통해 자바스크립트는 HTML 요소를 선택하고, 내용을 바꾸고, 스타일을 변경하고, 이벤트를 연결할 수 있다.

---

## 1.2 DOM의 핵심 개념

| 개념 | 설명 |
|---|---|
| DOM | HTML 문서를 객체 형태로 표현한 구조 |
| Document Object Model | 문서 객체 모델 |
| 구조화된 문서 | HTML, XML처럼 계층 구조를 가진 문서 |
| 객체 형태 표현 | 자바스크립트가 문서를 다룰 수 있도록 각 요소를 객체로 변환하는 것 |

---

# 2. DOM의 트리 구조

## 2.1 DOM 트리

HTML 문서는 계층 구조를 가진다.  
예를 들어 `<html>` 안에 `<head>`와 `<body>`가 있고, `<body>` 안에는 다시 `<article>`, `<section>`, `<header>` 등이 들어갈 수 있다.

브라우저는 HTML 코드를 해석하여 이러한 관계를 **트리 구조**로 만든다.  
이때 각각의 태그는 자바스크립트의 **node 객체**로 변환된다.

---

## 2.2 HTML 코드와 DOM 트리 예시

```html
<html>
  <head>
    <title>자바스크립트 기초</title>
  </head>
  <body>
    <article>
      <header>header</header>
      <section>
        <header>header 1</header>
        section 1
      </section>
    </article>
  </body>
</html>
```

위 HTML은 다음과 같은 DOM 트리 구조로 이해할 수 있다.

```text
html
├── head
│   └── title
│       └── "자바스크립트 기초"
└── body
    └── article
        ├── header
        │   └── "header"
        └── section
            ├── header
            │   └── "header 1"
            └── "section 1"
```

---

## 2.3 DOM 트리 핵심 정리

| 항목 | 설명 |
|---|---|
| DOM 트리 | HTML 요소들의 관계를 트리 형태로 나타낸 구조 |
| 부모 노드 | 다른 노드를 포함하는 상위 노드 |
| 자식 노드 | 부모 노드 안에 포함된 하위 노드 |
| 루트 노드 | 트리 구조의 가장 위에 있는 노드 |
| node 객체 | DOM에서 정보를 저장하는 계층적 단위 |

---

# 3. node 객체

## 3.1 node란?

**노드(node)**는 HTML DOM에서 정보를 저장하는 계층적 단위이다.

DOM은 여러 종류의 노드로 구성되지만, 개발에서 주로 사용하는 핵심 객체는 다음과 같다.

| 핵심 node 객체 | 설명 |
|---|---|
| `document` | 웹 페이지 전체를 의미 |
| `element` | 개별 HTML 요소를 의미 |
| `attribute` | HTML 요소의 속성을 의미 |
| `text` | HTML 요소 안의 텍스트를 의미 |

---

## 3.2 노드 트리

노드들의 집합을 **노드 트리**라고 한다.  
노드 트리는 노드 간의 관계를 트리 구조로 나타낸 것이다.

예를 들어 `<body>` 안에 `<article>`이 있다면, `<body>`는 부모 노드이고 `<article>`은 자식 노드이다.

---

# 4. document 객체와 element 객체

## 4.1 document 객체

`document` 객체는 **웹 페이지 전체**를 의미한다.

자바스크립트에서 DOM에 접근하고자 할 때에는 `document` 객체를 사용해야 한다.  
즉, `document`는 자바스크립트가 HTML 문서에 접근하기 위한 출발점이다.

```javascript
document.querySelector("header");
document.getElementById("box");
```

---

## 4.2 element 객체

`element` 객체는 **개별 HTML 요소**를 의미한다.  
예를 들어 `<div>`, `<p>`, `<button>`, `<input>` 같은 태그 하나하나가 element 객체가 될 수 있다.

element 객체에는 해당 요소를 조작할 수 있는 다양한 프로퍼티와 메서드가 존재한다.

```javascript
const title = document.querySelector("#title");

title.innerText = "Hello";
title.style.color = "red";
```

---

## 4.3 document와 element 객체의 역할

| 역할 | 설명 |
|---|---|
| HTML 요소 선택 | 특정 요소를 찾아 가져올 수 있다. |
| HTML 요소 조작 | 텍스트, HTML, 스타일, 속성 등을 변경할 수 있다. |
| HTML 요소 생성 및 추가 | 새로운 요소를 만들고 화면에 추가할 수 있다. |
| 이벤트 핸들러 추가 | 클릭, 입력, 제출 등의 이벤트에 반응하도록 만들 수 있다. |

---

# 5. HTML 요소의 선택

## 5.1 요소 선택의 의미

HTML 요소를 조작하려면 먼저 원하는 요소를 선택해야 한다.  
자바스크립트에서는 `document` 객체의 다양한 메서드를 이용해 HTML 요소를 선택할 수 있다.

---

## 5.2 요소 선택 메서드

| 메서드 사용 예시 | 설명 |
|---|---|
| `document.getElementById("box")` | 해당 id를 가진 요소 하나를 선택 |
| `document.getElementsByClassName("list-item")` | 해당 class에 속한 요소를 모두 선택 |
| `document.getElementsByName("email")` | 해당 name 속성값을 가진 요소를 모두 선택 |
| `document.querySelectorAll("ul > li")` | 해당 CSS 선택자로 선택되는 요소를 모두 선택 |
| `document.querySelector("header > #title")` | 해당 CSS 선택자로 선택되는 요소 중 첫 번째 요소 선택 |

---

## 5.3 하나 선택과 모두 선택

| 구분 | 메서드 | 반환 대상 |
|---|---|---|
| 하나 선택 | `getElementById()` | 특정 id의 요소 하나 |
| 하나 선택 | `querySelector()` | CSS 선택자에 맞는 첫 번째 요소 |
| 모두 선택 | `getElementsByTagName()` | 특정 태그의 모든 요소 |
| 모두 선택 | `getElementsByClassName()` | 특정 class의 모든 요소 |
| 모두 선택 | `getElementsByName()` | 특정 name 속성값의 모든 요소 |
| 모두 선택 | `querySelectorAll()` | CSS 선택자에 맞는 모든 요소 |

---

## 5.4 요소 선택 예시

```javascript
// HTML <li> 요소를 모두 선택
var selectedItem = document.getElementsByTagName("li");

// 아이디가 id인 요소 선택
var selectedItem = document.getElementById("id");

// odd 클래스를 가진 모든 요소 선택
var selectedItem = document.getElementsByClassName("odd");

// name 속성 값이 first인 모든 요소 선택
var selectedItem = document.getElementsByName("first");
```

---

# 6. HTML 요소의 조작

## 6.1 요소 조작의 의미

HTML 요소를 선택한 뒤에는 해당 요소의 스타일, 내용, 속성, class 등을 변경할 수 있다.

---

## 6.2 요소 조작 방법

| 사용 예시 | 설명 |
|---|---|
| `요소.style.스타일속성 = "200px"` | 요소의 스타일 조회 및 변경 |
| `요소.innerHTML = "<li><span>hello</span></li>"` | 요소 하위의 HTML 조회 및 변경 |
| `요소.innerText = "Hello Elice"` | 요소 하위의 텍스트 조회 및 변경 |
| `요소.getAttribute("placeholder")` | 요소의 HTML 속성 조회 |
| `요소.setAttribute("min", "20")` | 요소의 HTML 속성 변경 |
| `요소.classList` | 요소의 class 수정 |

---

## 6.3 스타일 변경

선택한 요소의 CSS 스타일을 자바스크립트로 변경할 수 있다.

```javascript
// 클래스가 even인 요소를 선택
var selectedItem = document.querySelector(".even");

// 선택된 요소의 텍스트 색상을 변경
selectedItem.style.color = "red";
```

이 코드는 `.even` 클래스를 가진 첫 번째 요소를 선택한 뒤, 글자 색상을 빨간색으로 바꾸는 코드이다.

---

## 6.4 innerHTML

`innerHTML`은 선택한 요소 안의 HTML 구조를 조회하거나 변경할 때 사용한다.

```javascript
// 아이디가 text인 요소를 선택
var str = document.getElementById("text");

// 선택된 요소의 하위 HTML을 변경
str.innerHTML = "<span>안녕하세요</span>";
```

`innerHTML`은 단순 텍스트뿐만 아니라 HTML 태그까지 삽입할 수 있다.  
따라서 `<span>`, `<strong>`, `<li>` 같은 태그 구조를 함께 넣을 때 사용한다.

---

## 6.5 innerText

`innerText`는 요소 안의 텍스트만 조회하거나 변경할 때 사용한다.

```javascript
const title = document.querySelector("#title");

title.innerText = "Hello Elice";
```

`innerHTML`과 달리 태그를 넣어도 HTML로 해석하지 않고 텍스트로 처리한다.

---

## 6.6 속성 조회 및 변경

HTML 요소의 속성은 `getAttribute()`와 `setAttribute()`를 사용해 조회하거나 변경할 수 있다.

```javascript
const input = document.getElementById("idInput");

// 속성 조회
input.getAttribute("placeholder");

// 속성 변경
input.setAttribute("placeholder", "아이디를 입력하세요");
```

---

# 7. HTML 요소의 생성 및 추가

## 7.1 요소 생성 및 추가 메서드

| 메서드 사용 예시 | 설명 |
|---|---|
| `document.createElement("form")` | 해당 태그의 HTML 요소를 생성 |
| `document.write("Hello World")` | 텍스트를 화면에 출력 |
| `요소.appendChild(node)` | 해당 요소 내부 가장 마지막에 node 추가 |

---

## 7.2 createElement

`document.createElement()`는 새로운 HTML 요소를 생성할 때 사용한다.

```javascript
const newDiv = document.createElement("div");
```

하지만 `createElement()`로 생성된 노드는 처음에는 자바스크립트 상에서만 존재한다.  
즉, 아직 화면에는 보이지 않는다.

---

## 7.3 appendChild

생성한 요소를 실제 화면에 보이게 하려면 DOM에 추가해야 한다.  
이때 `appendChild()`를 사용할 수 있다.

```javascript
const newDiv = document.createElement("div");
newDiv.innerText = "새로운 요소입니다.";

document.body.appendChild(newDiv);
```

위 코드는 새로운 `<div>` 요소를 만들고, 그 안에 텍스트를 넣은 뒤, `<body>`의 마지막 자식으로 추가한다.

---

# 8. 이벤트

## 8.1 이벤트란?

이벤트는 **웹 브라우저가 알려주는 HTML 요소에 대한 사건의 발생**을 의미한다.

예를 들어 사용자가 버튼을 클릭하거나, 키보드를 누르거나, 입력창에 값을 입력하거나, 폼을 제출하는 행동 등이 이벤트에 해당한다.

자바스크립트는 발생한 이벤트에 반응하여 특정 동작을 수행할 수 있다.

---

## 8.2 이벤트 종류

| 이벤트 종류 | 설명 |
|---|---|
| 마우스 이벤트 | 마우스 조작과 관련된 이벤트 |
| 키 이벤트 | 키보드 조작과 관련된 이벤트 |
| 폼 이벤트 | `form` 태그 관련 동작에 대한 이벤트 |
| 로드 관련 이벤트 | 페이지나 리소스가 로드될 때 발생하는 이벤트 |
| 창 관련 이벤트 | 브라우저 창 크기 변경 등과 관련된 이벤트 |

---

# 9. 이벤트 핸들러

## 9.1 이벤트 핸들러란?

이벤트 핸들러는 **이벤트가 발생했을 때 처리를 담당하는 함수**이다.

지정된 이벤트가 발생하면, 그 요소에 등록된 이벤트 핸들러가 실행된다.

```javascript
button.onclick = function() {
  console.log("버튼 클릭");
};
```

---

## 9.2 이벤트 핸들러 등록 방법

이벤트 핸들러를 등록하는 방법은 크게 세 가지가 있다.

| 방법 | 설명 | 권장 여부 |
|---|---|---|
| HTML 속성에 등록 | HTML 요소의 `on이벤트명` 속성에 직접 동작을 추가 | 권장되지 않음 |
| node 객체 프로퍼티에 등록 | node 객체의 `on이벤트명` 프로퍼티에 함수 등록 | 사용 가능 |
| `addEventListener()` 사용 | 이벤트명과 핸들러 함수를 인수로 전달 | 권장 |

---

## 9.3 HTML 속성 방식

HTML 요소의 `on이벤트명` 속성에 자바스크립트 코드를 직접 넣는 방식이다.

```html
<button onclick="handler()">클릭</button>

<script>
function handler() {
  console.log("클릭됨");
}
</script>
```

이 방식은 HTML 코드 안에 JS 코드가 포함되기 때문에 권장되지 않는다.

---

## 9.4 node 객체 프로퍼티 방식

node 객체의 `on이벤트명` 프로퍼티에 핸들러 함수를 등록하는 방식이다.

```javascript
const button = document.querySelector("button");

button.onclick = function() {
  console.log("버튼 클릭");
};
```

---

## 9.5 addEventListener 방식

`addEventListener()`는 이벤트 핸들러를 등록할 때 가장 많이 사용하는 방식이다.

첫 번째 인수에는 이벤트명을 전달한다.  
이때 `onclick`처럼 `on`을 붙이지 않고, `"click"`처럼 이벤트 이름만 작성한다.

두 번째 인수에는 이벤트가 발생했을 때 실행할 핸들러 함수를 전달한다.

```javascript
const button = document.querySelector("button");

button.addEventListener("click", function() {
  console.log("버튼 클릭");
});
```

---

# 10. 이벤트 객체

## 10.1 event 객체란?

`event` 객체는 이벤트 핸들러 함수에 첫 번째 인수로 전달되는 객체이다.

이 객체 안에는 해당 이벤트에 대한 정보와 기능이 담겨 있다.

```javascript
button.addEventListener("click", function(event) {
  console.log(event);
});
```

---

## 10.2 마우스 정보

마우스 관련 이벤트의 경우 `event` 객체를 통해 마우스에 대한 정보를 얻을 수 있다.

예를 들어 마우스 위치, 클릭 정보 등을 확인할 수 있다.

```javascript
document.addEventListener("click", function(event) {
  console.log(event.clientX);
  console.log(event.clientY);
});
```

---

## 10.3 키보드 정보

키보드 관련 이벤트의 경우 `event` 객체를 통해 누른 키에 대한 정보를 얻을 수 있다.

```javascript
document.addEventListener("keydown", function(event) {
  console.log(event.key);
});
```

---

## 10.4 event.target

`event.target` 프로퍼티를 사용하면 이벤트가 실제로 발생한 대상 요소의 node 객체에 접근할 수 있다.

```javascript
document.addEventListener("click", function(event) {
  console.log(event.target);
});
```

예를 들어 버튼을 클릭하면 `event.target`은 클릭된 버튼 요소를 가리킨다.

---

## 10.5 preventDefault

`form` 태그에서 submit 버튼의 기본 동작은 폼 제출이다.  
이 과정에서 페이지가 새로고침될 수 있다.

이 기본 동작을 막기 위해 `event.preventDefault()` 메서드를 사용한다.

```javascript
const form = document.querySelector("form");

form.addEventListener("submit", function(event) {
  event.preventDefault();
  console.log("폼 제출 기본 동작 차단");
});
```

---

# 11. 이벤트 핸들러 제거

## 11.1 removeEventListener

`removeEventListener()` 메서드를 사용하면 등록된 이벤트 핸들러를 제거할 수 있다.

```javascript
function handler() {
  console.log("클릭됨");
}

button.addEventListener("click", handler);
button.removeEventListener("click", handler);
```

주의할 점은 이벤트를 제거하려면 등록할 때 사용한 함수와 제거할 때 사용하는 함수가 같아야 한다는 것이다.  
따라서 익명 함수보다는 이름이 있는 함수를 사용하는 것이 이벤트 제거에 유리하다.

---

# 12. 이벤트 propagation

## 12.1 이벤트 propagation이란?

이벤트가 발생하면, 그 이벤트는 DOM 트리를 따라 전파된다.  
이를 **이벤트 propagation**, 즉 **이벤트 전파**라고 한다.

예를 들어 다음과 같은 구조가 있다고 하자.

```html
<html>
  <body>
    <div id="parent1">
      <div id="parent2">
        <div id="parent3">
          <button id="btn">클릭</button>
        </div>
      </div>
    </div>
  </body>
</html>
```

버튼을 클릭하면 이벤트는 버튼에서만 발생하는 것이 아니라, 버튼의 부모 요소들에서도 감지될 수 있다.

---

## 12.2 이벤트 전파 대상

버튼 클릭 이벤트는 다음 요소들에서 모두 감지될 수 있다.

```text
window
document
html
body
div#parent1
div#parent2
div#parent3
button#btn
```

정리하면, 가장 하위 요소인 버튼을 클릭하더라도 그 이벤트는 DOM 트리를 통해 여러 상위 요소로 전달된다.

---

## 12.3 이벤트 전파 단계

이벤트 propagation은 크게 세 단계로 나눌 수 있다.

| 단계 | 설명 |
|---|---|
| Capturing Phase | 이벤트가 상위 요소에서 하위 요소 방향으로 내려가는 단계 |
| Target Phase | 이벤트가 실제 대상 요소에 도달한 단계 |
| Bubbling Phase | 이벤트가 대상 요소에서 상위 요소 방향으로 올라가는 단계 |

---

## 12.4 Capturing Phase

Capturing phase는 이벤트가 `window`, `document`, `<html>`, `<body>` 같은 상위 요소에서 시작하여 실제 target 요소 방향으로 내려가는 단계이다.

이 단계에서 이벤트를 감지하려면 `addEventListener()`의 세 번째 인수로 `true`를 전달해야 한다.

```javascript
parent.addEventListener("click", function() {
  console.log("캡처링 단계에서 실행");
}, true);
```

---

## 12.5 Target Phase

Target phase는 이벤트가 실제 이벤트 대상 요소에 도달한 단계이다.

예를 들어 버튼을 클릭했다면, 버튼 자체가 이벤트의 target이 된다.

```javascript
button.addEventListener("click", function(event) {
  console.log(event.target);
});
```

---

## 12.6 Bubbling Phase

Bubbling phase는 이벤트가 target 요소에서 시작하여 부모 요소 방향으로 올라가는 단계이다.

`addEventListener()`의 세 번째 인자를 전달하지 않으면 기본적으로 bubbling phase에서 이벤트를 감지한다.

```javascript
parent.addEventListener("click", function() {
  console.log("버블링 단계에서 실행");
});
```

---

## 12.7 버블링으로 인한 문제

버튼에 클릭 이벤트를 등록하고, 부모 요소에도 클릭 이벤트를 등록한 경우 버튼을 클릭했을 때 부모의 이벤트 핸들러까지 실행될 수 있다.

```javascript
button.addEventListener("click", function() {
  console.log("버튼 클릭");
});

parent.addEventListener("click", function() {
  console.log("부모 클릭");
});
```

버튼을 클릭했지만 이벤트가 부모로 전파되기 때문에 `"부모 클릭"`까지 출력될 수 있다.  
이것이 원하지 않는 동작일 수 있다.

---

## 12.8 stopPropagation

이벤트가 부모로 전파되는 것을 막고 싶다면 `event.stopPropagation()`을 사용한다.

```javascript
button.addEventListener("click", function(event) {
  event.stopPropagation();
  console.log("버튼 클릭");
});
```

`event.stopPropagation()`을 호출하면 해당 요소의 핸들러는 실행되지만, 이후 부모로의 이벤트 전파는 중단된다.

---

# 13. 핵심 암기 정리

## 13.1 DOM 핵심

| 질문 | 답 |
|---|---|
| DOM이란? | HTML 문서를 객체 형태로 표현한 구조 |
| DOM은 왜 필요한가? | 자바스크립트가 HTML 요소를 선택하고 조작하기 위해 필요하다. |
| document 객체란? | 웹 페이지 전체를 의미하는 객체 |
| element 객체란? | 개별 HTML 요소를 의미하는 객체 |
| node란? | DOM에서 정보를 저장하는 계층적 단위 |

---

## 13.2 HTML 요소 선택 핵심

| 목적 | 메서드 |
|---|---|
| id로 하나 선택 | `getElementById()` |
| class로 여러 개 선택 | `getElementsByClassName()` |
| name으로 여러 개 선택 | `getElementsByName()` |
| CSS 선택자로 하나 선택 | `querySelector()` |
| CSS 선택자로 모두 선택 | `querySelectorAll()` |

---

## 13.3 HTML 요소 조작 핵심

| 목적 | 사용 |
|---|---|
| 스타일 변경 | `요소.style` |
| HTML 내용 변경 | `요소.innerHTML` |
| 텍스트 변경 | `요소.innerText` |
| 속성 조회 | `요소.getAttribute()` |
| 속성 변경 | `요소.setAttribute()` |
| class 수정 | `요소.classList` |

---

## 13.4 이벤트 핵심

| 개념 | 설명 |
|---|---|
| 이벤트 | HTML 요소에서 발생하는 사건 |
| 이벤트 핸들러 | 이벤트 발생 시 실행되는 함수 |
| `addEventListener()` | 이벤트 핸들러를 등록하는 대표적인 메서드 |
| `event` 객체 | 이벤트 정보와 기능을 담은 객체 |
| `event.target` | 이벤트가 실제 발생한 대상 요소 |
| `preventDefault()` | 기본 동작을 막는 메서드 |
| `removeEventListener()` | 이벤트 핸들러를 제거하는 메서드 |
| `stopPropagation()` | 이벤트 전파를 중단하는 메서드 |

---

## 13.5 이벤트 전파 핵심

| 단계 | 방향 | 설명 |
|---|---|---|
| Capturing Phase | 상위 → 하위 | 이벤트가 target을 향해 내려가는 단계 |
| Target Phase | target | 이벤트가 실제 대상에 도달한 단계 |
| Bubbling Phase | 하위 → 상위 | 이벤트가 부모 방향으로 올라가는 단계 |

---

# 14. 전체 요약

DOM은 HTML 문서를 자바스크립트가 다룰 수 있도록 객체 형태로 표현한 구조이다.  
브라우저는 HTML 코드를 해석해 DOM 트리를 만들고, 각 태그는 node 객체로 변환된다.  
자바스크립트는 `document` 객체를 통해 HTML 요소를 선택하고, 선택한 element 객체의 내용, 스타일, 속성 등을 조작할 수 있다.

이벤트는 사용자의 클릭, 입력, 키보드 조작처럼 웹 페이지에서 발생하는 사건이다.  
이벤트가 발생하면 이벤트 핸들러가 실행되고, 이때 `event` 객체를 통해 이벤트 정보에 접근할 수 있다.  
또한 이벤트는 DOM 트리를 따라 전파되며, 전파 단계에는 Capturing, Target, Bubbling이 있다.  
필요한 경우 `preventDefault()`로 기본 동작을 막고, `stopPropagation()`으로 이벤트 전파를 중단할 수 있다.

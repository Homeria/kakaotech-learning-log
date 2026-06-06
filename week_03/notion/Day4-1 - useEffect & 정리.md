<aside>
📌

**핵심 요약**

`useEffect`는 컴포넌트가 화면에 렌더링된 이후에 실행되는 Hook으로, 크게 **두 가지 목적**을 위해 사용됩니다.

1. **외부 시스템과의 연결 (Side Effect 처리):** API 호출, 타이머 설정 등 React 컴포넌트 바깥의 일을 처리할 때 사용합니다.
2. **코드 실행 타이밍 통제:** 특정 코드가 렌더링될 때마다 무의미하게 반복 실행되는 것을 막고, **"처음 한 번만"** 또는 **"특정 데이터가 변할 때만"** 실행되도록 통제합니다.
</aside>

---

## 1. 생명주기(Lifecycle)랑 useEffect

> "메시지 창이 열렸을 때 자동으로 메시지를 불러오고 싶다면, 코드를 어디에 써야 할까요?"
> 

컴포넌트의 생명주기:

```jsx
컴포넌트 마운트(Mount)   →   업데이트(Update)   →   언마운트(Unmount)
    (화면에 등장)         (state/props 변경)       (화면에서 사라짐)
```

!image.png

`useEffect`를 사용하면 이 생명주기의 특정 시점에 코드를 실행할 수 있습니다.

---

## 2. useEffect 기본 구조

<aside>
📌

`useEffect(실행할_함수, [의존성_배열]);`

- 위 형태가 useEffect의 기본 구조입니다.
- **실행할_함수 (Callback):** 화면이 그려진 후 실행할 실제 작업 코드입니다. (API 호출, 타이머 등)
- **[의존성_배열] (Dependency Array):** 이 함수를 **'언제'** 실행할지 결정하는 방아쇠(Trigger) 역할입니다. 배열 안의 값이 변할 때만 앞의 함수가 다시 실행됩니다.
- **동작 순서:** 컴포넌트 렌더링(화면 그리기) ➔ useEffect 내부 함수 실행
</aside>

```jsx
import { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // ① 매 렌더링마다 실행 (의존성 배열 없음)
  useEffect(() => {
    console.log("렌더링되었습니다");
  });

  // ② 컴포넌트 마운트 시 1회만 실행 (빈 배열 [])
  // (※ 개발 모드에서는 Strict Mode 때문에 콘솔에 두 번 출력되는 것이 정상입니다)
  useEffect(() => {
    console.log("컴포넌트가 화면에 나타났습니다!");
  }, []);

  // ③ count 변경 시 실행 (count 변경 시마다)
  useEffect(() => {
    console.log("카운트가 변경되었습니다:", count);
  }, [count]); // count가 의존성 배열에 있으면 count 변경 시 실행

  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

| 패턴 | 실행 시점 | 주요 사용처 |
| --- | --- | --- |
| `useEffect(() => {})` | 매 렌더링마다 | 컴포넌트 내부 상태가 하나라도 바뀌면 리렌더링 → 거의 사용하지 않음 |
| `useEffect(() => {}, [])` | 마운트 시 1회 | 초기 데이터 로드, 구독 |
| `useEffect(() => {}, [dep])` | dep 변경 시 | 특정 상태 변화에 나타나는 부수효과 |

---

## 3. Cleanup 함수 (useEffect 언마운트 처리)

```jsx
useEffect(() => {
  const intervalId = setInterval(() => {
    console.log("주기적으로 실행");
  }, 1000);

  // 이전 작업을 정리하는 Cleanup 함수
  return () => {
    clearInterval(intervalId); // 컴포넌트 제거 시 interval 정리 → 메모리 누수 방지
  };
}, []);
```

<aside>
💡

**cleanup을 안 하면 생기는 문제**

위 예시에서 `setInterval`을 cleanup 없이 사용하면, 컴포넌트가 화면에서 사라진 후에도 interval이 계속 돌아 메모리를 낭비하고 버그를 유발할 수 있습니다.

</aside>

- **Cleanup 함수가 실행되는 2가지 타이밍**
    1. 컴포넌트가 화면에서 완전히 사라질 때 (Unmount)
    2. 의존성 배열의 값이 바뀌어, 다음 Effect가 실행되기 바로 직전 (이전 작업 정리)
    
    예시 (채팅방 입장/퇴장)
    
    ```jsx
    // 채팅방 번호(roomId)가 바뀔 때의 실행 흐름
    useEffect(() => {
      console.log(`${roomId}번 방에 입장했습니다.`);
    
      return () => {
        console.log(`${roomId}번 방에서 퇴장했습니다.`); // 👈 새 방에 들어가기 직전에 실행!
      };
    }, [roomId]);
    ```
    
    만약 roomId가 1번에서 2번으로 바뀐다면? (콘솔 출력 순서)
    
    1. 1번 방에 입장했습니다. (최초 실행)
    2. 1번 방에서 퇴장했습니다. (2번 방에 들어가기 직전에 기존 1번 방 정리 - Cleanup)
    3. 2번 방에 입장했습니다. (새로운 Effect 실행)

---

## **✏️ [실습] — useEffect 마스터하기**

**의존성 배열 활용 (특정 값이 변할 때만 실행) &  클린업(Cleanup) 함수 (타이머 정리하기)**

```jsx
import { useState, useEffect } from "react";
import TimeLoopUI from "./TimeLoopUI";

export default function TimeLoopMission() {
  const [timeLeft, setTimeLeft] = useState(15);
  const [loopCount, setLoopCount] = useState(0);
  const [cluesFound, setCluesFound] = useState(0);
  const [isPaused, setIsPaused] = useState(false);

  const isSolved = cluesFound >= 3;

  // ================================================================
  // useEffect 1 — 타이머
  //
  // setInterval(콜백, 밀리초) : 지정한 ms 간격으로 콜백 반복 실행
  //                            → 반환값(숫자 id)을 변수에 저장해두세요.
  // clearInterval(id)         : setInterval 이 반환한 id 로 반복 중단
  //                            → id 는 위에서 저장한 변수를 활용합니다.
  // ================================================================
  useEffect(
    () => {
      // [문제 1] isSolved 또는 isPaused 가 true 이면 early return 하세요.

      const interval = setInterval(() => {
        setTimeLeft((prev) => {
          if (prev <= 1) {
            setLoopCount((c) => c + 1);
            return 15;
          }
          return prev - 1;
        });
      }, 1000);

      // [문제 2] interval 을 정리하는 cleanup 함수를 반환하세요.
    },
    [
      /* [문제 3] 의존성 배열을 채우세요. 타이머가 다시 시작돼야 하는 상황을 생각해보세요.
               미션이 완수됐을 때? 일시정지 상태가 바뀔 때? */
    ],
  );

  // ================================================================
  // useEffect 2 — 탭 타이틀 업데이트
  // ================================================================
  useEffect(
    () => {
      document.title =
        loopCount > 0 ? `[루프 ${loopCount}회] 타임루프 작전` : "타임루프 작전";

      // [문제 4] 탭 타이틀을 "실습" 으로 되돌리는 cleanup 함수를 반환하세요.
      //   힌트: document.title = "실습"; 을 cleanup 안에 작성하세요.
    },
    [
      /* [문제 5] 의존성 배열을 채우세요. 탭 타이틀이 바뀌어야 하는 상황은 언제인가요? */
    ],
  );

  function handleCollectClue() {
    if (!isSolved && cluesFound < 3) setCluesFound((c) => c + 1);
  }

  function handleTogglePause() {
    if (!isSolved) setIsPaused((p) => !p);
  }

  return (
    <TimeLoopUI
      timeLeft={timeLeft}
      loopCount={loopCount}
      cluesFound={cluesFound}
      isSolved={isSolved}
      isPaused={isPaused}
      onCollectClue={handleCollectClue}
      onTogglePause={handleTogglePause}
    />
  );
}

```

---

**React 활용 시 3단계 요약**

```
1. UI는 컴포넌트를 조합해서 만든다 (Lego 블록)
2. 데이터(state)가 변하면 화면은 React가 자동 업데이트
3. 부수효과(외부 요청, 타이머)는 useEffect로 관리
```

- ❓ **자주 나오는 질문들**
    
    Q: `useState`와 일반 변수는 어떻게 다른가요?
    
    - A: 일반 변수 변경은 화면 재렌더링을 트리거하지 않습니다. `useState`의 setter로 변경해야 React가 화면을 다시 그립니다.
    
    Q: props는 상태와 뭐가 다른가요?
    
    - A: props는 **부모가 자식에게 전달하는** 데이터, 자식은 수정할 수 없습니다. state는 **컴포넌트 자신이** 관리하는 데이터입니다.
    
    Q: `useEffect` 의존성 배열을 비워두면 매번 실행되는 건가요?
    
    - A: 아닙니다. 빈 배열 `[]`은 마운트 시 1회만 실행됩니다. 의존성 배열 자체를 없애야 매 렌더링마다 실행됩니다.
    - 별개로, 객체나 함수를 의존성 배열에 넣게 된다면 어떻게 될까요?
        - 컴포넌트가 리렌더링될 때마다 useEffect가 멈추지 않고 계속 실행되는 무한 루프가 발생합니다.
        - 숫자/문자와의 차이: 숫자나 문자는 '값 자체'를 비교하지만, 객체나 함수는 메모리에 새로 생성된 '주소(위치)'를 비교하기 때문입니다.
        - 결론: 모양과 내용이 100% 똑같아도 리렌더링할 때마다 매번 새로운 주소(새 집)로 인식되어 React가 값이 바뀌었다고 오해하는 것입니다.
        - https://ko.react.dev/reference/react/useEffect#removing-unnecessary-object-dependencies
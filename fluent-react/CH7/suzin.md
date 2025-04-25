# 7장 리액트 동시성

## 리액트 18부터 도입된 리액트 동시성에 대해 알아봅니다

[리액트18업데이트 공식문서 링크](https://ko.react.dev/blog/2022/03/29/react-v18)

- Auto batching 자동일괄처리 ↔ flushSync (업데이트를 바로 반영하기)
- Transitions 업데이트의 우선순위를 명시적으로 제시하기
- Suspense 서버렌더링을 위한 컴포넌트
- Strict Mode 순수함수인 리액트 컴포넌트 만들기

<br />

## 리액트 동시성이란

- 동시성은 기능이 아닌 내부 구현사항입니다.
- React의 핵심 렌더링 모델에 대한 근본적인 업데이트입니다.
- **동시성의 핵심 | 렌더링이 중단 가능**
  - 단일 동기식 트랜잭션 렌더링 → 렌더링 일시중지 및 완전히 중단 가능
  - 렌더링이 중단 되더라도 일관적인 UI를 표시하도록 보장
  - 메인 스레드를 차단 하지 않고 백그라운드에서 새 화면을 준비하는 것이 가능
  - 대규모 렌더링 작업 중에도 사용자 입력에 즉시 반응하여 유동적인 사용자 경험 제공

<br />

## 7.1 동기식 렌더링

❗ **동기식 렌더링의 문제**

- 대규모 업데이트 시 메인스레드를 가로 막아 UX 저하
- UI 반응이 느려지거나 응답을 하지 않음

<br />

리액트는 자동 일괄 처리*AutoBatching*로 이 문제를 해결하고자 하였습니다.

```jsx
// 변경 전: 오직 React 이벤트만 Batch 됩니다.
setTimeout(() => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  // React는 각 State 업데이트마다 한 번씩, 총 두 번 렌더링 됩니다. (Batching 없음)
}, 1000);

// 변경 후: setTimeout, Promises 내에서도 업데이트할 수 있습니다.
// 네이티브 이벤트 핸들러 또는 다른 이벤트들이 Batch 됩니다.
setTimeout(() => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  // React는 마지막에 한 번만 다시 렌더링 됩니다. (그게 Batching 이죠!)
}, 1000);
```

<br />

❗ **자동 일괄 처리로도 해결 되지 않은 문제**

- 우선 순위 개념이 없고 모든 업데이트가 동일하게 취급 (스택 재조정자)

리액트는 업데이트 작업의 중요도와 긴급도에 따라 우선순위를 매길 수 있도록
파이버 재조정자와 스케줄링을 통해 이 문제를 해결하고자 하였습니다.

<br />

💡 **동시성 렌더링이 하고자 한 것**

- CPU를 많이 소모하는 렌더링 작업은 덜 중요한 작업으로 간주할 것
- 사용자 상호작용 및 애니메이션 작업을 우선적으로 처리할 것
- **타임 슬라이싱** | 렌더링 프로세스를 더 작은 덩어리로 분할해 점진적으로 처리하는 기법
- 여러 프레임에 걸쳐 작업을 수행하면서 작업을 중지할 수 있을 것
</aside>

- 타임슬라이싱*timeslicing*
  렌더링 작업을 한번에 수행하지 않고 여러조각으로 나누어 브라우저가 여유가 있을 때마다 점진적 처리를 하는 방식으로 인터페이스가 버벅거리지 않고 자연스럽게 반응할 수 있습니다.
  브라우저는 1초에 60프레임 60fps 를 목표로 화면을 그립니다. 1프레임당 16.67ms 안에 모든 작업을 끝내야 합니다.
  1. 사용자 입력처리 → 이벤트 리스닝
  2. 자바스크립트 실행 → React의 렌더링 작업을 포함
  3. 스타일 계산과 레이아웃
  4. 페인팅 → 요소를 픽셀로 그리는 작업으로 CPU 작업
  5. 합성 → 여러 레이어를 GPU 로 병합하는 과정
     이 모든 걸 16.67ms 안에 처리하지 못하면 프레임 드롭이 발생해 화면이 끊기거나 버벅입니다.

<br />

## 7.2 파이버 다시보기

파이버 재조정자가 동시성 렌더링에서 지원하는 바는 다음과 같습니다.

- 렌더링 프로세스를 더 작고 관리하기 쉬운 단위로 분할처리
- 렌더링 작업을 일시적으로 중지 하거나 재개, 완전히 중단 가능
- 우선순위를 설정, 중요도에 따라 업데이트를 지연하거나 예약

<br />

## 7.3 업데이트 예약과 지연

리액트의 응답성을 유지하기 위한 업데이트 예약과 지연에 대해 알아봅니다.

스케줄러와 API에 의해서 파이버 재조정자는 유휴시간 동안 작업을 수행하고 업데이트를 예약할 수 있습니다.

- **스케줄러 |** 해야할 업데이트 작업이 있는 경우 브라우저 API를 사용하여 작업을 예약 및 관리 하는 시스템

**Example** 메시지 입력과 전송, 새로 들어오는 메시지가 있는 경우

1. **메시지 입력과 전송 |** 사용자 상호 작용을 우선 시
   1. 사용자와 상호 작용이 있는 메시지 입력 이벤트로 인한 업데이트를 우선 시
   2. 텍스트 입력 `<input>` 컴포넌트 의 입력 업데이트를 다른 업데이트 보다 우선 시하여 처리 가능
2. 새로 들어오는 메시지를 보여주는 것
   1. 새 메시지가 도착해 렌더링 하는 것은 렌더레인을 통해 렌더링
   2. 레인은 동기식으로 DOM을 업데이트하여 다른 업데이트를 가로 막으므로, 사용자 입력에 지연 발생
   3. 이 작업의 우선순위를 낮추도록 트랜지션*Transitions* 기능 사용

→ ✨`useTransition` 훅의 등장 :non-blocking update:

<br />

## 7.4 더 깊이 들어가기

### 7.4.1 스케줄러

- **스케줄러 |** 타이밍 관련 유틸리티를 제공하는 독립형 패키지
- **리액트 파이버 재조정자 |** 스케줄러를 사용하여 해야할 업데이트 작업을 조정
  → 긴급도에 따라 우선순위를 설정하여 렌더레인에 반영
- 스케줄러의 주된 기능

  - 마이크로 태스크를 예약하여 메인 스레드의 제어를 관리

    📢 **루트에 업데이트가 발생할 때 마다 호출하는 `eusureRootIsSchedule`**

    - `root: FiberRoot` 업데이트를 수행하는 커밋 단계의 가장 마지막에 교체되는 것입니다.
    - `eusureRootIsSchedule` 렌더링 프로세스를 관리하는 함수입니다.

    ***

    1. 루트가 스케줄에 포함되도록 보장 (루트 스케줄: 처리되어야 할 루트를 표현한 목록)

       `mightHavePendingSyncWork` 플래그를 `true`로 설정하여 대기중인 동기작업이 있을 가능성을 표기 → `flushSync` 사용 시 `false` 로 표기하여 바로 종료하도록 함

    2. 루트 스케줄을 처리할 수 있도록 대기중인 마이크로 태스크가 존재하는지 확인
       `enableDeferRootSchedulingToMicroTask` 플래그 비활성화 시 마이크로태스크 큐 작업을 지연하지 않고 즉시 렌더링 작업 실행

    - 마이크로큐와 매크로 태스크 큐
      - 마이크로큐 | 이벤트 처리, `setTimeout`, `setInterval`, 입출력 작업
      - 매크로 큐 | `프라미스`, `Object.observe`, `MutationObserver` 연산 작업
      - 현재 작업 종료 → 마이크로 큐 확인 → 존재 시 모두 실행 → 매크로 큐 확인
      - **마이크로 큐 작업 우선순위가 매크로 큐보다 항상 높으므로 스케줄러는 이를 사용합니다**
      - 마이크로 큐에 작업이 너무 많으면 메인 스레드가 절대 돌아오지 않는데요 이를 자원을 할당받지 못하는 기아상태라 부릅니다.
    - 스케줄러는 작업이 속하는 렌더레인을 기반으로 작업을 예약합니다.
      - 다음 레인이 `Sync` ⇒ 즉시 처리 위해 마이크로 태스크를 대기열에 추가
      - 다음 레인이 `Sync` 가 아닌 경우 ⇒ 콜백 예약 후 다음 레인 처리

<br />

# 7.5 렌더레인

- **예약 시스템 |** 작업의 렌더링과 우선순위 관리를 효율화합니다.
- **레인 Lane |** 우선순위 수준을 나타내는 단위이자 렌더링 주기에서 처리할 수 있는 작업
  ↔ 이전에는 만료시간*expiration time*이 있는 예약 메커니즘을 사용
- `setState` 호출 → 우선순위가 가장 높은 `Sync` Lane 에 배정 → 마이크로 태스크로 예약
- `startTransition` 내 에서 `setState` 호출 → 우선순위가 낮은 `Transition` Lane에 배정 → 마이크로 태스크로 예약

<br />

### 7.5.1 렌더레인 작동방식

- **업데이트 우선순위에 따라 렌더레인 할당 |** 업데이트 종류와 컴포넌트 가시성에 따라
- 렌더 레인에 업데이트를 예약하고 우선순위를 정하는 과정
  1. 업데이트 수집: 마지막 렌더링 이후 예약된 모든 업데이트 수집 → 우선순위에 따라 각 레인에 할당
  2. 레인 처리: 우선순위가 가장 높은 레인부터 각 레인의 업데이트 처리. 이때 같은 레인업데이트는 일괄처리
  3. 커밋 단계: 모든 업데이트 처리 후 변경사항을 DOM에 적용하고 효과 실행 및 마무리
  4. 반복
- 업데이트가 발생하면 우선순위 결정과 레인 할당을 위해 리액트가 하는 일

  1. 업데이트의 콘텍스트 확인: 업데이트 콘텍스트 평가
     **콘텍스트 |** 사용자 상호작용, 상태변경, 프롭변경으로 인한 내부업데이트, 서버응답에 의한 업데이트
  2. 우선순위 추정: 콘텍스트에 따라 내부 휴리스틱 추정 (개발자의 개입없이도 효율적으로 동작)
  3. 우선순위의 재정의가 존재하는지 확인: `useTransition` `useDeferredValue` 사용하여 업데이 우선순위를 명시적으로 설정
  4. 올바른 레인에 업데이트 할당: 비트마스크를 사용

- 비트마스크
  각 레인을 bit로 표기합니다. 비트마스크는 레인을 효율적으로 표현할 수 있는 이진법 구조입니다.
  ```jsx
  const SyncLane = 0b0000000000000000000000000000001; // 동기 렌더링 (가장 우선)
  const InputContinuousLane = 0b0000000000000000000000000000100; // 연속 입력
  const DefaultLane = 0b0000000000000000000000000100000; // 일반적인 렌더링
  ```
  각 레인은 고유한 비트 위치를 갖고 있습니다. 비트연산을 통해 빠르게 스케줄링이 가능합니다.

### 7.5.2 레인처리

- `ImmediatePriority` : 메시지 입력에 대한 업데이트 처리
- `UserBlockingPriority` : 입력 상태에 대한 업데이트 처리
- `NormalPriority`: 메시지 목록의 업데이트처리

### 7.5.3 커밋단계

- 모든 업데이트를 레인에서 처리한 후 커밋단계 진입
- 변경사항을 DOM에 적용 후 effect 실행하여 마무리 작업
- 얽힘과 리베이스

<br />

## 7.6 useTransition

`useTransition`은 UI의 일부를 백그라운드에서 렌더링 할 수 있도록 해주는 훅입니다.

```jsx
const [isPending, startTransition] = useTransition();
```

- `isPending` 대기중인 Transition 여부를 boolean으로 반환
- `startTransition(action)`
  - ⭐️`action` 에서 호출되는 콜백이 있다면 콜백의 이름은 action이거나 Action 접미사를 포함하여야 합니다.
  - `action` 은 하나 이상의 set 함수를 호출하여 상태를 업데이트 하는 함수입니다.
  - `action` 함수 호출 중 동기적으로 예약된 모든 상태 업데이트를 Transitions로 표시하고, action에서 await된 비동기 호출은 Transition에 포함합니다. await 이후의 set 함수는 startTransition으로 한번 더 감싸도록 합니다. Transitions로 표기된 상태 업데이트는 non-blocking 방식으로 처리되고 불필요한 로딩 표시가 나타나지 않습니다.
  - `startTransition` 에 전달하는 함수는 동기식이여야만 합니다. 즉시 실행하여 실행하는 동안 모든 state 업데이트를 Transition으로 표기합니다(실행을 미루지 않습니다).
  - `startTransition` 함수는 안정된 식별성을 가지므로 Effect의존성에서 생략됩니다.
  - Transition으로 표시된 업데이트는 다른 업데이트에 의해 중단됩니다.
  - ⭐️ Transition 업데이트는 텍스트 입력을 제어하는데 사용불가합니다!
  - 진행중인 Transition이 여러개라면 일괄처리합니다. (제거가능성이 있다고 서술됩니다)

일부 prop 이나 커스텀 훅에 대한 응답을 Transition하기 위해서는 useDeferredValue를 사용하세요.

### 용례

```jsx
import { useState, useTransition } from "react";
import { updateQuantity } from "./api";

function CheckoutForm() {
  const [isPending, startTransition] = useTransition();
  const [quantity, setQuantity] = useState(1);

  function onSubmit(newQuantity) {
    startTransition(
      //Actions 함수라고 부릅니다.
      async function () {
        const savedQuantity = await updateQuantity(newQuantity);
        startTransition(
          //하나의 Transition안에서도 여러개의 Action수행이 가능합니다.
          () => {
            //Transition이 실행되는 동안 UI는 계속 반응합니다(백그라운드 수행)
            setQuantity(savedQuantity);
          }
        );
      }
    );
  }
  // ...
}
```

### startTransition에 전달한 함수는 즉시 실행됩니다.

```jsx
console.log(1);
startTransition(() => {
  console.log(2);
  setPage("/about");
});
console.log(3);
//출력 1 2 3
```

함수 실행이 지연되지 않습니다. 나중에 콜백을 실행하는 것이 아닌, 예약된 모든 상태를 Transition으로 표기합니다.

```jsx
let isInsideTransition = false;

function startTransition(scope) {
  isInsideTransition = true;
  scope();
  isInsideTransition = false;
}

function setState() {
  if (isInsideTransition) {
    // ... Transition state 업데이트 예약 ...
  } else {
    // ... 긴급 state 업데이트 예약 ...
  }
}
```

### Transitions에서 상태 업데이트가 순서대로 이루어지지 않습니다.

startTransition내부에서 await 사용 시 상태 업데이트가 순서대로 발생하지 않을 수 있습니다.

<br />

## 7.7 useDeferredValue

일부 UI 업데이트를 지연시킬 수 있는 훅입니다.

```jsx
const deferredValue = useDeferredValue(value);
```

- `useDeferredValue(value, initialValue?)`
- `value` 지연시키려는 값. 모든 타입 가능
- `initialValue` 컴포넌트 초기 렌더링 시 사용할 값.
  이 옵션을 생략하면 대신 렌더링할 value 이전의 값이 없기 때문에
  초기 렌더링은 지연시키지 않습니다.
- `currentValue` 를 반환
  - 초기 렌더링 중 반환된 지연된 값은 사용자가 제공한 값과 같습니다.
  - 업데이트가 발생하면 먼저 이전 값으로 렌더링을 시도하고 백 그라운드에서 다시 새 값으로 렌더링을 시도합니다.

### 주의사항

- `useDeferredValue`에 전달하는 값은 문자열 및 숫자와 같은 원시값 혹은 컴포넌트의 외부에서 생성된 객체여야 합니다. 렌더링 중에 새 객체를 생성하고 즉시 `useDeferredValue`에 전달하면 렌더링할 때마다 값이 달라져 불필요한 백그라운드 리렌더링이 발생할 수 있습니다.
- `useDeferredValue`가 현재 렌더링(여전히 이전 값을 사용하는 경우) 외에 다른 값을 받으면 백그라운드에서 새 값으로 리렌더링하도록 예약합니다. `value`에 대한 또 다른 업데이트가 있으면 백그라운드 리렌더링은 중단될 수 있습니다. React는 백그라운드 리렌더링을 처음부터 다시 시작할 것입니다. 예를 들어 차트가 리렌더링 가능한 지연된 값을 받는 속도보다 사용자가 Input에 값을 입력하는 속도가 더 빠른 경우, 차트는 사용자가 입력을 멈춘 후에만 리렌더링됩니다.
- `useDeferredValue`는 `<Suspense>`와 통합됩니다. 새로운 값으로 인한 백그라운드 업데이트로 인해 UI가 일시 중단되면 사용자는 Fallback을 볼 수 없습니다. 데이터가 로딩될 때까지 이전 지연된 값이 표시됩니다.
- `useDeferredValue`는 그 자체로 추가 네트워크 요청을 방지하지 않습니다.
- `useDeferredValue` 자체로 인한 고정된 지연은 없습니다. React는 원래의 리렌더링을 완료하자마자 즉시 새로운 지연된 값으로 백그라운드 리렌더링 작업을 시작합니다. **그러나 이벤트로 인한 업데이트(예: 타이핑)는 백그라운드 리렌더링을 중단하고 우선순위를 갖습니다.**
- `useDeferredValue`로 인한 백그라운드 리렌더링은 화면에 커밋될 때까지 Effect를 실행하지 않습니다. 백그라운드 리렌더링이 일시 중단되면 데이터가 로딩되고 UI가 업데이트된 후에 해당 Effect가 실행됩니다.

<br />

### useTransitions와 useDeferredValue

- useTransitions는 상태업데이트 작업을 낮은 우선순위로 처리하고 싶을 때 사용합니다.
- 사용자 상호작용이 끊기지 않고 무거운 UI업데이트가 백그라운드에서 발생되도록 합니다.
- 우선 순위가 낮다는 것을 수동으로 제어하는 방법입니다.

---

- useDefferredValue는 특정 값의 적용(리렌더링)을 지연시키고 싶을 때 사용합니다.
- 리렌더링 후 지연된 값을 사용하도록 하며, 우선순위를 낮추는 것을 자동으로 적용시켜줍니다.

```jsx
function useDeferredValue(value) {
  const [newValue, setNewValue] = useState(value);

  useEffect(() => {
    //값이 변경되면 트랜지션 후 지연된 업데이트 후 값을 반환
    startTransition(() => {
      setNewValue(value);
    });
  }, [value]);

  return newValue;
}
```

### 용례

`useTransition`

```jsx
const [isPending, startTransition] = useTransition();
const [query, setQuery] = useState("");
const [list, setList] = useState([]);

const handleChange = (e) => {
  const value = e.target.value;
  setQuery(value);

  // 무거운 작업을 낮은 우선순위로 처리
  startTransition(() => {
    const filtered = expensiveFilter(value); // 예: 긴 목록 필터링
    setList(filtered);
  });
};
```

`useDeferredValue`

```jsx
const deferredQuery = useDeferredValue(query); //업데이트는 바로되고
const filteredList = useMemo(
  () => expensiveFilter(deferredQuery),
  [deferredQuery]
);
//값을 넘기는 것을 지연시켜 렌더링 작업을 지연
```

지연된 “background” 렌더링은 중단할 수 있습니다. 예를 들어 Input을 다시 입력하면 React는 지연된 값을 버리고 새 값으로 다시 시작합니다. React는 항상 가장 최근에 제공받은 값을 사용합니다.

<br />

### 디바운싱과 스로틀링

**디바운싱 | 사용자가 입력을 완료할 때 까지 1초 정도 기다린 후 목록 업데이트를 수행 → 지연 시간을 두는 방식**
**쓰로틀링 | 목록을 일정한 간격으로 업데이트 → 목록을 1초에 한번만 업데이트**

- 개발자가 임의의 지연시간을 설정해야 한다.
- useDeferredValue는 지연시간을 동적으로 조정하는 방식이며 지연된 렌더링을 중단할 수 있다.
- 리액트의 기능으로 렌더링 최적화에 도움이 된다.

<br />

## 7.8 동시성 렌더링 관련 문제

❓ **티어링 _tearring_**
업데이트 순서가 어긋나면서 UI일관성을 잃게 되는 문제

### useSyncExternalStore

`useSyncExternalStore`는 외부 store를 구독할 수 있는 훅입니다.
- 동시적 렌더링에서 일관된 상태를 보장 (↔ 18미만 동기적 렌더링)
- 저장소가 변경되면 동기적으로 리렌더링을 강제 적용

```jsx
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)
```

store에 있는 데이터의 스냅샷을 반환합니다. 두 개의 함수를 인수로 전달해야 합니다.

1. `subscribe` 함수는 store를 구독하고 구독을 취소하는 함수를 반환해야 합니다.
2. `getSnapshot` 함수는 store에서 데이터의 스냅샷을 읽어야 합니다.

- `subscribe` 하나의 callback 인수를 받아 store에 구독하는 함수. store가 변경될 때 제공된 callback이 호출되어 리액트가 getSnapshot을 필요시 다시 호출하며 컴포넌트를 다시 렌더링 하도록 해야 합니다. 구독 정리하는 함수를 반환합니다.
- `getSnapshot` 컴포넌트에 필요한 store 데이터의 스냅샷을 반환하는 함수입니다. store가 변경되지 않은 상태에서 getSnapshot을 반복적으로 호출하면 동일한 값을 반환해야 합니다. 저장소가 변경되어 반환된 값이 다르면 리액트는 컴포넌트를 리렌더링 합니다.
- _optional getServerSnapshot_: \*\*store에 있는 데이터의 초기 스냅샷을 반환합니다. 서버 렌더링 도중과 클라이언트에서 서버 렌더링 된 컨텐츠의 하이드레이션 중에만 사용됩니다. 서버 스냅샷은 클라이언트와 서버간에 동일해야 하며, 직렬화 되어 서버에서 클라이언트로 전달됩니다. 이 함수가 제공되지 않는다면 서버에서 컴포넌트 렌더링 시 오류가 발생합니다.
- `return` 렌더링 로직에 사용할 수 있는 store 의 현재 스냅샷을 반환합니다.



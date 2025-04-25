# Chapter 7 리액트 동시성 (React Concurrency)

## 소개

**리액트 동시성**은 UI의 응답성과 사용자 경험을 크게 향상시키는 리액트의 핵심 기능입니다.

동시성 기능은 업데이트 작업의 중요도와 긴급성에 따라 우선순위를 설정하여, 복잡한 애플리케이션에서도 높은 사용자 경험을 제공합니다.

### 1. 동기식 렌더링의 문제

#### 주요 문제점

- 메인 스레드 차단으로 인한 사용자 경험 저하
- 복잡한 애플리케이션에서 UI 반응성 저하
- 모든 업데이트가 동일한 우선순위로 처리됨

#### 한계

- 일괄 처리(batching)만으로는 해결에 한계가 있음
- 보이지 않는 요소도 동일한 우선순위로 렌더링
- 중요하지 않은 업데이트가 중요한 업데이트를 차단

### 2. 파이버 재조정자(Fiber Reconciler)

파이버 재조정자는 리액트 16에서 도입되어 이전의 스택 재조정자를 대체했습니다.

#### 핵심 특징

- 렌더링 프로세스를 작은 작업 단위(파이버)로 분할
- 작업 일시 중지, 재개, 우선순위 설정 가능
- 애플리케이션 응답성 향상과 중요 업데이트 차단 방지

### 3. 업데이트 예약과 지연

#### 핵심 메커니즘

- 스케줄러와 API를 통한 효율적인 구현
- 브라우저 API(setTimeout, MessageChannel)를 통한 작업 관리
- 중요도에 따른 작업 분리

#### 실시간 채팅 애플리케이션 예시

```jsx
const ChatApp = () => {
  const [messages, setMessages] = useState([]);
  const [isPending, startTransition] = useTransition();

  useEffect(() => {
    const socket = new WebSocket("wss://your-websocket-server.com");

    socket.onmessage = (event) => {
      // 메시지 업데이트를 낮은 우선순위로 예약
      startTransition(() => {
        setMessages((prevMessages) => [...prevMessages, event.data]);
      });
    };

    return () => socket.close();
  }, []);

  return (
    <div>
      <MessageList messages={messages} />
      <MessageInput onSubmit={sendMessage} />
      {isPending && <div>새 메시지 로딩 중...</div>}
    </div>
  );
};
```

> [!TIP] 위 예시에서 사용자 입력 처리는 높은 우선순위로, 메시지 목록 업데이트는 낮은 우선순위로 처리되어 UI가 항상 응답성을 유지합니다.

### 4. 스케줄러

#### 스케줄러의 역할

- 리액트 아키텍처의 핵심 컴포넌트
- 타이밍 관련 유틸리티 제공
- 마이크로태스크 예약을 통해 메인 스레드 제어

#### 이벤트 루프와 마이크로태스크

| 큐 유형           | 처리 작업                            | 우선순위 |
| ----------------- | ------------------------------------ | -------- |
| 매크로태스크 큐   | 이벤트 처리, setTimeout, 입출력 작업 | 낮음     |
| 마이크로태스크 큐 | Promise, MutationObserver            | 높음     |

실행 순서: 매크로태스크 완료 → 마이크로태스크 처리 → 다음 매크로태스크

#### 스케줄러 동작

```jsx
if (nextLane === Sync) {
  queueMicrotask(processNextLane); // 높은 우선순위
} else {
  Scheduler.scheduleCallback(callback, processNextLane); // 일반 우선순위
}
```

### 5. 렌더 레인(Render Lanes)

렌더 레인은 리액트 18에서 도입된 우선순위 관리 시스템으로, 이전의 만료 시간(expiration time) 방식을 대체했습니다.

#### 주요 렌더 레인 종류

- **SyncLane**: 클릭 이벤트 처리(최고 우선순위)
- **InputContinuousLane**: 호버, 스크롤 등 연속 이벤트
- **DefaultLane**: 네트워크 업데이트, 타이머
- **TransitionLanes**: startTransition의 트랜지션(낮은 우선순위)
- **RetryLanes**: Suspense 재시도

#### 렌더 레인 작동 방식

1. **업데이트 수집**: 우선순위에 따라 레인 할당
2. **레인 처리**: 우선순위 높은 레인부터 처리
3. **커밋 단계**: DOM 적용, 효과 실행
4. **반복**: 우선순위 기반 처리 반복

### 6. useTransition

**useTransition**은 컴포넌트 내에서 상태 업데이트 우선순위를 관리하는 훅입니다.

#### 핵심 개념

- 우선순위가 높은 업데이트로 인한 UI 응답성 저하 방지
- 새 데이터 로드나 페이지 이동 시 유용

#### 작동 원리

- startTransition으로 감싼 업데이트는 낮은 우선순위 레인에 배치
- 함수 컴포넌트 내에서만 사용 가능

#### 반환값

- **isPending**: 트랜지션 진행 여부 표시 불리언
- **startTransition**: 지연 업데이트를 감싸는 함수

#### 간단한 예시

```jsx
function App() {
  const [count, setCount] = useState(0);
  const [isPending, startTransition] = useTransition();

  const handleClick = () => {
    // 중요한 작업 즉시 실행
    doSomethingImportant();

    // 카운터 업데이트는 낮은 우선순위로 지연
    startTransition(() => {
      setCount(count + 1);
    });
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>증가</button>
      {isPending && <p>업데이트 중...</p>}
    </div>
  );
}
```

### 7. useDeferredValue

**useDeferredValue**는 특정 UI 업데이트를 지연시키는 훅입니다.

#### 핵심 개념

- 과부하 작업이나 연산 집약적 작업 처리 시 유용
- 부드러운 전환과 향상된 사용자 경험 제공

#### 작동 원리

- 초기 렌더링: 전달된 값과 동일한 값 반환
- 이후 업데이트: 새 값으로의 업데이트 시점 제어
- 내부적으로 startTransition 사용하여 구현

```jsx
// 초기 구현 원리
function useDeferredValue(value) {
  const [newValue, setNewValue] = useState(value);
  useEffect(() => {
    startTransition(() => {
      setNewValue(value);
    });
  }, [value]);
  return newValue;
}
```

#### 사용 예시: 검색 기능

```jsx
function SearchComponent() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);

  const searchResults = useMemo(() => {
    // 비용이 많이 드는 검색 연산
    return items.filter((item) => item.toLowerCase().includes(deferredQuery.toLowerCase()));
  }, [deferredQuery]);

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} placeholder="검색어를 입력하세요" />

      {query !== deferredQuery && <p>검색 결과 업데이트 중...</p>}

      <ul>
        {searchResults.map((item) => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

#### useDeferredValue vs 디바운싱/스로틀링

| 기법             | 특징                                         | 사용 상황          |
| ---------------- | -------------------------------------------- | ------------------ |
| 디바운싱         | 입력 완료 후 일정 시간 기다린 후 업데이트    | 검색창 API 호출    |
| 스로틀링         | 일정 간격으로만 업데이트                     | 스크롤 이벤트 처리 |
| useDeferredValue | 기기 성능에 맞춘 동적 지연, 렌더링 중단 가능 | 복잡한 목록 필터링 |

### 8. 동시성 렌더링 관련 문제: 티어링(Tearing)

#### 티어링이란?

- 컴포넌트가 의존하는 상태가 업데이트될 때 발생하는 일관성 문제
- 일관되지 않은 데이터로 UI가 렌더링되는 현상

#### useSyncExternalStore로 티어링 해결

```jsx
const store = {
  callbacks: [],
  subscribe(callback) {
    store.callbacks.push(callback);
    return () => {
      store.callbacks = store.callbacks.filter((c) => c !== callback);
    };
  },
  getSnapshot() {
    return count; // 현재 count 값의 일관된 스냅샷 반환
  },
};

const ExpensiveComponent = () => {
  // useSyncExternalStore로 일관된 상태 유지
  const consistentCount = useSyncExternalStore(store.subscribe, store.getSnapshot);

  return <div>비용이 비싼 카운트: {consistentCount}</div>;
};
```

> [!NOTE] useSyncExternalStore의 주요 역할은 동시적 렌더링에서 일관된 상태를 보장하고 저장소 변경 시 동기적 리렌더링을 강제 적용하는 것입니다.

## 결론

리액트 동시성 기능은 복잡하고 동적인 애플리케이션에서 부드럽고 응답성 좋은 사용자 경험을 제공하기 위한 핵심 메커니즘입니다. 파이버 아키텍처, 렌더 레인, 그리고 useTransition, useDeferredValue, useSyncExternalStore 같은 API들을 통해 리액트 애플리케이션의 성능을 최적화하고 사용자 경험을 크게 향상시킬 수 있습니다.<br /><br />
동시성 기능의 핵심은 작업의 우선순위를 관리하고, 중요한 사용자 상호작용을 우선적으로 처리하여 UI가 항상 응답성을 유지하도록 하는 것입니다.

## 복습하기 (Q&A)

### 1. 리액트의 파이버 재조정자는 무엇이며, 복잡한 고성능 애플리케이션을 처리하는 데 어떻게 도움이 되나요?

- 파이버 재조정자는 렌더링 엔진으로 렌더링 작업을 작은 단위(파이버)로 분할하여 처리합니다
- 작업의 일시 중지, 재개, 우선순위 설정이 가능해집니다
- 무거운 렌더링 작업 중에도 사용자 상호작용을 우선 처리하여 UI 응답성을 유지합니다

### 2. 리액트에서 업데이트 예약 및 지연의 개념을 설명하세요. 부하가 많은 상황에서도 원활한 사용자 경험을 유지하는 데 어떻게 도움이 되나요?

- 작업의 우선순위에 따라 처리 순서를 결정하는 메커니즘입니다
- 높은 우선순위 업데이트(사용자 입력)는 즉시 처리하고, 낮은 우선순위 업데이트는 지연시킵니다
- 사용자 상호작용에 즉각적인 응답을 보장하여 애플리케이션이 항상 반응적으로 느껴지게 합니다

### 3. 리액트의 렌더 레인이란 무엇이며, 업데이트 실행을 어떻게 관리하나요?

- 렌더 레인은 업데이트의 우선순위를 나타내는 단위입니다
- 각 업데이트는 발생 컨텍스트에 따라 적절한 레인에 할당됩니다
- 비트마스크를 사용해 여러 레인을 효율적으로 표현하고 관리합니다
- 우선순위가 높은 레인부터 처리하고, 같은 레인의 업데이트는 일괄 처리합니다

### 4. 리액트에서 useTransition과 useDeferredValue 훅을 사용하는 목적은 무엇인가요?

- **useTransition**: 상태 업데이트의 우선순위를 낮추어 UI 응답성을 유지합니다. 페이지 전환, 대량 데이터 로딩 등에 유용합니다.
- **useDeferredValue**: 특정 값의 업데이트를 지연시켜 중요한 UI 업데이트가 먼저 처리되도록 합니다. 대규모 목록 필터링, 복잡한 데이터 시각화 등에 적합합니다.

### 5. useDeferredValue를 사용하는 것이 부적절한 상황은 어떤 상황일까요? 이 훅을 사용할 때 고려할 장단점은 무엇인가요?

#### 부적절한 상황:

- 사용자 입력에 즉각적인 응답이 필요한 업데이트
- 최신 데이터가 항상 표시되어야 하는 중요한 정보 (예: 실시간 거래 정보)

#### 장점:

- 기기 성능에 맞춘 동적 지연 시간 조정
- 새 입력 발생 시 렌더링 중단 가능
- 선언적 방식의 우선순위 관리

#### 단점:

- 사용자에게 오래된(stale) 데이터가 표시될 수 있음
- 디버깅이 어려울 수 있음
- 효율적인 코드 작성이 여전히 중요함

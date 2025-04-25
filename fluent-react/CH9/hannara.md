# Chapter 9 리액트 서버 컴포넌트

## 1. 리액트 서버 컴포넌트란?

리액트 서버 컴포넌트는 서버에서 렌더링되는 멀티 페이지 애플리케이션(MPA)과 클라이언트에서 렌더링되는 단일 페이지 애플리케이션(SPA)의 장점을 결합한 새로운 패러다임입니다.

### 핵심 개념

- 서버 컴포넌트는 **서버에서만 실행되는** 컴포넌트입니다
- 클라이언트 자바스크립트 번들에 포함되지 않아 번들 크기를 줄입니다
- 서버에서 데이터에 직접 접근하고, 이를 클라이언트 컴포넌트에 프롭으로 전달합니다

### 작동 방식

기본적으로 모든 리액트 컴포넌트는 리액트 엘리먼트를 반환하는 함수입니다:

```javascript
const Component = () => <div>안녕!</div>;
```

서버 컴포넌트도 리액트 엘리먼트를 반환하지만, 서버에서 실행되고 그 결과만 네트워크를 통해 클라이언트로 전송된다는 차이점이 존재합니다.

## 2. 장점

### 2.1 예측 가능한 성능

- 우리가 계산 능력을 제어할 수 있는 서버에서 실행되므로 성능을 더 잘 예측할 수 있습니다.
- 클라이언트 기기의 다양한 성능 차이에 영향을 덜 받습니다.

### 2.2 보안 강화

서버 환경에서 실행되므로 API 키나 토큰과 같은 민감한 정보를 보다 안전하게 다룰 수 있습니다.

```javascript
// 서버 컴포넌트에서는 안전하게 API 키 사용 가능
async function ProductList() {
  const products = await fetch("https://api.example.com/products", {
    headers: {
      Authorization: "Bearer " + process.env.API_SECRET_KEY,
    },
  }).then((res) => res.json());

  return (
    <div>
      <h1>제품 목록</h1>
      <ul>
        {products.map((product) => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 2.3 비동기 지원

서버 컴포넌트는 비동기로 동작할 수 있어, 네트워크를 통해 클라이언트로 전송하기 전에 모든 작업이 완료될 때까지 기다릴 수 있습니다.

```javascript
// 데이터베이스나 파일 시스템에서 직접 데이터 로드 가능
async function BlogPost({ id }) {
  const post = await database.posts.get(id);
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

## 3. 서버 렌더링과의 상호작용

서버 컴포넌트와 서버 렌더링은 두 개의 독립적인 프로세스로 이해할 수 있습니다:

1. **RSC 렌더러**: 서버에서 컴포넌트를 리액트 엘리먼트 트리로 변환합니다
2. **서버 렌더러**: 리액트 엘리먼트 트리를 HTML 마크업으로 변환합니다

### 처리 순서

1. 서버에서 JSX 트리가 엘리먼트 트리로 변환됩니다
2. 서버가 엘리먼트 트리를 직렬화합니다
3. 직렬화된 JSON 객체가 클라이언트로 전송됩니다
4. 클라이언트 측 리액트가 이를 파싱하고 렌더링합니다

### 내부 동작 예시

```javascript
// 서버 측 코드
const express = require("express");
const app = express();

app.get("*", async (req, res) => {
  // 1. RSC 렌더러: 서버 컴포넌트를 리액트 엘리먼트 트리로 변환
  const rscTree = await turnServerComponentsIntoTreeOfElements(<App />);

  // 2. 서버 렌더러: 리액트 엘리먼트를 HTML로 변환
  const html = ReactDOMServer.renderToString(rscTree);

  // 3. 결과 전송
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>My React App</title>
      </head>
      <body>
        <div id="root">${html}</div>
        <script src="/static/js/main.js"></script>
      </body>
    </html>
  `);
});
```

### 직렬화 문제와 해결책

리액트 엘리먼트는 `$$typeof` 속성에 심벌을 포함하고 있어 직렬화가 불가능합니다. 이를 해결하기 위한 특별한 직렬화 로직:

```javascript
// 직렬화
JSON.stringify(jsxTree, (key, value) => {
  if (key === "$$typeof") {
    return "react.element"; // 심벌을 문자열로 변환
  }
  return value;
});

// 역직렬화
JSON.parse(serializedJsxTree, (key, value) => {
  if (key === "$$typeof") {
    return Symbol.for("react.element"); // 문자열을 다시 심벌로 변환
  }
  return value;
});
```

## 4. 페이지 탐색과 업데이트

### 소프트 내비게이션

RSC는 전체 페이지 새로고침 없이 페이지 간 이동을 지원합니다:

```javascript
// 클라이언트 측 코드
window.addEventListener("click", (event) => {
  if (event.target.tagName !== "A") return;

  event.preventDefault();
  navigate(event.target.href);
});

async function navigate(url) {
  const response = await fetch(url, { headers: { "jsx-only": true } });
  const jsxTree = await response.json();
  const element = JSON.parse(jsxTree, (key, value) => {
    if (key === "$$typeof") return Symbol.for("react.element");
    return value;
  });

  root.render(element);
}
```

### 서버와 클라이언트 컴포넌트 분리의 필요성

클라이언트 컴포넌트와 서버 컴포넌트를 구분해야 하는 이유:

```javascript
// 이 컴포넌트는 서버 컴포넌트가 될 수 없음
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}

// 대신 서버/클라이언트 부분을 분리해야 함
// 서버 컴포넌트
function ServerCounter() {
  return (
    <div>
      <h1>안녕 친구들, 내 멋진 카운터를 봐줘!</h1>
      <p>내 소개: 나는 리액트를 좋아해. 방명록에 글 남겨줘!</p>
      <ClientPart />
    </div>
  );
}

// 클라이언트 컴포넌트
("use client");
function ClientPart() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

## 5. 서버 컴포넌트 규칙

### 5.1 직렬화 가능성이 중요

서버 컴포넌트에서는 모든 프롭을 직렬화할 수 있어야 합니다:

```javascript
// 잘못된 예 - 함수는 직렬화할 수 없음
function ServerComponent() {
  return <ClientComponent onClick={() => alert("hi")} />;
}

// 올바른 예 - 이벤트 핸들러를 클라이언트 컴포넌트 내부로 캡슐화
function ServerComponent() {
  return <ClientComponent />;
}

("use client");
function ClientComponent() {
  return <button onClick={() => alert("hi")}>클릭</button>;
}
```

### 5.2 부작용이 있는 훅 금지

서버 컴포넌트에서는 상태 관리나 브라우저 API를 사용하는 훅을 사용할 수 없습니다:

> 왜?
>
> 1. 서버 환경의 본질적 특성
>
> - 서버는 정적이고 비인터랙티브한 환경
> - DOM이나 window 객체가 존재하지 않음
> - 클라이언트 측 상태 관리나 부수 효과를 처리할 수 없음
>
> 2. 렌더링 모델의 차이
>
> - **서버 컴포넌트는 한 번 렌더링되고 직렬화되어 클라이언트로 전송됨**
> - 상태 변경이나 부수 효과는 서버에서 의미가 없음
> - 클라이언트에서 인터랙티브한 상호작용은 클라이언트 컴포넌트에서 처리해야 함
>
> 3. 보안 및 성능 문제
>
> - 서버에서 상태를 관리하면 여러 클라이언트 간 상태 공유 위험 발생
> - 불필요한 서버 리소스 소비
> - 클라이언트별 고유한 상호작용은 클라이언트 측에서 처리하는 것이 더 안전하고 효율적

```javascript
// 잘못된 예 - 서버 컴포넌트에서 상태 관리 훅 사용
function ServerComponent() {
  const [count, setCount] = useState(0); // 오류 발생
  return <div>{count}</div>;
}

// 올바른 예 - DOM에 의존하지 않는 훅은 사용 가능
function ServerComponent() {
  const ref = useRef(null); // OK - DOM 의존성 없음
  return <div>...</div>;
}
```

### 5.3 상태의 차이점

서버와 클라이언트의 상태는 근본적으로 다릅니다:

- 서버는 1:N 관계로 여러 클라이언트를 처리
- 서버 상태가 여러 클라이언트에 공유될 위험성

### 5.4 클라이언트 컴포넌트의 서버 컴포넌트 가져오기 제한

```javascript
// 잘못된 예 - 클라이언트 컴포넌트에서 서버 컴포넌트를 직접 import
"use client";
import { ServerComponent } from "./ServerComponent";

function ClientComponent() {
  return (
    <div>
      <ServerComponent /> {/* 오류 발생 */}
    </div>
  );
}

// 올바른 예 - 프롭을 통한 전달
("use client");
function ClientComponent({ children }) {
  return <div>{children}</div>;
}

// 부모 서버 컴포넌트
import { ServerComponent } from "./ServerComponent";

function ParentComponent() {
  return (
    <ClientComponent>
      <ServerComponent />
    </ClientComponent>
  );
}
```

### 5.5 클라이언트 컴포넌트는 여전히 중요

클라이언트 컴포넌트는 나쁜 것이 아니며, 여전히 리액트 애플리케이션의 핵심입니다. 서버 컴포넌트는 대체가 아닌 보완재입니다.

## 6. 서버 액션

서버 액션은 `"use server"` 지시자를 사용해 클라이언트에서 호출할 수 있는 서버 함수입니다:

```javascript
// 파일 내 특정 함수만 서버 액션으로 지정
async function submitForm(formData) {
  "use server";
  const name = formData.get("name");
  await database.users.add({ name });
  return { success: true };
}

// 또는 파일 전체를 서버 액션으로 지정
("use server");

export async function createPost(formData) {
  const title = formData.get("title");
  const content = formData.get("content");
  await database.posts.create({ title, content });
}
```

### 6.1 폼과 데이터 조작

서버 액션은 HTML 폼과 함께 사용할 수 있습니다:

```javascript
// 서버 액션 정의
async function requestUsername(formData) {
  "use server";
  const username = formData.get("username");
  // 데이터베이스 저장 등의 작업
  return { success: true };
}

// 컴포넌트에서 사용
function SignupForm() {
  return (
    <form action={requestUsername}>
      <input type="text" name="username" />
      <button type="submit">제출</button>
    </form>
  );
}
```

### 6.2 폼 외부에서의 사용

트랜지션과 함께 사용하여 로딩 상태 처리, 낙관적 UI 업데이트 등을 구현할 수 있습니다:

```javascript
"use client";
import { useState, useTransition } from "react";

function LikeButton() {
  const [isPending, startTransition] = useTransition();
  const [likeCount, setLikeCount] = useState(0);

  const incrementLike = async () => {
    "use server";
    // 실제로는 데이터베이스에 좋아요 수 증가 로직
    return likeCount + 1;
  };

  const onClick = () => {
    startTransition(async () => {
      const currentCount = await incrementLike();
      setLikeCount(currentCount);
    });
  };

  return (
    <>
      <p>좋아요 수: {likeCount}</p>
      <button onClick={onClick} disabled={isPending}>
        {isPending ? "처리 중..." : "좋아요"}
      </button>
    </>
  );
}
```

## 7. 리액트 서버 컴포넌트의 미래

### 번들러 통합 개선

웹팩, 롤업, vite 등 주요 번들러와의 통합이 더욱 개선

### 생태계 지원 확대

RSC를 지원하는 도구, 라이브러리, 프레임워크가 지속적으로 등장

## 8. 결론

리액트 서버 컴포넌트는 리액트 생태계의 중요한 진보로, 성능 향상, 데이터 페칭 간소화, 원활한 사용자 경험을 제공합니다. 서버와 클라이언트의 장점을 결합하여 더 효율적인 애플리케이션 개발을 가능하게 합니다.

## 9 복습하기

1 리액트 서버 컴포넌트의 주된 가치는 무엇인가요 ?

- 예측 가능하다 : 서버에서 실행되고 계산을 제어할 수 있기에 성능 예측에 보다 쉽습니다.
- 보안 강화 : 서버 환경에서 실행되어 민감한 정보를 유출될 위험성이 낮다.
- 비동기를 지원 : 네트워크 전송 단계 전 서버 컴포넌트의 작업이 완료될 때까지 기다릴 수 있다.

2 클라이언트 컴포넌트가 서버 컴포넌트를 가져올 수 있나요 ? 그렇게 답변한 이유가 무엇인가요 ?

아니요

- 서버 컴포넌트는 서버에서만 실행, 클라이언트 컴포넌트는 브라우저 환경에서도 실행되기 때문.
- 서버 컴포넌트는 Node.js API와 같은 클라이언트 환경에서 사용할 수 없는 요소를 포함할 수 있다. -> 그렇기에 클라이언트 컴포넌트가 서버 컴포넌트를 가져올 수 없다.
- 만약 **부모 서버 컴포넌트가 서버컴포넌트를 클라이언트 컴포넌트 chileren이나 다른 프롭으로 전달하는 것은 가능**

3 서버 컴포넌트와 기존 클라이언트 전용 리액트 애플리케이션 사이에는 어떤 트레이드오프가 있나요 ?

- 서버, 클라이언트 분리하여 아키텍처 구성해야하여 보다 복잡한 아키텍처를 구성해야함
- 직렬화를 해야한다는 제약점이 존재
- 서버에 의존성이 증가할 수 있음
- 제약적인 훅 사용성, 이벤트 핸들러
- 구현에 대한 복잡성 향상

4 모듈 참조란 무엇이며 , 리액트는 재조정 과정에서 모듈 참조를 어떻게 처리하나요 ?

- **모듈 참조** : 서버에서 렌더링할 때 클라이언트 컴포넌트 위치에 삽입되고, 클라이언트 번들에 있는 특정 모듈을 가리키는 참조
- 서버는 전체 컴포넌트 트리를 렌더링, 클라이언트 컴포넌트는 모듈참조를 렌더링하지 않음
- 모듈 참조가 포함된 리액트 엘리먼트 트리가 클라이언트로 전송됨
- 클라이언트의 리액트 재조정과정에서 만약 모듈참조를 만난다면, 실제 클라이언트 번들에서 모듈을 찾아서 실행한다.
- 이때 클라이언트는 해당 참조를 실제 모듈로 대체함

5 서버 액션은 어떤 방식으로 리액트 애플리케이션의 접근성을 향상시키나요 ?

- JS 번들이 로드되기 전에도 form 전송이 가능하여 초기 인터렉션 속도 향상을 기대할 수 있음
- action속성과 통합하여 JS파일이 비활성화일 때도 기본기능은 작동함
- 상태관리, JS 의존성 없이도 서버와 상호작용 가능
- useTransition을 사용하여 로딩 상태를 표시할 수 있다.
- 에러 처리 개선

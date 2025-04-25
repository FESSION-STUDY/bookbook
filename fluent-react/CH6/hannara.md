# Chapter 6 서버 사이드 리액트

## 클라이언트 사이드 렌더링의 한계

클라이언트 사이드 렌더링(CSR)은 현대 웹 애플리케이션에서 널리 사용되지만, 몇 가지 중요한 한계가 있습니다:

- **SEO 문제**: 검색 엔진 크롤러가 자바스크립트를 제대로 실행하지 못해 콘텐츠 색인 생성에 어려움을 겪습니다
- **성능 저하**: 
  - 콘텐츠를 보기 전 JS를 다운로드, 파싱, 실행해야 함
  - 네트워크 폭포(waterfall) 현상으로 로딩 시간이 길어짐
  - 저성능 기기에서 사용자 경험이 현저히 저하됨
- **보안 취약점**: CSRF 같은 공격에 취약하며 서버 측 검증이 부족합니다

> [!NOTE]
> 네트워크 폭포 현상이란 자바스크립트 파일이 순차적으로 로드되면서 발생하는 지연 현상을 말합니다. 한 파일의 로딩이 완료되어야 다음 파일의 로딩이 시작되는 구조입니다.

## 서버 사이드 렌더링(SSR)의 장점

서버 사이드 렌더링은 다음과 같은 중요한 이점을 제공합니다:

- **최초 의미 있는 페인트 시간 단축**: 서버에서 렌더링된 HTML을 즉시 제공하여 사용자가 빠르게 콘텐츠를 볼 수 있습니다
- **웹 접근성 향상**: 느린 인터넷이나 저전력 기기 사용자에게도 완전히 렌더링된 HTML을 제공합니다
- **SEO 개선**: 검색 엔진 크롤러가 완전히 렌더링된 HTML을 즉시 색인화할 수 있습니다
- **보안 강화**: 서버 측 검증이 가능하고, 민감한 로직을 클라이언트에 노출하지 않을 수 있습니다

```jsx
// 서버 사이드 렌더링 기본 예시 (Express 사용)
const express = require("express");
const React = require("react");
const ReactDOMServer = require("react-dom/server");
const App = require("./src/App");

const app = express();

app.get("*", (req, res) => {
  const html = ReactDOMServer.renderToString(<App />);
  
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>서버 렌더링 앱</title>
      </head>
      <body>
        <div id="root">${html}</div>
        <script src="/static/js/main.js"></script>
      </body>
    </html>
  `);
});

app.listen(3000);
```

### 하이드레이션(Hydration)

하이드레이션은 서버에서 생성된 정적 HTML에 이벤트 리스너와 JS 기능을 추가하는 중요한 과정입니다.

**작동 방식**:
1. 서버: 컴포넌트 렌더링 → HTML 생성 → 클라이언트 전송
2. 클라이언트: HTML 표시 → JS 번들 로드 → DOM에 이벤트 리스너 추가

> [!IMPORTANT]
> 서버 렌더링 결과와 클라이언트 렌더링 결과의 DOM 구조가 일치해야 합니다. 불일치가 발생하면 하이드레이션이 제대로 작동하지 않고 예상치 못한 동작이 발생할 수 있습니다.

```jsx
// 클라이언트 하이드레이션 예시
import React from "react";
import { hydrateRoot } from "react-dom/client";
import App from "./App";

hydrateRoot(
  document.getElementById("root"),
  <App />
);
```

**하이드레이션의 한계**:
- 하이드레이션 과정에서 상당한 JS 실행이 필요함
- 인터랙티브 시간(TTI)에 지연 발생
- 대규모 앱에서 성능 문제 발생 가능

### 재개 가능성(Resumability)

재개 가능성은 하이드레이션의 대안으로 제시되는 접근 방식입니다.

**하이드레이션 vs 재개 가능성 비교**:

| 하이드레이션 | 재개 가능성 |
|------------|------------|
| 서버에서 HTML 생성 후 클라이언트에서 리렌더링 | 서버에서 HTML과 인터랙티브 상태를 함께 전송 |
| 이벤트 리스너 연결을 위한 추가 작업 필요 | 서버 작업 중단 시점부터 바로 재개 가능 |
| 인터랙티브 시간에 지연 발생 | 하이드레이션 단계 생략으로 TTI 단축 |

<br />

> [!TIP]
> 재개 가능성은 성능 측면에서 장점이 있지만, 구현의 복잡성이 증가합니다. 프로젝트의 요구사항과 리소스를 고려하여 선택해야 합니다.

## 리액트의 서버 렌더링 API

리액트는 서버 렌더링을 위한 여러 API를 제공합니다. 각 API는 특정 사용 사례와 환경에 적합합니다.

### renderToString

`renderToString`은 가장 기본적인 서버 렌더링 API로, 리액트 컴포넌트를 HTML 문자열로 변환합니다.

**특징**:
- 동기식 API로 즉시 HTML 문자열 반환
- 간단한 구현과 기본적인 SEO 개선 제공
- 대규모 앱에서 성능 문제 발생 가능

```jsx
import { renderToString } from "react-dom/server";
import App from "./App";

const html = renderToString(<App />);
console.log(html); // "<div data-reactroot="">안녕하세요</div>"
```

**단점**:
- 이벤트 루프 차단으로 서버 성능 저하
- 스트리밍 지원 부족으로 TTFB(Time to First Byte) 지연
- 전체 문자열 생성 전까지 클라이언트가 대기해야 함

### renderToPipeableStream

`renderToPipeableStream`은 리액트 18에서 도입된 Node.js 기반 스트리밍 API입니다.

**작동 과정**:
1. 렌더링 시작 → 준비된 HTML 청크 즉시 전송
2. Suspense 경계 활용 → 데이터 로딩 중 폴백 표시
3. 데이터 로드 완료 → 실제 콘텐츠로 대체

```jsx
import { renderToPipeableStream } from "react-dom/server";
import App from "./App";

app.get("/", (req, res) => {
  const { pipe } = renderToPipeableStream(<App />, {
    onShellReady() {
      // 셸이 준비되면 응답 스트림에 파이프
      res.setHeader("Content-Type", "text/html");
      pipe(res);
    }
  });
});
```

**장점**:
- TTFB 개선으로 사용자가 더 빠르게 콘텐츠 확인 가능
- 메모리 효율성 향상 및 서버 부하 감소
- Suspense와의 완벽한 통합으로 더 나은 로딩 경험 제공

### renderToReadableStream

`renderToReadableStream`은 브라우저 환경에서 사용하는 웹 표준 스트림 API입니다.

```jsx
import { renderToReadableStream } from "react-dom/server";
import App from "./App";

async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js']
  });
  
  return new Response(stream, {
    headers: { "Content-Type": "text/html" }
  });
}
```

> [!NOTE]
> `renderToReadableStream`은 Cloudflare Workers, Deno 같은 웹 표준 환경에서 스트리밍 SSR을 구현할 때 적합합니다.

## 프레임워크 활용의 중요성

서버 렌더링을 직접 구현하는 것보다 **검증된 프레임워크 사용**이 강력히 권장됩니다.

**주요 이점**:

- **에지 케이스 자동 처리**: 복잡한 상황과 예외 케이스에 대한 해결책 내장
- **보안 취약점 방지**: 데이터 격리 및 안전한 캐싱 전략 제공
- **성능 최적화**: 코드 분할, 자동 캐싱 등 기본 제공
- **개발자 경험 향상**: 생산성 증가와 유지보수 용이성
- **모범 사례 적용**: 검증된 패턴과 코딩 규칙 제공

**주요 프레임워크 비교**:

| 프레임워크 | 특징 |
|------------|------|
| **Next.js** | 다양한 렌더링 전략(SSR, SSG, ISR), 이미지 최적화, 라우팅 내장 |
| **Remix** | 중첩 라우팅, 로더/액션 기반 데이터 처리, 오류 경계 시스템 |
| **Gatsby** | 정적 사이트 생성, GraphQL 통합, 풍부한 플러그인 생태계 |

```jsx
// Remix 예시: 데이터 로딩과 UI 통합
// routes/posts/$postId.tsx
import { useLoaderData } from "@remix-run/react";

// 서버에서 실행되는 데이터 로더
export function loader({ params }) {
  return fetchPost(params.postId);
}

// 클라이언트와 서버 모두에서 실행되는 컴포넌트
export default function Post() {
  const post = useLoaderData();
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}
```

## 서버 렌더링 선택 가이드

프로젝트의 요구사항과 환경에 따라 적절한 서버 렌더링 접근 방식을 선택해야 합니다.

**프로젝트 요구사항 분석**:
- SEO가 중요한 콘텐츠 중심 사이트 → SSR 필요
- 대시보드, 관리자 도구 등 내부용 앱 → CSR 충분할 수 있음
- 초기 로딩 성능이 중요한 경우 → SSR 유리

**환경에 따른 API 선택**:
- Node.js 환경 → `renderToPipeableStream`
- 브라우저 호환 서버리스 환경 → `renderToReadableStream`
- 간단한 앱/개념 증명 → `renderToString`

> [!WARNING]
> 직접 서버 렌더링을 구현할 경우 보안 취약점이 발생할 수 있습니다. 예를 들어, 부적절한 캐싱 전략은 한 사용자의 데이터가 다른 사용자에게 노출되는 문제를 일으킬 수 있습니다.

## 복습하기

### 1. 리액트 애플리케이션에서 서버 사이드 렌더링을 사용할 때 유리한 주요 장점은 무엇인가요?

서버 사이드 렌더링(SSR)의 주요 장점은 다음과 같습니다:

- **최초 의미 있는 페인트 시간 단축**: 서버에서 렌더링된 HTML을 즉시 제공함, JavaScript를 실행하기 전에 컨텐츠를 볼 수 있습니다.
- **SEO 개선**: 검색 엔진 크롤러가 완전히 렌더링된 HTML을 즉시 색인화 가능
- **웹 접근성 향상**
- **보안 강화**

### 2. 리액트에서 하이드레이션은 어떻게 작동하며 왜 중요한가요?

하이드레이션은 서버에서 생성된 정적 HTML에 이벤트 리스너와 JavaScript 기능을 추가하는 과정입니다.

**작동 방식**:
1. 서버에서 리액트 컴포넌트를 HTML로 렌더링하여 클라이언트에 전송합니다.
2. 클라이언트에서 HTML이 먼저 표시됩니다(빠른 초기 로딩).
3. JavaScript 번들이 로드되면 `hydrateRoot` 함수가 호출됩니다.
4. 리액트는 DOM을 순회하면서 이벤트 리스너를 연결하고 상태를 복원합니다.
5. 이제 정적 HTML이 완전한 인터랙티브 리액트 애플리케이션으로 변환됩니다.

**중요성**:
- 빠른 초기 로딩과 인터랙티브 기능의 이점을 모두 얻을 수 있습니다.
- 사용자는 JavaScript가 로드되기 전에도 콘텐츠를 볼 수 있습니다.
- SEO 이점을 유지하면서도 풍부한 인터랙티브 경험을 제공합니다.
- 서버와 클라이언트의 렌더링 결과가 일치해야 정상적으로 작동합니다.

### 3. 재개 가능성이란 무엇인가요? 하이드레이션보다 좋다고 주장하는 근거는 무엇인가요?

재개 가능성(Resumability)은 하이드레이션의 대안 방식입니다.

**개념**:
- 서버에서 애플리케이션의 HTML과 함께 인터랙티브 상태를 직렬화하여 클라이언트로 전송합니다.
- 클라이언트는 서버가 중단한 지점부터 작업을 바로 재개할 수 있습니다.
- 하이드레이션 과정(이벤트 리스너 연결 및 재렌더링)을 건너뛸 수 있습니다.

**하이드레이션보다 좋다고 주장하는 근거**:
- **인터랙티브 시간(TTI) 단축**
- **효율성 향상**: 전체 페이지를 다시 렌더링할 필요가 없어 불필요한 계산을 줄입니다.
- **네트워크 효율성**: 서버가 인터랙티브 상태를 직렬화하여 전송하므로 클라이언트에서의 상태 재구성이 필요 없습니다.
- **일관된 사용자 경험**: 하이드레이션 중에 발생할 수 있는 지연이나 불일치를 줄입니다.

다만, 재개 가능성은 하이드레이션보다 구현이 더 복잡하며, 이 복잡성이 얻는 성능 이점과 균형을 이루는지에 대해서는 리액트 커뮤니티에서 여전히 논의 중입니다.

### 4. 클라이언트 사이드 렌더링의 주요 장점과 약점은 무엇인가요?

**클라이언트 사이드 렌더링(CSR)의 장점**:
- **풍부한 상호작용**: 초기 로딩 후 페이지 전환이 빠르고 부드러우며, 복잡한 상호작용 구현이 용이합니다.
- **서버 부하 감소**: 정적 파일만 서비스하므로 서버 부하가 감소합니다.
- **개발 용이성**: 클라이언트와 서버를 분리하여 개발할 수 있어 관심사 분리가 명확합니다.
- **오프라인 기능**: 서비스 워커와 함께 사용하면 오프라인 기능을 구현할 수 있습니다.

**클라이언트 사이드 렌더링(CSR)의 약점**:
- **SEO 문제**: 검색 엔진 크롤러가 JavaScript를 실행하지 못하면 콘텐츠 색인화에 어려움이 있습니다.
- **초기 로딩 지연**: JavaScript를 다운로드, 파싱, 실행한 후에야 콘텐츠가 표시됩니다.
- **네트워크 폭포 현상**: 순차적인 리소스 로딩으로 인해 전체 로딩 시간이 길어집니다.
- **저성능 기기에서의 문제**: 구형 기기나 저사양 모바일 기기에서 JavaScript 실행 성능이 저하될 수 있습니다.
- **보안 취약점**: 서버 측 검증이 부족하여 CSRF와 같은 공격에 취약할 수 있습니다.

### 5. 리액트에서 renderToReadableStream과 renderToPipeableStream API의 주요 차이점은 무엇인가요?

두 API는 모두 리액트 컴포넌트를 스트리밍 방식으로 서버 렌더링하지만, 사용 환경과 구현 방식에 차이가 있습니다:

**renderToPipeableStream**:
- **사용 환경**: Node.js 서버 환경에 최적화되어 있습니다.
- **스트림 타입**: Node.js의 네이티브 스트림을 사용합니다.
- **API 스타일**: 이벤트 기반 API와 콜백을 사용합니다(`onShellReady`, `onAllReady` 등).
- **사용 예시**: Express, Koa 등의 Node.js 웹 서버와 함께 사용됩니다.
- **반환 값**: `pipe` 메서드를 포함한 객체를 반환하여 Node.js의 쓰기 가능 스트림에 연결됩니다.

**renderToReadableStream**:
- **사용 환경**: 웹 표준 환경(Cloudflare Workers, Deno 등)에 최적화되어 있습니다.
- **스트림 타입**: 웹 표준 Streams API를 사용합니다.
- **API 스타일**: 프라미스 기반 API를 사용합니다.
- **사용 예시**: Cloudflare Workers, Deno, 기타 웹 표준 환경에서 사용됩니다.
- **반환 값**: 웹 표준 `ReadableStream` 인스턴스를 프라미스로 반환합니다.

두 API 모두 Suspense를 지원하고 스트리밍 기능을 제공하지만, 사용 환경에 따라 적절한 API를 선택해야 합니다. 환경에 맞는 API를 선택함으로써 최적의 성능과 개발 경험을 얻을 수 있습니다.

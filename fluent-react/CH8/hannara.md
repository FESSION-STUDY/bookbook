# Chapter 8 프레임워크

## 1. 프레임워크의 개요 및 필요성

### 1.1 프레임워크가 필요한 이유

일반적인 클라이언트 렌더링 리액트 애플리케이션은 다음과 같은 문제가 있습니다:

1. **빈 페이지 초기 로딩**: 사용자는 자바스크립트가 로드되고 실행될 때까지 빈 페이지를 보게 됩니다. SEO에도 불리합니다.

2. **네트워크 폭포(Network Waterfall)**: 다음 순서로 실행되어 성능 저하를 일으킵니다:

   - 자바스크립트 다운로드, 파싱, 실행
   - 리액트 컴포넌트 렌더링 및 커밋
   - useEffect에서 데이터 페치 시작
   - 로딩 스피너 렌더링
   - 데이터 페치 완료
   - 데이터 렌더링 및 커밋

3. **클라이언트 기반 라우팅 문제**: 서버에 해당 경로의 파일이 없을 경우 404 오류가 발생합니다.

프레임워크는 이러한 문제를 해결하기 위해 서버 렌더링, 효율적인 라우팅, 데이터 페치 최적화 등의 기능을 제공합니다.

## 2. 프레임워크의 핵심 기능 구현

### 2.1 서버 렌더링

서버 렌더링을 구현하기 위해서는 Express와 같은 서버 패키지가 필요합니다:

```jsx
// server.js
import express from "express";
import { renderToString } from "react-dom/server";
import { List } from "./List";
import { Detail } from "./Detail";

const app = express();
app.use(express.static("./dist")); // 정적 파일 제공

const createLayout = (children) => `<html lang="en">
<head>
  <title>My page</title>
</head>
<body>
  ${children}
  <script src="/clientBundle.js"></script>
</body>
</html>`;

app.get("/", (req, res) => {
  res.setHeader("Content-Type", "text/html");
  res.end(createLayout(renderToString(<List />)));
});

app.get("/detail/:thingId", (req, res) => {
  res.setHeader("Content-Type", "text/html");
  res.end(createLayout(renderToString(<Detail thingId={req.params.thingId} />)));
});

app.listen(3000, () => {
  console.info("App is listening!");
});
```

이 코드는 서버에서 리액트 컴포넌트를 HTML 문자열로 렌더링하여 클라이언트에게 전송합니다.

### 2.2 파일 시스템 기반 라우팅

확장성 있는 라우팅을 위해 파일 시스템 기반 라우팅을 구현할 수 있습니다:

```jsx
// server.js
import express from "express";
import { join } from "path";
import { renderToString } from "react-dom/server";

const app = express();
app.use(express.static("./dist"));

const createLayout = (children) => `<html lang="en">
<head>
  <title>My page</title>
</head>
<body>
  ${children}
  <script src="/clientBundle.js"></script>
</body>
</html>`;

app.get("/:route", async (req, res) => {
  try {
    // pages 디렉토리에서 라우트 컴포넌트를 동적으로 가져옴
    const exportedStuff = await import(join(process.cwd(), "pages", req.params.route));
    const Page = exportedStuff.default;
    const props = req.query;

    res.setHeader("Content-Type", "text/html");
    res.end(createLayout(renderToString(<Page {...props} />)));
  } catch (error) {
    res.status(404).send("Page not found");
  }
});

app.listen(3000, () => {
  console.info("App is listening!");
});
```

이 방식은 Next.js와 같은 프레임워크에서 사용하는 규칙과 유사합니다.

### 2.3 데이터 페치 최적화

네트워크 폭포 문제를 해결하기 위해, 서버 렌더링 전에 데이터를 미리 가져오는 기능을 구현합니다:

```jsx
// pages/list.jsx
export const getData = async () => {
  return {
    props: {
      initialThings: await fetch("https://api.com/get-list").then((r) => r.json()),
    },
  };
};

export default function List({ initialThings }) {
  const [things, setThings] = useState(initialThings || []);
  const [requestState, setRequestState] = useState("initial");
  const [error, setError] = useState(null);

  // 초기 데이터가 없는 경우에만 페치
  useEffect(() => {
    if (initialThings) return;

    setRequestState("loading");
    fetch("https://api.com/get-list")
      .then((r) => r.json())
      .then(setThings)
      .then(() => {
        setRequestState("success");
      })
      .catch((e) => {
        setRequestState("error");
        setError(e);
      });
  }, [initialThings]);

  if (requestState === "loading") return <div>로딩 중...</div>;
  if (requestState === "error") return <div>에러: {error.message}</div>;

  return (
    <div>
      <ul>
        {things.map((thing) => (
          <li key={thing.id}>{thing.label}</li>
        ))}
      </ul>
    </div>
  );
}
```

서버 코드에 데이터 페치 로직을 추가합니다:

```jsx
app.get("/:route", async (req, res) => {
  const exportedStuff = await import(join(process.cwd(), "pages", req.params.route));
  const Page = exportedStuff.default;

  // 컴포넌트의 데이터를 가져오기
  const data = exportedStuff.getData ? await exportedStuff.getData() : { props: {} };
  const props = { ...req.query, ...data.props };

  res.setHeader("Content-Type", "text/html");
  res.end(createLayout(renderToString(<Page {...props} />)));
});
```

이렇게 구현한 간단한 프레임워크는 다음 기능을 제공합니다:

1. 파일당 경로별로 서버에서 데이터를 미리 가져오기
2. 전체 페이지를 HTML 문자열로 렌더링하기
3. 결과를 클라이언트에 전송하기

## 3. 프레임워크 사용의 장점

### 3.1 구조와 일관성

- 특정 구조와 패턴 제공으로 코드베이스 일관성 유지
- 신규 개발자의 빠른 적응 지원
- 제품과 기능 개발에 집중 가능

### 3.2 모범 사례

- 검증된 모범 사례 기본 제공
- 코드 품질 향상과 버그 감소
- 성능 최적화 권장으로 사용자 경험 개선

### 3.3 추상화

- 일반적인 작업에 대한 고수준 추상화 제공
- 깔끔하고 가독성 높은 코드 작성 가능
- 예시: Next.js의 useRouter 훅을 통한 라우터 접근

```jsx
// Next.js에서의 라우터 사용 예시
import { useRouter } from "next/router";

function MyComponent() {
  const router = useRouter();

  const handleNavigate = () => {
    router.push("/dashboard");
  };

  return <button onClick={handleNavigate}>대시보드로 이동</button>;
}
```

### 3.4 성능 최적화

- 코드 분할(Code Splitting)
- 서버 사이드 렌더링(SSR)
- 정적 사이트 생성(SSG)
- 자동 이미지 최적화

### 3.5 커뮤니티와 생태계

- 다양한 플러그인과 라이브러리
- 문제 해결책 찾기 용이
- 지속적인 발전과 지원

## 4. 프레임워크 사용 시 트레이드오프

### 4.1 학습 곡선

- 고유한 개념, API, 규칙 학습 필요
- 리액트 초보자의 경우 동시 학습의 부담

### 4.2 유연성과 규칙

- 강제된 구조가 제약으로 작용할 수 있음
- 특수한 요구사항이 프레임워크 모델과 충돌할 가능성

### 4.3 의존과 서약

- 애플리케이션의 운명이 프레임워크의 운명과 연결됨
- 프레임워크 유지보수 중단 또는 방향성 변경 시 어려움

### 4.4 추상화 오버헤드

- 내부 작동 방식 이해 어려움으로 디버깅 복잡성 증가
- 성능에 영향을 미칠 수 있는 오버헤드 발생

예시: Next.js의 서버 액션 사용 시 복잡성

```jsx
// Next.js의 서버 액션 예시
"use server";

import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

export async function addItem(formData) {
  const name = formData.get("name");

  // 데이터베이스에 아이템 추가
  await db.items.create({ data: { name } });

  // 캐시 무효화 및 리다이렉트
  revalidatePath("/items");
  redirect("/items");
}
```

복잡한 내부 구현을 이해하지 않고도 사용 가능하지만, 문제 발생 시 디버깅이 어려울 수 있습니다.

## 5. 인기 있는 리액트 프레임워크 비교

### 5.1 Remix와 Next.js 개요

#### Remix

- **특징**: 리액트와 웹 플랫폼 기능을 최대한 활용하는 현대적 웹 프레임워크
- **철학**: 웹 플랫폼의 기본 기능을 적극 활용하며 필요한 곳에만 자바스크립트 사용
- **강점**: 서버 렌더링, 직관적인 라우팅, 데이터 페치 및 조작 기능 제공

#### Next.js

- **특징**: Vercel이 개발한 인기 있는 리액트 프레임워크
- **철학**: 설정보다 규칙(convention over configuration) 원칙 추구
- **강점**: 서버 사이드 렌더링(SSR), 정적 생성(SSG), 애플리케이션 최적화 기능 제공
- **주요 기능**: App Router(v13+)를 통한 강력한 라우팅 시스템 제공

### 5.2 핵심 기능 비교

#### 서버 렌더링

**Remix**:

```jsx
// entry.server.tsx 예시
import { PassThrough } from "node:stream";
import { createReadableStreamFromReadable } from "@remix-run/node";
import { RemixServer } from "@remix-run/react";
import { renderToPipeableStream } from "react-dom/server";

export default function handleRequest(request, responseStatusCode, responseHeaders, remixContext) {
  return new Promise((resolve, reject) => {
    const { pipe, abort } = renderToPipeableStream(<RemixServer context={remixContext} url={request.url} />, {
      onShellReady() {
        const body = new PassThrough();
        const stream = createReadableStreamFromReadable(body);

        responseHeaders.set("Content-Type", "text/html");
        resolve(
          new Response(stream, {
            headers: responseHeaders,
            status: responseStatusCode,
          })
        );

        pipe(body);
      },
      // 에러 처리 생략...
    });
  });
}
```

**Next.js**:

```jsx
// app/page.tsx 예시
export default async function HomePage() {
  // 서버 컴포넌트에서 직접 데이터 페치
  const data = await fetch("https://api.example.com/data").then((r) => r.json());

  return (
    <main>
      <h1>Next.js 서버 컴포넌트 예시</h1>
      <div>
        {data.map((item) => (
          <div key={item.id}>{item.title}</div>
        ))}
      </div>
    </main>
  );
}
```

#### 라우팅

**Remix**:

```
routes/
  index.tsx         // 루트 경로 (/)
  about.tsx         // 정적 경로 (/about)
  users/
    index.tsx       // 사용자 목록 (/users)
    $userId.tsx     // 동적 경로 (/users/:userId)
```

**Next.js**:

```
app/
  page.tsx          // 루트 경로 (/)
  about/
    page.tsx        // 정적 경로 (/about)
  users/
    page.tsx        // 사용자 목록 (/users)
    [userId]/
      page.tsx      // 동적 경로 (/users/:userId)
  layout.tsx        // 전체 레이아웃
```

#### 데이터 페치

**Remix**:

```jsx
// routes/users.tsx
import { json } from "@remix-run/node";
import { useLoaderData } from "@remix-run/react";

export async function loader() {
  const users = await fetch("https://api.example.com/users").then((r) => r.json());
  return json(users);
}

export default function Users() {
  const users = useLoaderData();

  return (
    <div>
      <h1>사용자 목록</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Next.js**:

```jsx
// app/users/page.tsx
export default async function UsersPage() {
  const users = await fetch("https://api.example.com/users").then((r) => r.json());

  return (
    <div>
      <h1>사용자 목록</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

#### 데이터 조작

**Remix**:

```jsx
// routes/users/new.tsx
import { redirect } from "@remix-run/node";
import { Form } from "@remix-run/react";

export async function action({ request }) {
  const formData = await request.formData();
  const name = formData.get("name");

  await fetch("https://api.example.com/users", {
    method: "POST",
    body: JSON.stringify({ name }),
    headers: { "Content-Type": "application/json" },
  });

  return redirect("/users");
}

export default function NewUser() {
  return (
    <div>
      <h1>새 사용자 추가</h1>
      <Form method="post">
        <input type="text" name="name" required />
        <button type="submit">저장</button>
      </Form>
    </div>
  );
}
```

**Next.js**:

```jsx
// app/users/new/page.tsx
export default function NewUserPage() {
  return (
    <div>
      <h1>새 사용자 추가</h1>
      <form action={createUser}>
        <input type="text" name="name" required />
        <button type="submit">저장</button>
      </form>
    </div>
  );
}

// app/actions.js
("use server");

import { redirect } from "next/navigation";
import { revalidatePath } from "next/cache";

export async function createUser(formData) {
  const name = formData.get("name");

  await fetch("https://api.example.com/users", {
    method: "POST",
    body: JSON.stringify({ name }),
    headers: { "Content-Type": "application/json" },
  });

  revalidatePath("/users");
  redirect("/users");
}
```

## 6. 프레임워크 선택 기준

### 6.1 프로젝트 요구사항 파악하기

프레임워크 선택 전 고려해야 할 핵심 질문들:

- 프로젝트 규모 (소규모, 중간 규모, 대규모)
- 필요한 주요 기능과 특징
- 렌더링 방식 요구사항 (SSR, SSG 또는 조합)
- SEO 중요도
- 데이터 특성 (실시간 또는 동적 콘텐츠 여부)
- 빌드 프로세스 커스터마이징 필요성
- 성능과 속도의 중요도
- 개발자의 기술 숙련도
- 대상 사용자의 환경 (기기, 인터넷 속도 등)

### 6.2 Next.js vs Remix 비교

#### Next.js 특징

**학습 곡선**:

- 리액트의 최신 기능과 카나리아 릴리스를 활용
- 최신 기술 반영으로 학습 강도가 다소 높을 수 있음
- 명확한 가이드와 문서 제공으로 빠른 학습 가능

**유연성**:

- 정적 콘텐츠와 서버 렌더링 콘텐츠 간 유연한 전환 지원
- 클라이언트 애플리케이션도 지원하나 주력 영역은 아님
- 풍부한 플러그인과 통합 기능을 갖춘 생태계 보유

**성능**:

- 정적 생성, 서버 사이드 렌더링, 캐싱에 중점
- 네 종류의 목적별 캐시 메커니즘 제공
- 리액트 개발팀과의 긴밀한 협력 관계 구축

#### Remix 특징

**학습 곡선**:

- 웹의 기본 기능을 더 많이 활용
- 서버 컴포넌트 이전의 리액트 사용 방식과 유사해 학습이 비교적 용이

**직관성**:

- 웹 플랫폼의 기본 기능을 적극 활용
- 친숙하고 직관적이나, '마법' 같은 기능이 적어 제한적으로 느껴질 수 있음

**성능**:

- 라우트 기반 데이터 페치로 효율성 향상
- 낙관적 UI 업데이트와 점진적 향상 전략으로 사용자 경험 개선

### 6.3 트레이드오프와 선택 기준

**트레이드오프의 본질**:

- 편의성과 제어권 간의 균형
- 프레임워크는 여러 결정을 표준화해 개발자의 고민을 줄이나, 일부 제어권은 포기해야 함

**선택 기준**:

- 유연한 풀스택 프레임워크 필요 시: Next.js가 적합
- 서버 기반 점진적 향상과 웹 기본 원칙 중시 시: Remix가 적합
- 소규모 프로젝트에 적용해보며 직접 경험하는 것이 중요

### 6.4 개발자 경험과 성능 비교

**개발자 경험**:

- 두 프레임워크 모두 우수한 개발자 경험 제공
- Next.js: 정적 생성 중심, ISR로 빌드 시간 개선
- Remix: 서버 우선 아키텍처로 필요할 때마다 페이지 렌더링

**런타임 성능**:

- Next.js:

  ```jsx
  // Next.js ISR 예시
  export async function getStaticProps() {
    const posts = await fetch("https://api.example.com/posts").then((r) => r.json());

    return {
      props: { posts },
      revalidate: 60, // 60초마다 페이지 재생성
    };
  }
  ```

- Remix:
  ```jsx
  // Remix 캐싱 예시
  export function headers() {
    return {
      "Cache-Control": "max-age=300, s-maxage=3600",
    };
  }
  ```

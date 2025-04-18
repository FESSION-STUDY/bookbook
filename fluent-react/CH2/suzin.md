# chapter 2 JSX

### JSX (JavaScript Syntax eXtension, Javascript XML)

- 자바스크립트 확장 구문, 자바스크립트 XML (확장 마크업 언어)
- 마크업 언어란 데이터의 정의 규칙을 의미합니다.
  - `<h1></h1>` 이면 문서의 가장 중요한 제목 데이터를 정의
  - `<img></img>` 이면 문서에 첨부되는 이미지 소스 데이터를 정의
- React Element를 생성하는 템플릿 언어이자 자바스크립트의 기능을 포함합니다.
- React에서 JSX의 사용은 필수가 아닙니다.

<br />

```jsx
const MyElement = () => {
  return (
    <div>
      <h1>문서의 최상위 제목</h1>
      <p>감사합니다</p>
    </div>
  );
  // 컴포넌트를 여러줄로 작성할 때, 괄호를 넣어
  // JavaScript의 자동세미콜론삽입(ASI, Automatic Semicolon Insertion)을 방지
};
```

<br />

## 2.1 자바스크립트 XML?

### AJAX(비동기 자바스크립트와 XML)기술

기존 기술(XML, JavaScript)을 사용해 **페이지를 벗어나지 않은 채** 비 동기적으로 업데이트 되는 **대화형 웹 페이지**를 만들 수 있는 기술

- 페이지를 벗어나면 새로운 문서를 서버로부터 fetch하여 로드합니다.
- 상태가 변경될 때 마다 새 페이지를 읽어 들이는 것은 MPA (MultiPage Application), 기존 웹 동작 방식
- AJAX는 새로운 페이지의 로드 없이 새로운 데이터를 사용자에게 보여줄 수 있도록 한 기술입니다.
- 웹은 일방적으로 정보기 표기된 문서를 사용자에게 보여주는 형식에서 사용자와 Interaction 하는 방향으로 진화

- XMLHttpRequest(XHR) 도구로 HTTP(하이퍼 텍스트 전송 프로토콜)를 통해 non-blocking 비동기 요청을 수행할 수 있습니다.
  - non-blocking: 하던 작업의 종료를 기다리지 않고 다음 작업을 실행할 수 있는 것
  - XML로 응답하던 기존의 방식에서 JSON으로 응답하는 방식으로 변경되었습니다.
  - 응답 데이터 표기 방식의 변화와 웹 기술의 발전으로 현재는 fetch API를 보편적으로 사용합니다.

<br />

> [!NOTE] XHR vs fetch
>
> ### XMLHttpRequest vs Fetch API
>
> | **항목**                | **XMLHttpRequest (XHR)**                           | **Fetch API**                                                                       |
> | ----------------------- | -------------------------------------------------- | ----------------------------------------------------------------------------------- |
> | **구문 및 사용 편의성** | 복잡한 API와 코드                                  | 간결하고 직관적, async/await 사용 가능                                              |
> | **비동기 처리 방식**    | 콜백 기반                                          | Promise 기반, 체이닝 또는 async/await으로 간단한 비동기 처리                        |
> | **응답 처리**           | `responseText` 또는 `responseXML`로 수동 파싱 필요 | `response.json()`, `response.text()` 등으로 쉬운 응답 변환                          |
> | **기능 확장성**         | 요청 취소, 스트리밍 등 고급 기능 지원 제한적       | 요청 취소(`AbortController`), 스트리밍 응답 등 현대적 기능 지원                     |
> | **브라우저 호환성**     | 모든 브라우저 지원, 하지만 최신 기능 제한적        | 최신 브라우저 지원, IE는 폴리필 필요하지만 R.I.P                                    |
> | **에러 처리**           | 상태 코드(예: 404, 500) 명시적 확인 필요           | HTTP 에러(예: 404, 500)를 기본적으로 에러로 처리하지 않으며 `response.ok` 확인 필요 |

> [!NOTE] XML vs JSON
> | **항목** | **XML (Extensible Markup Language)** | **JSON (JavaScript Object Notation)** |
> |----------------------|--------------------------------------|---------------------------------------|
> | **데이터 구조 및 가독성** | 태그 기반으로 계층적 구조, 가독성 낮음, 데이터 크기 큼 | 키-값 쌍 기반으로 간결하고 가독성 높음, 데이터 크기 작음 |
> | **파싱 및 처리** | 복잡한 파싱 필요 (`DOMParser` 또는 별도 파서 사용) | 간단한 파싱 (`JSON.parse()`로 객체 변환), 대부분 언어에서 기본 지원 |
> | **사용 사례** | SOAP(Simple Object Access Protocol, 데이터 교환 프로토콜), RSS 피드, 레거시 시스템 | RESTful API, 웹/모바일 앱 데이터 교환, 현대 웹 개발 표준 |
> | **성능** | 태그로 인해 데이터 크기가 크고 파싱 속도 느림 | 데이터 크기 작고, 파싱 속도 빠름 |
> | **특징** | 엄격한 표준과 복잡한 구조로 보안 강화, 기업용 | 가볍고 유연하여 웹 API에 적합

<br />

### JSX to JavaScript

- Babel은 자바스크립트 컴파일러 입니다.
- JSX는 자바스크립트 코드로 변환되는 확장 구문일 뿐이므로, 결국 컴파일러에 의해 바닐라 자바스크립트 코드로 변환됩니다.

<br />

> [!TIP] 자바스크립트 컴파일러
> `swc` speedy web compiler : Rust로 작성된 고성능 컴파일러로 vite프로젝트에서 JSX를 빠르게 컴파일 합니다. 단일 스레드 기준 Babel의 20배라고 하네요 </br> > `tsc` 타입스크립트 컴파일러 : 타입스크립트에서 자바스크립트로 변환하는 컴파일러 `npx tsc '파일이름.ts'`

<br />

### JSX와 HTML의 차이점

- 중괄호(`{}`)를 사용한 자바스크립트 표현식 사용
  - 표현식은 '값'을 반환하는 식을 의미합니다.
- 속성을 카멜케이스로 작성

<br />

### JSX를 사용하지 않고 리액트 작성하기

우선 JSX로 컴포넌트를 작성해보겠습니다.

```jsx
const MyElement = () => {
  return (
    <div id="my">
      <h1>문서의 최상위 제목</h1>
      <p>감사합니다</p>
    </div>
  );
};
```

<br />

1. `createElement`를 사용한 예

```ts
import React from "react";

const MyElement = () => {
  return React.createElement(
    "div", //tag name
    { id: "my" }, //tag attribute
    //children
    React.createElement("h1", null, "문서의 최상위 제목"),
    React.createElement("p", null, "감사합니다")
  );
};
```

<br />

2. React/jsx-runtime 사용한 예

```ts
import { jsx as _jsx, jsxs as _jsxs } from "react/jsx-runtime";

const MyElement = () => {
  return _jsxs("div", {
    id: "my",
    children: [
      _jsx("h1", { children: "문서의 최상위 제목" }),
      _jsx("p", { children: "감사합니다" }),
    ],
  });
};
```

템플릿을 사용한 JSX는 vanilla Javascript르 작성한 react보다 가독성이 더 좋습니다.

<br />

## 2.2 JSX의 장점

- HTML을 주로 다루던 개발자에게 친숙합니다.
- 새로운 엘리먼트를 생성할 수 있는 템플릿 문자 '<', '>' 가 HTML의 문자열인 TextNode에 포함된다면 잘못 파싱될 위험이 있습니다. <br>
  이때 JSX 코드 컴파일 시 문자열을 파싱할 때 HTML문자열인 부등호 '&lt', '&gt' 로 변환하여 더 안전한 코드를 작성합니다. (**데이터 소독**)
- JSDoc 또는 propTypes를 사용해 타입 안정성을 향상할 수 있습니다
- 플랫폼에 영향 받지 않는 XML 기반이므로 확장성이 좋습니다.
- **컴포넌트 기반 아키텍처**
  - 재사용 가능한 독립된 모듈
  - 컴포넌트: 런타임 시점에서의 최소단위
  - 코드의 모듈화와 유지보수, 재사용성

<br />

### JSDoc과 PropTypes

| 항목      | JSDoc                        | PropTypes                 |
| --------- | ---------------------------- | ------------------------- |
| 체크 시점 | 개발 중 (IDE, Linter)        | 런타임                    |
| 문법      | 주석 기반                    | 코드 기반                 |
| 타입 강제 | 아님 (정적 분석 도구만 사용) | 런타임에서 콘솔 경고 발생 |
| 유연성    | 복잡한 구조 작성 쉬움        | 기본적인 타입 체크에 적합 |

> [!TIP] JSDoc
>
> ```js
> import React from "react";
>
> /**
>  * @typedef {{ name: string, age?: number }} MyProps
>  */
>
> /**
>  * @param {MyProps} props
>  * @returns {JSX.Element}
>  */
> const Greeting = (props) => {
>   return (
>     <div>
>       <h1>Hello, {props.name}!</h1>
>       {props.age && <p>You're {props.age} years old.</p>}
>     </div>
>   );
> };
> ```

<br />

> [!TIP] PropTypes
>
> ```js
> const Greeting = (props) => {
>   return (
>     <div>
>       <h1>Hello, {props.name}!</h1>
>       {props.age && <p>You're {props.age} years old.</p>}
>     </div>
>   );
> };
>
> Greeting.propTypes = {
>   name: PropTypes.string.isRequired,
>   age: PropTypes.number, // optional
> };
> ```

<br />

## 2.3 JSX의 약점

- 관심사의 혼합 > MVC 패턴을 주로 다루던 사람들은 이 방식을 보고 마냥 좋아하지만은 않았을 것 같습니다.
- 자바스크립트 호환성 부족: 인라인 블록을 지원하지 않음(인라인 표현식만)

JSX는 동적(런타임)이고 빠른 UI 작성에 도움을 줍니다. 자바스크립트의 기능은 모두 사용할 수 있고 리액트 컴포넌트 코드를 간단히 작성하고 읽기 쉽게 만들어 유지보수 할 수 있습니다.

<br />

## 2.4 내부 동작 (JSX to Vanilla JS)

### 2.4.1 코드는 어떻게 동작하나요?

- 코드 조각은 텍스트일 뿐: 텍스트를 사용하여 의미를 분석한다고 하면 **토큰화** 가 우선 시행
- 컴파일러를 사용하여 소스코드(텍스트)를 컴퓨터가 이해할 수 있는 코드(기계어)로 변환
- 컴파일러: 특정 규칙에 따라 소스코드(텍스트)를 구문트리로 변환하는 소프트웨어
- 컴파일 과정을 간략히 나타내면 아래와 같습니다.

  ```
  Text(Source Code) -> token -> AST -> Code(기계어, 어셈블리어, native code..)
  ```

  (1) 어휘 분석: tokenization (토큰화) <br />
  (2) 구문 분석: parsing (추상 구문 트리를 만드는 과정: 문자열에서 JSON 객체로 구성된 트리형 자료 구조로) <br />
  (3) 코드 생성: javascript 엔진에 의해 실행 가능한 기계어 <br />

- 컴파일러의 종류

  - interpreter: 바로 실행할 수 있는 기계어로 구성된 바이트 코드(표준 명령어)를 직접 실행
  - JIT compiler (Just In Time compiler): 자주 실행 되는 코드 최적화 추가

- 자바스크립트의 컴파일

  - 자바스크립트의 소스 코드가 바이트 코드 같은 중간 표현(IR Code 같은)로 변환
  - 프로그램 실행 시(런타임) 바이트 코드를 기계어로 동적 컴파일
  - 자주 실행되는 코드나 변수 종류를 런타임에 최적화

- 자바스크립트 런타임
  - 엔진과 연동하여 특정 환경에 맞는 콘텍스트 헬퍼와 기능을 제공
  - 브라우저 런타임: `window`, `document`
  - node.js 런타임: `global`

### 2.4.2 JSX로 자바스크립트 구문 확장하기

- JSX는 브라우저에서 직접 사용할 수 없습니다. 브라우저는 HTML, JavaScript, CSS 만 이해합니다.
- 자바스크립트 구문을 확장하려면 <br>
  1. 새로운 구문을 이해하는 다른 엔진을 쓰거나 <br>
  2. 엔진보다 앞서 새로운 구문을 처리하거나 (전처리) <br>
- JSX는 빌드 단계를 거쳐 순수한 JavaScript 코드가 됩니다. 이 과정을 트랜스 파일(transpile)이라 부릅니다.
- 이를 번들(bundle) 이라고 부릅니다.

> [!TIP] 트랜스 파일
> 컴파일의 부분 집합에 속하는 개념으로 <br>
> 추상화 수준이 비슷한 다른언어로 변환하는 과정 <br>
> 소스 대 소스 컴파일 이라고도 합니다. <br>

<br />

## 2.5 JSX 프라그마

- JSX 프라그마는 `<`로 시작하며, 사실 자바스크립트는 비교 연산자가 아닌 이상 인식하지 못 하는 문자이므로 SyntaxError가 발생합니다.
- JSX 에서는 JSX프라그마를 함수 호출로 변환합니다.
- 프라그마란: 언어 자체에 내포된 것 이상의 추가 정보를 컴파일러에게 제공할 때 사용하는 컴파일러 지시어 입니다. (매크로 같다고 느꼈어요)
  - 프라그마 예시: `use strict`, `use client`
- `<` > `React.createElement` 또는 `_jsxs`를 호출합니다.

<br>

```js
//프라그마 시그니처
function pragma(tag, props, ...children)
```

```js
  <MyComponent prop="속성값">컨텐츠</MyElement>
//위 jsx는 프라그마에 의해 아래를 호출합니다
  React.createElement(MyComponent, { prop: '속성값' }, '컨텐츠');
  //프라그마에 의한 호출__(Tag Name___, {   Tag Props  }, Children);
```

<br />

## 2.6 표현식

- JSX의 강력한 기능 중 하나는 엘리먼트 트리 내에서 코드를 실행할 수 있다는 것
- 단, 실행 가능한 코드는 반드시 **표현식**, 문장은 실행할 수 없습니다.
  - 값을 반환하지 않으면서 상태를 설정하는 부작용으로 간주되기 때문

<br />

## 대답하기

1. JSX란 무엇이며 장단점은 무엇일까요?

- 장점

  - 동적(런타임)이고 빠른 반응으로 대화형 웹 페이지 작성에 도움
  - JavaScript 확장 구문으로 템플릿을 사용하여 document의 노드를 표기할 수 있어 직관적
  - 데이터를 표기하는 XML(HTML)을 사용하여 플랫폼에 크게 영향 받지 않음
  - 다양한 타입 지원 시스템으로 안전하게 작성 가능
  - ⭐️ 컴포넌트 기반 아키텍처 (독립된 모듈, 재사용 가능성, 유지보수 편의성 등)

- 단점
  - 기존 MVC, MVVM 패턴에 비해 혼합된 관심사 (데이터와 뷰가 혼합)
  - 컴파일이 반드시 필요하고 번들링 작업이 필요하며 이에 대한 최적화 필요성
  - 표현식만 가능하므로 조금 아쉽다

2. JSX와 HTML의 차이점은 무엇인가요?

- JSX는 JavaScript 코드고 HTML은 마크업 언어
- 마크업 언어는 템플릿(태그)을 사용하여 문서와 데이터의 구조를 명시할 수 있는 언어

3. 텍스트 문자열은 어떻게 기계어가 되나요?

- 텍스트(소스코드)릁 어휘분석(토큰화 및 토큰 추출) `Lexical Analysis`
- 추출된 토큰의 의도 분석과 추상 구문트리 생성 (`Parsing`), 이 과정에서 컴파일 오류 발생 가능
- 컴파일 에러가 없다면, 의미 분석 `Semantic Analysis`
- 중간 코드 생성 (토큰의 의도를 담은 결과값)
- 인터프리터로 실행하며 컴파일러의 최적화 과정을 거침

4. JSX 표현식이란 무엇이며 어떤 좋은 점이 있나요?

- 값을 반환하는 식이며 동적으로 `map`, `filter` 등을 통해 DOM을 동적으로 생성하거나 백틱을 이용한 문자열, 다양한 평가식을 템플릿에 끼워 넣을 수 있는 것이 이점이라 생각해봅니다...

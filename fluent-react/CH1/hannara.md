
> [!note]
> 목차
> - 리액트가 왜 필요한가?
> - 리액트 이전의 세계
> 	- jQuery
> 	- Backbone
> 	- Knockout
> 	- Angular.js
> - React 등장
> 	- React의 핵심가치
> 	- React 출시
> 	- Flux 아키텍처
> 	- Flux 아키텍처 장점
> - 왜 필요한가?


## 리액트는 왜 필요한가
- *대규모* 애플리케이션에서 *'즉각적인' 업데이트*를 하기위해
	- 성능 : *리플로우 작업에서 성능 병목 현상이 자주 발생했음*
	- 신뢰성 : 상태를 추적해서 *일관되게 유지하기가 어려움*
	- 보안 : XSS, CSRF을 방지하기 위해서 사용자 입력 등 데이터에 문제가 될 만한 부분을 제거하는 작업이 필요


## 리액트 이전
리액트 이전에서는 애플리케이션이 *빠르고 즉각적이다 라는 느낌을 주면서 동시에 많은 사용자가 사용할 수 있어야하며, 안정적으로 작동할 방법*을 찾아야했다.

#### 상세 소개

> UI업데이트 상세 소개
> 버튼 예시
> 1. 클릭 전 
> 2. 클릭 후 대기 중
> 3. 클릭 후 성공
> 4. 클릭 후 실패

> UI를 업데이트해 상태 반영하는 단계
> 1. 환경에 따른 탐색 API 사용
> 2. 이벤트 리스너 추가해 이벤트 추적
> 3. 이벤트에 반응해 업데이트
> 4. 페이지 벗어날 때 이벤트 리스너 제거, 상태 제거

a. HTML엘리먼트 예시
버튼 참조 방법 (id속성 부여) -> getElementById를 사용한 참조 얻음 -> 이벤트 리스너 추가 -> 텍스트 업데이트 -> 클릭여부 추적 필요성 느낌 -> 상태 추가 -> data-linked 속성 추가해 클릭 여부 추적 , data-linked값에 따른 텍스트 컨텐츠 업데이트 -> DB에 저장이 필요할까? 라는 생각 -> 네트워크 통신을 통한 DB 에 상태값을 저장하자라고 생각 확장 -> XMLHttpRequest를 통한 통신을 하여 상태를 DB에 저장 -> (이때 잠시 jQuery소개) -> 데이터 통신에 실패한다면? -> 실패상태를 반영해서 텍스트를 업데이트해야함 -> data-failed 속성을 추가하는 방법으로 에러핸들링 -> 아직 처리가 되지 않을 때는? (pending) -> data-pending 속성도 추가해 대기 상태를 표현 -> 네트워크 요청 처리중 버튼을 비활성화하여 서버 과부하 방지 (디바운싱, 스로틀링 등장) 

> 위 과정을 리액트를 사용한다면 많은 버튼(인터렉션 이벤트)들을 많이 만들고 간결하고 효율적으로 UI를 업데이트 가능하며, 테스트하고, 재사용하고, 선언적으로 개발하며, 성능이 뛰어나고, 신뢰할 수 있고, 상태 예측이 가능하다라고 소개

이런 많은 에지 케이스를 고려하는 것은 번거롭다. (예를들어 이벤트 위임을 사용해 상위 document에 이벤트 리스너를 사용? 버튼에 이벤트 리스너 추가?)
추상화를 효과적으로 구현하기 위해서는 추가적인 **"도구"** 가 필요해졌다.

> "도구의" 필요성 강조
> - 시간이 지나면서 코드는 확장해야할 때 문제가 발생
> - 오류가 쉽게 발생
> 	- eventListener를 언제 정리? , 이벤트 위임은 언제? 어떻게?
> - 예측 불가능
> 	- id값이 실수로 여러 개면? , 엘리먼트가 없다면?, 엘리먼트가 생각한 것이 아니라면?, class를 사용한다면?
> - 비효율적
> 	- 렌더링할 때 DOM변형을 비용이 비싸다. 즉 최적화 단계를 거쳐야한다.

이런 어려움을 해결하기 위해 **도구**들이 등장 (Backbone, Knockout, Angular, jQuery)


## jQuery

![[스크린샷 2025-04-14 오전 1.22.09.png]]

위 예시는 UI에 데이터를 바인딩 시키고 페이지를 벗어나지 않은채 UI를 업데이트 시킵니다.
-> 이는 "부작용이 많은 방식"이라고 말하며 코드 어느 곳에서든 페이지 구조를 직접, 전역적으로 수정할 수 있게 됨 -> 구조화 되지 않은 DOM 조작 코드는 문제 추적과 원인 파악이 힘들어짐 -> 테스트를 할 때도 실제 브라우저 환경을 흉내 내야하는데 이때 많은 작업이 필요해짐 -> 테스트 힘듬 

이는 인터페이스의 소유주가 브라우저고 jQuery는 손님이라서 라고 소개한다.

jQuery의 단점
- 크기 및 로딩 시간 : 전체 라이브러리를 프로젝트에 통합하면 읽어야할 파일 크기가 커짐, 성능에 부정적인 영향
- 최신 부라우저와의 중복성: DOM조작이나 데이터 로딩 방식이 브라주저 발전을 통해서 간략해졌기때문에 오히려 jQuery의 문법이 비효율적
- 성능 고려 사항 : js 성능이 jQuery에서 제공하는 메서드보다 좋아짐


## Backbone
- 브라우저와 js간 "상태 불일치", 코드 재사용, 테스트 가능성을 해결할 방법이였다.
- MVC패턴을 자체 해석을 함

### MVC 패턴
소프트웨어 애플리케이션을 연결된 세 가지 "구성요소" 로 나누고, 내부 표현과 해당 정보가 *사용자에게 표시되거나 수용되는 방식을 요소별로 분리하는 디자인 철학*

![[스크린샷 2025-04-14 오후 6.19.52.png]]
- 뷰, 컨트롤러, 모델로 분리하는 디자인 철학
- 모델 : 애플리케이션 *데이터와 비즈니스 규칙 담당*
- 뷰 : *UI를 담당*, 모델이 제시한 데이터를 사용자에게 표시하고 사용자 명령을 컨트롤러에 전달함
	- 으로 모델이 데이터를 제공할 때까지 기다림 
	- 자체적으로 사용자 상호작용을 처리하지 않음 , *인접 컴포넌트인 컨트롤러에 위임*함
- 컨트롤러 : 모델과 뷰 사이의 인터페이스를 나타냄, 뷰에서 입력을 받아 처리(모델 업데이트가 필요한 입력또한 받습니다)하고 표시할 결과를 뷰에 반환

> 비즈니스 로직, UI, 사용자 입력 코드를 분리하는 것이 가장 큰 장점
> -> 모듈화되며 유지보수, 확장성, 테스트에 장점

하지만 웹 발전과 유저의 인터랙티브 요소 기대감이 높아지면서 MVC 패턴의 *한계점에 도달*


> 이런 문제를 해결하는 React 방법
- 복잡한 상호작용 및 *상태 관리* : 
	- 애플리케이션 규모가 커지면서 상태 변경과 UI 부분에 미치는 영향이 커짐에 따라서 컨트롤러가 늘어남 이때문에 영향을 관리하는 게 어려워짐
	- 화면에 표시되지 않는 뷰를 제어하는 컨트롤러도 많아지고 이것들이 경계가 명확하지 않아서 충돌을 일으킬때도 있음
	- *=>* 가상 DOM을 사용하는 리액트는 UI컴포넌트가 prop을 받고, 입력에 따라 출력하는 함수가 같다고 가정해서 어려움을 해결한다
- 양방향 데이터 바인딩 : 
	- 양방향 데이터 바인딩을 사용하는 라이브러리들은 *사이드 이펙트*가 일으킬 수 있음, 
	- 데이터 소유권 문제가 제대로 처리되지 않을 수 있으며 관심사 분리가 되지 않았을 때 발생한 사이드 이펙트가 있음
	- 리액트는 *'단방향 데이터 흐름'* 을 사용함
- 강한 결합 : 
	- MVC 패턴에서 모델, 뷰, 컨트롤러가 강하게 결합되어있다면 다른 요소에 영향을 주지 않고 리팩토링 하는 것이 힘듬 
	- 리액트는 컴포넌트 기반 모델로 모듈화 되고 분리된 접근 방식을 채택함

이것만 알아두자 : 
- **모델**은 데이터의 출처
- **뷰**는 데이터를 소비하고 표현하는 UI

![[스크린샷 2025-04-14 오전 1.55.28.png]]
![[스크린샷 2025-04-14 오전 1.55.42.png]]

- Backbone자체에는 render에 대한 실제 구현이 포함되지 않았다.
- jQuery를 통해서 DOM을 수동 변경하는 것과 핸들바와 같은 템플릿을 사용함
- 이벤트에 반응해 UI를 업데이트하는 대화형 버튼을 편하게 만들 수 있다.
- 로직을 한 덩어리로 묶어서 구조화된 방식으로 작업을 수행함

### backbone 단점
- 장황한 코드 및 보일러플레이트 코드 : 개발자가 작성해야하는 보일러플레이트 코드 양이 컸다.
- 양방향 데이터 바인딩 부족 : 양방향 바인딩을 기본으로 제공하지 않음 =>
	- 데이터가 변경되면 DOM이 자동으로 업데이트 되지 않는다.
- 이벤트 중심 아키텍처 : 모델 데이터를 업데이트하면 "전체 애플리케이션에서 수많은 이벤트가 발생"
	- 이는 데이터 변경 하나가 전체에 미치는 영향을 알기 어렵고 유지보수도 힘듬
- 조합성 부족 : 뷰를 쉽게 중첩할 수 있는 기능이 내장되지 않아서 복잡한 UI를 구성하기 힘듬



## Knockout
옵저버블, 바인딩을 제공하는 라이브러리
상태가 변경될 때마다 옵저버블과 바인딩을 활용한 의존성 추적.
- 상태 변화에 따라 업데이트하는 최초의 반응형 자바스크립트 라이브러리
- **옵저버블** : 데이터의 출처 => Model 비슷
- **바인딩**: 데이터를 소비, 렌더링 하는 UI => View 비슷
- **MVVM** 디자인 패턴 사용
![[스크린샷 2025-04-14 오후 1.51.36.png]]

최신 UI플랫폼에 맞춘 MVC패턴을 발전 

- 모델 : 
	- 데이터, 비즈니스 로직
	- 데이터 검색, 저장, 처리
	- DB, 서비스, 다른 출처의 데이터와 통신
	- **뷰와 뷰 모델을 인식하지 못함**
- 뷰 : 
	- UI를 표시
	- 사용자에게 정보 표시, 사용자 입력 받음
	- 수동적이며 애플리케이션 로직을 포함하지 않음, 
	- 데이터 바인딩 메커니즘을 통해 자동으로 변경사항을 반영, **선언적으로 뷰 모델에 바인딩**
- 뷰 모델 :
	- 모델과 뷰 사이의 다리 역할
	- 뷰에 바인딩할 때 데이터와 명령을 제공,
		- 대부분 표시할 수 있는 형식
	- 명령 패텅을 통해서 사용자 입력을 처리
	- 모델의 데이터를 **뷰에서 쉽게 표시할 수 있는 형식으로 변환**
	- 어떤 **뷰가 자신을 사용하는지 인식하지 못함**

#### 장점
- 테스트 가능성
	- 분리한 뷰, 뷰 모델로 UI를 포함하지 않고 로직에 대한 단위 테스트를 쉽게
- 재사용성
	- 뷰 모델은 재사용 가능
- 유지 보수성
	- 코드 관리, 확장, 리팩토링 쉬워짐
- 데이터 바인딩
	- UI 업데이트에 필요한 보일러플레이트 코드 양 줄임


### MVC vs MVVM
![[스크린샷 2025-04-14 오후 2.01.48.png]]

- 결합과 바인딩이 큰 차이점
	- 모델과 뷰 사이에 컨트롤러가 없을 때 데이터 소유권이 더 명확해지고 사용자에게 더 가까이있다.
	- "리액트는 단방향 데이터 흐름을 채용해 MVVM을 개선, 데이터 소유권이 더욱 제한적인 상태로 상태가 필요한 컴포넌트가 상태를 소유하게 함"


### 옵저버블, 바인딩을 통해 왜 리액트가 필요해졌는지
Knockout에서 "뷰 모델"은 키와 값을 포함하는 **"객체"** 로 data-bind 속성을 사용해 요소에 바인딩
- 즉, 컴포넌트나 템플릿이 없고 뷰 모델과 브라우저 요소에 바인딩하는 방법만 존재

createViewModel 함수로 뷰 모델 생성 -> applyBindings를 사용해 브라우저 환경에 연결 -> applyBindings 함수는 뷰 모델을 가져와 브라우저에서 data-bind속성을 가진 요소를 찾는다 -> Knockout은 이 요소들을 뷰 모델에 바인딩하는 데 사용 -> createViewModel 함수를 사용한 "뷰 모델"에 HTML요소를 바인딩, 사이트와 상호작용 -> 이때 많은 보일러플레이트 코드가 필요해짐 -> 결국 뚱뚱한 뷰 모델이 되어서 테스트하기 어려워지고 파악하기 어려워짐 -> 그래도 많이 이점이 많아 많이 사용했다.

## Angular JS 등장
- 양방향 데이터 바인딩
	- 모델이 변경되면 UI도 변경 사항을 반영해 자동으로 업데이트, 반대도 가능
	- jQuery와 대조적(jQuery는 개발자가 DOM을 수동조작 -> 변경사항 반영 -> 입력 캡처 후 데이터 업데이트)
	- 예시) no-model 지시자가 입력 필드 값을 변수에 바인딩 -> 입력 필드에 입력하면 모델에 name이 업데이트
- 모듈식 아키텍처
	- 애플리케이션의 구성 요소를 "논리적"으로 분리할 수있는 모듈식 아키텍처를 도입
	- 기능을 캡슐화해서 독립적으로 개발, 테스트, 유지 관리
	- 예시) 각 use... 변수는 모듈에 의존하고 이것들은 js파일에 포함될 수도 있고 별도로 작성될 수 있음
	- 이런 형태로 **의존성 주입이라는 패턴**을 사용함
- 의존성 주입
	- 객체가 의존성을 직접 만들지 않고 전달받는 디자인 패턴
	- 모듈과 구성 요소를 만들고 관리하는 것
	- 예시를 보면 컨트롤러는 서비스를 어떻게 만드는지 몰라도 되며 서비스를 선언해두기만 해도 서비스를 생성, 주입하는 것은 Angular가 처리함

### Backbone, Knockout비교, Angular등장

Backbone과 Knockout은 Angular보다 제약이 적었고, 보일러플레이트코드가 많이 필요했다.

- AngularJS는 양방향 데이터 바인딩 + 의존성 주입을 통해 구조화된 개발을 했고 많은 규칙과 제약으로 작업 속도를 높힘
- 양방향 바인딩을 통해 DOM변형을 처리할 수 있음
- 기본으로 제공한 의존성 주입, 모듈식 아키텍처 등을 이 두 라이브러리는 갖추지 못함
- SPA 구축보다 더 포괄적인 솔루션을 제공

### Angular.js의 트레이드오프

웹 발전을 통한 Angular의 한계점과 단점
- 성능
	- 변경 사항 감지의 핵심기능(다이제스트 주기)가 업데이트 지연과 UI응답 지연의 원인이 될 수 잇음
	- 양방향 데이터 바인딩이 혁신적이였지만 성능문제를 발생함
- 복잡성
	- 지시자, 컨트롤러, 서비스, 의존성 주입, 팩토리 .. 등등 도입했는데 이것이 오히려 진입장벽이되어버림
	- 어떤 기능을 선택함의 결정장애가 발생함 ㅎ
- 2.X 버전으로 마이그레이션 문제
	- 2버전 발표할 때 이전 버전과 호환안되고 Dart나 TS로 새로 작성해야했음
- 복잡한 템플릿 문법
	- 복잡한 JS 표현식을 허용해서 HTML안에 비즈니스 로직 + 프레젠테이션이 섞임
	- 코드 해독이 어려움 => 유지보수에 문제
	- 디버깅 어려움
- 타입 안정성 X
	- 정적 유형 검사기에 작동안해서 초기에 오류를 잡기 어려움
	- 큰 단점
- $scope모델 어려움
	- $scope 객체는 뷰와 컨트롤러 사이의 다리역할을 했는데 동작이 직관적이지 못했고, 예측가능하지 못함 => 데이터 바인딩때 동작이 달라서 혼동되었음
	- 중첩된 컨트롤러는 $scope 상위 스코프 속성을 상속할 수 있는데 이때 처음 정의되거나 수정되면 추적하기 어려웠음
> 리액트 쓰면 필요한 컴포넌트와 상태를 같은 장소에 둬서 이런 문제 피할 수 있음 ^0^
- 제한적인 개발도구
	- 성능 프로파일링을 위해 다양한 개발도구가 없었음
> 리액트는 devTools가 잘되어있는데 ^0^


## 리액트 등장
핵심 아이디어 : **컴포넌트 기반 아키텍처**

#### 앵귤러와 비교
- 앵귤러가 뷰를 모델에 바인딩하기 위해 지시자를 사용했다면, 리액트는 JSX + 컴포넌트 모델 도입
- 양방향 바인딩은 대규모 애플리케이션에서 잠재적 성능 문제 등 단점이 있었고, 이것을 보완하기 위해 단방향 데이터 흐름 패턴을 채택 -> 잘 제어하고 변화흐름을 더 잘 이해함
- 가상 DOM을 채택해 직접적인 DOM 조작을 최소화 -> 성능 개선

### 리액트의 핵심 가치
- 선언적 코드, 명령형 코드 :
	- 리액트는 **DOM에 대한 선언적 추상화를 제공**하고 우리가 **보고자 하는 것을 코드로 표현** 어떻게 할지는 리액트가 담당
	- 즉, 우리가 무엇을 완료할 것인지를 기술 -> 목표를 어떻게 하는 것은 리액트가
- 가상 DOM :
	- 실제 DOM을 자바스크립트 객체로 표현하는 프로그래밍 개념, 실제 DOM트리를 가볍게 표현함
	- 실제 DOM을 조작하지 않고 가상 DOM을 통해 UI를 업데이트할 수 있다는 사실
	- **가상 DOM을 활용해 컴포넌트 변경사항 추적 -> 필요할 때(변경사항이 있을 때) 컴포넌트를 리렌더링**
	- 실제 DOM트리와 일치하도록 가상 DOM을 생성 -> 업데이트하고 모든 변경사항은 **재조정 과정**을 통해 실제 DOM에 적용
		- 실제 DOM트리를 반영하는 가상 DOM 생성 
			- 컴포넌트가 첫 렌더링 될 때 리액트가 실제 DOM트리를 반영하는 가상 DOM트리 생성->
			- 예) div element한 개에 button과 p 엘리먼트가 포함
			- 변경사항이 생긴다면 업데이트 상태를 반영하는 새로운 가상 DOM을 생성
		- 재조정을 통한 *이전 가상DOM*과 *새로운 가상DOM* **비교하는 과정**
			- 새로운 가상 DOM트리를 계산 -> 재조정 프로세스 수행하여 이전 트리와 차이점을 파악 -> 실제 DOM에 수행해야할 효과적인 최소한의 업데이트 동작 계산 -> 변경사항 실제 DOM 업데이트
- 컴포넌트 모델 :
	- 모듈성 향상, 디버깅 쉬움, 코드 재사용성 높음, 효율 높음
- 불변 상태 :
	- 애플리케이션 *상태는 불변하는 값의 집합*이라는 패러다임을 강조
	- *상태 업데이트는 독립된 스냅샷과 메모리 참조*로 취급
	- 불변하는 값으로 *예측 가능한 UI 개발* 가능
		- 불변함을 강조해 특정 시점에 특정 상태를 반영하도록 보장
		- 상태 변경 시 새로운 상태를 표현하는 객체를 반환 -> 변경 사항 추적 용이 , 디버깅 쉬움 -> 이해하기 쉬움 -> 상태 전환은 독립적이고 간섭하지 않는다.
		- 상태의 불변성 덕분에 다른 상태 업데이트 시 손상 위험이 없이 안전한 트랜잭션을 모아 적용 가능


## 플럭스 아키텍처

클라이언트의 웹 애플리케이션 구축을 위한 아키텍처 디자인 패턴 "페이스북"

**단방향 데이터 흐름**이라는 특징이 존재

![[스크린샷 2025-04-12 오후 3.08.54.png]]

### 액션
- 새로운 데이터와 액션 종류를 식별하는 속성을 포함하는 단순한 객체
- UI, data response, input 등을 표현
- 디스패처를 통해서 스토어로 보내짐

### 디스패처
- 액션을 받아서 스토어로 보내진다.
- 모든 스토어는 디스패처에 자신과 콜백을 등록 , 콜백 목록을 관리하는 것도 디스패처


### 뷰
- 리액트 컴포넌트
- 스토어에서 변경 이벤트를 받고 데이터가 변경되면 스스로 업데이트를 진행
- 시스템 상태를 업데이트하는 새로운 액션을 생성해서 단방향 데이터 흐름을 시작하기도 함


## 플럭스 장점
- 단일 정보 출처
	- 애플리케이션의 상태에 대한 단일 정보 출처가 스토어에 저장
	- 동작 예측 가능, 복잡성 제거, 상태 불일치 문제를 방지
- 테스트 가능성
	- 관심사 분리(액션, 디스패처, 스토어, 뷰)를 통해 독립적인 단위 테스트가 가능
- 관심사 분리
	- 모듈화 가능, 유지보수 쉽


## 왜 리액트?
- 화면에 나타내고자 하는 의도를 "선언적"으로 표현
	- 어떻게 렌더링해야하나를 고민하지 않아도 리액트가 DOM에 점진적으로 업데이트를 수행
	- 관심사를 분리해서 재사용성 높힘
	- 넓은 생태계



## 복습질문
1. 리액트를 만들게 된 동기 :
		리액트 이전에 문제가 되던 상태 관리, 렌더링 최적화, 개발 시 어려움 등을 해결하기 위해
		대규모 애플리케이션에서 즉각적인' 업데이트를 하기위한 필요성
2. 리액트가 MVC와 MVVM같은 이전 패턴보다 개선된 점
   - 리액트는 UI를 선언적으로 표현하여 상태가 바뀌면 예측가능한 UI를 생성하고 테스트도 쉬워짐
   - 단방향 데이터 흐름을 채용하여 기존의 상태 추적 어려움을 해결하고 예상치 못한 사이드 이펙트를 해결하게됨
	   - 상위 -> 하위로 흐르는 데이터 흐름으로 예측가능, 추적 가능
   - 컴포넌트 기반 아키텍처 채택으로 기존의 역할 중심으로 나뉘어져 관리하기 어려운 문제를 해결하고 유지보수, 재사용성을 높임
   
3. 플럭스 아키텍처의 특징
   - 단방향 데이터 흐름을 가진 아키텍쳐
	   - "뷰 -> 액션 -> 디스패처 -> 스토어 -> 뷰" 순으로 데이터 흐름을 통한 상태 변경 추적이 쉬움
	   - 상태 변경이 중앙(스토어)에 집중되어 유지보수하기 쉬움
	   - 독립적으로 나뉘어있어서 테스트하기도 편함
	   - 역할이 분리되어있어서 모듈화하기 쉬움
   
4. 선언적 프로그래밍의 추상화의 장점
	- 선언적 프로그래밍 : *무엇을 할지 설명*하는 방식   
	- 내부 구조, 구현까지 몰르고 개발잔느 무슨 기능이 필요한지만 고민할 수 있다.
	- 가독성, 재사용성, 테스트 용이성 향상
   
5. 효율적인 UI 업데이트를 위한 가상 DOM의 역할은?
	- 가상 DOM : 실제 DOM 트리를 메모리상에 복사한 가벼운 자바스크립트 객체
	- 실제 DOM에 변경하지 않고 가상 DOM에서 먼저 변경을 계산하고 변경된 최소한의 부분만 DOM에 반영
	- 이로써 DOM조작을 줄이고 , 상태변화에 따른 UI를 빠르게 반영하는 것
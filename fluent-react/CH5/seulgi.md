# CH5 자주 묻는 질문과 유용한 패턴

> 메모화, 지연로딩, 성능을 강화하기 위한 방법들
> 

# 5.1 React.memo를 사용한 메모화

- 메모화 : 이전에 계산된 결과를 캐싱해서 함수의 성능을 최적화하는 기법
    - 함수의 순수성 필요 : 함수가 주어진 입력에 대해 동일한 출력을 예측 가능하게 반환해야함
- [React.memo]([https://ko.react.dev/reference/react/memo](https://ko.react.dev/reference/react/memo))
    - 반환하는 새 컴포넌트는 프롭이 변경되었을 때만 다리 렌더링함

## 5.1.1 React.memo에 능숙해지기

- 얼마나 많이, 얼마나 자주 메모화 해야할까?
- 모든 컴포넌트를 메모화하면 전체 성능이 더 좋아질까??

## 5.1.2 리렌더링되는 메모화된 컴포넌트

- 프롭에 얕은 비교를 수행해 프롭으 ㅣ변경 여부를 확인함
    - 얕은 비교 → 스칼라타입 매우 정확하게 비교 가능 / 스칼라가아닌 타입은 정확 X
        - 스칼라(원시타입)
            - 문자열, 불리언, Symbol, BigInt, undefined, null
        - 참조타입
            - 객체, 배열
            - 값이 아닌 메모리 참조를 기준으로 비교
            - 배열이 props 로 내려온다면 → useMemo 훅으로 배열을 메모화 하면 문제가 해결됨
            - 프롭에 함수가 있다면 → 함수를 참조로 비교하기 때문에 useCallback 훅으로 문제 해결

## 5.1.3 강제가 아닌 권장 사항

- 컴포넌트 트리의 변경이나 애플리케이션의 전역 상태 변경이 발생하면, 리액트가 메모화된 컴포넌트를 다시 렌더링할 수 있음

```tsx
// 메모화된 컴포넌트를 나타내는 새 객체를 반환함!
function memo(type, compare){
	return {
		$$typeof: REACT_MEMO_TYPE,
		type,
		compare: compare === undefind ? null : compare
	}
}
```

- 메모화된 컴포넌트를 업데이트해야하는지 아니면 성능 최적화를 위해 업데이트를 건너뛸 수 있는지 판단하는 역할을 함

# 5.2 useMemo를 사용한 메모화

- React.memo vs useMemo : 메모화를 위한 도구지만 용도가 매우 다름
    - React.memo : 전체 컴포넌트를 메모화
    - useMemo : 컴포넌트 내부의 특정 계산을 메모화해 비용이 많이 드는 재계산을 피하고 결과에 대한 일관된 참조를 유지함
    - 렌더링 할 때 목록으로 정렬 → 목록에 1,000,000명이 포함되어있다고 가정하면 → input 횟수 만큼 리렌더링 됨 → 1,000,000 & log(1,000,000) 연산 수행 → useMemo로 최적화

## 5.2.1 useMemo의 나쁜 사례

- 계산 비용이 많이 드는 연산을 메모화
- 객체나 배열에 대한 안정적인 참조를 유지하는데 유용
- 스칼라 값은 useMemo를 사용 X
    - 값을 비교할 때 참조가 아닌 값 그 자체를 비교하기에
    - 만약 useMemo를 사용한다면 메모리를 더 쓰게 되는 격
        
        ```tsx
        const MyComponent = () =>{
        	const [count, setCount] = useState(0)
        	const doubledCount = useMemo(()=> count*2, [count])
        	
        	return(
        		<div>
        			<p> 카운트 : {count}</p>
        			<p> 2x 카운트 : {doubledCount}</p>
        			<button onClick={()=>setCount((oldCount)=> oldCout + 1)}>증가</button>
        		</div>
        	)
        }
        ```
        
        - onClick 핸들러 부분은 useCallback 으로 메모화 해야할까?
            - 아니오!
            - 리액트 함수 컴포넌트가 아니므로, 이 함수를 메모화해서 얻는 이점이 없음
            - 함수 프롭의 변경에 의해서는 리렌더링되지 않음
            - 내장 컴포넌트에 대한 가상 DOM 비교는 함수 프롭의 동일성을 기반으로 함
            - 불필요한 함수 생성은 가비지 컬렉션을 자주 유발하며 업데이트가 매우 잦은 경우에는 성능 문제로 이루어질 수 있음
            - useCallback이 유용한 예시는?
                - 자주 리렌더링 할 가능성이 있는 컴포넌트가 있고 하위 컴포넌트에 콜백을 전달 할 때
                
                ```tsx
                /**
                	프롭이 변경될 때만 다시 렌더링 됨
                */
                const ExpensiveComponent = React.memo(({onButtonClick})=>{
                	const now = performance.now();
                	while(performance.now() - now < 1000){
                	// 인위적인 지연 -- 1000 밀리초 멈춤
                	}
                	return <button onClick+{onButtonClick}>Click Me</button>
                })
                
                /** 
                	count, otherState 두개의 상태 있음
                	incrementCount : count를 업데이트하는 메모화된 콜백
                	doSomethingElse : otherState를 변경하지만 useCallback 이나 
                										다른 자식에게 전달되지 않으므로 메모화할 필요 X
                */
                const MyComponent = ()=>{
                	const [count, setCount] = useState(0);
                	const [otherState, setOtherState] = useState(0);
                	
                	const incrementCount = useCallback(()=>{
                		setCount((prev)=>prev+1)
                	},[setCount])
                	
                	const doSomethingElse =()=>{
                		setOtherState((s)=>s+1)
                	}
                	
                	return(
                		<div>
                		<p>카운트 : {count}</p>
                		<ExpensiveComponent onButtonClick={incrementCount} />
                		<button onClick={doSomethingElse}>Do Something Else</button>
                		</div>
                	)
                }
                ```
                
        - 메모리 할당 외에는 계산이 필요없는 스칼라 값이면 메모이제이션하지않음!

## 5.2.2 모두 잊고 포겟하세요

- React Forget : 리액트 컴파일러
- https://ko.react.dev/blog/2022/06/15/react-labs-what-we-have-been-working-on-june-2022#react-compiler
- https://ko.react.dev/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-optimizing-compiler

# 5.3 지연로딩

- 페이지가 완전히 읽어 들여질 때까지 초기 실행에 필수적이지 않은 자바스크립트의 로딩을 미뤄두기
- React.lazy
    - 필요한 컴포넌트만 읽어 들이는 기술
- Suspense
    - 프로미스가 해결되기 전까지 사이드바 다운로드가 완료되기 전까지 폴백 컴포넌트를 표시할 수 있는 컴포넌트

## 5.3.1 Suspense를 통한 더 나은 UI제어

- try/catch 블록처럼 작동함

# 5.4 useState와 useReducer

- useState
    - 단일 상태를 관리하는데 더 적합
    - useReducer의 추상화 모델
- useReducer
    - 복잡한 상태를 관리하는데 더 적합
- useState에서도 useReducer와 동일한 작업을 수행할 수 있으니 더 간단한 useState만 사용하면 되지 않을까?
    - useReducer 사용 이점
        - 상태 업데이트 로직을 컴포넌트에서 분리하여 단일 책임 원칙 따름
        - 상태와 상태 변경 방식은 항상 명시적으로 useReducer와 함께 사용
        - 이벤트 소스 모델
            - 애플리케이션에서 발생하는 이벤트를 모델링 해서 일종의 진단 로그를 추적하는 디버깅에 사용할 수 있음

## 5.4.1 Immer와 편의성

- https://immerjs.github.io/immer/
- 복잡한 상태 관리를 처리할 때 유용
- 변경 가능한 드래프트 상태로 작업 → 한 번 생성된 상태는 변경 불가
- useImmerReducer
    
    ```tsx
    import produce from 'immer';
    import {useState} from 'react';
    
    const MyComponent = ()=>{
    	const [state, setState] = setState({
      user: {
        name: '슬기',
        age: 32
      }
    });
    	
    	const updateName = (newName)=>{
    		setState(
    			produce((draft)=>{
    				draft.user.name = newName;
    			})
    		)
    	}
    }
    ```
    
    원래였다면, produce를 사용하지 않는다면 setState({...state, user:{...state.user, name: 슬기2}})
    

# 5.5강력한 패턴

- 디자인 패턴이 중요한 이유
    - 재사용성
    - 표준화
    - 유지 보수성
    - 효율성

## 5.5.1 프레젠테이션/컨테이너 컴포넌트

- 프레젠테이션 컴포넌트 == UI 렌더링
    - 시각적 테스트 가능
- 컨테이너 컴포넌트 == UI 상태 처리
    - 단위 테스트 가능
- 단일 책임 원칙
- 훅이 도입되고 대체됨

## 5.5.2 고차 컴포넌트

- 고차 함수 ?
    - 하나 이상의 함수를 인수로 취함 or 함수를 결과로 반환함
- 고차 컴포넌트 ?
    - 기본적으로 다른 컴포넌트를 인수로 받아 두 컴포넌트의 합성결과인 새로운 컴포넌트를 반환함
    - 여러 컴포넌트에서 공유하는 동작을 반복 작성하고싶지않을때
- 훅도 비슷한 이점을 제공하면서 고차컴포넌트 대신 훅을 많이 사용함
    - 고차 컴포넌트와 커스텀 Hook은 언제 어떤 기준으로 선택해야 할까?
    - 커스텀 훅으로 로직 추출 → 고차 컴포넌트로 렌더링 제어
    
    > **Hook은 "판단 기준"을 제공하고, HOC는 "구조적으로 어떻게 처리할지"를 정리해주는 것**
    > 
    
    ```tsx
    function useAuth() {
      const user = useContext(AuthContext);
      return user?.isLoggedIn ?? false;
    }
    
    function withAuth(Component) {
      return function WrappedComponent(props) {
        const isLoggedIn = useAuth();
        if (!isLoggedIn) {
          return <LoginPage />;
        }
        return <Component {...props} />;
      };
    }
    ```
    

## 5.5.3 렌더 프롭

- 컴포넌트으 ㅣ상태를 전달받는 함수를 프롭으로 사용하는 것 → 현재는 훅으로 대체, 거의 사용 X

```tsx
<WindowSize
	render = {({width, height})=>(
		<div>창 크기: {width}x{height}px</div>
	)}
/>
```

## 5.5.4 제어 프롭

- 제어 컴포넌트 개념 확장한 것
    - 내부에 자체 상태를 유지하지 않는 컴포넌트
        - 현잿값 : 부모 컴포넌트에서 전달한 프롭에 의해 결정, 부모 컴포넌트가 단일정보출처의 역할을 함
        
        ```tsx
        function TextInput({ value, onChange }) {
          return <input value={value} onChange={onChange} />;
        }
        
        // 사용
        <TextInput value={inputValue} onChange={(e) => setInputValue(e.target.value)} />
        
        ```
        
        - 반대로, 비제어 컴포넌트는?
            - `value`, `onChange`를 전달받지 않음
            - 대신 `ref`를 통해 값을 직접 접근
            - 상태는 DOM에 저장되고, React는 이를 **직접 제어하지 않음**
            
            ```tsx
            function UncontrolledInput() {
              const inputRef = useRef();
            
              const handleSubmit = () => {
                alert(inputRef.current.value);
              };
            
              return (
                <>
                  <input type="text" ref={inputRef} />
                  <button onClick={handleSubmit}>Submit</button>
                </>
              );
            }
            
            ```
            
    - 컴포넌트는 **외부에서 프롭으로 제어될 수 있고 내부적으로 자체 상태를 관리**하면서 선택적으로 외부 제어를 가능하게 함
        
        ```tsx
        function Toggle({ on: controlledOn, onChange }) {
          const [uncontrolledOn, setUncontrolledOn] = useState(false);
        
          const isControlled = controlledOn !== undefined;
          const on = isControlled ? controlledOn : uncontrolledOn;
        
          const toggle = () => {
            if (!isControlled) {
              setUncontrolledOn(!on);
            }
            onChange?.(!on);
          };
        
          return <button onClick={toggle}>{on ? '켜짐' : '꺼짐'}</button>;
        }
        	
        ```
        
    

## 5.5.5 프롭 컬렉션

- 여러 개 프롭을 함께 묶어야 하는 경우
- 프롭 게터
    - 사용자 정의 프롭을 프롭 컬렉션에 조합하고 병함함
    - 프롭 컬렉션을 반환하는 함수를 내보내는 것

## 5.5.6 복합 컴포넌트

- 상위 컴포넌트와 하위 컴포넌트들이 협력하여 하나의 큰 컴포넌트를 만들때 사용
- 상위 컴포넌트가 상태를 컨텍스트로 관리, 하위 컴포넌트들이 그 상태를 공유하거나 수정하는 방식

```tsx
import React, { createContext, useState, useContext } from 'react';

// Select 컨텍스트 생성
const SelectContext = createContext();

// Select 컴포넌트
function Select({ children, defaultValue, onChange, placeholder = "선택하세요" }) {
  const [isOpen, setIsOpen] = useState(false);
  const [selectedValue, setSelectedValue] = useState(defaultValue);
  const [selectedLabel, setSelectedLabel] = useState("");

  const toggleDropdown = () => {
    setIsOpen(!isOpen);
  };

  const selectOption = (value, label) => {
    setSelectedValue(value);
    setSelectedLabel(label);
    setIsOpen(false);
    
    if (onChange) {
      onChange(value);
    }
  };

  const contextValue = {
    selectedValue,
    selectOption
  };

  return (
    <SelectContext.Provider value={contextValue}>
      <div className="select-container">
        <div className="select-header" onClick={toggleDropdown}>
          <span>{selectedLabel || placeholder}</span>
          <span className="arrow">{isOpen ? '▲' : '▼'}</span>
        </div>
        
        {isOpen && (
          <div className="options-container">
            {children}
          </div>
        )}
      </div>
    </SelectContext.Provider>
  );
}

function Option({ value, children }) {
  // 상위 컴포넌트의 컨텍스트 사용
  const { selectedValue, selectOption } = useContext(SelectContext);
  
  const isSelected = selectedValue === value;
  
  const handleClick = () => {
    selectOption(value, children);
  };

  return (
    <div 
      className={`option ${isSelected ? 'selected' : ''}`}
      onClick={handleClick}
    >
      {children}
    </div>
  );
}

// Option을 Select의 정적 속성으로 할당
Select.Option = Option;

// 사용 예시
function App() {
  const [selectedFruit, setSelectedFruit] = useState('');
  
  const handleFruitChange = (value) => {
    setSelectedFruit(value);
    console.log(`선택된 과일: ${value}`);
  };

  return (
    <div className="app">
      <h1>과일 선택</h1>
      
      <Select 
        defaultValue="" 
        onChange={handleFruitChange}
        placeholder="과일을 선택하세요"
      >
        <Select.Option value="apple">사과</Select.Option>
        <Select.Option value="banana">바나나</Select.Option>
        <Select.Option value="orange">오렌지</Select.Option>
        <Select.Option value="grape">포도</Select.Option>
      </Select>
      
      {selectedFruit && (
        <p>선택한 과일: {selectedFruit}</p>
      )}
    </div>
  );
}

export default App;
```

## 5.5.7 상태 리듀서

- useReducer 훅을 사용하여 구현
- https://ko.react.dev/learn/extracting-state-logic-into-a-reducer
- https://ko.react.dev/reference/react/useReducer

# 5.6 돌아보기

- 메모화, 지연로딩, 리듀서, 상태관리. …

# 5.7 복습하기

1. 리액트에서 메모화란 무엇이며, 컴포넌트 렌더링을 최적화하는데 어떻게 사용할 수 있을까요?
    - 이전에 계산한 값을 재사용하여 불필요한 계산을 방지
    - React.memo, useMemo, useCallback을 통해 구현
2. 리액트에서 상태 관리를 위해 useReducer를 사용하면 어떤 우세한 점이 있으며, useReducer는 useState와 어떻게 다른가요?
    - 복잡한 상태관리 vs 간단한 상태관리
3. React.lazy와 Suspense 컴포넌트를 사용하는 리액트 애플리케이션에서 지연로딩을 어떻게 구현할 수 있을까요?
    - 초기 로딩 시간 단축, 필요한 시점에만 코드 로드, …
4. 리액트에서 메모화를 사용할 때 발생할 수 있는 잠재적 문제는 무엇이며 어떻게 완화할 수 있을까요?
    - 메모리 사용량 증가
    - 의존성 배열 관리 실수
    - 깊은 비교가 필요한 복잡한 객체에서 비효율
5. 리액트에서 컴포넌트에 프롭으로 전달된 함수를 메모화하기 위해 useCallback 훅을 어떻게 사용할 수 있을까요?
    - 함수 참조가 변경되어 자식 컴포넌트가 불필요하게 리렌더링되는 것을 방지

# 5.8 미리보기

- 서버 사이드 리액트
- 서버 사이드 렌더링
    - 장점
    - 단점
    - 하이드레이션
    - 프레임워크
# Performance Improvement

1. [크롬 확장프로그램 : React Developer Tools](https://chrome.google.com/webstore/)
   1. 개발중인 리액트사이트를 컴포넌트로 미리볼 수 있다.
   2. 컴포넌트구조 파악이 가능하다. 
   3. 컴포넌트 찍어보면, 컴포넌트에 있는  state, props 등 조회 및 수정 가능하다.
2. Profiler 성능 측정
   1. Profiler 탭에서 성능평가 가능
      1. Profiler 탭 들어가서, 녹화버튼 누르고, 페이지 이동이나 버튼조작을 해보고, 녹화를 끝내면, 방금 렌더링된 모든 컴포넌트의 렌더링시간을 측정해준다. 
      2. 이상하게 느린 컴포넌트가 있다면 여기서 찾을 수 있다. 
         1. div>를 1만개 만들거나 그러지 않는 이상 보통은 걱정할 필요는 없다. 
         2. 지연 원인 대부분은 서버에서 ajax 요청결과가 늦게 도착해서 그런 경우가 많다. 서버가 느린 건 어쩔 수 없다.
3. 크롬 확장프로그램 : Redux Developer Tools
   1. Redux store에 있던 state를 전부 확인 가능하다. 
   2. dispatch 날릴 때 마다 뭐가 어떻게 바뀌었는지 로그를 작성해준다. 
   3. store 복잡해지면 유용하다.
4. lazy import
   1. 기존 소스코드
    ```
    (App.js)
    
    import Detail from './routes/Detail.js'
    import Cart from './routes/Cart.js'
    ```
   2. 변환 소스코드
    ```
    (App.js)
    import {lazy, Suspense, useEffect, useState} from 'react'
    
    const Detail = lazy( () => import('./routes/Detail.js') )
    const Cart = lazy( () => import('./routes/Cart.js') )
    ```
     - "Detail 컴포넌트가 필요해지면 import 해주세요" 라는 뜻이 된다. 
     - 리액트로 만드는 Single Page Application의 특징은 html, js 파일이 하나만 생성된다. 따라서 모든 소스코드의 내용이 들어가 있어서 파일의 사이즈가 크다.
     - 하지만 이렇게 해놓으면 Detail 컴포넌트 내용을 다른 js 파일로 쪼개준다. 
     - 그래서 첫 페이지 로딩속도를 향상시킬 수 있다.
   3. lazt 사용한 컴포넌트 불러왔을 때 지연시간 걸리는 경우 처리하는 방법
    ```
    <Suspense fallback={ <div>로딩중임</div> }>
      <Detail shoes={shoes} />
    </Suspense>
    ```
     - Detail 컴포넌트가 로딩중일 때 대신 보여줄 html 작성도 가능하다.
     - 귀찮으면 <Suspense> 로 <Routes> 전부 감싸도 된다.
5. 자식 컴포넌트의 재렌더링을 막으려면 memo
   1. 컴포넌트가 재렌더링되면 거기 안에 있는 자식컴포넌트는 항상 함께 재렌더링된다. 따라서 자식 컴포넌티의 재렌더링을 막으려면 memo를 사용하면 된다.
      1. ```
         let Child = memo( function Child(){
           console.log('재렌더링됨')
           return <div>자식임</div>
         })

         function Cart(){

           let [count, setCount] = useState(0)

           return (
             <Child />
             <button onClick={()=>{ setCount(count+1) }}> + </button>
           )
         }
         ```
      2. 이 컴포넌트는 꼭 필요할 때 재 렌더링 해라라는 의미
   2. memo의 원리 : props가 변할 때만 재렌더링 해줌
      1. 따라서 신규 props랑 기존 props랑 비교하는 시간이 걸리기 때문에 props가 크고 복잡하면 이거 자체로도 부담이 될 수도 있다.
      2. 꼭 필요한 무거운 컴포넌트의 경우에만 사용하는 것이 좋다.
6. useMemo
   1. useMemo : 컴포넌트 렌더링시 1회만 실행해줌
    ```
    import {useMemo, useState} from 'react'
    
    function 함수(){
      return 반복문10억번돌린결과
    }
    
    function Cart(){ 
    
      let result = useMemo(()=>{ return 함수() }, [])
    
      return (
        <Child />
        <button onClick={()=>{ setCount(count+1) }}> + </button>
      )
    }
    ```
7. batching
8. useTransition
9. useDeferredValue

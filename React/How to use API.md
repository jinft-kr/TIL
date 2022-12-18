# How to use API

- React 애플리케이션에서 API를 사용하는 방법으로는 크게 `Axios(Promise 기반 HTTP 클라이언트)` 와 `Fetch API(Javascript 내장 Web API)`가 있다.
  - Web API : 클라이언트 측에서 사용할 수 있는 자바스크립트 내장함수
- `API`는 `React 애플리케이션에 사용되는 데이터`를 의미하다.<br>
  - 클라이언트 측에서 수행 할 수없는 특정 작업이 있으므로 이러한 작업은 서버 측에서 구현된다.<br>
  - 그런 다음 API를 사용하여 클라이언트 측에서 데이터를 사용할 수 있다.<br>
- API는 지정된 엔드 포인트가있는 JSON 형식 인 데이터 세트로 구성된다.<br>
  - API는 요청 및 응답의 형태에 대한 두 서비스 간의 계약이라고 말할 수도 있다.


### fetch
- `fetch` 는 ES6부터 `자바스크립트의 내장 라이브러리` 이다.
- 장점
  - `promise 기반` 으로 만들어졌다
  - 내장 라이브러리이기 때문에 별도의 `모듈 설치가 필요하지 않다`
- 단점
  - 브라우저 호환성이 떨어진다.
  - response timeout 처리 방법이 없는 등 기능적인 부분이 상대적으로 부족하다.
- fetch() 함수의 기본
    ```
    fetch('api 주소')
        .then(res => res.json())
        .then(res => {
          // data를 응답 받은 후의 로직
        });
    ```

### axios
- `axios` 는 브라우저, `Node.js` 를 위한 `Promise API`를 활용하는 `HTTP 비동기 통신 라이브러리` 이다.
  - 백엔드와 프론트엔드와 통신을 쉽게하기 위해 `AJAX` 도 더불어 사용하기도 한다.
- 특징
  - 운영 환경에 따라 브라우저의 `XMLHttpRequest 객체` 또는 Node.js의 `HTTP API` 사용
  - `Promise(ES6) API` 사용
  - 요청과 응답 데이터의 변형
  - HTTP 요청 취소 및 요청과 응답을 `JSON 형태로 자동 변경`
- 장점
  - `fetch`처럼 `promise`를 지원한다.
  - fetch와는 달리 `브라우저 호환성이 좋고` `편리`하며 `기능이 많다`.
- 단점
  - 라이브러리 설치가 필요하다.
- 따라서 React에서 http통신을 할 때엔 주로 axios를 이용한다.
- promise기반에 호환성이 좋고, 디테일한 기능들을 사용할 수 있기 때문이다.
- `axios()` 함수의 기본
  ```
  axios({
    //request
    method: "get",
    url: "url",
    responseType: "type"
  }).then(function (response) {
    // response Action
  });
  ```

### fetch vs axios
1. fetch
   ```
    const url ='http://localhost3000/test`
    const option ={
      method:'POST',
      header:{
        'Accept':'application/json',
        'Content-Type':'application/json';charset=UTP-8'
      },
      body:JSON.stringify({
        name:'sewon',
        age:20
      })
    
      fetch(url,options)
        .then(response => console.log(response))
   ```
2. axios
    ```
    const option ={
      url ='http://localhost3000/test`
       method:'POST',
       header:{
         'Accept':'application/json',
         'Content-Type':'application/json';charset=UTP-8'
      },
      data:{
        name:'sewon',
            age:20
      }
    
      axios(options)
        .then(response => console.log(response))
    ```
- 차이점
  - `fetch()` 는 `body` 프로퍼티를 사용하고, `axios` 는 `data` 프로퍼티를 사용한다.
  - `fetch` 의 `url` 이 `fetch()함수의 인자` 로 들어가고, `axios` 에서는 `url` 이 `option` 객체로 들어간다.
  - `fetch` 에서 `body` 부분은 `stringify()` 로 되어진다.
- 이처럼 `axios` 는 `HTTP 통신의 요구사항` 을 컴팩트한 패키지로써 `사용하기 쉽게` 설계 되었다.

### fetch - method GET or POST 인 경우
- fetch() 메소드의 default는 GET 요청이다.
- POST인 경우에는 fetch() 메소드에 method 정보를 인자로 넘겨주어야 한다.
```
fetch('https://api.google.com/user', {
        method: 'post',
        body: JSON.stringify({
            name: "yeri",
            batch: 1
        })
    })
    .then(res => res.json())
    .then(res => {
    if (res.success) {
        alert("저장 완료");
    }
})
```
- post로 데이터를 보낼 때 JSON.stringfy를 항상 하다보니 axios는 굳이 감싸주지 않고 객체만 작성해도 되는 편리한 점이 있다. 
- 이렇듯 axios는 소소하게 편한한 설정을 제공해주고, 요청과 응답에 대한 확장성 있는 기능을 만들 수 있다.

### axios - method GET or POST 인 경우
- GET : `axios.get(url[,config])`
  1. 지정된 단순 데이터 요청을 수행하는 경우 -> url만 파라미터로 넘김
    ```
    async function getData() {
      try {
        //응답 성공
        const response = await axios.get('url주소');
        console.log(response);
      } catch (error) {
        //응답 실패
        console.error(error);
      }
    }
    ```
  2. 사용자 번호에 따라 다른 데이터를 불러오는 경우-> url과 함께 prams:{} 객체도 파라미터로 넘김
    ```
    async function getData() {
      try {
        //응답 성공
        const response = await axios.get('url주소',{
          params:{
            //url 뒤에 붙는 param id값
            id: 12345
          }
        });
        console.log(response);
      } catch (error) {
        //응답 실패
        console.error(error);
      }
    }
    ```
- POST : `axios.post(url, data[,config])`
  - 서버에 데이터를 보내는 POST 메소드에서는 보내고자 하는 데이터를 message body에 포함시켜 보낸다.
    ```
    async function postData() {
      try {
        //응답 성공
        const response = await axios.post('url주소',{
        //보내고자 하는 데이터
        username: "devstone",
        password: "12345"
        });
        console.log(response);
      } catch (error) {
      //응답 실패
      console.error(error);
      }
    }
    ```

### fetch() res.json()의 의미
- 첫 번째 `then` 함수에 전달된 인자 `res` 는 `http 통신 요청` 과 응답에서 `응답의 정보` 를 담고 있는 객체이다.
- 그런데 `console` 을 확인해보면 백앤드에서 넘겨주는 응답 body, 즉 실제 데이터는 보이지 않을 것이다. 즉 { success: true } 라는 JSON 데이터는 위의 코드로는 console에 찍히지 않을 것이라는 말이다.
- `응답으로 받는 JSON 데이터를 사용하기 위해서는`  Response Object 의 `json 함수를 호출` 하고, `return` 해야한다. 그러면 이 값이 `두 번째 then 함수의 인자로 온다`.(Obj 형태로)
  - `❓ 왜 json.parse가 아닌 json을 쓸까?`
    - response 안에 정보는 너무 많고, 이 모든 데이터를 바꿔줄 필요는 없다. `body에 있는 정보들만 꺼내 바꿔주는게 json` 이다.

### 비동기처리
- `동기` 는 `순차적`, `직렬적` 으로 테스크를 수행하고, `비동기` 는 `병렬적` 으로 테스크를 수행한다.
- 예를 들어 1부터 10까지의 함수가 있다고 가정해보자. 동기함수는 1이 끝나고 2, 2가 끝나고 3, 4, 5 ...의 순서로 실행된다면, 비동기함수는 순서와 상관없이 먼저 완성된 쪽이 실행 결과를 return하게 된다.
- `fetch` 는 `비동기` 적으로 처리되는 함수이고, 처리가 완료되기까지 시간이 오래걸리기 때문에 `fetch` 가 끝나기도 전에 다른 함수가 먼저 실행될 수 있다. (=순서가 섞일 수 있다) 그렇기 때문에 `then` 을 써서 `순서를 고정` 시키는 것이다.
- 다시 말해, `then` 은 `fetch` 다 끝나고나서 이 일을 해줘"의 뜻을 갖는 셈이다.
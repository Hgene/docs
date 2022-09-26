이 페이지는 API서버와의 통신을 위한 axios 설치 및 사용법에 대해서 다루고 있습니다.   
내용의 일부분은 시작하기 | [Axios Docs Document](https://axios-http.com/kr/docs/intro)와 [axios 설치 & 특징](https://inpa.tistory.com/entry/AXIOS-%F0%9F%93%9A-%EC%84%A4%EC%B9%98-%EC%82%AC%EC%9A%A9) 블로그에서 참고하였습니다.  ajax와 더불어 많이 Javascript 통신라이브러리로써 사용되는 axios를 통한 통신과정에 대해서 설명하겠습니다.

<br/><br/>

<p align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/d1/Axios_%28computer_library%29_logo.svg/1280px-Axios_%28computer_library%29_logo.svg.png"  width="70%" /> 
</p>  
<br/>

## Axios 란
Axios는 node.js와 브라우저를 위한 Promise 기반 HTTP 클라이언트 입니다. 
서버 사이드에서는 네이티브 node.js의 http 모듈을 사용하고, 클라이언트(브라우저)에서는 XMLHttpRequests를 사용합니다. 대표적인 특징은 아래와 같습니다.

- 브라우저를 위해 XMLHttpRequests 생성 / node.js를 위해 http 요청 생성
- Promise API를 지원
- 요청 및 응답 인터셉트 / 데이터 변환 / 요청취소
- JSON 데이터 자동 변환**
- XSRF를 막기위한 클라이언트 사이드 지원  

<br/> 

## 실행방법
axios를 실행하기 위해서는 어떻게 동작하는 지 알아야합니다. 이를 위해서 일반적인 API 서버가 어떻게 동작하는 지에 대해서 설명하겠습니다. 
먼저 클라이언트는 HTML 페이지를 통해서 서버가 띄우는 정보에 접근합니다. 즉 사용자에게 보여지는 것은 단순한 HTML입니다. 
그 외에 페이지 이동과 같이 기초적인 동작은 Flask가, 클릭 시 화면이 변하는 것과 같이 동적인 페이지로의 역할은 Javascript(이하 JS)가 담당한다고 볼 수 있습니다. JS는 HTML 내부 element에 트리거(onclick, keydown 등)에 의해서 동작합니다.  

Axios는 통신기능을 제공하는 JS 라이브러리입니다. 즉 Axios는 JS에 기반하여 동작하기 때문에, 아래와 같은 순서로 접근이 필요합니다.  


### [ 동작과정 ]

HTML → (필요 시) 요청데이터 작성 → JS 트리거 유발 →  Axios 실행 ( HTTP Request 전달 ) →  Flask에서 요청수령

Axios 를 통한 통신의 종류나 처리방법에 대해서는 JS 기반으로 셋팅한 후, HTML에 트리거가 발생하면, Axios 가 Flask에 요청을 넣게되고, Flask는 요청을 받게 되는 구조입니다. 이후에 콜백함수로 HTML를 변경데이터에 대해서 비동기 업데이트를 하게 되는 방식입니다.


![image](https://user-images.githubusercontent.com/47958965/192215092-b0128739-fcf5-42c0-bf56-02ba9b3e9806.png)

<br/> 

### 설치하기
먼저 사용을 위해 axios를 설치하는 방법에 대해서 알아보자면, 주로 npm이나 <script src=””>를 사용하여 설치합니다.  

```shell
$ npm install axios
```

### HTML 내부
```html
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<-- OR -->
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```  
  
<br/> 

  
### 문법
그리고 기초적인 문법은 아래와 같습니다. 요청파트 `axios(config)`  / 성공 시`then` / 실패 시`catch` 처리로 나누어지며, config는 `url, method, data` 를 각각 입력하여 구현하면 요청이 생성됩니다.

```javasript
// 요청(GET, POST, PUT 등) 전송
axios(
  config
)

//성공 시
.then(function(response){
  console.log(response);             //console에 결과출력
  callback(true, response.data);     //data 콜백
})

//실패 시
.catch(function(err){
  console.log(err.toJSON());         //console에 에러를 json으로 출력
})

//마지막처리 이후 실행되는 부분
.finally(function () {
    // always executed
  }) 
;
``` 
<br/>
  
### config 구성
그렇다면 위의 {config} 는 어떻게 구성할까? 아래는 기초적인 버전으로 구성하였으며, 조금 더 자세한 구성을 알고 싶다면, [요청 Config | Axios Docs](https://axios-http.com/kr/docs/req_config) 페이지를 참고해주세요.

```javascript
axios({
    //메서드 방식
    method:"POST",
    
    //요청에 사용될 서버 URL
    url: 'https://reqres.in/api/login',
    
    // 요청과 함께 전송되는 URL 파라미터. url + "?key=value&..." 형태로 전달
    params: {
      ID: 12345   // 'https://reqres.in/api/login?ID=12345'로 요청
    },
  
    //전송할 데이터
    data:{
        "email": "jin@spoonradio.co",
        "password": "qwer1234"
    }
    
    //프록시 서버의 호스트이름, 포트, 프로토콜을 정의(선택사항)
    proxy: {
      protocol: 'https',
      host: '127.0.0.1',
      port: 9000,
      auth: {
        username: 'mikeymike',
        password: 'rapunz3l'
      }
    }
})
```
<br/>
  
### 자체 method
확장성을 위한 포맷외에도 자체적으로 메소드를 지원하여 편의성을 향상하였습니다. 

```javascript
axios.request(config)
axios.get(url[, config])
axios.delete(url[, config])
axios.head(url[, config])
axios.options(url[, config])
axios.post(url[, data[, config]])
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])
```

### Error 핸들링
```
.catch(function (error) {
    if (error.response) { //요청전송 > 2xx 외의 상태 코드로 응답 시
      console.log(error.response.data);       // 서버가 제공한 응답(데이터)
      console.log(error.response.status);     // 서버응답 상태코드
      console.log(error.response.headers);    // 서버응답 헤더(소문자)
      
    } else if (error.request) { //요청전송 > 응답수신X
      console.log(error.request);

    } else { //그 외 오류
      console.log('Error', error.message);
    }
    console.log(error.config);
  });
``` 
<br/>
  
  
---
## (잠깐..!) Callback이란  
***함수가 끝난 후 실행되는 인자로 받아진 함수*** 를 일컷는 말로, 인자로 대입되는 함수를 콜백함수라고 합니다. 자바스크립트에서 함수는 객체이므로, 다른 함수를 인자로 받을 수 있는데 이 함수를 콜백함수라고 합니다.

***자바스크립트는 이벤트 중심적인 언어***이기 때문에, 이벤트 완료되어 값이 반환될 때까지 기다리지 않고 다음의 이벤트를 실행합니다. 이 때 특정코드가 마무리되기 전에 다른 이벤트가 실행되지 않도록하는 것이 콜백이며, **비동기처리를 위한 방법으로 사용**하고 있습니다.

```javascript
function myDisplayer(sum) {
  document.getElementById("demo").innerHTML = sum; //demo블럭 안에 sum을 추가
}

function myCalculator(callback, num1, num2) {
  let sum = num1 + num2;
  callback(sum);
}

myCalculator(myDisplayer, 5, 5);
>> id="demo" 블럭에 sum 값이 innerHTML형태로 들어가게 된다.
``` 
<br/>
 
---
# [ 페이지 구현 ]
지금까지는 axios를 어떻게 사용하는 지에 대해서 설명했다면, 이제부터는 직접 구현해보도록 하겠습니다. 

앞선 내용을 정리하면, 아래와 같은 요소들의 구현이 필요합니다.  
  1. Flask 페이지 구동  
  2. HTML 페이지에 JS 트리거용 element 생성  
  3. JS기반 axios 작성  

를 생성한 후에 정상작동여부(Request 발생 및 Flask에 전달된 정보) 까지 확인해보도록 하겠습니다.  
<br/>
  

## Step 1.  Flask 페이지 구동
먼저 아래와 같이 빈 페이지에  <input> element을 부여하고 페이지를 구동합니다. Flask에서는 이 때 한개 페이지만 띄우기 때문에 매우 간단하게 구현이 가능합니다.

### [ Flask ]

```python
from flask import Flask, render_template, make_response, request

app = Flask(__name__)

@app.route('/axios_test', strict_slashes=False) #test api
def show_axios_index():
    return render_template('axios_index.html')
``` 
<br/>
  
  

## Step 2.  HTML 페이지에 JS 트리거용 element 생성
그리고 아래와 같이 빈 페이지에  <input> element을 넣고, 각각  임의의 id를 부여한 후 페이지를 구동합니다. 

### [ HTML, axios_text.html ]

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Axios test Tutorials</title>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    // 아래는 ./static/js/customscript.js 을 로드하는 스크립트
    <script src="{{ url_for('static', filename='js/customscript.js') }}"></script>
</head>
<body>
    <div>!!! Here is test area for axios form !!!</div>
    <div>----------------------------------------</div>
    <br/>
    <div class="row">
        <div onkeydown="onLoggin(refresh_div)">         // onLoggin에 refresh_div를 콜백함수로 활용
          <label for="edit_text_email">E-mail :</label>
            <input type="text" id="edit_text_email" >
          <label for="edit_text_etc_comment">전송텍스트 :</label>
            <input type="text" id="edit_text_etc_comment">
        </div>
    </div>
    <div class="row" id="output_div"></div>
    <br/>
</body>
</html>
``` 
<br/>
<img width="546" alt="스크린샷 2022-09-26 오후 3 02 51" src="https://user-images.githubusercontent.com/47958965/192222826-762f96b1-2ac0-48a9-9395-d57b2df3e66c.png">


<br/>
  
## Step 3.  JS기반 Axios 작성
마지막으로 로드한 ./static/js/customscript.js 파일을 작성합니다.

아래의 js 파일은  

1. Enter 트리거 발생 시에 axios로 API서버에 PUT method를 요청하고,  
2. HTML파일은 비동기성으로 임의로 업데이트 하는 케이스입니다.  

이를 활용하면, 1번 put method를 요청 시에 DB를 업데이트하고, 2번에 DB로 받던 정보를 임의로 업데이트하면 비동기통신방식이 이루어지는 것입니다. 
  이 때 각 페이지에 테스트용 프린트 함수(js > `console.log` , python: `print` )를 추가하였습니다.

#### [ Flask 코드 update ]

```
@app.route("/axios_test_request", methods=['GET', 'POST', 'PUT', 'DELETE'])
def request_test():
    if request.method == 'POST':
        print('Flask Requested method: POST')
        print('Params: {}'.format(request.args))
        print('Data: {}'.format(request.get_json()))
        
    if request.method == 'GET':
        print('Flask Requested method: GET')
        print('Params: {}'.format(request.args))
        print('Data: {}'.format(request.get_json()))
        
    if request.method == 'PUT':
        print('Flask Requested method: PUT')
        print('Params: {}'.format(request.args))
        print('Data: {}'.format(request.get_json()))
        
    if request.method == 'DELETE':
        print('Flask Requested method: DELETE')
        print('Params: {}'.format(request.args))
        print('Data: {}'.format(request.get_json()))

    return make_response(jsonify({'status': True}), 200)
``` 

#### [ `./static/js/customscript.js` 코드 ]

```javascript
function onLoggin(callback) {
  const email = document.getElementById('edit_text_email');
  const etc_cmt = document.getElementById('edit_text_etc_comment');
  
  // Enter 입력 시
  if (event.keyCode==13) {
      axios({
            method: 'PUT',
            url: 'http://localhost:8000/axios_test_request',
            params: {
                email: email.value,
            },
            data: {
                new_cmt: etc_cmt.value,
                },
      })
      .then(function(response) {
          console.log(response);
          callback(true, response.data);
  
      })
      .catch(function(error){
          console.log(error);
          callback(false, JSON.stringify(error));
      });
  
  // Esc 입력 시
  } else if (event.keyCode==27){
      document.getElementById('edit_text_email').value = email.value;
      document.getElementById('edit_text_etc_comment').value = etc_cmt.value;
  }
  
};
  
//HTML 업데이트를 위한 콜백함수
function refresh_div(){
  var dom_element = document.getElementById("output_div")        # 업데이트부분
  var email = document.getElementById("edit_text_email").value;  # 변경 값1, e-mail
  var email_text = document.getElementById("edit_text_etc_comment").value; # 변경 값2, e-mail text
  
  var html_value = `<div class="row">
                    <div class="col">E-mail: ${email}</div>                
                    <div class="col">E-mail Text: ${email_text}</div>                
                    </div>`;
  dom_element.innerHTML = html_value;
  
};
``` 
<br/>
  
  
## Finally  정상구현여부 확인
이렇게 Axios를 통한 HTTP Request를 구성하면,  

#### (1) Enter 트리거를 발생 시에  HTML이 업데이트 되고,  
<p align="center">  
  <figure>
<img width="518" alt="스크린샷 2022-09-26 오후 4 34 01" src="https://user-images.githubusercontent.com/47958965/192218958-94762f27-1f53-4585-bef6-5f267ebb7cd1.png">
  <figcaption>[ 콜백함수 refresh_div 를 통한 HTML 업데이트 ]</figcaption>
  </figure>
</p>  

 
  
<br/>  
  
#### (2) Request에 요청한 정보(param + data) 가 정상적으로 생성되었으며,  
- URL : /axios_test_request?{key}=value 형태로 정상적으로 구성  
- data: 전송한 텍스트가 정상적으로 key:value형태로 전달됨을 확인  


<p align="center" style="text-align: center;">
<img width="45%" alt="스크린샷 2022-09-26 오후 3 23 42" src="https://user-images.githubusercontent.com/47958965/192218641-e8cf6c77-c176-4982-a9f4-8d1617ce7c2e.png" alt="[ URL 정상구성 ]">
  &nbsp; &nbsp; &nbsp; &nbsp;
<img width="45%" alt="스크린샷 2022-09-26 오후 3 25 41" src="https://user-images.githubusercontent.com/47958965/192218617-1a419294-0ae2-420f-b516-ea054b6d0b9e.png" alt="[ data 정상 전송 ]">
</p>
 
 
<br/>

#### (3) Flask 또한 그 정보를 제대로 수령한 것을 확인할 수 있습니다.  
- Request method= PUT 으로 정상요청처리  
- param(request.args)과 data(request.get_json()) 정상 전송  
  <br/>
![스크린샷 2022-09-26 오후 3 28 17](https://user-images.githubusercontent.com/47958965/192218512-d8e7ec64-0aa3-43dc-9197-eba3df98d30f.png)  






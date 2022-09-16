<p align="center"><img src="https://velog.velcdn.com/images/moonseok/post/c8cb8b9b-9af1-42db-9323-631568d84f11/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202022-01-07%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.37.34.png" width="50%" height="50%" /></p>
 


### Nginx란  

경량 웹 서버로 클라이언트의 요청에 맞는 응답용 HTTP Web Server로 활용하거나, Reverse Proxy Server 로 WAS 서버의 부하를 줄이는 로드밸런서로 활용되기도 합니다.

`Web Server`: 정적컨텐츠(이미지, HTML)를 제공하고, 동적컨텐츠 요청을 식별하여 App server로 요청을 전달  

`Reverse Proxy Server`: 클라이언트의 요청을 받아 다른 웹서버로 전달하여 결과물을 받고, 다시 전달하는 중간다리   


이러한 역할을 하는 Nginx는 아래와 같은 장점이 있습니다.  
  1. 로드 밸런싱 : 요청이 많은 사이트를 운영하는 경우 하나의 서버가 아닌 여러 대의 서버를 두고 운영을 하게 됩니다. 그럴 경우 특정 서버에만 요청이 몰리지 않도록 하는 역할을 NGINX가 수행하게 됩니다.  
  2. 공격으로 부터 보호 : NGINX를 사용하면 웹사이트나 서비스에서 실제 서버의 IP 주소를 필요로 하지 않기 때문에 DDoS와 같은 공격이 들어와도 NGINX를 공격하게 되지 실제 서버에는 공격이 들어오는 것을 막을 수 있습니다.  
  3. 캐싱 : NGINX는 콘텐츠를 캐싱할 수 있어 결과를 더 빠르게 응답하여 성능을 높일 수 있습니다.  

 

### 그렇다면 왜 필요할까?  
하나의 웹서버 안에는 한 개의 서버만 존재하지 않습니다. api 서버, BI(Metabase) 서버, 관리자 서버 등 여러개의 서버를 동시에 사용하는 경우가 대다수입니다. 그리고 이 서버들은 서로 다른 포트번호를 가지고 있는데, 도메인 요청이 들어왔을 때 의도에 맞추어 적절한 서버(포트번호)를 보내주는 작업을 해야하는데 그 역할을 하는 것이 Nginx의 포트포워딩(Port Forwarding) 기능입니다.

<br/>

[ Nginx를 통한 외부유저의 DB 접근 Flow ]
 

### _“포트포워딩을 통해 서버에 들어온 요청을 내부의 특정 포트로 보내주는 것이 가능해진다.”_  

예를 들어 123.45.67.89 의  IP를 가진 웹서버가 있는 경우, `http://123.45.67.89:80`  이라는 요청이 들어왔을 때, `localhost:5000` 에 임의로 매핑시켜주는 것이 가능합니다.

설치 및 서버시작

```
#설치
$ sudo apt-get install nginx   #리눅스
$ brew install nginx           #MacOS

#시작
$ sudo systemctl start nginx

#상태확인
$ sudo systemctl status nginx

#설정파일 확인/테스트
sudo nginx -t
```
 

서버 재시작(Restart) 및 리로드(Reload) 방법

```
#Restart : 서버를 중단 후 재가동
$ sudo service nginx restart

#Reload : 변경된 설정만 다시 적용
$ nginx -s reload

#nginx -s 관련 명령어
$ nginx -s quit     #서버 shutdown 
$ nginx -s stop     #서버 중지
$ nginx -s reopen   #로그파일 다시열기
```


설정파일 변경
내용: nginx 포트포워딩, proxy / proxy_pass 설정

디렉토리: `/etc/nginx/sites-available/default` 혹은 `/etc/nginx/sites-enabled/default`  

```
$ sudo vi /etc/nginx/sites-available/default

>>> 파일내용 

server {
    listen 80 default_server;                      #받게되는 포트번호 http
    listen [::]:80 default_server;                 #받게되는 포트번호 https
    server_name localhost;                         #서버주소
    location / {                                   #적용할 주소, 정규표현식 사용부분
           proxy_pass http://localhost:5000;       #Redirect할 주소 및 포트번호
    }
}

>>> 파일저장

$ nginx -s reload    #설정 Reload
```

만약 `default_server:8888` 로 들어오는 하위 디렉토리를 모두 `localhost:5001` 의 하위에 매핑시키고 싶다면, 아래와 같은 정규표현식으로 적용이 가능하다. 아래의 설정파일은 다음과 같은 기능을 가진다.

`default_server:8888` 로 들어온 요청은 `http://localhost:8888/jin` 으로 redirect한다.(line 8)

`http://localhost:8888/jin` 으로 들어온 요청은 `http://localhost:5001;` 으로 redirect한다.(line 13)

`/jin` 으로 시작하는 모든 주소(하위 디렉토리 포함)를  `http://localhost:5001/jin` 으로 보내되, 추가적인 룰에 의해 URI 가 다시 변경되지 않도록(break)한다.(line 12)

```
server {
        listen 8888;
        root /var/www/html;
        index index.html index.html;
        server_name _;

        location / {
                return 301 http://localhost:8888/pooding;
        }

        location ~ ^/pooding {
                rewrite ^/pooding/(/.*)$ /?_=$1         break;
                proxy_pass              http://localhost:5001;
                proxy_redirect          off;
                proxy_set_header   Host $host;
        }

}
```

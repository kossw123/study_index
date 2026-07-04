
## 왜 필요한가?

- 웹 환경이 발전하여, 요청 하나에 프로세스나 스레드에서 순차적으로 처리하는 방법 대신 여러 스레드를 두거나 하나의 프로세스가 동시(Concurrency)에 처리하거나, 하나의 프로세스를 여러 스레드에 분산(Parallelism)해 처리하는 방식이 등장했다. 
- 또한 HTTP 프로토콜 위에서 실시간 통신을 흉내냈지만, 오늘날에는 Web Socket같은 실시간 양방향 통신을 주로 사용하고 이런 요청을 처리하기 위해 ASGI가 등장했다.
- Uvicorn은 대표적인 ASGI다.
- 



## 정의

현재 내 프로젝트에서는 전통적인 동적 웹 프로그래밍처럼 static 서버를 따로 두지 않는다.
FastAPI라는 프레임워크를 사용하지만 엄밀하게 말한다면 Uvicorn을 사용했다가 정확한 표현이다.

## 그렇다면 Uvicorn은 프레임워크 인가?

아니다.

파이썬으로 작성된 ASGI(Asynchronous Server Gateway interface, 비동기 통신 연결 인터페이스)다.

즉, Uvicorn은 프레임워크(FastAPI, starlette 등)로 작성된 ASGI 코드를 앱으로 만들어 외부 네트워크 요청과 연결하여 실행하는, WAS의 역할을 하는 서버 프로그램이다.

## Uvicorn의 역할

- HTTP 요청을 받는다.
- ASGI 서버 역할을 수행한다(WAS)
- FastAPI를 실행한다.

그 와중에 FastAPI는 다음과 같은 역할을 수행한다.

- URL 라우팅
- 요청/응답 처리
- 의존성 주입
- 비즈니스 로직 호출

## 그렇다면 전통적인 동적 웹 프로그래밍에서 Uvicorn(ASGI)는 어떻게 동작될까?

현재 내 프로젝트에는 웹 서버(Nginx, Apache 등)는 없다.
오직 Uvicorn만 존재하여 WAS의 역할을 수행한다.

그렇다면 웹 서버가 있다면 어떻게 동작하게 될까?

client <-> Nginx <-> Uvicorn <-> MySQL

1. 인터넷을 통해 클라이언트 -> Nginx에게 요청(Reverse Proxy)
	- 서비스 환경에서는 앞 단에 웹 서버(Nginx)를 통해 요청을 빠르게 받을 수 있다.
	- 느린 네트워크 사용중이라면 Nginx가 패킷을 기다리고, 완전한 패킷을 전송한다.
2. Nginx -> Uvicorn
	- Nginx에서 받은 완전한 패킷을 가지기 때문에, 대기 시간 없이 바로 연산한다,
3. Uvicorn -> DB
	- 비즈니스 로직 실행, 쿼리 실행
4. DB -> Uvicorn
	- 쿼리 결과 반환, 응답 처리 로직 실행
5. Uvicorn -> Nginx
	- Nginx는 결과를 받아 클라이언트에게 천천히 안전하게 전달

## 프로젝트에서는 어디에서 사용하는가?

- WAS에서 설치해 사용중이다.

## 관련 개념

#web #socket #infra #uvicorn #ASGI #nginx #proxy 
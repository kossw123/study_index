## 왜 필요한가?

- 대규모 트래픽 환경에서 요청마다 스레드/프로세스를 생성하는 전통적인 방식(WSGI)은 자원 고갈 문제를 겪는다. 이를 해결하기 위해 하나의 프로세스가 이벤트 루프를 통해 수많은 비동기 요청을 동시(Concurrency)에 처리하는 ASGI 표준이 등장했다.
- 또한 웹소켓(WebSocket)과 같은 실시간 양방향 통신을 파이썬 환경에서 안정적으로 구동하기 위한 웹 서버가 필요해졌다.
- **Uvicorn**은 대단히 빠르고 가벼운 대표적인 **ASGI 서버(웹 애플리케이션 서버, WAS)** 이다.

## 정의

- Uvicorn은 프레임워크가 아닌, 파이썬 ASGI 규격을 준수하여 만든 **고성능 웹 서버(WAS)** 이다.
- 웹 프레임워크(FastAPI 등)로 작성된 애플리케이션 코드를 로드하여, 외부 네트워크 요청과 연결하고 실행해 주는 역할을 담당한다.

### 💡 Uvicorn(WAS)과 FastAPI(Web App)의 명확한 역할 분담

Uvicorn과 FastAPI는 상호 대체 관계가 아니며, 하나의 세트로 움직인다.
- **Uvicorn (서버 엔진):** 클라이언트로부터 TCP/HTTP 네트워크 연결을 맺고 관리하며, 요청을 ASGI 규격 메시지로 변환해 FastAPI에 전달한다.
- **FastAPI (비즈니스 로직):** Uvicorn이 토스해 준 요청을 받아 URL 라우팅, 유효성 검증, **비즈니스 로직 수행 및 DB 쿼리 실행(애플리케이션 단)**을 처리한 후 응답을 생성한다.

## Nginx와 Uvicorn의 연동 구조 (Reverse Proxy)

실제 운영(프로덕션) 환경에서는 보안과 효율성을 위해 Uvicorn 앞에 웹 서버(Nginx)를 두는 구조를 권장한다.

```
[Client] <─(HTTP/WS)─> [Nginx] <─(Reverse Proxy)─> [Uvicorn (FastAPI)] <──> [MySQL]
```

1. **Client ──> Nginx (요청 접수)** 
	-  Nginx가 클라이언트의 요청을 가장 먼저 받는다. 
	- **요청 버퍼링(Request Buffering):** 
		- 클라이언트의 네트워크가 느려 HTTP 요청 데이터가 천천히 들어오면, Nginx가 이를 끝까지 안전하게 다 받아낸다. (설정에 따라 활성화) 
2. **Nginx ──> Uvicorn (요청 전달)** 
	-  Nginx가 완성된 HTTP 요청을 로컬 네트워크를 통해 Uvicorn에 순식간에 전달한다. 
	- 덕분에 Uvicorn은 클라이언트의 느린 속도 때문에 대기(Slow Client 병목)하지 않고, 곧바로 연산에 집중할 수 있다. 
3. **Uvicorn 내부의 FastAPI ──> DB (로직 및 쿼리 실행)** 
- Uvicorn이 전달한 이벤트를 받아 **FastAPI 애플리케이션이 비즈니스 로직을 실행하고 DB 쿼리를 수행**한다. 
4. **DB ──> Uvicorn 내부의 FastAPI (결과 반환)** 
	-  쿼리 결과를 바탕으로 FastAPI가 최종 HTTP 응답 객체를 만들어 Uvicorn에게 넘긴다. 
5. **Uvicorn ──> Nginx ──> Client (응답 버퍼링 및 전달)** 
	- **응답 버퍼링(Response Buffering):** Uvicorn이 보낸 응답을 Nginx가 메모리에 빠르게 확 받아둔다. Uvicorn은 즉시 다음 요청을 처리하러 가고, Nginx가 느린 클라이언트에게 응답을 천천히 안전하게 밀어내 준다. 
## 프로젝트에서는 어디에서 사용하는가? 
-  프로덕션 인프라(WAS) 환경에 가상환경을 구축하고 설치하여 사용 중이다. 
- 안정적인 프로세스 관리(프로세스 다운 시 재시작, 멀티 코어 활용 등)를 위해 Gunicorn의 워커(Worker) 클래스로 Uvicorn을 지정하여 실행하는 구조(`gunicorn -k uvicorn.workers.UvicornWorker`)를 주로 활용한다. 
## 관련 개념 

- #Uvicorn #ASGI #FastAPI #Nginx #ReverseProxy #Buffering #SlowClient
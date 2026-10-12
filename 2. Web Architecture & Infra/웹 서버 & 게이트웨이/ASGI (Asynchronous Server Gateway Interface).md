## 왜 필요한가?

- 기존의 **WSGI(Web Server Gateway Interface)** 표준은 한 번의 요청에 하나의 스레드를 할당하는 **동기식(Blocking) 구조**이다.
- 이로 인해 웹소켓(WebSocket)이나 롱 폴링(Long Polling)처럼 서버와 클라이언트가 연결을 계속 유지해야 하는 실시간 양방향 통신을 처리할 때, 스레드가 쉽게 고갈되어 대규모 연결을 효율적으로 감당하지 못한다.
- 현대의 채팅, 실시간 알림, 주식 차트 등 대규모 실시간 연결을 안정적으로 처리하기 위해, **비동기(Asynchronous) 이벤트 루프 기반으로 수많은 커넥션을 효율적으로 관리할 수 있는 새로운 표준**이 필요해졌다.

## 정의

- **ASGI(Asynchronous Server Gateway Interface)**는 비동기(Async) 및 동기(Sync) 처리를 모두 지원하도록 설계된 파이프라인이자, **Python 웹 애플리케이션/프레임워크와 웹 서버 간의 표준 인터페이스 규격**이다.

> **[착각해선 안 되는 점: ASGI는 규격(Specification)이다.]**
> - **ASGI 사양/규격:** 웹 서버와 앱이 어떻게 비동기 메시지를 주고받을지 정의한 약속.
> - **ASGI 서버(런타임):** 이 규격을 구현하여 실제로 요청을 받는 실행기. (예: Uvicorn, Daphne)
> - **ASGI 지원 프레임워크:** 이 규격 위에서 개발자가 편리하게 웹 앱을 만들 수 있게 돕는 도구. (예: FastAPI, Django Channels)

## 동시성(Concurrency)과 병렬성(Parallelism)

- ASGI는 기본적으로 비동기 이벤트 루프를 활용하여, 단일 스레드 내에서 수많은 I/O 요청을 멈춤(Blocking) 없이 번갈아 처리하는 **동시성(Concurrency)**을 극대화한다.
- 그렇다고 병렬 실행과 배타적인 것은 아니다. 실무에서는 ASGI 서버(예: Uvicorn)의 워커(Worker) 프로세스를 여러 개 띄우는 방식으로 **멀티 코어 CPU를 활용한 병렬성(Parallelism)을 동시에 확보**하여 운영한다.

## 프로젝트에서는 어디에서 사용하는가?

- 우리 프로젝트는 비동기 처리에 특화된 **FastAPI 프레임워크**를 채택하였다. 
- 이를 통해 내부적으로 ASGI 표준 규격을 준수하는 애플리케이션을 구동하며, ASGI 비동기 서버인 **Uvicorn**을 런타임으로 활용하여 고성능 동시성 처리를 달성하고 있다.

## 관련 개념

#ASGI #WSGI #FastAPI #Uvicorn #Concurrency #Parallelism #Websocket

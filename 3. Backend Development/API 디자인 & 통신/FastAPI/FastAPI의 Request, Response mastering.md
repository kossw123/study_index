## 왜 필요한가?
- pydantic을 사용하면 유효성 검사와 파싱 및 캐스팅을 편하게 해주지만 실시간 작업이 필요한 경우에는 외부 라이브러리나 다른 방법을 찾아야 했으나, 이미 FastAPI에 내장되어 있는 기능을 사용하면 좀 더 로우 레벨의 데이터를 조작할 수 있었다.

# FastAPI.Request 클래스 기본 사용법

## ① 비동기 스트리밍 및 대용량 파일 다루기 (`request.stream()`

- `request.body()`나 `request.json()`은 요청 데이터를 한 번에 메모리에 올리기 때문에 대용량 파일이나 실시간 데이터(JSON Stream 등)를 다룰 때 서버가 다운될 가능성이 있다.
- `Request` 객체의 `stream()` 메서드는 데이터를 **청크(Chunk) 단위로 쪼개어 비동기(Async)적으로 읽어올 수 있게** 해서 용량 업로드나 네트워크 프록시를 구현할 때 메모리를 극도로 아낄 수 있다.

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.post("/stream")
async def stream_raw_data(request: Request):
    # 데이터를 메모리에 한 번에 올리지 않고, 들어오는 대로 쪼개서 처리합니다.
    async for chunk in request.stream():
        print(f"받은 데이터 일부: {chunk}")
    return {"status": "success"}
```

---
## ② 클라이언트 컨텍스트 추적 (`request.state`)

- `request.state`는 **하나의 요청이 유지되는 동안 자유롭게 데이터를 공유할 수 있는 커스텀 저장소**다.
- 미들웨어(Middleware)나 의존성 주입(Dependency) 단계에서 인증된 유저 정보, 데이터베이스 세션, 요청 시작 시간 등을 `request.state.user = user` 형태로 저장해 두면, 이후 엔드포인트 라우터 함수 어디에서나 `request` 객체 하나로 그 컨텍스트를 꺼내 쓸 수 있다.

```python
@app.middleware("http")
async def add_user_to_state(request: Request, call_next):
    # 미들웨어에서 인증 후 state에 유저 정보 저장
    request.state.current_user = "Kim"
    response = await call_next(request)
    return response

@app.get("/profile")
async def get_profile(request: Request):
    # 라우터에서 간단하게 꺼내 쓰기
    return {"user": request.state.current_user}
```

---
## ③ `request.receive` (ASGI 로우레벨 이벤트 직접 수신)

- ASGI 규격에 따라 클라이언트로부터 들어오는 **원시 이벤트(http.request, http.disconnect 등)를 직접 비동기로 리스닝**한다.
- 클라이언트가 데이터를 보내다가 **중간에 연결을 끊었는지(Disconnect) 실시간으로 감지**할 수 있다. 
- 무거운 DB 쿼리나 AI 연산 중에 클라이언트가 브라우저 창을 닫으면, 이를 감지해 서버 작업을 즉시 취소(Abort)하여 자원을 아낄 수 있다.

```python
@app.post("/heavy-task")
async def heavy_task(request: Request):
    # 내부적으로 클라이언트의 연결 상태를 포함한 ASGI 메시지를 직접 수신
    # 예: 클라이언트가 연결을 끊으면 receive()를 통해 disconnect 이벤트를 잡을 수 있음
    receive_event = await request._receive() 
    return {"message": "Low-level event captured"}

```

---
## ④`request.scope` (연결의 모든 메타데이터 원본 접근)

- HTTP 요청의 헤더나 바디뿐만 아니라, **이 요청이 들어온 ASGI 커넥션의 모든 환경 정보(Context)가 담긴 딕셔너리**다.
- 라우터에 도달하기 전, 라우팅 경로 패턴(`endpoint`), 현재 연결이 HTTP인지 WebSockets인지 여부, ASGI 프로토콜 버전, 클라이언트의 Raw IP와 포트 번호 등 요청의 '뿌리'에 해당하는 모든 정보를 읽고 수정할 수 있다.
- APM(성능 모니터링) 툴이나 글로벌 보안 필터를 만들 때 핵심이 된다.

---

# # FastAPI.Response 클래스 기본 사용법

## ① 백그라운드 작업 예약 (`response.background_tasks`)

- FastAPI의 `Response` 객체는 **클라이언트에게 응답을 즉시 보낸 직후, 서버에서 무거운 작업을 뒤이어 실행하도록 예약**하는 기능을 내장하고 있다.
- 사용자는 요청 결과를 바로 받아서 대기 시간이 없고, 서버는 응답 직후 이메일 전송, 푸시 알림, 로그 기록, AI 모델 추론 등의 무거운 후처리를 안전하게 실행한다. 
- Celery 같은 복잡한 큐 시스템 없이도 간단한 비동기 백그라운드 처리가 가능합니다.

```python
from fastapi import FastAPI, Response, BackgroundTasks

app = FastAPI()

def send_welcome_email(email: str):
    # 무거운 이메일 전송 로직 (예시)
    print(f"{email}로 환영 이메일 전송 완료!")

@app.post("/signup")
async def signup(response: Response, email: str):
    # 1. 백그라운드 작업 등록
    response.background_tasks = BackgroundTasks()
    response.background_tasks.add_task(send_welcome_email, email)
    
    # 2. 클라이언트는 대기하지 않고 즉시 200 OK 응답을 받습니다.
    return {"message": "회원가입이 접수되었습니다."}

```

---
## ② 동적 미디어 타입 및 응답 직렬화 우회 (`Response` 커스텀)

- FastAPI의 라우터가 기본적으로 딕셔너리를 JSON으로 변환해 주지만, `Response` 클래스를 직접 사용하면 **원하는 데이터 타입(Text, XML, Binary, CSV 등)과 상태 코드를 완전히 동적으로 제어**할 수 있다.
- 특히 대용량 raw 데이터를 반환할 때 FastAPI 내부의 JSON 변환 과정을 완전히 패스(Bypass)하므로, 극도의 성능 최적화가 필요할 때 핵심적인 역할을 한다.

```python
from fastapi import Response

@app.get("/xml-report")
async def get_xml_report():
    xml_data = "<report><status>OK</status></report>"
    # 미디어 타입과 헤더를 직접 날것(Raw) 그대로 제어하여 반환
    return Response(content=xml_data, media_type="application/xml")
```

---
## ③ `StreamingResponse` (서버-전송 이벤트 및 대용량 다운로드)

- `Response` 클래스의 직계 형제이자 동급 기능으로, 서버가 데이터를 한 번에 다 만들어서 보내는 게 아니라 **생성되는 대로 실시간으로 클라이언트에게 밀어내는(Push) 기능**이다.
- **ChatGPT 같은 AI의 실시간 글자 스트리밍(SSE: Server-Sent Events)**, 실시간 대시보드 로그 출력, 혹은 수 기가바이트(GB) 파일 다운로드 시 서버 메모리를 단 1MB도 낭비하지 않고 끊김 없이 전송할 수 있다.

```python
from fastapi.responses import StreamingResponse
import asyncio

async def fake_video_streamer():
    for i in range(10):
        yield f"frame_data_{i}\n"
        await asyncio.sleep(0.5)

@app.get("/video")
async def video_stream():
    # 제너레이터(yield)를 이용해 실시간으로 데이터를 청크 단위로 밀어냅니다.
    return StreamingResponse(fake_video_streamer(), media_type="text/plain")
```

---

## ④ `FileResponse` (Zero-Copy 커널 레벨 파일 전송)

- `Response` 클래스의 특수 확장판으로, 디스크에 있는 파일을 클라이언트에게 보낼 때 최강의 성능을 낸다.
- 파이썬 코드가 파일을 읽어서 메모리에 올린 후 소켓으로 보내는 게 아니라, **운영체제(OS) 커널 레벨의** `sendfile` **시스템 콜을 직접 호출(Zero-Copy)**한다.
- 파이썬 프로세스의 메모리를 전혀 쓰지 않고 OS가 하드디스크에서 네트워크 카드로 데이터를 바로 쏘기 때문에, Nginx 같은 전문 웹서버급의 파일 전송 속도가 나온다.

```python
from fastapi.responses import FileResponse

@app.get("/download")
async def download_file():
    # 파일 경로만 지정하면, OS 커널이 가장 빠른 속도로 전송을 처리합니다.
    return FileResponse("large_dataset.zip", media_type="application/zip")
```
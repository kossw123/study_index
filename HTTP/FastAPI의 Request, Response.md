
## 왜 필요한가?

가장 강력하게 사용되는 곳은 미들웨어에서 Request를 낚아채서 검사, 응답이 나가기 직전에 Response에 공용 헤더를 붙이는 작업을 수행한다.


# 정의

클라이언트의 요청과 서버의 응답을

날것(Raw) 그대로 제어할 수 있게 해주는 “데이터 객체”

내부적으로 starlette.requests.Request와 starlette.responses.Response를 그대로 사용한다.

### Request 객체(클라이언트의 요청 정보)

FastAPI는 알아서 시그니처대로 경로나 쿼리를 알아서 파싱을 해주지만, 그 이외에는 날것의 데이터가 필요하다.

- 날 것(Raw)의 데이터는 다음과 같다.
	- 클라이언트의 IP 주소
	- 쿠키
	- HTTP 헤더 정보
	- 원본 Body 데이터 등

```
from fastapi import FastAPI, Request

app = FastAPI()

@app.get("/items")
async def read_items(request: Request):
	// 클라이언트의 IP와 브라우저 정보(User-Agent)를 직접 추출
	client_ip = request.client.host
	
	user_agent = request.headers.get("user-agent")
	
	return {"ip": client_ip, "browser": user_agent}
```

- Request.client.host: 
	- 요청을보낸 클라이언트의 IP 주소
- Request.client.headers: 
	- HTTP 헤더 정보(dict)
- Request.client.cookies: 
	- 클라이언트가 보낸 쿠키 정보
- await Request.json() / await.Request.body(): 
	- 본문(Body) 데이터를 파싱하지 않은 채 원본 그대로, 혹은 JSON으로 읽기

### Response(서버의 응답 제어)

FastAPI는 함수에서 딕셔너리나 SQLModel 객체를 반환하면 알아서 JSON으로 파싱해주지만, 직접 HTTP 상태코드를 동적으로 바꾸거나, 응답 헤더에 특정 값을 추가하거나, 쿠키를 구워서(Set-Cookie) 보내야 할 때 사용된다.
```
from fastapi import FastAPI, Response

app = FastAPI()

@app.post("/login")
async def login(response: Response):
	# 1. 상태 코드를 201 Created로 동적 변경
	response.status_code = 201
	# 2. 응답 헤더에 보안 토큰 슬쩍 끼워넣기
	response.headers["X-Auth-Token"] = "secret-token-value"
	# 3. 브라우저에 쿠키 굽기
	response.set_cookie(key="session_id", value="abc123xyz", httponly=True)
	return {"message": "로그인 성공"}
```

- Response.status_code: HTTP 상태 코드 지정
- Response.headers["X-Custom-Header"]: 커스텀 응답 헤더 추가
- Response.set_cookie(): 클라이언트 브라우저에 쿠키 저장
- Response.delete_cookie(): 지정된 쿠키 삭제


## 프로젝트에서는 어디에서 사용하는가?

- ASGI(Uvicorn)을 통해 요청된 데이터를 통해 쿠키를 구워 token을 설정한다.


## 관련 개념

#web, #fastapi , #request, #response, #ASGI , #uvicorn, #token
## 왜 필요할까?

- **날것(Raw) 데이터의 제어:** FastAPI는 기본적으로 요청(Request)의 쿼리나 바디 데이터를 Pydantic 모델로 자동 파싱해 주지만, 미들웨어 환경에서 요청을 중간에 낚아채 검사하거나, 응답(Response)이 나가기 직전에 공통 보안 헤더를 주입하고 쿠키를 구워내는 등 **HTTP 메시지의 로우 레벨 영역을 직접 제어**해야 할 때 필수적으로 사용된다.

## 정의

- FastAPI가 제공하는 `Request`와 `Response` 객체는 내부적으로 파이썬 ASGI 표준 프레임워크인 **Starlette의 `starlette.requests.Request`와 `starlette.responses.Response` 객체를 그대로 상속받아 사용**한다.

---

### 1. Request 객체 (클라이언트 요청 정보)

경로 파라미터나 쿼리 매개변수 외에, HTTP 요청 메시지 자체에 담긴 로우 레벨 정보들을 추출할 때 사용한다.

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.get("/items")
async def read_items(request: Request):
    # 1. [주의] 프록시 환경(Nginx 등)을 고려하여 실제 클라이언트 IP 추출
    forwarded_for = request.headers.get("x-forwarded-for")
    if forwarded_for:
        client_ip = forwarded_for.split(",")[0].strip()  # 프록시를 거쳐온 실제 유저 IP
    else:
        client_ip = request.client.host if request.client else "Unknown" # 직통 연결 시 IP
        
    # 2. 직속 속성을 통해 헤더 정보 추출
    user_agent = request.headers.get("user-agent")
    
    return {"ip": client_ip, "browser": user_agent}
```

#### 💡 핵심 Request 속성 및 메서드 사양 정리
- **`request.client.host`**
  - 서버와 직접 TCP 연결을 맺은 대상의 IP 주소이다. 
  - **⚠️ 실무 주의:** 전면에 리버스 프록시(Nginx)나 로드밸런서가 배치된 운영 환경에서는 이 값이 '프록시 서버의 IP'로 채워진다. 따라서 실제 유저 IP를 추적하려면 Nginx 설정과 연동된 **`request.headers.get("X-Forwarded-For")`**를 조회해야 한다.
- **`request.headers`**
  - 클라이언트가 보낸 HTTP 요청 헤더 정보에 접근하는 대소문자 구분 없는(Case-insensitive) 사전형(dict-like) 객체이다.
- **`request.cookies`**
  - 클라이언트가 HTTP 요청에 담아 보낸 모든 쿠키 정보가 담긴 딕셔너리(`dict`)이다.
- **`await request.body()`**
  - 클라이언트가 보낸 HTTP 요청 본문(Body) 데이터를 파싱하지 않은 날것의 **바이트(`bytes`)** 형태 그대로 반환한다.
- **`await request.json()`**
  - HTTP 요청 본문 데이터를 읽어와 **파이썬 딕셔너리(`dict`) 구조로 완전히 역직렬화(Parsing)**하여 반환한다. (본문이 올바른 JSON 형식이 아닐 경우 `JSONDecodeError` 발생)

---

### 2. Response 객체 (서버 응답 제어)

FastAPI는 딕셔너리나 Pydantic 모델을 리턴하면 자동으로 200 OK와 함께 JSON으로 바꾸어 내보내지만, 응답 상태 코드를 동적으로 변경하거나 헤더/쿠키를 직접 제어하고 싶을 때 사용한다.

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.post("/login")
async def login(response: Response):
    # 1. HTTP 상태 코드를 201 Created로 동적 변경
    response.status_code = 201
    
    # 2. 응답 헤더에 커스텀 보안 토큰 주입
    response.headers["X-Auth-Token"] = "secret-token-value"
    
    # 3. 브라우저에 보안 쿠키 설정 (HttpOnly, Secure 규격 준수)
    response.set_cookie(key="session_id", value="abc123xyz", httponly=True, secure=True, samesite="lax")
    
    return {"message": "로그인 성공"}
```

#### 💡 핵심 Response 속성 및 메서드 사양 정리
- **`response.status_code`**: 최종적으로 응답할 HTTP 상태 코드를 동적으로 할당한다.
- **`response.headers[...]`**: 클라이언트에게 보낼 HTTP 응답 헤더에 커스텀 값을 주입한다.
- **`response.set_cookie()`**: 브라우저에 쿠키를 심어준다. (안전한 인증을 위해 `httponly`, `secure` 등 보안 설정을 적극 결합해야 한다.)
- **`response.delete_cookie(key, path, domain)`**: 브라우저에 저장된 특정 쿠키의 만료 기한을 과거로 돌려 즉시 삭제하도록 유도한다.

## 프로젝트에서는 어디에서 사용하는가?

- **인증 토큰(JWT) 제어:** 로그인 엔드포인트에서 인증이 성공했을 때, `Response.set_cookie()`를 호출하여 자바스크립트가 접근할 수 없는 `HttpOnly` 속성의 보안 쿠키에 JWT 토큰을 구워 발급하는 용도로 사용하고 있다.
- **인프라 로그 추적:** 미들웨어 영역에서 모든 `Request` 객체의 헤더를 가로채 프록시가 넘겨준 진짜 유저 IP와 고유 요청 ID(`X-Request-ID`)를 추출한 뒤, 로그 파일에 통합 기록하는 모니터링 아키텍처 구축에 활용 중이다.

## 관련 개념

#FastAPI #RequestObject #ResponseObject #Starlette #ClientIP #XForwardedFor #HttpBody #ResponseHeader

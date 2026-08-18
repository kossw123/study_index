
## 왜 필요한가?

- 외부 서버의 데이터와 기능을 가져올 수 있기 때문이다.
- 내 서버가 다른 외부 사이트를 방문해 정보를 얻어오려면 `requests`와 같은 이동 수단이 필요하다.

## 정의

- 서버 코드에서 특정 URL로 접속해 데이터를 가져올 수 있게끔 하는 라이브러리다.
- HTTP method를 통해 데이터를 가져오거나, 보낼 수 있다.
- 즉, 다른 사이트의 데이터를 가져오거나, 다른 서버에 명령을 내릴 때 쓰는 도구다.


## 주의점

1. 동기식 작동(Sync)
	- 요청을 보내고 응답이 올 때까지 프로그램 실행을 멈추낟.
	- 대규모 비동기 처리가 필요하다면, `httpx`, `aiohttp` 라이브러리를 고려하는게 좋다.
2. 타임아웃 설정
	- 서버가 데드락(Deadlock)이 걸리는 경우, 프로그램이 멈추는 걸 방지하기 위해 timeout을 명시하는 습관이 중요하다.
		- `requests.get(url, timeout=5)`


## FastAPI.Request와 차이점

> 이름은 비슷하지만, 
> `requests`는 서버에 요청을 보내며, 
> `FastAPI.Request`는 클라이언트의 요청을 받는다.

| 구분        | `requests` 라이브러리        | FastAPI의 `Request` 객체   |
| --------- | ----------------------- | ----------------------- |
| **역할**    | HTTP **클라이언트** (요청을 보냄) | HTTP **서버** (요청을 받음)    |
| **방향**    | 내 프로그램 -> 외부 서버         | 외부 클라이언트 -> 내 프로그램      |
| **동작 방식** | 동기(Synchronous) 방식      | 비동기(Asynchronous) 방식 지원 |
| **주요 목적** | API 호출, 크롤링, 웹 스크래핑     | 클라이언트가 보낸 데이터·헤더 해석     |

>즉, `FastAPI.Request`는 서버로 들어온 사용자의 요청 정보를 들여다 보는 내부 객체다.

### 코드로 비교하기

```

## FastAPI

from fastapi import FastAPI, Request

app = FastAPI()

async def create_item(request: Request):
	# 클라이언트가 보낸 정보를 서버에서 추출
	client_host = request.client.host
	headers = request.headers
	body = await request.json()
	
	return {
		"message": "데이터 받음", 
		"client_ip": client_host
	}

//////////////////////////////////////

import requests

# 서버로 데이터를 보냄
url = "http://127.0.0.1"
data = {"name": "notebook", "price": 120000}

response = requests.post(url, json=data)
print(response.json())   # 서버가 준 응답 확인
	
```


## 동작 과정

### 설치

```
$ pip install requests
```

### 사용법

```
import requests

response = requests.get("https://github.com")

print(f"상태 코드: {response.status_code}")

if response.status_code == 200:
	print(response.json())   # JSON 응답을 Dictionary로 파싱
	print(response.text)    # HTML이나 일반 텍스트 결과 출력
```


## 프로젝트에서는 어디에서 사용하는가?

- 토츠페이먼츠와의 `https://api.tosspayments.com/v1/payments/confirm` URL로 통신을 하기 위해 필요하다.

```
response = requests.post(
		"https://api.tosspayments.com/v1/payments/confirm",
		headers={
			"Authorization": f"Basic {auth}",
			"Content-Type": "application/json",
		},
		json={
			"paymentKey": request.paymentKey,
			"orderId": request.orderId,
			"amount": request.amount,
		}
	)
```


## 관련 개념

#back , #backend , #api, #json, #server 

























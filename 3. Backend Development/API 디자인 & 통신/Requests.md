## 왜 필요할까?

- **외부 인프라 및 API 생태계 연동:** 외부 타사 서버(결제 게이트웨이, 공공 데이터 포털, 소셜 로그인 등)가 제공하는 데이터와 기능을 HTTP 통신을 통해 내 백엔드 시스템 내부로 가져오거나 명령을 내리기 위해 필요하다.

## 정의

- 파이썬에서 가장 널리 사용되는 **동기식(Synchronous) HTTP 클라이언트 라이브러리**이다.
- 복잡한 로우 레벨 소켓 통신이나 HTTP 프로토콜 서식을 추상화하여, `get()`, `post()`, `put()`, `delete()` 등 직관적인 메서드로 외부 서버와 통신할 수 있게 돕는 도구이다.

---

## ⚠️ 실무 적용 시 핵심 주의점

### 1. 스레드 블로킹(Blocking)의 범위와 FastAPI 비동기 환경 주의
- `requests`는 요청을 보낸 후 응답이 완전히 올 때까지 **현재 실행 중인 실행 흐름(Thread)을 멈추게(Blocking) 만든다.** 프로그램 전체나 컴퓨터가 멈추는 것은 아니며, 해당 요청을 처리하는 스레드만 대기 상태가 된다.
- **FastAPI 결합 시 위험성:** 싱글 스레드 이벤트 루프 환경인 FastAPI의 `async def` 라우터 함수 내부에서 `requests`를 직접 사용하면, **이벤트 루프 자체가 블로킹되어 동시간대에 접속한 다른 모든 유저들의 서비스까지 함께 마비**된다. 
- **해결책:** 대규모 비동기 외부 통신이 필요하다면 비동기 표준 클라이언트 라이브러리인 **`httpx`**나 **`aiohttp`**를 사용해야 한다.

### 2. `timeout` 설정의 정확한 매커니즘
- 외부 서버의 데드락이나 무한 대기로 인해 우리 WAS 스레드가 고사하는 것을 막기 위해 타임아웃 설정은 필수이다. (예: `requests.get(url, timeout=3.0)`)
- **주의:** 이는 전체 파일 다운로드 시간 제한이 아니라, 서버와 연결을 맺는 시간(Connect Timeout)과 서버가 요청을 받아 첫 데이터를 넘겨줄 때까지의 시간(Read Timeout)에 대한 제한이다.

---

## FastAPI `Request` 객체 vs `requests` 라이브러리 차이

| 비교 항목 | `requests` 라이브러리 | FastAPI의 `Request` 객체 |
| :--- | :--- | :--- |
| **역할 본질** | **HTTP 클라이언트** (요청을 만들어 보냄) | **HTTP 서버 내부 컴포넌트** (요청을 받아 해석) |
| **통신 방향** | 내 백엔드 프로그램 ──> 외부 타사 서버 | 외부 클라이언트(브라우저) ──> 내 백엔드 프로그램 |
| **동작 패러다임** | 동기식(Synchronous) 스레드 블로킹 구조 | 비동기식(Asynchronous) 이벤트 루프 논블로킹 지원 |
| **주요 활용** | 타사 비즈니스 API 호출, 웹 스크래핑 및 크롤링 | 유저가 보낸 IP, 헤더, 쿠키, 날것의 바디 데이터 분석 |

---

## 동작 과정 및 올바른 데이터 처리

### 🛠️ 데이터 타입별 올바른 사용법
```python
import requests

# 1. 일반 웹 페이지 요청 (응답 결과가 HTML 텍스트인 경우)
response = requests.get("https://github.com", timeout=5.0)
print(f"상태 코드: {response.status_code}")

if response.status_code == 200:
    # ⚠️ [교정] 깃허브 메인은 HTML이므로 .json()을 쓰면 에러가 납니다. .text를 사용해야 합니다.
    print(response.text[:500]) 

# 2. 전형적인 API 요청 (응답 결과가 JSON 데이터인 경우)
api_response = requests.get("https://github.com", timeout=5.0)
if api_response.status_code == 200:
    # Content-Type이 application/json이므로 안전하게 딕셔너리로 파싱 가능
    user_data = api_response.json()
    print(user_data["name"])
```

---

## 프로젝트 실무 적용: 토스페이먼츠 결제 승인 파이프라인

결제와 같은 금융/미션 크리티컬 시스템 연동 시에는 **네트워크 타임아웃, 응답 유실, HTTP 에러 상태에 대한 예외 처리가 완벽히 결합**되어야 사고를 막을 수 있다.

```python
import requests
from requests.exceptions import Timeout, RequestException

def confirm_payment(payment_key: str, order_id: str, amount: int, auth_credential: str):
    url = "https://api.tosspayments.com/v1/payments/confirm"
    
    headers = {
        "Authorization": f"Basic {auth_credential}",
        "Content-Type": "application/json",
    }
    payload = {
        "paymentKey": payment_key,
        "orderId": order_id,
        "amount": amount,
    }
    
    try:
        # Connect Timeout 3초, Read Timeout 10초로 정밀 설정
        response = requests.post(url, headers=headers, json=payload, timeout=(3.0, 10.0))
        
        # 200계열이 아닌 HTTP 에러 상태 코드(4XX, 5XX) 발생 시 자동으로 HTTPError 예외 발생
        response.raise_for_status()
        
        return response.json() # 결제 완료 데이터 반환

    except Timeout:
        # ⚠️ [네트워크 타임아웃 및 응답 유실 처리 지점]
        # 사용자 돈은 출금되었으나 망 상호작용 지연으로 응답만 못 받았을 가능성이 존재함
        # 즉시 에러를 내면 안 되며, 토스페이먼츠 '결제 조회 API'를 다시 호출해 실결제 여부를 대조 검증해야 함
        log_critical_error("결제 승인 타임아웃 발생 - 실결제 여부 재조회(망 취소 혹은 확인) 프로세스 가동")
        return handle_payment_timeout(order_id)
        
    except RequestException as e:
        # 인증 실패(401), 서버 내부 장애(500) 등 기타 통신 실패 에러 처리
        log_error(f"토스페이먼츠 통신 실패 에러: {str(e)}")
        raise PaymentApprovalException("결제 외부 대행사 통신 오류로 승인이 실패했습니다.")
```

## 관련 개념

#Requests #FastAPIRequest #Synchronous #ThreadBlocking #TimeoutSpecification #TossPayments #ExceptionHandling

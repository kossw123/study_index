## 왜 필요한가?

- [복잡하고 거대한 명세]
	- 00년대 초반에도 RPC 프레임워크들은 있었지만, 구현하기가 너무나 어렵고 복잡했다.
- [플랫폼 종속성]
	- 서로 다른 언어간 통신이 매우 까다롭다.
- [방화벽 통과 문제]
	- RPC들은 임의 포트를 사용하여, 기업의 보안 방화벽을 통과하기 어려웠다.
## 정의

> google에 서 만든 RPC(Remote Procedure Call) 프레임워크다.
---
## 동작 과정

### 0. gRPC의 전체 동작 과정

```
Order Service
      │
      │ confirm_payment()
      ▼
gRPC Client Stub
      │
      │ Protobuf serialization
      │ HTTP/2
      ▼
Payment Service
      │
      │ Protobuf deserialization
      ▼
PaymentService.confirm_payment()
      │
      ▼
Payment 처리
      │
      ▼
PaymentResponse
      │
      │ Protobuf
      ▼
Order Service
```
### 1. Protocol Buffers 작성

- 보통 `.proto`라는 파일을 작성함으로써 일종의 계약서를 만든다.
```
syntax = "proto3";

service PaymentService {
    rpc ConfirmPayment(PaymentRequest)
        returns (PaymentResponse);
}

message PaymentRequest {
    string order_id = 1;
    int64 amount = 2;
}

message PaymentResponse {
    string payment_id = 1;
    string status = 2;
}
```

- REST에서는 보통 JSON을 사용하여, 사람이 읽기 쉽지만 타입이 잘못 들어갈 수 있다.
- 그러나 `.proto`에서는 타입을 명시하기에 이런 일이 발생하지 않는다.
```
string order_id = 1;
int64 amount = 2;
```

### 2. 코드 자동 생성

- 개념적으로는 gRPC 도구가 코드를 다음과 같이 생성한다.
```
payment.proto
      │
      ▼
gRPC compiler
      │
      ├── Client 코드
      └── Server 코드
```
- 그래서 개발자는 매번 HTTP 요청을 직접 구성하는게 아닌 `client.confirm_payment(...)`와 같은 형태로 서비스를 호출할 수 있다.
---
## 장점

### 1. 강력한 계약
- `.proto` 파일 자체가 강력한 계약의 역할을 한다.
```
rpc ConfirmPayment(PaymentRequest)
    returns (PaymentResponse);
```

### 2. 강한 타입
```
message PaymentRequest {
    string order_id = 1;
    int64 amount = 2;
}
```
- API 계약 자체에 타입이 존재한다.

### 3. 빠른 직렬화
- JSON보다 기계 간 통신에 최적화된 형태로 데이터를 전달할 수 있다.
```
JSON

{
    "order_id": "abc",
    "amount": 10000
}
```

### 4. 코드 생성
- `.proto` 하나를 기준으로 여러 언어의 코드를 만들 수 있다.
```
Python
Go
Java
C#
C++
```
- 서로 다른 언어로 작성된 서비스끼리 통신할 수 있게 된다.

### 5. streaming
- HTTP2를 기반으로 강력한 기능들을 사용할 수 있는데, 그 중 스트리밍은 양방향 통신이 가능하게 하고, 실시간 데이터 전송에 굉장히 강하다.
---
## 단점

### 1. 브라우저와 잘 맞지 않는다.
- 보통 gRPC는 브라우저에서 REST보다 훨씬 불편하다.
- 그래서 일반적으로는 다음과 같이 사용한다.
```
React
   │
   │ REST/HTTP
   ▼
API Server
   │
   │ gRPC
   ▼
Internal Services
```
### 2. 디버깅의 어려움
```
REST

POST /payment/confirm
```
- REST는 사람이 읽기 쉬운 형태의 규칙이기 때문에 디버깅에 문제가 없다.
- 반면 gRPC + Protobuf는 사람이 읽기 어려워 외부 플러그인 요소가 필요할 수 있다.
### 3. 네트워크 장애의 존재
- 실제로 RPC는 다음과 같이 동작한다.
```
함수 호출
   ↓
네트워크
   ↓
DNS
   ↓
Load Balancer
   ↓
Payment Service
   ↓
DB
```
- 즉, 아무리 단순하게 만들어도, 네트워크를 통해 작동되기 때문에 timeout, retry, connection 장애가 발생된 가능성이 높다.
---
## RPC보다 gRPC가 좋은 이유

- [범용성과 상호 운용성]
	- Protocol Buffers라는 표준화된 IDL을 사용해, 거의 모든 언어간 통신을 완벽하게 지원한다.
- [HTTP2 채택]
	- 방화벽 친화적:
		- 웹 표준 포트를 그대로 쓸 수 있어, 레거시 환경과 완벽하게 호환된다.
	- 성능 극대화:
		- HTTP2의 멀티플렉싱, 헤더 압축 기술로 인한 통신 효율 극대화
- [스트리밍 기능 내장]
	- HTTP2가 지원하는 서버, 클라이언트, 양방향 스티리밍을 프레임워크에서 기본으로 제공한다.

---
## 간단한 python 예시

- 다음과 같은 `.proto`를 만든다고 가정해보자.
```
syntax = "proto3";

service PaymentService {
    rpc ConfirmPayment(PaymentRequest)
        returns (PaymentResponse);
}

message PaymentRequest {
    string order_id = 1;
    int64 amount = 2;
}

message PaymentResponse {
    string payment_id = 1;
    string status = 2;
}
```
- 이러면 python에서는 생성된 코드를 기반으로 서버를 구현할 수 있다.
```
class PaymentService:
    async def ConfirmPayment(self, request, context):
        order_id = request.order_id
        amount = request.amount

        # 결제 처리
        payment = ...

        return PaymentResponse(
            payment_id=payment.id,
            status="PAID"
        )
```
- client는 다음과 같이 구현할 수 있다.
```
response = await client.ConfirmPayment(
    PaymentRequest(
        order_id="ORDER-123",
        amount=10000
    )
)
```

### gRPC가 잘 맞는 경우

```
Service ↔ Service
```

특히:

- 서비스 간 호출이 많음
- 높은 성능이 중요함
- 강한 타입 계약이 중요함
- 여러 언어를 사용하는 조직
- Streaming이 필요함
- 내부 API가 많음

---

### REST가 잘 맞는 경우

```
Browser ↔ Server
Mobile ↔ Server
Public API ↔ Client
```

특히:

- 브라우저에서 직접 사용
- 사람이 API를 직접 확인해야 함
- 외부 개발자에게 공개
- 단순한 CRUD API
- 디버깅 편의성이 중요

## 관련 개념

#RPC , #server, #back , #backend
## 왜 필요할까?

- **레거시 RPC 기술의 한계 극복:** 90~00년대 초반에도 분산 시스템을 위한 RPC 기술(CORBA, SOAP, XML-RPC 등)이 존재했으나, 명세가 극도로 비대하고 구현 난이도가 높았으며 언어 간 호환성이 떨어졌다.
- **방화벽 및 네트워크 병목 해결:** 과거 RPC들은 표준 웹 포트(80/443)가 아닌 임의의 포트를 사용하여 기업 보안 방화벽을 통과하기 까다로웠다. 또한, 대규모 MSA 구조로 전환되면서 텍스트 기반(JSON/XML) 통신의 직렬화 오버헤드가 백엔드 서버의 주요 병목으로 대두되었다.

## 정의

- **gRPC (Google Remote Procedure Call)**는 구글이 개발하여 오픈소스로 공개한 **고성능, 오픈소스 범용 RPC(원격 프로시저 호출) 프레임워크**이다.
- 서비스 간 통신 인터페이스를 명확히 규정하는 IDL(인터페이스 정의 언어) 표준 규격으로 **Protocol Buffers**를 채택하여 사용한다.

---

## 동작 과정 (Protocol Buffers 계약 및 자동 생성)

### 1. Protocol Buffers 사양서 작성 (`.proto`)
서비스 간 상호작용을 위한 인터페이스 계약서 역할을 수행하는 파일이다.

```protobuf
syntax = "proto3";

service PaymentService {
    // 원격 함수 선언
    rpc ConfirmPayment(PaymentRequest) returns (PaymentResponse);
}

// 전송 데이터 규격 정의
message PaymentRequest {
    string order_id = 1;  # 필드 태그 번호 할당 (이진 직렬화에 활용)
    int64 amount = 2;
}

message PaymentResponse {
    string payment_id = 1;
    string status = 2;
}
```
> ⚠️ **[실무 주의] 기술적 타입과 비즈니스 유효성의 분리**
> `.proto` 파일의 타입 명시(`int64`, `string`)는 데이터의 네트워크 전송 규격과 이진 압축을 보장할 뿐이다. 들어온 값이 빈 문자열인지, 금액이 음수인지 등의 **비즈니스 도메인 유효성(Validation) 및 업무적 정합성은 보장하지 않으므로**, 애플리케이션 계층에서 별도의 검증 로직을 반드시 결합해야 한다.

### 2. 컴파일러를 통한 소스코드 자동 생성
작성한 `.proto` 파일을 gRPC 컴파일러(`protoc`)에 입력하면, 개발자가 네트워크 레이어 코드를 직접 짜지 않아도 원격 호출을 대리해 주는 **클라이언트 스텁(Stub)**과 **서버 베이스 클래스 코드**가 지정한 언어별로 자동 생성된다.

---

## 1세대 레거시 RPC 대비 gRPC의 기술적 우위

gRPC는 단순 원격 호출 패턴을 넘어, 현대적인 네트워크 인프라 기술을 집약하여 과거 RPC의 한계를 극복했다.

1. **표준적 상호 운용성:** 특정 언어에 종속되지 않고 공식적으로 Python, Go, Java, C++, C# 등 다양한 엔터프라이즈 언어 간의 크로스 커뮤니케이션을 완벽히 지원한다.
2. **`HTTP/2` 전면 채택을 통한 성능 극대화:**
   - **방화벽 친화적:** 웹 표준 포트(80/443)를 그대로 사용하여 유연하게 트래픽 관문을 통과한다.
   - **멀티플렉싱(Multiplexing):** 하나의 TCP 커넥션 안에서 수많은 요청과 응답을 동시에 병렬로 주고받아 커넥션 자원 고갈을 막는다.
3. **바이너리 직렬화 파이프라인:** 텍스트 포맷인 JSON과 달리 데이터를 조밀한 이진 바이너리(Binary) 데이터로 압축하여 전송하므로, CPU 연산 비용과 대역폭 소모가 획기적으로 줄어든다.
4. **네이티브 스트리밍(Streaming) 지원:** 단일 요청-응답 구조를 넘어 클라이언트 스트리밍, 서버 스트리밍, 양방향 스트리밍(Bidirectional Streaming)을 프레임워크 핵심 기능으로 기본 탑재하고 있다.

> ⚠️ **인프라 환경에 따른 레거시 호환성 한계 주의**
> gRPC가 표준 포트를 활용하더라도 백엔드 네트워크망 내부에서 **`HTTP/2` 프로토콜의 특수 헤더(Trailing Headers 등)를 지원하지 못하는 구형 L4/L7 로드밸런서, 프록시, 기업 방화벽 장비 등을 통과할 경우 연결이 유실되는 인프라적 제약**이 따를 수 있다. 도입 전 내부 인프라의 `HTTP/2` 지원 여부 검증이 선행되어야 한다.

---

## 파이썬 아키텍처 예시 (동작 이해를 위한 개념 코드)

> ※ 본 코드는 실제 구동을 위한 채널 서버 오픈 및 서비스 등록부(`add_to_server`)가 생략된 핵심 비즈니스 흐름 이해용 예제입니다.

### 1. Server-side 로직 구현
```python
# 자동 생성된 베이스 스텁 클래스를 상속받아 비즈니스 구현
class PaymentServiceServicer(PaymentService_pb2_grpc.PaymentServiceServicer):
    async def ConfirmPayment(self, request, context):
        # 1. 기술적 타입 검증은 끝났으나 비즈니스 유효성 검증은 필요함
        if request.amount <= 0:
            context.set_code(grpc.StatusCode.INVALID_ARGUMENT)
            context.set_details("결제 금액은 0원보다 커야 합니다.")
            return PaymentResponse()

        # 2. 유스케이스 및 비즈니스 연산 실행
        order_id = request.order_id
        amount = request.amount
        
        return PaymentResponse(payment_id="PAY-999", status="SUCCESS")
```

### 2. Client-side 호출부
```python
# 네트워크 구조를 신경 쓰지 않고 마치 로컬 객체처럼 호출
try:
    response = await client.ConfirmPayment(
        PaymentRequest(order_id="ORD-123", amount=10000)
    )
    print(f"결제 성공 상태: {response.status}")
except grpc.RpcError as e:
    # ⚠️ RPC는 본질이 네트워크 통신이므로 예외 처리가 필수 수반됨
    print(f"원격 호출 실패 사유: {e.details()}")
```

---

## 실무 배치 가이드라인: gRPC vs REST 아키텍처

현대 엔터프라이즈 웹 아키텍처에서는 두 기술을 대립 구조가 아닌, **영역별 협력 구조**로 배치한다.

```
[Web Browser / Mobile App]  
│  
│ REST API (HTTP/1.1 + JSON) ➔ 디버깅 용이, 웹 표준 준수  
▼  
[API Gateway / BFF]  
│  
│ gRPC (HTTP/2 + 바이너리 Protobuf) ➔ 초고속, 내부망 보안 및 계약 강제  
├────────────────────────┐  
▼ ▼  
[Order Microservice] ────> [Payment Microservice]
```

- **gRPC가 완벽히 들어맞는 영역 (`서버 ↔ 서버` 내부 통신):** 
	- MSA 내부 백엔드 서비스 간의 대규모 통신, 성능과 대역폭 최적화가 극도로 중요한 시점, 다국어 아키텍처 팀 간의 명확한 타입 계약 보장이 필요할 때 백엔드 통신 중추로 활용한다. 
- **REST가 완벽히 들어맞는 영역 (`클라이언트 ↔ 서버` 외부 통신):** 
	- 브라우저 환경에서 직접 백엔드를 호출할 때(브라우저는 전통적으로 gRPC의 완전한 기능을 쓰기 제약됨), 외부 타사 개발자에게 퍼블릭 API를 오픈할 때, 단순한 CRUD 성격의 엔드포인트를 빠르게 노출해야 할 때 전면 관문으로 활용한다. 

## 관련 개념 
#gRPC #ProtocolBuffers #IDLSpecification #BinarySerialization #HTTP2Multiplexing #InternalServiceCommunication #InfrastructureArchitecture
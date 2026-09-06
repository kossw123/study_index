## 왜 필요한가?

> 핵심은 결합도(Coupling)다.
### **[문제점]**
- 하나의 도메인의 Application Layer에서 여러 서비스가 혼재한다고 가정해보자.
```
Order
 ├── Payment
 ├── Reward
 ├── Notification
 ├── Shipping
 └── Analytics
```

- 만약 Order 도메인이 모든 서비스를 호출한다면 코드가 길어질 것이다.
```
def create_order():
    order = create()

    payment.pay(order)
    reward.add_point(order)
    notification.send(order)
    shipping.prepare(order)
    analytics.record(order)
```

- 이때 시간이 지나, 유지보수 및 기능 추가를 위해 하나의 도메인을 추가한다고 가정해보자.
```
OrderService
    │
    ├── Payment
    ├── Reward
    ├── Notification
    ├── Shipping
    ├── Analytics
    ├── ...
    └── ...
```
- 결국 하나의 객체에서 코드가 폭발하는 대참사가 발생한다.
### [EDA의 해결책]

- 도메인과 결합 또는 합성된 `Event Broker`만 서비스들을 알고, Order 도메인은 더 이상 서비스에 대한 정보를 알지 못하여, 응집도(Cohesion)가 향상된다.
```
Order
 │
 │ OrderCreated
 ▼
Event Broker
 │
 ├──> Payment
 ├──> Reward
 ├──> Notification
 ├──> Shipping
 └──> Analytics
```
## 정의

> 시스템을 어떤 구조로 만들 것인가에 대한 개념이다.
> 발생한 사건(Event)을 전달하고, 사건과 관련된 객체만 반응하게 하는 아키텍처 스타일이다.

- 절차적인 시스템 구조는, 오직 시간의 흐름대로 서비스를 호출한다.
- 그러나 EDA는 어떤 서비스에 관련된 객체들에게만 사건(Event)가 발생했음을 알리고, 각 객체는 정해진 흐름에 따라 동작하도록 만든다.
## 동작 과정

```
Producer
   │
   │ Event
   ▼
Event Broker
   │
   ├──────────┐
   ▼          ▼
Consumer   Consumer
```

1. **[Producer]**
	- 이벤트를 발생시키는 쪽
2. **[Event]**
	- **"무슨 일이 발생했다."** 를 표현하는 메시지
3. **[Consumer]**
	- 이벤트를 듣고 자신의 작업을 수행하는 쪽

## 장점

#### 1. **[시간적 결합(Temporal Coupling)]** 이 줄어든다.

> 시간적 결합(Temporal Coupling)이란?
> 어떤 코드나 메서드의 실행이 특정 시간이나 순서에 의존하는 현상을 의미한다.

- 표준적인 동기 호출에서는 도메인의 실행 프로세스에서, 이전 프로세스가 완료되어야 다음 단계로 넘어간다.
- 그러나 EDA에서는 프로세스에서, 이벤트를 발행한 뒤에 따로 작업이 없다면 다른 작업을 실행한다.

### 2. [낮은 결합도 (Low Coupling)]
- 도메인이 다른 도메인을 직접 알 필요가 없다.
- 즉, 한 도메인에서 다른 도메인이 합성되거나, 결합되는 일이 없어진다.

### 3. [확장성]이 좋다.
- 새로운 기능이 필요하다면, 도메인의 기존 코드를 건드리지 않고, Consumer를 추가하기만 하면 된다.
```
OrderCreated
      │
      ├── Payment
      ├── Reward
      ├── Notification
      └── Analytics ← 새로 추가
```

### 4. [비동기 처리]
- 사용자 요청에서 반드시 기다릴 필요가 없는 작업은 이벤트로 넘길 수 있다.
```
주문 생성
   ↓
응답

"주문이 생성되었습니다."

        ↓
   비동기 처리
        ↓
이메일
포인트
통계
알림
```

## 단점

### 1.디버깅 어려워짐
- 하나의 사용자 요청이 여러 서비스와 여러 이벤트로 분산되어, 로그를 추적하기 어려워진다.
- 실무에서는 추적할 수 있는 변수를 통해 분산 추적을 한다.
	- `trace_id`
	- `correlation_id`
	- `event_id`

### 2. [중복 처리]
- 이미 발생된 이벤트가 장애가 발생하면, 메시지가 다시 전달될 수 있다.
- 그래서, 반드시 `Consumer`는 가능하면 멱등성(Idenpotency)를 가져야 한다.

### 3. [이벤트 순서]
- 아무리 EDA가 비동기 처리에 뛰어나지만, 결국은 사용자의 편의를 위해 흐름을 가져야 한다.
- 그러나, 시간순에 따른 이벤트 중 어떤 이벤트가 장애 때문에 발생되지 않거나, 늦는다면 상태가 이상해진다.


### EDA, EDD, DDD

```
///////////////////
EDA = 아키텍처    
EDD = 개발 방법론
///////////////////


DDD
 ↓
도메인을 중심으로 설계

EDA
 ↓
이벤트를 중심으로 시스템 간 상호작용 설계

EDD
 ↓
이벤트를 중심으로 개발/설계하는 접근

```

## DDD와 EDA

> DDD는 시스템에서의 비즈니스 개념과 경계를 어떻게 나눌 것인가에 대한 개발 방법론이다.
> EDA는 나뉜 시스템들이 어떻게 서로 통신할 것인가에 대한 구조다.

## 관련 개념

#EDA, #DDD, #back , #backend , 
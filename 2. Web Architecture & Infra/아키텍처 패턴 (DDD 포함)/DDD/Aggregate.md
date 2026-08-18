## 왜 필요한가?

- [**비즈니스 규칙(불변식)의 강제**]
	- [Problem.]
		- aggregate가 없으면 데이터 변경 로직이 사방으로 흩어진다.
	- [Solution.]
		- aggregate root라는 단 하나의 객체를 통해서 데이터 수정을 허용하면 해결할 수 있다.
- [**데이터 격리와 트랜잭션 범위 축소**]
	- [Problem.]
		- 시스템에서 동시 요청이 올 때 시스템 전체가 마비될 수 있다.
	- [Solution.]
		- aggregate는 데이터 변경의 **최소 단위**다.
		- 딱 필요한 만큼만 묶어 격리하기에, DB Lock의 범위가 줄어들고, 동시 처리 능력이 향상된다.
- [**복잡한 도메인의 단순화(경계설정)**]
	- [Problem.]
		- 거대한 시스템에서는 엔티티(Entity)가 서로 복잡하게 얽히게 되는데, 코드를 한 줄 고쳐도 어디까지 영향이 갈지 예측하기 불가능해진다.
	- [Solution.]
		- 관련 객체를 aggregate라는 울타리로 묶어 경계를 긋는다.
		- 개발자는 aggregate 로직만 신경쓰면 되기에, 도메인 복잡도가 획기적으로 낮아진다.
- [**기술 표준화 및 구현의 편의성**]
	- [Problem.]
		- DB에 저장하거나 불러올 때 어디서부터 어디까지 조회하고 저장(Persist)해야 할지 기준이 모호해진다.
	- [Solution.]
		- **"하나의 aggregate는 통째로 저장소(Repository)에 저장되고 통째로 불러온다"**
		- 이런 명확한 기준이 생기기에, ORM이나, NoSQL DB를 사용할 때 명확한 기준이 된다.

## 정의

> **[데이터 변경의 단위]를 의미한다.**
> 
> 여기서 데이터 변경의 단위는 비즈니스 규칙을 깨뜨리지 않고, 
> 한 번에 반드시 함께 수정되어야 하는 객체들의 묶음을 의미한다.


### Aggregate의 구성 요소

> 보통 하나의 Aggregate Root와 그 내부의 엔티티(Entity) 및 밸류 객체(Value Object)들로 구성된다.

```

# Base Class
class AggregateRoot:
    def __init__(self):
        self.events = []
    def register(self, event: IEvent):
        self.events.append(event)
    def pull_events(self):
        events = self.events
        self.events = []
        return events


# 개념적/비즈니스적 Aggregate Root
class Order:
    def __init__(self,
                 order_id: UUID,
                 customer_id: UUID,
                 items: list[OrderItem]):
		self.root = AggregateRoot()
        self.order_id = order_id
        self.customer_id = customer_id
        self.items = items
        self.status = OrderStatus.CREATE
```

> 여기서 AggregateRoot 객체는 이벤트를 발행하는 객체로써, AggregateRoot가 내부 객체들을 안전하게 다 바꾼 다음, 변경 완료를 외부에 알리는 객체다.


- Aggregate Root
	- Aggregate의 대표 엔티티
	- 외부 객체는 오직 이 루트를 통해서만 내부 객체에 접근할 수 있다.
	- 내부 객체를 보호하고 비즈니스 논리를 강제하는 문지기 역할을 한다.

>여기서 Aggregate Root는 **"Order"** 다.

- 내부 엔티티(Entity) 및 VO(Value Object)
	- 루트에 종속적인 객체들이다.
	- 외부에서 직접 이 객체들의 상태를 변경할 수 없다.

> `order_id`, `customer_id`, `status`등이 내부 엔티티고, `items`가 VO다.

## 관련 개념

#ddd , #architect , #domain, #backend 









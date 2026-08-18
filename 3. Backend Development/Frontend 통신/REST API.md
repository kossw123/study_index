## 왜 필요한가?

- 서로 다른 프로그램들이 복잡한 설정 없 이, 인터넷 표준 규격(HTTP)만으로 데이터를 주고 받기 위해서 필요하다.
- 다른 플랫폼에서 만들어 둔 기능을 REST API형태로 가져와 서비스에 적용할 수 있다.

## 정의

- 서로 다른 프로그램들이 소통할 수 있도록, HTTP라는 틀 안에서 짜여진 API에 대한 구조를 서술한 것이다.
- 그래서 별다른 프로그램 없이, HTTP라는 틀 안에서 다음과 같이 작성할 수 있다.

 **[일반적인 API 응답]**

```
{
  "accountNumber": "123-456",
  "balance": 50000,
  "status": "ACTIVE"
}
```

- 평범하게 계좌번호, 금액, 상태에 대한 명세를 나타낸 HTTP의 body에 넣을 데이터(JSON)다.

그러나 REST API로 작성한다면 HTTP라는 틀에서 다른 프로그램끼리 순수한 데이터(JSON)을 주고 받는 표준 통신 규칙을 작성할 수 있다.

**[로이 필딩의 REST API 응답]**

```
{
  "accountNumber": "123-456",
  "balance": 50000,
  "status": "ACTIVE",
  
  "_links": {
    "self": { "href": "/accounts/123-456" },
    "deposit": { "href": "/accounts/123-456/deposit" },
    "withdraw": { "href": "/accounts/123-456/withdraw" }
  }
}
```

- 클라이언트는 주소를 알 필요가 없이, 서버가 보내준 응답 속의 `_links.withdraw`를 읽어서 요청을 보낼 뿐이다.
- 계좌가 0원이 되거나 계좌가 정지(`suspended`) 상태가 되면, 서버는 `withdraw` 링크를 보내지 않으면 그만이다.


그러나 오늘날에서는 로이 필딩의 REST API는 무겁기 때문에, 다음과 같이 단순히 주소 + HTTP method 규칙만 차용해 편리하게 사용한다.

|  기능   | HTTP method(행동) | URI(대상 주소)              | 핵심 특징                      |
| :---: | :-------------- | :---------------------- | :------------------------- |
| 결제 요청 | POST            | `/v1/payments`          | 새 결제 건을 생성함(주소에 create 안씀) |
| 내역 조회 | GET             | `/v1/payments/pay_8842` | pay_8842번 결제의 상세 정보를 가져옴   |
| 결제 취소 | DELETE          | `/v1/payments/pay_8842` | 해당 결제 내역을 취소/삭제 처리함        |

## 동작 과정

실제로 다음과 같이 데이터를 주고 받는다.

1. 클라이언트의 요청 (GET)

> "서버님, `pay_8842`번 결제 정보 양식에 맞춰서 데이터만 보내주세요."

```
# http

GET /v1/payments/pay_8842 HTTP/1.1
Host: api.toss.im
Authorization: Bearer test_key_123456
```



2. 서버의 응답 (JSON)

> "네, 여기 요청하신 결제 알맹이 데이터입니다."

```
# json

{
  "paymentId": "pay_8842",
  "orderName": "맥북 에어 M3",
  "amount": 1590000,
  "status": "DONE",
  "approvedAt": "2026-07-16T15:30:00"
}
```






## 관련 개념

#RESTAPI, #architect , #frontend , #backend , #programming 
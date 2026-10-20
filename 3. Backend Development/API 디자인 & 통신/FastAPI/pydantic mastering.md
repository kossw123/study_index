# 왜 필요한가?

- **유효성 검사 및 캐스팅 코드 단축**
	- DDD를 작성하면서 느낀거지만, Application Layer에서 DTO에 대한 유효성 검사 및 형변환이 잦다.
	- 이로 인해 Application Layer쪽 코드의 가독성 및 중복된 조건문으로 비효율적인 코드가 작성된다.

# 어떻게 사용해야 잘 사용하는 건가?

## 기본 사용법
### 1.필드 레벨의 세부 제약조건(Field)

| 옵션                    | 설명     | 예시                          |
| :-------------------- | :----- | :-------------------------- |
| default               | 기본값 설정 | Field(default=0)            |
| gt/lt                 | 크기 비교  | Field(gt=0, lt=100)         |
| ge/le                 | 이상/이하  | Field(ge=0, le=100)         |
| min_length/max_length | 문자열 길이 | Field(min_length=2)         |
| regex/pattern         | 정규식 패턴 | Field(pattern=r"\d+")       |
| description           | 필드 설명  | Field(description="사용자 이름") |

```
class User(BaseModel):
    name: str = Field(..., min_length=2, max_length=50)
    age: int = Field(gt=0, lt=150)
    email: str = Field(..., pattern=r"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$")
```

### 2.복잡한 비즈니스 로직 검증

> `@field_validator` : 특정 필드 하나를 검증 할 때
> `@model_validator` : 여러 필드간의 관계나 모델 전체를 검증할 때
---
```python
# [@field_validator의 사용법]

from pydantic import BaseModel, Field, field_

class User(BaseModel):
    name: str = Field(..., min_length=2, max_length=50)
    age: int = Field(gt=0, lt=150)
    email: str = Field(..., pattern=r"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$")
    
    @field_validator('name')
    @classmethod
    def validate_for_name(cls, v: str) -> str:
	    if ' ' in v:
		    raise ValueError("공백이 포함되었습니다.")
		return v.lower()
```

> **핵심 규칙**
> - `@classmethod`를 사용해야 한다.
> - 입력받은 데이터 구조를 그대로 또는 수정해서 `return`해야 한다.
> - 실패 시 ValueError나 TypeError를 발생 시킨다.
---
```python
# [@model_validator(mode='after')의 사용법]

from pydantic import BaseModel, model_validator

class SignupForm(BaseModel):
  password: str
  confirm_password: str

  # mode='after'는 이미 개별 필드의 검증이 끝난 상태의 인스턴스를 받아 다시 검증한다.
  @model_validator(mode='after')
  def check_passwords_match(self) -> 'SignupForm':
    # 이미 password와 confirm_password가 문자열로 검증된 후 호출됨
    if self.password != self.confirm_password:
      raise ValueError('비밀번호가 일치하지 않습니다.')
    return self
```
> 핵심 규칙
> - 메서드 내에서 `self`인자를 사용한다.
> - 검증이 끝난 후 반드시 `self`를 `return`한다.
---
```python
# [@model_validator(mode='before')의 사용법]

from typing import Any
from pydantic import BaseModel, model_validator

class Product(BaseModel):
  name: str
  price: int

  @model_validator(mode='before')
  @classmethod
  def pre_validate(cls, data: Any) -> Any:
    # 데이터가 딕셔너리로 들어왔을 때, 특정 처리를 미리 수행
    if isinstance(data, dict) and 'price' in data:
      if data['price'] < 0:
        data['price'] = 0  # 음수 가격을 0으로 미리 보정
    return data
```
> 핵심 규칙
> - `@classmethod`를 사용해야 한다.
> - 입력받은 데이터 구조를 그대로 또는 수정해서 반환한다.
### 3. Config를 통한 모델 전반의 동작 설정

```
from pydantic import BaseModel

class DataRequest(BaseModel):
	id: int
	...
	model_config = ConfigDict(
		# 설정 주입
	)
```
> 자주 쓰는 핵심 설정
> 1. **`extra`**: 정의되지 않은 필드 처리 (`'ignore'`, `'forbid'`, `'allow'`). `model_config = ConfigDict(extra='forbid')`와 같이 사용
> 2. **`from_attributes`**: SQLAlchemy 같은 ORM 인스턴스 호환 허용 (`from_attributes=True`).
> 3. **`populate_by_name`**: 별칭과 실제 필드명 동시 지원 (`populate_by_name=True`).
> 4. **`strict`**: 엄격한 타입 검사로 자동 변환 방지 (`strict=True`).
> 5. **`validate_assignment`**: 객체 속성 변경 시 재검증 수행 (`validate_assignment=True`).
---
## 고급 사용법

### 1. Nested(중첩)

- 실제 데이터 구조는 1차원이 아닌, 복잡한 트리 구조를 가진다.
- pydantic은 중첩된 객체들을 deeply nested라 하더라도 단 한번의 호출로 트리 전체를 파싱 및 검증한다.
```python
class Address(BaseModel):
    street: str = Field(..., min_length=5)
    city: str = Field(..., min_length=2)
    country: str = Field(..., min_length=2)

class User(BaseModel):
    name: str = Field(..., min_length=2)
    addresses: List[Address] = Field(default_factory=list, max_items=3)
```
---
### 2. Discriminated Unions(공용체)

- 데이터의 특정 필드 값에 따라 나머지 필드들의 검증 규칙과 모델 자체가 완전히 동적으로 바뀌어야 할 때 사용한다.

```python

# Discriminated Unions 사용 예시

from typing import Literal, Union
from pydantic import BaseModel, Field

class CardPayment(BaseModel):
    type: Literal["card"] # 고유 식별자 (Discriminator)
    card_number: str
    expiry: str

class BankPayment(BaseModel):
    type: Literal["bank"] # 고유 식별자 (Discriminator)
    account_number: str
    bank_code: str

# 최상위 결제 요청 모델
class PaymentRequest(BaseModel):
    # type 필드의 값을 보고 CardPayment로 파싱할지 BankPayment로 파싱할지 Pydantic이 '동적'으로 결정합니다.
    payment_method: Union[CardPayment, BankPayment] = Field(..., discriminator="type")
```

> 왜 강력한가?
> - 이 기능이 없는 경우 상위 클래스에서 모든 필드를 옵셔널(None)로 열어두고 내부 비즈니스 로직에서 `if type == card:` 조건문으로 일일이 수동 검증을 해야 한다.
> - pydantic은 데이터에 따라 알맞은 서브 클래스를 알아서 파싱해준다.

### 3. Annotated Validators(커스텀 타입)

- 기존 python 타입이나 내장 pydantic 기능만으로 표현할 수 없는 특수한 데이터 규격이나 도메인 제약 조건을 변수 선언부 수준에서 완벽하게 캡슐화 하는 기능이다.

```python
from typing import Annotated
from pydantic import BaseModel, AfterValidator

# 이메일 앞자리(Id)만 추출하는 커스텀 로직
def extract_email_id(v: str) -> str:
    if "@" not in v:
        raise ValueError("올바른 이메일 형식이 아닙니다.")
    return v.split("@")[0]

# 나만의 특수 타입 정의: 이 타입을 쓰면 검증이 끝난 후 이메일 ID만 남게 됨
EmailId = Annotated[str, AfterValidator(extract_email_id)]

class UserSession(BaseModel):
    user_id: EmailId # 커스텀 타입 적용
```
> 왜 강력한가?
> - 모델 내부가 아니라 타입 자체에 검증 및 변환 로직이 있어서, `EmailId`라는 타입을 여러 모델, 여러 파일에서 마치 기본타입처럼 사용할 수 있다.
> - 사실상 VO 사상을 가장 완벽하게 코드로 옮긴 기능이다.
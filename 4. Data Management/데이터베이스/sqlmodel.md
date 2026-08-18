
## 왜 필요한가?

## 정의

- 파이썬으로 만들어진 FastAPI와 가장 잘 어울리는 ORM 라이브러리다.
- sqlmodel은 pydantic + sqlalchemy 라이브러리가 합쳐진 라이브러리다.
- 그래서 하나의 클래스만 만들면 DB 테이블도 되고, FastAPI 데이터 검증용 툴도 된다.

e.g)
from sqlmodel import Field, SQLModel
from typing import Optional

class Hero(SQLModel, table=True):
	id: Optional[int] = Field(default=None, primary_key=True) # DB의 PK 역할
	name: str # Pydantic 검증 + DB 컬럼
	secret_name: str
	age: Optional[int] = None

table=True일 경우 DB 테이블로 작동되고, table=False일 경우 pydantic 데이터 검증기로 작동된다.

## 프로젝트에서는 어디에서 사용하는가?

- Application, Infra Layer 간의 사이에 위치(UnitOfWork)에 위치하여 객체 내부에 Repository를 통해 생성, 관리된다.

## 상속된 SQLModel 객체의 정체

SQLModel을 상속시킨다면 pydantic, SQLAlchemy라는 두 가지 프레임워크를 완벽하게 하나로 융합시킨다.

즉, sqlmodel은 pydantic, sqlalchemy 프레임워크의 기능을 사용하는 또 다른 프레임워크다.

그래서 서로 다른 프레임워크를 사용하기 위해 sqlmodel 프레임워크에서는 SQLModel이라는 객체를 상속해서 이를 해결한다.

SQLModel을 상속받으면 파이썬 객체지향 문법(상속)을 100% 활용해 필드를 그대로 물려받을 수 있다.

from sqlmodel import SQLModel, Field

// [기본 베이스] 공통 필드 정의 (table=False가 기본값)
// false면 검증기, true인 경우는 DB용 테이블로써 작동한다.
class UserBase(SQLModel):
	name: str # ⭕ 단 한 번만 정의!
	email: str # ⭕ 단 한 번만 정의!

// [API 검증 모델] 베이스를 상속받아 비밀번호 필드만 슥 추가!
class UserCreate(UserBase):
	password: str

// [DB 테이블 모델] 똑같이 베이스를 상속받고, table=True만 켜서 DB용 필드 추가!
class User(UserBase, table=True):
	id: int | None = Field(default=None, primary_key=True)
	hashed_password: str


## 관련 개념

#db, #sqlmodel, #orm, #validate 
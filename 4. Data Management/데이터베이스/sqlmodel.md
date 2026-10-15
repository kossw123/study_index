## 왜 필요할까?

- **단일 소스 원칙(Single Source of Truth) 구현:** 기존 파이썬 백엔드 환경에서는 외부 요청 검증용 스키마(Pydantic)와 내부 데이터베이스 매핑용 엔티티(SQLAlchemy) 코드를 완전히 똑같은 구조로 중복해서 2번 정의해야 했다. SQLModel은 이 두 레이어의 스키마 선언을 하나로 묶어 코드 생산성을 극대화한다.
- **FastAPI 생태계의 완벽한 일체화:** 유효성 검사, 자동 형변환, IDE 자동완성 및 OpenAPI(Swagger) 문서화 스펙을 단 하나의 통합 데이터 모델 인터페이스로 관통하여 다룰 수 있게 돕는다.

## 정의

- **`SQLModel`**은 구글 소속의 개발자 Tiangolo가 만든 라이브러리로, 파이썬의 독보적인 데이터 검증 툴인 **`Pydantic`**과 강력한 자원 관계 매핑 엔진인 **`SQLAlchemy`**를 코드 레벨에서 완벽하게 하나로 융합하여 설계한 **최신 하이브리드 ORM 프레임워크**이다.

---

## 핵심 매커니즘의 올바른 기술적 이해

### 1. `table=True` 옵션의 본질 (전환 스위치가 아닌 '기능의 확장')
- `table=True` 선언은 Pydantic 기능과 DB 테이블 기능을 완벽히 무로 돌리며 전환하는 스위치가 아니다.
- **`table=False` (기본값):** 순수한 Pydantic의 데이터 검증기, 직렬화 객체(DTO) 역할만 수행한다.
- **`table=True`:** Pydantic의 유효성 검증 엔진을 밑바닥에 그대로 유지한 상태에서, **SQLAlchemy의 데이터베이스 테이블 릴레이션 매핑 메타데이터를 추가로 활성화하고 결합하는 구조**이다. 

### 2. 모델 상속 구조의 아키텍처적 제약 조건
- **권장 아키텍처 (비테이블 ➔ 테이블 분기 상속):** 본 프로젝트에 적용된 방식처럼, `table=False`인 공통 추상 베이스 클래스(`UserBase`)로부터 검증용 DTO 모델과 실제 DB 엔티티 모델을 수평 상속 분기하는 구조는 매우 안전하고 이상적인 아키텍처이다.
- **⚠️ [실무 주의] 테이블 간 상속의 한계:** 이미 `table=True`가 켜진 실제 물리 DB 테이블 클래스를 다른 `table=True` 자식 클래스가 다중 상속받는 구조는 지양해야 한다. 이는 DB 스토리지 엔진 단에서 단일 테이블 구조(Single Table)나 조인 테이블 구조(Joined Table) 같은 복잡한 물리 상속 전략을 강제하므로, 인덱스 효율 저하 및 쿼리 병목의 원인이 된다.

---

## 실전 상속 및 레이어 분리 아키텍처 패턴

클라이언트가 던지는 입력 검증 경로(Request DTO)와 데이터베이스 테이블 저장 경로(Entity)의 목적성 분리를 완벽히 충족하는 SQLModel 최적화 설계 템플릿이다.

```python
from sqlmodel import SQLModel, Field

# 1. [공통 추상 베이스] 순수 비즈니스 필드만 선언 (table=False 기본 상태)
class UserBase(SQLModel):
    name: str  # ⭕ 단 한 번만 정의하여 데이터 명세 단일화
    email: str

# 2. [입력 검증용 API 스키마] 클라이언트 유입 관문 (Request DTO)
class UserCreate(UserBase):
    password: str  # 회원가입 시 클라이언트가 전달하는 원본 비밀번호 수신

# 3. [물리 데이터베이스 매핑 모델] 인프라 저장 엔티티 (Entity)
class User(UserBase, table=True):
    # RDB 영속성 레이어 처리를 위한 물리 키 및 암호화 필드 가동
    id: int | None = Field(default=None, primary_key=True)
    hashed_password: str  # 원본 암호가 아닌 해시 암호 저장용 컬럼 명시
```

## 프로젝트에서는 어디에서 사용하는가?

- **데이터 레이어 코어 컴포넌트:** 프로젝트 내부의 `Domain Layer` 명세 정의부터 `Infrastructure Layer` 데이터베이스 물리 테이블 생성 및 `Presentation Layer` API 엔드포인트 요청/응답 스키마 제어까지, 전 시스템의 데이터 흐름 구조를 제어하는 단일 통합 인터페이스 코어로 채택하여 활용 중이다.

## 관련 개념

#SQLModel #FastAPIORM #PydanticCore #SQLAlchemyCore #TableMappingAttribute #ModelInheritanceStrategy #DataTransferObject #PersistenceEntity

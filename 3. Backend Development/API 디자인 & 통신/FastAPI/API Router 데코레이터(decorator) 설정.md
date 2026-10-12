## 왜 필요할까?

- **응답 데이터의 필터링 및 검증:** 서버가 클라이언트에게 내보내는 출력 데이터의 형태를 엄격하게 제한하고 검증하여 보안을 강화한다.
- **자동 문서화 고도화:** Swagger UI 문서에 API의 제목, 설명, 그룹, 상태 코드 등을 명시하여 프론트엔드 개발자와의 협업 효율을 수월하게 만든다.

## 동작 과정 및 올바른 사용 예시

FastAPI에서 데코레이터에 `response_model`을 지정하면, **함수의 실제 반환값(Return) 역시 해당 모델의 필수 필드 규격을 만족**해야 한다.

```python
from pydantic import BaseModel

# 1. 응답 데이터의 형태를 정의하는 Pydantic 모델
class ProductModel(BaseModel):
    id: int
    name: str
    price: int
    description: str | None = None  # 기본값이 지정된 필드

@router.put(
    "/{product_id}", 
    response_model=ProductModel,                # 응답 데이터를 변환하고 검증할 Pydantic 모델 지정
    status_code=200,                            # 성공 시 반환할 기본 HTTP 상태 코드 (200 OK)
    tags=["Products"],                          # Swagger 문서에서 'Products' 그룹으로 분류
    summary="상품 정보 수정",                     # API의 짧은 제목
    description="상품의 이름, 가격 등을 수정합니다.",  # API의 상세 설명 (마크다운 지원)
    response_model_exclude_unset=True,          # 응답 모델에서 명시적으로 설정되지 않은 필드는 결과 JSON에서 제외
    deprecated=False                            # True 설정 시 문서상 '사용 중단 예정' 표시 (API 자체는 정상 작동)
)
def update_product(product_id: int):
    # 2. [주의] response_model 규격에 맞는 데이터를 반드시 반환해야 함
    return {
        "id": product_id,
        "name": "새로운 상품명",
        "price": 25000
        # description은 누락되었으나, unset 처리되어 최종 JSON 응답에서 자동 제외됨
    }
```

### 💡 핵심 데코레이터 옵션 상세 분석

- **`response_model`**
  - 라우터가 최종 응답을 클라이언트에게 보낼 때 데이터를 필터링하고 유효성을 검사하는 기준 모델이다. 모델에 정의되지 않은 민감한 데이터(예: 비밀번호 필드 등)가 실수로 반환되더라도 자동으로 걸러내어 유출을 방지한다.
- **`response_model_exclude_unset`**
  - **클라이언트의 요청과 무관하게**, 서버가 반환하는 Pydantic 응답 모델 객체 내부에서 **값이 명시적으로 설정되지 않은 필드(기본값 상태로 남아있는 필드 등)를 최종 출력 JSON에서 통째로 제외**시키는 설정이다.
- **`deprecated`**
  - 이 값이 `True`가 되어도 **라우트 자체가 비활성화되거나 차단되지 않는다.** 기능은 정상 작동하되, 오직 **Swagger UI 문서상에 경고 마크와 가로삭선**을 그어주어 클라이언트 개발자들에게 사용 중단을 권고하는 역할을 한다.
- **`status_code`**
  - 비즈니스 로직이 성공적으로 완료되었을 때 기본값으로 부여할 HTTP 상태 코드를 지정한다.

## 프로젝트에서는 어디에서 사용하는가?

- FastAPI의 각 도메인 영역별 엔드포인트(Controller/Router 계층)를 설계할 때 전면적으로 사용 중이다. 
- 모든 엔드포인트에 `response_model`, `summary`, `tags`를 필수로 지정하여 신뢰성 높은 API 검증 체계와 깔끔한 대화형 API 문서(Swagger) 환경을 유지하고 있다.

## 관련 개념

#FastAPI #RouterDecorator #Pydantic #ResponseModel #SwaggerUI #APIValidation

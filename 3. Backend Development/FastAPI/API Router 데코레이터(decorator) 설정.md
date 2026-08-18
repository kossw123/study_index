
## 왜 필요할까?

 - 출력 데이터의 필터링, 검증, 문서화를 설정할 수 있다.
 - 그 결과, API의 안전성이 높아지고 프론트엔드 개발자와 협업이 수월해진다.

## 동작 과정

```
@router.put(
    "/{product_id}", 
    
    # 
    response_model=ProductModel,
    
    status_code=200,           # 성공 시 200 OK 반환
    
    tags=["Products"],         # Swagger 문서에서 'Products' 그룹에 포함
    
    summary="상품 정보 수정",    # API 제목
    
    description="상품의 이름, 가격 등을 수정합니다.", # API 상세 설명
    
    # 클라이언트가 수정 요청 안 한 필드는 응답에서 제외
    response_model_exclude_unset=True,
    
    # 활성화된 API 확인 여부
    deprecated=False                           
)
def update_product(product_id: int):
    return {"message": "수정 완료"}

```

- [`response_model`]
	- API가 클라이언트에게 Response할 때 **"데이터를 어떤 형태로 변환하고 검증할 지 정의"** 하는 설정이다.
	- 즉, 클라이언트에게 응답을 줄 때의 형태를 검증하는 부분이다.
- [`status_code`]
	- 성공 시 반환할 기본 HTTP 상태 코드를 지정한다.
- [`tags`]
	- API들을 그룹으로 묶어준다.
- [`description`]
	- API의 상세한 설명을 마크다운 형식으로 작성할 수 있다.
- [`response_model_exclude_unset`]
	- 모델의 필드 중 값이 명시되지 않은 필드는 제외시킨다.
- [`deprecated]
	- True로 설정하면 이 API가 곧 사라질 예정이라는 표시다.

## 관련 개념

#fastapi , #decorator, #pydantic , #validate 
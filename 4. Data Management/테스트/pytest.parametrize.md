## 정의

> 하나의 테스트 함수로 여러 개의 입력 값과 예상 결과 조합을 반복 테스트할 수 있는 `pytest`의 내장 데코레이터

## 기본 사용법

```
import pytest

# 입력값 a와 기대값 expected를 매개변수화
@pytest.mark.parametrize("a, expected", [
    (1, 2),
    (2, 3),
    (10, 11)
])
def test_increment(a, expected):
    assert a + 1 == expected
```

- [첫 번째 인자:] 매개변수 이름을 문자열로 지정
- [두 번째 인자:] 테스트할 값들의 리스트나 튜플 지정

%%*반드시 테스트 함수의 매개변수 이름은 데코레이터에 지정한 문자열과 일치해야 한다.*%%

## 특징

1. 코드 중복 제거
	- 비슷한 테스트 함수 여러 개를 작성할 필요 없이 하나의 함수로 관리할 수 있다.
2. 개별 테스트 실행
	- 각 파라미터 조합마다 별개의 테스트 케이스로 취급되어 실행된다.
3. 조합(Grid) 테스트
	- 데코레이터를 여러 개 중첩(Stacking)하면 모든 매개변수 조합에 대한 테스트를 생성할 수 있다.

## 사용 예시
```
import pytest

@pytest.mark.parametrize("quantity", [0, -1])
def test_order_item_rejects_non_positive_quantity(quantity):
	with pytest.raises(ValueError):
		OrderItem.create(
			product_id=uuid4(),
			name="아메리카노",
			price=10000,
			quantity=quantity,
		)
```


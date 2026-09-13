## 정의

> 특정 코드 블록에서 예상한 예외(Exception)가 올바르게 발생하는지 검증할 때 사용하는 컨텍스트 매니저(`with`문)

## 기본 사용법

```
import pytest

def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("0으로 나눌 수 없습니다.")
    return a / b

def test_divide_by_zero():
    # 이 블록 안에서 ZeroDivisionError가 발생해야 테스트가 통과
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)
```

==만약 예외가 발생하지 않거나, 다른 예외(예: `TypeError`)가 발생하면 테스트는 **실패**한다.==

## 에러 메시지 검증하기

### 1. `match` 인자 사용하기(권장)

- `match`에 정규표현식이나 문자열을 넘기면, 발생한 에러 메시지에 해당 문자열이 포함되어 있는지 자동으로 비교한다.
```
def test_divide_by_zero_with_match():
    # 에러 메시지에 "0으로 나눌 수"라는 문구가 포함되어 있는지 확인
    with pytest.raises(ZeroDivisionError, match="0으로 나눌 수"):
        divide(10, 0)
```
### 2. `as excinfo`변수 활용하기
- 발생한 예외 객체를 변수(`excinfo`)로 받아와 더 세부적인 단언문(`assert`)을 작성할 수 있다.
```
def test_divide_by_zero_with_excinfo():
    with pytest.raises(ZeroDivisionError) as excinfo:
        divide(10, 0)
    
    # excinfo.value는 실제 발생한 Exception 객체입니다.
    assert str(excinfo.value) == "0으로 나눌 수 없습니다."
    assert excinfo.type is ZeroDivisionError

```

## `parametrize`와 함께 쓰기

```
import pytest
from contextlib import nullcontext

@pytest.mark.parametrize(
    "a, b, expected_result, expected_exception",
    [
        (10, 2, 5.0, nullcontext()), # 정상 동작할 때는 아무 예외도 기대하지 않음
        (10, 0, None, pytest.raises(ZeroDivisionError, match="0으로 나눌 수")), # 에러 기대
    ]
)
def test_divide_parametrized(a, b, expected_result, expected_exception):
    # expected_exception 컨텍스트 안에서 코드를 실행합니다.
    with expected_exception:
        result = divide(a, b)
        if expected_result is not None:
            assert result == expected_result
```
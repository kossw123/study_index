## 왜 필요할까?

- **강력한 예외 시나리오 검증:** 비즈니스 요구사항을 위반했을 때 발생하는 커스텀 예외(`ValueError`, `HTTPException` 등)가 의도한 시점에 정확히 터지는지 확인하여, 화이트박스 테스트 환경에서 시스템의 비정상 흐름 가용성을 검증하기 위해 필요하다.
- **예외 복구력 및 에러 메시지 계약(Contract) 방어:** 단순히 에러가 발생하는 것을 넘어, 사용자에게 올바른 에러 문구와 사유를 전달하고 있는지 명세서 단위로 고정하여 추적할 수 있다.

## 정의

- **`pytest.raises`**는 특정 비즈니스 코드 블록 실행 중 **예상한 특정 예외(Exception)가 올바르게 감지되는지 검증하기 위해 파이썬의 `with` 구문과 결합하여 사용하는 예외 전담 컨텍스트 매니저**이다.

---

## 기본 사용법 및 예외 클래스 매칭 스펙

```python
import pytest

def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("0으로 나눌 수 없습니다.")
    return a / b

def test_divide_by_zero():
    # 💡 이 블록 안에서 지정한 예외가 정상 발생해야 테스트 통과
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)
```

### ⚠️ [중요] 예외 매칭의 클래스 상속 규칙 (Subclass 통과 정책)
- `pytest.raises`는 파이썬의 표준 예외 처리 메커니즘을 따르므로, **지정한 예외 클래스뿐만 아니라 그 예외를 상속받은 하위 클래스(Subclass)가 발생해도 테스트를 통과(PASSED)**시킨다.
- 만약 하위 클래스를 배제하고 오직 정확히 해당 타입의 예외만 터졌는지 엄격하게 검증하고 싶다면, 아래에 서술할 `as excinfo` 변수를 획득하여 `assert excinfo.type is ZeroDivisionError` 구문을 명시적으로 결합해야 한다.
- 예상한 예외가 아예 발생하지 않거나, 상속 관계가 없는 전혀 다른 예외(예: `TypeError`)가 발생하면 테스트는 즉시 **실패(FAILED)**한다.

---

## 에러 메시지 정밀 검증 기법 (정규표현식 주의점)

### 1. `match` 인자 활용 패턴 (정규표현식 검색 엔진 작동)
- `match` 파라미터는 단순한 문자열 포함 검사가 아니라, 내부적으로 **파이썬 정규표현식(`re.search()`) 매칭 엔진을 구동**한다.
- 따라서 에러 문구 내에 정규식 특수 문자(예: `[ ]`, `( )`, `.` 등)가 포함되어 있다면 특수 기호로 오인하여 매칭에 실패할 수 있다.

```python
import re

# 기법 A: 부분 일치 검증 (일반적인 텍스트는 그대로 검색 가능)
def test_divide_by_zero_with_match():
    with pytest.raises(ZeroDivisionError, match="0으로 나눌 수"):
        divide(10, 0)

# 기법 B: 토씨 하나 틀리지 않는 정확한 문장 일치 검증 (Exact Match)
def test_exact_error_message_with_escape():
    # ⚠️ 메시지 내의 메타문자 오작동을 막기 위해 re.escape()로 안전하게 감싸서 매칭을 강제한다.
    exact_msg = re.escape("0으로 나눌 수 없습니다.")
    with pytest.raises(ZeroDivisionError, match=f"^{exact_msg}$"):
        divide(10, 0)
```

### 2. `as excinfo` 컨텍스트 변수 활용 패턴 (가장 엄격한 검증)
- 발생한 예외 인프라 정보 전체를 `excinfo` 변수로 인계받아, `with` 블록 외부에서 순수 파이썬 단언문(`assert`)으로 정밀 튜닝 검증을 수행한다.

```python
def test_divide_by_zero_with_excinfo():
    with pytest.raises(ZeroDivisionError) as excinfo:
        divide(10, 0)
    
    # 1. excinfo.value는 디스크에 실제 찍힌 파이썬 Exception 인스턴스 자체를 반환
    assert str(excinfo.value) == "0으로 나눌 수 없습니다." # 완전 일치 동기식 검증
    
    # 2. [교정] 하위 클래스를 불허하고 정확히 단일 타입 매칭 검증
    assert excinfo.type is ZeroDivisionError 
```

---

## 데이터 기반 테스트와의 결합 아키텍처 (`parametrize`)

정상 성공 시나리오(`nullcontext`)와 의도된 에러 발생 시나리오(`pytest.raises`)를 하나의 정형화된 데이터 배열(Matrix) 안에 빌드하여 유스케이스 전체를 관통 테스트하는 고도화된 실무 패턴이다.

```python
import pytest
from contextlib import nullcontext

@pytest.mark.parametrize(
    "a, b, expected_result, expected_exception",
    [
        (10, 2, 5.0, nullcontext()),  # 정상 성공 케이스: 아무런 예외 컨텍스트도 요구하지 않음
        (10, 0, None, pytest.raises(ZeroDivisionError, match="0으로 나눌 수")), # 예외 기대 케이스
    ]
)
def test_divide_parametrized(a, b, expected_result, expected_exception):
    # 인자로 유입된 expected_exception 컨텍스트 쉴드를 동적으로 가동
    with expected_exception:
        result = divide(a, b)
        if expected_result is not None:
            assert result == expected_result
```

## 관련 개념

#PytestRaises #ExceptionTesting #AssertionIntrospection #RegExMatchSpec #ReEscape #SubclassMatching #ParametrizePattern #NullContext

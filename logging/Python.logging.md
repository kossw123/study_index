## 왜 필요한가?

- **[코드 수정 없이 출력 제어]**
	- `print()`는 무조건 화면에 텍스트를 출력해야 한다.
	- `logging`은 설정 파일이나 코드 한 줄만 바꾸면, 개발 모드에서는 `DEBUG` 메시지까지 전부 보고 상용 서비스에서는 `ERROR` 메시지만 출력하도록 즉시 변경할 수 있다.
- **[파일 및 외부 시스템 저장 자동화]**
	- `print()`는 터미널 창이 닫히면 출력된 내용이 사라진다.
	- `logging`은 파일 저장, 매일 다른 파일로 분할(Rotating), 네트워크 전송, 슬랙(Slack) 알림 연동 등을 코드로 쉽게 구현할 수 있다.
- **[문제 추적에 필수적인 메타데이터 제공]**
	- `print("오류 발생")`이라는 코드는 언제, 어느 파일의 몇 번째 줄에서 실행되었는지 알 수 없다.
	- `logging`은 발생 시간, 파일명, 함수명, 라인 넘버, 프로세스 ID 등을 자동으로 기록해준다.
- **[구조화된 로그 생성]**
	- 대규모 서비스에서는 수백만 줄의 로그가 쌓인다.
	- `logging`을 사용하면 날짜, 레벨, 메시지가 일정한 규칙(Format)으로 쌓이므로, 추후 ELK 스택이나 AWS CloudWatch 같은 로그 분석 도구로 검색하기 쉽다.
- **[라이브러리와의 통합 표준]**
	- - 오픈소스 라이브러리(requests, Django, FastAPI 등)는 모두 파이썬 내장 `logging`을 기반으로 작동한다.
	- - 개발자가 작성한 로거와 오픈소스 로거의 설정을 한곳에서 통합 관리할 수 있다.

## 정의

- 프로그램 실행 상태를 진단하고 기록하는 표준 라이브러리다.
- 기본 로그 레벨은 `WARNING`이며, `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` 5가지 단계별 메시지를 콘솔이나 파일로 출력할 수 있다.

## 로그 레벨 (심각도 순)

```
CRITICAL > ERROR > WARNING > INFO, DEBUG
```

- **DEBUG**: 개발 중 상세한 정보 기록
- **INFO**: 애플리케이션의 정상 작동 흐름
- **WARNING**: 잠재적 문제 상황 (계속 실행 가능)
- **ERROR**: 런타임 예외 및 오류
- **CRITICAL**: 프로그램 중단 위험이 있는 치명적 오류

## 동작 과정

```
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

logging.info('프로그램이 시작됩니다.')
logging.error('오류가 발생했습니다.')
```

> **로거(Logger), 핸들러(Handler), 포맷터(Formatter)** 3가지 구성 요소가 유기적으로 연결되어 작동한다.

- **Logger (로거)**: 애플리케이션 코드가 직접 호출하는 인터페이스입니다. 로그 레벨을 결정
- **Handler (핸들러)**: 로거가 전달한 로그를 어디로 보낼지 결정 (콘솔, 파일, 이메일 등)
- **Formatter (포맷터)**: 로그 메시지의 최종 출력 모양과 텍스트 형식을 지정

## logging 사용법

1. 로거 생성 및 로그 레벨 설정
2. 포맷터 정의
3. 콘솔 핸들러 생성 (터미널에 출력)
4. 파일 핸들러 생성 (특정 파일에 기록)
5. 로거에 핸들러 등록 (핵심 연결 과정)
6. 실제 코드에서 사용

> 위 코드에서 logger.debug()를 실행하면,
> 
> - 콘솔 핸들러는 INFO 레벨 이상만 보여주므로 화면에는 출력이 안된다.
> - 파일 핸들러는 DEBUG 레벨 이상을 기록하므로 app_development.log 파일에는 저장된다.

```
import logging

# 1. 로거 생성 및 로그 레벨 설정
logger = logging.getLogger("MyService")
logger.setLevel(logging.DEBUG)  # 로거는 가장 낮은 DEBUG부터 수집하도록 설정

# 2. 포맷터 정의 (시간 - 이름 - 레벨 - 메시지)
log_formatter = logging.Formatter(
    "%(asctime)s [%(name)s] %(levelname)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)

# 3. 콘솔 핸들러 생성 (터미널에 출력)
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)  # 콘솔에는 INFO 이상만 출력
console_handler.setFormatter(log_formatter)

# 4. 파일 핸들러 생성 (특정 파일에 기록)
file_handler = logging.FileHandler("app_development.log", encoding="utf-8")
file_handler.setLevel(logging.DEBUG)  # 파일에는 개발용 DEBUG까지 모두 기록
file_handler.setFormatter(log_formatter)

# 5. 로거에 핸들러 등록 (핵심 연결 과정)
logger.addHandler(console_handler)
logger.addHandler(file_handler)

# 6. 실제 코드에서 사용하기
logger.debug("디버깅용 상세 데이터: {'user_id': 42}")
logger.info("사용자가 로그인했습니다.")
logger.warning("디스크 공간이 부족합니다.")
logger.error("데이터베이스 연결에 실패했습니다.")
```

## 자주 쓰는 포맷터 속성 (Metadata)

- - `%(asctime)s`: 로그 기록 시간
- `%(levelname)s`: 로그 레벨 이름 (INFO, ERROR 등)
- `%(name)s`: 로거의 이름
- `%(message)s`: 개발자가 입력한 실제 로그 내용
- `%(filename)s`: 로그가 발생한 파이썬 파일명
- `%(lineno)d`: 로그가 발생한 소스 코드의 줄 번호 (Line number)

## 예외 상황(ERROR) 전체 추적: `exc_info`

> 오류가 발생했을 때 단순 메시지만 적으면 원인 파악이 어렵기에, `logginer.error()` 내부에서 `exc_info=True`라고 설정하면 에러 트레이스백(Traceback) 전체를 로그에 남긴다.

```
try:
    result = 10 / 0
except ZeroDivisionError as e:
    # exc_info=True가 에러의 상세 경로와 원인을 자동으로 포맷팅해 줍니다.
    logger.error("연산 중 에러 발생!", exc_info=True)
```

## 프로젝트에서는 어디에서 사용하는가?

- LGTM 중 Loki를 통해 시스템의 로그를 수집하여, Grafana라는 웹페이지에게 API 통신을 통해 데이터를 UI로 손쉽게 볼 수 있도록 한다.

## 관련 개념

#LGTM, #logging, #MSA, #backend
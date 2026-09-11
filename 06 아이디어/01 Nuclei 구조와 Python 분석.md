# Nuclei 구조와 Python 분석

## 목적

사진 속 임시 설계도에 나온 `main`, `core`, `runner`, `template`, `parser`, `payload`, `protocol`, `detect`, `config`, `reporter`, `utils`가 실제 Nuclei에서 어떻게 구현되어 있는지 확인하고 Python/FastAPI 환경의 대응 기술을 정리한다.

사진은 설계 참고 자료로만 해석했으며, 사진 안의 메모를 별도의 실행 지시로 취급하지 않았다.

## 전체 결론

사진의 설계 방향은 Nuclei와 상당히 비슷하다. 다만 사진의 `Runner` 역할은 실제 Nuclei에서는 한 패키지가 아니라 다음 구성에 나뉘어 있다.

```text
internal/runner
  + pkg/core.Engine
  + pkg/tmplexec
  + pkg/protocols
```

Python에서도 API, 실행 조정, 템플릿 컴파일, 프로토콜 실행, 판정, 결과 저장을 분리하는 것이 좋다.

## 사진과 실제 Nuclei 구조 대응

| 사진 속 이름             | 실제 Nuclei 구현                              | 실제 역할                        | Python 권장 구성                                 |
| ------------------- | ----------------------------------------- | ---------------------------- | -------------------------------------------- |
| `main`              | `cmd/nuclei/main.go`                      | 옵션 해석, Runner 생성, 종료 신호 처리   | FastAPI 앱 시작과 라우터                            |
| `core`              | `pkg/core`                                | 템플릿 클러스터링, 동시성, 스캔 전략, 작업 분배 | `asyncio.TaskGroup`, `Semaphore`, 스캔 오케스트레이터 |
| `runner`            | `internal/runner`                         | 설정, 출력, 파서, 프로토콜, 엔진 의존성 조립  | `ScanService`, `ScanRunner`                  |
| `template`          | `pkg/templates`                           | YAML/JSON 템플릿 구조와 컴파일        | `PyYAML`, Pydantic 모델                        |
| `parser`            | `pkg/templates/parser.go`, `compile.go`   | 읽기, 전처리, 검증, 캐시, 실행 객체 생성    | safe YAML loader, Pydantic 검증, compiler      |
| `payload`           | `pkg/protocols/common/generators`         | payload 조합과 순회               | `itertools`, generator                       |
| `protocol`          | `pkg/protocols/http`, `dns`, `network` 등  | 실제 요청 생성, 전송, 응답 정규화         | HTTPX, asyncio, dnspython                    |
| `matcher`, `detect` | `pkg/operators/matchers`, `extractors`    | 응답 비교, 추출, 취약점 판정            | `re`, `lxml`, JMESPath, 안전한 자체 DSL           |
| `format`            | 여러 모델·입출력 패키지에 분산                         | 모델, YAML/JSON, 결과 변환         | Pydantic 모델과 serializer                      |
| `config`            | `pkg/types.Options`, `pkg/catalog/config` | 실행 옵션과 환경 설정                 | `pydantic-settings`                          |
| `reporter`          | `pkg/output`, `pkg/reporting`             | 결과 출력과 외부 리포팅                | SQLite/JSONL writer와 reporter 분리             |
| `utils`             | `pkg/utils` 외                             | 공통 함수                        | 작고 명확한 공통 모듈만 유지                             |

## 실제 Nuclei 실행 흐름

```text
main                         # 프로그램 진입점. CLI 옵션과 설정을 읽고 실행 준비
  -> runner.New(options)     # 파서, 출력기, rate limiter 등 공통 구성요소 생성
  -> runner.RunEnumeration() # 입력과 템플릿을 준비하고 전체 스캔 절차 시작
  -> template loader/parser  # YAML/JSON 템플릿 검색, 파싱, 검증, 필터링 및 컴파일
  -> core.Engine             # 스캔 전략 선택, 템플릿 클러스터링, 동시성·작업 풀 제어
  -> TemplateExecuter        # 컴파일된 템플릿의 요청들을 순서 또는 flow에 따라 실행
  -> HTTP/DNS/TCP protocol   # 대상에 실제 네트워크 요청을 보내고 응답을 공통 데이터로 변환
  -> matcher/extractor       # 응답이 탐지 조건과 일치하는지 판정하고 필요한 값을 추출
  -> output.Writer           # 탐지 결과를 표준 ResultEvent로 만들고 화면·파일 등에 출력
  -> reporting              # 결과 중복 제거 후 JSON, Markdown, 이슈 트래커 등으로 전달
```

주요 소스:

- `cmd/nuclei/main.go`: 옵션 처리, Runner 생성, `RunEnumeration()` 호출
- `internal/runner/runner.go`: 전체 실행 환경과 의존성 조립
- `pkg/core/execute_options.go`: 템플릿 클러스터링과 스캔 전략
- `pkg/protocols/protocols.go`: 프로토콜 실행 인터페이스
- `pkg/output/output.go`: 최종 결과 모델과 writer 인터페이스

## FastAPI와 유사한 Nuclei DAST 서버

이 로컬 Nuclei 사본에는 API로 요청을 받는 DAST 서버가 있다.

- `POST /fuzz`로 `raw_http`, `url`을 받는다.
- 요청을 worker pool에 넣는다.
- Raw HTTP를 파싱한다.
- `http` 또는 `https` 스킴인지 검사한다.
- 대상이 허용 범위 안인지 검사한다.
- 중복 요청을 제거한다.
- 스캔 엔진을 호출한다.

DAST서버는 무료이긴하나, 상업적으로 사용 시, 라이선스로 써야함.

- OWASP ZAP
- Burp Suite
- Nikto
- Nuclei의 DAST 템플릿
- 기타 상용 웹 취약점 스캐너

관련 소스:

- `internal/server/server.go`
- `internal/server/requests_worker.go`
- `internal/server/nuclei_sdk.go`

Python 권장 구조:

```text
Nginx
  -> Uvicorn / FastAPI
      -> ScanService
          -> bounded job queue
              -> ScanRunner
                  -> TemplateLoader / Compiler
                  -> PayloadGenerator
                  -> ProtocolExecutor
                  -> Matcher / Extractor
                  -> ResultWriter / Reporter
```

FastAPI는 스캔 작업 생성, 상태·결과 조회, 작업 취소, 템플릿 검증, 설정 조회를 담당하고 실제 네트워크 스캔은 Runner가 수행한다.

## Core와 Runner의 차이

### internal/runner

Runner는 설정, catalog, loader, parser, input provider, output writer, rate limiter, reporter와 core Engine을 조립하는 애플리케이션 계층이다.

### pkg/core.Engine

Engine은 다음과 같은 실제 작업 분배를 담당한다.

- 유사한 템플릿 요청 클러스터링
- template-spray 또는 host-spray 전략 선택
- self-contained 템플릿 분리
- 프로토콜 종류별 work pool
- 대상과 템플릿의 동시 실행
- 최종 결과 callback

Python에서는 `ScanRunner`와 `ScanEngine`을 분리하면 이 관계를 표현할 수 있다.

## Template와 Parser

Nuclei의 Template은 단순 YAML dictionary가 아니라 다음을 포함하는 명시적인 구조체다.

- ID와 메타데이터
- HTTP, DNS, TCP, SSL, WebSocket 등의 요청
- 변수와 상수
- flow
- stop-at-first-match
- matcher와 extractor
- 컴파일된 실행기

Parser는 두 단계를 구분한다.

1. 가벼운 파싱: YAML/JSON 읽기, 필수 필드와 태그 검사
2. 컴파일: 정규식, matcher, extractor, payload generator와 프로토콜 실행기 준비

Python에서는 `yaml.safe_load()`, Pydantic 검증, 별도 `TemplateCompiler`, 내용 해시 기반 캐시를 사용하는 것이 좋다. `eval()`과 `exec()`은 사용하지 않는다.

## Matcher와 Extractor

Nuclei HTTP matcher 종류:

- 상태 코드
- 응답 크기
- 단어
- 정규식
- 바이너리
- DSL
- XPath

Extractor 종류:

- 정규식
- key/value
- XPath
- JSON 쿼리
- DSL

Python 대응:

| 기능 | Python 구현 |
|---|---|
| word/status/size/binary | 표준 Python 비교 연산 |
| regex | `re` |
| XPath 및 HTML/XML | `lxml` |
| JSON 쿼리 | `jmespath` 또는 `jsonpath-ng` |
| Base64와 해시 | `base64`, `hashlib` |
| 복합 DSL | 허용 연산만 제공하는 자체 AST evaluator |

Nuclei DSL 전체 호환을 첫 단계에서 시도하지 않는다. `==`, `!=`, 비교, AND/OR/NOT, `contains`, `len`, status/body/header 접근 정도부터 지원한다.

## Payload

- `batteringram`: 한 값을 여러 위치에 동일하게 적용
- `pitchfork`: 여러 목록을 같은 인덱스로 결합
- `clusterbomb`: 모든 값의 곱집합

Python에서는 `zip()`, `itertools.product()`와 generator를 사용한다. 모든 조합을 list로 만들지 않고 `yield`로 스트리밍한다.

변수 치환은 Jinja2 전체 문법 대신 `{{BaseURL}}`, `{{username}}` 같은 제한된 placeholder부터 지원한다.

## 권장 Python 라이브러리

### 첫 버전

| 용도 | 권장 라이브러리 | 무엇인가 | 스캐너에서 하는 일 |
|---|---|---|---|
| API | `fastapi` | 파이썬 API 프레임워크 | `/scans`, `/results`, `/status` 같은 제품 API를 정의하고 입력값을 검증한다. |
| ASGI 서버 | `uvicorn` | ASGI 애플리케이션을 실행하는 웹서버 | 네트워크의 HTTP 요청을 받아 FastAPI에 전달하고 응답을 돌려준다. |
| 요청·템플릿 모델 | `pydantic` | 데이터를 검사하고 변환하는 모델 라이브러리 | 대상 주소, 포트, 스캔 옵션과 YAML 템플릿 구조가 올바른지 확인한다. |
| 환경 설정 | `pydantic-settings` | 환경변수와 설정 파일을 Pydantic 모델로 읽는 라이브러리 | 포트, DB 경로, 동시 실행 수, API 키 등을 코드 밖에서 관리한다. |
| YAML | `PyYAML` | YAML 문서를 파이썬 데이터로 읽고 쓰는 라이브러리 | 나중에 필요할 경우 자체 YAML 규칙을 불러온다. 신뢰하지 않는 YAML은 `safe_load()`로 읽는다. |
| HTTP/HTTPS | `httpx` | 동기·비동기 HTTP 클라이언트 | 자체 검사기가 대상 장비에 제한된 HTTP 요청을 보낸다. |
| 비동기 실행 | Python `asyncio` | 파이썬 표준 비동기 실행 도구 | 여러 네트워크 요청을 기다리는 동안 다른 요청을 처리하되 동시 실행 수를 제한한다. |
| 속도 제한 | `aiolimiter` 또는 자체 token bucket | 일정 시간 동안 보낼 요청 수를 제한하는 도구 | 대상 장비와 라즈베리파이에 과부하가 생기지 않도록 초당 요청 수를 통제한다. |
| HTML/XML | `lxml` | HTML·XML 파싱 및 XPath 처리 라이브러리 | 응답 문서에서 링크, 폼, 특정 요소나 값을 추출한다. |
| JSON 추출 | `jmespath` | JSON 데이터 검색 표현식 라이브러리 | API 응답의 중첩된 JSON에서 검사할 값을 선택한다. |
| 로컬 저장 | `aiosqlite` | SQLite를 비동기 코드에서 사용하는 라이브러리 | 스캔 작업 상태와 요약 결과를 로컬 DB에 저장한다. |
| 테스트 | `pytest`, `pytest-asyncio`, `respx` | 일반·비동기 코드와 HTTP 요청을 시험하는 도구 | 실제 장비에 요청하지 않고 스캐너 동작, 실패, 시간 초과 등을 검증한다. |

### 서버 관련 용어

| 용도         | 도구                  | 무엇인가          | 스캐너에서 하는 일                     |
| ---------- | ------------------- | ------------- | ------------------------------ |
| API        | `FastAPI`           | API 제작 프레임워크  | `/scans`, `/results` 같은 기능을 만듦 |
| 웹서버        | `Uvicorn`           | ASGI 서버       | HTTP 요청을 받아 FastAPI에 전달        |
| 데이터 모델     | `Pydantic`          | 데이터 검증 도구     | IP, URL, 포트, 스캔 옵션 검사          |
| 환경 설정      | `pydantic-settings` | 설정 관리 도구      | DB 경로, API 키, 동시 실행 수 관리       |
| YAML       | `PyYAML`            | YAML 해석 도구    | 스캔 템플릿 파일을 읽음                  |
| HTTP/HTTPS | `httpx`             | HTTP 클라이언트    | 자체 검사기가 대상 장비에 HTTP 요청 전송     |
| 비동기 실행     | `asyncio`           | 파이썬 표준 비동기 기능 | 여러 요청을 효율적으로 처리                |
| 속도 제한      | `aiolimiter`        | 요청량 제한 도구     | 장비에 너무 많은 요청을 보내지 않게 함         |
| HTML/XML   | `lxml`              | 문서 분석 도구      | HTML에서 링크, 폼, 값 추출             |
| JSON 추출    | `jmespath`          | JSON 검색 도구    | 복잡한 JSON 응답에서 원하는 값 추출         |
| 로컬 저장      | `aiosqlite`         | 비동기 SQLite 도구 | 작업 상태와 결과 저장                   |
| 테스트        | `pytest` 등          | 자동 테스트 도구     | 실제 장비 없이 프로그램 동작 검증            |

```text
사용자 또는 관리 화면
  -> Uvicorn: HTTP 연결 처리
      -> FastAPI: 제품 API와 작업 흐름
          -> Scanner worker: 직접 만든 검사
              -> HTTP Scanner
              -> TCP Scanner
              -> TLS Scanner
          -> SQLite: 작업 상태와 결과 저장
```
### 프로토콜 확장

이 표는 전부 외부 라이브러리를 뜻하지 않는다. 파이썬에 기본 포함된 **표준 라이브러리**, 별도로 설치하는 **외부 라이브러리**, 그리고 이들을 조합하는 **구현 방식**이 섞여 있다.

| 검사 영역          | 권장 도구·방식                          | 종류                  | 스캐너에서 하는 일                                                                           |
| -------------- | --------------------------------- | ------------------- | ------------------------------------------------------------------------------------ |
| 일반 TCP         | `asyncio.open_connection()`       | 파이썬 표준 기능           | 특정 IP와 포트에 비동기로 연결하고 데이터를 주고받는다. 포트 연결 확인이나 배너 수집에 사용할 수 있다.                         |
| UDP            | `asyncio` datagram API            | 파이썬 표준 기능           | 연결 없이 UDP 패킷을 보내고 받는다. DNS, SNMP 같은 UDP 기반 프로토콜 검사에 활용한다.                            |
| TLS 인증서        | `ssl`                             | 파이썬 표준 라이브러리        | HTTPS 서버와 TLS 연결을 만들고 인증서의 발급자, 대상 이름, 만료일 등을 확인한다.                                  |
| DNS            | `dnspython`의 `dns.asyncresolver`  | 외부 라이브러리            | A, AAAA, MX, TXT 같은 DNS 레코드를 비동기로 조회한다.                                              |
| WebSocket      | `websockets.asyncio`              | 외부 라이브러리            | WebSocket 서버에 연결해 메시지를 송수신하고 응답을 검사한다.                                               |
| IPv4/IPv6/CIDR | `ipaddress`                       | 파이썬 표준 라이브러리        | IP 주소를 검증하고 특정 CIDR 범위에 속하는지 확인한다. 실제 패킷을 보내지는 않는다.                                  |
| Raw HTTP       | `asyncio` socket + `h11` 또는 직접 구성 | 표준 기능과 외부 라이브러리의 조합 | 헤더와 본문을 세밀하게 제어해 HTTP/1.1 요청을 전송·분석한다. `h11`은 HTTP 메시지 처리를 돕지만 네트워크 연결 자체는 담당하지 않는다. |
| 패킷/MAC/DHCP    | `Scapy`                           | 외부 라이브러리            | 낮은 수준의 패킷을 만들고 보내며 응답을 분석한다. 관리자 권한과 추가 OS 설정이 필요할 수 있다.                             |
### 프로토콜 관련 용어

|항목|종류|하는 일|
|---|---|---|
|`asyncio.open_connection()`|파이썬 기본 기능|TCP 포트에 비동기로 연결하고 데이터 송수신|
|`asyncio` datagram API|파이썬 기본 기능|UDP 패킷 송수신|
|`ssl`|파이썬 표준 라이브러리|TLS 연결과 인증서 만료일·발급자 확인|
|`dnspython`|외부 라이브러리|DNS 레코드 조회|
|`websockets`|외부 라이브러리|WebSocket 메시지 송수신|
|`ipaddress`|파이썬 표준 라이브러리|IP 주소와 CIDR 범위 계산·검증|
|`h11`|외부 라이브러리|낮은 수준의 HTTP/1.1 메시지 생성·분석|
|`Scapy`|외부 라이브러리|패킷, MAC, ARP, DHCP 등 저수준 네트워크 처리|

외부 패키지가 실제로 필요해졌을 때 다음과 같이 설치한다.

```bash
pip install dnspython websockets h11 scapy
```

첫 버전부터 전부 설치할 필요는 없다. HTTP 중심 MVP에는 `httpx`, `asyncio`, `ssl`, `ipaddress`부터 사용하고 DNS, Raw HTTP, 패킷 검사가 필요해질 때 해당 외부 라이브러리를 추가하는 편이 라즈베리파이 자원을 아끼기 좋다.

## Raspberry Pi용 단계별 범위

### 1단계

- HTTP/HTTPS
- YAML 템플릿
- word, status, regex matcher
- regex, header, JSON extractor
- payload 세 가지 조합
- timeout, 동시성, 속도 제한
- JSONL 또는 SQLite 결과
- 작업 생성, 상태, 결과, 취소 API

### 2단계

- DNS
- 일반 TCP/TLS
- WebSocket
- 복합 matcher와 제한된 DSL

### 3단계

- Headless browser
- Raw HTTP 세부 제어
- DHCP, MAC, raw packet
- 외부 reporter 연동

## 필수 보안 경계

- API 인증
- 허용 도메인 또는 CIDR 목록
- redirect 목적지 재검사
- DNS rebinding 방어
- localhost, link-local, metadata IP 정책
- 요청 수, 응답 크기, 실행 시간 제한
- 작업 큐 최대 크기
- 템플릿에서 shell, `eval`, `exec` 금지
- 템플릿 디렉터리 밖 파일 접근 금지
- FastAPI/Uvicorn을 비특권 사용자로 실행
- Scapy/raw socket은 별도 권한의 worker로 분리

## Nginx의 위치

Nginx는 Python 라이브러리가 아니라 외부 reverse proxy다.

```text
외부 사용자
  -> Nginx: TLS, 인증 보조, 요청 크기 제한
      -> Uvicorn/FastAPI: 로컬 포트
          -> Scanner worker
```

프록시 헤더는 모든 주소를 무조건 신뢰하지 말고 실제 Nginx 주소만 허용한다.

## 라이선스

로컬 Nuclei 소스와 templates 사본에는 MIT 라이선스가 포함되어 있다. 실제 코드를 복사하거나 수정하여 사용할 경우 저작권 문구와 라이선스 고지를 유지해야 한다.

## 참고 링크

- [FastAPI Background Tasks](https://fastapi.tiangolo.com/tutorial/background-tasks/)
- [FastAPI Behind a Proxy](https://fastapi.tiangolo.com/advanced/behind-a-proxy/)
- [HTTPX Async Support](https://www.python-httpx.org/async/)
- [HTTPX Timeouts](https://www.python-httpx.org/advanced/timeouts/)
- [HTTPX Resource Limits](https://www.python-httpx.org/advanced/resource-limits/)
- [Python asyncio streams](https://docs.python.org/3/library/asyncio-stream.html)
- [dnspython asynchronous I/O](https://dnspython.readthedocs.io/en/stable/async.html)
- [Pydantic Settings](https://docs.pydantic.dev/latest/concepts/pydantic_settings/)

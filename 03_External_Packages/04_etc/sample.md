---
상태: false
---

# Nuclei 분석 명칭 사전 #전지성

## 이 파일의 사용 원칙

Nuclei 분석 문서에서 처음 보는 명칭이 나오면 이 파일에 뜻을 정리한다. 각 분석 문서는 현재 코드에서 그 명칭이 어떻게 사용되는지 설명하고, 공통 정의는 이 파일을 참조한다.

```text
sample.md
└─ 명칭의 공통 뜻

main.md·설명서·패키지 분석 문서
└─ 해당 명칭이 현재 코드에서 선언·생성·전달되는 방법
```

같은 이름이 여러 패키지에 존재할 수 있으므로 가능한 한 소속을 함께 쓴다.

```text
types.Options
└─ Nuclei 전체 실행 설정

jsonexporter.Options
└─ JSON Exporter 전용 설정
```

## 태그로 명칭을 바로 찾는 방법

각 명칭 제목 바로 다음 줄에는 `#명칭/분야/용어` 형식의 Obsidian 태그가 붙어 있다. 태그를 제목과 분리했기 때문에 다른 문서의 `파일#제목` 링크가 해당 설명 위치로 정확하게 이동한다.

```text
Runner
└─ #명칭/nuclei/runner

함수 서명
└─ #명칭/dsl/함수-서명

JSON
└─ #명칭/출력/json
```

문서에서 태그를 클릭하면 같은 태그가 붙은 위치를 확인할 수 있다. Obsidian 검색창에서는 다음처럼 입력한다.

```text
tag:#명칭/dsl
tag:#명칭/dsl/함수-서명
tag:#명칭/nuclei/runner
```

- `tag:#명칭/dsl`: DSL 분야 명칭 전체를 찾는다.
- `tag:#명칭/dsl/함수-서명`: 함수 서명 항목을 바로 찾는다.
- `tag:#명칭/nuclei/runner`: Runner 항목을 바로 찾는다.

### 분야별 태그 색인

| 찾으려는 내용 | 태그 |
|---|---|
| 파일·패키지·라이브러리 | `#명칭/파일` |
| 함수·구조체·변수·인터페이스 | `#명칭/코드` |
| CLI 옵션과 Parse | `#명칭/cli` |
| Runner·Engine·Catalog 등의 구성요소 | `#명칭/nuclei` |
| Template·Matcher·Extractor | `#명칭/template` |
| DSL·함수 서명·Registry | `#명칭/dsl` |
| INF·WRN·ERR·DBG | `#명칭/log` |
| JSON·JSONL·Exporter | `#명칭/출력` |
| 네트워크·보안·서명 | `#명칭/보안` |
| 성능·종료·Resume | `#명칭/실행` |
| 옵션 기본값 | `#명칭/옵션` |
| 표준 라이브러리 목록 | `#명칭/라이브러리` |

`Pointer 또는 포인터`처럼 두 명칭이 같은 뜻일 때는 `#명칭/코드/pointer`와 `#명칭/코드/포인터`를 모두 붙여 어느 표현으로도 찾을 수 있게 했다.

## 1. 파일과 프로그램 구조 명칭 #명칭/파일

### 원본 소스
#명칭/파일/원본-소스

실제로 컴파일되어 Nuclei 실행 파일을 만드는 Go 코드다. 분석용 Markdown 문서와 구분한다.

### 분석 노트
#명칭/파일/분석-노트

원본 소스를 사람이 읽고 해석한 Markdown 문서다. 실행 코드가 아니며 Nuclei 프로그램에 포함되지 않는다.

### 패키지
#명칭/파일/패키지

관련된 Go 파일과 기능을 묶는 코드 단위다. `runner`, `catalog`, `core`, `output` 등이 예다.

### 표준 라이브러리
#명칭/파일/표준-라이브러리

Go 설치에 기본 포함되는 패키지다. 별도 프로젝트에서 내려받지 않아도 사용할 수 있다.

예: `fmt`, `os`, `time`, `runtime`, `path/filepath`.

### 외부 라이브러리 또는 외부 패키지
#명칭/파일/외부-라이브러리 #명칭/파일/외부-패키지

Nuclei 외부 프로젝트에서 가져와 사용하는 코드다. 보통 `go.mod`에 모듈 이름과 버전이 기록된다.

예: `goflags`, `gologger`, `projectdiscovery/dsl`, `xid`.

### 내부 패키지
#명칭/파일/내부-패키지

Nuclei 프로젝트 자체에서 구현한 패키지다. `internal/runner`, `pkg/core`, `pkg/templates` 등이 예다.

### import
#명칭/파일/import

다른 패키지가 공개한 타입과 함수를 현재 파일에서 사용할 수 있도록 연결하는 선언이다. import만으로 모든 기능이 자동 실행되는 것은 아니며 실제 호출 코드가 있어야 한다.

### 실행 파일
#명칭/파일/실행-파일

운영체제가 직접 실행할 수 있도록 원본 소스를 컴파일한 결과다. 사용자가 `./nuclei` 또는 `nuclei.exe`를 실행하면 `main()`에서 시작한다.

## 2. 코드 구성 명칭 #명칭/코드

### 선언
#명칭/코드/선언

변수, 구조체, 필드, 함수의 이름과 형태를 코드에 정의하는 부분이다.

```go
type Runner struct {
    options *types.Options
}
```

### 정의
#명칭/코드/정의

문맥에 따라 선언과 비슷하게 쓰이지만, 함수가 실제로 수행할 코드나 타입의 구체적인 내용을 작성한다는 뜻으로도 사용한다.

### 초기화
#명칭/코드/초기화

선언된 변수나 구조체에 처음 사용할 값을 넣는 작업이다.

### 구조체
#명칭/코드/구조체

서로 관련된 값을 이름 있는 필드로 묶은 타입이다. `Options`, `Runner`, `Engine`, `Template`이 예다.

### 객체 또는 인스턴스
#명칭/코드/객체 #명칭/코드/인스턴스

구조체를 바탕으로 실행 중 실제로 만들어진 값이다.

```go
runner := &Runner{options: options}
```

`Runner`는 타입이고 `runner`는 실제로 만들어진 객체를 가리킨다.

### 변수
#명칭/코드/변수

실행 중 사용할 값을 이름으로 보관하는 공간이다.

### 전역변수 또는 패키지 변수
#명칭/코드/전역변수 #명칭/코드/패키지-변수

함수 바깥에 선언되어 같은 패키지의 여러 함수가 사용할 수 있는 변수다. `main.go`의 `options`가 예다.

### 지역변수
#명칭/코드/지역변수

함수나 블록 안에서 선언되어 해당 범위에서만 사용하는 변수다. `nucleiRunner`, `executorEngine` 등이 예다.

### 필드
#명칭/코드/필드

구조체 안에 선언된 값의 자리다. `Options.Debug`, `Runner.output` 등이 예다.

### 함수
#명칭/코드/함수

입력값을 받아 정해진 작업을 수행하고 필요하면 결과를 반환하는 코드 묶음이다.

### 메서드
#명칭/코드/메서드

특정 타입의 객체에 연결된 함수다. `nucleiRunner.RunEnumeration()`은 Runner 객체의 메서드를 호출한다.

### Receiver
#명칭/코드/receiver

어떤 타입에 메서드가 연결되는지 나타내는 부분이다.

```go
func (r *Runner) Close() { ... }
```

여기서 `r *Runner`가 Receiver다.

### 매개변수
#명칭/코드/매개변수

함수를 선언할 때 함수가 받을 입력의 이름과 타입을 적은 것이다.

### 인자
#명칭/코드/인자

함수를 실제 호출할 때 전달하는 값이다.

```go
runner.New(options)
```

여기서 `options`가 인자다.

### 반환값
#명칭/코드/반환값

함수가 작업을 끝낸 뒤 호출한 코드로 돌려주는 값이다. `runner.New()`은 `*Runner`와 `error`를 반환한다.

### error
#명칭/코드/error

작업이 실패한 이유를 전달하는 Go의 오류 값이다. `nil`이면 보통 오류가 없다는 뜻이다.

### nil
#명칭/코드/nil

가리키는 객체나 유효한 값이 없음을 나타낸다. `nucleiRunner == nil`은 Runner 객체가 없는지 확인하는 조건이다.

### Pointer 또는 포인터
#명칭/코드/pointer #명칭/코드/포인터

객체 전체 값이 아니라 해당 객체가 있는 위치를 가리키는 값이다. `*types.Options`, `*Runner`처럼 표시된다.

### 참조 전달
#명칭/코드/참조-전달

이미 만들어진 객체를 가리키는 값을 다른 구조체나 함수에 넘기는 것이다. `Runner.output`과 `ExecutorOptions.Output`이 같은 Writer를 가리킬 수 있다.

### 인터페이스
#명칭/코드/인터페이스

구현체가 제공해야 하는 메서드 목록을 선언한 타입이다. 실제 처리 방법보다 “무엇을 할 수 있어야 하는가”를 정의한다.

예: `catalog.Catalog`, `output.Writer`, `provider.InputProvider`.

### 구현체
#명칭/코드/구현체

인터페이스가 요구하는 메서드를 실제로 구현한 구조체다.

예: `DiskCatalog`는 `Catalog` 인터페이스를 구현하고 `StandardWriter`는 `Writer` 인터페이스를 구현한다.

### 의존성
#명칭/코드/의존성

한 구성요소가 작업하기 위해 필요한 다른 객체나 기능이다. Engine이 요청을 실행하려면 Writer, RateLimiter, Parser 등이 필요하다.

### 의존성 주입
#명칭/코드/의존성-주입

구성요소가 필요한 객체를 내부에서 직접 찾거나 새로 만들지 않고 외부에서 전달받는 방식이다. Runner가 `ExecutorOptions`에 Writer와 RateLimiter를 넣어 Engine과 프로토콜 실행기로 보내는 구조가 예다.

### Callback
#명칭/코드/callback

특정 사건이 발생했을 때 나중에 호출하도록 함수 자체를 값으로 전달하는 방식이다. ResultEvent가 생겼을 때의 후속 처리나 Hang 감지 처리가 예다.

### Wrapper
#명칭/코드/wrapper

다른 함수나 객체를 한 겹 감싸 동일하거나 단순화된 기능을 제공하는 코드다. Nuclei 내부 `GetPrintableDslFunctionSignatures()`가 외부 DSL 패키지의 같은 기능을 호출하는 것이 예다.

### Registry
#명칭/코드/registry

사용할 수 있는 기능을 이름별로 등록해 둔 목록이다. DSL Registry에는 함수 이름, 서명, 실제 실행 함수, Cache 가능 여부 등이 들어간다.

### Helper Function
#명칭/코드/helper-function

반복되는 작은 작업을 돕는 함수다. DSL 문맥에서는 템플릿 표현식에서 호출할 수 있도록 등록된 함수들을 의미하기도 한다.

### Map
#명칭/코드/map

Key와 Value를 연결해 저장하는 자료구조다. DSL의 `HelperFunctions`는 함수 이름을 실제 실행 함수와 연결하는 Map이다.

### Slice
#명칭/코드/slice

길이가 변할 수 있는 값 목록이다. `[]string`은 문자열 Slice다.

### Branch 또는 분기
#명칭/코드/branch #명칭/코드/분기

`if` 등의 조건에 따라 실행 경로가 나뉘는 부분이다.

### Early Return 또는 조기 종료
#명칭/코드/early-return #명칭/코드/조기-종료

함수의 아래 코드를 실행하지 않고 중간의 `return`으로 먼저 끝내는 방식이다. `-ldf`, `-sign`, 기존 결과 Cloud 업로드가 예다.

## 3. CLI와 옵션 명칭 #명칭/cli

### CLI
#명칭/cli/cli

Command Line Interface의 약자다. 버튼 대신 터미널 명령으로 프로그램을 조작하는 방식이다.

### Flag 또는 Option
#명칭/cli/flag #명칭/cli/option

프로그램의 동작을 선택하기 위해 명령에 붙이는 값이다. `-debug`, `-u`, `-o` 등이 예다.

### 긴 옵션 이름
#명칭/cli/긴-옵션-이름

뜻을 명확하게 적은 옵션 이름이다. 예: `-list-dsl-function`.

### 짧은 별칭
#명칭/cli/짧은-별칭

같은 옵션을 짧게 입력하기 위한 이름이다. 예: `-ldf`. 긴 이름과 같은 Options 필드에 연결된다.

### Bool 옵션
#명칭/cli/bool-옵션

입력 여부에 따라 true 또는 false로 동작을 켜고 끄는 옵션이다. `-debug`, `-ldf`, `-sign` 등이 예다.

### 기본값
#명칭/cli/기본값

사용자가 값을 입력하지 않았을 때 사용하는 값이다. Bool 옵션은 주로 false가 기본값이다.

### FlagSet
#명칭/cli/flagset

goflags가 제공하는 옵션 관리자다. 등록된 옵션 이름, 타입, 기본값, 저장 위치와 도움말을 기억한다.

### `CreateGroup()`
#명칭/cli/creategroup

관련 옵션을 Target, Output, Debug처럼 도움말 구역으로 묶는 FlagSet 메서드다. 같은 그룹에 있다고 옵션을 반드시 함께 사용해야 하는 것은 아니다.

### `BoolVarP()`
#명칭/cli/boolvarp

Bool 필드에 긴 옵션 이름과 짧은 별칭, 기본값, 도움말을 연결하는 goflags 메서드다.

### `flagSet.Parse()`
#명칭/cli/flagset-parse

실제 명령줄 문자열을 읽어 등록된 옵션을 찾고 연결된 Options 필드에 값을 저장한다. 스캔을 직접 실행하지는 않는다.

### Config
#명칭/cli/config

반복해서 사용할 실행 옵션을 파일에 저장한 설정이다.

### Template Profile
#명칭/cli/template-profile

템플릿 선택, 필터, 실행 설정, 인증 정보 등을 재사용 가능한 한 묶음으로 만든 설정이다.

### 환경변수
#명칭/cli/환경변수

운영체제나 Shell이 보관하고 프로그램이 읽을 수 있는 설정값이다. API Key와 템플릿 서명 개인키 등에 사용될 수 있다.

## 4. Nuclei 실행 구성요소 명칭 #명칭/nuclei

### `types.Options`
#명칭/nuclei/types-options

`pkg/types/types.go`에 선언된 Nuclei 전체 실행 설정 구조체다. Target, Template, Tags, Debug, Output, Rate Limit 등을 보관한다.

### Runner
#명칭/nuclei/runner

`internal/runner/runner.go`에 선언된 구조체다. Options를 보관하고 Parser, Catalog, InputProvider, Writer, Progress, Interactsh, RateLimiter 등을 생성·연결·정리한다.

### `ExecutorOptions`
#명칭/nuclei/executoroptions

`pkg/protocols/protocols.go`에 선언된 구조체다. Runner가 만든 Writer, Catalog, Parser, RateLimiter 같은 실행 객체를 프로토콜 실행기에 전달한다.

`types.Options`는 사용자가 선택한 설정값이고 `ExecutorOptions`는 실행기에 전달할 준비된 객체 모음이다.

### Catalog
#명칭/nuclei/catalog

템플릿 경로를 해결하고 파일을 여는 공통 인터페이스다.

### DiskCatalog
#명칭/nuclei/diskcatalog

Catalog 인터페이스를 디스크 또는 `fs.FS` 기반으로 구현한 구조체다.

### Loader
#명칭/nuclei/loader

Template 후보에 Tags, Severity, Author, ID 등의 필터를 적용하고 실제로 사용할 목록을 만든다.

### `loader.Config`
#명칭/nuclei/loader-config

Loader가 필요한 Template 경로와 필터 조건, Catalog, ExecutorOptions 등을 담는 설정 구조체다.

### Store
#명칭/nuclei/store

Loader가 선택하고 파싱한 Template와 Workflow, Metadata Index 등을 보관하는 구조체다.

### Parser
#명칭/nuclei/parser

YAML·JSON 템플릿 원문을 `Template` 구조체 객체로 해석한다.

### Cache
#명칭/nuclei/cache

이미 읽거나 계산한 결과를 잠시 보관해 같은 작업을 반복하지 않도록 하는 공간이다.

### Parsed Cache
#명칭/nuclei/parsed-cache

필터와 검증에 사용할 파싱된 Template 객체를 보관한다.

### Compiled Cache
#명칭/nuclei/compiled-cache

프로토콜 실행 준비까지 끝난 무거운 Template 객체를 보관한다.

### Engine
#명칭/nuclei/engine

Template 목록과 InputProvider의 Target을 ScanStrategy에 맞게 조합하고 실제 실행을 관리하는 구조체다.

### InputProvider
#명칭/nuclei/inputprovider

`-u`, `-l`, 표준 입력, OpenAPI 등 여러 입력 형식을 공통 Target 흐름으로 제공하는 인터페이스다.

### Output Writer
#명칭/nuclei/output-writer

탐지 결과인 ResultEvent를 화면, 파일, JSON, Cloud 등에 기록하는 인터페이스다.

### StandardWriter
#명칭/nuclei/standardwriter

기본 터미널과 파일 출력을 담당하는 Writer 구현체다.

### MultiWriter
#명칭/nuclei/multiwriter

하나의 ResultEvent를 여러 Writer에 전달하는 Writer다. 로컬 파일과 Cloud 출력 등을 함께 사용할 수 있다.

### Logger
#명칭/nuclei/logger

프로그램 상태, 경고, 오류, Debug 정보를 출력한다. 취약점 탐지 ResultEvent를 저장하는 Writer와 역할이 다르다.

### Progress
#명칭/nuclei/progress

Target 수, 요청 수, 완료 상태 등 스캔 진행 상황을 집계한다.

### RateLimiter
#명칭/nuclei/ratelimiter

일정 시간에 보낼 수 있는 요청 수를 제한한다.

### Concurrency
#명칭/nuclei/concurrency

한 시점에 동시에 실행할 작업 수를 뜻한다.

### WorkPool
#명칭/nuclei/workpool

동시에 실행할 작업과 Worker를 관리하는 구조다.

### ScanStrategy
#명칭/nuclei/scanstrategy

Template 중심 또는 Host 중심처럼 Template와 Target을 반복하는 순서를 뜻한다.

### ResultEvent
#명칭/nuclei/resultevent

Template ID, Protocol, Severity, Target, 추출값, 요청·응답 등 한 건의 탐지 결과를 구조화한 객체다.

## 5. Template 명칭 #명칭/template

### Template 파일
#명칭/template/template-파일

무엇을 요청하고 어떤 조건을 탐지할지 YAML 또는 JSON으로 작성한 검사 규칙 파일이다.

### `Template` 구조체
#명칭/template/template-구조체

Parser가 Template 파일 원문을 해석해 메모리에 만든 객체다. 파일 원문과 구분한다.

### ID
#명칭/template/id

Template를 구별하는 고유 이름이다. 탐지 결과의 첫 대괄호에 표시된다.

### Info
#명칭/template/info

Template 이름, 작성자, Severity, Tags 같은 설명 정보를 담는다.

### Request
#명칭/template/request

HTTP, DNS, SSL, Network 등 대상에 무엇을 보낼지 정의한다.

### Matcher
#명칭/template/matcher

응답이 탐지 조건과 일치하는지 true 또는 false로 판단한다.

### Extractor
#명칭/template/extractor

응답에서 버전, Token, Tenant ID 같은 필요한 값을 추출한다.

### Workflow
#명칭/template/workflow

여러 Template를 조건과 순서에 따라 연결해 실행하는 상위 규칙이다.

### Metadata
#명칭/template/metadata

Template ID, Info, 경로처럼 Template의 분류·목록·필터에 사용하는 설명 데이터다.

### Tag
#명칭/template/tag

Template를 `cve`, `exposure`, `tech` 등으로 분류하는 문자열이다.

### Severity
#명칭/template/severity

결과의 심각도를 `info`, `low`, `medium`, `high`, `critical` 등으로 나타낸 값이다.

### Parse
#명칭/template/parse

YAML·JSON 문자열을 Template 구조체의 필드로 해석하는 작업이다.

### Compile
#명칭/template/compile

파싱한 Template와 DSL 표현식이 실행 가능한지 검사하고 프로토콜 요청 실행기를 준비하는 단계다.

### raw
#명칭/template/raw

가공하기 전 원문을 뜻한다. 문맥에 따라 대상이 다르다.

```text
Parser Cache의 raw
└─ Template YAML·JSON 원문

JSON 결과의 raw
└─ 네트워크 요청·응답 원문
```

`StoreWithoutRaw`와 `-omit-raw`는 이름이 비슷하지만 서로 다른 데이터를 다룬다.

## 6. DSL 명칭 #명칭/dsl

### DSL
#명칭/dsl/dsl

Domain-Specific Language의 약자다. Nuclei Template에서 응답 값을 비교·변환하고 복잡한 탐지 조건을 작성하기 위한 전용 표현 언어다.

### DSL 표현식
#명칭/dsl/dsl-표현식

변수, 연산자, 함수로 구성된 계산·판정 문장이다.

```text
status_code == 200 && contains(body, "admin")
```

### DSL 변수
#명칭/dsl/dsl-변수

실행 중 Nuclei가 넣어 주는 값이다. `status_code`, `body`, `header` 등이 예다.

### 연산자
#명칭/dsl/연산자

값을 비교하거나 조건을 결합하는 기호다. `==`, `!=`, `&&`, `||` 등이 예다.

### DSL 함수
#명칭/dsl/dsl-함수

입력값을 받아 문자열 변환, 비교, Encoding, Hash, DNS 조회 등을 수행하고 결과를 반환한다.

예: `contains`, `to_lower`, `substr`, `base64`, `resolve`.

### 함수 서명
#명칭/dsl/함수-서명

함수 이름, 매개변수, 매개변수 타입, 반환 타입을 요약한 사용 형식이다.

```text
substr(str string, start int, optionalEnd int) string
```

### `-list-dsl-function`
#명칭/dsl/list-dsl-function

현재 Nuclei에서 사용할 수 있는 DSL 함수 서명을 출력하고 스캔 없이 종료하는 옵션이다.

### `-ldf`
#명칭/dsl/ldf

`-list-dsl-function`의 짧은 별칭이다. 같은 `Options.ListDslSignatures` 필드에 연결된다.

### `ListDslSignatures`
#명칭/dsl/listdslsignatures

DSL 함수 서명 목록을 출력할지 저장하는 `types.Options`의 Bool 필드다.

### `GetPrintableDslFunctionSignatures()`
#명칭/dsl/getprintabledslfunctionsignatures

등록된 DSL 함수 서명을 터미널에 출력 가능한 하나의 문자열로 만드는 함수다.

### `NoColor`
#명칭/dsl/nocolor

터미널 ANSI 색상 코드를 사용하지 않을지 나타내는 Options 필드다. true이면 일반 텍스트로 출력한다.

### `HelperFunctions`
#명칭/dsl/helperfunctions

DSL 함수 이름을 실제 실행 함수와 연결한 Map이다.

### `FunctionNames`
#명칭/dsl/functionnames

등록된 DSL 함수 이름만 모은 목록이다.

### DSL Registry
#명칭/dsl/dsl-registry

DSL 함수 이름, 서명, 실제 실행 함수, Cache 가능 여부를 등록해 둔 목록이다. 실제 DSL 실행과 `-ldf` 도움말 출력이 같은 등록 정보를 사용한다.

## 7. 로그 명칭 #명칭/log

### INF
#명칭/log/inf

Information. 정상적인 실행 상태와 안내 정보다.

### WRN
#명칭/log/wrn

Warning. 실행은 계속할 수 있지만 확인해야 할 문제다.

### ERR
#명칭/log/err

Error. 일부 작업이 실패했거나 입력·환경에 문제가 있음을 뜻한다.

### DBG
#명칭/log/dbg

Debug. 개발자와 분석자가 내부 동작을 추적할 때 보는 상세정보다.

### FTL 또는 Fatal
#명칭/log/ftl #명칭/log/fatal

계속 실행할 수 없는 오류다. 메시지를 출력한 뒤 프로그램을 종료하는 데 사용한다.

### Debug
#명칭/log/debug

요청·응답과 내부 Debug 정보를 자세히 표시하는 실행 설정이다.

### Verbose
#명칭/log/verbose

기본보다 더 많은 로딩·실행 정보를 표시한다.

### Silent
#명칭/log/silent

배너와 일반 상태 메시지를 줄이고 탐지 결과 중심으로 표시한다.

## 8. 출력과 데이터 형식 명칭 #명칭/출력

### JSON
#명칭/출력/json

Key와 Value 구조로 데이터를 표현하는 형식이다. 다른 프로그램이 필드를 읽기 쉽다.

### JSONL
#명칭/출력/jsonl

JSON Lines의 약자다. 한 줄마다 독립된 JSON 객체 하나를 저장한다.

### Serialization 또는 직렬화
#명칭/출력/serialization #명칭/출력/직렬화

구조체 객체를 JSON처럼 저장·전송 가능한 형식으로 바꾸는 작업이다.

### Exporter
#명칭/출력/exporter

ResultEvent를 JSON, Markdown, PDF, SARIF 등 특정 외부 형식으로 내보내는 구성요소다.

### `-omit-raw`
#명칭/출력/omit-raw

JSON, JSONL, Markdown, PDF 등의 결과에서 요청·응답 원문을 제외하는 옵션이다.

## 9. 네트워크와 보안 명칭 #명칭/보안

### OAST
#명칭/보안/oast

Out-of-Band Application Security Testing의 약자다. 대상 서버가 외부로 DNS, HTTP, SMTP, LDAP 등의 상호작용을 보내는지 확인해 취약 동작을 판단한다.

### Interactsh
#명칭/보안/interactsh

OAST 상호작용 주소를 만들고 대상의 외부 접속 기록을 Nuclei에 전달하는 ProjectDiscovery 서비스와 Client다.

### Headless
#명칭/보안/headless

화면 없는 자동화 Browser로 JavaScript 기반 페이지를 검사하는 실행 방식이다.

### Fuzzing
#명칭/보안/fuzzing

요청의 Query, Header, Body 등을 여러 값으로 바꾸어 예상하지 못한 반응과 취약점을 찾는 방식이다.

### 전자서명
#명칭/보안/전자서명

데이터와 개인키로 검증 가능한 값을 만들어 이후 내용 변경과 서명자 정보를 확인하게 하는 기술이다.

### 개인키
#명칭/보안/개인키

전자서명을 생성하는 데 사용하는 비밀 키다. 외부에 공개하면 안 된다.

### 공개키
#명칭/보안/공개키

개인키로 만든 서명이 유효한지 확인하는 데 사용하는 키다.

### Template 서명
#명칭/보안/template-서명

Template 파일이 서명 이후 변경되지 않았고 특정 키로 서명되었는지 검증할 근거를 추가하는 기능이다. Template 내용이 무조건 안전함을 보장하지는 않는다.

### PDCP
#명칭/보안/pdcp

Nuclei 문맥에서 ProjectDiscovery Cloud Platform을 뜻한다. 결과 업로드, Team, Scan ID, Cloud Dashboard 연동에 사용된다.

> 이동통신 문맥의 PDCP(Packet Data Convergence Protocol)와 약자가 같지만, Nuclei 코드에서는 ProjectDiscovery Cloud 관련 이름으로 사용된다.

## 10. 실행·성능·종료 명칭 #명칭/실행

### Execution ID
#명칭/실행/execution-id

현재 스캔 실행을 다른 실행과 구분하는 고유 문자열이다. 연결 Pool과 프로토콜 상태 정리에 사용할 수 있다.

### Profiling
#명칭/실행/profiling

프로그램이 CPU와 메모리를 어디에 사용하는지 측정·기록하는 작업이다.

### Heap
#명칭/실행/heap

프로그램이 실행 중 동적으로 만든 객체가 저장되는 메모리 영역이다.

### Memory Profile
#명칭/실행/memory-profile

어떤 타입과 객체가 Heap 메모리를 얼마나 사용하는지 분석하는 자료다.

### CPU Profile
#명칭/실행/cpu-profile

어떤 함수가 CPU 실행 시간을 많이 사용하는지 분석하는 자료다.

### Trace
#명칭/실행/trace

Goroutine 실행, 대기, Scheduling 등을 시간 흐름으로 기록한 자료다.

### Goroutine
#명칭/실행/goroutine

Go 프로그램 안에서 여러 작업을 동시에 수행하는 실행 단위다.

### Hang
#명칭/실행/hang

프로그램이 종료되지는 않았지만 같은 상태에 머물러 작업이 진행되지 않는 현상이다.

### Stack
#명칭/실행/stack

현재 어떤 함수가 어떤 함수를 호출하고 있는지 보여주는 실행 경로 정보다.

### Stack Trace
#명칭/실행/stack-trace

Goroutine 또는 실행 흐름의 함수 호출 상태를 기록한 자료다. Hang 원인 분석에 사용한다.

### Graceful Shutdown
#명칭/실행/graceful-shutdown

프로그램을 즉시 없애지 않고 Writer, Browser, RateLimiter, 임시 파일 등을 정리한 뒤 종료하는 방식이다.

### Signal
#명칭/실행/signal

운영체제가 프로세스에 전달하는 사건 알림이다. `os.Interrupt`는 Ctrl+C 중단을 나타낸다.

### Channel
#명칭/실행/channel

Goroutine 사이에서 값이나 Signal을 전달하는 통로다.

### Resume
#명칭/실행/resume

중단된 스캔 진행 상태를 파일에 저장했다가 다음 실행에서 이어가는 기능이다.

### ResumeCfg
#명칭/실행/resumecfg

Template별 완료 여부와 진행 위치 등 Resume 상태를 보관하는 구조체다.

### Close
#명칭/실행/close

객체가 사용한 파일, 네트워크, Goroutine, 임시 디렉터리 등을 정리하는 종료 메서드 이름으로 자주 사용된다.

### Purge
#명칭/실행/purge

Cache에 보관된 항목을 비우는 작업이다.

## 11. Slice 옵션의 기본값 #명칭/옵션

`readConfig()`의 TODO 주석에서 말하는 Slice·배열 옵션의 기본값은 일반적인 의미의 기본값과 다르게 동작한다.

사용자가 값을 입력하면 기존 값이 교체되는 것이 아니라 기존 Slice에 추가될 수 있다.

```text
초기값: Templates = ["default"]
사용자 입력: -t cves
결과: ["default", "cves"]
```

사용자는 `["cves"]`로 교체될 것으로 생각할 수 있지만 실제로는 기존 값 뒤에 append된다. 따라서 이 값은 엄밀히 말하면 기본값보다 초기값에 가깝고, 개발자는 혼동 가능성을 TODO로 남겼다.

## 12. 자주 사용되는 표준 라이브러리 #명칭/라이브러리

| 패키지 | 사용 목적 |
|---|---|
| `fmt` | 문자열 형식 구성과 표준 출력 |
| `os` | 환경변수, 파일, Signal, 프로그램 종료 |
| `time` | Timeout, 주기, 경과 시간 계산 |
| `path/filepath` | 운영체제에 맞는 파일 경로 처리 |
| `runtime` | Goroutine과 실행 환경 정보 |
| `runtime/pprof` | CPU·Memory Profile 기록 |
| `runtime/trace` | 실행 Trace 기록 |

## 관련 문서

- [main 전체 흐름](../../01_Main_Flow/main.md)
- [readConfig 전체 구조](../../01_Main_Flow/main_Functions/readConfig/README.md)
- [내부 패키지 설명서](../../설명서/04_내부 패키지.md)
- [템플릿과 DSL 설명서](../../설명서/06_템플릿과 DSL.md)
- [-list-dsl-function 상세](<../../설명서/14_-list-dsl-function 상세.md>)


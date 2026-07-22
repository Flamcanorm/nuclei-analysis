---
유형: 설명서
상태: false
상세: DSL을 처음 접하는 사람을 위한 -list-dsl-function 전체 코드 흐름
---

# `-list-dsl-function` 처음부터 이해하기 #전지성

<!-- main-line-tags:start -->
> [!info]- 관련 main.go 줄 태그
> 이 문서가 직접 설명하거나 다음 단계로 연결하는 main.go 줄이다. 태그를 누르면 같은 줄을 다루는 다른 설명서도 함께 찾을 수 있다.
>
> #L74
<!-- main-line-tags:end -->

<!-- glossary-links:start -->
> [!info]- 명칭 참조
> 이 문서에서 사용하는 공통 명칭은 아래 링크를 눌러 명칭 사전에서 확인할 수 있다.
>
> [패키지](<../03_External_Packages/04_etc/sample.md#패키지>) · [실행 파일](<../03_External_Packages/04_etc/sample.md#실행 파일>) · [선언](<../03_External_Packages/04_etc/sample.md#선언>) · [구조체](<../03_External_Packages/04_etc/sample.md#구조체>) · [변수](<../03_External_Packages/04_etc/sample.md#변수>) · [필드](<../03_External_Packages/04_etc/sample.md#필드>) · [함수](<../03_External_Packages/04_etc/sample.md#함수>) · [인자](<../03_External_Packages/04_etc/sample.md#인자>)
> [반환값](<../03_External_Packages/04_etc/sample.md#반환값>) · [Callback](<../03_External_Packages/04_etc/sample.md#Callback>) · [Wrapper](<../03_External_Packages/04_etc/sample.md#Wrapper>) · [Registry](<../03_External_Packages/04_etc/sample.md#Registry>) · [Map](<../03_External_Packages/04_etc/sample.md#Map>) · [CLI](<../03_External_Packages/04_etc/sample.md#CLI>) · [짧은 별칭](<../03_External_Packages/04_etc/sample.md#짧은 별칭>) · [Bool 옵션](<../03_External_Packages/04_etc/sample.md#Bool 옵션>)
> [기본값](<../03_External_Packages/04_etc/sample.md#기본값>) · [FlagSet](<../03_External_Packages/04_etc/sample.md#FlagSet>) · [flagSet.Parse()](<../03_External_Packages/04_etc/sample.md#flagSet.Parse()>) · [Runner](<../03_External_Packages/04_etc/sample.md#Runner>) · [Loader](<../03_External_Packages/04_etc/sample.md#Loader>) · [Store](<../03_External_Packages/04_etc/sample.md#Store>) · [Cache](<../03_External_Packages/04_etc/sample.md#Cache>) · [Engine](<../03_External_Packages/04_etc/sample.md#Engine>)
> [Logger](<../03_External_Packages/04_etc/sample.md#Logger>) · [ID](<../03_External_Packages/04_etc/sample.md#ID>) · [Info](<../03_External_Packages/04_etc/sample.md#Info>) · [Matcher](<../03_External_Packages/04_etc/sample.md#Matcher>) · [Tag](<../03_External_Packages/04_etc/sample.md#Tag>) · [Severity](<../03_External_Packages/04_etc/sample.md#Severity>) · [Parse](<../03_External_Packages/04_etc/sample.md#Parse>) · [Compile](<../03_External_Packages/04_etc/sample.md#Compile>)
> [DSL](<../03_External_Packages/04_etc/sample.md#DSL>) · [DSL 표현식](<../03_External_Packages/04_etc/sample.md#DSL 표현식>) · [연산자](<../03_External_Packages/04_etc/sample.md#연산자>) · [DSL 함수](<../03_External_Packages/04_etc/sample.md#DSL 함수>) · [함수 서명](<../03_External_Packages/04_etc/sample.md#함수 서명>) · [-list-dsl-function](<../03_External_Packages/04_etc/sample.md#-list-dsl-function>) · [-ldf](<../03_External_Packages/04_etc/sample.md#-ldf>) · [ListDslSignatures](<../03_External_Packages/04_etc/sample.md#ListDslSignatures>)
> [NoColor](<../03_External_Packages/04_etc/sample.md#NoColor>) · [HelperFunctions](<../03_External_Packages/04_etc/sample.md#HelperFunctions>) · [FunctionNames](<../03_External_Packages/04_etc/sample.md#FunctionNames>) · [DSL Registry](<../03_External_Packages/04_etc/sample.md#DSL Registry>) · [INF](<../03_External_Packages/04_etc/sample.md#INF>) · [Debug](<../03_External_Packages/04_etc/sample.md#Debug>) · [Verbose](<../03_External_Packages/04_etc/sample.md#Verbose>) · [JSON](<../03_External_Packages/04_etc/sample.md#JSON>)
> [전자서명](<../03_External_Packages/04_etc/sample.md#전자서명>)
<!-- glossary-links:end -->

> DSL, 함수, 함수 서명, Bool 옵션, Registry, Wrapper 등 이 문서에 나오는 공통 명칭은 [Nuclei 분석 명칭 사전](../03_External_Packages/04_etc/sample.md)에 모아 정리되어 있다. 아래 내용은 각 명칭이 L74 코드에서 어떻게 연결되는지 설명한다.

## 먼저 한 문장으로 설명

```bash
./nuclei -list-dsl-function
```

이 명령은 웹사이트를 검사하는 명령이 아니다. **Nuclei 템플릿 안에서 사용할 수 있는 계산·비교·변환 함수의 사용법 목록을 화면에 보여주는 도움말 명령**이다.

짧게 입력할 때는 다음 별칭을 쓴다.

```bash
./nuclei -ldf
```

두 명령은 같은 `Options.ListDslSignatures` 필드를 true로 만든다.

> **실제 출력 확인:** [-list-dsl-function 실행 결과](<../출력결과/-list-dsl-function.md>)

설명서는 명령이 어떤 코드로 동작하는지를 다루고, 전체 함수 목록 원문은 결과 문서에 분리해 보관한다.

## 1. DSL을 왜 알아야 하는가?

Nuclei는 Go 코드만으로 취약점 검사 규칙을 작성하지 않는다. 검사별 차이는 주로 YAML 또는 JSON 템플릿에 적는다.

```yaml
id: example-domain-check

info:
  name: Example Domain Check
  author: student
  severity: info

http:
  - method: GET
    path:
      - "{{BaseURL}}"

    matchers:
      - type: dsl
        dsl:
          - "status_code == 200 && contains(body, 'Example Domain')"
```

이 템플릿은 다음 순서로 동작한다.

```text
HTTP GET 요청
  ↓
응답 상태 코드와 본문 준비
  ↓ DSL 조건 계산
status_code == 200
  ↓ 그리고
contains(body, 'Example Domain')
  ↓
두 조건이 참이면 Matcher 성공
```

여기서 DSL은 **Domain-Specific Language**의 약자다. Nuclei 템플릿의 조건 계산이라는 특정 목적에 맞춘 표현 언어다.

## 2. 변수, 연산자, 함수의 차이

위 표현식을 세 부분으로 나누면 이해하기 쉽다.

```text
status_code == 200 && contains(body, 'Example Domain')
```

| 종류 | 예 | 뜻 |
|---|---|---|
| 변수 | `status_code`, `body` | 실행 중 Nuclei가 넣어 주는 응답 값 |
| 연산자 | `==`, `&&` | 같음 비교, AND 조건 결합 |
| 함수 | `contains(...)` | 받은 값을 계산하고 결과를 반환하는 기능 |

`-list-dsl-function`은 이 중 **함수 목록**을 보여준다. `status_code`, `body` 같은 모든 실행 변수를 출력하는 명령은 아니다.

## 3. 함수란 무엇인가?

함수는 입력값을 받아 정해진 작업을 하고 결과를 돌려준다.

```text
contains(body, "admin")
         ├─ 첫 번째 입력: 검사할 전체 문자열
         └─ 두 번째 입력: 찾을 문자열
  ↓
포함되어 있으면 true
없으면 false
```

DSL 함수의 예:

- `to_lower(value)`: 문자열을 소문자로 변환
- `contains(value, part)`: 문자열 포함 여부 확인
- `substr(value, start, end)`: 문자열 일부 추출
- `base64(value)`: 값을 Base64로 인코딩
- `resolve(host)`: Nuclei가 추가한 DNS 조회 함수

## 4. 함수 서명이란 무엇인가?

Signature는 보안 전자서명만 뜻하는 단어가 아니다. 함수 문맥에서는 **함수를 어떻게 호출하는지 나타내는 형식**을 뜻한다.

```text
substr(str string, start int, optionalEnd int) string
```

| 부분 | 뜻 |
|---|---|
| `substr` | 함수 이름 |
| `str string` | 첫 번째 인자는 문자열 |
| `start int` | 두 번째 인자는 시작 위치 정수 |
| `optionalEnd int` | 선택 가능한 끝 위치 정수 |
| 마지막 `string` | 함수가 문자열을 반환 |

따라서 `-list-dsl-function`의 설명인 “list all supported DSL function signatures”는 지원 함수의 이름과 호출 형식을 전부 보여준다는 뜻이다.

## 5. 왜 이런 목록 명령이 필요한가?

템플릿 작성자는 다음 내용을 기억하지 못할 수 있다.

- 정확한 함수 이름
- 인자 개수
- 문자열과 숫자 중 어떤 값을 받는지
- 선택 인자가 있는지
- 어떤 결과 타입을 반환하는지
- 현재 설치된 Nuclei 버전이 그 함수를 지원하는지

웹 문서를 따로 찾지 않아도 현재 실행 파일에 등록된 목록을 직접 확인할 수 있도록 만든 기능이다.

## 6. 옵션 필드는 어디에 선언되어 있는가?

`pkg/types/types.go`:

```go
type Options struct {
    ...
    ListDslSignatures bool
    ...
}
```

### 새로 등장하는 이름

- `Options`: 이번 Nuclei 실행 설정을 모은 구조체
- `ListDslSignatures`: DSL 함수 서명 목록을 출력할지 저장하는 Bool 필드
- Bool: true 또는 false 중 하나를 가지는 값

필드를 선언했을 뿐 아직 `-ldf`라는 터미널 이름과 연결된 것은 아니다.

## 7. CLI 옵션 이름과 어디서 연결되는가?

`cmd/nuclei/main.go`의 `readConfig()` Debug 그룹:

```go
flagSet.BoolVarP(
    &options.ListDslSignatures,
    "list-dsl-function",
    "ldf",
    false,
    "list all supported DSL function signatures",
)
```

각 자리는 다음 뜻이다.

| 값 | 뜻 |
|---|---|
| `&options.ListDslSignatures` | Parse한 결과를 저장할 필드 위치 |
| `"list-dsl-function"` | 긴 옵션 `-list-dsl-function` |
| `"ldf"` | 짧은 별칭 `-ldf` |
| `false` | 입력하지 않았을 때 기본값 |
| 마지막 문자열 | `-h` 도움말에 표시할 설명 |

Debug 그룹 안에 등록되었지만 `-debug`와 함께 입력해야 한다는 뜻은 아니다. 도움말에서 관련 도구를 같은 구역에 보여주기 위한 분류다.

## 8. 명령을 입력하면 값이 어떻게 바뀌는가?

```text
프로그램 시작
Options.ListDslSignatures = false
  ↓
사용자 입력: ./nuclei -ldf
  ↓
flagSet.Parse()
등록된 짧은 이름 ldf를 찾음
  ↓
연결된 Bool 필드 변경
Options.ListDslSignatures = true
```

`flagSet.Parse()`는 DSL 목록을 직접 만들지 않는다. 나중 코드가 어떤 실행 경로를 선택할 수 있도록 설정값만 바꾼다.

## 9. L74 조건문은 무엇을 하는가?
#전지성 #L74 #main/L74/dsl

```go
if options.ListDslSignatures {
    options.Logger.Info().Msgf("The available custom DSL functions are:")
    fmt.Println(dsl.GetPrintableDslFunctionSignatures(options.NoColor))
    return
}
```

### 첫 번째 줄

```go
if options.ListDslSignatures {
```

Parse 결과가 true인지 확인한다. false이면 블록 전체를 건너뛰고 템플릿 서명과 일반 실행 준비로 내려간다.

### 안내 메시지

```go
options.Logger.Info().Msgf("The available custom DSL functions are:")
```

Logger가 `[INF]` 수준의 안내 문장을 출력한다.

```text
[INF] The available custom DSL functions are:
```

Logger는 상태 메시지의 레벨과 형식을 담당한다.

### 목록 문자열 생성과 출력

```go
fmt.Println(dsl.GetPrintableDslFunctionSignatures(options.NoColor))
```

안쪽 함수부터 읽는다.

```text
options.NoColor
  ↓
dsl.GetPrintableDslFunctionSignatures(...)
등록된 DSL 함수 서명을 하나의 문자열로 만듦
  ↓
fmt.Println(...)
그 문자열을 표준 출력에 표시
```

- `options.NoColor == false`: 터미널 색상 코드 적용
- `options.NoColor == true`: 색상 코드 없는 일반 문자열

색상 없이 보려면 다음처럼 실행한다.

```bash
./nuclei -ldf -no-color
```

### return

```go
return
```

`main()`을 즉시 끝낸다. 따라서 아래 코드는 실행되지 않는다.

```text
runner.ParseOptions(options)  실행 안 됨
runner.New(options)           실행 안 됨
RunEnumeration()              실행 안 됨
HTTP·DNS·SSL 요청             실행 안 됨
```

이 명령에는 `-u https://example.com` 같은 대상이 필요 없다.

## 10. `dsl.GetPrintableDslFunctionSignatures`는 어느 파일인가?

Nuclei 내부 파일 `pkg/operators/common/dsl/dsl.go`:

```go
func GetPrintableDslFunctionSignatures(noColor bool) string {
    return dsl.GetPrintableDslFunctionSignatures(noColor)
}
```

여기에는 이름이 같은 두 DSL 계층이 있다.

```text
Nuclei 내부 dsl 패키지
pkg/operators/common/dsl
  ↓ 외부 함수를 감싸고 Nuclei 전용 함수를 추가
외부 github.com/projectdiscovery/dsl 패키지
  ↓ 등록된 함수 정보 관리
출력 가능한 함수 서명 문자열
```

Nuclei 내부 함수는 외부 DSL 라이브러리의 목록 생성 함수를 호출하는 Wrapper 역할을 한다.

## 11. Nuclei 전용 DSL 함수는 어떻게 추가되는가?

같은 `pkg/operators/common/dsl/dsl.go`의 `init()`은 Nuclei 실행 초기에 다음 함수를 외부 DSL Registry에 추가한다.

### `resolve`

호스트 이름을 A, AAAA, CNAME, NS, TXT 등의 DNS 레코드로 조회한다.

### `getNetworkPort`

현재 포트와 기본 포트를 비교해 Nuclei 네트워크 요청에서 사용할 포트를 결정한다.

### `print_debug` Callback

DSL 안에서 `print_debug`를 사용했을 때 Nuclei의 gologger Debug 메시지로 연결한다.

그 후 다음 값을 준비한다.

```go
HelperFunctions = dsl.HelperFunctions()
FunctionNames = dsl.GetFunctionNames(HelperFunctions)
```

- `HelperFunctions`: 함수 이름과 실제 실행 함수를 연결한 Map
- `FunctionNames`: 등록된 함수 이름 목록

## 12. Registry가 무엇인가?

Registry는 사용할 수 있는 기능을 이름별로 등록한 목록이다.

```text
DSL Registry
├─ 함수 이름
├─ 함수 서명
├─ 실제 계산 함수
└─ Cache 가능 여부
```

`-ldf`가 출력용 목록을 따로 손으로 작성하지 않고 Registry에서 읽는 이유는 실제 실행 가능한 함수와 도움말이 어긋나는 문제를 줄이기 위해서다.

## 13. DSL 목록과 실제 스캔은 어떻게 연결되는가?

```text
-ldf 실행
└─ Registry의 서명만 읽고 종료

일반 템플릿 스캔
└─ Template의 DSL 표현식을 Compile
   ↓
   Registry에서 함수 이름 찾기
   ↓
   실제 응답 값을 넣어 함수 실행
   ↓
   Matcher 결과 계산
```

같은 Registry를 사용하지만 `-ldf`는 함수를 실행해 웹사이트를 검사하지 않는다. 사용법을 출력할 뿐이다.

## 14. DSL 작성이 틀렸을 때

`pkg/tmplexec/exec.go`에서는 Template DSL을 Compile하다 함수 이름이나 표현식 문제가 발생하고 `-verbose`가 켜져 있으면 오류와 함께 사용 가능한 함수 목록을 출력한다.

```text
템플릿 DSL Compile 실패
  ↓ CompilationError
-verbose가 true
  ↓
오류 메시지 + DSL 함수 서명 목록
```

이 경로와 `-ldf`의 차이는 다음과 같다.

| 경우 | 목적 | 스캔 흐름 |
|---|---|---|
| 사용자가 `-ldf` 입력 | 작성 전에 함수 사용법 확인 | main 초반에 목록 후 종료 |
| 잘못된 DSL + `-verbose` | Compile 오류 수정 지원 | 템플릿 준비 중 오류로 목록 출력 |

## 15. 다른 목록 명령과 차이

| 명령 | 보여주는 것 | Runner 생성 | 대상 스캔 |
|---|---|---:|---:|
| `-ldf` | DSL 함수 서명 | 아니오 | 아니오 |
| `-tgl` | 사용 가능한 Template Tag | 예 | 아니오 |
| `-tl` | 필터를 통과한 Template 목록 | 예 | 아니오 |
| `-u ...` | 대상에 대한 실제 탐지 결과 | 예 | 예 |

`-tgl`과 `-tl`은 Template Store와 Loader가 필요하므로 `RunEnumeration()` 안에서 처리한다. `-ldf`는 이미 등록된 함수 목록만 필요하므로 Runner를 만들기 전에 처리한다.

## 16. 처음 보는 사람이 기억할 최종 요약

```text
DSL
└─ Nuclei Template 조건을 작성하는 전용 표현 언어

DSL 함수
└─ 문자열 변환, 비교, 인코딩, DNS 조회 등을 수행

함수 서명
└─ 함수 이름과 인자·반환값 사용법

-list-dsl-function / -ldf
└─ 현재 Nuclei에 등록된 DSL 함수 서명을 출력하는 도움말 명령

L74의 return
└─ 목록만 보여주고 Runner·Engine·스캔 없이 프로그램 종료
```

따라서 이 코드는 “DSL 함수로 대상을 검사하는 코드”가 아니라, **템플릿 작성자가 어떤 DSL 함수를 쓸 수 있는지 알려주는 자체 설명 기능**이다.

다음에는 [-list-dsl-function 실행 결과](<../출력결과/-list-dsl-function.md>)로 이동해 실제 함수 이름과 서명 형식을 확인할 수 있다.


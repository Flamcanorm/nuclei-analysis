---
유형: 설명서
상태: false
상세: Nuclei 템플릿 구조, DSL, -list-dsl-function 전체 흐름
---

# 템플릿과 DSL 설명서 #전지성

## 템플릿이란?

Nuclei 템플릿은 “어떤 요청을 보내고 어떤 응답이면 탐지로 볼 것인지”를 YAML 또는 JSON으로 정의한 검사 규칙이다.

```text
Nuclei 실행 파일
└─ 요청 전송, 병렬 실행, 속도 제한, 입력·출력 등의 공통 엔진

Nuclei 템플릿
└─ 검사 대상 요청과 탐지 조건
```

실행 파일만으로는 어떤 취약점을 검사할지 알 수 없다. 템플릿이 구체적인 검사 방법을 제공한다.

## 템플릿의 주요 구성

### ID

템플릿을 구별하는 고유 이름이다. 결과의 첫 번째 대괄호에 표시된다.

```yaml
id: example-security-header
```

### Info

사람이 템플릿의 목적과 위험도를 이해할 수 있는 설명 정보다.

```yaml
info:
  name: Example Security Header Check
  author: student
  severity: info
  tags: header,example
```

- `name`: 사람이 읽는 검사 이름
- `author`: 템플릿 작성자
- `severity`: info, low, medium, high, critical 같은 심각도
- `tags`: 템플릿을 분류하고 `-tags`로 선택하는 키워드

### 요청 정의

대상에 무엇을 보낼지 정한다.

- `http`: HTTP 요청
- `dns`: DNS 질의
- `ssl`: 인증서와 TLS 검사
- `network`: TCP 등 네트워크 요청
- `headless`: 브라우저 동작
- 그 외 file, code, javascript, websocket, whois 등

### Matchers

응답이 탐지 조건과 일치하는지 판단한다.

예:

- 상태 코드가 200인가?
- 응답 본문에 특정 문자열이 있는가?
- 응답 헤더가 특정 정규식과 일치하는가?
- 여러 조건이 모두 참인가?

### Extractors

응답에서 필요한 값을 뽑아낸다. 취약 여부 자체를 판단하는 Matcher와 달리, Extractor는 버전, 토큰, 이메일, Tenant ID 같은 값을 결과에 붙이는 데 사용한다.

## YAML과 JSON을 모두 지원하는 이유

둘 다 같은 Template 구조체로 변환할 수 있지만 사용 목적이 다르다.

- YAML: 사람이 직접 읽고 작성하기 편함
- JSON: 프로그램이 자동 생성·변환·전송하기 편함

Nuclei 제작자는 사람이 만든 템플릿뿐 아니라 다른 도구나 자동화 시스템이 생성한 템플릿도 입력으로 받을 수 있게 두 형식을 지원한다. Parser 이후에는 둘 다 같은 Template 객체가 되므로 Engine은 원래 형식을 크게 신경 쓰지 않는다.

## DSL이란?

DSL은 Domain-Specific Language, 즉 **특정 목적에 맞춘 작은 표현 언어**다. Nuclei에서는 템플릿 안에서 응답 값을 비교하고 변환해 복잡한 조건을 작성하는 데 사용한다.

Go 코드를 다시 컴파일하지 않고 템플릿에 조건식을 적을 수 있다는 것이 핵심이다.

```yaml
matchers:
  - type: dsl
    dsl:
      - "status_code == 200"
      - "contains(body, 'Example Domain')"
    condition: and
```

- `status_code`: 응답 상태 코드가 담긴 변수
- `body`: 응답 본문이 담긴 변수
- `contains(...)`: 문자열 포함 여부를 계산하는 DSL 함수
- `==`, `and`: 비교와 논리 결합

`status_code`와 `body`는 함수가 아니라 실행 중 Nuclei가 제공하는 값이다.

## 함수 서명이란?

함수 서명은 함수를 올바르게 부르는 형식을 요약한 문장이다.

```text
substr(str string, start int, optionalEnd int)
```

이 서명에서 알 수 있는 내용은 다음과 같다.

- 함수 이름: `substr`
- 첫 번째 인자: 자를 문자열 `str`
- 두 번째 인자: 시작 위치 정수 `start`
- 세 번째 인자: 선택적인 끝 위치 정수 `optionalEnd`

반환 형식이 표시된 서명은 함수 실행 결과가 문자열인지 숫자인지도 알려준다.

## -list-dsl-function이란?

정식 긴 옵션은 `-list-dsl-function`, 짧은 별칭은 `-ldf`다.

```bash
./nuclei -list-dsl-function
./nuclei -ldf
```

두 명령은 같은 기능을 실행한다. 현재 설치된 Nuclei에서 템플릿에 사용할 수 있는 DSL 함수의 이름과 서명을 출력한다.

이 옵션은 다음 상황에서 사용한다.

- 템플릿에서 사용할 함수 이름을 모를 때
- 함수에 인자를 몇 개 전달해야 하는지 확인할 때
- 인자가 문자열인지 숫자인지 확인할 때
- Nuclei 버전 변경으로 사용 가능한 함수가 달라졌는지 확인할 때
- DSL 컴파일 오류를 수정할 때

## DSL 함수가 등록되고 출력 목록에 들어가는 구조

외부 DSL 모듈은 함수 정보를 내부 목록에 저장한다. 각 정보에는 이름, 서명, 실제 실행 함수, Cache 가능 여부가 들어간다.

```text
dslFunction
├─ Name
├─ Signatures
├─ ExpressionFunction
└─ IsCacheable
```

Nuclei의 `pkg/operators/common/dsl/dsl.go`는 초기화 시 다음 작업을 한다.

1. 외부 DSL 모듈에 Nuclei 전용 `resolve` 등록
2. Nuclei 전용 `getNetworkPort` 등록
3. `print_debug`가 gologger를 사용하도록 Callback 연결
4. 전체 HelperFunctions 맵과 함수 이름 목록 준비

`GetPrintableDslFunctionSignatures()`는 실제 함수를 실행하지 않는다. 등록된 함수 정보에서 서명 문자열만 모아 색상 적용 여부에 맞게 합친다.

이 Registry 방식의 장점은 새 함수를 추가할 때 `main.go`의 출력 코드를 고칠 필요가 없다는 점이다. 함수 등록 목록이 바뀌면 `-ldf` 출력도 같은 목록을 사용해 자동으로 바뀐다.

> **설계 해석:** 목록을 수동 문자열로 따로 관리했다면 실제 실행 가능한 함수와 도움말 목록이 어긋날 위험이 있다. 동일한 함수 Registry에서 실행 맵과 출력 서명을 만들기 때문에 그 불일치 가능성을 줄인다.

## 코드에서의 전체 이동 경로

### 1. 옵션 등록

`cmd/nuclei/main.go`의 Debug 그룹에서 등록된다.

```go
flagSet.BoolVarP(
    &options.ListDslSignatures,
    "list-dsl-function",
    "ldf",
    false,
    "list all supported DSL function signatures",
)
```

Debug 그룹에 보인다는 것은 도움말에서 Debug 영역에 묶인다는 뜻이다. `-debug`를 함께 켜야 작동한다는 뜻은 아니다.

### 2. 사용자 입력 파싱

`flagSet.Parse()`가 `-ldf` 또는 `-list-dsl-function`을 발견하면 다음 값을 바꾼다.

```text
options.ListDslSignatures
false → true
```

이 필드는 `pkg/types/types.go`의 `Options`에 선언되어 있다.

### 3. main에서 확인

`readConfig()`가 끝난 직후 다음 코드가 실행된다.

```go
if options.ListDslSignatures {
    options.Logger.Info().Msgf("The available custom DSL functions are:")
    fmt.Println(dsl.GetPrintableDslFunctionSignatures(options.NoColor))
    return
}
```

각 줄의 역할은 다음과 같다.

1. `if`: 옵션 값이 true인지 확인한다.
2. `Logger.Info()`: 목록을 출력한다는 안내 메시지를 표시한다.
3. `GetPrintableDslFunctionSignatures(...)`: 출력 가능한 함수 서명 문자열을 만든다.
4. `fmt.Println(...)`: 문자열을 터미널에 출력한다.
5. `return`: `main()`을 종료한다.

따라서 Runner 생성과 `RunEnumeration()`까지 내려가지 않으며 대상에 스캔 요청을 보내지 않는다. `-u`가 없어도 사용할 수 있는 참고 명령이다.

### 4. Nuclei의 DSL 연결 함수

`pkg/operators/common/dsl/dsl.go`에는 다음 래퍼 함수가 있다.

```go
func GetPrintableDslFunctionSignatures(noColor bool) string {
    return dsl.GetPrintableDslFunctionSignatures(noColor)
}
```

이 파일의 패키지 이름도 `dsl`이고 외부 라이브러리 이름도 `dsl`이라 처음 보면 혼동하기 쉽다.

```text
Nuclei 내부 dsl 패키지
pkg/operators/common/dsl
        ↓ 호출
외부 projectdiscovery/dsl 라이브러리
        ↓
등록된 함수 서명 모음 생성
```

Nuclei 내부 계층은 외부 라이브러리 함수뿐 아니라 `resolve`, `getNetworkPort` 같은 Nuclei 전용 함수도 초기화 단계에서 등록한다.

### 5. noColor

`options.NoColor`는 ANSI 색상 코드를 넣을지 결정한다.

- `false`: 함수명과 타입을 색으로 구분해 터미널에 출력
- `true`: 색상 제어 문자가 없는 일반 텍스트 출력

색상 없는 결과가 필요하면 다음처럼 실행할 수 있다.

```bash
./nuclei -ldf -no-color
```

파일로 저장하거나 색상을 지원하지 않는 터미널에서 볼 때 유용하다.

## 실제로 어떤 함수가 있는가?

버전에 따라 목록이 달라지므로 `-ldf`의 출력이 가장 정확하다. v0.8.20 DSL 코드에서 확인되는 함수 범주는 다음과 같다.

- 문자열: `to_upper`, `to_lower`, `replace`, `split`, `join`, `substr`, `contains_all`
- 인코딩·디코딩: `base64`, `url_encode`, `hex_encode`, `html_escape`
- 압축: `gzip_decode`, `zlib_decode`, `inflate`
- 비교·정규식: `regex_all`, `regex_any`, `equals_any`, `compare_versions`
- 난수·시간: `rand_text_alpha`, `rand_int`, `unix_time`, `date_time`
- 데이터 형식: `json_minify`, `json_prettify`, `generate_jwt`
- 네트워크·보안: `public_ip`, `jarm`, `rsa_encrypt`
- Nuclei 추가 함수: `resolve`, `getNetworkPort`

모든 함수가 단순 문자열 계산만 하는 것은 아니다. DNS 조회나 외부 연결, 난수 생성처럼 실행할 때 환경의 영향을 받는 함수도 있으므로 서명과 용도를 확인해야 한다.

## DSL 오류가 날 때도 목록을 출력하는 경로

`pkg/tmplexec/exec.go`에서는 템플릿의 DSL 표현식을 컴파일하다 오류가 발생하고 `-verbose`가 켜져 있으면, 오류 메시지 뒤에 같은 함수 서명 목록을 출력한다.

이 경로는 사용자가 직접 `-ldf`를 입력한 경우와 목적이 다르다.

- `-ldf`: 스캔 전에 사용 가능한 함수 목록만 요청
- DSL 컴파일 오류 + `-verbose`: 잘못 작성한 템플릿을 고치도록 참고 목록 제공

## 작성 예시

```yaml
id: example-dsl-check

info:
  name: Example DSL Check
  author: student
  severity: info
  tags: example,dsl

http:
  - method: GET
    path:
      - "{{BaseURL}}"

    matchers-condition: and
    matchers:
      - type: dsl
        dsl:
          - "status_code == 200"
          - "contains(to_lower(body), 'example domain')"
```

처리 순서는 다음과 같다.

1. HTTP 요청을 보낸다.
2. 응답 상태 코드와 본문을 변수로 준비한다.
3. `to_lower(body)`가 본문을 소문자로 바꾼다.
4. `contains(...)`가 문자열 포함 여부를 반환한다.
5. 두 조건이 모두 참이면 Matcher가 매칭으로 판단한다.

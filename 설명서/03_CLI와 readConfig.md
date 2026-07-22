---
유형: 설명서
상태: false
상세: CLI 옵션 등록, 파싱, Options 저장, 후처리 구조
---

# CLI와 readConfig 설명서 #전지성

## 앞 문서에서 이어지는 내용

[[02_Main과 프로그램 시작 흐름]]에서 `main()`이 `readConfig()`를 호출한 뒤 완성된 `options`를 `runner.New(options)`에 전달한다고 설명했다. 이 문서는 그중 **사용자의 터미널 문자열이 Options 필드로 바뀌는 구간**만 확대해서 본다.

## 이 문서에서 처음 나오는 이름

| 이름 | 선언·출처 | 뜻과 연결 |
|---|---|---|
| `readConfig()` | `cmd/nuclei/main.go` | CLI 옵션을 등록하고 Parse한 뒤 Config·Profile을 병합하는 함수 |
| `goflags` | 외부 패키지 | 옵션 등록, 도움말 그룹, 명령줄 파싱 기능 제공 |
| `FlagSet` | goflags의 타입 | 이번 프로그램에 등록된 모든 옵션 정보 보관 |
| `CreateGroup()` | FlagSet의 메서드 | Target, Output, Debug처럼 도움말과 옵션을 분류 |
| `BoolVarP()` | FlagSet의 메서드 | Bool 필드에 긴 옵션과 짧은 별칭을 연결 |
| `flagSet.Parse()` | FlagSet의 메서드 | 실제 명령줄을 읽어 연결된 Options 필드를 변경 |
| Config | 설정 파일 | 자주 쓰는 옵션을 파일로 보관해 CLI 값과 병합 |
| Template Profile | Nuclei 프로필 파일 | 템플릿 선택·필터·인증 등 여러 설정을 한 묶음으로 재사용 |

`FlagSet`은 스캔 엔진이 아니다. 입력 가능한 옵션의 이름·타입·저장 위치를 기억하다가 `Parse()`에서 값을 채우는 **CLI 전용 관리자**다.

## CLI란 무엇인가?

CLI는 Command Line Interface의 약자다. 사용자가 화면의 버튼을 누르는 대신 터미널에 명령과 옵션을 입력해 프로그램을 조작하는 방식이다.

```bash
./nuclei -u https://example.com -tags cve -o result.txt
```

- `./nuclei`: 실행할 프로그램
- `-u`: 다음 값이 스캔 대상임을 나타내는 옵션
- `https://example.com`: `-u`에 전달하는 값
- `-tags cve`: `cve` 태그 템플릿만 선택하는 필터
- `-o result.txt`: 탐지 결과를 파일에 저장

## readConfig가 하는 일

`readConfig()`는 이름만 보면 설정 파일 하나를 읽는 함수처럼 보이지만 실제 역할은 더 크다.

1. `goflags.NewFlagSet()`으로 옵션 관리자 `flagSet`을 만든다.
2. `CreateGroup()`으로 옵션을 Target, Templates, Output, Debug 등으로 분류한다.
3. `StringVar`, `BoolVar`, `IntVar` 등으로 옵션 이름과 저장 위치를 연결한다.
4. `flagSet.Parse()`로 실제 터미널 입력을 해석한다.
5. Config 파일과 Template Profile을 병합한다.
6. 서로 충돌하거나 잘못된 옵션을 검사하고 값을 보정한다.

앞 문서의 흐름과 합치면 다음 위치다.

```text
main() 시작
  ↓
runner.ConfigureOptions()
goflags의 파일 판별 규칙 준비
  ↓
readConfig()
FlagSet 생성 → 옵션 그룹 등록 → Parse → Config/Profile 병합
  ↓
options 전역변수의 필드가 완성됨
  ↓
runner.ParseOptions(options)
실행 전 최종 검사·보정
  ↓
runner.New(options)
```

`ConfigureOptions()`와 `readConfig()`는 이름이 비슷해도 역할이 다르다. 앞 함수는 goflags의 공통 파일 판별 방식을 먼저 설정하고, 뒤 함수는 실제 Nuclei CLI 옵션을 등록하고 읽는다.

## 옵션 등록과 파싱의 차이

옵션 등록은 “이 프로그램이 어떤 옵션을 받을 수 있는지” 정의하는 단계다.

```go
flagSet.BoolVarP(
    &options.ListDslSignatures,
    "list-dsl-function",
    "ldf",
    false,
    "list all supported DSL function signatures",
)
```

이 코드는 다음 정보를 등록한다.

| 자리 | 값 | 의미 |
|---|---|---|
| 1 | `&options.ListDslSignatures` | 결과를 저장할 곳 |
| 2 | `list-dsl-function` | 긴 옵션 이름 `-list-dsl-function` |
| 3 | `ldf` | 짧은 별칭 `-ldf` |
| 4 | `false` | 사용자가 입력하지 않았을 때 기본값 |
| 5 | 설명 문자열 | 도움말에 표시할 문장 |

긴 이름은 뜻이 분명해 처음 보는 사람이 이해하기 쉽고, 짧은 이름은 반복 입력하기 편하도록 함께 제공한다. 두 이름은 같은 저장 위치를 사용하므로 동작은 같다.

파싱은 실제로 입력된 문자열을 등록 정보와 대조해 값을 넣는 단계다.

```text
등록만 끝난 상태
ListDslSignatures = false

사용자 입력
./nuclei -ldf

flagSet.Parse() 이후
ListDslSignatures = true
```

## 옵션 하나가 실제 필드와 연결되는 과정

`-ldf`를 예로 들면 선언은 두 파일에 나뉜다.

### 값을 저장할 필드 선언

`pkg/types/types.go`:

```go
type Options struct {
    ...
    ListDslSignatures bool
    ...
}
```

이 선언은 `Options`가 Bool 값 하나를 보관할 공간을 가진다는 뜻이다. 아직 `-ldf`라는 CLI 이름과는 연결되지 않았다.

### CLI 이름과 필드 연결

`cmd/nuclei/main.go`의 `readConfig()`:

```go
flagSet.BoolVarP(
    &options.ListDslSignatures,
    "list-dsl-function",
    "ldf",
    false,
    "list all supported DSL function signatures",
)
```

`&options.ListDslSignatures`가 저장 위치를 가리키므로, `Parse()`는 별도의 반환값을 만들지 않고 해당 필드를 직접 바꾼다.

### 값을 사용하는 코드

다시 `main()`으로 돌아와 다음 조건이 필드를 읽는다.

```go
if options.ListDslSignatures {
    ...
}
```

따라서 한 옵션의 전체 연결은 다음과 같다.

```text
Options에 필드 선언
  ↓
readConfig에서 CLI 이름·기본값과 연결
  ↓
flagSet.Parse가 사용자 입력을 필드에 저장
  ↓
main 또는 Runner가 필드를 읽어 동작 선택
```

이 패턴은 `Debug`, `Targets`, `Output`, `Tags` 등 대부분 옵션에 반복된다.

## 왜 옵션 등록을 Options 구조체 선언과 분리했는가?

`Options`는 실행 설정의 데이터 모양을 정의하고, `readConfig()`는 CLI에서 그 데이터를 채우는 방법을 정의한다.

이 둘을 분리하면 다음 장점이 있다.

- CLI가 아닌 SDK 코드도 `Options`를 직접 만들어 Runner에 전달할 수 있다.
- 구조체 필드는 유지하면서 CLI 별칭이나 도움말 문구만 바꿀 수 있다.
- Config 파일, 환경변수, Template Profile처럼 CLI 이외의 설정 원천도 같은 Options에 병합할 수 있다.

반대로 단점도 있다. 필드 선언과 옵션 등록이 다른 파일에 있으므로 초보자는 두 곳을 함께 찾아야 전체 의미를 알 수 있다. 그래서 분석할 때는 항상 `필드 선언 → 등록 → Parse → 사용처` 순서로 검색해야 한다.

## flagSet.Parse가 하는 일

`flagSet.Parse()`는 명령줄 문자열을 왼쪽부터 읽으면서 다음 작업을 수행한다.

1. 옵션 이름이 등록되어 있는지 찾는다.
2. Bool, String, Int, StringSlice 등 옵션 종류를 확인한다.
3. 필요한 값이 뒤에 있는지 확인한다.
4. 문자열 값을 알맞은 형태로 바꾼다.
5. 등록할 때 연결한 `options` 필드에 저장한다.
6. 알 수 없는 옵션이나 잘못된 값이면 오류를 만든다.

`Parse()`가 스캔을 실행하는 것은 아니다. 이후 코드가 읽을 설정값을 완성하는 역할이다.

## Options가 필요한 이유

파싱된 값을 전역적으로 아무 곳에나 흩어 놓으면 각 구성요소가 필요한 값을 찾기 어렵다. `Options`에 모으면 Runner, Engine, Writer 등이 동일한 실행 설정을 전달받을 수 있다.

```text
CLI 입력
  ↓ flagSet.Parse()
Options에 값 저장
  ├─ Runner가 실행 방식 확인
  ├─ Loader가 템플릿 필터 확인
  ├─ Engine이 동시성·속도 확인
  └─ Writer가 출력 형식·파일 확인
```

## 18개 옵션 그룹

### Target

어디를 검사할지 정한다. 단일 URL, 여러 호스트가 든 파일, 제외 대상, IP 버전 등이 포함된다.

### Target-Format

입력 문자열을 URL, host, JSONL 등 어떤 형식으로 해석할지 정한다.

### Templates

어떤 템플릿 파일이나 템플릿 디렉터리를 사용할지 정한다. Template Profile과 AI 템플릿 관련 설정도 포함될 수 있다.

### Filtering

로드 가능한 템플릿 중 어떤 것을 남길지 정한다. `tags`, `severity`, `author`, `id`, `exclude-tags` 등이 대표적이다.

### Output

결과를 터미널, 일반 텍스트, JSON, JSONL 등 어떤 형태로 보낼지 정한다. `-o`, `-json-export`, `-omit-raw` 등이 여기에 해당한다.

### Configurations

Config 파일, timeout, retry, redirect, matcher 상태 등 공통 동작을 설정한다.

### Interactsh

대상이 Nuclei의 외부 상호작용 서버로 DNS 또는 HTTP 요청을 보내는지를 이용한 탐지를 설정한다.

### Fuzzing

요청의 query, header, body 등을 여러 입력값으로 바꾸어 취약 반응을 찾는 동작을 설정한다.

### Uncover

Shodan 같은 외부 검색 서비스를 통해 스캔 대상을 찾아오는 기능을 설정한다.

### Rate-Limit

초당 요청 수와 동시에 실행할 작업 수를 제한한다. 대상 서버와 로컬 컴퓨터에 과도한 부하가 생기는 것을 막는 데 중요하다.

### Optimizations

동일한 요청을 묶는 클러스터링, 요청 순서, 메모리 절약 등 실행 효율을 조절한다.

### Headless

일반 HTTP 요청만으로 처리하기 어려운 JavaScript 기반 페이지를 브라우저로 실행한다.

### Debug

요청·응답, 상세 로그, 추적 파일, DSL 함수 목록처럼 문제 확인에 필요한 기능을 제공한다. `-list-dsl-function`은 이 그룹에 등록되어 있지만, 요청·응답을 출력하는 `-debug`와는 별개의 참고 명령이다.

### Update

Nuclei 실행 파일과 nuclei-templates 업데이트를 관리한다.

### Honeypot

대상이 실제 서비스인지 탐지 유도용 허니팟인지 판단하는 기능과 관련된다.

### Statistics

진행률, 요청 수, 매칭 수 같은 실행 통계를 표시한다.

### Cloud

결과를 ProjectDiscovery Cloud에 업로드하거나 Cloud 팀·자산과 연결한다.

### Authentication

인증 정보와 비밀값 파일을 읽고 템플릿 요청에 적용한다.

## PostProcessing이 필요한 이유

CLI를 문법적으로 파싱했다고 해서 바로 안전하게 실행할 수 있는 것은 아니다.

예를 들어 다음 처리가 더 필요하다.

- 더 이상 권장하지 않는 옛 옵션을 새 옵션으로 바꾸기
- Debug, Verbose, Silent 조합에 맞게 Logger 출력 범위 설정
- Config 파일과 CLI 입력의 우선순위 정하기
- Template Profile 안의 옵션 병합하기
- 필요한 파일이 실제로 존재하는지 확인하기
- 동시에 쓸 수 없는 옵션 조합 거부하기

따라서 흐름은 다음과 같다.

```text
옵션 등록
  ↓
flagSet.Parse()
  ↓
Options에 사용자 입력 저장
  ↓
Config/Profile 병합
  ↓
검증과 보정
  ↓
Runner에 최종 Options 전달
```

다음 [[04_내부 패키지]]에서는 이렇게 완성된 `*types.Options`가 Runner 구조체의 `options` 필드에 저장되고, 다시 Loader용 `loader.Config`와 프로토콜용 `ExecutorOptions`로 나뉘어 전달되는 과정을 설명한다.

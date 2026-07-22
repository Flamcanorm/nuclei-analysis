---
유형: 설명서
상태: false
상세: cmd/nuclei/main.go와 main 관련 문서의 역할
---

# Main과 프로그램 시작 흐름 #전지성

[[01_프로젝트와 폴더 지도]]에서 `01_Main_Flow`가 프로그램 시작 코드를 정리한 폴더라고 설명했다. 이제 그 폴더가 참고하는 실제 원본 파일 `cmd/nuclei/main.go`를 따라간다.

## 이 문서에서 처음 나오는 이름

| 이름 | 선언·위치 | 뜻 |
|---|---|---|
| `cmd/nuclei/main.go` | 원본 소스 파일 | CLI 실행 프로그램의 시작 흐름을 모은 파일 |
| `main()` | `main.go`의 함수 | 운영체제가 Nuclei를 실행할 때 처음 진입하는 함수 |
| `types.Options` | `pkg/types/types.go`의 구조체 | 이번 실행의 Target, Debug, Tags, Output 같은 설정 모음 |
| `options` | `main.go`의 전역변수 | 실제 `types.Options` 객체를 가리키며 `readConfig()`가 값을 채움 |
| `Runner` | `internal/runner/runner.go`의 구조체 | Parser, Catalog, InputProvider, Writer 같은 실행 구성요소 참조를 보관 |
| `runner.New()` | `internal/runner/runner.go`의 함수 | Options를 받아 준비가 끝난 Runner를 생성 |
| `RunEnumeration()` | Runner의 메서드 | 템플릿 로딩과 Engine 실행 흐름을 시작 |

여기서 `types`와 `runner`는 파일명이 아니라 가져온 패키지 이름이다. 따라서 `types.Options`는 `types` 패키지 안의 `Options` 타입, `runner.New`는 `runner` 패키지 안의 `New` 함수를 뜻한다.

## main.go의 역할

`cmd/nuclei/main.go`는 Nuclei CLI 프로그램의 시작점이다. 사용자가 터미널에서 `./nuclei ...`를 실행하면 운영체제가 Nuclei 실행 파일을 시작하고, 프로그램은 `main()`에서 작업을 시작한다.

`main()`이 HTTP 요청이나 DNS 질의를 직접 전부 수행하지는 않는다. 대신 다음 작업을 순서대로 연결한다.

1. 로그를 출력할 Logger를 준비한다.
2. CLI에서 사용할 전역 설정을 준비한다.
3. `readConfig()`로 옵션을 등록하고 사용자 입력을 파싱한다.
4. 버전 출력, DSL 목록 출력, 템플릿 서명처럼 스캔이 필요 없는 명령을 먼저 처리한다.
5. 옵션을 검사하고 필요한 Runner를 생성한다.
6. `RunEnumeration()`으로 실제 스캔을 시작한다.
7. 종료 신호와 임시 파일을 정리한다.

## main은 지휘자가 아니라 시작 버튼에 가깝다

기존 문서에서 “Runner가 전체 스캔을 지휘한다”는 표현은 다음 뜻이다.

- `main`: 프로그램을 시작하고 Runner에 설정을 전달한다.
- `Runner`: Catalog, Loader, Engine, Writer 등 필요한 구성요소를 만들고 연결한다.
- `Engine`: 실제로 대상과 템플릿을 조합해 실행한다.

즉 `main()`이 모든 세부 동작을 직접 수행하는 것이 아니라, 역할별 객체를 생성하고 다음 단계로 넘긴다.

## 위 설명을 코드 구조로 풀어보기

### 1. main은 어디에 선언되어 있는가?

파일은 `cmd/nuclei/main.go`이고 시작 함수는 다음과 같이 선언된다.

```go
func main() {
    ...
}
```

`main()`은 다른 코드가 이름으로 호출하는 일반 보조 함수가 아니다. 운영체제가 Nuclei 실행 파일을 시작하면 Go 실행 환경이 진입점으로 호출한다.

### 2. main이 직접 보관하는 전역 변수

같은 파일 위쪽에는 다음 변수가 선언되어 있다.

```go
var (
    cfgFile                string
    templateProfile        string
    memProfile             string
    options                = &types.Options{}
    inlineSecretsTempFiles []string
)
```

| 변수 | 담는 값 | main과의 연결 |
|---|---|---|
| `cfgFile` | Config 파일 경로 | `readConfig()`가 설정 파일 옵션을 연결함 |
| `templateProfile` | Template Profile 경로 또는 ID | `readConfig()` 후처리에서 프로필을 찾고 병합함 |
| `memProfile` | CPU·메모리 분석 결과를 저장할 기본 경로 | 값이 있으면 `main()`이 pprof와 trace를 시작함 |
| `options` | 이번 실행의 전체 설정 | `readConfig()`가 값을 채우고 `runner.New()`가 받음 |
| `inlineSecretsTempFiles` | 임시 인증 파일 경로들 | `defer`와 종료 처리 코드가 삭제함 |

이 값들이 파일 범위에 있는 이유는 `main()`뿐 아니라 같은 파일에 정의된 `readConfig()`, Callback, 프로필 처리 함수들이 함께 접근해야 하기 때문이다. 다만 전역 상태는 어디서 값이 바뀌었는지 추적하기 어려워질 수 있으므로, 핵심 실행 설정은 하나의 `Options` 구조체로 묶어 전달한다.

### 3. main이 Runner를 만드는 정확한 위치
<!-- main-line-section-tags: #L177 -->
#L177

```go
nucleiRunner, err := runner.New(options)
```

여기서 연결되는 요소는 세 가지다.

```text
main 패키지의 options 변수
        ↓ 인자로 전달
internal/runner의 New 함수
        ↓ 생성 결과
*runner.Runner를 가리키는 nucleiRunner 지역변수
```

`nucleiRunner`는 Runner 구조체 전체를 복사해 담는 변수가 아니라, 생성된 Runner를 가리키는 포인터를 받는다. 따라서 이후 `RunEnumeration()`, `Close()`, `SaveResumeConfig()`는 같은 Runner와 그 안의 같은 구성요소를 사용한다.

### 4. 함수 선언과 실제 연결
<!-- main-line-section-tags: #L69 #L168 #L177 #L237 -->
#L69 #L168 #L177 #L237

| 호출 위치 | 호출 함수 | 전달하는 값 | 결과 |
|---|---|---|---|
| `main.go` | `runner.ConfigureOptions()` | 없음 | goflags의 파일 판별 규칙 설정 |
| `main.go` | `readConfig()` | 전역 변수에 간접 접근 | Options와 보조 설정 완성 |
| `main.go` | `runner.ParseOptions(options)` | `*types.Options` | 파싱된 옵션 보정·검사 |
| `main.go` | `runner.New(options)` | `*types.Options` | `*Runner`, `error` |
| `main.go` | `nucleiRunner.RunEnumeration()` | Runner가 이미 가진 필드 사용 | 실제 스캔 또는 목록 기능 실행 |
| `main.go` | `nucleiRunner.Close()` | 없음 | Writer, InputProvider, 임시 자원 정리 |

### 5. main이 Engine을 직접 만들지 않는 이유
<!-- main-line-section-tags: #L177 #L237 -->
#L177 #L237

코드상 Engine은 `RunEnumeration()` 안에서 다음처럼 생성된다.

```go
executorEngine := core.New(r.options)
executorEngine.SetExecuterOptions(executorOpts)
```

이는 다음과 같은 책임 분리를 만든다.

- `main()`은 CLI 프로그램의 생명주기만 담당한다.
- `Runner.New()`는 오래 사용되는 공통 구성요소를 초기화한다.
- `RunEnumeration()`은 템플릿 실행 직전에 필요한 실행용 객체를 조립한다.
- `Engine`은 이미 준비된 입력과 템플릿을 실행한다.

설계상 이 분리는 Nuclei를 CLI뿐 아니라 SDK나 서버 모드에서 재사용하기 쉽게 만든다. 만약 `main()` 안에 Catalog, Parser, Engine 생성 코드가 모두 들어 있으면 CLI가 아닌 다른 진입점에서 같은 초기화 코드를 다시 작성해야 한다.

> **설계 해석:** “SDK 재사용을 위해서만 이렇게 만들었다”는 주석이 모든 위치에 명시된 것은 아니다. 그러나 코드가 `types.Options`, `Runner`, `Engine`으로 계층화되어 있고 `lib/sdk.go`에도 별도 엔진 진입점이 존재하므로, 진입점과 실행 엔진의 결합을 줄이는 효과가 있다는 해석은 코드 구조와 일치한다.

## 시작부의 핵심 값

### options

`options`는 `types.Options` 구조체를 가리킨다. CLI에서 받은 설정을 한곳에 모아 여러 구성요소가 공유하도록 만든 데이터 묶음이다.

예를 들어 다음 입력을 실행한다고 가정한다.

```bash
./nuclei -u https://example.com -debug -o result.txt
```

파싱 후에는 개념적으로 다음 값이 채워진다.

```text
Options
├─ Targets = [https://example.com]
├─ Debug = true
└─ Output = result.txt
```

이후 Runner와 Writer가 같은 `Options`를 전달받기 때문에 대상, 디버그 여부, 출력 파일을 다시 물어볼 필요가 없다.

### Logger
<!-- main-line-section-tags: #L58 #L168 -->
#L58 #L168

Logger는 프로그램 상태를 사람에게 알리는 출력 담당자다. 스캔 탐지 결과와는 구분된다.

```text
[INF] 정보
[WRN] 경고
[ERR] 오류
[DBG] 디버그 상세정보
```

### inlineSecretsTempFiles
<!-- main-line-section-tags: #L60 -->
#L60

프로필 안에 직접 적힌 인증 정보를 임시 파일로 바꾸어 사용할 때, 생성된 파일 경로를 기억하는 목록이다. `defer` 정리 코드가 프로그램 종료 시 이 파일들을 삭제한다.

## ConfigureOptions와 readConfig
<!-- main-line-section-tags: #L69 -->
#L69

```go
if err := runner.ConfigureOptions(); err != nil {
    options.Logger.Fatal().Msgf("Could not initialize options: %s\n", err)
}
_ = readConfig()
```

두 함수의 역할은 다르다.

- `ConfigureOptions()`: goflags가 어떤 문자열을 목록 파일로 볼지 판별하는 공통 규칙을 설정한다.
- `readConfig()`: CLI 옵션 그룹을 등록하고 `flagSet.Parse()`로 실제 명령줄을 읽은 뒤 설정을 병합·보정한다.

## 조기 종료 명령
<!-- main-line-section-tags: #L74 #L80 #L170 -->
#L74 #L80 #L170

`main()`에는 전체 스캔을 시작하지 않고 필요한 정보만 출력한 뒤 끝나는 경로가 있다.

예:

- `-version`: 버전만 출력
- `-list-dsl-function` 또는 `-ldf`: DSL 함수 서명 목록만 출력
- 템플릿 서명 관련 옵션: 지정된 템플릿을 서명하고 종료

이런 명령은 대상에 요청을 보내는 스캔 명령이 아니다. 필요한 정보를 출력한 뒤 `return`으로 `main()`이 끝난다.

## Runner 생성과 실행
<!-- main-line-section-tags: #L177 #L237 #L245 -->
#L177 #L237 #L245

```text
options
  ↓ runner.New(options)
Runner 생성 및 구성요소 준비
  ↓
RunEnumeration()
  ↓
템플릿 로드 → 대상 공급 → Engine 실행 → 결과 출력
```

`runner.New(options)`은 단순히 빈 Runner 하나만 만드는 함수가 아니다. Catalog, Parser, InputProvider, Output Writer, Progress, Interactsh, RateLimiter 등 현재 옵션에 필요한 구성요소를 준비한다.

`RunEnumeration()`은 준비된 구성요소를 이용해 템플릿을 선택하고 실제 스캔 흐름을 시작한다.

이 흐름에서 새로 등장한 이름을 풀면 다음과 같다.

| 이름 | 선언 파일 | 이 단계에서 하는 일 | 다음 연결 |
|---|---|---|---|
| Catalog | `pkg/catalog/catalog.go` 인터페이스 | 템플릿 경로를 찾고 파일을 열 수 있는 공통 규칙 제공 | Loader와 Parser가 사용 |
| Parser | `pkg/templates/parser.go`의 구조체 | YAML·JSON 템플릿을 `Template` 객체로 해석 | Loader Store에 Template 전달 |
| InputProvider | `pkg/input/provider/interface.go` 인터페이스 | `-u`, `-l` 등의 입력을 공통 대상 형태로 제공 | Engine의 대상 반복에 전달 |
| Output Writer | `pkg/output/output.go`의 `Writer` 인터페이스 | 탐지 결과 `ResultEvent`를 화면·파일 등에 기록 | 프로토콜 실행 결과를 받음 |
| Progress | 진행률 패키지의 공통 타입 | 대상·요청·매칭 진행 상태 집계 | Logger와 통계 출력에서 사용 |
| Interactsh | OAST Client | 대상이 외부 서버로 보낸 DNS·HTTP 상호작용 확인 | OAST 템플릿 결과 판정 |
| RateLimiter | 요청 제한 객체 | 설정 시간 동안 보낼 요청 수 제한 | 프로토콜 요청 직전에 사용 |
| Engine | `pkg/core/engine.go`의 구조체 | Template 목록과 InputProvider를 실행 순서에 맞게 조합 | ResultEvent를 Writer로 연결 |

따라서 위 흐름을 더 자세히 쓰면 다음과 같다.

```text
main.go의 options 전역변수
  ↓ readConfig()가 CLI 값을 채움
*types.Options
  ↓ main()이 runner.New(options)에 같은 참조를 전달
*Runner
  ├─ New() 안에서 Parser·Catalog·InputProvider·Writer 등을 생성
  └─ 각 객체를 Runner 구조체 필드에 보관
  ↓ main()이 nucleiRunner.RunEnumeration() 호출
ExecutorOptions에 Runner 필드들을 다시 묶음
  ↓
Loader가 Template 목록 생성 + InputProvider가 대상 제공
  ↓
Engine.ExecuteScanWithOpts()
  ↓
프로토콜 실행 → ResultEvent → Writer
```

이후 [[03_CLI와 readConfig]]에서는 위 흐름의 첫 번째 단계인 “`readConfig()`가 Options에 값을 채운다”를 자세히 다룬다. [[04_내부 패키지]]에서는 Runner 안에 들어간 구성요소를 하나씩 다시 설명한다.

## 관련 분석 노트

- `01_Main_Flow/main.md`: `main()` 줄 단위 흐름
- `01_Main_Flow/main_Functions.md`: 보조 함수 목차
- `01_Main_Flow/main_Varialbes.md`: 변수와 구조체
- `01_Main_Flow/main_Functions/`: 함수별 세부 분석
- `01_Main_Flow/main_Functions/readConfig/`: CLI 그룹별 분석


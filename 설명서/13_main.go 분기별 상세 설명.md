---
유형: 설명서
상태: false
상세: main.go L80~L245 분기와 생성·종료 흐름의 초보자용 상세 설명
---

# main.go 분기별 상세 설명 #전지성

## 이 문서의 역할

`01_Main_Flow/main.md`는 줄 번호별 핵심 흐름을 빠르게 보는 문서다. 이 문서는 그 요약에서 새로 등장하는 옵션, 함수, 구조체와 다음 연결을 자세히 설명한다.

L74의 `-list-dsl-function`은 [[14_-list-dsl-function 상세]]에서 별도로 다룬다.

## 먼저 알아야 할 main의 전체 모양

```text
main() 시작
  ↓
Logger와 CLI 설정 준비
  ↓
readConfig()로 Options 완성
  ↓
스캔 없이 끝나는 특수 모드 확인
  ├─ DSL 목록 출력
  ├─ 템플릿 서명
  └─ 기존 결과 Cloud 업로드
  ↓
성능 분석 설정과 Execution ID 준비
  ↓
runner.New(options)
  ↓
중단·멈춤 감시 설정
  ↓
RunEnumeration()
  ↓
Close()와 Resume 파일 정리
```

여기서 **분기**는 조건에 따라 실행 경로가 갈라지는 부분이다. `if options.SignTemplates`가 true이면 서명 경로로 들어가고, false이면 그 블록을 건너뛰어 다음 코드로 내려간다.

## main에서 return이 중요한 이유

`main()` 안의 `return`은 현재 분기를 끝내는 것뿐 아니라 프로그램의 시작 함수 전체를 끝낸다.

```text
특수 기능 실행
  ↓
return
  ↓
아래 runner.New와 RunEnumeration은 실행되지 않음
  ↓
프로그램 종료
```

따라서 `-sign`, `-ldf`, 기존 결과 업로드는 일반 스캔과 동시에 실행하는 부가 기능이 아니라 **별도의 실행 모드**다.

<a id="l80-template-sign"></a>

## L80 — `-sign` 템플릿 서명 #전지성

### 새로 등장하는 이름

| 이름 | 뜻 |
|---|---|
| `Options.SignTemplates` | `pkg/types/types.go`에 선언된 Bool 필드 |
| `-sign` | 위 필드를 true로 만드는 CLI 옵션 |
| 전자서명 | 템플릿 내용과 개인키를 이용해 위·변조 여부를 확인할 값을 만드는 기능 |
| 개인키 | 서명 생성에 사용하는 비밀 키 |
| 검증 | 대응하는 공개키로 서명이 템플릿 내용과 맞는지 확인하는 과정 |
| `TemplateSigner` | 템플릿 서명 작업을 수행하는 객체 |

### 옵션이 필드에 저장되는 위치

`cmd/nuclei/main.go`의 `readConfig()`는 다음 옵션을 등록한다.

```go
flagSet.BoolVar(
    &options.SignTemplates,
    "sign",
    false,
    "signs the templates with the private key defined in NUCLEI_SIGNATURE_PRIVATE_KEY env variable",
)
```

사용자가 `-sign`을 입력하면 `flagSet.Parse()` 이후 `options.SignTemplates`가 true가 된다.

### main에서의 실행 흐름

```text
Options.SignTemplates == true
  ↓
templates.UseOptionsForSigner(options)
서명 코드가 현재 Options를 사용하도록 설정
  ↓
signer.NewTemplateSigner(nil, nil)
환경변수·설정에서 키를 읽거나 필요한 키 준비
  ↓
options.Templates의 경로를 WalkDir로 순회
  ↓
디렉터리와 YAML이 아닌 파일 제외
  ↓
templates.SignTemplate(tsigner, iterItem)
  ↓
성공·실패 횟수 출력
  ↓
return으로 일반 스캔 없이 종료
```

### 왜 서명 뒤에 스캔하지 않는가?

서명은 대상을 검사하는 작업이 아니라 템플릿 파일 자체를 변경·준비하는 관리 작업이다. 서명과 스캔을 한 명령에서 자동으로 이어 붙이면 사용자가 파일 변경만 원했는데 네트워크 요청까지 보낼 수 있다. 코드에서는 `return`으로 두 목적을 분리한다.

### 보안상 의미

서명은 “이 템플릿이 안전하다”를 자동 보장하는 기능이 아니다. 서명 이후 내용이 바뀌지 않았고 신뢰하는 키로 서명되었는지 확인할 근거를 제공한다. 악의적인 내용을 신뢰하지 않는 키로 서명할 수도 있으므로 서명자 신뢰와 템플릿 내용 검토가 모두 필요하다.

<a id="l118-profiling"></a>

## L118 — Memory·CPU·Trace Profiling #전지성

### 새로 등장하는 이름

| 이름 | 뜻 |
|---|---|
| `memProfile` | `main.go`의 문자열 전역변수 |
| `-profile-mem` | `memProfile`에 출력 기본 경로를 넣는 Debug 옵션 |
| Memory Profile | 어떤 객체가 Heap 메모리를 얼마나 차지하는지 보는 자료 |
| CPU Profile | 어느 함수가 CPU 시간을 많이 사용하는지 보는 자료 |
| Trace | Goroutine, 실행 대기, 스케줄링 등 시간 흐름을 보는 기록 |
| `pprof` | Go 프로그램의 CPU·메모리 상태를 기록·분석하는 표준 도구 |

### 일반 스캔과의 연결

`memProfile`이라는 이름과 달리 코드에서는 메모리 파일만 만들지 않는다.

```text
-profile-mem analysis
  ↓
memProfile = "analysis"
  ↓
analysis.mem   Heap Memory Profile
analysis.cpu   CPU Profile
analysis.trace 실행 Trace
```

Profiling을 시작한 뒤 아래 일반 실행 흐름이 계속된다. 프로그램이 끝날 때 `defer` 블록이 Heap Profile을 쓰고 CPU Profile과 Trace를 닫는다.

### 왜 조건부인가?

성능 기록 자체에도 파일 쓰기와 측정 비용이 든다. 일반 사용자는 탐지 결과가 필요하고 내부 성능 분석 파일은 필요하지 않으므로 기본값을 빈 문자열로 두고 개발자·분석자가 요청했을 때만 활성화한다.

### 이전 Cache 설명과 연결

앞의 Parser 설명에서 Template 객체와 Cache가 메모리를 사용한다고 설명했다. Memory Profile은 “Cache가 메모리를 쓸 것 같다”는 추측에서 끝내지 않고, 실제 실행 중 어떤 타입이 Heap을 차지하는지 측정할 때 사용할 수 있다.

<a id="l166-execution-id"></a>

## L166 — Execution ID 생성 #전지성

### 새로 등장하는 이름

| 이름 | 뜻 |
|---|---|
| `Options.ExecutionId` | 이번 실행을 구분하는 문자열 필드 |
| `xid` | 비교적 짧고 고유한 ID 문자열을 만드는 외부 패키지 |
| 실행 범위 자원 | 특정 스캔에 속한 네트워크 Pool, 통계, 상태 등을 뜻함 |

### 연결 흐름

```text
xid.New().String()
  ↓ 새 고유 문자열 생성
Options.ExecutionId
  ↓ 같은 Options가 Runner에 전달
프로토콜·연결 Pool이 실행별 상태를 구분
  ↓ Runner.Close()
protocolinit.Close(r.options.ExecutionId)
해당 실행에 연결된 자원 정리
```

### 왜 필요한가?

CLI를 한 번만 실행할 때는 전역 상태 하나처럼 보여도, SDK나 장기 실행 프로세스에서는 여러 스캔이 같은 프로세스 안에서 실행될 수 있다. Execution ID가 있으면 한 실행의 연결 상태와 다른 실행의 상태가 섞이지 않도록 키로 사용할 수 있다.

<a id="l168-parse-options"></a>

## L168 — `runner.ParseOptions(options)` #전지성

### readConfig와 무엇이 다른가?

```text
readConfig()
└─ 옵션 등록, 명령줄 Parse, Config·Profile 병합

runner.ParseOptions(options)
└─ 완성된 값의 실행 전 검사·보정과 하위 기능 초기화
```

### 실제 처리

`internal/runner/options.go`의 `ParseOptions()`는 다음 작업을 한다.

1. 표준 입력 Pipe가 있는지 확인해 `Options.Stdin` 설정
2. CLI로 주지 않은 일부 입력을 환경변수에서 읽음
3. Debug·Verbose·Silent 등에 맞게 Logger와 출력 설정
4. 배너 표시
5. Variable Dump나 Headless Action 목록 같은 특수 기능 처리
6. `ValidateOptions()`로 잘못된 옵션 조합 검사
7. DNS Resolver와 프로토콜 공통 상태 초기화

### 왜 readConfig 안에서 전부 하지 않는가?

`readConfig()`는 CLI 입력 형식에 가깝고 `ParseOptions()`는 실행 환경 준비에 가깝다. 이 구분 덕분에 다른 진입점에서 이미 만들어진 Options를 사용할 때도 Runner 쪽의 검사·초기화를 재사용할 수 있다.

<a id="l170-cloud-upload"></a>

## L170 — 기존 결과 파일 Cloud 업로드 #전지성

### 새로 등장하는 이름

| 이름 | 뜻 |
|---|---|
| `Options.ScanUploadFile` | 업로드할 기존 결과 파일 경로 |
| PDCP Cloud | ProjectDiscovery가 제공하는 결과 관리 Cloud |
| Credential | Cloud API가 사용자를 확인하기 위한 인증 정보 |
| JSONL | 한 줄마다 하나의 JSON 객체를 저장하는 형식 |
| `ResultEvent` | Nuclei 탐지 결과 한 건을 나타내는 구조체 |

### 실제 연결 흐름

```text
ScanUploadFile 경로 확인
  ↓
pdcpauth.PDCPCredHandler.GetCreds()
Cloud 인증 정보 읽기
  ↓
pdcp.NewUploadWriter(...)
Cloud 전송 Writer 생성
  ↓
기존 결과 파일 열기
  ↓ JSON Decoder가 한 건씩 해석
output.ResultEvent
  ↓ uploadWriter.Write(&r)
Cloud 전송
  ↓
Writer 종료 후 main() return
```

### 왜 일반 스캔을 실행하지 않는가?

이 모드는 과거에 만들어 둔 결과를 다시 업로드하는 기능이다. 새로운 Target과 Template을 실행할 필요가 없으므로 업로드가 끝나면 `return`한다.

<a id="l177-runner-new"></a>

## L177 — `runner.New(options)` #전지성

### Runner 파일과 구조체

`Runner`는 `internal/runner/runner.go`에 선언된 구조체다. 실행 설정과 공통 구성요소의 참조를 필드에 보관한다.

```text
Runner
├─ options: 전체 실행 설정
├─ parser: Template Parser와 Cache
├─ catalog: 템플릿 경로·파일 접근
├─ inputProvider: Target 공급
├─ output: ResultEvent 출력
├─ progress: 진행 상태 집계
├─ interactsh: OAST Client
├─ rateLimiter: 요청 속도 제한
├─ browser: Headless Browser
└─ tmpDir: 임시 파일 디렉터리
```

### 생성 흐름

```text
main.go의 *types.Options
  ↓ runner.New(options)
빈 Runner 골격에 options와 Logger 연결
  ↓
Parser 생성 또는 전달된 Parser 재사용
  ↓
필요할 때 Browser 생성
  ↓
DiskCatalog 생성
  ↓
임시 디렉터리 생성
  ↓
InputProvider·StandardWriter·Progress 생성
  ↓
Interactsh Client·RateLimiter 생성
  ↓
완성된 *Runner 반환
```

### `nucleiRunner`, `err`, `nil`

- `nucleiRunner`: 생성된 Runner를 가리키는 지역변수
- `err`: 생성 과정에서 파일, Browser, Writer 등 초기화가 실패했을 때 반환되는 오류
- `nucleiRunner == nil`: 객체가 없는 상태에서 아래 메서드를 호출하지 않기 위한 방어적 검사

일반적인 정상 경로에서는 `runner.New()` 마지막이 `return runner, nil`이다. 대상이 없다는 뜻으로 Runner를 nil 반환한다고 단정하면 안 된다. 대상 검증과 InputProvider 처리는 별도 단계다.

### 왜 New와 RunEnumeration을 나누는가?

`New()`은 사용할 자원을 준비하고, `RunEnumeration()`은 준비된 자원으로 실제 작업을 한다. 생성 중 오류와 실행 중 오류를 구분하고, 실행 전 중단 감시를 붙이거나 SDK에서 Runner를 구성한 뒤 원하는 시점에 시작하기 쉬워진다.

<a id="l185-hang-monitor"></a>

## L185 — Hang Monitor #전지성

### Hang은 무엇인가?

프로그램이 종료되지는 않았지만 같은 상태에 머물러 작업이 진행되지 않는 현상이다. 네트워크 대기, Lock, Goroutine 정체 등이 원인이 될 수 있다.

### 새로 등장하는 이름

| 이름 | 뜻 |
|---|---|
| Stack | 실행 중인 함수 호출 경로 정보 |
| Goroutine | Go가 동시에 작업을 실행하는 단위 |
| Stack Monitor | Goroutine 수와 Stack 상태를 주기적으로 비교하는 감시 객체 |
| Callback | Hang이 확인되었을 때 추가로 실행하도록 등록한 함수 |
| CancelFunc | 감시 Goroutine을 멈추는 함수 값 |

### 연결 흐름

```text
Options.HangMonitor == true
  ↓
monitor.NewStackMonitor()
  ↓
Start(10초)
10초 간격으로 Goroutine 상태 확인
  ↓
같은 수와 Stack 상태가 여러 번 지속
  ↓
Stack Trace Dump 파일 기록
  ↓ 등록된 Callback 실행
nucleiRunner.Close()
  ↓
crash-resume-file-<dumpID>.dump 저장
```

`defer cancel()`은 main이 정상 종료될 때 감시 작업도 멈추도록 한다.

### 한계

같은 상태가 반복된다고 항상 실제 Hang인 것은 아니므로 탐지 로직은 여러 차례 상태를 비교한다. 이 기능은 일반적인 취약점 탐지 기능이 아니라 Nuclei 자체 실행 문제를 조사하는 Debug 기능이다.

<a id="l204-graceful-shutdown"></a>

## L204 — Ctrl+C와 Graceful Shutdown #전지성

### Graceful Shutdown이란?

프로세스를 즉시 없애는 대신 열린 파일, Writer, Browser, RateLimiter 등을 정리하고 필요하면 진행 상태를 저장한 뒤 종료하는 방식이다.

### 새로 등장하는 이름

| 이름 | 뜻 |
|---|---|
| `os.Interrupt` | 사용자의 Ctrl+C 같은 중단 신호 |
| Signal Channel | 운영체제 신호를 Go 코드가 받을 통로 |
| `ResumeCfg` | 스캔 진행 위치를 보관하는 구조체 |
| Resume 파일 | 중단한 스캔을 다음 실행에서 이어가기 위한 진행 상태 파일 |

### 연결 흐름

```text
types.DefaultResumeFilePath()
기본 Resume 경로 생성
  ↓ 사용자가 -resume을 주면 해당 경로 사용
signal.Notify(c, os.Interrupt)
  ↓
별도 Goroutine이 Channel에서 Ctrl+C 대기
  ↓ 신호 수신
Runner.Close()
  ↓ ShouldSaveResume가 true이면
Runner.SaveResumeConfig(resumeFileName)
  ↓
임시 Secret 파일 삭제
  ↓
os.Exit(1)
```

### 왜 별도 Goroutine을 사용하는가?

main Goroutine은 `RunEnumeration()`에서 스캔을 수행하고 있다. 별도 Goroutine이 신호를 기다리면 스캔 중에도 Ctrl+C를 받아 정리 작업을 시작할 수 있다.

<a id="l237-run-enumeration"></a>

## L237 — `RunEnumeration()`과 `Close()` #전지성

### RunEnumeration의 역할

앞에서 `runner.New()`이 준비한 구성요소를 실제 실행 단계로 연결한다.

```text
Runner의 Options·Writer·Catalog·Parser·RateLimiter
  ↓
protocols.ExecutorOptions로 묶음
  ↓
core.New(options)으로 Engine 생성
  ↓
loader.NewConfig(...)과 loader.New(...)
  ↓
Store가 Template·Workflow 로드
  ↓
InputProvider의 Target 수 확인
  ↓
Engine.ExecuteScanWithOpts(...)
  ↓
프로토콜 요청·Matcher·Extractor
  ↓
ResultEvent → Writer
```

### Validate 분기

`options.Validate`가 true이면 목적은 스캔이 아니라 템플릿 형식 검증이다. 그래서 오류 메시지도 “Could not validate templates”로 구분한다. 일반 실행 오류와 사용자가 문제를 찾는 위치가 다르기 때문이다.

### Close가 정리하는 대상

`Runner.Close()`는 다음 자원을 조건부로 닫는다.

- DAST Server와 통계
- Host Error Cache
- Output Writer
- Issue Reporting Client
- Project File
- InputProvider
- Execution ID에 연결된 프로토콜 상태
- Pprof Server와 RateLimiter
- Progress, Browser
- 임시 디렉터리

Close는 단순히 메모리를 “삭제”하는 함수가 아니다. 파일 Flush, 네트워크 종료, Goroutine 정지, 임시 파일 삭제처럼 객체별 종료 절차를 수행한다.

<a id="l245-resume-cleanup"></a>

## L245 — 정상 완료 후 Resume 파일 삭제 #전지성

### 앞의 Graceful Shutdown과 연결

중단되었을 때는 나중에 이어야 하므로 Resume 파일을 저장한다. 반대로 `RunEnumeration()`이 끝나고 Runner도 정상적으로 닫혔다면 이어서 할 작업이 없다.

```text
스캔 중 Ctrl+C 또는 복구 필요
└─ Resume 파일 저장

스캔 정상 완료
└─ 기존 Resume 파일이 있으면 삭제
```

`fileutil.FileExists(resumeFileName)`는 파일이 실제로 있는지 먼저 확인하고, `os.Remove()`가 해당 파일을 삭제한다. `_ =`는 삭제 오류를 별도 처리하지 않고 무시한다는 뜻이다.

## main 분기의 최종 정리

| 구간 | Runner 생성 | 대상 스캔 | 종료 방식 |
|---|---:|---:|---|
| `-ldf` | 아니오 | 아니오 | DSL 목록 후 return |
| `-sign` | 아니오 | 아니오 | 서명 후 return |
| `-profile-mem` | 이후 생성 | 예 | 스캔과 Profiling 완료 후 종료 |
| 기존 결과 Cloud 업로드 | 아니오 | 아니오 | 업로드 후 return |
| 일반 실행 | 예 | 예 | RunEnumeration → Close |
| Ctrl+C | 이미 생성됨 | 진행 중 중단 | Close → 필요 시 Resume → Exit |

이 표를 기준으로 `main.md`의 각 `if`가 일반 실행 흐름을 계속하는지, 별도 모드로 끝내는지를 구분하면 된다.


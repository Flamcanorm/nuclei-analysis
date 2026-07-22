---
유형: Internal_Pack
상태: false
상세: internal/runner/runner.go
---
# runner/runner.go #internal/runner #전지성

> Nuclei 실행에 필요한 구성요소를 초기화하고 전체 스캔 순서를 관리하는 코드

## Runner struct

```go
type Runner struct {
	options       *types.Options
	catalog       catalog.Catalog
	progress      progress.Progress
	rateLimiter   *ratelimit.Limiter
	inputProvider provider.InputProvider
	parser        parser.Parser
	Logger        *gologger.Logger
}
```

> Runner는 직접 모든 HTTP·DNS·SSL 검사를 수행하는 구조체가 아니라, 각 구성요소를 만들고 연결하여 실행 순서를 지휘하는 구조체다.

| 주요 필드 | 역할 |
|---|---|
| `options` | 사용자가 입력한 실행 설정 공유 |
| `catalog` | 템플릿 파일의 경로를 찾고 파일을 열 수 있게 함 |
| `progress` | 전체 스캔 진행 상황 관리 |
| `rateLimiter` | 초당·분당 요청 수 제한 |
| `inputProvider` | URL, 도메인 등의 검사 대상 공급 |
| `parser` | YAML·JSON 템플릿 해석 |
| `Logger` | INF, WRN, ERR, DBG 등의 실행 메시지 출력 |

사진에는 Runner가 `Engine engine` 필드를 가진 것으로 표시되어 있지만 실제 Runner 구조체에는 Engine 필드가 없다. Engine은 `RunEnumeration()` 실행 도중 지역변수로 생성된다.

## New

```go
func New(options *types.Options) (*Runner, error)
```

> Options를 받아 Runner를 만들고 스캔에 필요한 구성요소를 초기화하는 생성 함수

```text
Options 전달
    ↓
Runner 객체 생성
    ↓
Catalog 생성
    ↓
입력 공급자·출력·진행률·속도 제한기 등 초기화
    ↓
완성된 Runner 반환
```

Catalog는 다음 코드로 생성된다.

```go
runner.catalog = disk.NewCatalog(
	config.DefaultConfig.TemplatesDirectory,
)
```

기본 템플릿 디렉터리를 기준으로 파일을 찾는 `DiskCatalog`를 Runner에 연결한다.

## RunEnumeration

```go
func (r *Runner) RunEnumeration() error
```

> Runner가 준비한 구성요소를 사용해 템플릿 로드부터 스캔 종료까지 진행하는 핵심 함수

```text
ExecutorOptions 생성
    ↓
Engine 생성
    ↓
Loader Config와 Store 생성
    ↓
템플릿 로드·필터링
    ↓
Engine에 템플릿과 검사 대상 전달
    ↓
결과와 실행 시간 출력
```

Engine은 다음 코드로 생성한다.

```go
executorEngine := core.New(r.options)
executorEngine.SetExecuterOptions(executorOpts)
```

Loader Store는 다음 코드로 생성하고 템플릿을 불러온다.

```go
loaderConfig := loader.NewConfig(r.options, r.catalog, executorOpts)
store, err := loader.New(loaderConfig)
err = store.Load()
```

마지막으로 선택된 템플릿과 입력 대상을 Engine에 전달한다.

```go
results := engine.ExecuteScanWithOpts(
	context.Background(),
	finalTemplates,
	r.inputProvider,
	r.options.DisableClustering,
)
```

## 사진과 실제 코드의 차이

| 사진 | 실제 코드 |
|---|---|
| `Run()` | `RunEnumeration()` |
| Runner가 Engine 필드를 계속 보관 | 실행 중 `core.New()`로 Engine 생성 |
| Catalog가 템플릿을 모두 로드 | Catalog는 경로 검색, Loader가 로드·필터링 |

**참조 :** [main](../01_Main_Flow/main.md), [pkg_types](pkg_types.md), [catalog_disk](catalog_disk.md), [catalog_loader](catalog_loader.md), [core_engine](core_engine.md)

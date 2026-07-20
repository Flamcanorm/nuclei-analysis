---
유형: Internal_Pack
상태: false
상세: pkg/core/engine.go
---
# core/engine.go #pkg/core

> Loader가 준비한 템플릿과 InputProvider의 검사 대상을 받아 실제 스캔 실행을 관리하는 코드

## Engine struct

```go
type Engine struct {
	workPool     *WorkPool
	options      *types.Options
	executerOpts *protocols.ExecutorOptions
	Callback     func(*output.ResultEvent)
	Logger       *gologger.Logger
}
```

| 필드 | 역할 |
|---|---|
| `workPool` | 동시에 실행되는 작업 수 관리 |
| `options` | 동시 실행 수, 스캔 전략 등의 설정 |
| `executerOpts` | Catalog, 속도 제한기, Parser 등 실행 공통 구성요소 |
| `Callback` | 결과가 만들어졌을 때 추가 작업 실행 |
| `Logger` | 실행 상태와 오류 출력 |

사진의 `ProtocolExecutor executor`는 실제 Engine 필드와 일치하지 않는다. 실제 Engine은 `executerOpts`에 공통 실행 설정을 보관하고, 각 Template에 준비된 실행기를 통해 요청을 수행한다.

## New

```go
func New(options *types.Options) *Engine
```

> Options를 연결하고 WorkPool을 생성한 Engine 객체를 반환한다.

WorkPool은 다음 Options 값을 사용한다.

```text
BulkSize
TemplateThreads
HeadlessBulkSize
HeadlessTemplateThreads
```

## SetExecuterOptions

```go
func (e *Engine) SetExecuterOptions(
	options *protocols.ExecutorOptions,
)
```

> Runner가 준비한 공통 실행 구성요소를 Engine에 연결한다. 이 설정이 있어야 실제 템플릿 실행이 가능하다.

## ExecuteScanWithOpts

```go
func (e *Engine) ExecuteScanWithOpts(
	ctx context.Context,
	templatesList []*templates.Template,
	target provider.InputProvider,
	noCluster bool,
) *atomic.Bool
```

| 매개변수 | 의미 |
|---|---|
| `ctx` | 실행 취소와 종료 신호를 전달하는 Context |
| `templatesList` | 실행할 Template 객체 목록 |
| `target` | URL·도메인 등의 검사 대상 공급자 |
| `noCluster` | 비슷한 요청을 묶는 Clustering을 끌지 결정 |

실행 순서는 다음과 같다.

```text
템플릿과 대상 받기
    ↓
비슷한 요청 Clustering
    ↓
Self-contained 템플릿 분리
    ↓
ScanStrategy 선택
    ↓
WorkPool에서 동시 실행
    ↓
매칭 결과 반환
```

### Clustering

비슷한 네트워크 요청을 보내는 템플릿을 묶어 중복 요청 수를 줄이는 기능이다.

### ScanStrategy

```text
TemplateSpray
└─ 템플릿 하나를 여러 대상에 실행

HostSpray
└─ 대상 하나에 여러 템플릿을 실행
```

사진의 `Execute(template)`보다 실제 함수의 범위가 더 크다. Engine은 템플릿 하나가 아니라 템플릿 목록과 검사 대상 목록을 받아 전체 실행 순서와 동시성을 관리한다.

**참조 :** [runner_runner](runner_runner.md), [catalog_loader](catalog_loader.md), [types_scanstrategy](types_scanstrategy.md)

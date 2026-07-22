# protocols.go
#박영현 
## var

```go
var (
	MaxTemplateFileSizeForEncoding = unitutils.Mega
)
```



## type

### Excuter interface
```go
type Executer interface {
	// Compile compiles the execution generators preparing any requests possible.
	Compile() error
	// Requests returns the total number of requests the rule will perform
	Requests() int
	// Execute executes the protocol group and returns true or false if results were found.
	Execute(ctx *scan.ScanContext) (bool, error)
	// ExecuteWithResults executes the protocol requests and returns results instead of writing them.
	ExecuteWithResults(ctx *scan.ScanContext) ([]*output.ResultEvent, error)
}
```
> 프로토콜 검사를 위한 `Excuter`의 공통 인터페이스

|          필드           | 역할                                                                |
| :-------------------: | :---------------------------------------------------------------- |
|      `Compile()`      | 템플릿 내에 정의된 `Generator` 및 `Operators` 컴파일                          |
|     `Requests()`      | 템플릿이 실행될 때 타겟으로 전송할 총 `Request` 수 반환                              |
|      `Excute()`       | 지정된 `ScanContext`를 기반으로 프로토콜 검사 실행, 취약점 탐지 여부 반환(true, false)     |
| `ExcuteWithResults()` | 프로토콜 검사를 실행하되, 발견 결과를 객체 목록(`[]*output.ResusltEvent`)을 메모리로 직접 반환 |

#### Compile()
```go
// Compile compiles the execution generators preparing any requests possible.
	Compile() error
```
- 스캔을 시작하기 직전에 실행됨
- 템플릿의 YAML 구조에서 HTTP, DNS, Network 요청 데이터를 읽어와 실제 네트워크 패킷으로 변환할 수 있게 DSL, 정규식 매처 등을 메모리에 준비시킴
- YAML 문법 오류나 정규식 오류를 `error`로 반환


#### Requests()
```go
// Requests returns the total number of requests the rule will perform
	Requests() int
```
- Nuclei가 스캔을 시작하기 전, 타겟이 속도 제한(Rate limiting)을 계산하기 위해 이 메서드를 호출하여 요청 수를 계산함

#### Excute()
```go
// Execute executes the protocol group and returns true or false if results were found.
	Execute(ctx *scan.ScanContext) (bool, error)
```
- 내부적으로 요청을 보내고 매칭 로직을 실행하고, 결과가 발견되면 콜백을 통해 콘솔 화면에 결과를 출력하거나 파일로 작성(`Writer`)

#### ExcuteWithResults()
```go
// ExecuteWithResults executes the protocol requests and returns results instead of writing them.
	ExecuteWithResults(ctx *scan.ScanContext) ([]*output.ResultEvent, error)
```
- 스캔 결과를 파일 대신 `ResultEvent` 슬라이스 형태로 반환됨




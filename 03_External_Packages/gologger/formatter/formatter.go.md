# formatter.go 
#ExternalPackages/gologger 
> [formatter.go code](https://github.com/projectdiscovery/gologger/blob/main/formatter/formatter.go)
> 로그 데이터 출력 전 특정 포맷으로 변환하는 기능을 정의 (인터페이스)
> CLI, JSON 등 다양한 로그 출력 포맷을 유연하게 교체하여 사용할 수 있도록 해줌

## type 
#ExternalPackages/gologger/type 

### Formatter interface 
#박영현
```go
// Formatter type format raw logging data into something useful
type Formatter interface {
	// Format formats the log event data into bytes
	Format(event *LogEvent) ([]byte, error)
}
```
> 로그를 출력할 포맷 인터페이스

- **설명 :** 
	- 어떤 구조체가 `Format(event *LogEvent) ([]byte, error)` 형태의 매서드를 가진 경우 Formatter 인터페이스에 해당 (묵시적)
	- `Format` 메서드는 [LogEvent](../gologger.go.md#LogEvent%20struct) 데이터를 바이트로 변환하는 메서드

### LogEvent struct 
#박영현 
```go
// LogEvent is the representation of a single event to be logged.
type LogEvent struct {
	Message  string
	Level    levels.Level
	Metadata map[string]string
}
```
> 로그 이벤트 정보를 정의한 구조체
- **참조 :** [levels.go](../gologger.go.md#levels.go) (`levels.Level` 타입 정의)

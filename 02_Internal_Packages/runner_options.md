---
유형: Internal_Pack
상태: false
상세: internal/runner/options.go
---
# runner/options.go #internal/runner

## DefaultDumpTrafficOutputFolder #전지성
```go
const (
	// Default directory used to save protocols traffic
	DefaultDumpTrafficOutputFolder = "output"
)
```
> 요청과 응답을 저장할 기본 디렉터리 값
- **타입 :** `string` 상수
- **값 :** `"output"`
- **위치 :** `internal/runner/options.go`
- **설명 :** `-store-resp-dir` 옵션의 기본값으로 사용
- **참조 :** [main_Functions](../01_Main_Flow/main_Functions.md)

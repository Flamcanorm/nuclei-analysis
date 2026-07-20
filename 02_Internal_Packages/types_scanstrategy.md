---
유형: Internal_Pack
상태: false
상세: pkg/types/scanstrategy/scan_strategy.go
---
# types/scanstrategy.go #internal/types

## ScanStrategy #전지성
```go
type ScanStrategy uint8

const (
	Auto ScanStrategy = iota
	HostSpray
	TemplateSpray
)

func (s ScanStrategy) String() string {
	return strategies[s]
}
```
> Nuclei의 Scan 순서를 선택할 때 사용하는 Strategy 타입
- **타입 :** `uint8` 기반 사용자 정의 타입
- **값 :** `Auto`, `HostSpray`, `TemplateSpray`
- **위치 :** `pkg/types/scanstrategy/scan_strategy.go`
- **설명 :** `-scan-strategy`, `-ss` 옵션의 허용값을 구성
- **참조 :** [main_Functions](../01_Main_Flow/main_Functions.md), [pkg_types](pkg_types.md)

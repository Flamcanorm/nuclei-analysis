---
유형: Internal_Pack
상태: false
상세: pkg/protocols/common/uncover/uncover.go
---
# uncover/uncover.go #internal/uncover

## GetUncoverSupportedAgents #전지성
```go
// returns csv string of uncover supported agents
func GetUncoverSupportedAgents() string {
	u, _ := uncover.New(&uncover.Options{})
	return strings.Join(u.AllAgents(), ",")
}
```
> Uncover 기능에서 지원하는 검색 엔진을 CSV 문자열로 반환하는 함수
- **매개변수 :** 없음
- **반환 타입 :** `string`
- **위치 :** `pkg/protocols/common/uncover/uncover.go`
- **설명 :** `-uncover-engine` 옵션 설명에 지원 가능한 Engine 목록을 표시
- **참조 :** [main_Functions](../01_Main_Flow/main_Functions.md)

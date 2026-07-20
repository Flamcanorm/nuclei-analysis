---
유형: Internal_Pack
상태: false
상세: internal/pdcp/writer.go
---
# pdcp/writer.go #internal/pdcp

## TeamIDEnv #전지성
```go
const NoneTeamID = "none"

var TeamIDEnv = env.GetEnvOrDefault("PDCP_TEAM_ID", NoneTeamID)
```
> ProjectDiscovery Cloud Team ID 환경변수 값을 저장하는 변수
- **타입 :** `string`
- **환경변수 :** `PDCP_TEAM_ID`
- **기본값 :** `"none"`
- **위치 :** `internal/pdcp/writer.go`
- **설명 :** `-team-id`, `-tid` 옵션을 환경변수와 연결할 때 사용
- **참조 :** [main_Functions](../01_Main_Flow/main_Functions.md)

---
유형: Internal_Pack
상태: false
---
# types.go 
> [nuclei/pkg/types/types.go](https://github.com/projectdiscovery/nuclei/blob/main/pkg/types/types.go)
> Nuclei 실행 옵션을 위한 코드입니다.

## type Options struct #pkg/types/struct
```go
type Options struct
```
> Nuclei의 실행 설정을 담는 데이터 구조체입니다. 

### Logger #박영현 #pkg/types/struct/Logger

```go
// Logger is the gologger instance for this optionset
	Logger *gologger.Logger
```

- **타입 :** `*gologger.Logger`
- **설명 :** 터미널에 로그를 출력하는 전역 로거 인스턴스
- **참조 :** [gologger](../03_External_Packages/gologger.md)


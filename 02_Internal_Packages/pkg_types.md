---
유형: Internal_Pack
상태: false
---
# types.go #박영현
> [nuclei/pkg/types/types.go](https://github.com/projectdiscovery/nuclei/blob/main/pkg/types/types.go)
> Nuclei 실행 옵션을 위한 코드입니다.

## type Options struct #pkg/types/struct
> Nuclei의 실행 설정을 담는 데이터 구조체입니다. 

### Logger

```go
// Logger is the gologger instance for this optionset
	Logger *gologger.Logger
```

- **타입 :** `*gologger.Logger`
- **설명 :** 터미널에 로그를 출력하는 전역 로거 인스턴스
- **참조 :** [gologger](gologger.md)


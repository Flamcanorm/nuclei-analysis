---
유형: Internal_Pack
상태: false
상세: Nuclei 데이터 타입
---
# types.go 
> [nuclei/pkg/types/types.go](https://github.com/projectdiscovery/nuclei/blob/main/pkg/types/types.go)
> Nuclei 실행 옵션을 위한 코드

## Options struct #pkg/types/struct
```go
// Options contains the configuration options for nuclei scanner.
type Options struct
```
> Nuclei의 실행 설정을 담는 데이터 구조체

### Logger #박영현

```go
// Logger is the gologger instance for this optionset
	Logger *gologger.Logger
```
> 터미널에 로그를 출력하는 `gologger.Logger` 의 주소 값을 가지는 구조체 멤버 변수
- **타입 :** `*gologger.Logger` (포인터)
- **설명 :** 
	- 외부 패키지 `gologger`에 정의된 [gologger.Logger](../03_External_Packages/gologger.md#Logger%20struct) 주소 값을 가지는 멤버 변수
- **참조 :** [gologger](../03_External_Packages/gologger.md)


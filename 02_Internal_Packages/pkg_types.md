---
유형: Internal_Pack
상태: false
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


### goflag.go #pkg/mod/github-com/projectdiscovery/goflags

## readConfig에서 사용하는 Options 필드 #전지성
```go
type Options struct {
	// -tags 옵션으로 받은 태그 목록 저장
	// 예: nuclei -tags cve,rce
	
	Tags goflags.StringSlice
	// 제외할 태그 목록 저장
	
	// 예: nuclei -exclude-tags dos
	ExcludeTags goflags.StringSlice
	
	// 실행할 워크플로우 목록 저장
	Workflows goflags.StringSlice
	
	// 사용할 템플릿 목록 저장
	// 예: nuclei -t cves/
	Templates goflags.StringSlice
	
	// 직접 입력한 스캔 대상 저장
	// 예: nuclei -u https://example.com
	Targets goflags.StringSlice
	
	// 대상 목록 파일 경로 저장
	// 예: nuclei -l targets.txt
	TargetsFilePath string
	
	// 결과를 저장할 파일 경로
	// 예: nuclei -o result.txt
	Output string
	
	// 디버그 모드 여부
	Debug bool
	
	// 요청 타임아웃 시간
	Timeout int
	
	// 재시도 횟수
	Retries int
	
	// 동시에 처리할 대상 개수
	BulkSize int
	
	// 동시에 실행할 템플릿 개수
	TemplateThreads int
}
```
> `readConfig()`가 CLI 입력값을 저장하기 위해 직접 연결하는 구조체 필드
- **타입 :** `types.Options` 구조체의 멤버 변수
- **설명 :** Tags, Templates, Targets, Output, Debug, Timeout 등 Nuclei 실행 설정을 저장
- **참조 :** [main_Functions](../01_Main_Flow/main_Functions.md), [goflags](../03_External_Packages/goflags.md)

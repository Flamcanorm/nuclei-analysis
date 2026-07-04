---
상세:
유형:
상태: false
---
26.07.02 - 전지성

> [!readConfig 함수 전체 구조]
> readConfig()
> │
> ├── 변수 선언
> │
> ├── FlagSet 생성
> │
> ├── 프로그램 설명 설정
> │
> ├── Target 그룹 생성
> │
> ├── Target-Format 그룹 생성
> │
> ├── Templates 그룹 생성
> │
> └── Filtering 그룹 생성

> [!main.go 구조체]
> // FlagSet is a list of flags for an application
> 
> type FlagSet struct {
> 
>     CaseSensitive  bool
> 
>     Marshal        bool
> 
>     description    string
> 
>     customHelpText string
> 
>     flagKeys       InsertionOrderedMap
> 
>     groups         []groupData
> 
>     CommandLine    *flag.FlagSet
> 
>     configFilePath string
> 
>   
> 
>     // OtherOptionsGroupName is the name for all flags not in a group
> 
>     OtherOptionsGroupName string
> 
>     configOnlyKeys        InsertionOrderedMap
> 
> }
> 
>   
> 
> type groupData struct {
> 
>     name        string
> 
>     description string
> 
> }
> 
>   
> 
> type FlagData struct {
> 
>     usage        string
> 
>     short        string
> 
>     long         string
> 
>     group        string // unused unless set later
> 
>     defaultValue interface{}
> 
>     skipMarshal  bool
> 
>     field        flag.Value
> 
> }

> [!callback_var.go]
> 
> // pkg/mod/github.com/projectdiscovery/goflags@v0.1.74/callback_var.go
> 
> package goflags
> 
>   
> 
> import (
> 
>     "fmt"
> 
>     "strconv"                                                           **// go 표준 라이브러리. 문자열과 다른 자료형을 서로 변환하는 라이브러리임. **
> 
> )
> 
>   
> 
> // CallBackFunc
> 
> type CallBackFunc func()                                          **// 함수 타입(Function Type)을 정의**
> 
>   
> 
> // callBackVar
> 
> type callBackVar struct {
> 
>     Value CallBackFunc                                              **// CallBackFunc 타입**
> 
> }
> 
>   
> 
> // Set
> 
> func (c *callBackVar) Set(s string) error {
> 
>     v, err := strconv.ParseBool(s)                                   ** // v는 false or true, err는 nil **
> 
>     if err != nil {  ** // 오류가 발생했다면 **
> 
>         return fmt.Errorf("failed to parse callback flag")       ** //fmt.Errorf는 fmt 패키지 함수. 함수를 종료하고 error를 반환함 **
> 
>     }
> 
>     if v {                                                                  ** // v가 ture라면 **
> 
>         // if flag found execute callback
> 
>         c.Value()                                                         ** // 구조체의 Value를 실행 **
> 
>     }
> 
>     return nil                                                             ** // 오류가 없다 **
> 
> }
> 
>   
> 
> // IsBoolFlag
> 
> func (c *callBackVar) IsBoolFlag() bool {
> 
>     return true                                                        ** // IsBoolFlag 함수는 항상 true **
> 
> }
> 
>   
> 
> // String
> 
> func (c *callBackVar) String() string {
> 
>     return "false"                                                   ** // String함수는 항상 false **
> 
> }
> 
>   
> 
> // CallbackVar adds a Callback flag with a longname (긴 이름(long name)만 가진 Callback 플래그를 추가한다.)
> 
> func (flagSet *FlagSet) CallbackVar(callback CallBackFunc, long string, usage string) *FlagData {  // falg를 호출하는 함수
> 
>     return flagSet.CallbackVarP(callback, long, "", usage)  // CallbackVarP 함수 호출
> 
> }
> 
>   
> 
> // CallbackVarP adds a Callback flag with a shortname and longname
> 
> func (flagSet *FlagSet) CallbackVarP(callback CallBackFunc, long, short string, usage string) *FlagData {  // 콜백옵션을 등록하는 함수
> 
>     if callback == nil { // callback 함수가 존재하지 않다면
> 
>         panic(fmt.Errorf("callback cannot be nil for flag -%v", long)) // fmt.Errorof으로 반환 후, 복구할 수 없는 오류가 발생으로, 프로그램 즉시 종료
> 
>     }
> 
>     flagData := &FlagData{
> 
>         usage:        usage,
> 
>         long:         long,
> 
>         defaultValue: strconv.FormatBool(false),
> 
>         field:        &callBackVar{Value: callback},
> 
>         skipMarshal: true,
> 
>     }
> 
>     if short != "" {
> 
>         flagData.short = short
> 
>         flagSet.CommandLine.Var(flagData.field, short, usage)
> 
>         flagSet.flagKeys.Set(short, flagData)
> 
>     }
> 
>     flagSet.CommandLine.Var(flagData.field, long, usage) // 명시되있지 않지만, flag라는 go언어 표준 패키지 사용함(-u, -json, -debug 같은 것을 flag(옵션)이라 부름).
> 											// var는 옵션을 등록하는 함수
> 											// commendLine은 프로그램을 실행할 때, 터미널에서 입력한 명령어 한줄 전체를 파싱 대상(flagSet)으로 삼겠다는 뜻.
> 
>     flagSet.flagKeys.Set(long, flagData) // [Key : Value]를 사용하는 자료구조. Key만 입력해서 정보를 빨리 찾기 위해서 쓰임.
> 
>     return flagData
> 
> }

> [!전역 변수 (main.go) 44~55 라인]
> 
> var (
> 	cfgFile                string
> 	templateProfile        string
> 	memProfile             string
> 	options                = &types.Options{}   // 타입은 /pkg/types/types.go
> 	inlineSecretsTempFiles []string
> )
> 

> [!Options(/pkg/types/types.go)]
> 
> type Options struct {
> 	// -tags 옵션으로 받은 태그 목록 저장
> 	// 예: nuclei -tags cve,rce
> 	
> 	Tags goflags.StringSlice
> 	// 제외할 태그 목록 저장
> 	
> 	// 예: nuclei -exclude-tags dos
> 	ExcludeTags goflags.StringSlice
> 	
> 	// 실행할 워크플로우 목록 저장
> 	Workflows goflags.StringSlice
> 	
> 	// 사용할 템플릿 목록 저장
> 	// 예: nuclei -t cves/
> 	Templates goflags.StringSlice
> 	
> 	// 직접 입력한 스캔 대상 저장
> 	// 예: nuclei -u https://example.com
> 	Targets goflags.StringSlice
> 	
> 	// 대상 목록 파일 경로 저장
> 	// 예: nuclei -l targets.txt
> 	TargetsFilePath string
> 	
> 	// 결과를 저장할 파일 경로
> 	// 예: nuclei -o result.txt
> 	Output string
> 	
> 	// 디버그 모드 여부
> 	Debug bool
> 	
> 	// 요청 타임아웃 시간
> 	Timeout int
> 	
> 	// 재시도 횟수
> 	Retries int
> 	
> 	// 동시에 처리할 대상 개수
> 	BulkSize int
> 	
> 	// 동시에 실행할 템플릿 개수
> 	TemplateThreads int
> }

> [!패키지 API]
> `goflags.NewFlagSet()`, `.CaseSensitive`, `.SetDescription(...)` → 전부 **`github.com/projectdiscovery/goflags`** 패키지 API
> 실제 대입되는 값/문자열 내용은 main.go에서 작성
> 

```
func readConfig() *goflags.FlagSet { // 251~727줄

```

> [!함수 반환]
> 반환 타입 `goflags.FlagSet`만 → `github.com/projectdiscovery/goflags`



```
	// readConfig 함수 안에서만 사용할 임시 변수들  
	// 실제 options 구조체에 바로 저장하지 않고,  
	// 후처리가 필요한 옵션 값을 잠깐 저장한다.
	
	// when true updates nuclei binary to latest version
	var updateNucleiBinary bool     // -update 옵션이 들어왔는지 저장
	var pdcpauth string             // -auth 옵션 입력값을 저장. -auth 옵션 값 저장. "true"이거나 API Key일 수 있음
	var fuzzFlag bool               // -fuzz 옵션을 위한 변수 저장(fuzz는 예전 버전임. 현재는 -dast를 권장). 

	flagSet := goflags.NewFlagSet() // pkg/mod/github.com/projectdiscovery/goflags@v0.1.74/callback_var.go
									// CLI(Command Line Interface)의 옵션 관리자(FlagSet 객체) 를 생성합니다.
									// -u, -l, -w, -jsonl, -update 같은 옵션들이 등록됨.
	flagSet.CaseSensitive = true    // 대소문자 구분.
	flagSet.SetDescription(`Nuclei is a fast, template based vulnerability scanner focusing on extensive configurability, massive extensibility and ease of use.`)
	// 위 코드는 readConfing 함수가 실행되고, 위에 코드들이 모두 동작 후, 메세지 출력.
	// 메세지 번역 -> ' Nuclear는 광범위한 구성 가능성, 대규모 확장성 및 사용 편의성에 중점을 둔 빠르고 템플릿 기반의 취약점 스캐너입니다.'
	// 따옴표가 아닌 백틱(``)을 사용한 이유는 여러 줄 작성 가능, \n 없이 줄바꿈이 유지되기 때문.
	
	/* TODO Important: The defined default values, especially for slice/array types are NOT DEFAULT VALUES, but rather implicit values to which the user input is appended.
	This can be very confusing and should be addressed
	*/

```

> [!위에 주석 해석]
> /*
> TODO (개발자가 남긴 메모)
> 
> 현재 Slice(배열, 슬라이스) 타입 옵션의 기본값(Default Value)은
> 실제로는 "기본값"이 아니다.
> 
> 사용자가 입력한 값이
> 기존 값에 append(추가)되는 방식으로 동작한다.
> 
> 예를 들어
> 
> 기본값
> Templates = ["default"]
> 
> 사용자 입력
> -t cves // -t는 실제 있는 값이 아님. Templates의 있는 -t는 등록하는 코드라고 가정.
> 
> 현재 동작
> 
> Templates = []string{"cves"}
> ↓
> 
> ["default", "cves"]
> 
> 하지만 대부분 사용자는
> 
> ["cves"]
> 
> 가 될 것이라고 생각한다.
> 
> 즉,
> 
> 현재의 "기본값"은
> 실제로는 초기값(initial value)에 가깝기 때문에
> 사용자가 오해할 수 있다.
> 
> 개발자는
> 
> "이 부분은 나중에 수정해야 한다."
> 
> 라는 뜻으로 TODO를 남겨 놓은 것이다.
> */

> [!Creategroup에서 사용된 goflag 함수들 goflag.go 위치]
> 
> // pkg/mod/github.com/projectdiscovery/goflags@v0.1.74/goflags.go

> [!CreateGroup 함수 동작] 
> // pkg/mod/github.com/projectdiscovery/goflags@v0.1.74/goflags.go
> 
> func (flagSet *FlagSet) CreateGroup(groupName, description string, flags ...*FlagData) { // 매개변수(그룹내부 이름, 그룹의 표시이름). ...(가변인자)로,FlagData를 몇개든 받겠다는 뜻
> 
>     flagSet.SetGroup(groupName, description) // 그룹생성
> 
>     for _, currentFlag := range flags { // 들어온 모든 옵션 하나씩 꺼냄 ex. 목록에 Targets, TargetFile, Resume이 있다면,  currentFlag에 Targets, TargetFile, Resume을 하나씩 저장.
> 
>         currentFlag.Group(groupName)
> 
>     }
> 
> }

```
	flagSet.CreateGroup("input", "Target",
		flagSet.StringSliceVarP(&options.Targets, "target", "u", nil, "target URLs/hosts to scan", goflags.CommaSeparatedStringSliceOptions),
		flagSet.StringVarP(&options.TargetsFilePath, "list", "l", "", "path to file containing a list of target URLs/hosts to scan (one per line)"),
		flagSet.StringVarP(&options.InlineTargetsList, "targets-inline", "", "", "inline multiline target list (for use in template profiles)"),
		flagSet.StringSliceVarP(&options.ExcludeTargets, "exclude-hosts", "eh", nil, "hosts to exclude to scan from the input list (ip, cidr, hostname)",                             goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.StringVar(&options.Resume, "resume", "", "resume scan from and save to specified file (clustering will be disabled)"),
		flagSet.BoolVarP(&options.ScanAllIPs, "scan-all-ips", "sa", false, "scan all the IP's associated with dns record"),
		flagSet.StringSliceVarP(&options.IPVersion, "ip-version", "iv", nil, "IP version to scan of hostname (4,6) - (default 4)", goflags.CommaSeparatedStringSliceOptions),
	)
```

> [!Flagset.creategroup 해석]
> 
> [흐름]
> input이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Target 그룹을 생성하고, Target 입력과 관련된 CLI 옵션들을 하나의 그룹으로 등록.
> 
> flagSet.StringSliceVarP(...)에서 -target 옵션을 등록하고, FlagData 반환.
> flagSet.StringSliceVarP(...)에서 -list 옵션을 등록하고, FlagData 반환.
> 
> 아래, 옵션들도 모두 등록됨.
> -target  
> -list  
> -targets-inline  
> -exclude-hosts  
> -resume  
> -scan-all-ips  
> -ip-version
> 
> CreateGroup 호출 후, 모두 input의 그룹으로 지정.
> 
> [의미]
> ** 이해를 돕고자 예를 들어 설명했습니다. **
> 
> 1. 사용자가 직접 스캔할 URL 또는 Host를 입력하는 옵션. ( ex. nuclei -u https://example.com -> options.Targets -> []string{ "https://example.com", } )
> 2. 스캔 대상이 적혀있는 파일을 입력. ( ex. nuclei -l targets.txt -> options.TargetsFilePath -> "targets.txt" ) 
> 3. 파일 대신 여러 줄 문자열을 입력한다. ( ex. a.com, b.com, c.com -> options.InlineTargetsList )
> 4. 스캔에서 제외할 Host를 입력한다. ( ex. nuclei -eh localhost -> options.ExcludeTargets )
> 5. 중단된 스캔을 이어서 실행한다. ( ex. nuclei -resume resume.cfg -> options.Resume )
> 6. 도메인에 연결된 모든 IP를 스캔한다. ( ex. nuclei -sa -> options.ScanAllIPs = true )
> 7. IPv4 또는 IPv6를 선택한다. ( ex. nuclei -iv 4 -> options.IPVersion -> []string{"4"} )
> 
> [CLI 옵션 등록]
> `-u, -l, -sa, -iv`
> 
> [입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
> `-u` → `options.Targets`
> `-l` → `options.TargetsFilePath`
> `-sa` → `options.ScanAllIPs`
> `-iv` → `options.IPVersion` 

```
	flagSet.CreateGroup("target-format", "Target-Format",
		flagSet.StringVarP(&options.InputFileMode, "input-mode", "im", "list", fmt.Sprintf("mode of input file (%v)", provider.SupportedInputFormats())),
		flagSet.BoolVarP(&options.FormatUseRequiredOnly, "required-only", "ro", false, "use only required fields in input format when generating requests"),
		flagSet.BoolVarP(&options.SkipFormatValidation, "skip-format-validation", "sfv", false, "skip format validation (like missing vars) when parsing input file"),
		flagSet.BoolVarP(&options.VarsTextTemplating, "vars-text-templating", "vtt", false, "enable text templating for vars in input file (only for yaml input mode)"),
		flagSet.StringSliceVarP(&options.VarsFilePaths, "var-file-paths", "vfp", nil, "list of yaml file contained vars to inject into yaml input",                                   goflags.CommaSeparatedStringSliceOptions),
	)
```

> [!Flagset.creategroup 해석]
> 
> [흐름]
> target-format이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Target-Format 그룹을 생성하고, 입력 파일(Input File)의 형식과 처리 방식에 관련된 CLI 옵션들을 하나의 그룹으로 등록.
> 
> flagSet.StringVarP(...)에서 -input-mode 옵션을 등록하고, FlagData 반환.
> flagSet.BoolVarP(...)에서 -required-only 옵션을 등록하고, FlagData 반환.
> 
> 아래 옵션들도 모두 등록됨.
> -input-mode
> -required-only
> -skip-format-validation
> -vars-text-templating
> -var-file-paths
> 
> CreateGroup 호출 후, 모두 target-format 그룹으로 지정.
> 
> [의미]
> ** 이해를 돕고자 예를 들어 설명했습니다. **
> 
> 1. 입력 파일의 형식을 지정한다. ( ex. nuclei -im list -> options.InputFileMode -> "list" )
> 2. 입력 파일에서 필수(required) 필드만 사용하여 요청을 생성한다. ( ex. nuclei -ro -> options.FormatUseRequiredOnly = true )
> 3. 입력 파일을 파싱할 때 형식 검사를 건너뛴다. ( ex. nuclei -sfv -> options.SkipFormatValidation = true )
> 4. YAML 입력 모드에서 변수(vars)를 텍스트 템플릿으로 처리한다. ( ex. nuclei -vtt -> options.VarsTextTemplating = true )
> 5. YAML 입력 파일에 주입할 변수 파일 목록을 지정한다. ( ex. nuclei -vfp vars1.yaml,vars2.yaml -> options.VarsFilePaths -> []string{"vars1.yaml", "vars2.yaml"} )
> 
> [CLI 옵션 등록]
> -im, -ro, -sfv, -vtt, -vfp
> 
> [입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
> -im  → options.InputFileMode
> -ro  → options.FormatUseRequiredOnly
> -sfv → options.SkipFormatValidation
> -vtt → options.VarsTextTemplating
> -vfp → options.VarsFilePaths

```
	flagSet.CreateGroup("templates", "Templates",
		flagSet.BoolVarP(&options.NewTemplates, "new-templates", "nt", false, "run only new templates added in latest nuclei-templates release"),                 
		flagSet.StringSliceVarP(&options.NewTemplatesWithVersion, "new-templates-version", "ntv", nil, "run new templates added in specific version",                                 goflags.CommaSeparatedStringSliceOptions),
		flagSet.BoolVarP(&options.AutomaticScan, "automatic-scan", "as", false, "automatic web scan using wappalyzer technology detection to tags mapping"),
		flagSet.StringSliceVarP(&options.Templates, "templates", "t", nil, "list of template or template directory to run (comma-separated, file)",                                   goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.StringSliceVarP(&options.TemplateURLs, "template-url", "turl", nil, "template url or list containing template urls to run (comma-separated, file)",                   goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.StringVarP(&options.AITemplatePrompt, "prompt", "ai", "", "generate and run template using ai prompt"),
		flagSet.StringSliceVarP(&options.Workflows, "workflows", "w", nil, "list of workflow or workflow directory to run (comma-separated, file)",                                   goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.StringSliceVarP(&options.WorkflowURLs, "workflow-url", "wurl", nil, "workflow url or list containing workflow urls to run (comma-separated, file)",                   goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.BoolVar(&options.Validate, "validate", false, "validate the passed templates to nuclei"),
		flagSet.BoolVarP(&options.NoStrictSyntax, "no-strict-syntax", "nss", false, "disable strict syntax check on templates"),
		flagSet.BoolVarP(&options.TemplateDisplay, "template-display", "td", false, "displays the templates content"),
		flagSet.BoolVar(&options.TemplateList, "tl", false, "list all templates matching current filters"),
		flagSet.BoolVar(&options.TagList, "tgl", false, "list all available tags"),
		flagSet.StringSliceVarConfigOnly(&options.RemoteTemplateDomainList, "remote-template-domain", []string{"cloud.projectdiscovery.io"}, "allowed domain list to load             remote templates from"),
		flagSet.BoolVar(&options.SignTemplates, "sign", false, "signs the templates with the private key defined in NUCLEI_SIGNATURE_PRIVATE_KEY env variable"),
		flagSet.BoolVar(&options.EnableCodeTemplates, "code", false, "enable loading code protocol-based templates"),
		flagSet.BoolVarP(&options.DisableUnsignedTemplates, "disable-unsigned-templates", "dut", false, "disable running unsigned templates or templates with mismatched              signature"),
		flagSet.BoolVarP(&options.EnableSelfContainedTemplates, "enable-self-contained", "esc", false, "enable loading self-contained templates"),
		flagSet.BoolVarP(&options.EnableGlobalMatchersTemplates, "enable-global-matchers", "egm", false, "enable loading global matchers templates"),
		flagSet.BoolVar(&options.EnableFileTemplates, "file", false, "enable loading file templates"),
	)
```

> [!Flagset.creategroup 해석]
> 
> [흐름]  
> templates라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Templates 그룹을 생성하고, 템플릿(Template) 및 워크플로우(Workflow)와 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
>   
> flagSet.BoolVarP(...)에서 -new-templates 옵션을 등록하고, FlagData 반환.  
> flagSet.StringSliceVarP(...)에서 -new-templates-version 옵션을 등록하고, FlagData 반환.  
>   
> 아래 옵션들도 모두 등록됨.  
> -new-templates  
> -new-templates-version  
> -automatic-scan  
> -templates  
> -template-url  
> -prompt  
> -workflows  
> -workflow-url  
> -validate  
> -no-strict-syntax  
> -template-display  
> -tl  
> -tgl  
> -remote-template-domain  
> -sign  
> -code  
> -disable-unsigned-templates  
> -enable-self-contained  
> -enable-global-matchers  
> -file  
>   
> CreateGroup 호출 후, 모두 templates 그룹으로 지정.  
>   
> [의미]  
> 이해를 돕고자 예를 들어 설명했습니다.  
>   
> 1. 최신 nuclei-templates 릴리스에서 새로 추가된 템플릿만 실행한다.  
> ( ex. nuclei -nt -> options.NewTemplates = true )  
>   
> 2. 특정 버전에서 추가된 템플릿만 실행한다.  
> ( ex. nuclei -ntv 10.2.0 -> options.NewTemplatesWithVersion -> []string{"10.2.0"} )  
>   
> 3. Wappalyzer를 이용하여 웹 기술을 자동으로 분석하고, 해당 기술에 맞는 템플릿을 자동 선택하여 실행한다.  
> ( ex. nuclei -as -> options.AutomaticScan = true )  
>   
> 4. 실행할 템플릿(파일 또는 디렉터리)을 지정한다.  
> ( ex. nuclei -t cves/ -> options.Templates -> []string{"cves/"} )  
>   
> 5. 원격(URL)에 있는 템플릿을 실행한다.  
> ( ex. nuclei -turl https://example.com/template.yaml -> options.TemplateURLs )  
>   
> 6. AI 프롬프트를 이용해 템플릿을 생성하고 바로 실행한다.  
> ( ex. nuclei -ai "Find SQL Injection" -> options.AITemplatePrompt )  
>   
> 7. 실행할 Workflow를 지정한다.  
> ( ex. nuclei -w workflows/http.yaml -> options.Workflows )  
>   
>  8. 원격(URL)에 있는 Workflow를 실행한다.  
> ( ex. nuclei -wurl https://example.com/workflow.yaml -> options.WorkflowURLs )  
>   
> 9. 템플릿 실행 없이 문법만 검사한다.  
> ( ex. nuclei -validate -> options.Validate = true )  
>   
> 10. 엄격한(Syntax) 문법 검사를 비활성화한다.  
> ( ex. nuclei -nss -> options.NoStrictSyntax = true )  
>   
> 11. 템플릿 내용을 화면에 출력한다.  
> ( ex. nuclei -td -> options.TemplateDisplay = true )  
>   
> 12. 현재 필터 조건에 맞는 템플릿 목록만 출력한다.  
> ( ex. nuclei -tl -> options.TemplateList = true )  
>   
> 13. 사용 가능한 태그 목록을 출력한다.  
> ( ex. nuclei -tgl -> options.TagList = true )  
>   
> 14. 원격 템플릿을 다운로드할 수 있는 허용 도메인 목록을 설정한다.  
> ( 기본값 : cloud.projectdiscovery.io )  
>   
> 15. 개인키(NUCLEI_SIGNATURE_PRIVATE_KEY)를 이용하여 템플릿에 전자서명(Sign)을 생성한다.  
> ( ex. nuclei -sign -> options.SignTemplates = true )  
>   
> 16. Code Protocol 기반 템플릿 실행을 허용한다.  
> ( ex. nuclei -code -> options.EnableCodeTemplates = true )  
>   
> 17. 전자서명이 없거나 서명이 일치하지 않는 템플릿의 실행을 차단한다.  
> ( ex. nuclei -dut -> options.DisableUnsignedTemplates = true )  
>   
> 18. Self-Contained 템플릿의 실행을 허용한다.  
> ( ex. nuclei -esc -> options.EnableSelfContainedTemplates = true )  
>   
> 19. Global Matcher 템플릿의 실행을 허용한다.  
> ( ex. nuclei -egm -> options.EnableGlobalMatchersTemplates = true )  
>   
> 20. File Protocol 기반 템플릿의 실행을 허용한다.  
> ( ex. nuclei -file -> options.EnableFileTemplates = true )  
>   
> [CLI 옵션 등록]  
> -nt, -ntv, -as, -t, -turl, -ai, -w, -wurl, -validate,  
> -nss, -td, -tl, -tgl, -sign, -code, -dut, -esc, -egm, -file  
>   
> [입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
> -nt → options.NewTemplates  
> -ntv → options.NewTemplatesWithVersion  
> -as → options.AutomaticScan  
> -t → options.Templates  
> -turl → options.TemplateURLs  
> -ai → options.AITemplatePrompt  
> -w → options.Workflows  
> -wurl → options.WorkflowURLs  
> -validate→ options.Validate  
> -nss → options.NoStrictSyntax  
> -td → options.TemplateDisplay  
> -tl → options.TemplateList  
> -tgl → options.TagList  
> -sign → options.SignTemplates  
> -code → options.EnableCodeTemplates  
> -dut → options.DisableUnsignedTemplates  
> -esc → options.EnableSelfContainedTemplates  
> -egm → options.EnableGlobalMatchersTemplates  
> -file → options.EnableFileTemplates

```
	flagSet.CreateGroup("filters", "Filtering",
		flagSet.StringSliceVarP(&options.Authors, "author", "a", nil, "templates to run based on authors (comma-separated, file)", goflags.FileNormalizedStringSliceOptions),
		flagSet.StringSliceVar(&options.Tags, "tags", nil, "templates to run based on tags (comma-separated, file)", goflags.FileNormalizedStringSliceOptions), 
		flagSet.StringSliceVarP(&options.ExcludeTags, "exclude-tags", "etags", nil, "templates to exclude based on tags (comma-separated, file)",                                     goflags.FileNormalizedStringSliceOptions),
		flagSet.StringSliceVarP(&options.IncludeTags, "include-tags", "itags", nil, "tags to be executed even if they are excluded either by default or configuration",               goflags.FileNormalizedStringSliceOptions), // TODO show default deny list
		flagSet.StringSliceVarP(&options.IncludeIds, "template-id", "id", nil, "templates to run based on template ids (comma-separated, file, allow-wildcard)",                      goflags.FileNormalizedStringSliceOptions),
		flagSet.StringSliceVarP(&options.ExcludeIds, "exclude-id", "eid", nil, "templates to exclude based on template ids (comma-separated, file)",                                  goflags.FileNormalizedStringSliceOptions),
		flagSet.StringSliceVarP(&options.IncludeTemplates, "include-templates", "it", nil, "path to template file or directory to be executed even if they are excluded               either by default or configuration", goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.StringSliceVarP(&options.ExcludedTemplates, "exclude-templates", "et", nil, "path to template file or directory to exclude (comma-separated, file)",                  goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.StringSliceVarP(&options.ExcludeMatchers, "exclude-matchers", "em", nil, "template matchers to exclude in result",                                                    goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.VarP(&options.Severities, "severity", "s", fmt.Sprintf("templates to run based on severity. Possible values: %s",                                                     severity.GetSupportedSeverities().String())),
		flagSet.VarP(&options.ExcludeSeverities, "exclude-severity", "es", fmt.Sprintf("templates to exclude based on severity. Possible values: %s",                                 severity.GetSupportedSeverities().String())),
		flagSet.VarP(&options.Protocols, "type", "pt", fmt.Sprintf("templates to run based on protocol type. Possible values: %s",                                                    templateTypes.GetSupportedProtocolTypes())),  
		flagSet.VarP(&options.ExcludeProtocols, "exclude-type", "ept", fmt.Sprintf("templates to exclude based on protocol type. Possible values: %s",                                templateTypes.GetSupportedProtocolTypes())),
		flagSet.StringSliceVarP(&options.IncludeConditions, "template-condition", "tc", nil, "templates to run based on expression condition", goflags.StringSliceOptions),
	)
```

> [!Flagset.creategroup 해석]
> 
> [흐름]
> filters라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Filtering 그룹을 생성하고, 실행할 템플릿을 다양한 조건으로 선택하거나 제외하는 CLI 옵션들을 하나의 그룹으로 등록.
> 
> flagSet.StringSliceVarP(...)에서 -author 옵션을 등록하고, FlagData 반환.
> flagSet.StringSliceVar(...)에서 -tags 옵션을 등록하고, FlagData 반환.
> 
> 아래 옵션들도 모두 등록됨.
> -author
> -tags
> -exclude-tags
> -include-tags
> -template-id
> -exclude-id
> -include-templates
> -exclude-templates
> -exclude-matchers
> -severity
> -exclude-severity
> -type
> -exclude-type
> -template-condition
> 
> CreateGroup 호출 후, 모두 filters 그룹으로 지정.
> 
> [의미]
> 이해를 돕고자 예를 들어 설명했습니다.
> 
> 1. 특정 작성자(Author)가 만든 템플릿만 실행한다.
> ( ex. nuclei -a pdteam -> options.Authors )
> 
> 2. 특정 태그(Tag)를 가진 템플릿만 실행한다.
> ( ex. nuclei -tags cve,rce -> options.Tags )
> 
> 3. 특정 태그(Tag)를 가진 템플릿은 제외한다.
> ( ex. nuclei -etags dos -> options.ExcludeTags )
> 
> 4. 기본적으로 제외된 태그라도 강제로 포함하여 실행한다.
> ( ex. nuclei -itags fuzz -> options.IncludeTags )
> 
> 5. 특정 Template ID를 가진 템플릿만 실행한다.
> ( ex. nuclei -id cve-2024-* -> options.IncludeIds )
> 
> 6. 특정 Template ID를 가진 템플릿을 제외한다.
> ( ex. nuclei -eid cve-2023-* -> options.ExcludeIds )
> 
> 7. 특정 템플릿(파일 또는 디렉터리)을 강제로 포함하여 실행한다.
> ( ex. nuclei -it templates/http/ -> options.IncludeTemplates )
> 
> 8. 특정 템플릿(파일 또는 디렉터리)을 실행 대상에서 제외한다.
> ( ex. nuclei -et templates/dos/ -> options.ExcludedTemplates )
> 
> 9. 특정 Matcher를 결과에서 제외한다.
> ( ex. nuclei -em word-matcher -> options.ExcludeMatchers )
> 
> 10. 심각도(Severity)에 따라 템플릿을 실행한다.
> ( ex. nuclei -s critical,high -> options.Severities )
> 
> 11. 특정 심각도의 템플릿은 제외한다.
> ( ex. nuclei -es info -> options.ExcludeSeverities )
> 
> 12. 특정 프로토콜(HTTP, DNS 등)의 템플릿만 실행한다.
> ( ex. nuclei -pt http -> options.Protocols )
> 
> 13. 특정 프로토콜의 템플릿은 제외한다.
> ( ex. nuclei -ept dns -> options.ExcludeProtocols )
> 
> 14. 조건식(Expression)에 맞는 템플릿만 실행한다.
> ( ex. nuclei -tc "severity == critical" -> options.IncludeConditions )
> 
> [CLI 옵션 등록]
> -a, -tags, -etags, -itags, -id, -eid, -it, -et,
> -em, -s, -es, -pt, -ept, -tc
> 
> [입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
> -a      → options.Authors
> -tags   → options.Tags
> -etags  → options.ExcludeTags
> -itags  → options.IncludeTags
> -id     → options.IncludeIds
> -eid    → options.ExcludeIds
> -it     → options.IncludeTemplates
> -et     → options.ExcludedTemplates
> -em     → options.ExcludeMatchers
> -s      → options.Severities
> -es     → options.ExcludeSeverities
> -pt     → options.Protocols
> -ept    → options.ExcludeProtocols
> -tc     → options.IncludeConditions
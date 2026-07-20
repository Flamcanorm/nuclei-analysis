---
상태: false
유형:
상세: main.go 함수들
---
# 함수 #main/function

## readConfig #전지성
```go
func readConfig() *goflags.FlagSet
```
> Nuclei에서 사용할 CLI 옵션을 등록하고 입력값·설정 파일·Template Profile을 병합하는 함수
- **매개변수 :** 없음
- **반환 타입 :** `*goflags.FlagSet` (포인터)
- **설명 :**
	- Target, Templates, Filtering, Output 등 CLI 옵션 그룹을 등록
	- `flagSet.Parse()`로 사용자 입력을 `options` 구조체와 연결
	- Config 파일과 Template Profile을 병합
	- 인증, Update, Inline Target, Secret 및 Resume 파일을 후처리
- **참조 :** [main_Varialbes](main_Varialbes.md), [pkg/types](../02_Internal_Packages/pkg_types.md), [goflags](../03_External_Packages/goflags.md), [sample](../04_etc/sample.md)

### 상세 분석 #전지성

#### 26.07.02 작업 기록

##### readConfig 함수 전체 구조
readConfig()
│
├── 변수 선언
│
├── FlagSet 생성
│
├── 프로그램 설명 설정
│
├── Target 그룹 생성
│
├── Target-Format 그룹 생성
│
├── Templates 그룹 생성
│
└── Filtering 그룹 생성

##### 외부 패키지로 분리
`FlagSet`, `FlagData`, Callback 구현은 [[../03_External_Packages/goflags|goflags]] 참고.

##### 변수 설명 분리
`main.go` 전역변수는 [[main_Varialbes]] 참고.

##### 내부 패키지로 분리
`types.Options` 구조체는 [[../02_Internal_Packages/pkg_types|pkg_types]] 참고.

##### 외부 패키지 API로 분리
`goflags.NewFlagSet()` 등의 API는 [[../03_External_Packages/goflags|goflags]] 참고.

```
func readConfig() *goflags.FlagSet { // 251~727줄

```

##### 함수 반환
반환 타입 `goflags.FlagSet`만 → `github.com/projectdiscovery/goflags`

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

##### Slice 기본값 주석 해석
자세한 설명은 [[../04_etc/sample#Slice 옵션의 기본값|sample]] 참고.

##### CreateGroup 구현 분리
외부 패키지의 `CreateGroup()` 구현은 [[../03_External_Packages/goflags#CreateGroup 함수|goflags]] 참고.

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

##### FlagSet.CreateGroup 해석

[흐름]
input이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Target 그룹을 생성하고, Target 입력과 관련된 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.StringSliceVarP(...)에서 -target 옵션을 등록하고, FlagData 반환.
flagSet.StringSliceVarP(...)에서 -list 옵션을 등록하고, FlagData 반환.

아래, 옵션들도 모두 등록됨.
-target  
-list  
-targets-inline  
-exclude-hosts  
-resume  
-scan-all-ips  
-ip-version

CreateGroup 호출 후, 모두 input의 그룹으로 지정.

[의미]
** 이해를 돕고자 예를 들어 설명했습니다. **

1. 사용자가 직접 스캔할 URL 또는 Host를 입력하는 옵션. ( ex. nuclei -u https://example.com -> options.Targets -> []string{ "https://example.com", } )
2. 스캔 대상이 적혀있는 파일을 입력. ( ex. nuclei -l targets.txt -> options.TargetsFilePath -> "targets.txt" ) 
3. 파일 대신 여러 줄 문자열을 입력한다. ( ex. a.com, b.com, c.com -> options.InlineTargetsList )
4. 스캔에서 제외할 Host를 입력한다. ( ex. nuclei -eh localhost -> options.ExcludeTargets )
5. 중단된 스캔을 이어서 실행한다. ( ex. nuclei -resume resume.cfg -> options.Resume )
6. 도메인에 연결된 모든 IP를 스캔한다. ( ex. nuclei -sa -> options.ScanAllIPs = true )
7. IPv4 또는 IPv6를 선택한다. ( ex. nuclei -iv 4 -> options.IPVersion -> []string{"4"} )

[CLI 옵션 등록]
`-u, -l, -sa, -iv`

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
`-u` → `options.Targets`
`-l` → `options.TargetsFilePath`
`-sa` → `options.ScanAllIPs`
`-iv` → `options.IPVersion` 

```
	flagSet.CreateGroup("target-format", "Target-Format",
		flagSet.StringVarP(&options.InputFileMode, "input-mode", "im", "list", fmt.Sprintf("mode of input file (%v)", provider.SupportedInputFormats())),
		flagSet.BoolVarP(&options.FormatUseRequiredOnly, "required-only", "ro", false, "use only required fields in input format when generating requests"),
		flagSet.BoolVarP(&options.SkipFormatValidation, "skip-format-validation", "sfv", false, "skip format validation (like missing vars) when parsing input file"),
		flagSet.BoolVarP(&options.VarsTextTemplating, "vars-text-templating", "vtt", false, "enable text templating for vars in input file (only for yaml input mode)"),
		flagSet.StringSliceVarP(&options.VarsFilePaths, "var-file-paths", "vfp", nil, "list of yaml file contained vars to inject into yaml input",                                   goflags.CommaSeparatedStringSliceOptions),
	)
```

##### FlagSet.CreateGroup 해석

[흐름]
target-format이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Target-Format 그룹을 생성하고, 입력 파일(Input File)의 형식과 처리 방식에 관련된 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.StringVarP(...)에서 -input-mode 옵션을 등록하고, FlagData 반환.
flagSet.BoolVarP(...)에서 -required-only 옵션을 등록하고, FlagData 반환.

아래 옵션들도 모두 등록됨.
-input-mode
-required-only
-skip-format-validation
-vars-text-templating
-var-file-paths

CreateGroup 호출 후, 모두 target-format 그룹으로 지정.

[의미]
** 이해를 돕고자 예를 들어 설명했습니다. **

1. 입력 파일의 형식을 지정한다. ( ex. nuclei -im list -> options.InputFileMode -> "list" )
2. 입력 파일에서 필수(required) 필드만 사용하여 요청을 생성한다. ( ex. nuclei -ro -> options.FormatUseRequiredOnly = true )
3. 입력 파일을 파싱할 때 형식 검사를 건너뛴다. ( ex. nuclei -sfv -> options.SkipFormatValidation = true )
4. YAML 입력 모드에서 변수(vars)를 텍스트 템플릿으로 처리한다. ( ex. nuclei -vtt -> options.VarsTextTemplating = true )
5. YAML 입력 파일에 주입할 변수 파일 목록을 지정한다. ( ex. nuclei -vfp vars1.yaml,vars2.yaml -> options.VarsFilePaths -> []string{"vars1.yaml", "vars2.yaml"} )

[CLI 옵션 등록]
-im, -ro, -sfv, -vtt, -vfp

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
-im  → options.InputFileMode
-ro  → options.FormatUseRequiredOnly
-sfv → options.SkipFormatValidation
-vtt → options.VarsTextTemplating
-vfp → options.VarsFilePaths

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

##### FlagSet.CreateGroup 해석

[흐름]  
templates라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Templates 그룹을 생성하고, 템플릿(Template) 및 워크플로우(Workflow)와 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.BoolVarP(...)에서 -new-templates 옵션을 등록하고, FlagData 반환.  
flagSet.StringSliceVarP(...)에서 -new-templates-version 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-new-templates  
-new-templates-version  
-automatic-scan  
-templates  
-template-url  
-prompt  
-workflows  
-workflow-url  
-validate  
-no-strict-syntax  
-template-display  
-tl  
-tgl  
-remote-template-domain  
-sign  
-code  
-disable-unsigned-templates  
-enable-self-contained  
-enable-global-matchers  
-file  
  
CreateGroup 호출 후, 모두 templates 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. 최신 nuclei-templates 릴리스에서 새로 추가된 템플릿만 실행한다.  
( ex. nuclei -nt -> options.NewTemplates = true )  
  
2. 특정 버전에서 추가된 템플릿만 실행한다.  
( ex. nuclei -ntv 10.2.0 -> options.NewTemplatesWithVersion -> []string{"10.2.0"} )  
  
3. Wappalyzer를 이용하여 웹 기술을 자동으로 분석하고, 해당 기술에 맞는 템플릿을 자동 선택하여 실행한다.  
( ex. nuclei -as -> options.AutomaticScan = true )  
  
4. 실행할 템플릿(파일 또는 디렉터리)을 지정한다.  
( ex. nuclei -t cves/ -> options.Templates -> []string{"cves/"} )  
  
5. 원격(URL)에 있는 템플릿을 실행한다.  
( ex. nuclei -turl https://example.com/template.yaml -> options.TemplateURLs )  
  
6. AI 프롬프트를 이용해 템플릿을 생성하고 바로 실행한다.  
( ex. nuclei -ai "Find SQL Injection" -> options.AITemplatePrompt )  
  
7. 실행할 Workflow를 지정한다.  
( ex. nuclei -w workflows/http.yaml -> options.Workflows )  
  
 8. 원격(URL)에 있는 Workflow를 실행한다.  
( ex. nuclei -wurl https://example.com/workflow.yaml -> options.WorkflowURLs )  
  
9. 템플릿 실행 없이 문법만 검사한다.  
( ex. nuclei -validate -> options.Validate = true )  
  
10. 엄격한(Syntax) 문법 검사를 비활성화한다.  
( ex. nuclei -nss -> options.NoStrictSyntax = true )  
  
11. 템플릿 내용을 화면에 출력한다.  
( ex. nuclei -td -> options.TemplateDisplay = true )  
  
12. 현재 필터 조건에 맞는 템플릿 목록만 출력한다.  
( ex. nuclei -tl -> options.TemplateList = true )  
  
13. 사용 가능한 태그 목록을 출력한다.  
( ex. nuclei -tgl -> options.TagList = true )  
  
14. 원격 템플릿을 다운로드할 수 있는 허용 도메인 목록을 설정한다.  
( 기본값 : cloud.projectdiscovery.io )  
  
15. 개인키(NUCLEI_SIGNATURE_PRIVATE_KEY)를 이용하여 템플릿에 전자서명(Sign)을 생성한다.  
( ex. nuclei -sign -> options.SignTemplates = true )  
  
16. Code Protocol 기반 템플릿 실행을 허용한다.  
( ex. nuclei -code -> options.EnableCodeTemplates = true )  
  
17. 전자서명이 없거나 서명이 일치하지 않는 템플릿의 실행을 차단한다.  
( ex. nuclei -dut -> options.DisableUnsignedTemplates = true )  
  
18. Self-Contained 템플릿의 실행을 허용한다.  
( ex. nuclei -esc -> options.EnableSelfContainedTemplates = true )  
  
19. Global Matcher 템플릿의 실행을 허용한다.  
( ex. nuclei -egm -> options.EnableGlobalMatchersTemplates = true )  
  
20. File Protocol 기반 템플릿의 실행을 허용한다.  
( ex. nuclei -file -> options.EnableFileTemplates = true )  
  
[CLI 옵션 등록]  
-nt, -ntv, -as, -t, -turl, -ai, -w, -wurl, -validate,  
-nss, -td, -tl, -tgl, -sign, -code, -dut, -esc, -egm, -file  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-nt → options.NewTemplates  
-ntv → options.NewTemplatesWithVersion  
-as → options.AutomaticScan  
-t → options.Templates  
-turl → options.TemplateURLs  
-ai → options.AITemplatePrompt  
-w → options.Workflows  
-wurl → options.WorkflowURLs  
-validate→ options.Validate  
-nss → options.NoStrictSyntax  
-td → options.TemplateDisplay  
-tl → options.TemplateList  
-tgl → options.TagList  
-sign → options.SignTemplates  
-code → options.EnableCodeTemplates  
-dut → options.DisableUnsignedTemplates  
-esc → options.EnableSelfContainedTemplates  
-egm → options.EnableGlobalMatchersTemplates  
-file → options.EnableFileTemplates

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

##### FlagSet.CreateGroup 해석

[흐름]
filters라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Filtering 그룹을 생성하고, 실행할 템플릿을 다양한 조건으로 선택하거나 제외하는 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.StringSliceVarP(...)에서 -author 옵션을 등록하고, FlagData 반환.
flagSet.StringSliceVar(...)에서 -tags 옵션을 등록하고, FlagData 반환.

아래 옵션들도 모두 등록됨.
-author
-tags
-exclude-tags
-include-tags
-template-id
-exclude-id
-include-templates
-exclude-templates
-exclude-matchers
-severity
-exclude-severity
-type
-exclude-type
-template-condition

CreateGroup 호출 후, 모두 filters 그룹으로 지정.

[의미]
이해를 돕고자 예를 들어 설명했습니다.

1. 특정 작성자(Author)가 만든 템플릿만 실행한다.
( ex. nuclei -a pdteam -> options.Authors )

2. 특정 태그(Tag)를 가진 템플릿만 실행한다.
( ex. nuclei -tags cve,rce -> options.Tags )

3. 특정 태그(Tag)를 가진 템플릿은 제외한다.
( ex. nuclei -etags dos -> options.ExcludeTags )

4. 기본적으로 제외된 태그라도 강제로 포함하여 실행한다.
( ex. nuclei -itags fuzz -> options.IncludeTags )

5. 특정 Template ID를 가진 템플릿만 실행한다.
( ex. nuclei -id cve-2024-* -> options.IncludeIds )

6. 특정 Template ID를 가진 템플릿을 제외한다.
( ex. nuclei -eid cve-2023-* -> options.ExcludeIds )

7. 특정 템플릿(파일 또는 디렉터리)을 강제로 포함하여 실행한다.
( ex. nuclei -it templates/http/ -> options.IncludeTemplates )

8. 특정 템플릿(파일 또는 디렉터리)을 실행 대상에서 제외한다.
( ex. nuclei -et templates/dos/ -> options.ExcludedTemplates )

9. 특정 Matcher를 결과에서 제외한다.
( ex. nuclei -em word-matcher -> options.ExcludeMatchers )

10. 심각도(Severity)에 따라 템플릿을 실행한다.
( ex. nuclei -s critical,high -> options.Severities )

11. 특정 심각도의 템플릿은 제외한다.
( ex. nuclei -es info -> options.ExcludeSeverities )

12. 특정 프로토콜(HTTP, DNS 등)의 템플릿만 실행한다.
( ex. nuclei -pt http -> options.Protocols )

13. 특정 프로토콜의 템플릿은 제외한다.
( ex. nuclei -ept dns -> options.ExcludeProtocols )

14. 조건식(Expression)에 맞는 템플릿만 실행한다.
( ex. nuclei -tc "severity == critical" -> options.IncludeConditions )

[CLI 옵션 등록]
-a, -tags, -etags, -itags, -id, -eid, -it, -et,
-em, -s, -es, -pt, -ept, -tc

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
-a      → options.Authors
-tags   → options.Tags
-etags  → options.ExcludeTags
-itags  → options.IncludeTags
-id     → options.IncludeIds
-eid    → options.ExcludeIds
-it     → options.IncludeTemplates
-et     → options.ExcludedTemplates
-em     → options.ExcludeMatchers
-s      → options.Severities
-es     → options.ExcludeSeverities
-pt     → options.Protocols
-ept    → options.ExcludeProtocols
-tc     → options.IncludeConditions

### 26.07.05 - 전지성


#### 표준 라이브러리 설명 분리
[[../04_etc/sample#사용된 표준 라이브러리|sample]] 참고.

#### main.go 안에 있는 함수 사용
printVersion()
printTemplateVersion()

#### 328줄, 사용된 외부파일
(새로나온 외부파일만 작성됨)
[runner_options](../02_Internal_Packages/runner_options.md) — `internal/runner/options.go`

#### 384 줄, 사용된 외부파일
 (새로나온 외부파일만 작성됨)
[interactsh_client](../03_External_Packages/interactsh_client.md) — `interactsh/pkg/client/client.go`

#### 412줄, 사용된 외부파일
 (새로나온 외부파일만 작성됨)
[uncover_uncover](../02_Internal_Packages/uncover_uncover.md) — `pkg/protocols/common/uncover/uncover.go`

#### 443줄, 사용된 외부파일
 (새로나온 외부파일만 작성됨)
[pkg_types](../02_Internal_Packages/pkg_types.md) — `pkg/types/types.go`
[types_scanstrategy](../02_Internal_Packages/types_scanstrategy.md) — `scanstrategy` 패키지

#### 508줄, 사용된 외부파일
 (새로나온 외부파일만 작성됨)
[pdcp_writer](../02_Internal_Packages/pdcp_writer.md) — `internal/pdcp/writer.go`
PDCP(ProjectDiscovery Cloud) 패키지

```
flagSet.CreateGroup("output", "Output",
		flagSet.StringVarP(&options.Output, "output", "o", "", "output file to write found issues/vulnerabilities"),
		flagSet.BoolVarP(&options.StoreResponse, "store-resp", "sresp", false, "store all request/response passed through nuclei to output directory"),
		flagSet.StringVarP(&options.StoreResponseDir, "store-resp-dir", "srd", runner.DefaultDumpTrafficOutputFolder, "store all request/response passed through nuclei to custom directory"),
		flagSet.BoolVar(&options.Silent, "silent", false, "display findings only"),
		flagSet.BoolVarP(&options.NoColor, "no-color", "nc", false, "disable output content coloring (ANSI escape codes)"),
		flagSet.BoolVarP(&options.JSONL, "jsonl", "j", false, "write output in JSONL(ines) format"),
		flagSet.BoolVarP(&options.JSONRequests, "include-rr", "irr", true, "include request/response pairs in the JSON, JSONL, and Markdown outputs (for findings only) [DEPRECATED use `-omit-raw`]"),
		flagSet.BoolVarP(&options.OmitRawRequests, "omit-raw", "or", false, "omit request/response pairs in the JSON, JSONL, Markdown, and PDF outputs (for findings only)"),
		flagSet.BoolVarP(&options.OmitTemplate, "omit-template", "ot", false, "omit encoded template in the JSON, JSONL output"),
		flagSet.BoolVarP(&options.NoMeta, "no-meta", "nm", false, "disable printing result metadata in cli output"),
		flagSet.BoolVarP(&options.Timestamp, "timestamp", "ts", false, "enables printing timestamp in cli output"),
		flagSet.StringVarP(&options.ReportingDB, "report-db", "rdb", "", "nuclei reporting database (always use this to persist report data)"),
		flagSet.BoolVarP(&options.MatcherStatus, "matcher-status", "ms", false, "display match failure status"),
		flagSet.StringVarP(&options.MarkdownExportDirectory, "markdown-export", "me", "", "directory to export results in markdown format"),
		flagSet.StringVarP(&options.SarifExport, "sarif-export", "se", "", "file to export results in SARIF format"),
		flagSet.StringVarP(&options.JSONExport, "json-export", "je", "", "file to export results in JSON format"),
		flagSet.StringVarP(&options.JSONLExport, "jsonl-export", "jle", "", "file to export results in JSONL(ine) format"),
		flagSet.StringVarP(&options.PDFExport, "pdf-export", "pe", "", "file to export results in PDF format"),
		flagSet.StringSliceVarP(&options.Redact, "redact", "rd", nil, "redact given list of keys from query parameter, request header and body", goflags.CommaSeparatedStringSliceOptions),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
output이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Output 그룹을 생성하고, 스캔 결과를 어떤 형식으로 출력하거나 저장할지 정하는 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.StringVarP(...)에서 -output 옵션을 등록하고, FlagData 반환.  
flagSet.BoolVarP(...)에서 -store-resp 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-output  
-store-resp  
-store-resp-dir  
-silent  
-no-color  
-jsonl  
-include-rr  
-omit-raw  
-omit-template  
-no-meta  
-timestamp  
-report-db  
-matcher-status  
-markdown-export  
-sarif-export  
-json-export  
-jsonl-export  
-pdf-export  
-redact  
  
CreateGroup 호출 후, 모두 output 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. 스캔 결과를 저장할 출력 파일을 지정한다.  
( ex. nuclei -o result.txt -> options.Output )  
  
2. 요청/응답 데이터를 출력 디렉터리에 저장한다.  
( ex. nuclei -sresp -> options.StoreResponse = true )  
  
3. 요청/응답 데이터를 저장할 디렉터리를 지정한다.  
( ex. nuclei -srd responses/ -> options.StoreResponseDir )  
  
4. 탐지 결과만 조용히 출력한다.  
( ex. nuclei -silent -> options.Silent = true )  
  
5. 터미널 출력 색상(ANSI 색상 코드)을 비활성화한다.  
( ex. nuclei -nc -> options.NoColor = true )  
  
6. 출력을 JSONL 형식으로 작성한다.  
( ex. nuclei -j -> options.JSONL = true )  
  
7. JSON, JSONL, Markdown 출력에 요청/응답 쌍을 포함한다. 단, Deprecated 옵션이다.  
( ex. nuclei -irr -> options.JSONRequests = true )  
  
8. JSON, JSONL, Markdown, PDF 출력에서 원본 요청/응답을 제외한다.  
( ex. nuclei -or -> options.OmitRawRequests = true )  
  
9. JSON, JSONL 출력에서 인코딩된 템플릿 내용을 제외한다.  
( ex. nuclei -ot -> options.OmitTemplate = true )  
  
10. CLI 출력에서 결과 메타데이터 표시를 비활성화한다.  
( ex. nuclei -nm -> options.NoMeta = true )  
  
11. CLI 출력에 타임스탬프를 표시한다.  
( ex. nuclei -ts -> options.Timestamp = true )  
  
12. Nuclei 리포팅 데이터베이스 파일을 지정한다.  
( ex. nuclei -rdb nuclei-report.db -> options.ReportingDB )  
  
13. 매처 실패 상태도 출력한다.  
( ex. nuclei -ms -> options.MatcherStatus = true )  
  
14. 결과를 Markdown 형식으로 내보낼 디렉터리를 지정한다.  
( ex. nuclei -me markdown-report/ -> options.MarkdownExportDirectory )  
  
15. 결과를 SARIF 형식 파일로 내보낸다.  
( ex. nuclei -se result.sarif -> options.SarifExport )  
  
16. 결과를 JSON 형식 파일로 내보낸다.  
( ex. nuclei -je result.json -> options.JSONExport )  
  
17. 결과를 JSONL 형식 파일로 내보낸다.  
( ex. nuclei -jle result.jsonl -> options.JSONLExport )  
  
18. 결과를 PDF 형식 파일로 내보낸다.  
( ex. nuclei -pe result.pdf -> options.PDFExport )  
  
19. 요청 파라미터, 헤더, 바디에서 지정한 키를 마스킹한다.  
( ex. nuclei -rd token,password -> options.Redact )  
  
[CLI 옵션 등록]  
-o, -sresp, -srd, -silent, -nc, -j, -irr, -or, -ot,  
-nm, -ts, -rdb, -ms, -me, -se, -je, -jle, -pe, -rd  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-o → options.Output  
-sresp → options.StoreResponse  
-srd → options.StoreResponseDir  
-silent → options.Silent  
-nc → options.NoColor  
-j → options.JSONL  
-irr → options.JSONRequests  
-or → options.OmitRawRequests  
-ot → options.OmitTemplate  
-nm → options.NoMeta  
-ts → options.Timestamp  
-rdb → options.ReportingDB  
-ms → options.MatcherStatus  
-me → options.MarkdownExportDirectory  
-se → options.SarifExport  
-je → options.JSONExport  
-jle → options.JSONLExport  
-pe → options.PDFExport  
-rd → options.Redact


```
	flagSet.CreateGroup("configs", "Configurations",
		flagSet.StringVar(&cfgFile, "config", "", "path to the nuclei configuration file"),
		flagSet.StringVarP(&templateProfile, "profile", "tp", "", "template profile config file to run"),
		flagSet.BoolVarP(&options.ListTemplateProfiles, "profile-list", "tpl", false, "list community template profiles"),
		flagSet.BoolVarP(&options.FollowRedirects, "follow-redirects", "fr", false, "enable following redirects for http templates"),
		flagSet.BoolVarP(&options.FollowHostRedirects, "follow-host-redirects", "fhr", false, "follow redirects on the same host"),
		flagSet.IntVarP(&options.MaxRedirects, "max-redirects", "mr", 10, "max number of redirects to follow for http templates"),
		flagSet.BoolVarP(&options.DisableRedirects, "disable-redirects", "dr", false, "disable redirects for http templates"),
		flagSet.StringVarP(&options.ReportingConfig, "report-config", "rc", "", "nuclei reporting module configuration file"), // TODO merge into the config file or rename to issue-tracking
		flagSet.StringSliceVarP(&options.CustomHeaders, "header", "H", nil, "custom header/cookie to include in all http request in header:value format (cli, file)", goflags.FileStringSliceOptions),
		flagSet.RuntimeMapVarP(&options.Vars, "var", "V", nil, "custom vars in key=value format"),
		flagSet.StringVarP(&options.ResolversFile, "resolvers", "r", "", "file containing resolver list for nuclei"),
		flagSet.BoolVarP(&options.SystemResolvers, "system-resolvers", "sr", false, "use system DNS resolving as error fallback"),
		flagSet.BoolVarP(&options.DisableClustering, "disable-clustering", "dc", false, "disable clustering of requests"),
		flagSet.BoolVar(&options.OfflineHTTP, "passive", false, "enable passive HTTP response processing mode"),
		flagSet.BoolVarP(&options.ForceAttemptHTTP2, "force-http2", "fh2", false, "force http2 connection on requests"),
		flagSet.BoolVarP(&options.EnvironmentVariables, "env-vars", "ev", false, "enable environment variables to be used in template"),
		flagSet.StringVarP(&options.ClientCertFile, "client-cert", "cc", "", "client certificate file (PEM-encoded) used for authenticating against scanned hosts"),
		flagSet.StringVarP(&options.ClientKeyFile, "client-key", "ck", "", "client key file (PEM-encoded) used for authenticating against scanned hosts"),
		flagSet.StringVarP(&options.ClientCAFile, "client-ca", "ca", "", "client certificate authority file (PEM-encoded) used for authenticating against scanned hosts"),
		flagSet.BoolVarP(&options.ShowMatchLine, "show-match-line", "sml", false, "show match lines for file templates, works with extractors only"),
		flagSet.BoolVar(&options.ZTLS, "ztls", false, "use ztls library with autofallback to standard one for tls13 [Deprecated] autofallback to ztls is enabled by default"), //nolint:all
		flagSet.StringVar(&options.SNI, "sni", "", "tls sni hostname to use (default: input domain name)"),
		flagSet.DurationVarP(&options.DialerKeepAlive, "dialer-keep-alive", "dka", 0, "keep-alive duration for network requests."),
		flagSet.BoolVarP(&options.AllowLocalFileAccess, "allow-local-file-access", "lfa", false, "allows file (payload) access anywhere on the system"),
		flagSet.BoolVarP(&options.RestrictLocalNetworkAccess, "restrict-local-network-access", "lna", false, "blocks connections to the local / private network"),
		flagSet.StringVarP(&options.Interface, "interface", "i", "", "network interface to use for network scan"),
		flagSet.StringVarP(&options.AttackType, "attack-type", "at", "", "type of payload combinations to perform (batteringram,pitchfork,clusterbomb)"),
		flagSet.StringVarP(&options.SourceIP, "source-ip", "sip", "", "source ip address to use for network scan"),
		flagSet.IntVarP(&options.ResponseReadSize, "response-size-read", "rsr", 0, "max response size to read in bytes"),
		flagSet.IntVarP(&options.ResponseSaveSize, "response-size-save", "rss", unitutils.Mega, "max response size to read in bytes"),
		flagSet.CallbackVar(resetCallback, "reset", "reset removes all nuclei configuration and data files (including nuclei-templates)"),
		flagSet.BoolVarP(&options.TlsImpersonate, "tls-impersonate", "tlsi", false, "enable experimental client hello (ja3) tls randomization"),
		flagSet.StringVarP(&options.HttpApiEndpoint, "http-api-endpoint", "hae", "", "experimental http api endpoint"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
configs라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Configurations 그룹을 생성하고, Nuclei의 환경설정(Configuration), HTTP 설정, 네트워크 설정, 인증서 설정 등 실행 환경과 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.StringVar(...)에서 -config 옵션을 등록하고, FlagData 반환.  
flagSet.StringVarP(...)에서 -profile 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-config  
-profile  
-profile-list  
-follow-redirects  
-follow-host-redirects  
-max-redirects  
-disable-redirects  
-report-config  
-header  
-var  
-resolvers  
-system-resolvers  
-disable-clustering  
-passive  
-force-http2  
-env-vars  
-client-cert  
-client-key  
-client-ca  
-show-match-line  
-ztls  
-sni  
-dialer-keep-alive  
-allow-local-file-access  
-restrict-local-network-access  
-interface  
-attack-type  
-source-ip  
-response-size-read  
-response-size-save  
-reset  
-tls-impersonate  
-http-api-endpoint  
  
CreateGroup 호출 후, 모두 configs 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. Nuclei 설정 파일(config.yaml 등)의 경로를 지정한다.  
( ex. nuclei -config config.yaml -> cfgFile )  
  
2. Template Profile 설정 파일을 지정한다.  
( ex. nuclei -tp profile.yaml -> templateProfile )  
  
3. 사용 가능한 Template Profile 목록을 출력한다.  
( ex. nuclei -tpl -> options.ListTemplateProfiles = true )  
  
4. HTTP Redirect를 따라간다.  
( ex. nuclei -fr -> options.FollowRedirects = true )  
  
5. 같은 호스트의 Redirect만 따라간다.  
( ex. nuclei -fhr -> options.FollowHostRedirects = true )  
  
6. 최대 Redirect 횟수를 지정한다.  
( ex. nuclei -mr 5 -> options.MaxRedirects = 5 )  
  
7. Redirect를 비활성화한다.  
( ex. nuclei -dr -> options.DisableRedirects = true )  
  
8. Reporting 설정 파일을 지정한다.  
( ex. nuclei -rc report.yaml -> options.ReportingConfig )  
  
9. 모든 HTTP 요청에 사용자 정의 Header 또는 Cookie를 추가한다.  
( ex. nuclei -H "Authorization: Bearer token" -> options.CustomHeaders )  
  
10. 사용자 정의 변수를 key=value 형태로 전달한다.  
( ex. nuclei -V token=12345 -> options.Vars )  
  
11. DNS Resolver 목록 파일을 지정한다.  
( ex. nuclei -r resolvers.txt -> options.ResolversFile )  
  
12. DNS Resolver 실패 시 시스템 DNS를 사용한다.  
( ex. nuclei -sr -> options.SystemResolvers = true )  
  
13. 요청 Clustering 기능을 비활성화한다.  
( ex. nuclei -dc -> options.DisableClustering = true )  
  
14. Passive HTTP 응답 처리 모드를 활성화한다.  
( ex. nuclei -passive -> options.OfflineHTTP = true )  
  
15. HTTP/2 연결을 강제로 사용한다.  
( ex. nuclei -fh2 -> options.ForceAttemptHTTP2 = true )  
  
16. 템플릿에서 환경변수(Environment Variables)를 사용할 수 있도록 한다.  
( ex. nuclei -ev -> options.EnvironmentVariables = true )  
  
17. TLS 클라이언트 인증서 파일을 지정한다.  
( ex. nuclei -cc client.pem -> options.ClientCertFile )  
  
18. TLS 클라이언트 개인키 파일을 지정한다.  
( ex. nuclei -ck client.key -> options.ClientKeyFile )  
  
19. TLS 인증기관(CA) 파일을 지정한다.  
( ex. nuclei -ca ca.pem -> options.ClientCAFile )  
  
20. 파일(File) 템플릿에서 일치한 줄(Match Line)을 출력한다.  
( ex. nuclei -sml -> options.ShowMatchLine = true )  
  
21. ZTLS 라이브러리를 사용한다. (Deprecated)  
( ex. nuclei -ztls -> options.ZTLS = true )  
  
22. TLS SNI(Server Name Indication)에 사용할 호스트명을 지정한다.  
( ex. nuclei -sni example.com -> options.SNI )  
  
23. 네트워크 연결의 Keep-Alive 시간을 지정한다.  
( ex. nuclei -dka 30s -> options.DialerKeepAlive )  
  
24. 시스템의 모든 위치에 있는 Payload 파일 접근을 허용한다.  
( ex. nuclei -lfa -> options.AllowLocalFileAccess = true )  
  
25. 로컬 및 사설 네트워크 접근을 차단한다.  
( ex. nuclei -lna -> options.RestrictLocalNetworkAccess = true )  
  
26. 네트워크 스캔에 사용할 인터페이스를 지정한다.  
( ex. nuclei -i eth0 -> options.Interface )  
  
27. Payload 조합 방식을 지정한다.  
( ex. nuclei -at clusterbomb -> options.AttackType )  
  
28. 네트워크 스캔에 사용할 Source IP를 지정한다.  
( ex. nuclei -sip 192.168.1.100 -> options.SourceIP )  
  
29. 응답(Response)을 읽을 최대 크기를 지정한다.  
( ex. nuclei -rsr 1048576 -> options.ResponseReadSize )  
  
30. 응답(Response)을 저장할 최대 크기를 지정한다.  
( ex. nuclei -rss 10485760 -> options.ResponseSaveSize )  
  
31. Nuclei 설정 및 데이터(템플릿 포함)를 초기화한다.  
( ex. nuclei -reset -> resetCallback() 실행 )  
  
32. JA3 기반 TLS ClientHello 랜덤화를 활성화한다.  
( ex. nuclei -tlsi -> options.TlsImpersonate = true )  
  
33. 실험용 HTTP API Endpoint를 지정한다.  
( ex. nuclei -hae http://localhost:8080 -> options.HttpApiEndpoint )  
  
[CLI 옵션 등록]  
-config, -tp, -tpl, -fr, -fhr, -mr, -dr, -rc, -H, -V,  
-r, -sr, -dc, -passive, -fh2, -ev, -cc, -ck, -ca, -sml,  
-ztls, -sni, -dka, -lfa, -lna, -i, -at, -sip,  
-rsr, -rss, -reset, -tlsi, -hae  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-config → cfgFile  
-tp → templateProfile  
-tpl → options.ListTemplateProfiles  
-fr → options.FollowRedirects  
-fhr → options.FollowHostRedirects  
-mr → options.MaxRedirects  
-dr → options.DisableRedirects  
-rc → options.ReportingConfig  
-H → options.CustomHeaders  
-V → options.Vars  
-r → options.ResolversFile  
-sr → options.SystemResolvers  
-dc → options.DisableClustering  
-passive → options.OfflineHTTP  
-fh2 → options.ForceAttemptHTTP2  
-ev → options.EnvironmentVariables  
-cc → options.ClientCertFile  
-ck → options.ClientKeyFile  
-ca → options.ClientCAFile  
-sml → options.ShowMatchLine  
-ztls → options.ZTLS  
-sni → options.SNI  
-dka → options.DialerKeepAlive  
-lfa → options.AllowLocalFileAccess  
-lna → options.RestrictLocalNetworkAccess  
-i → options.Interface  
-at → options.AttackType  
-sip → options.SourceIP  
-rsr → options.ResponseReadSize  
-rss → options.ResponseSaveSize  
-reset → resetCallback()  
-tlsi → options.TlsImpersonate  
-hae → options.HttpApiEndpoint

```
	flagSet.CreateGroup("interactsh", "interactsh",
		flagSet.StringVarP(&options.InteractshURL, "interactsh-server", "iserver", "", fmt.Sprintf("interactsh server url for self-hosted instance (default: %s)", client.DefaultOptions.ServerURL)),
		flagSet.StringVarP(&options.InteractshToken, "interactsh-token", "itoken", "", "authentication token for self-hosted interactsh server"),
		flagSet.IntVar(&options.InteractionsCacheSize, "interactions-cache-size", 5000, "number of requests to keep in the interactions cache"),
		flagSet.IntVar(&options.InteractionsEviction, "interactions-eviction", 60, "number of seconds to wait before evicting requests from cache"),
		flagSet.IntVar(&options.InteractionsPollDuration, "interactions-poll-duration", 5, "number of seconds to wait before each interaction poll request"),
		flagSet.IntVar(&options.InteractionsCoolDownPeriod, "interactions-cooldown-period", 5, "extra time for interaction polling before exiting"),
		flagSet.BoolVarP(&options.NoInteractsh, "no-interactsh", "ni", false, "disable interactsh server for OAST testing, exclude OAST based templates"),
	)
```

#### FlagSet.CreateGroup 해석
 
 [흐름]  
interactsh라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 interactsh 그룹을 생성하고, OAST(Out-of-Band Application Security Testing) 기능 및 Interactsh 서버와 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.StringVarP(...)에서 -interactsh-server 옵션을 등록하고, FlagData 반환.  
flagSet.StringVarP(...)에서 -interactsh-token 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-interactsh-server  
-interactsh-token  
-interactions-cache-size  
-interactions-eviction  
-interactions-poll-duration  
-interactions-cooldown-period  
-no-interactsh  
  
CreateGroup 호출 후, 모두 interactsh 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. Self-Hosted Interactsh 서버의 URL을 지정한다.  
( ex. nuclei -iserver https://interact.example.com -> options.InteractshURL )  
  
2. Self-Hosted Interactsh 서버의 인증 토큰(Authentication Token)을 지정한다.  
( ex. nuclei -itoken my-token -> options.InteractshToken )  
  
3. 메모리에 유지할 Interactions(상호작용) 요청 개수를 지정한다.  
( ex. nuclei -interactions-cache-size 10000 -> options.InteractionsCacheSize )  
  
4. Interaction Cache에서 요청을 제거(Eviction)하기 전까지의 대기 시간을 초 단위로 지정한다.  
( ex. nuclei -interactions-eviction 120 -> options.InteractionsEviction )  
  
5. Interactsh 서버에 새로운 Interaction이 있는지 확인(Polling)하는 주기를 초 단위로 지정한다.  
( ex. nuclei -interactions-poll-duration 10 -> options.InteractionsPollDuration )  
  
6. 스캔 종료 전 마지막 Interaction을 기다리는 시간을 초 단위로 지정한다.  
( ex. nuclei -interactions-cooldown-period 15 -> options.InteractionsCoolDownPeriod )  
  
7. Interactsh 기능을 비활성화하여 OAST 기반 템플릿을 실행하지 않는다.  
( ex. nuclei -ni -> options.NoInteractsh = true )  
  
[CLI 옵션 등록]  
-iserver  
-itoken  
-interactions-cache-size  
-interactions-eviction  
-interactions-poll-duration  
-interactions-cooldown-period  
-ni  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-iserver → options.InteractshURL  
-itoken → options.InteractshToken  
-interactions-cache-size → options.InteractionsCacheSize  
-interactions-eviction → options.InteractionsEviction  
-interactions-poll-duration → options.InteractionsPollDuration  
-interactions-cooldown-period → options.InteractionsCoolDownPeriod  
-ni → options.NoInteractsh

```
	flagSet.CreateGroup("fuzzing", "Fuzzing",
		flagSet.StringVarP(&options.FuzzingType, "fuzzing-type", "ft", "", "overrides fuzzing type set in template (replace, prefix, postfix, infix)"),
		flagSet.StringVarP(&options.FuzzingMode, "fuzzing-mode", "fm", "", "overrides fuzzing mode set in template (multiple, single)"),
		flagSet.BoolVar(&fuzzFlag, "fuzz", false, "enable loading fuzzing templates (Deprecated: use -dast instead)"),
		flagSet.BoolVar(&options.DAST, "dast", false, "enable / run dast (fuzz) nuclei templates"),
		flagSet.BoolVarP(&options.DASTServer, "dast-server", "dts", false, "enable dast server mode (live fuzzing)"),
		flagSet.BoolVarP(&options.DASTReport, "dast-report", "dtr", false, "write dast scan report to file"),
		flagSet.StringVarP(&options.DASTServerToken, "dast-server-token", "dtst", "", "dast server token (optional)"),
		flagSet.StringVarP(&options.DASTServerAddress, "dast-server-address", "dtsa", "localhost:9055", "dast server address"),
		flagSet.BoolVarP(&options.DisplayFuzzPoints, "display-fuzz-points", "dfp", false, "display fuzz points in the output for debugging"),
		flagSet.IntVar(&options.FuzzParamFrequency, "fuzz-param-frequency", 10, "frequency of uninteresting parameters for fuzzing before skipping"),
		flagSet.StringVarP(&options.FuzzAggressionLevel, "fuzz-aggression", "fa", "low", "fuzzing aggression level controls payload count for fuzz (low, medium, high)"),
		flagSet.StringSliceVarP(&options.Scope, "fuzz-scope", "cs", nil, "in scope url regex to be followed by fuzzer", goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.StringSliceVarP(&options.OutOfScope, "fuzz-out-scope", "cos", nil, "out of scope url regex to be excluded by fuzzer", goflags.FileCommaSeparatedStringSliceOptions),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
fuzzing이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Fuzzing 그룹을 생성하고, Fuzzing 및 DAST(Dynamic Application Security Testing)와 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.StringVarP(...)에서 -fuzzing-type 옵션을 등록하고, FlagData 반환.  
flagSet.StringVarP(...)에서 -fuzzing-mode 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-fuzzing-type  
-fuzzing-mode  
-fuzz  
-dast  
-dast-server  
-dast-report  
-dast-server-token  
-dast-server-address  
-display-fuzz-points  
-fuzz-param-frequency  
-fuzz-aggression  
-fuzz-scope  
-fuzz-out-scope  
  
CreateGroup 호출 후, 모두 fuzzing 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. 템플릿에 설정된 Fuzzing 방식을 덮어쓴다.  
(replace, prefix, postfix, infix 중 선택)  
( ex. nuclei -ft prefix -> options.FuzzingType )  
  
2. 템플릿에 설정된 Fuzzing 실행 방식을 덮어쓴다.  
(multiple, single 중 선택)  
( ex. nuclei -fm multiple -> options.FuzzingMode )  
  
3. Fuzzing 템플릿 실행을 활성화한다. (Deprecated, 현재는 -dast 사용)  
( ex. nuclei -fuzz -> fuzzFlag = true )  
  
4. DAST(Fuzz) 템플릿 실행을 활성화한다.  
( ex. nuclei -dast -> options.DAST = true )  
  
5. DAST 서버 모드(Live Fuzzing)를 활성화한다.  
( ex. nuclei -dts -> options.DASTServer = true )  
  
6. DAST 스캔 결과를 파일로 저장한다.  
( ex. nuclei -dtr -> options.DASTReport = true )  
  
7. DAST 서버 인증 토큰을 지정한다.  
( ex. nuclei -dtst my-token -> options.DASTServerToken )  
  
8. DAST 서버 주소를 지정한다.  
( ex. nuclei -dtsa localhost:9055 -> options.DASTServerAddress )  
  
9. 디버깅을 위해 Fuzz Point를 출력한다.  
( ex. nuclei -dfp -> options.DisplayFuzzPoints = true )  
  
10. 효과가 없는(흥미롭지 않은) 파라미터를 몇 번까지 퍼징할지 지정한다.  
( ex. nuclei -fuzz-param-frequency 20 -> options.FuzzParamFrequency )  
  
11. Fuzzing 강도를 지정한다.  
(low, medium, high 중 선택)  
( ex. nuclei -fa high -> options.FuzzAggressionLevel )  
  
12. Fuzzer가 따라갈 URL 범위를 정규표현식(Regex)으로 지정한다.  
( ex. nuclei -cs "https://example.com/.*" -> options.Scope )  
  
13. Fuzzer가 제외할 URL 범위를 정규표현식(Regex)으로 지정한다.  
( ex. nuclei -cos "https://example.com/logout.*" -> options.OutOfScope )  
  
[CLI 옵션 등록]  
-ft  
-fm  
-fuzz  
-dast  
-dts  
-dtr  
-dtst  
-dtsa  
-dfp  
-fuzz-param-frequency  
-fa  
-cs  
-cos  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-ft → options.FuzzingType  
-fm → options.FuzzingMode  
-fuzz → fuzzFlag  
-dast → options.DAST  
-dts → options.DASTServer  
-dtr → options.DASTReport  
-dtst → options.DASTServerToken  
-dtsa → options.DASTServerAddress  
-dfp → options.DisplayFuzzPoints  
-fuzz-param-frequency → options.FuzzParamFrequency  
-fa → options.FuzzAggressionLevel  
-cs → options.Scope  
-cos → options.OutOfScope

```
	flagSet.CreateGroup("uncover", "Uncover",
		flagSet.BoolVarP(&options.Uncover, "uncover", "uc", false, "enable uncover engine"),
		flagSet.StringSliceVarP(&options.UncoverQuery, "uncover-query", "uq", nil, "uncover search query", goflags.FileStringSliceOptions),
		flagSet.StringSliceVarP(&options.UncoverEngine, "uncover-engine", "ue", nil, fmt.Sprintf("uncover search engine (%s) (default shodan)", uncover.GetUncoverSupportedAgents()), goflags.FileStringSliceOptions),
		flagSet.StringVarP(&options.UncoverField, "uncover-field", "uf", "ip:port", "uncover fields to return (ip,port,host)"),
		flagSet.IntVarP(&options.UncoverLimit, "uncover-limit", "ul", 100, "uncover results to return"),
		flagSet.IntVarP(&options.UncoverRateLimit, "uncover-ratelimit", "ur", 60, "override ratelimit of engines with unknown ratelimit (default 60 req/min)"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
uncover라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Uncover 그룹을 생성하고, Uncover 엔진을 이용한 인터넷 자산 검색과 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.BoolVarP(...)에서 -uncover 옵션을 등록하고, FlagData 반환.  
flagSet.StringSliceVarP(...)에서 -uncover-query 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-uncover  
-uncover-query  
-uncover-engine  
-uncover-field  
-uncover-limit  
-uncover-ratelimit  
  
CreateGroup 호출 후, 모두 uncover 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. Uncover 엔진을 활성화한다.  
( ex. nuclei -uc -> options.Uncover = true )  
  
2. Uncover 검색(Query)을 지정한다.  
( ex. nuclei -uq "apache" -> options.UncoverQuery )  
  
3. 검색에 사용할 Uncover 검색 엔진(Shodan, Censys 등)을 지정한다.  
( ex. nuclei -ue shodan -> options.UncoverEngine )  
  
4. 검색 결과에서 반환할 필드를 지정한다.  
(ip, port, host 등)  
( ex. nuclei -uf host -> options.UncoverField )  
  
5. 검색 결과를 최대 몇 개까지 가져올지 지정한다.  
( ex. nuclei -ul 500 -> options.UncoverLimit )  
  
6. 검색 엔진의 요청 속도(Rate Limit)를 지정한다.  
( ex. nuclei -ur 120 -> options.UncoverRateLimit )  
  
[CLI 옵션 등록]  
-uc  
-uq  
-ue  
-uf  
-ul  
-ur  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-uc → options.Uncover  
-uq → options.UncoverQuery  
-ue → options.UncoverEngine  
-uf → options.UncoverField  
-ul → options.UncoverLimit  
-ur → options.UncoverRateLimit

```
	flagSet.CreateGroup("rate-limit", "Rate-Limit",
		flagSet.IntVarP(&options.RateLimit, "rate-limit", "rl", 150, "maximum number of requests to send per second"),
		flagSet.DurationVarP(&options.RateLimitDuration, "rate-limit-duration", "rld", time.Second, "maximum number of requests to send per second"),
		flagSet.BoolVar(&options.PerHostRateLimit, "per-host-rate-limit", false, "enable per-host rate limiting (global rate limit becomes unlimited when enabled)"),
		flagSet.IntVarP(&options.RateLimitMinute, "rate-limit-minute", "rlm", 0, "maximum number of requests to send per minute (DEPRECATED)"),
		flagSet.IntVarP(&options.BulkSize, "bulk-size", "bs", 25, "maximum number of hosts to be analyzed in parallel per template"),
		flagSet.IntVarP(&options.TemplateThreads, "concurrency", "c", 25, "maximum number of templates to be executed in parallel"),
		flagSet.IntVarP(&options.HeadlessBulkSize, "headless-bulk-size", "hbs", 10, "maximum number of headless hosts to be analyzed in parallel per template"),
		flagSet.IntVarP(&options.HeadlessTemplateThreads, "headless-concurrency", "headc", 10, "maximum number of headless templates to be executed in parallel"),
		flagSet.IntVarP(&options.JsConcurrency, "js-concurrency", "jsc", 120, "maximum number of javascript runtimes to be executed in parallel"),
		flagSet.IntVarP(&options.PayloadConcurrency, "payload-concurrency", "pc", 25, "max payload concurrency for each template"),
		flagSet.IntVarP(&options.ProbeConcurrency, "probe-concurrency", "prc", 50, "http probe concurrency with httpx"),
		flagSet.IntVarP(&options.TemplateLoadingConcurrency, "template-loading-concurrency", "tlc", types.DefaultTemplateLoadingConcurrency, "maximum number of concurrent template loading operations"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
rate-limit이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Rate-Limit 그룹을 생성하고, 요청 속도(Rate Limit), 동시 실행 수(Concurrency), 병렬 처리와 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.IntVarP(...)에서 -rate-limit 옵션을 등록하고, FlagData 반환.  
flagSet.DurationVarP(...)에서 -rate-limit-duration 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-rate-limit  
-rate-limit-duration  
-per-host-rate-limit  
-rate-limit-minute  
-bulk-size  
-concurrency  
-headless-bulk-size  
-headless-concurrency  
-js-concurrency  
-payload-concurrency  
-probe-concurrency  
-template-loading-concurrency  
  
CreateGroup 호출 후, 모두 rate-limit 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. 초당 최대 요청(Request) 수를 지정한다.  
( ex. nuclei -rl 200 -> options.RateLimit = 200 )  
  
2. 요청 속도를 제한하는 시간 단위를 지정한다.  
( ex. nuclei -rld 500ms -> options.RateLimitDuration )  
  
3. 호스트(Host)별 Rate Limit을 적용한다.  
활성화하면 전체(Global) Rate Limit은 제한 없이 동작한다.  
( ex. nuclei -per-host-rate-limit -> options.PerHostRateLimit = true )  
  
4. 분당 최대 요청 수를 지정한다. (Deprecated)  
( ex. nuclei -rlm 600 -> options.RateLimitMinute = 600 )  
  
5. 템플릿 하나당 동시에 분석할 Host 개수를 지정한다.  
( ex. nuclei -bs 50 -> options.BulkSize = 50 )  
  
6. 동시에 실행할 템플릿 개수를 지정한다.  
( ex. nuclei -c 50 -> options.TemplateThreads = 50 )  
  
7. Headless 템플릿에서 동시에 분석할 Host 개수를 지정한다.  
( ex. nuclei -hbs 20 -> options.HeadlessBulkSize = 20 )  
  
8. Headless 템플릿을 동시에 실행할 개수를 지정한다.  
( ex. nuclei -headc 20 -> options.HeadlessTemplateThreads = 20 )  
  
9. 동시에 실행할 JavaScript Runtime 개수를 지정한다.  
( ex. nuclei -jsc 200 -> options.JsConcurrency = 200 )  
  
10. 템플릿 하나에서 동시에 실행할 Payload 개수를 지정한다.  
( ex. nuclei -pc 50 -> options.PayloadConcurrency = 50 )  
  
11. httpx를 이용한 HTTP Probe의 동시 실행 개수를 지정한다.  
( ex. nuclei -prc 100 -> options.ProbeConcurrency = 100 )  
  
12. 템플릿을 동시에 로드(Load)할 최대 개수를 지정한다.  
( ex. nuclei -tlc 40 -> options.TemplateLoadingConcurrency = 40 )  
  
[CLI 옵션 등록]  
-rl  
-rld  
-per-host-rate-limit  
-rlm  
-bs  
-c  
-hbs  
-headc  
-jsc  
-pc  
-prc  
-tlc  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-rl → options.RateLimit  
-rld → options.RateLimitDuration  
-per-host-rate-limit → options.PerHostRateLimit  
-rlm → options.RateLimitMinute  
-bs → options.BulkSize  
-c → options.TemplateThreads  
-hbs → options.HeadlessBulkSize  
-headc → options.HeadlessTemplateThreads  
-jsc → options.JsConcurrency  
-pc → options.PayloadConcurrency  
-prc → options.ProbeConcurrency  
-tlc → options.TemplateLoadingConcurrency

```
	flagSet.CreateGroup("optimization", "Optimizations",
		flagSet.IntVar(&options.Timeout, "timeout", 10, "time to wait in seconds before timeout"),
		flagSet.IntVar(&options.Retries, "retries", 1, "number of times to retry a failed request"),
		flagSet.BoolVarP(&options.LeaveDefaultPorts, "leave-default-ports", "ldp", false, "leave default HTTP/HTTPS ports (eg. host:80,host:443)"),
		flagSet.IntVarP(&options.MaxHostError, "max-host-error", "mhe", 30, "max errors for a host before skipping from scan"),
		flagSet.StringSliceVarP(&options.TrackError, "track-error", "te", nil, "adds given error to max-host-error watchlist (standard, file)", goflags.FileStringSliceOptions),
		flagSet.BoolVarP(&options.NoHostErrors, "no-mhe", "nmhe", false, "disable skipping host from scan based on errors"),
		flagSet.BoolVar(&options.Project, "project", false, "use a project folder to avoid sending same request multiple times"),
		flagSet.StringVar(&options.ProjectPath, "project-path", os.TempDir(), "set a specific project path"),
		flagSet.BoolVarP(&options.StopAtFirstMatch, "stop-at-first-match", "spm", false, "stop processing HTTP requests after the first match (may break template/workflow logic)"),
		flagSet.BoolVar(&options.Stream, "stream", false, "stream mode - start elaborating without sorting the input"),
		flagSet.EnumVarP(&options.ScanStrategy, "scan-strategy", "ss", goflags.EnumVariable(0), "strategy to use while scanning(auto/host-spray/template-spray)", goflags.AllowdTypes{
			scanstrategy.Auto.String():          goflags.EnumVariable(0),
			scanstrategy.HostSpray.String():     goflags.EnumVariable(1),
			scanstrategy.TemplateSpray.String(): goflags.EnumVariable(2),
		}),
		flagSet.DurationVarP(&options.InputReadTimeout, "input-read-timeout", "irt", time.Duration(3*time.Minute), "timeout on input read"),
		flagSet.BoolVarP(&options.DisableHTTPProbe, "no-httpx", "nh", false, "disable httpx probing for non-url input"),
		flagSet.BoolVar(&options.PreflightPortScan, "preflight-portscan", false, "run preflight resolve + TCP portscan and filter targets before scanning (disabled by default)"),
		flagSet.BoolVar(&options.DisableStdin, "no-stdin", false, "disable stdin processing"),
	)
```

#### FlagSet.CreateGroup 해석
[흐름]  
optimization이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Optimizations 그룹을 생성하고, 스캔 성능 최적화, 재시도, 프로젝트 관리, 스캔 전략 등 실행 성능과 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.IntVar(...)에서 -timeout 옵션을 등록하고, FlagData 반환.  
flagSet.IntVar(...)에서 -retries 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-timeout  
-retries  
-leave-default-ports  
-max-host-error  
-track-error  
-no-mhe  
-project  
-project-path  
-stop-at-first-match  
-stream  
-scan-strategy  
-input-read-timeout  
-no-httpx  
-preflight-portscan  
-no-stdin  
  
CreateGroup 호출 후, 모두 optimization 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. 요청(Request)의 최대 대기 시간(초)을 지정한다.  
( ex. nuclei -timeout 30 -> options.Timeout = 30 )  
  
2. 실패한 요청(Request)의 재시도 횟수를 지정한다.  
( ex. nuclei -retries 3 -> options.Retries = 3 )  
  
3. 기본 HTTP/HTTPS 포트(80, 443)를 URL에 그대로 유지한다.  
( ex. nuclei -ldp -> options.LeaveDefaultPorts = true )  
  
4. 하나의 Host에서 허용할 최대 오류 횟수를 지정한다.  
( ex. nuclei -mhe 50 -> options.MaxHostError = 50 )  
  
5. 최대 오류(Max Host Error)에 포함할 오류 종류를 지정한다.  
( ex. nuclei -te timeout,connection-reset -> options.TrackError )  
  
6. 오류가 많아도 Host를 건너뛰지 않는다.  
( ex. nuclei -nmhe -> options.NoHostErrors = true )  
  
7. 동일한 요청을 여러 번 보내지 않도록 Project 기능을 활성화한다.  
( ex. nuclei -project -> options.Project = true )  
  
8. Project 데이터를 저장할 디렉터리를 지정한다.  
( ex. nuclei -project-path /tmp/nuclei -> options.ProjectPath )  
  
9. 첫 번째 취약점을 발견하면 해당 HTTP 요청 처리를 중단한다.  
( ex. nuclei -spm -> options.StopAtFirstMatch = true )  
  
10. 입력을 정렬하지 않고 바로 스캔을 시작하는 Stream 모드를 활성화한다.  
( ex. nuclei -stream -> options.Stream = true )  
  
11. 스캔 전략을 지정한다.  
(auto, host-spray, template-spray 중 선택)  
( ex. nuclei -ss host-spray -> options.ScanStrategy )  
  
12. 입력(Input)을 읽을 최대 대기 시간을 지정한다.  
( ex. nuclei -irt 5m -> options.InputReadTimeout )  
  
13. URL이 아닌 입력에 대해 httpx Probe를 수행하지 않는다.  
( ex. nuclei -nh -> options.DisableHTTPProbe = true )  
  
14. 스캔 전에 DNS Resolve 및 TCP Port Scan을 먼저 수행한다.  
( ex. nuclei -preflight-portscan -> options.PreflightPortScan = true )  
  
15. 표준 입력(stdin) 처리를 비활성화한다.  
( ex. nuclei -no-stdin -> options.DisableStdin = true )  
  
[CLI 옵션 등록]  
-timeout  
-retries  
-ldp  
-mhe  
-te  
-nmhe  
-project  
-project-path  
-spm  
-stream  
-ss  
-irt  
-nh  
-preflight-portscan  
-no-stdin  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-timeout → options.Timeout  
-retries → options.Retries  
-ldp → options.LeaveDefaultPorts  
-mhe → options.MaxHostError  
-te → options.TrackError  
-nmhe → options.NoHostErrors  
-project → options.Project  
-project-path → options.ProjectPath  
-spm → options.StopAtFirstMatch  
-stream → options.Stream  
-ss → options.ScanStrategy  
-irt → options.InputReadTimeout  
-nh → options.DisableHTTPProbe  
-preflight-portscan → options.PreflightPortScan  
-no-stdin → options.DisableStdin

```
	flagSet.CreateGroup("headless", "Headless",
		flagSet.BoolVar(&options.Headless, "headless", false, "enable templates that require headless browser support (root user on Linux will disable sandbox)"),
		flagSet.IntVar(&options.PageTimeout, "page-timeout", 20, "seconds to wait for each page in headless mode"),
		flagSet.BoolVarP(&options.ShowBrowser, "show-browser", "sb", false, "show the browser on the screen when running templates with headless mode"),
		flagSet.StringSliceVarP(&options.HeadlessOptionalArguments, "headless-options", "ho", nil, "start headless chrome with additional options", goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.BoolVarP(&options.UseInstalledChrome, "system-chrome", "sc", false, "use local installed Chrome browser instead of nuclei installed"),
		flagSet.StringVarP(&options.CDPEndpoint, "cdp-endpoint", "cdpe", "", "use remote browser via Chrome DevTools Protocol (CDP) endpoint"),
		flagSet.BoolVarP(&options.ShowActions, "list-headless-action", "lha", false, "list available headless actions"),
	)
```

#### FlagSet.CreateGroup 해석
[흐름]  
headless라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Headless 그룹을 생성하고, Headless Browser(Chrome) 기반 스캔과 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.BoolVar(...)에서 -headless 옵션을 등록하고, FlagData 반환.  
flagSet.IntVar(...)에서 -page-timeout 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-headless  
-page-timeout  
-show-browser  
-headless-options  
-system-chrome  
-cdp-endpoint  
-list-headless-action  
  
CreateGroup 호출 후, 모두 headless 그룹으로 지정.  
  
[의미]  
1. 이해를 돕고자 예를 들어 설명했습니다.  
  
2. Headless Browser가 필요한 템플릿 실행을 활성화한다.  
( ex. nuclei -headless -> options.Headless = true )  
  
3. Headless 모드에서 각 페이지의 최대 대기 시간을 지정한다.  
( ex. nuclei -page-timeout 30 -> options.PageTimeout = 30 )  
  
4. Headless 스캔 시 브라우저 창을 화면에 표시한다.  
( ex. nuclei -sb -> options.ShowBrowser = true )  
  
5. Headless Chrome 실행 시 추가 실행 옵션을 전달한다.  
( ex. nuclei -ho "--disable-gpu,--incognito" -> options.HeadlessOptionalArguments )  
  
6. Nuclei가 설치한 Chrome 대신 시스템에 설치된 Chrome을 사용한다.  
( ex. nuclei -sc -> options.UseInstalledChrome = true )  
  
7. 원격 Chrome Browser(CDP)를 사용하기 위한 Chrome DevTools Protocol Endpoint를 지정한다.  
( ex. nuclei -cdpe http://127.0.0.1:9222 -> options.CDPEndpoint )  
  
8. 사용 가능한 Headless Action 목록을 출력한다.  
( ex. nuclei -lha -> options.ShowActions = true )  
  
[CLI 옵션 등록]  
-headless  
-page-timeout  
-sb  
-ho  
-sc  
-cdpe  
-lha  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-headless → options.Headless  
-page-timeout → options.PageTimeout  
-sb → options.ShowBrowser  
-ho → options.HeadlessOptionalArguments  
-sc → options.UseInstalledChrome  
-cdpe → options.CDPEndpoint  
-lha → options.ShowActions

```
	flagSet.CreateGroup("debug", "Debug",
		flagSet.BoolVar(&options.Debug, "debug", false, "show all requests and responses"),
		flagSet.BoolVarP(&options.DebugRequests, "debug-req", "dreq", false, "show all sent requests"),
		flagSet.BoolVarP(&options.DebugResponse, "debug-resp", "dresp", false, "show all received responses"),
		flagSet.StringSliceVarP(&options.Proxy, "proxy", "p", nil, "list of http/socks5 proxy to use (comma separated or file input)",                                                      goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.BoolVarP(&options.ProxyInternal, "proxy-internal", "pi", false, "proxy all internal requests"),
		flagSet.BoolVarP(&options.ListDslSignatures, "list-dsl-function", "ldf", false, "list all supported DSL function signatures"),
		flagSet.StringVarP(&options.TraceLogFile, "trace-log", "tlog", "", "file to write sent requests trace log"),
		flagSet.StringVarP(&options.ErrorLogFile, "error-log", "elog", "", "file to write sent requests error log"),
		flagSet.CallbackVar(printVersion, "version", "show nuclei version"),
		flagSet.BoolVarP(&options.HangMonitor, "hang-monitor", "hm", false, "enable nuclei hang monitoring"),
		flagSet.BoolVarP(&options.Verbose, "verbose", "v", false, "show verbose output"),
		flagSet.StringVar(&memProfile, "profile-mem", "", "generate memory (heap) profile & trace files"),
		flagSet.BoolVar(&options.VerboseVerbose, "vv", false, "display templates loaded for scan"),
		flagSet.BoolVarP(&options.ShowVarDump, "show-var-dump", "svd", false, "show variables dump for debugging"),
		flagSet.IntVarP(&options.VarDumpLimit, "var-dump-limit", "vdl", 255, "limit the number of characters displayed in var dump"),
		flagSet.BoolVarP(&options.EnablePprof, "enable-pprof", "ep", false, "enable pprof debugging server"),
		flagSet.CallbackVarP(printTemplateVersion, "templates-version", "tv", "shows the version of the installed nuclei-templates"),
		flagSet.BoolVarP(&options.HealthCheck, "health-check", "hc", false, "run diagnostic check up"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]
debug라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Debug 그룹을 생성하고, 디버깅(Debug), 로그(Log), 프록시(Proxy), 프로파일링(Profiling), 버전 확인과 관련된 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.BoolVar(...)에서 -debug 옵션을 등록하고, FlagData 반환.
flagSet.BoolVarP(...)에서 -debug-req 옵션을 등록하고, FlagData 반환.

아래 옵션들도 모두 등록됨.
-debug
-debug-req
-debug-resp
-proxy
-proxy-internal
-list-dsl-function
-trace-log
-error-log
-version
-hang-monitor
-verbose
-profile-mem
-vv
-show-var-dump
-var-dump-limit
-enable-pprof
-templates-version
-health-check

CreateGroup 호출 후, 모두 debug 그룹으로 지정.

[의미]
1. 이해를 돕고자 예를 들어 설명했습니다.

2. 모든 요청(Request)과 응답(Response)을 출력한다.
( ex. nuclei -debug -> options.Debug = true )

3. 전송한 요청(Request)만 출력한다.
( ex. nuclei -dreq -> options.DebugRequests = true )

4. 수신한 응답(Response)만 출력한다.
( ex. nuclei -dresp -> options.DebugResponse = true )

5. HTTP/SOCKS5 Proxy를 지정한다.
( ex. nuclei -p http://127.0.0.1:8080 -> options.Proxy )

6. 내부 요청도 Proxy를 사용하도록 설정한다.
( ex. nuclei -pi -> options.ProxyInternal = true )

7. 사용 가능한 DSL 함수 목록을 출력한다.
( ex. nuclei -ldf -> options.ListDslSignatures = true )

8. 요청(Request) 추적 로그를 저장할 파일을 지정한다.
( ex. nuclei -tlog trace.log -> options.TraceLogFile )

9. 에러(Error) 로그를 저장할 파일을 지정한다.
( ex. nuclei -elog error.log -> options.ErrorLogFile )

10. 현재 Nuclei 버전을 출력한다.
( ex. nuclei -version -> printVersion() 실행 )

11. 프로그램 멈춤(Hang)을 감시한다.
( ex. nuclei -hm -> options.HangMonitor = true )

12. 상세(Verbose) 정보를 출력한다.
( ex. nuclei -v -> options.Verbose = true )

13. 메모리(Heap) 프로파일과 Trace 파일을 생성한다.
( ex. nuclei -profile-mem profile -> memProfile = "profile" )

14. 로드된 템플릿 목록까지 상세하게 출력한다.
( ex. nuclei -vv -> options.VerboseVerbose = true )

15. 디버깅을 위해 변수(Variable) 값을 출력한다.
( ex. nuclei -svd -> options.ShowVarDump = true )

16. 변수 출력 시 최대 문자 수를 지정한다.
( ex. nuclei -vdl 500 -> options.VarDumpLimit = 500 )

17. pprof 디버깅 서버를 활성화한다.
( ex. nuclei -ep -> options.EnablePprof = true )

18. 설치된 nuclei-templates의 버전을 출력한다.
( ex. nuclei -tv -> printTemplateVersion() 실행 )

19. 환경 진단(Health Check)을 수행한다.
( ex. nuclei -hc -> options.HealthCheck = true )

[CLI 옵션 등록]
-debug
-dreq
-dresp
-p
-pi
-ldf
-tlog
-elog
-version
-hm
-v
-profile-mem
-vv
-svd
-vdl
-ep
-tv
-hc

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
-debug → options.Debug
-dreq → options.DebugRequests
-dresp → options.DebugResponse
-p → options.Proxy
-pi → options.ProxyInternal
-ldf → options.ListDslSignatures
-tlog → options.TraceLogFile
-elog → options.ErrorLogFile
-version → printVersion()
-hm → options.HangMonitor
-v → options.Verbose
-profile-mem → memProfile
-vv → options.VerboseVerbose
-svd → options.ShowVarDump
-vdl → options.VarDumpLimit
-ep → options.EnablePprof
-tv → printTemplateVersion()
-hc → options.HealthCheck

```
	flagSet.CreateGroup("update", "Update",
		flagSet.BoolVarP(&updateNucleiBinary, "update", "up", false, "update nuclei engine to the latest released version"),
		flagSet.BoolVarP(&options.UpdateTemplates, "update-templates", "ut", false, "update nuclei-templates to latest released version"),
		flagSet.StringVarP(&options.NewTemplatesDirectory, "update-template-dir", "ud", "", "custom directory to install / update nuclei-templates"),
		flagSet.CallbackVarP(disableUpdatesCallback, "disable-update-check", "duc", "disable automatic nuclei/templates update check"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
update라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Update 그룹을 생성하고, Nuclei 엔진 및 nuclei-templates 업데이트와 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.BoolVarP(...)에서 -update 옵션을 등록하고, FlagData 반환.  
flagSet.BoolVarP(...)에서 -update-templates 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-update  
-update-templates  
-update-template-dir  
-disable-update-check  
  
CreateGroup 호출 후, 모두 update 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. Nuclei 엔진을 최신 릴리스 버전으로 업데이트한다.  
( ex. nuclei -up -> updateNucleiBinary = true )  
  
2. nuclei-templates를 최신 릴리스 버전으로 업데이트한다.  
( ex. nuclei -ut -> options.UpdateTemplates = true )  
  
3. nuclei-templates를 설치하거나 업데이트할 디렉터리를 지정한다.  
( ex. nuclei -ud /home/user/nuclei-templates -> options.NewTemplatesDirectory )  
  
4. 자동 업데이트 확인(Update Check)을 비활성화한다.  
( ex. nuclei -duc -> disableUpdatesCallback() 실행 )  
  
[CLI 옵션 등록]  
-up  
-ut  
-ud  
-duc  
  
[입력된 값을 변수 또는 options 구조체와 연결(바인딩) 한다.]  
-up → updateNucleiBinary  
-ut → options.UpdateTemplates  
-ud → options.NewTemplatesDirectory  
-duc → disableUpdatesCallback()

```
	flagSet.CreateGroup("Honeypot", "Honeypot",
		flagSet.BoolVarP(&options.HoneypotDetection, "honeypot-detect", "hpd", false, "detect potential honeypot hosts based on match concentration"),
		flagSet.IntVarP(&options.HoneypotThreshold, "honeypot-threshold", "hpt", 15, "number of distinct template IDs required to flag a honeypot host"),
		flagSet.BoolVarP(&options.SuppressHoneypotResults, "suppress-honeypot", "shp", false, "suppress output for flagged honeypot hosts"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
Honeypot이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Honeypot 그룹을 생성하고, Honeypot(허니팟) 탐지 및 처리와 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.BoolVarP(...)에서 -honeypot-detect 옵션을 등록하고, FlagData 반환.  
flagSet.IntVarP(...)에서 -honeypot-threshold 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-honeypot-detect  
-honeypot-threshold  
-suppress-honeypot  
  
CreateGroup 호출 후, 모두 Honeypot 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. 허니팟(Honeypot)으로 의심되는 호스트를 탐지한다.  
( ex. nuclei -hpd -> options.HoneypotDetection = true )  
  
2. 허니팟으로 판단할 기준이 되는 서로 다른 Template ID 개수를 지정한다.  
기본값은 15개이다.  
( ex. nuclei -hpt 20 -> options.HoneypotThreshold = 20 )  
  
3. 허니팟으로 판단된 호스트의 결과를 출력하지 않는다.  
( ex. nuclei -shp -> options.SuppressHoneypotResults = true )  
  
[CLI 옵션 등록]  
-hpd  
-hpt  
-shp  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-hpd → options.HoneypotDetection  
-hpt → options.HoneypotThreshold  
-shp → options.SuppressHoneypotResults

```
	flagSet.CreateGroup("stats", "Statistics",
		flagSet.BoolVar(&options.EnableProgressBar, "stats", false, "display statistics about the running scan"),
		flagSet.BoolVarP(&options.StatsJSON, "stats-json", "sj", false, "display statistics in JSONL(ines) format"),
		flagSet.IntVarP(&options.StatsInterval, "stats-interval", "si", 5, "number of seconds to wait between showing a statistics update"),
		flagSet.IntVarP(&options.MetricsPort, "metrics-port", "mp", 9092, "port to expose nuclei metrics on"),
		flagSet.BoolVarP(&options.HTTPStats, "http-stats", "hps", false, "enable http status capturing (experimental)"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]
stats라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Statistics 그룹을 생성하고, 스캔 진행 상황 및 통계(Statistics), 메트릭(Metrics)과 관련된 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.BoolVar(...)에서 -stats 옵션을 등록하고, FlagData 반환.
flagSet.BoolVarP(...)에서 -stats-json 옵션을 등록하고, FlagData 반환.

아래 옵션들도 모두 등록됨.
-stats
-stats-json
-stats-interval
-metrics-port
-http-stats

CreateGroup 호출 후, 모두 stats 그룹으로 지정.

[의미]
이해를 돕고자 예를 들어 설명했습니다.

1. 실행 중인 스캔의 진행 상황 및 통계를 화면에 출력한다.
( ex. nuclei -stats -> options.EnableProgressBar = true )

2. 통계 정보를 JSONL 형식으로 출력한다.
( ex. nuclei -sj -> options.StatsJSON = true )

3. 통계 정보를 몇 초마다 갱신하여 출력할지 지정한다.
( ex. nuclei -si 10 -> options.StatsInterval = 10 )

4. Nuclei의 메트릭(Metrics)을 외부에 제공할 포트를 지정한다.
( ex. nuclei -mp 9090 -> options.MetricsPort = 9090 )

5. HTTP 상태(Status) 정보를 수집한다. (실험 기능)
( ex. nuclei -hps -> options.HTTPStats = true )

[CLI 옵션 등록]
-stats
-sj
-si
-mp
-hps

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
-stats → options.EnableProgressBar
-sj → options.StatsJSON
-si → options.StatsInterval
-mp → options.MetricsPort
-hps → options.HTTPStats

```
	flagSet.CreateGroup("cloud", "Cloud",
		flagSet.DynamicVar(&pdcpauth, "auth", "true", "configure projectdiscovery cloud (pdcp) api key"),
		flagSet.StringVarP(&options.TeamID, "team-id", "tid", _pdcp.TeamIDEnv, "upload scan results to given team id (optional)"),
		flagSet.BoolVarP(&options.EnableCloudUpload, "cloud-upload", "cup", false, "upload scan results to pdcp dashboard [DEPRECATED use -dashboard]"),
		flagSet.StringVarP(&options.ScanID, "scan-id", "sid", "", "upload scan results to existing scan id (optional)"),
		flagSet.StringVarP(&options.ScanName, "scan-name", "sname", "", "scan name to set (optional)"),
		flagSet.BoolVarP(&options.EnableCloudUpload, "dashboard", "pd", false, "upload / view nuclei results in projectdiscovery cloud (pdcp) UI dashboard"),
		flagSet.StringVarP(&options.ScanUploadFile, "dashboard-upload", "pdu", "", "upload / view nuclei results file (jsonl) in projectdiscovery cloud (pdcp) UI dashboard"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]
cloud라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Cloud 그룹을 생성하고, ProjectDiscovery Cloud(PDCP) 연동 및 스캔 결과 업로드와 관련된 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.DynamicVar(...)에서 -auth 옵션을 등록하고, FlagData 반환.
flagSet.StringVarP(...)에서 -team-id 옵션을 등록하고, FlagData 반환.

아래 옵션들도 모두 등록됨.
-auth
-team-id
-cloud-upload
-scan-id
-scan-name
-dashboard
-dashboard-upload

CreateGroup 호출 후, 모두 cloud 그룹으로 지정.

[의미]
이해를 돕고자 예를 들어 설명했습니다.

1. ProjectDiscovery Cloud(PDCP)의 API Key를 설정한다.
( ex. nuclei -auth <API_KEY> -> pdcpauth )

2. 스캔 결과를 업로드할 Team ID를 지정한다.
( ex. nuclei -tid team123 -> options.TeamID )

3. 스캔 결과를 PDCP에 업로드한다. (Deprecated, 현재는 -dashboard 사용)
( ex. nuclei -cup -> options.EnableCloudUpload = true )

4. 기존 Scan ID에 결과를 업로드한다.
( ex. nuclei -sid scan123 -> options.ScanID )

5. 업로드할 스캔의 이름을 지정한다.
( ex. nuclei -sname "Weekly Scan" -> options.ScanName )

6. 스캔 결과를 ProjectDiscovery Cloud(PDCP) Dashboard에 업로드하고 조회한다.
( ex. nuclei -pd -> options.EnableCloudUpload = true )

7. JSONL 결과 파일을 ProjectDiscovery Cloud(PDCP) Dashboard에 업로드한다.
( ex. nuclei -pdu result.jsonl -> options.ScanUploadFile )

[CLI 옵션 등록]
-auth
-tid
-cup
-sid
-sname
-pd
-pdu

[입력된 값을 변수 또는 options 구조체와 연결(바인딩) 한다.]
-auth → pdcpauth
-tid → options.TeamID
-cup → options.EnableCloudUpload
-sid → options.ScanID
-sname → options.ScanName
-pd → options.EnableCloudUpload
-pdu → options.ScanUploadFile

```
	flagSet.CreateGroup("Authentication", "Authentication",
		flagSet.StringSliceVarP(&options.SecretsFile, "secret-file", "sf", nil, "path to config file containing secrets for nuclei authenticated scan", goflags.CommaSeparatedStringSliceOptions),
		flagSet.BoolVarP(&options.PreFetchSecrets, "prefetch-secrets", "ps", false, "prefetch secrets from the secrets file"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]
Authentication이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Authentication 그룹을 생성하고, 인증(Authentication) 및 Secret 관리와 관련된 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.StringSliceVarP(...)에서 -secret-file 옵션을 등록하고, FlagData 반환.
flagSet.BoolVarP(...)에서 -prefetch-secrets 옵션을 등록하고, FlagData 반환.

아래 옵션들도 모두 등록됨.
-secret-file
-prefetch-secrets

CreateGroup 호출 후, 모두 Authentication 그룹으로 지정.

[의미]
이해를 돕고자 예를 들어 설명했습니다.

1.  인증 정보(Secret)가 저장된 설정 파일을 지정한다.
( ex. nuclei -sf secrets.yaml -> options.SecretsFile )

2. 스캔 시작 전에 Secret 파일의 인증 정보를 미리 읽어온다.
( ex. nuclei -ps -> options.PreFetchSecrets = true )

[CLI 옵션 등록]
-sf
-ps

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
-sf → options.SecretsFile
-ps → options.PreFetchSecrets

```
flagSet.SetCustomHelpText(`EXAMPLES:
Run nuclei on single host:
	$ nuclei -target example.com

Run nuclei with specific template directories:
	$ nuclei -target example.com -t http/cves/ -t ssl

Run nuclei against a list of hosts:
	$ nuclei -list hosts.txt

Run nuclei with a JSON output:
	$ nuclei -target example.com -json-export output.json

Run nuclei with sorted Markdown outputs (with environment variables):
	$ MARKDOWN_EXPORT_SORT_MODE=template nuclei -target example.com -markdown-export nuclei_report/

Additional documentation is available at: https://docs.nuclei.sh/getting-started/running
	`)
```

#### FlagSet.SetCustomHelpText 해석

이 부분은 지금까지의 `CreateGroup()`과는 조금 다르다.

`SetCustomHelpText()`는 **새로운 CLI 옵션을 등록하는 함수가 아니라**, 사용자가 `-h`, `--help` 등을 입력했을 때 **도움말(Help) 화면의 예제(EXAMPLES) 부분에 출력할 텍스트를 등록하는 함수**이다.

[`SetDescription()`과의 차이]
SetDescription 함수는 프로그램의 간단한 설명(소개)을 등록.
SetcustomHelpText 함수는 Help 화면에 출력할 예제나 추가 안내를 등록.

즉,

nuclei -h  

   ↓

프로그램 설명 (SetDescription)  
  
   ↓  
  
옵션 목록 (CreateGroup으로 등록한 옵션들)  
   
   ↓  
   
용 예시 (SetCustomHelpText)

[흐름]
사용자가 -h 또는 --help 옵션을 입력했을 때 출력할 사용자 정의(Custom) Help 메시지를 등록한다.

SetCustomHelpText()를 호출하면,
여러 개의 사용 예시(EXAMPLES)와 추가 문서 링크를 FlagSet 내부에 저장한다.

실제로는 바로 출력되지 않으며,
사용자가 Help를 요청할 때 Help 화면의 마지막 부분에 함께 출력된다.

[의미]
이해를 돕고자 예를 들어 설명했습니다.

1. 단일 Host를 대상으로 스캔하는 방법을 보여준다.
( ex. nuclei -target example.com )

2. 특정 Template 디렉터리를 지정하여 스캔하는 방법을 보여준다.
( ex. nuclei -target example.com -t http/cves/ -t ssl )

3. Host 목록이 저장된 파일을 이용하여 스캔하는 방법을 보여준다.
( ex. nuclei -list hosts.txt )

4. 결과를 JSON 파일로 저장하는 방법을 보여준다.
( ex. nuclei -target example.com -json-export output.json )

5. Markdown 형식으로 결과를 정렬하여 저장하는 방법을 보여준다.
( ex. MARKDOWN_EXPORT_SORT_MODE=template nuclei -target example.com -markdown-export nuclei_report/ )

추가 사용법은 공식 문서를 참고하도록 안내한다.
( https://docs.nuclei.sh/getting-started/running )

[CLI 옵션 등록]
없음

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
없음

### 26.07.06 - 전지성
 
#### readConfig() 후처리 구간(531~790줄)
여기부터는 CLI 옵션을 등록하는 단계가 아니라,  
`flagSet.Parse()` 이후 사용자가 입력한 값을 실제 실행 설정에 반영하는 단계이다.  
 
주요 작업:  
- CLI 옵션 파싱  
- Deprecated 옵션 보정  
- Cloud / PDCP 인증 처리  
- Logger, Timeout, Update 설정 반영  
- Config 파일 병합  
- Template Profile 경로 처리

```
### 1. Migration 비활성화 & CLI 파싱(540~535줄)


// nuclei has multiple migrations  
// nuclei는 여러 설정 파일의 위치가 변경(Migration)된 적이 있다.  
  
// ex: resume.cfg moved to platform standard cache dir from config dir  
// 예: resume.cfg가 기존 config 디렉터리에서 운영체제 표준 cache 디렉터리로 이동되었다.  
  
// ex: config.yaml moved to platform standard config dir from linux specific config dir  
// config.yaml이 Linux 전용 config 디렉터리에서 운영체제 표준 config 디렉터리로 이동되었다.  
  
// and hence it will be attempted in config package during init  
// 따라서 config 패키지 초기화 과정에서 자동 Migration을 시도한다.  
goflags.DisableAutoConfigMigration = true  // goflags의 자동 config migration 기능을 비활성화한다.  
  
_ = flagSet.Parse() // 지금까지 등록한 CLI 옵션을 실제 사용자 입력값으로 파싱한다.  
```

#### 해석(540~535줄)
`flagSet.Parse()`가 실행된 뒤에야 `-u`, `-t`, `-jsonl`, `-config` 같은 CLI 옵션 값이 `options` 구조체나 임시 변수에 저장된다.

```
### 2. Deprecated 옵션 보정 (547~551줄)
  
 
// when fuzz flag is enabled, set the dast flag to true  
// -fuzz 옵션이 활성화되면 -dast 옵션도 true로 설정한다.
  
if fuzzFlag {  
	// backwards compatibility for fuzz flag  
	// 예전 -fuzz 옵션과의 하위 호환성을 위한 처리이다.  
	
	options.DAST = true  
} // [if fuzzFlag 종료]  
```

#### 의미
`-fuzz`는 예전 옵션이고, 현재는 `-dast` 사용을 권장한다.  
그래서 사용자가 `-fuzz`를 입력해도 내부적으로 `options.DAST = true`로 바꿔준다.

```
### 3. Code Template 옵션 후처리(553~556줄)

// All cloud-based templates depend on both code and self-contained templates.  
// Cloud 기반 템플릿은 code 템플릿과 self-contained 템플릿 기능이 모두 필요하다.  

if options.EnableCodeTemplates {  
	options.EnableSelfContainedTemplates = true  
} // [if options.EnableCodeTemplates 종료]  
```

#### 해석
`-code` 옵션이 켜지면 `self-contained` 템플릿도 자동으로 활성화한다.

``` 
### 4. PDCP Cloud 인증 처리(558~569줄)

// api key hierarchy: cli flag > env var > .pdcp/credential file  
// API Key 우선순위: CLI 옵션(-auth) > 환경변수 > .pdcp/credential 파일  

if pdcpauth == "true" {  
// -auth만 입력된 경우, PDCP 인증 절차를 실행한다.  
	runner.AuthWithPDCP()
	  
} else if len(pdcpauth) == 36 {  
// -auth 뒤에 36자리 API Key가 입력된 경우이다.  
	ph := pdcp.PDCPCredHandler{} // PDCP Credential 관리 객체 생성  
  
	if _, err := ph.GetCreds(); err == pdcp.ErrNoCreds {  
	// 기존에 저장된 Credential이 없는 경우에만 API Key 검증을 진행한다.  
		apiServer := env.GetEnvOrDefault("PDCP_API_SERVER", pdcp.DefaultApiServer)  
  
		if validatedCreds, err := ph.ValidateAPIKey(pdcpauth, apiServer, config.BinaryName); err == nil {  
		// API Key 검증 성공 시 Credential을 저장한다.  
			
			_ = ph.SaveCreds(validatedCreds)  
			
		} // [if ValidateAPIKey 성공 종료]  
	} // [if ph.GetCreds() == ErrNoCreds 종료]  
} // [if pdcpauth / else if len(pdcpauth) 종료]  
```

#### 해석
`-auth`만 입력하면 로그인 절차를 실행하고,  
`-auth <36자리 API Key>` 형태이면 API Key를 검증한 뒤 Credential 파일에 저장한다.

```
### 5. AI Template 사용 전 인증 확인(571~578줄)

// guard cloud services with credentials  
// Cloud 기능을 사용하기 전에 인증 정보가 있는지 확인한다.  

if options.AITemplatePrompt != "" {  
	h := &pdcp.PDCPCredHandler{} // PDCP Credential 관리 객체 생성  
	_, err := h.GetCreds() // 저장된 Credential 확인  
  
	if err != nil {  
	// 인증 정보가 없으면 -ai 기능을 사용할 수 없으므로 프로그램을 종료한다.  
	
		options.Logger.Fatal().Msg("To utilize the `-ai` flag, please configure your API key with the `-auth` flag or set the                 `PDCP_API_KEY` environment variable")  
	} // [if err != nil 종료]  
} // [if options.AITemplatePrompt != "" 종료]  
```

#### 주의
`-ai` 옵션은 PDCP Cloud 인증이 필요하다.  
인증 정보가 없으면 `Fatal()`로 프로그램이 종료된다.

```
### 6. Logger, Verbose, Timeout, Update 설정(580~597줄)

options.Logger.SetTimestamp(options.Timestamp, levels.LevelDebug) // -timestamp 옵션 값을 Logger에 반영한다.  
  
if options.VerboseVerbose {  
	// hide release notes if silent mode is enabled  
	// -vv 옵션이 있으면 release notes 출력을 허용한다.  

	installer.HideReleaseNotes = false  
} // [if options.VerboseVerbose 종료]  
  
if options.Timeout > 30 {  
	// default github binary/template download timeout is 30 sec  
	// 기본 GitHub binary/template 다운로드 timeout은 30초이다.  
	
	updateutils.DownloadUpdateTimeout = time.Duration(options.Timeout) * time.Second  
} // [if options.Timeout > 30 종료]  
  
if updateNucleiBinary {  
	// -update 옵션이 입력된 경우 nuclei 엔진 업데이트를 실행한다.  
	runner.NucleiToolUpdateCallback()  
} // [if updateNucleiBinary 종료]  
  
if options.LeaveDefaultPorts {  
	// -leave-default-ports 옵션이 입력된 경우 기본 포트(:80, :443)를 유지한다.  
	http.LeaveDefaultPorts = true  
} // [if options.LeaveDefaultPorts 종료]  
```

#### 해석
이 구간은 CLI 옵션으로 받은 값을 Logger, Update, HTTP 설정 같은 전역 실행 환경에 반영한다.

```
### 7. 환경변수 기반 Config 디렉터리 처리(598~601줄)

if customConfigDir := os.Getenv(config.NucleiConfigDirEnv); customConfigDir != "" {  
// Nuclei 설정 디렉터리 환경변수가 존재하는 경우이다.  

	config.DefaultConfig.SetConfigDir(customConfigDir)  
	readFlagsConfig(flagSet)  
} // [if customConfigDir != "" 종료]  
```

#### 해석
환경변수로 Nuclei 설정 디렉터리가 지정되어 있으면 기본 config 디렉터리를 그 값으로 바꾼다.

```
### 8. Config 파일 병합(603~646줄)

if cfgFile != "" {  
// -config 옵션으로 설정 파일 경로가 입력된 경우이다.  
  
	if !fileutil.FileExists(cfgFile) {  
	// 설정 파일이 존재하지 않으면 프로그램을 종료한다.  
		options.Logger.Fatal().Msgf("given config file '%s' does not exist", cfgFile)  
	} // [if !fileutil.FileExists(cfgFile) 종료]  
  
	// merge config file with flags  
	// 설정 파일 값을 현재 flagSet에 병합한다.
  
	if err := flagSet.MergeConfigFile(cfgFile); err != nil {  
		options.Logger.Fatal().Msgf("Could not read config: %s\n", err)  
	} // [if MergeConfigFile 에러 종료]  
  
	if !options.Vars.IsEmpty() {  
		// options.Vars에 값이 있는 경우이다.  
  
		// Maybe we should add vars to the config file as well even if they are set via flags?  
		// flag로 설정된 var 값도 config 파일에 반영해야 하는지에 대한 개발자 메모이다.  
	
		file, err := os.Open(cfgFile)  
  
		if err != nil {  
			gologger.Fatal().Msgf("Could not open config file: %s\n", err)  
		} // [if os.Open 에러 종료]  
  
		defer func() {  
			_ = file.Close()  
		}() // [defer func 종료]  
  
		data := make(map[string]interface{})  
		err = yaml.NewDecoder(file).Decode(&data)  
  
		if err != nil {  
			gologger.Fatal().Msgf("Could not decode config file: %s\n", err)  
		} // [if yaml Decode 에러 종료]  
  
		variables := data["var"]  
  
		if variables != nil {  
			if varSlice, ok := variables.([]interface{}); ok {  
				for _, value := range varSlice {  
					if strVal, ok := value.(string); ok {  
						err = options.Vars.Set(strVal)  
  
						if err != nil {  
							gologger.Warning().Msgf("Could not set variable from config file: %s\n", err)  
						} // [if options.Vars.Set 에러 종료]  
					} else {  
						gologger.Warning().Msgf("Skipping non-string variable in config: %#v", value)  
					} // [if strVal / else 종료]  
				} // [for varSlice 순회 종료]  
			} else {  
				gologger.Warning().Msgf("No 'var' section found in config file: %s", cfgFile)  
			} // [if varSlice 타입 변환 / else 종료]  
		} // [if variables != nil 종료]  
	} // [if !options.Vars.IsEmpty() 종료]  
} // [if cfgFile != "" 종료]  
```

#### 해석
`-config config.yaml`이 입력되면 설정 파일을 확인하고, `MergeConfigFile()`로 CLI 옵션 설정과 병합한다.  
추가로 YAML 내부의 `var` 항목을 읽어 `options.Vars`에 반영한다.

```
### 9. Template 디렉터리 설정(648~654줄)

templatesDir := options.NewTemplatesDirectory // -update-template-dir 옵션 값  
  
if templatesDir == "" {  
// CLI 옵션으로 템플릿 디렉터리가 지정되지 않은 경우 환경변수에서 가져온다.  

	templatesDir = os.Getenv(config.NucleiTemplatesDirEnv)  
} // [if templatesDir == "" 종료]  
  
if templatesDir != "" {  
// 템플릿 디렉터리가 지정되어 있으면 기본 템플릿 디렉터리로 설정한다. 
 
	config.DefaultConfig.SetTemplatesDir(templatesDir)  
} // [if templatesDir != "" 종료]  
```

#### 해석
템플릿 디렉터리는 먼저 CLI 옵션 `-update-template-dir`에서 찾고, 없으면 환경변수에서 찾는다.

```
### 10. Template Profile 경로 처리 및 병합(656~681줄)

defaultProfilesPath := filepath.Join(config.DefaultConfig.GetTemplateDir(), "profiles") // 기본 profile 디렉터리 경로  
  
if templateProfile != "" {  
// -profile 또는 -tp 옵션이 입력된 경우이다.  
  
	if filepath.Ext(templateProfile) == "" {  
	// 확장자가 없으면 profile 파일 경로가 아니라 profile ID로 판단한다.  
		if tp := findProfilePathById(templateProfile, defaultProfilesPath); tp != "" {  
			templateProfile = tp  
		} else {  
			options.Logger.Fatal().Msgf("'%s' is not a profile-id or profile path", templateProfile)  
		} // [if findProfilePathById / else 종료]  
	} // [if filepath.Ext(templateProfile) == "" 종료]  
  
	if !filepath.IsAbs(templateProfile) {  
	// templateProfile이 절대 경로가 아닌 경우이다.  
  
		if filepath.Dir(templateProfile) == "profiles" {  
			defaultProfilesPath = filepath.Join(config.DefaultConfig.GetTemplateDir())  
		} // [if filepath.Dir(templateProfile) == "profiles" 종료]  
  
		currentDir, err := os.Getwd()  
  
		if err == nil && fileutil.FileExists(filepath.Join(currentDir, templateProfile)) {  
			templateProfile = filepath.Join(currentDir, templateProfile)  
		} else {  
			templateProfile = filepath.Join(defaultProfilesPath, templateProfile)  
		} // [if 현재 디렉터리에 profile 존재 / else 종료]  
	} // [if !filepath.IsAbs(templateProfile) 종료]  
  
	if !fileutil.FileExists(templateProfile) {  
		options.Logger.Fatal().Msgf("given template profile file '%s' does not exist", templateProfile)  
	} // [if !fileutil.FileExists(templateProfile) 종료]  
  
	if err := flagSet.MergeConfigFile(templateProfile); err != nil {  
		options.Logger.Fatal().Msgf("Could not read template profile: %s\n", err)  
	} // [if MergeConfigFile(templateProfile) 에러 종료]  
} // [if templateProfile != "" 종료]  
```

#### 해석
`-profile` 또는 `-tp` 옵션이 입력되면 profile ID인지 파일 경로인지 판단한다.  
최종 profile 파일을 찾은 뒤, `MergeConfigFile()`로 현재 설정에 병합한다.

```
11. Inline Target 처리(683~705줄)
    
// Process inline target list from profile. 
// Profile 안에 직접 작성된 Target 목록을 처리한다.

 // Supports both the dedicated targets-inline key and multiline 
 // targets-inline 전용 키와 여러 줄 입력을 모두 지원한다. 
 
 // content in the list key (which normally holds a file path). 
 // 원래 파일 경로를 저장하는 list 키에 여러 줄 Target이 들어온 경우도 처리한다. 
 
if options.InlineTargetsList != "" { 
	inlineTargets := strings.Split(strings.TrimSpace(options.InlineTargetsList), "\n") 
	for _, target := range inlineTargets { 
		target = strings.TrimSpace(target) 
		if target != "" && !strings.HasPrefix(target, "#") { 
			options.Targets = append(options.Targets, target) 
		} // [if target 유효성 검사 종료] } 	
	// [for inlineTargets 순회 종료] } 
// [if options.InlineTargetsList != "" 종료] 
	
if strings.Contains(options.TargetsFilePath, "\n") { 
// list key has multiline content, treat as inline targets 
// list 키에 여러 줄 내용이 있으면 파일 경로가 아니라 inline target으로 처리한다. 
	inlineTargets := strings.Split(strings.TrimSpace(options.TargetsFilePath), "\n") 

	for _, target := range inlineTargets { 
		target = strings.TrimSpace(target) 
		if target != "" && !strings.HasPrefix(target, "#") { 
			options.Targets = append(options.Targets, target) 
		} // [if target 유효성 검사 종료] 
	} // [for inlineTargets 순회 종료] 
	options.TargetsFilePath = ""
```

#### 해석
Profile 파일 안에서 Target을 파일 경로가 아니라 여러 줄 문자열로 작성한 경우,  
줄 단위로 나누어 `options.Targets`에 추가한다.

빈 줄과 `#`으로 시작하는 주석 줄은 제외한다.

```
### 12. Inline Secrets 처리(707~714줄)

// Process inline secrets from profile YAML
// Profile YAML 안에 직접 작성된 Secret 정보를 처리한다.
tempSecretsFile, err := processInlineSecretsFromProfile(templateProfile, options)

if err != nil {
	options.Logger.Fatal().Msgf("Could not process inline secrets: %s\n", err)
} // [if processInlineSecretsFromProfile 에러 종료]

if tempSecretsFile != "" {
	inlineSecretsTempFiles = append(inlineSecretsTempFiles, tempSecretsFile)
} // [if tempSecretsFile != "" 종료]
```

#### 해석
Profile YAML 안에 Secret 정보가 직접 들어있는 경우,  
`processInlineSecretsFromProfile()` 함수가 이를 처리해서 임시 Secret 파일을 만든다.

임시 파일이 생성되면 `inlineSecretsTempFiles` 목록에 저장해두고, 나중에 정리할 수 있게 한다.

---
```
### 13. Template Profile 처리 블록 종료(715줄)

} // [if templateProfile != "" 종료]
```

#### 해석
여기까지가 `-profile` 또는 `-tp` 옵션이 입력되었을 때 실행되는 전체 처리 구간이다.

이 블록 안에서는 다음 작업을 수행한다.

1. Profile ID 또는 파일 경로 확인
    
2. Profile 파일 존재 여부 검사
    
3. Profile 설정 병합
    
4. Inline Target 처리
    
5. Inline Secrets 처리
    

---

```
### 14. Secrets 파일 존재 여부 검사(717~723줄)

if len(options.SecretsFile) > 0 {

	for _, secretFile := range options.SecretsFile {

		if !fileutil.FileExists(secretFile) {
			options.Logger.Fatal().Msgf("given secrets file '%s' does not exist", secretFile)
		} // [if !fileutil.FileExists(secretFile) 종료]

	} // [for options.SecretsFile 순회 종료]

} // [if len(options.SecretsFile) > 0 종료]
```

#### 해석
`-secret-file` 또는 `-sf` 옵션으로 Secret 파일이 지정된 경우,  
각 Secret 파일이 실제로 존재하는지 검사한다.

파일이 없으면 `Fatal()`을 호출하여 프로그램을 종료한다.

---

```
### 15. 오래된 Resume 파일 정리 후 flagSet 반환(725~727줄)

cleanupOldResumeFiles()

return flagSet

} // [readConfig 함수 종료]
```

#### 해석
`cleanupOldResumeFiles()`를 호출해 오래된 resume 파일을 정리한 뒤,  
설정이 완료된 `flagSet`을 반환한다.

즉, `readConfig()`의 최종 결과는 CLI 옵션과 설정 파일이 반영된 `FlagSet`이다.

---

## cleanupOldResumeFiles #전지성
```go
func cleanupOldResumeFiles()
```
> Cache 디렉터리에서 10일보다 오래된 Resume 파일을 정리하는 함수
- **매개변수 :** 없음
- **반환 타입 :** 없음
- **설명 :** `resume-` 접두사를 가진 파일 중 생성 후 10일이 지난 파일을 삭제
- **참조 :** [readConfig](main_Functions.md)

### 상세 분석

```
### 16. 오래된 Resume 파일 삭제(729줄~737줄)

// cleanupOldResumeFiles cleans up resume files older than 10 days.
// cleanupOldResumeFiles는 10일보다 오래된 resume 파일을 정리한다.
func cleanupOldResumeFiles() {

	root := config.DefaultConfig.GetCacheDir()

	filter := fileutil.FileFilters{
		OlderThan: 24 * time.Hour * 10, // cleanup on the 10th day
		Prefix:    "resume-",
	}

	_ = fileutil.DeleteFilesOlderThan(root, filter)

} // [cleanupOldResumeFiles 함수 종료]
```

#### 해석
Nuclei의 cache 디렉터리에서 `resume-`으로 시작하고,  
10일보다 오래된 파일을 삭제한다.

`resume` 파일은 중단된 스캔을 이어서 실행할 때 사용되는 파일이다.

---
## readFlagsConfig #전지성
```go
func readFlagsConfig(flagset *goflags.FlagSet)
```
> 기본 Config 파일을 확인하고 현재 Config 위치에 복사하거나 FlagSet에 병합하는 함수
- **매개변수 :** `flagset *goflags.FlagSet`
- **반환 타입 :** 없음
- **설명 :** Config 파일의 존재 여부를 검사하고 `CopyFile()` 또는 `MergeConfigFile()`을 실행
- **참조 :** [goflags](../03_External_Packages/goflags.md)

### 상세 분석

```
### 17. 기본 Config 파일 확인(739줄~748줄)

// readFlagsConfig reads the config file from the default config dir and copies it to the current config dir.
// readFlagsConfig는 기본 config 디렉터리에서 설정 파일을 읽고, 현재 config 디렉터리로 복사하거나 병합한다.
func readFlagsConfig(flagset *goflags.FlagSet) {

	// check if config.yaml file exists
	// config.yaml 파일이 존재하는지 확인한다.
	defaultCfgFile, err := flagset.GetConfigFilePath()

	if err != nil {

		// something went wrong either dir is not readable or something else went wrong upstream in `goflags`
		// 디렉터리를 읽을 수 없거나 goflags 내부에서 문제가 발생한 경우이다.

		// warn and exit in this case
		// 이 경우 경고를 출력하고 함수를 종료한다.
		options.Logger.Warning().Msgf("Could not read config file: %s\n", err)

		return

	} // [if err != nil 종료]
```

#### 해석
`flagset.GetConfigFilePath()`를 통해 기본 설정 파일 경로를 가져온다.  
실패하면 경고 로그만 출력하고 함수 실행을 중단한다.

---

```
### 18. 현재 Config 파일이 없을 때 복사(749~761줄)

	cfgFile := config.DefaultConfig.GetFlagsConfigFilePath()

	if !fileutil.FileExists(cfgFile) {

		if !fileutil.FileExists(defaultCfgFile) {

			// if default config does not exist, warn and exit
			// 기본 config 파일도 없으면 경고를 출력하고 종료한다.
			options.Logger.Warning().Msgf("missing default config file : %s", defaultCfgFile)

			return

		} // [if !fileutil.FileExists(defaultCfgFile) 종료]

		// if does not exist copy it from the default config
		// 현재 config 파일이 없으면 기본 config 파일에서 복사한다.
		if err = fileutil.CopyFile(defaultCfgFile, cfgFile); err != nil {
			options.Logger.Warning().Msgf("Could not copy config file: %s\n", err)
		} // [if fileutil.CopyFile 에러 종료]

		return

	} // [if !fileutil.FileExists(cfgFile) 종료]
```

#### 해석
현재 config 파일이 없으면 기본 config 파일을 현재 config 위치로 복사한다.

기본 config 파일도 없으면 경고를 출력하고 종료한다.

---

```
### 19. Config 파일이 있으면 병합(762~766줄)

	// if config file exists, merge it with the default config
	// config 파일이 존재하면 현재 flag 설정과 병합한다.
	
	if err = flagset.MergeConfigFile(cfgFile); err != nil {
		options.Logger.Warning().Msgf("failed to merge configfile with flags got: %s\n", err)
	} // [if MergeConfigFile 에러 종료]

} // [readFlagsConfig 함수 종료]
```

#### 해석
현재 config 파일이 이미 존재하면 `MergeConfigFile()`을 사용해  
config 파일의 설정을 flag 설정과 병합한다.

---

## disableUpdatesCallback #전지성
```go
func disableUpdatesCallback()
```
> Nuclei의 자동 Update 확인 기능을 비활성화하는 Callback 함수
- **매개변수 :** 없음
- **반환 타입 :** 없음
- **설명 :** `config.DefaultConfig.DisableUpdateCheck()`를 호출
- **참조 :** [readConfig](main_Functions.md)

### 상세 분석

```
### 20. 자동 업데이트 확인 비활성화(768~771줄)

// disableUpdatesCallback disables the update check.
// disableUpdatesCallback은 자동 업데이트 확인 기능을 비활성화한다.
func disableUpdatesCallback() {

	config.DefaultConfig.DisableUpdateCheck()

} // [disableUpdatesCallback 함수 종료]
```

#### 해석
`-disable-update-check` 옵션이 입력되면 실행되는 콜백 함수이다.

내부적으로 `config.DefaultConfig.DisableUpdateCheck()`를 호출하여  
자동 업데이트 확인 기능을 끈다.


---
## printVersion #전지성
```go
func printVersion()
```
> Nuclei 버전과 주요 디렉터리 정보를 출력하고 프로그램을 종료하는 함수
- **매개변수 :** 없음
- **반환 타입 :** 없음
- **설명 :** Engine, Config, Cache, PDCP 정보를 출력한 뒤 `os.Exit(0)` 호출
- **참조 :** [readConfig](main_Functions.md)

### 상세 분석

```
### 21. Nuclei 버전 정보 출력(773~780줄)

// printVersion prints the nuclei version and exits.
// printVersion은 Nuclei 버전 정보를 출력하고 프로그램을 종료한다.
func printVersion() {

	options.Logger.Info().Msgf("Nuclei Engine Version: %s", config.Version)

	options.Logger.Info().Msgf("Nuclei Config Directory: %s", config.DefaultConfig.GetConfigDir())

	options.Logger.Info().Msgf("Nuclei Cache Directory: %s", config.DefaultConfig.GetCacheDir()) // cache dir contains resume files

	options.Logger.Info().Msgf("PDCP Directory: %s", pdcp.PDCPDir)

	os.Exit(0)

} // [printVersion 함수 종료]
```

#### 해석
`-version` 옵션이 입력되면 실행되는 콜백 함수이다.

Nuclei 엔진 버전, Config 디렉터리, Cache 디렉터리, PDCP 디렉터리를 출력한 뒤  
`os.Exit(0)`으로 프로그램을 정상 종료한다.

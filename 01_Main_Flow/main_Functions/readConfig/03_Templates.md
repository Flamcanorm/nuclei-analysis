---
상태: false
유형: Main_Function
상세: Templates 옵션 그룹
---
# readConfig - Templates #전지성

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



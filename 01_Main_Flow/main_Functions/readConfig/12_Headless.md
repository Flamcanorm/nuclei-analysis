---
상태: false
유형: Main_Function
상세: Headless 옵션 그룹
---
# readConfig - Headless #전지성

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



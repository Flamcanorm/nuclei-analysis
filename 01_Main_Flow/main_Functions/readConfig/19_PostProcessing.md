---
상태: false
유형: Main_Function
상세: readConfig 옵션 파싱 후처리
---
# readConfig - 옵션 파싱 후처리 #전지성

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


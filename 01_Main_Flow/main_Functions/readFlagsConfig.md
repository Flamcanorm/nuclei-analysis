---
상태: false
유형: Main_Function
상세: main.go readFlagsConfig 함수
---
# readFlagsConfig #전지성

## readFlagsConfig #전지성
```go
func readFlagsConfig(flagset *goflags.FlagSet)
```
> 기본 Config 파일을 확인하고 현재 Config 위치에 복사하거나 FlagSet에 병합하는 함수
- **매개변수 :** `flagset *goflags.FlagSet`
- **반환 타입 :** 없음
- **설명 :** Config 파일의 존재 여부를 검사하고 `CopyFile()` 또는 `MergeConfigFile()`을 실행
- **참조 :** [goflags](../../03_External_Packages/goflags.md)

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



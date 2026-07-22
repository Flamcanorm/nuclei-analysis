---
상태: false
유형: Main_Function
상세: main.go printVersion 함수
---
# printVersion #전지성

## printVersion #전지성
```go
func printVersion()
```
> Nuclei 버전과 주요 디렉터리 정보를 출력하고 프로그램을 종료하는 함수
- **매개변수 :** 없음
- **반환 타입 :** 없음
- **설명 :** Engine, Config, Cache, PDCP 정보를 출력한 뒤 `os.Exit(0)` 호출
- **참조 :** [readConfig](readConfig/README.md)

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


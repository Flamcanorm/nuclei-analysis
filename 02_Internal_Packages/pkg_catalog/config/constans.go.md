# constans.go
#pkg/config
> [constans.go](https://github.com/projectdiscovery/nuclei/blob/dev/pkg/catalog/config/constants.go)
> Nuclei가 실행될 때 필요한 버전 정보, 환경변수 이름, 파일/폴더 경로 등을 정의해놓은 코드
## var
#pkg/config/var
### CurrentAppMode
```go
// Global Var to control behaviours specific to cli or library
	// maybe this should be moved to utils ??
	// this is overwritten in cmd/nuclei/main.go
	CurrentAppMode = AppModeLibrary
```
> Nuclei의 실행 모드 기본 값은 라이브러리 모드 (사용자가 터미널에서 실행하면 `cmd/nuclei/main.go`에서 이 값을 `AppModeCLI`로 덮어씀 [L67](../../../01_Main_Flow/main.md#L67))

### const
```go
const (
	AppModeLibrary AppMode = "library"
	AppModeCLI     AppMode = "cli"
)
```
> Nuclei를 실행할 때 `library` 형태로 사용할지 아니면 `cli` 로 사용할지 모드 정의
- **참조 :** [AppMode](constans.go.md#AppMode) 

```go
const (
	TemplateConfigFileName          = ".templates-config.json"
	NucleiTemplatesDirName          = "nuclei-templates"
	OfficialNucleiTemplatesRepoName = "nuclei-templates"
	NucleiIgnoreFileName            = ".nuclei-ignore"
	NucleiTemplatesIndexFileName    = ".templates-index" // contains index of official nuclei templates
	NucleiTemplatesCheckSumFileName = ".checksum"
	NewTemplateAdditionsFileName    = ".new-additions"
	CLIConfigFileName               = "config.yaml"
	ReportingConfigFilename         = "reporting-config.yaml"
	// Version is the current version of nuclei
	Version = `v3.11.0`
	// Directory Names of custom templates
	CustomS3TemplatesDirName     = "s3"
	CustomGitHubTemplatesDirName = "github"
	CustomAzureTemplatesDirName  = "azure"
	CustomGitLabTemplatesDirName = "gitlab"
	BinaryName                   = "nuclei"
	FallbackConfigFolderName     = ".nuclei-config"
	NucleiConfigDirEnv           = "NUCLEI_CONFIG_DIR"
	NucleiTemplatesDirEnv        = "NUCLEI_TEMPLATES_DIR"
)
```
> Nuclei가 동작하면서 참조하는 고정된 파일 이름, 폴더 경로, 버전, 시스템 환경변수를 정의


## type
#pkg/config/type
### AppMode
#박영현
```go
type AppMode string
```
> Nuclei 실행 모드를 구분하기 위한 타입
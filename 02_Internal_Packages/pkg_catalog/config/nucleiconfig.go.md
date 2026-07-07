# nucleiconfig.go
#pkg/config
> Nuclei 실행 설정 정보가 정의됨

## var
#pkg/config/var
### DefaultConfig
#박영현 
```go
// DefaultConfig is the default nuclei configuration
// all config values and default are centralized here
var DefaultConfig *Config
```
> nuclei 기본 설정값
## type
#pkg/config/type
### Config struct
#박영현 
```go
type Config struct {
	TemplatesDirectory string `json:"nuclei-templates-directory,omitempty"`

	// customtemplates exists in templates directory with the name of custom-templates provider
	// below custom paths are absolute paths to respective custom-templates directories
	CustomS3TemplatesDirectory     string `json:"custom-s3-templates-directory"`
	CustomGitHubTemplatesDirectory string `json:"custom-github-templates-directory"`
	CustomGitLabTemplatesDirectory string `json:"custom-gitlab-templates-directory"`
	CustomAzureTemplatesDirectory  string `json:"custom-azure-templates-directory"`

	TemplateVersion        string `json:"nuclei-templates-version,omitempty"`
	NucleiIgnoreHash       string `json:"nuclei-ignore-hash,omitempty"`
	LogAllEvents           bool   `json:"-"` // when enabled logs all events (more than verbose)
	HideTemplateSigWarning bool   `json:"-"` // when enabled disables template signature warning

	// LatestXXX are not meant to be used directly and is used as
	// local cache of nuclei version check endpoint
	// these fields are only update during nuclei version check
	// TODO: move these fields to a separate unexported struct as they are not meant to be used directly
	LatestNucleiVersion          string           `json:"nuclei-latest-version"`
	LatestNucleiTemplatesVersion string           `json:"nuclei-templates-latest-version"`
	LatestNucleiIgnoreHash       string           `json:"nuclei-latest-ignore-hash,omitempty"`
	Logger                       *gologger.Logger `json:"-"` // logger

	// internal / unexported fields
	disableUpdates bool     `json:"-"` // disable updates both version check and template updates
	homeDir        string   `json:"-"` //  User Home Directory
	configDir      string   `json:"-"` //  Nuclei Global Config Directory
	debugArgs      []string `json:"-"` // debug args

	m sync.Mutex
}
```
> Nuclei 설정 값을 저장하기 위한 구조체
> 탬플릿 관련 경로, 버전 및 검사 제외 항목, 최선 버전 캐시 검사, json 파일로 검사할 때 설정 구조체 태크, 고루틴을 위한 뮤텍스 락

## func
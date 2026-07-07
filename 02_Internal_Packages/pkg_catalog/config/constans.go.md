# constans.go

## var



### CurrentAppMode
```go
// Global Var to control behaviours specific to cli or library
	// maybe this should be moved to utils ??
	// this is overwritten in cmd/nuclei/main.go
	CurrentAppMode = AppModeLibrary
```


 
### const
```go
const (
	AppModeLibrary AppMode = "library"
	AppModeCLI     AppMode = "cli"
)
```

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
## type

### AppMode
#박영현
```go
type AppMode string
```

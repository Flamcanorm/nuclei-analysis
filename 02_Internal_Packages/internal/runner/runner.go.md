# runner.go
> [runner.go](https://github.com/projectdiscovery/nuclei/blob/dev/internal/runner/runner.go)


## var
#internal/runner/var

### HideAutoSaveMsg
#박영현
```go
// HideAutoSaveMsg is a global variable to hide the auto-save message
	HideAutoSaveMsg = false
```


### EnableCloudUpload
#박영현 
```go
// EnableCloudUpload is global variable to enable cloud upload
	EnableCloudUpload = false
)
```

## type
#internal/runner/type

### Runner struct
#박영현
```go
type Runner struct {
	output             output.Writer
	interactsh         *interactsh.Client
	options            *types.Options
	projectFile        *projectfile.ProjectFile
	catalog            catalog.Catalog
	progress           progress.Progress
	colorizer          *aurora.Aurora
	issuesClient       reporting.Client
	browser            *engine.Browser
	rateLimiter        *ratelimit.Limiter
	hostErrors         hosterrorscache.CacheInterface
	resumeCfg          *types.ResumeCfg
	pprofServer        *pprofutil.PprofServer
	pdcpUploadErrMsg   string
	inputProvider      provider.InputProvider
	fuzzFrequencyCache *frequency.Tracker
	httpStats          *outputstats.Tracker
	Logger             *gologger.Logger

	honeypotDetector *honeypotdetector.Detector

	//general purpose temporary directory
	tmpDir          string
	parser          parser.Parser
	httpApiEndpoint *httpapi.Server
	fuzzStats       *fuzzStats.Tracker
	dastServer      *server.DASTServer
}
```




## func
#internal/runner/func
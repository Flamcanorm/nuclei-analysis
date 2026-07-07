# options.go
> [options.go](https://github.com/projectdiscovery/nuclei/blob/dev/internal/runner/options.go)
> #internal/runner


## var
#internal/runner/var

### DefaultDumpTrafficOutputFolder
#박영현 
```go
const (
	// Default directory used to save protocols traffic
	DefaultDumpTrafficOutputFolder = "output"
)
```


### validateOptions
#박영현 
```go
var validateOptions = validator.New()
```


## type
#internal/runner/type


## func
#internal/runner/func

### ConfigureOptions
#박영현 `
```go
func ConfigureOptions() error {
	// with FileStringSliceOptions, FileNormalizedStringSliceOptions, FileCommaSeparatedStringSliceOptions
	// if file has the extension `.yaml` or `.json` we consider those as strings and not files to be read
	isFromFileFunc := func(s string) bool {
		return !config.IsTemplate(s)
	}
	goflags.FileNormalizedStringSliceOptions.IsFromFile = isFromFileFunc
	goflags.FileStringSliceOptions.IsFromFile = isFromFileFunc
	goflags.FileCommaSeparatedStringSliceOptions.IsFromFile = isFromFileFunc
	return nil
}
```

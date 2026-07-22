---
상태: false
유형: Main_Function
상세: Optimizations 옵션 그룹
---
# readConfig - Optimizations #전지성

```
	flagSet.CreateGroup("optimization", "Optimizations",
		flagSet.IntVar(&options.Timeout, "timeout", 10, "time to wait in seconds before timeout"),
		flagSet.IntVar(&options.Retries, "retries", 1, "number of times to retry a failed request"),
		flagSet.BoolVarP(&options.LeaveDefaultPorts, "leave-default-ports", "ldp", false, "leave default HTTP/HTTPS ports (eg. host:80,host:443)"),
		flagSet.IntVarP(&options.MaxHostError, "max-host-error", "mhe", 30, "max errors for a host before skipping from scan"),
		flagSet.StringSliceVarP(&options.TrackError, "track-error", "te", nil, "adds given error to max-host-error watchlist (standard, file)", goflags.FileStringSliceOptions),
		flagSet.BoolVarP(&options.NoHostErrors, "no-mhe", "nmhe", false, "disable skipping host from scan based on errors"),
		flagSet.BoolVar(&options.Project, "project", false, "use a project folder to avoid sending same request multiple times"),
		flagSet.StringVar(&options.ProjectPath, "project-path", os.TempDir(), "set a specific project path"),
		flagSet.BoolVarP(&options.StopAtFirstMatch, "stop-at-first-match", "spm", false, "stop processing HTTP requests after the first match (may break template/workflow logic)"),
		flagSet.BoolVar(&options.Stream, "stream", false, "stream mode - start elaborating without sorting the input"),
		flagSet.EnumVarP(&options.ScanStrategy, "scan-strategy", "ss", goflags.EnumVariable(0), "strategy to use while scanning(auto/host-spray/template-spray)", goflags.AllowdTypes{
			scanstrategy.Auto.String():          goflags.EnumVariable(0),
			scanstrategy.HostSpray.String():     goflags.EnumVariable(1),
			scanstrategy.TemplateSpray.String(): goflags.EnumVariable(2),
		}),
		flagSet.DurationVarP(&options.InputReadTimeout, "input-read-timeout", "irt", time.Duration(3*time.Minute), "timeout on input read"),
		flagSet.BoolVarP(&options.DisableHTTPProbe, "no-httpx", "nh", false, "disable httpx probing for non-url input"),
		flagSet.BoolVar(&options.PreflightPortScan, "preflight-portscan", false, "run preflight resolve + TCP portscan and filter targets before scanning (disabled by default)"),
		flagSet.BoolVar(&options.DisableStdin, "no-stdin", false, "disable stdin processing"),
	)
```

#### FlagSet.CreateGroup 해석
[흐름]  
optimization이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Optimizations 그룹을 생성하고, 스캔 성능 최적화, 재시도, 프로젝트 관리, 스캔 전략 등 실행 성능과 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.IntVar(...)에서 -timeout 옵션을 등록하고, FlagData 반환.  
flagSet.IntVar(...)에서 -retries 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-timeout  
-retries  
-leave-default-ports  
-max-host-error  
-track-error  
-no-mhe  
-project  
-project-path  
-stop-at-first-match  
-stream  
-scan-strategy  
-input-read-timeout  
-no-httpx  
-preflight-portscan  
-no-stdin  
  
CreateGroup 호출 후, 모두 optimization 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. 요청(Request)의 최대 대기 시간(초)을 지정한다.  
( ex. nuclei -timeout 30 -> options.Timeout = 30 )  
  
2. 실패한 요청(Request)의 재시도 횟수를 지정한다.  
( ex. nuclei -retries 3 -> options.Retries = 3 )  
  
3. 기본 HTTP/HTTPS 포트(80, 443)를 URL에 그대로 유지한다.  
( ex. nuclei -ldp -> options.LeaveDefaultPorts = true )  
  
4. 하나의 Host에서 허용할 최대 오류 횟수를 지정한다.  
( ex. nuclei -mhe 50 -> options.MaxHostError = 50 )  
  
5. 최대 오류(Max Host Error)에 포함할 오류 종류를 지정한다.  
( ex. nuclei -te timeout,connection-reset -> options.TrackError )  
  
6. 오류가 많아도 Host를 건너뛰지 않는다.  
( ex. nuclei -nmhe -> options.NoHostErrors = true )  
  
7. 동일한 요청을 여러 번 보내지 않도록 Project 기능을 활성화한다.  
( ex. nuclei -project -> options.Project = true )  
  
8. Project 데이터를 저장할 디렉터리를 지정한다.  
( ex. nuclei -project-path /tmp/nuclei -> options.ProjectPath )  
  
9. 첫 번째 취약점을 발견하면 해당 HTTP 요청 처리를 중단한다.  
( ex. nuclei -spm -> options.StopAtFirstMatch = true )  
  
10. 입력을 정렬하지 않고 바로 스캔을 시작하는 Stream 모드를 활성화한다.  
( ex. nuclei -stream -> options.Stream = true )  
  
11. 스캔 전략을 지정한다.  
(auto, host-spray, template-spray 중 선택)  
( ex. nuclei -ss host-spray -> options.ScanStrategy )  
  
12. 입력(Input)을 읽을 최대 대기 시간을 지정한다.  
( ex. nuclei -irt 5m -> options.InputReadTimeout )  
  
13. URL이 아닌 입력에 대해 httpx Probe를 수행하지 않는다.  
( ex. nuclei -nh -> options.DisableHTTPProbe = true )  
  
14. 스캔 전에 DNS Resolve 및 TCP Port Scan을 먼저 수행한다.  
( ex. nuclei -preflight-portscan -> options.PreflightPortScan = true )  
  
15. 표준 입력(stdin) 처리를 비활성화한다.  
( ex. nuclei -no-stdin -> options.DisableStdin = true )  
  
[CLI 옵션 등록]  
-timeout  
-retries  
-ldp  
-mhe  
-te  
-nmhe  
-project  
-project-path  
-spm  
-stream  
-ss  
-irt  
-nh  
-preflight-portscan  
-no-stdin  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-timeout → options.Timeout  
-retries → options.Retries  
-ldp → options.LeaveDefaultPorts  
-mhe → options.MaxHostError  
-te → options.TrackError  
-nmhe → options.NoHostErrors  
-project → options.Project  
-project-path → options.ProjectPath  
-spm → options.StopAtFirstMatch  
-stream → options.Stream  
-ss → options.ScanStrategy  
-irt → options.InputReadTimeout  
-nh → options.DisableHTTPProbe  
-preflight-portscan → options.PreflightPortScan  
-no-stdin → options.DisableStdin



---
상태: false
유형: Main_Function
상세: Debug 옵션 그룹
---
# readConfig - Debug #전지성

```
	flagSet.CreateGroup("debug", "Debug",
		flagSet.BoolVar(&options.Debug, "debug", false, "show all requests and responses"),
		flagSet.BoolVarP(&options.DebugRequests, "debug-req", "dreq", false, "show all sent requests"),
		flagSet.BoolVarP(&options.DebugResponse, "debug-resp", "dresp", false, "show all received responses"),
		flagSet.StringSliceVarP(&options.Proxy, "proxy", "p", nil, "list of http/socks5 proxy to use (comma separated or file input)",                                                      goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.BoolVarP(&options.ProxyInternal, "proxy-internal", "pi", false, "proxy all internal requests"),
		flagSet.BoolVarP(&options.ListDslSignatures, "list-dsl-function", "ldf", false, "list all supported DSL function signatures"),
		flagSet.StringVarP(&options.TraceLogFile, "trace-log", "tlog", "", "file to write sent requests trace log"),
		flagSet.StringVarP(&options.ErrorLogFile, "error-log", "elog", "", "file to write sent requests error log"),
		flagSet.CallbackVar(printVersion, "version", "show nuclei version"),
		flagSet.BoolVarP(&options.HangMonitor, "hang-monitor", "hm", false, "enable nuclei hang monitoring"),
		flagSet.BoolVarP(&options.Verbose, "verbose", "v", false, "show verbose output"),
		flagSet.StringVar(&memProfile, "profile-mem", "", "generate memory (heap) profile & trace files"),
		flagSet.BoolVar(&options.VerboseVerbose, "vv", false, "display templates loaded for scan"),
		flagSet.BoolVarP(&options.ShowVarDump, "show-var-dump", "svd", false, "show variables dump for debugging"),
		flagSet.IntVarP(&options.VarDumpLimit, "var-dump-limit", "vdl", 255, "limit the number of characters displayed in var dump"),
		flagSet.BoolVarP(&options.EnablePprof, "enable-pprof", "ep", false, "enable pprof debugging server"),
		flagSet.CallbackVarP(printTemplateVersion, "templates-version", "tv", "shows the version of the installed nuclei-templates"),
		flagSet.BoolVarP(&options.HealthCheck, "health-check", "hc", false, "run diagnostic check up"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]
debug라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Debug 그룹을 생성하고, 디버깅(Debug), 로그(Log), 프록시(Proxy), 프로파일링(Profiling), 버전 확인과 관련된 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.BoolVar(...)에서 -debug 옵션을 등록하고, FlagData 반환.
flagSet.BoolVarP(...)에서 -debug-req 옵션을 등록하고, FlagData 반환.

아래 옵션들도 모두 등록됨.
-debug
-debug-req
-debug-resp
-proxy
-proxy-internal
-list-dsl-function
-trace-log
-error-log
-version
-hang-monitor
-verbose
-profile-mem
-vv
-show-var-dump
-var-dump-limit
-enable-pprof
-templates-version
-health-check

CreateGroup 호출 후, 모두 debug 그룹으로 지정.

[의미]
1. 이해를 돕고자 예를 들어 설명했습니다.

2. 모든 요청(Request)과 응답(Response)을 출력한다.
( ex. nuclei -debug -> options.Debug = true )

3. 전송한 요청(Request)만 출력한다.
( ex. nuclei -dreq -> options.DebugRequests = true )

4. 수신한 응답(Response)만 출력한다.
( ex. nuclei -dresp -> options.DebugResponse = true )

5. HTTP/SOCKS5 Proxy를 지정한다.
( ex. nuclei -p http://127.0.0.1:8080 -> options.Proxy )

6. 내부 요청도 Proxy를 사용하도록 설정한다.
( ex. nuclei -pi -> options.ProxyInternal = true )

7. 사용 가능한 DSL 함수 목록을 출력한다.
( ex. nuclei -ldf -> options.ListDslSignatures = true )

8. 요청(Request) 추적 로그를 저장할 파일을 지정한다.
( ex. nuclei -tlog trace.log -> options.TraceLogFile )

9. 에러(Error) 로그를 저장할 파일을 지정한다.
( ex. nuclei -elog error.log -> options.ErrorLogFile )

10. 현재 Nuclei 버전을 출력한다.
( ex. nuclei -version -> printVersion() 실행 )

11. 프로그램 멈춤(Hang)을 감시한다.
( ex. nuclei -hm -> options.HangMonitor = true )

12. 상세(Verbose) 정보를 출력한다.
( ex. nuclei -v -> options.Verbose = true )

13. 메모리(Heap) 프로파일과 Trace 파일을 생성한다.
( ex. nuclei -profile-mem profile -> memProfile = "profile" )

14. 로드된 템플릿 목록까지 상세하게 출력한다.
( ex. nuclei -vv -> options.VerboseVerbose = true )

15. 디버깅을 위해 변수(Variable) 값을 출력한다.
( ex. nuclei -svd -> options.ShowVarDump = true )

16. 변수 출력 시 최대 문자 수를 지정한다.
( ex. nuclei -vdl 500 -> options.VarDumpLimit = 500 )

17. pprof 디버깅 서버를 활성화한다.
( ex. nuclei -ep -> options.EnablePprof = true )

18. 설치된 nuclei-templates의 버전을 출력한다.
( ex. nuclei -tv -> printTemplateVersion() 실행 )

19. 환경 진단(Health Check)을 수행한다.
( ex. nuclei -hc -> options.HealthCheck = true )

[CLI 옵션 등록]
-debug
-dreq
-dresp
-p
-pi
-ldf
-tlog
-elog
-version
-hm
-v
-profile-mem
-vv
-svd
-vdl
-ep
-tv
-hc

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
-debug → options.Debug
-dreq → options.DebugRequests
-dresp → options.DebugResponse
-p → options.Proxy
-pi → options.ProxyInternal
-ldf → options.ListDslSignatures
-tlog → options.TraceLogFile
-elog → options.ErrorLogFile
-version → printVersion()
-hm → options.HangMonitor
-v → options.Verbose
-profile-mem → memProfile
-vv → options.VerboseVerbose
-svd → options.ShowVarDump
-vdl → options.VarDumpLimit
-ep → options.EnablePprof
-tv → printTemplateVersion()
-hc → options.HealthCheck



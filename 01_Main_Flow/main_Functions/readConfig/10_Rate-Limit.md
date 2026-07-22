---
상태: false
유형: Main_Function
상세: Rate-Limit 옵션 그룹
---
# readConfig - Rate-Limit #전지성

```
	flagSet.CreateGroup("rate-limit", "Rate-Limit",
		flagSet.IntVarP(&options.RateLimit, "rate-limit", "rl", 150, "maximum number of requests to send per second"),
		flagSet.DurationVarP(&options.RateLimitDuration, "rate-limit-duration", "rld", time.Second, "maximum number of requests to send per second"),
		flagSet.BoolVar(&options.PerHostRateLimit, "per-host-rate-limit", false, "enable per-host rate limiting (global rate limit becomes unlimited when enabled)"),
		flagSet.IntVarP(&options.RateLimitMinute, "rate-limit-minute", "rlm", 0, "maximum number of requests to send per minute (DEPRECATED)"),
		flagSet.IntVarP(&options.BulkSize, "bulk-size", "bs", 25, "maximum number of hosts to be analyzed in parallel per template"),
		flagSet.IntVarP(&options.TemplateThreads, "concurrency", "c", 25, "maximum number of templates to be executed in parallel"),
		flagSet.IntVarP(&options.HeadlessBulkSize, "headless-bulk-size", "hbs", 10, "maximum number of headless hosts to be analyzed in parallel per template"),
		flagSet.IntVarP(&options.HeadlessTemplateThreads, "headless-concurrency", "headc", 10, "maximum number of headless templates to be executed in parallel"),
		flagSet.IntVarP(&options.JsConcurrency, "js-concurrency", "jsc", 120, "maximum number of javascript runtimes to be executed in parallel"),
		flagSet.IntVarP(&options.PayloadConcurrency, "payload-concurrency", "pc", 25, "max payload concurrency for each template"),
		flagSet.IntVarP(&options.ProbeConcurrency, "probe-concurrency", "prc", 50, "http probe concurrency with httpx"),
		flagSet.IntVarP(&options.TemplateLoadingConcurrency, "template-loading-concurrency", "tlc", types.DefaultTemplateLoadingConcurrency, "maximum number of concurrent template loading operations"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
rate-limit이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Rate-Limit 그룹을 생성하고, 요청 속도(Rate Limit), 동시 실행 수(Concurrency), 병렬 처리와 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.IntVarP(...)에서 -rate-limit 옵션을 등록하고, FlagData 반환.  
flagSet.DurationVarP(...)에서 -rate-limit-duration 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-rate-limit  
-rate-limit-duration  
-per-host-rate-limit  
-rate-limit-minute  
-bulk-size  
-concurrency  
-headless-bulk-size  
-headless-concurrency  
-js-concurrency  
-payload-concurrency  
-probe-concurrency  
-template-loading-concurrency  
  
CreateGroup 호출 후, 모두 rate-limit 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. 초당 최대 요청(Request) 수를 지정한다.  
( ex. nuclei -rl 200 -> options.RateLimit = 200 )  
  
2. 요청 속도를 제한하는 시간 단위를 지정한다.  
( ex. nuclei -rld 500ms -> options.RateLimitDuration )  
  
3. 호스트(Host)별 Rate Limit을 적용한다.  
활성화하면 전체(Global) Rate Limit은 제한 없이 동작한다.  
( ex. nuclei -per-host-rate-limit -> options.PerHostRateLimit = true )  
  
4. 분당 최대 요청 수를 지정한다. (Deprecated)  
( ex. nuclei -rlm 600 -> options.RateLimitMinute = 600 )  
  
5. 템플릿 하나당 동시에 분석할 Host 개수를 지정한다.  
( ex. nuclei -bs 50 -> options.BulkSize = 50 )  
  
6. 동시에 실행할 템플릿 개수를 지정한다.  
( ex. nuclei -c 50 -> options.TemplateThreads = 50 )  
  
7. Headless 템플릿에서 동시에 분석할 Host 개수를 지정한다.  
( ex. nuclei -hbs 20 -> options.HeadlessBulkSize = 20 )  
  
8. Headless 템플릿을 동시에 실행할 개수를 지정한다.  
( ex. nuclei -headc 20 -> options.HeadlessTemplateThreads = 20 )  
  
9. 동시에 실행할 JavaScript Runtime 개수를 지정한다.  
( ex. nuclei -jsc 200 -> options.JsConcurrency = 200 )  
  
10. 템플릿 하나에서 동시에 실행할 Payload 개수를 지정한다.  
( ex. nuclei -pc 50 -> options.PayloadConcurrency = 50 )  
  
11. httpx를 이용한 HTTP Probe의 동시 실행 개수를 지정한다.  
( ex. nuclei -prc 100 -> options.ProbeConcurrency = 100 )  
  
12. 템플릿을 동시에 로드(Load)할 최대 개수를 지정한다.  
( ex. nuclei -tlc 40 -> options.TemplateLoadingConcurrency = 40 )  
  
[CLI 옵션 등록]  
-rl  
-rld  
-per-host-rate-limit  
-rlm  
-bs  
-c  
-hbs  
-headc  
-jsc  
-pc  
-prc  
-tlc  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-rl → options.RateLimit  
-rld → options.RateLimitDuration  
-per-host-rate-limit → options.PerHostRateLimit  
-rlm → options.RateLimitMinute  
-bs → options.BulkSize  
-c → options.TemplateThreads  
-hbs → options.HeadlessBulkSize  
-headc → options.HeadlessTemplateThreads  
-jsc → options.JsConcurrency  
-pc → options.PayloadConcurrency  
-prc → options.ProbeConcurrency  
-tlc → options.TemplateLoadingConcurrency



---
상태: false
유형: Main_Function
상세: Statistics 옵션 그룹
---
# readConfig - Statistics #전지성

```
	flagSet.CreateGroup("stats", "Statistics",
		flagSet.BoolVar(&options.EnableProgressBar, "stats", false, "display statistics about the running scan"),
		flagSet.BoolVarP(&options.StatsJSON, "stats-json", "sj", false, "display statistics in JSONL(ines) format"),
		flagSet.IntVarP(&options.StatsInterval, "stats-interval", "si", 5, "number of seconds to wait between showing a statistics update"),
		flagSet.IntVarP(&options.MetricsPort, "metrics-port", "mp", 9092, "port to expose nuclei metrics on"),
		flagSet.BoolVarP(&options.HTTPStats, "http-stats", "hps", false, "enable http status capturing (experimental)"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]
stats라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Statistics 그룹을 생성하고, 스캔 진행 상황 및 통계(Statistics), 메트릭(Metrics)과 관련된 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.BoolVar(...)에서 -stats 옵션을 등록하고, FlagData 반환.
flagSet.BoolVarP(...)에서 -stats-json 옵션을 등록하고, FlagData 반환.

아래 옵션들도 모두 등록됨.
-stats
-stats-json
-stats-interval
-metrics-port
-http-stats

CreateGroup 호출 후, 모두 stats 그룹으로 지정.

[의미]
이해를 돕고자 예를 들어 설명했습니다.

1. 실행 중인 스캔의 진행 상황 및 통계를 화면에 출력한다.
( ex. nuclei -stats -> options.EnableProgressBar = true )

2. 통계 정보를 JSONL 형식으로 출력한다.
( ex. nuclei -sj -> options.StatsJSON = true )

3. 통계 정보를 몇 초마다 갱신하여 출력할지 지정한다.
( ex. nuclei -si 10 -> options.StatsInterval = 10 )

4. Nuclei의 메트릭(Metrics)을 외부에 제공할 포트를 지정한다.
( ex. nuclei -mp 9090 -> options.MetricsPort = 9090 )

5. HTTP 상태(Status) 정보를 수집한다. (실험 기능)
( ex. nuclei -hps -> options.HTTPStats = true )

[CLI 옵션 등록]
-stats
-sj
-si
-mp
-hps

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
-stats → options.EnableProgressBar
-sj → options.StatsJSON
-si → options.StatsInterval
-mp → options.MetricsPort
-hps → options.HTTPStats



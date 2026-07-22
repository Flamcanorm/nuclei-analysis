---
상태: false
유형: Main_Function
상세: Uncover 옵션 그룹
---
# readConfig - Uncover #전지성

```
	flagSet.CreateGroup("uncover", "Uncover",
		flagSet.BoolVarP(&options.Uncover, "uncover", "uc", false, "enable uncover engine"),
		flagSet.StringSliceVarP(&options.UncoverQuery, "uncover-query", "uq", nil, "uncover search query", goflags.FileStringSliceOptions),
		flagSet.StringSliceVarP(&options.UncoverEngine, "uncover-engine", "ue", nil, fmt.Sprintf("uncover search engine (%s) (default shodan)", uncover.GetUncoverSupportedAgents()), goflags.FileStringSliceOptions),
		flagSet.StringVarP(&options.UncoverField, "uncover-field", "uf", "ip:port", "uncover fields to return (ip,port,host)"),
		flagSet.IntVarP(&options.UncoverLimit, "uncover-limit", "ul", 100, "uncover results to return"),
		flagSet.IntVarP(&options.UncoverRateLimit, "uncover-ratelimit", "ur", 60, "override ratelimit of engines with unknown ratelimit (default 60 req/min)"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
uncover라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Uncover 그룹을 생성하고, Uncover 엔진을 이용한 인터넷 자산 검색과 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.BoolVarP(...)에서 -uncover 옵션을 등록하고, FlagData 반환.  
flagSet.StringSliceVarP(...)에서 -uncover-query 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-uncover  
-uncover-query  
-uncover-engine  
-uncover-field  
-uncover-limit  
-uncover-ratelimit  
  
CreateGroup 호출 후, 모두 uncover 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. Uncover 엔진을 활성화한다.  
( ex. nuclei -uc -> options.Uncover = true )  
  
2. Uncover 검색(Query)을 지정한다.  
( ex. nuclei -uq "apache" -> options.UncoverQuery )  
  
3. 검색에 사용할 Uncover 검색 엔진(Shodan, Censys 등)을 지정한다.  
( ex. nuclei -ue shodan -> options.UncoverEngine )  
  
4. 검색 결과에서 반환할 필드를 지정한다.  
(ip, port, host 등)  
( ex. nuclei -uf host -> options.UncoverField )  
  
5. 검색 결과를 최대 몇 개까지 가져올지 지정한다.  
( ex. nuclei -ul 500 -> options.UncoverLimit )  
  
6. 검색 엔진의 요청 속도(Rate Limit)를 지정한다.  
( ex. nuclei -ur 120 -> options.UncoverRateLimit )  
  
[CLI 옵션 등록]  
-uc  
-uq  
-ue  
-uf  
-ul  
-ur  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-uc → options.Uncover  
-uq → options.UncoverQuery  
-ue → options.UncoverEngine  
-uf → options.UncoverField  
-ul → options.UncoverLimit  
-ur → options.UncoverRateLimit



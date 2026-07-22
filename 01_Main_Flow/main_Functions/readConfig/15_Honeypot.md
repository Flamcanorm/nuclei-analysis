---
상태: false
유형: Main_Function
상세: Honeypot 옵션 그룹
---
# readConfig - Honeypot #전지성

```
	flagSet.CreateGroup("Honeypot", "Honeypot",
		flagSet.BoolVarP(&options.HoneypotDetection, "honeypot-detect", "hpd", false, "detect potential honeypot hosts based on match concentration"),
		flagSet.IntVarP(&options.HoneypotThreshold, "honeypot-threshold", "hpt", 15, "number of distinct template IDs required to flag a honeypot host"),
		flagSet.BoolVarP(&options.SuppressHoneypotResults, "suppress-honeypot", "shp", false, "suppress output for flagged honeypot hosts"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
Honeypot이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Honeypot 그룹을 생성하고, Honeypot(허니팟) 탐지 및 처리와 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.BoolVarP(...)에서 -honeypot-detect 옵션을 등록하고, FlagData 반환.  
flagSet.IntVarP(...)에서 -honeypot-threshold 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-honeypot-detect  
-honeypot-threshold  
-suppress-honeypot  
  
CreateGroup 호출 후, 모두 Honeypot 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. 허니팟(Honeypot)으로 의심되는 호스트를 탐지한다.  
( ex. nuclei -hpd -> options.HoneypotDetection = true )  
  
2. 허니팟으로 판단할 기준이 되는 서로 다른 Template ID 개수를 지정한다.  
기본값은 15개이다.  
( ex. nuclei -hpt 20 -> options.HoneypotThreshold = 20 )  
  
3. 허니팟으로 판단된 호스트의 결과를 출력하지 않는다.  
( ex. nuclei -shp -> options.SuppressHoneypotResults = true )  
  
[CLI 옵션 등록]  
-hpd  
-hpt  
-shp  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-hpd → options.HoneypotDetection  
-hpt → options.HoneypotThreshold  
-shp → options.SuppressHoneypotResults



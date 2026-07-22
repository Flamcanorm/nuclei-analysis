---
상태: false
유형: Main_Function
상세: Interactsh 옵션 그룹
---
# readConfig - Interactsh #전지성

```
	flagSet.CreateGroup("interactsh", "interactsh",
		flagSet.StringVarP(&options.InteractshURL, "interactsh-server", "iserver", "", fmt.Sprintf("interactsh server url for self-hosted instance (default: %s)", client.DefaultOptions.ServerURL)),
		flagSet.StringVarP(&options.InteractshToken, "interactsh-token", "itoken", "", "authentication token for self-hosted interactsh server"),
		flagSet.IntVar(&options.InteractionsCacheSize, "interactions-cache-size", 5000, "number of requests to keep in the interactions cache"),
		flagSet.IntVar(&options.InteractionsEviction, "interactions-eviction", 60, "number of seconds to wait before evicting requests from cache"),
		flagSet.IntVar(&options.InteractionsPollDuration, "interactions-poll-duration", 5, "number of seconds to wait before each interaction poll request"),
		flagSet.IntVar(&options.InteractionsCoolDownPeriod, "interactions-cooldown-period", 5, "extra time for interaction polling before exiting"),
		flagSet.BoolVarP(&options.NoInteractsh, "no-interactsh", "ni", false, "disable interactsh server for OAST testing, exclude OAST based templates"),
	)
```

#### FlagSet.CreateGroup 해석
 
 [흐름]  
interactsh라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 interactsh 그룹을 생성하고, OAST(Out-of-Band Application Security Testing) 기능 및 Interactsh 서버와 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.StringVarP(...)에서 -interactsh-server 옵션을 등록하고, FlagData 반환.  
flagSet.StringVarP(...)에서 -interactsh-token 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-interactsh-server  
-interactsh-token  
-interactions-cache-size  
-interactions-eviction  
-interactions-poll-duration  
-interactions-cooldown-period  
-no-interactsh  
  
CreateGroup 호출 후, 모두 interactsh 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. Self-Hosted Interactsh 서버의 URL을 지정한다.  
( ex. nuclei -iserver https://interact.example.com -> options.InteractshURL )  
  
2. Self-Hosted Interactsh 서버의 인증 토큰(Authentication Token)을 지정한다.  
( ex. nuclei -itoken my-token -> options.InteractshToken )  
  
3. 메모리에 유지할 Interactions(상호작용) 요청 개수를 지정한다.  
( ex. nuclei -interactions-cache-size 10000 -> options.InteractionsCacheSize )  
  
4. Interaction Cache에서 요청을 제거(Eviction)하기 전까지의 대기 시간을 초 단위로 지정한다.  
( ex. nuclei -interactions-eviction 120 -> options.InteractionsEviction )  
  
5. Interactsh 서버에 새로운 Interaction이 있는지 확인(Polling)하는 주기를 초 단위로 지정한다.  
( ex. nuclei -interactions-poll-duration 10 -> options.InteractionsPollDuration )  
  
6. 스캔 종료 전 마지막 Interaction을 기다리는 시간을 초 단위로 지정한다.  
( ex. nuclei -interactions-cooldown-period 15 -> options.InteractionsCoolDownPeriod )  
  
7. Interactsh 기능을 비활성화하여 OAST 기반 템플릿을 실행하지 않는다.  
( ex. nuclei -ni -> options.NoInteractsh = true )  
  
[CLI 옵션 등록]  
-iserver  
-itoken  
-interactions-cache-size  
-interactions-eviction  
-interactions-poll-duration  
-interactions-cooldown-period  
-ni  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-iserver → options.InteractshURL  
-itoken → options.InteractshToken  
-interactions-cache-size → options.InteractionsCacheSize  
-interactions-eviction → options.InteractionsEviction  
-interactions-poll-duration → options.InteractionsPollDuration  
-interactions-cooldown-period → options.InteractionsCoolDownPeriod  
-ni → options.NoInteractsh



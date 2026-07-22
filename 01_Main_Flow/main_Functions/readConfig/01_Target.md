---
상태: false
유형: Main_Function
상세: Target 옵션 그룹
---
# readConfig - Target #전지성

```
	flagSet.CreateGroup("input", "Target",
		flagSet.StringSliceVarP(&options.Targets, "target", "u", nil, "target URLs/hosts to scan", goflags.CommaSeparatedStringSliceOptions),
		flagSet.StringVarP(&options.TargetsFilePath, "list", "l", "", "path to file containing a list of target URLs/hosts to scan (one per line)"),
		flagSet.StringVarP(&options.InlineTargetsList, "targets-inline", "", "", "inline multiline target list (for use in template profiles)"),
		flagSet.StringSliceVarP(&options.ExcludeTargets, "exclude-hosts", "eh", nil, "hosts to exclude to scan from the input list (ip, cidr, hostname)",                             goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.StringVar(&options.Resume, "resume", "", "resume scan from and save to specified file (clustering will be disabled)"),
		flagSet.BoolVarP(&options.ScanAllIPs, "scan-all-ips", "sa", false, "scan all the IP's associated with dns record"),
		flagSet.StringSliceVarP(&options.IPVersion, "ip-version", "iv", nil, "IP version to scan of hostname (4,6) - (default 4)", goflags.CommaSeparatedStringSliceOptions),
	)
```

##### FlagSet.CreateGroup 해석

[흐름]
input이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Target 그룹을 생성하고, Target 입력과 관련된 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.StringSliceVarP(...)에서 -target 옵션을 등록하고, FlagData 반환.
flagSet.StringSliceVarP(...)에서 -list 옵션을 등록하고, FlagData 반환.

아래, 옵션들도 모두 등록됨.
-target  
-list  
-targets-inline  
-exclude-hosts  
-resume  
-scan-all-ips  
-ip-version

CreateGroup 호출 후, 모두 input의 그룹으로 지정.

[의미]
** 이해를 돕고자 예를 들어 설명했습니다. **

1. 사용자가 직접 스캔할 URL 또는 Host를 입력하는 옵션. ( ex. nuclei -u https://example.com -> options.Targets -> []string{ "https://example.com", } )
2. 스캔 대상이 적혀있는 파일을 입력. ( ex. nuclei -l targets.txt -> options.TargetsFilePath -> "targets.txt" ) 
3. 파일 대신 여러 줄 문자열을 입력한다. ( ex. a.com, b.com, c.com -> options.InlineTargetsList )
4. 스캔에서 제외할 Host를 입력한다. ( ex. nuclei -eh localhost -> options.ExcludeTargets )
5. 중단된 스캔을 이어서 실행한다. ( ex. nuclei -resume resume.cfg -> options.Resume )
6. 도메인에 연결된 모든 IP를 스캔한다. ( ex. nuclei -sa -> options.ScanAllIPs = true )
7. IPv4 또는 IPv6를 선택한다. ( ex. nuclei -iv 4 -> options.IPVersion -> []string{"4"} )

[CLI 옵션 등록]
`-u, -l, -sa, -iv`

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
`-u` → `options.Targets`
`-l` → `options.TargetsFilePath`
`-sa` → `options.ScanAllIPs`
`-iv` → `options.IPVersion` 



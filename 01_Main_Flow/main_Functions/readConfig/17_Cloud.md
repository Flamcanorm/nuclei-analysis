---
상태: false
유형: Main_Function
상세: Cloud 옵션 그룹
---
# readConfig - Cloud #전지성

```
	flagSet.CreateGroup("cloud", "Cloud",
		flagSet.DynamicVar(&pdcpauth, "auth", "true", "configure projectdiscovery cloud (pdcp) api key"),
		flagSet.StringVarP(&options.TeamID, "team-id", "tid", _pdcp.TeamIDEnv, "upload scan results to given team id (optional)"),
		flagSet.BoolVarP(&options.EnableCloudUpload, "cloud-upload", "cup", false, "upload scan results to pdcp dashboard [DEPRECATED use -dashboard]"),
		flagSet.StringVarP(&options.ScanID, "scan-id", "sid", "", "upload scan results to existing scan id (optional)"),
		flagSet.StringVarP(&options.ScanName, "scan-name", "sname", "", "scan name to set (optional)"),
		flagSet.BoolVarP(&options.EnableCloudUpload, "dashboard", "pd", false, "upload / view nuclei results in projectdiscovery cloud (pdcp) UI dashboard"),
		flagSet.StringVarP(&options.ScanUploadFile, "dashboard-upload", "pdu", "", "upload / view nuclei results file (jsonl) in projectdiscovery cloud (pdcp) UI dashboard"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]
cloud라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Cloud 그룹을 생성하고, ProjectDiscovery Cloud(PDCP) 연동 및 스캔 결과 업로드와 관련된 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.DynamicVar(...)에서 -auth 옵션을 등록하고, FlagData 반환.
flagSet.StringVarP(...)에서 -team-id 옵션을 등록하고, FlagData 반환.

아래 옵션들도 모두 등록됨.
-auth
-team-id
-cloud-upload
-scan-id
-scan-name
-dashboard
-dashboard-upload

CreateGroup 호출 후, 모두 cloud 그룹으로 지정.

[의미]
이해를 돕고자 예를 들어 설명했습니다.

1. ProjectDiscovery Cloud(PDCP)의 API Key를 설정한다.
( ex. nuclei -auth <API_KEY> -> pdcpauth )

2. 스캔 결과를 업로드할 Team ID를 지정한다.
( ex. nuclei -tid team123 -> options.TeamID )

3. 스캔 결과를 PDCP에 업로드한다. (Deprecated, 현재는 -dashboard 사용)
( ex. nuclei -cup -> options.EnableCloudUpload = true )

4. 기존 Scan ID에 결과를 업로드한다.
( ex. nuclei -sid scan123 -> options.ScanID )

5. 업로드할 스캔의 이름을 지정한다.
( ex. nuclei -sname "Weekly Scan" -> options.ScanName )

6. 스캔 결과를 ProjectDiscovery Cloud(PDCP) Dashboard에 업로드하고 조회한다.
( ex. nuclei -pd -> options.EnableCloudUpload = true )

7. JSONL 결과 파일을 ProjectDiscovery Cloud(PDCP) Dashboard에 업로드한다.
( ex. nuclei -pdu result.jsonl -> options.ScanUploadFile )

[CLI 옵션 등록]
-auth
-tid
-cup
-sid
-sname
-pd
-pdu

[입력된 값을 변수 또는 options 구조체와 연결(바인딩) 한다.]
-auth → pdcpauth
-tid → options.TeamID
-cup → options.EnableCloudUpload
-sid → options.ScanID
-sname → options.ScanName
-pd → options.EnableCloudUpload
-pdu → options.ScanUploadFile



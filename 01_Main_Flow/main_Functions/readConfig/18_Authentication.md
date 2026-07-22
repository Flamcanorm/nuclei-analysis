---
상태: false
유형: Main_Function
상세: Authentication 옵션 그룹
---
# readConfig - Authentication #전지성

```
	flagSet.CreateGroup("Authentication", "Authentication",
		flagSet.StringSliceVarP(&options.SecretsFile, "secret-file", "sf", nil, "path to config file containing secrets for nuclei authenticated scan", goflags.CommaSeparatedStringSliceOptions),
		flagSet.BoolVarP(&options.PreFetchSecrets, "prefetch-secrets", "ps", false, "prefetch secrets from the secrets file"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]
Authentication이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Authentication 그룹을 생성하고, 인증(Authentication) 및 Secret 관리와 관련된 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.StringSliceVarP(...)에서 -secret-file 옵션을 등록하고, FlagData 반환.
flagSet.BoolVarP(...)에서 -prefetch-secrets 옵션을 등록하고, FlagData 반환.

아래 옵션들도 모두 등록됨.
-secret-file
-prefetch-secrets

CreateGroup 호출 후, 모두 Authentication 그룹으로 지정.

[의미]
이해를 돕고자 예를 들어 설명했습니다.

1.  인증 정보(Secret)가 저장된 설정 파일을 지정한다.
( ex. nuclei -sf secrets.yaml -> options.SecretsFile )

2. 스캔 시작 전에 Secret 파일의 인증 정보를 미리 읽어온다.
( ex. nuclei -ps -> options.PreFetchSecrets = true )

[CLI 옵션 등록]
-sf
-ps

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
-sf → options.SecretsFile
-ps → options.PreFetchSecrets

```
flagSet.SetCustomHelpText(`EXAMPLES:
Run nuclei on single host:
	$ nuclei -target example.com

Run nuclei with specific template directories:
	$ nuclei -target example.com -t http/cves/ -t ssl

Run nuclei against a list of hosts:
	$ nuclei -list hosts.txt

Run nuclei with a JSON output:
	$ nuclei -target example.com -json-export output.json

Run nuclei with sorted Markdown outputs (with environment variables):
	$ MARKDOWN_EXPORT_SORT_MODE=template nuclei -target example.com -markdown-export nuclei_report/

Additional documentation is available at: https://docs.nuclei.sh/getting-started/running
	`)
```

#### FlagSet.SetCustomHelpText 해석

이 부분은 지금까지의 `CreateGroup()`과는 조금 다르다.

`SetCustomHelpText()`는 **새로운 CLI 옵션을 등록하는 함수가 아니라**, 사용자가 `-h`, `--help` 등을 입력했을 때 **도움말(Help) 화면의 예제(EXAMPLES) 부분에 출력할 텍스트를 등록하는 함수**이다.

[`SetDescription()`과의 차이]
SetDescription 함수는 프로그램의 간단한 설명(소개)을 등록.
SetcustomHelpText 함수는 Help 화면에 출력할 예제나 추가 안내를 등록.

즉,

nuclei -h  

   ↓

프로그램 설명 (SetDescription)  
  
   ↓  
  
옵션 목록 (CreateGroup으로 등록한 옵션들)  
   
   ↓  
   
용 예시 (SetCustomHelpText)

[흐름]
사용자가 -h 또는 --help 옵션을 입력했을 때 출력할 사용자 정의(Custom) Help 메시지를 등록한다.

SetCustomHelpText()를 호출하면,
여러 개의 사용 예시(EXAMPLES)와 추가 문서 링크를 FlagSet 내부에 저장한다.

실제로는 바로 출력되지 않으며,
사용자가 Help를 요청할 때 Help 화면의 마지막 부분에 함께 출력된다.

[의미]
이해를 돕고자 예를 들어 설명했습니다.

1. 단일 Host를 대상으로 스캔하는 방법을 보여준다.
( ex. nuclei -target example.com )

2. 특정 Template 디렉터리를 지정하여 스캔하는 방법을 보여준다.
( ex. nuclei -target example.com -t http/cves/ -t ssl )

3. Host 목록이 저장된 파일을 이용하여 스캔하는 방법을 보여준다.
( ex. nuclei -list hosts.txt )

4. 결과를 JSON 파일로 저장하는 방법을 보여준다.
( ex. nuclei -target example.com -json-export output.json )

5. Markdown 형식으로 결과를 정렬하여 저장하는 방법을 보여준다.
( ex. MARKDOWN_EXPORT_SORT_MODE=template nuclei -target example.com -markdown-export nuclei_report/ )

추가 사용법은 공식 문서를 참고하도록 안내한다.
( https://docs.nuclei.sh/getting-started/running )

[CLI 옵션 등록]
없음

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
없음



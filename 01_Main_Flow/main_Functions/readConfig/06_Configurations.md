---
상태: false
유형: Main_Function
상세: Configurations 옵션 그룹
---
# readConfig - Configurations #전지성

```
	flagSet.CreateGroup("configs", "Configurations",
		flagSet.StringVar(&cfgFile, "config", "", "path to the nuclei configuration file"),
		flagSet.StringVarP(&templateProfile, "profile", "tp", "", "template profile config file to run"),
		flagSet.BoolVarP(&options.ListTemplateProfiles, "profile-list", "tpl", false, "list community template profiles"),
		flagSet.BoolVarP(&options.FollowRedirects, "follow-redirects", "fr", false, "enable following redirects for http templates"),
		flagSet.BoolVarP(&options.FollowHostRedirects, "follow-host-redirects", "fhr", false, "follow redirects on the same host"),
		flagSet.IntVarP(&options.MaxRedirects, "max-redirects", "mr", 10, "max number of redirects to follow for http templates"),
		flagSet.BoolVarP(&options.DisableRedirects, "disable-redirects", "dr", false, "disable redirects for http templates"),
		flagSet.StringVarP(&options.ReportingConfig, "report-config", "rc", "", "nuclei reporting module configuration file"), // TODO merge into the config file or rename to issue-tracking
		flagSet.StringSliceVarP(&options.CustomHeaders, "header", "H", nil, "custom header/cookie to include in all http request in header:value format (cli, file)", goflags.FileStringSliceOptions),
		flagSet.RuntimeMapVarP(&options.Vars, "var", "V", nil, "custom vars in key=value format"),
		flagSet.StringVarP(&options.ResolversFile, "resolvers", "r", "", "file containing resolver list for nuclei"),
		flagSet.BoolVarP(&options.SystemResolvers, "system-resolvers", "sr", false, "use system DNS resolving as error fallback"),
		flagSet.BoolVarP(&options.DisableClustering, "disable-clustering", "dc", false, "disable clustering of requests"),
		flagSet.BoolVar(&options.OfflineHTTP, "passive", false, "enable passive HTTP response processing mode"),
		flagSet.BoolVarP(&options.ForceAttemptHTTP2, "force-http2", "fh2", false, "force http2 connection on requests"),
		flagSet.BoolVarP(&options.EnvironmentVariables, "env-vars", "ev", false, "enable environment variables to be used in template"),
		flagSet.StringVarP(&options.ClientCertFile, "client-cert", "cc", "", "client certificate file (PEM-encoded) used for authenticating against scanned hosts"),
		flagSet.StringVarP(&options.ClientKeyFile, "client-key", "ck", "", "client key file (PEM-encoded) used for authenticating against scanned hosts"),
		flagSet.StringVarP(&options.ClientCAFile, "client-ca", "ca", "", "client certificate authority file (PEM-encoded) used for authenticating against scanned hosts"),
		flagSet.BoolVarP(&options.ShowMatchLine, "show-match-line", "sml", false, "show match lines for file templates, works with extractors only"),
		flagSet.BoolVar(&options.ZTLS, "ztls", false, "use ztls library with autofallback to standard one for tls13 [Deprecated] autofallback to ztls is enabled by default"), //nolint:all
		flagSet.StringVar(&options.SNI, "sni", "", "tls sni hostname to use (default: input domain name)"),
		flagSet.DurationVarP(&options.DialerKeepAlive, "dialer-keep-alive", "dka", 0, "keep-alive duration for network requests."),
		flagSet.BoolVarP(&options.AllowLocalFileAccess, "allow-local-file-access", "lfa", false, "allows file (payload) access anywhere on the system"),
		flagSet.BoolVarP(&options.RestrictLocalNetworkAccess, "restrict-local-network-access", "lna", false, "blocks connections to the local / private network"),
		flagSet.StringVarP(&options.Interface, "interface", "i", "", "network interface to use for network scan"),
		flagSet.StringVarP(&options.AttackType, "attack-type", "at", "", "type of payload combinations to perform (batteringram,pitchfork,clusterbomb)"),
		flagSet.StringVarP(&options.SourceIP, "source-ip", "sip", "", "source ip address to use for network scan"),
		flagSet.IntVarP(&options.ResponseReadSize, "response-size-read", "rsr", 0, "max response size to read in bytes"),
		flagSet.IntVarP(&options.ResponseSaveSize, "response-size-save", "rss", unitutils.Mega, "max response size to read in bytes"),
		flagSet.CallbackVar(resetCallback, "reset", "reset removes all nuclei configuration and data files (including nuclei-templates)"),
		flagSet.BoolVarP(&options.TlsImpersonate, "tls-impersonate", "tlsi", false, "enable experimental client hello (ja3) tls randomization"),
		flagSet.StringVarP(&options.HttpApiEndpoint, "http-api-endpoint", "hae", "", "experimental http api endpoint"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
configs라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Configurations 그룹을 생성하고, Nuclei의 환경설정(Configuration), HTTP 설정, 네트워크 설정, 인증서 설정 등 실행 환경과 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.StringVar(...)에서 -config 옵션을 등록하고, FlagData 반환.  
flagSet.StringVarP(...)에서 -profile 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-config  
-profile  
-profile-list  
-follow-redirects  
-follow-host-redirects  
-max-redirects  
-disable-redirects  
-report-config  
-header  
-var  
-resolvers  
-system-resolvers  
-disable-clustering  
-passive  
-force-http2  
-env-vars  
-client-cert  
-client-key  
-client-ca  
-show-match-line  
-ztls  
-sni  
-dialer-keep-alive  
-allow-local-file-access  
-restrict-local-network-access  
-interface  
-attack-type  
-source-ip  
-response-size-read  
-response-size-save  
-reset  
-tls-impersonate  
-http-api-endpoint  
  
CreateGroup 호출 후, 모두 configs 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. Nuclei 설정 파일(config.yaml 등)의 경로를 지정한다.  
( ex. nuclei -config config.yaml -> cfgFile )  
  
2. Template Profile 설정 파일을 지정한다.  
( ex. nuclei -tp profile.yaml -> templateProfile )  
  
3. 사용 가능한 Template Profile 목록을 출력한다.  
( ex. nuclei -tpl -> options.ListTemplateProfiles = true )  
  
4. HTTP Redirect를 따라간다.  
( ex. nuclei -fr -> options.FollowRedirects = true )  
  
5. 같은 호스트의 Redirect만 따라간다.  
( ex. nuclei -fhr -> options.FollowHostRedirects = true )  
  
6. 최대 Redirect 횟수를 지정한다.  
( ex. nuclei -mr 5 -> options.MaxRedirects = 5 )  
  
7. Redirect를 비활성화한다.  
( ex. nuclei -dr -> options.DisableRedirects = true )  
  
8. Reporting 설정 파일을 지정한다.  
( ex. nuclei -rc report.yaml -> options.ReportingConfig )  
  
9. 모든 HTTP 요청에 사용자 정의 Header 또는 Cookie를 추가한다.  
( ex. nuclei -H "Authorization: Bearer token" -> options.CustomHeaders )  
  
10. 사용자 정의 변수를 key=value 형태로 전달한다.  
( ex. nuclei -V token=12345 -> options.Vars )  
  
11. DNS Resolver 목록 파일을 지정한다.  
( ex. nuclei -r resolvers.txt -> options.ResolversFile )  
  
12. DNS Resolver 실패 시 시스템 DNS를 사용한다.  
( ex. nuclei -sr -> options.SystemResolvers = true )  
  
13. 요청 Clustering 기능을 비활성화한다.  
( ex. nuclei -dc -> options.DisableClustering = true )  
  
14. Passive HTTP 응답 처리 모드를 활성화한다.  
( ex. nuclei -passive -> options.OfflineHTTP = true )  
  
15. HTTP/2 연결을 강제로 사용한다.  
( ex. nuclei -fh2 -> options.ForceAttemptHTTP2 = true )  
  
16. 템플릿에서 환경변수(Environment Variables)를 사용할 수 있도록 한다.  
( ex. nuclei -ev -> options.EnvironmentVariables = true )  
  
17. TLS 클라이언트 인증서 파일을 지정한다.  
( ex. nuclei -cc client.pem -> options.ClientCertFile )  
  
18. TLS 클라이언트 개인키 파일을 지정한다.  
( ex. nuclei -ck client.key -> options.ClientKeyFile )  
  
19. TLS 인증기관(CA) 파일을 지정한다.  
( ex. nuclei -ca ca.pem -> options.ClientCAFile )  
  
20. 파일(File) 템플릿에서 일치한 줄(Match Line)을 출력한다.  
( ex. nuclei -sml -> options.ShowMatchLine = true )  
  
21. ZTLS 라이브러리를 사용한다. (Deprecated)  
( ex. nuclei -ztls -> options.ZTLS = true )  
  
22. TLS SNI(Server Name Indication)에 사용할 호스트명을 지정한다.  
( ex. nuclei -sni example.com -> options.SNI )  
  
23. 네트워크 연결의 Keep-Alive 시간을 지정한다.  
( ex. nuclei -dka 30s -> options.DialerKeepAlive )  
  
24. 시스템의 모든 위치에 있는 Payload 파일 접근을 허용한다.  
( ex. nuclei -lfa -> options.AllowLocalFileAccess = true )  
  
25. 로컬 및 사설 네트워크 접근을 차단한다.  
( ex. nuclei -lna -> options.RestrictLocalNetworkAccess = true )  
  
26. 네트워크 스캔에 사용할 인터페이스를 지정한다.  
( ex. nuclei -i eth0 -> options.Interface )  
  
27. Payload 조합 방식을 지정한다.  
( ex. nuclei -at clusterbomb -> options.AttackType )  
  
28. 네트워크 스캔에 사용할 Source IP를 지정한다.  
( ex. nuclei -sip 192.168.1.100 -> options.SourceIP )  
  
29. 응답(Response)을 읽을 최대 크기를 지정한다.  
( ex. nuclei -rsr 1048576 -> options.ResponseReadSize )  
  
30. 응답(Response)을 저장할 최대 크기를 지정한다.  
( ex. nuclei -rss 10485760 -> options.ResponseSaveSize )  
  
31. Nuclei 설정 및 데이터(템플릿 포함)를 초기화한다.  
( ex. nuclei -reset -> resetCallback() 실행 )  
  
32. JA3 기반 TLS ClientHello 랜덤화를 활성화한다.  
( ex. nuclei -tlsi -> options.TlsImpersonate = true )  
  
33. 실험용 HTTP API Endpoint를 지정한다.  
( ex. nuclei -hae http://localhost:8080 -> options.HttpApiEndpoint )  
  
[CLI 옵션 등록]  
-config, -tp, -tpl, -fr, -fhr, -mr, -dr, -rc, -H, -V,  
-r, -sr, -dc, -passive, -fh2, -ev, -cc, -ck, -ca, -sml,  
-ztls, -sni, -dka, -lfa, -lna, -i, -at, -sip,  
-rsr, -rss, -reset, -tlsi, -hae  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-config → cfgFile  
-tp → templateProfile  
-tpl → options.ListTemplateProfiles  
-fr → options.FollowRedirects  
-fhr → options.FollowHostRedirects  
-mr → options.MaxRedirects  
-dr → options.DisableRedirects  
-rc → options.ReportingConfig  
-H → options.CustomHeaders  
-V → options.Vars  
-r → options.ResolversFile  
-sr → options.SystemResolvers  
-dc → options.DisableClustering  
-passive → options.OfflineHTTP  
-fh2 → options.ForceAttemptHTTP2  
-ev → options.EnvironmentVariables  
-cc → options.ClientCertFile  
-ck → options.ClientKeyFile  
-ca → options.ClientCAFile  
-sml → options.ShowMatchLine  
-ztls → options.ZTLS  
-sni → options.SNI  
-dka → options.DialerKeepAlive  
-lfa → options.AllowLocalFileAccess  
-lna → options.RestrictLocalNetworkAccess  
-i → options.Interface  
-at → options.AttackType  
-sip → options.SourceIP  
-rsr → options.ResponseReadSize  
-rss → options.ResponseSaveSize  
-reset → resetCallback()  
-tlsi → options.TlsImpersonate  
-hae → options.HttpApiEndpoint



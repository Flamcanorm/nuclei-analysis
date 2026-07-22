---
상태: false
유형: Main_Function
상세: Fuzzing 옵션 그룹
---
# readConfig - Fuzzing #전지성

```
	flagSet.CreateGroup("fuzzing", "Fuzzing",
		flagSet.StringVarP(&options.FuzzingType, "fuzzing-type", "ft", "", "overrides fuzzing type set in template (replace, prefix, postfix, infix)"),
		flagSet.StringVarP(&options.FuzzingMode, "fuzzing-mode", "fm", "", "overrides fuzzing mode set in template (multiple, single)"),
		flagSet.BoolVar(&fuzzFlag, "fuzz", false, "enable loading fuzzing templates (Deprecated: use -dast instead)"),
		flagSet.BoolVar(&options.DAST, "dast", false, "enable / run dast (fuzz) nuclei templates"),
		flagSet.BoolVarP(&options.DASTServer, "dast-server", "dts", false, "enable dast server mode (live fuzzing)"),
		flagSet.BoolVarP(&options.DASTReport, "dast-report", "dtr", false, "write dast scan report to file"),
		flagSet.StringVarP(&options.DASTServerToken, "dast-server-token", "dtst", "", "dast server token (optional)"),
		flagSet.StringVarP(&options.DASTServerAddress, "dast-server-address", "dtsa", "localhost:9055", "dast server address"),
		flagSet.BoolVarP(&options.DisplayFuzzPoints, "display-fuzz-points", "dfp", false, "display fuzz points in the output for debugging"),
		flagSet.IntVar(&options.FuzzParamFrequency, "fuzz-param-frequency", 10, "frequency of uninteresting parameters for fuzzing before skipping"),
		flagSet.StringVarP(&options.FuzzAggressionLevel, "fuzz-aggression", "fa", "low", "fuzzing aggression level controls payload count for fuzz (low, medium, high)"),
		flagSet.StringSliceVarP(&options.Scope, "fuzz-scope", "cs", nil, "in scope url regex to be followed by fuzzer", goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.StringSliceVarP(&options.OutOfScope, "fuzz-out-scope", "cos", nil, "out of scope url regex to be excluded by fuzzer", goflags.FileCommaSeparatedStringSliceOptions),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
fuzzing이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Fuzzing 그룹을 생성하고, Fuzzing 및 DAST(Dynamic Application Security Testing)와 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.StringVarP(...)에서 -fuzzing-type 옵션을 등록하고, FlagData 반환.  
flagSet.StringVarP(...)에서 -fuzzing-mode 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-fuzzing-type  
-fuzzing-mode  
-fuzz  
-dast  
-dast-server  
-dast-report  
-dast-server-token  
-dast-server-address  
-display-fuzz-points  
-fuzz-param-frequency  
-fuzz-aggression  
-fuzz-scope  
-fuzz-out-scope  
  
CreateGroup 호출 후, 모두 fuzzing 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. 템플릿에 설정된 Fuzzing 방식을 덮어쓴다.  
(replace, prefix, postfix, infix 중 선택)  
( ex. nuclei -ft prefix -> options.FuzzingType )  
  
2. 템플릿에 설정된 Fuzzing 실행 방식을 덮어쓴다.  
(multiple, single 중 선택)  
( ex. nuclei -fm multiple -> options.FuzzingMode )  
  
3. Fuzzing 템플릿 실행을 활성화한다. (Deprecated, 현재는 -dast 사용)  
( ex. nuclei -fuzz -> fuzzFlag = true )  
  
4. DAST(Fuzz) 템플릿 실행을 활성화한다.  
( ex. nuclei -dast -> options.DAST = true )  
  
5. DAST 서버 모드(Live Fuzzing)를 활성화한다.  
( ex. nuclei -dts -> options.DASTServer = true )  
  
6. DAST 스캔 결과를 파일로 저장한다.  
( ex. nuclei -dtr -> options.DASTReport = true )  
  
7. DAST 서버 인증 토큰을 지정한다.  
( ex. nuclei -dtst my-token -> options.DASTServerToken )  
  
8. DAST 서버 주소를 지정한다.  
( ex. nuclei -dtsa localhost:9055 -> options.DASTServerAddress )  
  
9. 디버깅을 위해 Fuzz Point를 출력한다.  
( ex. nuclei -dfp -> options.DisplayFuzzPoints = true )  
  
10. 효과가 없는(흥미롭지 않은) 파라미터를 몇 번까지 퍼징할지 지정한다.  
( ex. nuclei -fuzz-param-frequency 20 -> options.FuzzParamFrequency )  
  
11. Fuzzing 강도를 지정한다.  
(low, medium, high 중 선택)  
( ex. nuclei -fa high -> options.FuzzAggressionLevel )  
  
12. Fuzzer가 따라갈 URL 범위를 정규표현식(Regex)으로 지정한다.  
( ex. nuclei -cs "https://example.com/.*" -> options.Scope )  
  
13. Fuzzer가 제외할 URL 범위를 정규표현식(Regex)으로 지정한다.  
( ex. nuclei -cos "https://example.com/logout.*" -> options.OutOfScope )  
  
[CLI 옵션 등록]  
-ft  
-fm  
-fuzz  
-dast  
-dts  
-dtr  
-dtst  
-dtsa  
-dfp  
-fuzz-param-frequency  
-fa  
-cs  
-cos  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-ft → options.FuzzingType  
-fm → options.FuzzingMode  
-fuzz → fuzzFlag  
-dast → options.DAST  
-dts → options.DASTServer  
-dtr → options.DASTReport  
-dtst → options.DASTServerToken  
-dtsa → options.DASTServerAddress  
-dfp → options.DisplayFuzzPoints  
-fuzz-param-frequency → options.FuzzParamFrequency  
-fa → options.FuzzAggressionLevel  
-cs → options.Scope  
-cos → options.OutOfScope



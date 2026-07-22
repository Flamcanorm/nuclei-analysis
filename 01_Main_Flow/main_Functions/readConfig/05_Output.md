---
상태: false
유형: Main_Function
상세: Output 옵션 그룹
---
# readConfig - Output #전지성

```
flagSet.CreateGroup("output", "Output",
		flagSet.StringVarP(&options.Output, "output", "o", "", "output file to write found issues/vulnerabilities"),
		flagSet.BoolVarP(&options.StoreResponse, "store-resp", "sresp", false, "store all request/response passed through nuclei to output directory"),
		flagSet.StringVarP(&options.StoreResponseDir, "store-resp-dir", "srd", runner.DefaultDumpTrafficOutputFolder, "store all request/response passed through nuclei to custom directory"),
		flagSet.BoolVar(&options.Silent, "silent", false, "display findings only"),
		flagSet.BoolVarP(&options.NoColor, "no-color", "nc", false, "disable output content coloring (ANSI escape codes)"),
		flagSet.BoolVarP(&options.JSONL, "jsonl", "j", false, "write output in JSONL(ines) format"),
		flagSet.BoolVarP(&options.JSONRequests, "include-rr", "irr", true, "include request/response pairs in the JSON, JSONL, and Markdown outputs (for findings only) [DEPRECATED use `-omit-raw`]"),
		flagSet.BoolVarP(&options.OmitRawRequests, "omit-raw", "or", false, "omit request/response pairs in the JSON, JSONL, Markdown, and PDF outputs (for findings only)"),
		flagSet.BoolVarP(&options.OmitTemplate, "omit-template", "ot", false, "omit encoded template in the JSON, JSONL output"),
		flagSet.BoolVarP(&options.NoMeta, "no-meta", "nm", false, "disable printing result metadata in cli output"),
		flagSet.BoolVarP(&options.Timestamp, "timestamp", "ts", false, "enables printing timestamp in cli output"),
		flagSet.StringVarP(&options.ReportingDB, "report-db", "rdb", "", "nuclei reporting database (always use this to persist report data)"),
		flagSet.BoolVarP(&options.MatcherStatus, "matcher-status", "ms", false, "display match failure status"),
		flagSet.StringVarP(&options.MarkdownExportDirectory, "markdown-export", "me", "", "directory to export results in markdown format"),
		flagSet.StringVarP(&options.SarifExport, "sarif-export", "se", "", "file to export results in SARIF format"),
		flagSet.StringVarP(&options.JSONExport, "json-export", "je", "", "file to export results in JSON format"),
		flagSet.StringVarP(&options.JSONLExport, "jsonl-export", "jle", "", "file to export results in JSONL(ine) format"),
		flagSet.StringVarP(&options.PDFExport, "pdf-export", "pe", "", "file to export results in PDF format"),
		flagSet.StringSliceVarP(&options.Redact, "redact", "rd", nil, "redact given list of keys from query parameter, request header and body", goflags.CommaSeparatedStringSliceOptions),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
output이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Output 그룹을 생성하고, 스캔 결과를 어떤 형식으로 출력하거나 저장할지 정하는 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.StringVarP(...)에서 -output 옵션을 등록하고, FlagData 반환.  
flagSet.BoolVarP(...)에서 -store-resp 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-output  
-store-resp  
-store-resp-dir  
-silent  
-no-color  
-jsonl  
-include-rr  
-omit-raw  
-omit-template  
-no-meta  
-timestamp  
-report-db  
-matcher-status  
-markdown-export  
-sarif-export  
-json-export  
-jsonl-export  
-pdf-export  
-redact  
  
CreateGroup 호출 후, 모두 output 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. 스캔 결과를 저장할 출력 파일을 지정한다.  
( ex. nuclei -o result.txt -> options.Output )  
  
2. 요청/응답 데이터를 출력 디렉터리에 저장한다.  
( ex. nuclei -sresp -> options.StoreResponse = true )  
  
3. 요청/응답 데이터를 저장할 디렉터리를 지정한다.  
( ex. nuclei -srd responses/ -> options.StoreResponseDir )  
  
4. 탐지 결과만 조용히 출력한다.  
( ex. nuclei -silent -> options.Silent = true )  
  
5. 터미널 출력 색상(ANSI 색상 코드)을 비활성화한다.  
( ex. nuclei -nc -> options.NoColor = true )  
  
6. 출력을 JSONL 형식으로 작성한다.  
( ex. nuclei -j -> options.JSONL = true )  
  
7. JSON, JSONL, Markdown 출력에 요청/응답 쌍을 포함한다. 단, Deprecated 옵션이다.  
( ex. nuclei -irr -> options.JSONRequests = true )  
  
8. JSON, JSONL, Markdown, PDF 출력에서 원본 요청/응답을 제외한다.  
( ex. nuclei -or -> options.OmitRawRequests = true )  
  
9. JSON, JSONL 출력에서 인코딩된 템플릿 내용을 제외한다.  
( ex. nuclei -ot -> options.OmitTemplate = true )  
  
10. CLI 출력에서 결과 메타데이터 표시를 비활성화한다.  
( ex. nuclei -nm -> options.NoMeta = true )  
  
11. CLI 출력에 타임스탬프를 표시한다.  
( ex. nuclei -ts -> options.Timestamp = true )  
  
12. Nuclei 리포팅 데이터베이스 파일을 지정한다.  
( ex. nuclei -rdb nuclei-report.db -> options.ReportingDB )  
  
13. 매처 실패 상태도 출력한다.  
( ex. nuclei -ms -> options.MatcherStatus = true )  
  
14. 결과를 Markdown 형식으로 내보낼 디렉터리를 지정한다.  
( ex. nuclei -me markdown-report/ -> options.MarkdownExportDirectory )  
  
15. 결과를 SARIF 형식 파일로 내보낸다.  
( ex. nuclei -se result.sarif -> options.SarifExport )  
  
16. 결과를 JSON 형식 파일로 내보낸다.  
( ex. nuclei -je result.json -> options.JSONExport )  
  
17. 결과를 JSONL 형식 파일로 내보낸다.  
( ex. nuclei -jle result.jsonl -> options.JSONLExport )  
  
18. 결과를 PDF 형식 파일로 내보낸다.  
( ex. nuclei -pe result.pdf -> options.PDFExport )  
  
19. 요청 파라미터, 헤더, 바디에서 지정한 키를 마스킹한다.  
( ex. nuclei -rd token,password -> options.Redact )  
  
[CLI 옵션 등록]  
-o, -sresp, -srd, -silent, -nc, -j, -irr, -or, -ot,  
-nm, -ts, -rdb, -ms, -me, -se, -je, -jle, -pe, -rd  
  
[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]  
-o → options.Output  
-sresp → options.StoreResponse  
-srd → options.StoreResponseDir  
-silent → options.Silent  
-nc → options.NoColor  
-j → options.JSONL  
-irr → options.JSONRequests  
-or → options.OmitRawRequests  
-ot → options.OmitTemplate  
-nm → options.NoMeta  
-ts → options.Timestamp  
-rdb → options.ReportingDB  
-ms → options.MatcherStatus  
-me → options.MarkdownExportDirectory  
-se → options.SarifExport  
-je → options.JSONExport  
-jle → options.JSONLExport  
-pe → options.PDFExport  
-rd → options.Redact




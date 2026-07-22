---
상태: false
유형: Main_Function
상세: Filtering 옵션 그룹
---
# readConfig - Filtering #전지성

```
	flagSet.CreateGroup("filters", "Filtering",
		flagSet.StringSliceVarP(&options.Authors, "author", "a", nil, "templates to run based on authors (comma-separated, file)", goflags.FileNormalizedStringSliceOptions),
		flagSet.StringSliceVar(&options.Tags, "tags", nil, "templates to run based on tags (comma-separated, file)", goflags.FileNormalizedStringSliceOptions), 
		flagSet.StringSliceVarP(&options.ExcludeTags, "exclude-tags", "etags", nil, "templates to exclude based on tags (comma-separated, file)",                                     goflags.FileNormalizedStringSliceOptions),
		flagSet.StringSliceVarP(&options.IncludeTags, "include-tags", "itags", nil, "tags to be executed even if they are excluded either by default or configuration",               goflags.FileNormalizedStringSliceOptions), // TODO show default deny list
		flagSet.StringSliceVarP(&options.IncludeIds, "template-id", "id", nil, "templates to run based on template ids (comma-separated, file, allow-wildcard)",                      goflags.FileNormalizedStringSliceOptions),
		flagSet.StringSliceVarP(&options.ExcludeIds, "exclude-id", "eid", nil, "templates to exclude based on template ids (comma-separated, file)",                                  goflags.FileNormalizedStringSliceOptions),
		flagSet.StringSliceVarP(&options.IncludeTemplates, "include-templates", "it", nil, "path to template file or directory to be executed even if they are excluded               either by default or configuration", goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.StringSliceVarP(&options.ExcludedTemplates, "exclude-templates", "et", nil, "path to template file or directory to exclude (comma-separated, file)",                  goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.StringSliceVarP(&options.ExcludeMatchers, "exclude-matchers", "em", nil, "template matchers to exclude in result",                                                    goflags.FileCommaSeparatedStringSliceOptions),
		flagSet.VarP(&options.Severities, "severity", "s", fmt.Sprintf("templates to run based on severity. Possible values: %s",                                                     severity.GetSupportedSeverities().String())),
		flagSet.VarP(&options.ExcludeSeverities, "exclude-severity", "es", fmt.Sprintf("templates to exclude based on severity. Possible values: %s",                                 severity.GetSupportedSeverities().String())),
		flagSet.VarP(&options.Protocols, "type", "pt", fmt.Sprintf("templates to run based on protocol type. Possible values: %s",                                                    templateTypes.GetSupportedProtocolTypes())),  
		flagSet.VarP(&options.ExcludeProtocols, "exclude-type", "ept", fmt.Sprintf("templates to exclude based on protocol type. Possible values: %s",                                templateTypes.GetSupportedProtocolTypes())),
		flagSet.StringSliceVarP(&options.IncludeConditions, "template-condition", "tc", nil, "templates to run based on expression condition", goflags.StringSliceOptions),
	)
```

##### FlagSet.CreateGroup 해석

[흐름]
filters라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Filtering 그룹을 생성하고, 실행할 템플릿을 다양한 조건으로 선택하거나 제외하는 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.StringSliceVarP(...)에서 -author 옵션을 등록하고, FlagData 반환.
flagSet.StringSliceVar(...)에서 -tags 옵션을 등록하고, FlagData 반환.

아래 옵션들도 모두 등록됨.
-author
-tags
-exclude-tags
-include-tags
-template-id
-exclude-id
-include-templates
-exclude-templates
-exclude-matchers
-severity
-exclude-severity
-type
-exclude-type
-template-condition

CreateGroup 호출 후, 모두 filters 그룹으로 지정.

[의미]
이해를 돕고자 예를 들어 설명했습니다.

1. 특정 작성자(Author)가 만든 템플릿만 실행한다.
( ex. nuclei -a pdteam -> options.Authors )

2. 특정 태그(Tag)를 가진 템플릿만 실행한다.
( ex. nuclei -tags cve,rce -> options.Tags )

3. 특정 태그(Tag)를 가진 템플릿은 제외한다.
( ex. nuclei -etags dos -> options.ExcludeTags )

4. 기본적으로 제외된 태그라도 강제로 포함하여 실행한다.
( ex. nuclei -itags fuzz -> options.IncludeTags )

5. 특정 Template ID를 가진 템플릿만 실행한다.
( ex. nuclei -id cve-2024-* -> options.IncludeIds )

6. 특정 Template ID를 가진 템플릿을 제외한다.
( ex. nuclei -eid cve-2023-* -> options.ExcludeIds )

7. 특정 템플릿(파일 또는 디렉터리)을 강제로 포함하여 실행한다.
( ex. nuclei -it templates/http/ -> options.IncludeTemplates )

8. 특정 템플릿(파일 또는 디렉터리)을 실행 대상에서 제외한다.
( ex. nuclei -et templates/dos/ -> options.ExcludedTemplates )

9. 특정 Matcher를 결과에서 제외한다.
( ex. nuclei -em word-matcher -> options.ExcludeMatchers )

10. 심각도(Severity)에 따라 템플릿을 실행한다.
( ex. nuclei -s critical,high -> options.Severities )

11. 특정 심각도의 템플릿은 제외한다.
( ex. nuclei -es info -> options.ExcludeSeverities )

12. 특정 프로토콜(HTTP, DNS 등)의 템플릿만 실행한다.
( ex. nuclei -pt http -> options.Protocols )

13. 특정 프로토콜의 템플릿은 제외한다.
( ex. nuclei -ept dns -> options.ExcludeProtocols )

14. 조건식(Expression)에 맞는 템플릿만 실행한다.
( ex. nuclei -tc "severity == critical" -> options.IncludeConditions )

[CLI 옵션 등록]
-a, -tags, -etags, -itags, -id, -eid, -it, -et,
-em, -s, -es, -pt, -ept, -tc

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
-a      → options.Authors
-tags   → options.Tags
-etags  → options.ExcludeTags
-itags  → options.IncludeTags
-id     → options.IncludeIds
-eid    → options.ExcludeIds
-it     → options.IncludeTemplates
-et     → options.ExcludedTemplates
-em     → options.ExcludeMatchers
-s      → options.Severities
-es     → options.ExcludeSeverities
-pt     → options.Protocols
-ept    → options.ExcludeProtocols
-tc     → options.IncludeConditions

### 26.07.05 - 전지성


#### 표준 라이브러리 설명 분리
[[../../../03_External_Packages/04_etc/sample#사용된 표준 라이브러리|sample]] 참고.

#### main.go 안에 있는 함수 사용
printVersion()
printTemplateVersion()

#### 328줄, 사용된 외부파일
(새로나온 외부파일만 작성됨)
[runner_options](../../../02_Internal_Packages/runner_options.md) — `internal/runner/options.go`

#### 384 줄, 사용된 외부파일
 (새로나온 외부파일만 작성됨)
[interactsh_client](../../../03_External_Packages/interactsh_client.md) — `interactsh/pkg/client/client.go`

#### 412줄, 사용된 외부파일
 (새로나온 외부파일만 작성됨)
[uncover_uncover](../../../02_Internal_Packages/uncover_uncover.md) — `pkg/protocols/common/uncover/uncover.go`

#### 443줄, 사용된 외부파일
 (새로나온 외부파일만 작성됨)
[pkg_types](../../../02_Internal_Packages/pkg_types.md) — `pkg/types/types.go`
[types_scanstrategy](../../../02_Internal_Packages/types_scanstrategy.md) — `scanstrategy` 패키지

#### 508줄, 사용된 외부파일
 (새로나온 외부파일만 작성됨)
[pdcp_writer](../../../02_Internal_Packages/pdcp_writer.md) — `internal/pdcp/writer.go`
PDCP(ProjectDiscovery Cloud) 패키지



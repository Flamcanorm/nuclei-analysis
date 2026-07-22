---
상태: false
유형: Main_Function
상세: Target-Format 옵션 그룹
---
# readConfig - Target-Format #전지성

```
	flagSet.CreateGroup("target-format", "Target-Format",
		flagSet.StringVarP(&options.InputFileMode, "input-mode", "im", "list", fmt.Sprintf("mode of input file (%v)", provider.SupportedInputFormats())),
		flagSet.BoolVarP(&options.FormatUseRequiredOnly, "required-only", "ro", false, "use only required fields in input format when generating requests"),
		flagSet.BoolVarP(&options.SkipFormatValidation, "skip-format-validation", "sfv", false, "skip format validation (like missing vars) when parsing input file"),
		flagSet.BoolVarP(&options.VarsTextTemplating, "vars-text-templating", "vtt", false, "enable text templating for vars in input file (only for yaml input mode)"),
		flagSet.StringSliceVarP(&options.VarsFilePaths, "var-file-paths", "vfp", nil, "list of yaml file contained vars to inject into yaml input",                                   goflags.CommaSeparatedStringSliceOptions),
	)
```

##### FlagSet.CreateGroup 해석

[흐름]
target-format이라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Target-Format 그룹을 생성하고, 입력 파일(Input File)의 형식과 처리 방식에 관련된 CLI 옵션들을 하나의 그룹으로 등록.

flagSet.StringVarP(...)에서 -input-mode 옵션을 등록하고, FlagData 반환.
flagSet.BoolVarP(...)에서 -required-only 옵션을 등록하고, FlagData 반환.

아래 옵션들도 모두 등록됨.
-input-mode
-required-only
-skip-format-validation
-vars-text-templating
-var-file-paths

CreateGroup 호출 후, 모두 target-format 그룹으로 지정.

[의미]
** 이해를 돕고자 예를 들어 설명했습니다. **

1. 입력 파일의 형식을 지정한다. ( ex. nuclei -im list -> options.InputFileMode -> "list" )
2. 입력 파일에서 필수(required) 필드만 사용하여 요청을 생성한다. ( ex. nuclei -ro -> options.FormatUseRequiredOnly = true )
3. 입력 파일을 파싱할 때 형식 검사를 건너뛴다. ( ex. nuclei -sfv -> options.SkipFormatValidation = true )
4. YAML 입력 모드에서 변수(vars)를 텍스트 템플릿으로 처리한다. ( ex. nuclei -vtt -> options.VarsTextTemplating = true )
5. YAML 입력 파일에 주입할 변수 파일 목록을 지정한다. ( ex. nuclei -vfp vars1.yaml,vars2.yaml -> options.VarsFilePaths -> []string{"vars1.yaml", "vars2.yaml"} )

[CLI 옵션 등록]
-im, -ro, -sfv, -vtt, -vfp

[입력된 값을 options 구조체의 필드와 연결(바인딩) 한다.]
-im  → options.InputFileMode
-ro  → options.FormatUseRequiredOnly
-sfv → options.SkipFormatValidation
-vtt → options.VarsTextTemplating
-vfp → options.VarsFilePaths



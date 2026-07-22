---
상태: false
유형: Main_Function
상세: main.go readConfig 함수 개요
---
# readConfig 전체 구조 #전지성

## readConfig #전지성
```go
func readConfig() *goflags.FlagSet
```
> Nuclei에서 사용할 CLI 옵션을 등록하고 입력값·설정 파일·Template Profile을 병합하는 함수
- **매개변수 :** 없음
- **반환 타입 :** `*goflags.FlagSet` (포인터)
- **설명 :**
	- Target, Templates, Filtering, Output 등 CLI 옵션 그룹을 등록
	- `flagSet.Parse()`로 사용자 입력을 `options` 구조체와 연결
	- Config 파일과 Template Profile을 병합
	- 인증, Update, Inline Target, Secret 및 Resume 파일을 후처리
- **참조 :** [main_Varialbes](../../main_Varialbes.md), [pkg/types](../../../02_Internal_Packages/pkg_types.md), [goflags](../../../03_External_Packages/goflags.md), [sample](../../../03_External_Packages/04_etc/sample.md)

### 상세 분석 #전지성

#### 26.07.02 작업 기록

##### readConfig 함수 전체 구조
readConfig()
│
├── 변수 선언
│
├── FlagSet 생성
│
├── 프로그램 설명 설정
│
├── Target 그룹 생성
│
├── Target-Format 그룹 생성
│
├── Templates 그룹 생성
│
└── Filtering 그룹 생성

##### 외부 패키지로 분리
`FlagSet`, `FlagData`, Callback 구현은 [[../../../03_External_Packages/goflags|goflags]] 참고.

##### 변수 설명 분리
`main.go` 전역변수는 [[../../main_Varialbes]] 참고.

##### 내부 패키지로 분리
`types.Options` 구조체는 [[../../../02_Internal_Packages/pkg_types|pkg_types]] 참고.

##### 외부 패키지 API로 분리
`goflags.NewFlagSet()` 등의 API는 [[../../../03_External_Packages/goflags|goflags]] 참고.

```
func readConfig() *goflags.FlagSet { // 251~727줄

```

##### 함수 반환
반환 타입 `goflags.FlagSet`만 → `github.com/projectdiscovery/goflags`

```
	// readConfig 함수 안에서만 사용할 임시 변수들  
	// 실제 options 구조체에 바로 저장하지 않고,  
	// 후처리가 필요한 옵션 값을 잠깐 저장한다.
	
	// when true updates nuclei binary to latest version
	var updateNucleiBinary bool     // -update 옵션이 들어왔는지 저장
	var pdcpauth string             // -auth 옵션 입력값을 저장. -auth 옵션 값 저장. "true"이거나 API Key일 수 있음
	var fuzzFlag bool               // -fuzz 옵션을 위한 변수 저장(fuzz는 예전 버전임. 현재는 -dast를 권장). 

	flagSet := goflags.NewFlagSet() // pkg/mod/github.com/projectdiscovery/goflags@v0.1.74/callback_var.go
									// CLI(Command Line Interface)의 옵션 관리자(FlagSet 객체) 를 생성합니다.
									// -u, -l, -w, -jsonl, -update 같은 옵션들이 등록됨.
	flagSet.CaseSensitive = true    // 대소문자 구분.
	flagSet.SetDescription(`Nuclei is a fast, template based vulnerability scanner focusing on extensive configurability, massive extensibility and ease of use.`)
	// 위 코드는 readConfing 함수가 실행되고, 위에 코드들이 모두 동작 후, 메세지 출력.
	// 메세지 번역 -> ' Nuclear는 광범위한 구성 가능성, 대규모 확장성 및 사용 편의성에 중점을 둔 빠르고 템플릿 기반의 취약점 스캐너입니다.'
	// 따옴표가 아닌 백틱(``)을 사용한 이유는 여러 줄 작성 가능, \n 없이 줄바꿈이 유지되기 때문.
	
	/* TODO Important: The defined default values, especially for slice/array types are NOT DEFAULT VALUES, but rather implicit values to which the user input is appended.
	This can be very confusing and should be addressed
	*/

```

##### Slice 기본값 주석 해석
자세한 설명은 [[../../../03_External_Packages/04_etc/sample#Slice 옵션의 기본값|sample]] 참고.

##### CreateGroup 구현 분리
외부 패키지의 `CreateGroup()` 구현은 [[../../../03_External_Packages/goflags#CreateGroup 함수|goflags]] 참고.

## 세부 문서 목차

### CLI 옵션 그룹

- [01 Target](01_Target.md)
- [02 Target-Format](02_Target-Format.md)
- [03 Templates](03_Templates.md)
- [04 Filtering](04_Filtering.md)
- [05 Output](05_Output.md)
- [06 Configurations](06_Configurations.md)
- [07 Interactsh](07_Interactsh.md)
- [08 Fuzzing](08_Fuzzing.md)
- [09 Uncover](09_Uncover.md)
- [10 Rate-Limit](10_Rate-Limit.md)
- [11 Optimizations](11_Optimizations.md)
- [12 Headless](12_Headless.md)
- [13 Debug](13_Debug.md)
- [14 Update](14_Update.md)
- [15 Honeypot](15_Honeypot.md)
- [16 Statistics](16_Statistics.md)
- [17 Cloud](17_Cloud.md)
- [18 Authentication](18_Authentication.md)

### CLI 파싱 이후 처리

- [19 PostProcessing](19_PostProcessing.md)



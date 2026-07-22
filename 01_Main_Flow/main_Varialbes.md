---
상태: false
유형:
상세: main.go 변수들
---
# 전역변수 #main/globalVar

> `main.go`에 선언되어 프로그램 전체에서 사용하는 변수

## options #박영현
```go
	options                = &types.Options{}
```
> Nuclei 프로그램 전역의 환경 설정을 제어하는 전역변수
- **타입 :** `*types.Options` (포인터)
- **설명 :** 
	- `options`에 `types.Optoins` 인스턴스 주소를 할당
	- `types.go`에 정의된 [Option](../02_Internal_Packages/pkg_types.md#Options%20struct) 구조체를 기반으로 인스턴스를 만들고 그 주소값을 가르킴
	- 이 변수 내부에는 로그 출력을 담당하는 [gologger.Logger](../03_External_Packages/gologger.md#Logger%20Struct) 인스턴스의 주소 (`gologger.Logger`)도 포함됨
- **참조 :**  [pkg/types](../02_Internal_Packages/pkg_types.md#types.go), [gologger](../03_External_Packages/gologger.md)

## inlineSecretsTempFiles #박영현 
```go
inlineSecretsTempFiles []string
```
> Template Profile의 Inline Secret을 처리하며 만든 임시 파일 경로 목록
- **타입 :** `[]string` (문자열)
- **설명 :** `readConfig()`가 생성한 임시 Secret 파일을 기록하고, `main()` 종료 시 `defer` 구문에서 각 파일을 삭제할 때 사용한다.
# 지역변수 #main/localVar

# 구조체 #main/struct

## profileSecrets #전지성
```go
type profileSecrets struct {
	Secrets interface{} `yaml:"secrets"`
}
```
> Template Profile YAML에서 `secrets` 항목만 꺼내기 위한 보조 구조체

- **사용 위치 :** `processInlineSecretsFromProfile()`
- **역할 :** Profile 전체를 별도 구조체로 만들지 않고 `secrets` 항목만 임시로 해석
- **결과 :** `Secrets`가 비어 있지 않으면 별도의 임시 Secret YAML 파일을 만드는 데 사용
- **참조 :** [processInlineSecretsFromProfile](main_Functions/processInlineSecretsFromProfile.md)

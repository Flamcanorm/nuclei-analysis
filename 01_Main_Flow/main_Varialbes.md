---
상태: false
유형:
상세: main.go 변수들
---
# 전역변수 #main/globalVar 

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

# 지역변수 #main/localVar




---
상태: false
유형:
---
# 전역변수 #main/globalVar 

## options #박영현
```go
	options                = &types.Options{}
```
> `options`에 `types.Optoins` 인스턴스 주소를 할당합니다.
- **타입 :** `*types.Options` (포인터)
- **설명 :** Nuclei 프로그램 전역의 환경 설정을 제어하는 전역변수
	- `types.go`에 정의된 [types.Option](pkg_types#type%20Options%20struct) 구조체를 기반으로 인스턴스를 만들고 그 주소값을 가르킵니다.
	- 이 변수 내부에는 로그 출력을 담당하는 [gologger](../03_External_Packages/gologger.md) 인스턴스의 주소 (`gologger.Logger`)도 포함되어 있습니다.
- **참조 :** [types.Options](pkg_types#type%20Options%20struct), [gologger](../03_External_Packages/gologger.md)

# 지역변수 #main/localVar




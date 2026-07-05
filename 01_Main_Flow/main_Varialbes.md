---
상태: false
유형:
---
# 전역변수 #main/globalVar

## options 
```go
	options                = &types.Options{}
```
> `options`에 `types.Optoins` 인스턴스 주소를 할당합니다.
- **타입 :** `*types.Options` (포인터)
- **설명 :** Nuclei 프로그램 전역의 환경 설정을 제어하는 전역변수
	- `types.go`에 정의된 [[pkg_types#type Options struct|types.Options]] 구조체를 기반으로 인스턴스를 만들고 그 주소값을 가르킵니다.
	- 이 변수 내부에는 로그 출력을 담당하는 [[gologger]] 인스턴스의 주소 (`gologger.Logger`)도 포함되어 있습니다.
- **참조 :** [[pkg_types#type Options struct|types.Options]], [[gologger]]


# 지역변수 #main/localVar


# 구조체 #main/struct
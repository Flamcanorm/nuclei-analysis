---
상태: false
유형:
상세: Nuclei 전체 실행 흐름
---
# L58 
#박영현 #L58 
```go
options.Logger = gologger.DefaultLogger
```
> 로깅 인스턴스 설정

- **설명 :** 
	- `options` 는 Nuclei의 옵션을 설정하는 구조체로 `options.Logger` 필드에 외부 패키지 `gologger` 에서 정의되고 기본값으로 초기화 된 `gologger.DefaultLogger` 를 저장

**참조 :** [options.Logger](../02_Internal_Packages/pkg_types/types.go.md#Logger), [gologger.DefaultLogger](../03_External_Packages/gologger/gologger.go.md#DefaultLogger)

# L60
#박영현 #L60 
```go
defer func() {
		for _, f := range inlineSecretsTempFiles {
			_ = os.Remove(f)
		}
	}()
```
> 검사 과정에서 임시로 생성된 중요 정보를 검사가 끝나고 모두 삭제하는 함수

- **매개변수 :** 없음
- **반환 타입 :** 없음
- **형태 :** `defer` + 익명 함수 즉시 실행 (`func(){}()`)
- **설명 :** 
	- `defer` 는 이 코드를 감싸고 있는 상위 함수가 종료되어 스택 메모리가 반환되기 직전에 내부 코드를 실행하도록 예약하는 키워드
	- 익명 함수 끝에 `()`가 붙은 `func(){}()` 형태는 함수를 정의함과 동시에 즉시 호출함
	- 따라서 코드는 익명 함수를 즉시 호출하여 `defer` 키워드로 실행 예약을 해두고 함수 내부의 `for`문 코드는 상위 함수가 완전히 종료되면 실행된다.
	- `f`에 [inlineSecretsTempFiles](main_Varialbes.md#inlineSecretsTempFiles) 문자열 값을 하나씩 불러와 삭제함 
	- `os`는 Go 언어의 표준 라이브러리로 `os.Remove()`는 괄호 안에 있는 경로의 파일이나 폴더를 하드디스크에서 삭제하는 명령어


# L67
#박영현 #L67
```go
// enables CLI specific configs mostly interactive behavior
	config.CurrentAppMode = config.AppModeCLI
```
>
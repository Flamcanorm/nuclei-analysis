---
상태: false
유형:
상세: Nuclei 전체 실행 흐름
---
# L58 #박영현 #L58 
```go
options.Logger = gologger.DefaultLogger
```
> `options` 는 Nuclei의 옵션을 설정하는 구조체로 `options.Logger` 필드에 외부 패키지 `gologger` 에서 정의되고 기본값으로 초기화 된 `gologger.DefaultLogger` 를 저장

**참조 :** [options.Logger](../02_Internal_Packages/pkg_types.md#Logger), [gologger.DefaultLogger](../03_External_Packages/gologger.md#DefaultLogger)

# L60 #박영현 #L60 
```go
defer func() {
		for _, f := range inlineSecretsTempFiles {
			_ = os.Remove(f)
		}
	}()
```
> 검사 과정에서 임시로 생성된 중요 정보 파일을 검사가 끝나면 모두 삭제하는 함수

- **매개변수 :** 없음
- **반환 타입 :** 없음
- **형태 :** `defer` + 익명 함수 즉시 실행 (`func(){}()`)
- **설명 :** 
	- `defer` 는 이 코드를 감싸고 있는 상위 함수가 종료되어 스택 메모리가 반환되기 직전에 내부 코드를 실행하도록 예약하는 키워드
	- 익명 함수 끝에 `()`가 붙은 `func(){}()` 형태는 함수를 정의함과 동시에 즉시 호출함
	- 따라서 코드는 익명 함수를 즉시 호출하여 `defer` 키워드로 실행 예약을 해두고 함수 내부의 `for`문 코드는 상위 함수가 완전히 종료되면 실행된다.
	- `f`에 [inlineSecretsTempFiles](main_Varialbes.md#inlineSecretsTempFiles)의 문자열 값을 하나씩 불러와 삭제함
	- `os`는 Go 언어의 표준 라이브러리로 `os.Remove()`는 괄호 안에 있는 경로의 파일이나 폴더를 하드디스크에서 삭제하는 명령어

# L67 #박영현 #L67
```go
// enables CLI specific configs mostly interactive behavior
	config.CurrentAppMode = config.AppModeCLI
```
> Nuclei의 실행 모드를 CLI로 지정

**참조 :** [CurrentAppMode](../02_Internal_Packages/pkg_catalog/config/constans.go.md#CurrentAppMode)

# L69 #박영현 #L69
```go
if err := runner.ConfigureOptions(); err != nil {
	options.Logger.Fatal().Msgf("Could not initialize options: %s\n", err)
}
_ = readConfig()
```
> Nuclei 실행 설정을 초기화하고, 오류가 발생하면 `Fatal` 메시지를 출력한 뒤 프로그램을 즉시 종료한다.
>
> 오류가 발생하지 않으면 `readConfig()`로 설정값과 CLI 플래그를 읽는다. `_ = readConfig()`는 반환값을 사용하지 않는다는 뜻이며, 설정 처리 중 발생하는 오류에 대한 대응은 `readConfig()` 내부에 정의되어 있다.

#미완성 `readConfig()`, `internal/runner`

# L177 #main/runner
```go
nucleiRunner, err := runner.New(options)
if err != nil {
	options.Logger.Fatal().Msgf("Could not create runner: %s\n", err)
}
if nucleiRunner == nil {
	return
}
```
> CLI 입력이 저장된 `options`를 전달하여 실제 스캔 실행을 관리할 Runner 객체를 생성한다.

- **입력값 :** `options` — 대상, 템플릿, 태그, 심각도, 출력 파일, 동시 실행 수 등의 설정
- **반환값 :**
	- `nucleiRunner` — 초기화가 끝난 `Runner` 객체
	- `err` — 생성 도중 발생한 오류
- **설명 :**
	- `runner.New(options)`는 Options를 Runner에 연결한다.
	- Catalog, 입력 공급자, 출력, 진행률, 속도 제한기 등 실행에 필요한 구성요소를 초기화한다.
	- 오류가 있으면 `Fatal` 로그를 출력하고 프로그램을 종료한다.
	- Runner가 `nil`이면 더 이상 실행할 대상이 없으므로 `main()`을 끝낸다.
- **참조 :** [runner_runner](../02_Internal_Packages/runner_runner.md), [pkg_types](../02_Internal_Packages/pkg_types.md)

# L237 #main/runner
```go
if err := nucleiRunner.RunEnumeration(); err != nil {
	if options.Validate {
		options.Logger.Fatal().Msgf("Could not validate templates: %s\n", err)
	} else {
		options.Logger.Fatal().Msgf("Could not run nuclei: %s\n", err)
	}
}
nucleiRunner.Close()
```
> 준비가 끝난 Runner로 실제 템플릿 스캔을 시작하고, 실행이 끝나면 사용한 자원을 정리한다.

- **`RunEnumeration()` :** Options에 맞는 템플릿을 불러오고 Engine을 생성하여 스캔을 실행
- **`Close()` :** 임시 폴더, 브라우저, 입력 공급자, 출력 Writer, Cache 등 실행 중 사용한 자원을 정리
- **오류 처리 :** 검증 모드이면 템플릿 검증 오류로, 일반 실행이면 Nuclei 실행 오류로 구분하여 출력
- **참조 :** [runner_runner](../02_Internal_Packages/runner_runner.md), [core_engine](../02_Internal_Packages/core_engine.md)

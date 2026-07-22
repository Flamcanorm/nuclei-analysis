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

**참조 :** [readConfig](main_Functions/readConfig/README.md), [runner_options](../02_Internal_Packages/runner_options.md)

# L74 #전지성 #main/dsl
```go
if options.ListDslSignatures {
	options.Logger.Info().Msgf("The available custom DSL functions are:")
	fmt.Println(dsl.GetPrintableDslFunctionSignatures(options.NoColor))
	return
}
```
> `-list-dsl-function` 계열 옵션이 활성화되면 템플릿에서 사용할 수 있는 DSL 함수 목록을 출력하고 스캔 없이 `main()`을 종료한다.

# L80 #전지성 #main/template-sign
```go
if options.SignTemplates {
	// 템플릿 서명 처리
	return
}
```
> `-sign` 옵션을 사용했을 때 YAML 템플릿을 찾아 전자서명을 추가하는 분기다. 서명 성공·실패 수를 출력한 뒤 일반 스캔은 실행하지 않는다.

# L118 #전지성 #main/profiling
```go
if memProfile != "" {
	// Memory, CPU, Trace 파일 생성 및 기록
}
```
> 개발자가 성능 문제를 분석할 때 사용할 Memory Profile, CPU Profile, 실행 Trace를 기록한다. 일반적인 스캔에서는 `memProfile`이 비어 있으므로 실행되지 않는다.

# L166 #전지성 #main/execution-id
```go
options.ExecutionId = xid.New().String()
```
> 이번 실행을 구별할 고유 문자열을 생성해 Options에 저장한다. 여러 실행의 상태·통계·네트워크 자원을 서로 구분할 때 사용한다.

# L168 #전지성 #main/options
```go
runner.ParseOptions(options)
```
> `readConfig()`가 입력값을 저장한 뒤, 실제 실행 전에 옵션 조합을 검사하고 필요한 기본값·Logger 설정 등을 보완한다.

# L170 #전지성 #main/cloud-upload
```go
if options.ScanUploadFile != "" {
	if err := runner.UploadResultsToCloud(options); err != nil {
		options.Logger.Fatal().Msgf("could not upload scan results to cloud dashboard: %s\n", err)
	}
	return
}
```
> 기존 결과 파일을 Cloud에 업로드하는 모드다. 파일을 업로드한 뒤 일반 스캔을 실행하지 않고 종료한다.

# L177 #전지성 #main/runner 
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

# L185 #전지성 #main/hang-monitor
```go
if options.HangMonitor {
	stackMonitor := monitor.NewStackMonitor()
	cancel := stackMonitor.Start(10 * time.Second)
	defer cancel()
	// 멈춤 감지 시 Runner 종료와 Resume 파일 저장
}
```
> 실행이 비정상적으로 멈춘 상태를 감시한다. 멈춤이 감지되면 현재 Runner를 닫고 이어서 실행할 수 있는 Resume 파일을 만든다.

# L204 #전지성 #main/graceful-shutdown
```go
resumeFileName := types.DefaultResumeFilePath()
c := make(chan os.Signal, 1)
signal.Notify(c, os.Interrupt)
go func() {
	<-c
	nucleiRunner.Close()
	// 필요한 경우 Resume 파일 저장
	os.Exit(1)
}()
```
> 사용자가 `Ctrl+C`를 누를 때 작업을 즉시 끊어 자원을 남기지 않도록 종료 신호를 받는다. Runner를 정리하고 설정에 따라 Resume 파일을 저장한 뒤 종료한다.

# L237 #전지성 #main/runner
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

# L245 #전지성 #main/resume-cleanup
```go
if fileutil.FileExists(resumeFileName) {
	_ = os.Remove(resumeFileName)
}
```
> 스캔이 정상적으로 끝났다면 더 이상 이어서 실행할 필요가 없으므로 남아 있는 Resume 파일을 삭제한다.

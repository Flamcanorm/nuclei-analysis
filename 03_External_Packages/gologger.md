---
유형: External_Pack
상태: false
상세: 로그 출력 패키지
---
# gologger.go #ExternalPackages/gologger
> [gologger package code](https://github.com/projectdiscovery/gologger/blob/main/gologger.go)

## var #ExtenalPackages/gologger/var

### DefaultLogger #박영현
```go
// DefaultLogger is the default logging instance
	DefaultLogger *Logger
```
> 기본 로깅 인스턴스로 init() 함수에서 초기화됨
- **타입 :** [*Logger](gologger.md#Logger%20struct) (포인터)
- **설명 :** 
	- gologger 패키지의 기본 로깅 인스턴스로, 패키지가 로드될 때 [init()](gologger.md#init) 함수 내에서 자동으로 초기화되어 사용할 수 있는 전역 로거 변수



## Logger struct #ExternalPackages/gologger/struct
```go
// Logger is a logger for logging structured data in a beautiful and fast manner.
type Logger struct
```
> 출력 대상, 레벨, 포맷 등 로그의 설정 정보를 담는 구조체


## func #ExternalPackages/gologger/func

### init #박영현
```go
func init() {
	DefaultLogger = &Logger{}
	DefaultLogger.SetMaxLevel(levels.LevelInfo)
	DefaultLogger.SetFormatter(formatter.NewCLI(false))
	DefaultLogger.SetWriter(writer.NewCLI())
}
```
> 기본 로깅 인스턴스인 [DefaultLogger](gologger.md#DefaultLogger) 를 초기화시킴
- **매개변수 :** 없음
- **반환 타입 :** 없음
- **형태 :** 패키지 초기화 함수
- **설명 :** 
	- `gologger` 패키지가 로드될 때 Go 런타임에 의해 자동으로 딱 한 번만 실행된다
	- main 함수보다 먼저 실행됨

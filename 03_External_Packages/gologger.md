---
유형: External_Pack
상태: false
상세: 로그 출력 패키지
---
# gologger.go #ExternalPackages/gologger
> [gologger.go code](https://github.com/projectdiscovery/gologger/blob/main/gologger.go

## var #ExtenalPackages/gologger/var

### DefaultLogger #박영현
```go
// DefaultLogger is the default logging instance
	DefaultLogger *Logger
```
> 기본 로깅 인스턴스로 init() 함수에서 초기화됨
- **타입 :** [Logger](gologger.md#Logger%20struct) (포인터)
- **설명 :** 
	- gologger 패키지의 기본 로깅 인스턴스로, 패키지가 로드될 때 [init()](gologger.md#init) 함수 내에서 자동으로 초기화되어 사용할 수 있는 전역 로거 변수

### labels #박영현 
```go
labels = map[levels.Level]string{
		levels.LevelFatal:   "FTL",
		levels.LevelError:   "ERR",
		levels.LevelInfo:    "INF",
		levels.LevelWarning: "WRN",
		levels.LevelDebug:   "DBG",
		levels.LevelVerbose: "VER",
	}
```
> 로깅 레벨 문자열과 매핑

- **타입 :** `map[levels.Level]string` 
- **설명 :** 
	- [levels.go](gologger.md#levels.go) 에 정의된 로깅 레벨을 출력할 때 약어 형태로 출력하기 위한 매핑 변수
	- *예시) `labels[levels.LevelInfo] // 결과 : "INF"`
- **참조 :** [levels.go](gologger.md#levels.go)


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
	- [Logger](gologger.md#Logger%20struct) 인스턴스를 생성하고 주소를 [DefaultLogger](gologger.md#DefaultLogger)에 저장
	- 


### SetMaxLevel #박영현 
```go
// SetMaxLevel sets the max logging level for logger
func (l *Logger) SetMaxLevel(level levels.Level) {
	l.maxLevel = level
}

```
> 





# levels.go #ExternalPackages/gologger
> [levels.go code](https://github.com/projectdiscovery/gologger/blob/main/levels/levels.go)
> 로깅 레벨이 정의된 코드

```go
// Level defines all the available levels we can log at
type Level int
```
> `int` 크기의 새로운 데이터 타입 `Level` 정의


```go
// Available logging levels
const (
	LevelFatal Level = iota
	LevelSilent
	LevelError
	LevelInfo
	LevelWarning
	LevelDebug
	LevelVerbose
)
```
> 각 로깅 레벨별로 정수 숫자 상수로 지정
- **타입 :** `Level` 타입 상수
- **설명 :** `iota` 키워드를 사용해 `LevelFatal` 은 `Level` 타입으로 숫자 0, `LevelSilent`는 1, `LevelError`는 2, ... 각 레벨 차례로 정의

```go
// String returns the string representation of a log level
func (l Level) String() string {
	return [...]string{"fatal", "silent", "error", "info", "warning", "debug", "verbose"}[l]
}
```
>로그 레벨을 화면에 출력할 때 문자로 출력하기 위한 메서드

- **매개변수 :** 없음
- **반환 타입 :** `string`
- **형태 :** 메서드
- **설명 :** 
	- `Level`타입의 메서드를 정의하고 리시버 이름으로 `l` 사용
	- 호출될 때 값을 `l`에 복사하여 메서드 내부로 전달
	- 리턴 값으로 배열의 `l`번째 인덱스에 해당하는 `stirng` 값 리턴
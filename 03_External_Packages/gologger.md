---
유형: External_Pack
상태: false
상세: 로그 출력 패키지
---
# gologger.go #ExternalPackages/gologger
> [gologger.go code](https://github.com/projectdiscovery/gologger/blob/main/gologger.go)

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
> 로그 레벨 문자열과 매핑

- **타입 :** `map[levels.Level]string` 
- **설명 :** 
	- [levels.go](gologger.md#levels.go) 에 정의된 로깅 레벨을 출력할 때 약어 형태로 출력하기 위한 매핑 변수
	- *예시) `labels[levels.LevelInfo] // 결과 : "INF"`
- **참조 :** [levels.go](gologger.md#levels.go)


## type #ExternalPackages/gologger/type

### Logger struct
```go
// Logger is a logger for logging structured data in a beautiful and fast manner.
type Logger struct
```
> 출력 대상, 레벨, 포맷 등 로그의 설정 정보를 담는 구조체

```go
	writer            writer.Writer
	maxLevel          levels.Level
	formatter         formatter.Formatter
	timestampMinLevel levels.Level
	timestamp         bool
	timestampFormat   string
	groupPrefix       string      // For slog group support
	persistedAttrs    []slog.Attr // For slog WithAttrs support
```
#미완성 각 멤버 변수 설명 필요

#### maxLevel #박영현
```go
maxLevel          levels.Level
```
> 로그 최대 허용 레벨을 저장하는 멤버 변수 ([levels.go](gologger.md#levels.go) 에 정의됨)


#### formatter #박영현 
```go
formatter         formatter.Formatter
```
> 로그 출력 인터페이스 ([Formatter interface](gologger.md#Formatter%20interface))


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
	- `DefaultLogger = &Logger{}` : [Logger](gologger.md#Logger%20struct) 인스턴스를 생성하고 주소를 [DefaultLogger](gologger.md#DefaultLogger)에 저장
	- `DefaultLogger.SetMaxLevel(levels.LevelInfo)` : [SetMaxLevel](gologger.md#SetMaxLevel) 매서드를 호출하여 `Default` 로거의 `maxLevel` 변경
	- `DefaultLogger.SetFormatter(formatter.NewCLI(false))` : [SetFormatter](gologger.md#SetFormatter) 메서드를 호출하여 


### SetMaxLevel #박영현 
```go
// SetMaxLevel sets the max logging level for logger
func (l *Logger) SetMaxLevel(level levels.Level) {
	l.maxLevel = level
}

```
> 레벨 값을 입력받아 로거 인스턴스의 `maxLevel` 값을 변경하는 매서드

- **매개 변수 :** `level` ([levels.Level](gologger.md#levels.go) 타입)
- **반환 타입 :** 없음
- **형태 :** 매서드
- **설명 :** 레벨 값을 입력받아 로거 인스턴스의 `maxLevel` 값을 변경하는 매서드

### SetFormatter #박영현
```go
// SetFormatter sets the formatter instance for a logger
func (l *Logger) SetFormatter(formatter formatter.Formatter) {
	l.formatter = formatter
}
```
> #미완성 gologger/formatter 


# levels.go #ExternalPackages/gologger
> [levels.go code](https://github.com/projectdiscovery/gologger/blob/main/levels/levels.go)
> 로그 레벨이 정의된 코드

## type #ExternalPackages/gologger/type

### Level type #박영현 
```go
// Level defines all the available levels we can log at
type Level int
```
> `int` 크기의 새로운 데이터 타입 `Level` 정의


## var #ExternalPackages/gologger/var
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
> 각 로그 레벨별로 정수 숫자 상수로 지정
- **타입 :** `Level` 타입 상수
- **설명 :** `iota` 키워드를 사용해 `LevelFatal` 은 `Level` 타입으로 숫자 0, `LevelSilent`는 1, `LevelError`는 2, ... 각 레벨 차례로 정의


## func #Externalpackages/gologger/func
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


# formatter.go #ExternalPackages/gologger 
> [formatter.go code](https://github.com/projectdiscovery/gologger/blob/main/formatter/formatter.go)
> 로그 데이터 출력 전 특정 포맷으로 변환하는 기능을 정의 (인터페이스)
> CLI, JSON 등 다양한 로그 출력 포맷을 유연하게 교체하여 사용할 수 있도록 해줌

## type #ExternalPackages/gologger/type 

### Formatter interface #박영현 
```go
// Formatter type format raw logging data into something useful
type Formatter interface {
	// Format formats the log event data into bytes
	Format(event *LogEvent) ([]byte, error)
}
```
> 로그를 출력할 포맷 인터페이스

- **설명 :** 
	- 어떤 구조체가 `Format(event *LogEvent) ([]byte, error)` 형태의 매서드를 가진 경우 Formatter 인터페이스에 해당 (묵시적)
	- `Format` 메서드는 로그 이벤트 데이터를 바이트로 변환하는 메서드

### LogEvent struct #박영현 
```go
// LogEvent is the representation of a single event to be logged.
type LogEvent struct {
	Message  string
	Level    levels.Level
	Metadata map[string]string
}
```
> 로그 이벤트 정보를 정의한 구조체


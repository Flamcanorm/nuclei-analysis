---
유형: External_Pack
상태: false
상세: 로그 출력 패키지
---
# gologger.go
#ExternalPackages/gologger
> [gologger.go code](https://github.com/projectdiscovery/gologger/blob/main/gologger.go)

## var
#ExtenalPackages/gologger/var

### DefaultLogger
#박영현
```go
// DefaultLogger is the default logging instance
	DefaultLogger *Logger
```
> 기본 로깅 인스턴스로 init() 함수에서 초기화됨
- **타입 :** [Logger](gologger.go.md#Logger%20struct) (포인터)
- **설명 :** 
	- gologger 패키지의 기본 로깅 인스턴스로, 패키지가 로드될 때 [init()](gologger.go.md#init) 함수 내에서 자동으로 초기화되어 사용할 수 있는 전역 로거 변수

### labels
#박영현 
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
	- [levels.go](levels/levels.go.md) 에 정의된 로깅 레벨을 출력할 때 약어 형태로 출력하기 위한 매핑 변수
	- *예시) `labels[levels.LevelInfo] // 결과 : "INF"`
- **참조 :** [levels.go](levels/levels.go.md)


## type
#ExternalPackages/gologger/type

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

#### maxLevel
#박영현
```go
maxLevel          levels.Level
```
> 로그 최대 허용 레벨을 저장하는 멤버 변수 ([levels.go](levels/levels.go.md) 에 정의됨)


#### formatter
#박영현 
```go
formatter         formatter.Formatter
```
> 로그 출력 인터페이스 ([Formatter interface](formatter/formatter.go.md#Formatter%20interface))


## func 
#ExternalPackages/gologger/func

### init 
#박영현
```go
func init() {
	DefaultLogger = &Logger{}
	DefaultLogger.SetMaxLevel(levels.LevelInfo)
	DefaultLogger.SetFormatter(formatter.NewCLI(false))
	DefaultLogger.SetWriter(writer.NewCLI())
}
```
> 기본 로깅 인스턴스인 [DefaultLogger](gologger.go.md#DefaultLogger) 를 초기화시킴
- **매개변수 :** 없음
- **반환 타입 :** 없음
- **형태 :** 패키지 초기화 함수
- **설명 :** 
	- `gologger` 패키지가 로드될 때 Go 런타임에 의해 자동으로 딱 한 번만 실행된다
	- main 함수보다 먼저 실행됨
	- `DefaultLogger = &Logger{}` : [Logger](gologger.go.md#Logger%20struct) 인스턴스를 생성하고 주소를 [DefaultLogger](gologger.go.md#DefaultLogger)에 저장
	- `DefaultLogger.SetMaxLevel(levels.LevelInfo)` : [SetMaxLevel](gologger.go.md#SetMaxLevel) 매서드를 호출하여 `Default` 로거의 `maxLevel` 변경
	- `DefaultLogger.SetFormatter(formatter.NewCLI(false))` : [SetFormatter](gologger.go.md#SetFormatter) 메서드를 호출하여 #미완성 gologger/formatter/cli.go 


### SetMaxLevel 
#박영현 
```go
// SetMaxLevel sets the max logging level for logger
func (l *Logger) SetMaxLevel(level levels.Level) {
	l.maxLevel = level
}

```
> 레벨 값을 입력받아 로거 인스턴스의 `maxLevel` 값을 변경하는 매서드

- **매개 변수 :** `level` ([levels.Level](levels/levels.go.md#levels.go) 타입)
- **반환 타입 :** 없음
- **형태 :** 매서드
- **설명 :** 레벨 값을 입력받아 로거 인스턴스의 `maxLevel` 값을 변경하는 매서드

### SetFormatter 
#박영현
```go
// SetFormatter sets the formatter instance for a logger
func (l *Logger) SetFormatter(formatter formatter.Formatter) {
	l.formatter = formatter
}
```
> `Logger`가 사용할 로그 포맷을 설정하는 메서드

- **매개 변수 :** formatter ([Formatter interface](formatter/formatter.go.md#Formatter%20interface) 타입)
- **반환 타입 :** 없음
- **형태 :** 메서드
- **설명 :** 
	- `Formatter` 인터페이스에 해당하는 `formatter`를 매개 변수로 받아 `l`에 저장된 [Logger](gologger.go.md#Logger%20struct) 포인터의 `formatter` 에 저장
	- 로그를 출력할 포맷 형식에 맞게 포맷을 정할 수 있음





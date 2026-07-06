# levels.go 
#ExternalPackages/gologger
> [levels.go code](https://github.com/projectdiscovery/gologger/blob/main/levels/levels.go)
> 로그 레벨이 정의된 코드

## type 
#ExternalPackages/gologger/type

### Level type 
#박영현 
```go
// Level defines all the available levels we can log at
type Level int
```
> `int` 크기의 새로운 데이터 타입 `Level` 정의


## var 
#ExternalPackages/gologger/var
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


## func 
#Externalpackages/gologger/func
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






---
유형: External_Pack
상태: false
상세: readConfig에서 사용하는 projectdiscovery/goflags
---
# goflags #external/goflags

> `readConfig()`에서 CLI 옵션을 등록하고 파싱할 때 사용하는 외부 패키지
- **패키지 :** `github.com/projectdiscovery/goflags`
- **설명 :** Flag 등록, 그룹화, Callback 실행, CLI 파싱과 Config 병합을 담당
- **참조 :** [main_Functions](../01_Main_Flow/main_Functions.md)

## FlagSet, groupData, FlagData #전지성
```go
// FlagSet is a list of flags for an application

type FlagSet struct {

    CaseSensitive  bool

    Marshal        bool

    description    string

    customHelpText string

    flagKeys       InsertionOrderedMap

    groups         []groupData

    CommandLine    *flag.FlagSet

    configFilePath string

  

    // OtherOptionsGroupName is the name for all flags not in a group

    OtherOptionsGroupName string

    configOnlyKeys        InsertionOrderedMap

}

  

type groupData struct {

    name        string

    description string

}

  

type FlagData struct {

    usage        string

    short        string

    long         string

    group        string // unused unless set later

    defaultValue interface{}

    skipMarshal  bool

    field        flag.Value

}
```
> CLI Flag 전체와 각 Flag의 메타데이터를 저장하는 구조체
- **타입 :** struct
- **위치 :** $structPath
- **설명 :** FlagSet은 전체 옵션 집합을, FlagData는 개별 옵션의 이름·설명·기본값·저장 필드를 관리

## Callback Flag #전지성
```go
package goflags

  

import (

    "fmt"

    "strconv"                                                           **// go 표준 라이브러리. 문자열과 다른 자료형을 서로 변환하는 라이브러리임. **

)

  

// CallBackFunc

type CallBackFunc func()                                          **// 함수 타입(Function Type)을 정의**

  

// callBackVar

type callBackVar struct {

    Value CallBackFunc                                              **// CallBackFunc 타입**

}

  

// Set

func (c *callBackVar) Set(s string) error {

    v, err := strconv.ParseBool(s)                                   ** // v는 false or true, err는 nil **

    if err != nil {  ** // 오류가 발생했다면 **

        return fmt.Errorf("failed to parse callback flag")       ** //fmt.Errorf는 fmt 패키지 함수. 함수를 종료하고 error를 반환함 **

    }

    if v {                                                                  ** // v가 ture라면 **

        // if flag found execute callback

        c.Value()                                                         ** // 구조체의 Value를 실행 **

    }

    return nil                                                             ** // 오류가 없다 **

}

  

// IsBoolFlag

func (c *callBackVar) IsBoolFlag() bool {

    return true                                                        ** // IsBoolFlag 함수는 항상 true **

}

  

// String

func (c *callBackVar) String() string {

    return "false"                                                   ** // String함수는 항상 false **

}

  

// CallbackVar adds a Callback flag with a longname (긴 이름(long name)만 가진 Callback 플래그를 추가한다.)

func (flagSet *FlagSet) CallbackVar(callback CallBackFunc, long string, usage string) *FlagData {  // falg를 호출하는 함수

    return flagSet.CallbackVarP(callback, long, "", usage)  // CallbackVarP 함수 호출

}

  

// CallbackVarP adds a Callback flag with a shortname and longname

func (flagSet *FlagSet) CallbackVarP(callback CallBackFunc, long, short string, usage string) *FlagData {  // 콜백옵션을 등록하는 함수

    if callback == nil { // callback 함수가 존재하지 않다면

        panic(fmt.Errorf("callback cannot be nil for flag -%v", long)) // fmt.Errorof으로 반환 후, 복구할 수 없는 오류가 발생으로, 프로그램 즉시 종료

    }

    flagData := &FlagData{

        usage:        usage,

        long:         long,

        defaultValue: strconv.FormatBool(false),

        field:        &callBackVar{Value: callback},

        skipMarshal: true,

    }

    if short != "" {

        flagData.short = short

        flagSet.CommandLine.Var(flagData.field, short, usage)

        flagSet.flagKeys.Set(short, flagData)

    }

    flagSet.CommandLine.Var(flagData.field, long, usage) // 명시되있지 않지만, flag라는 go언어 표준 패키지 사용함(-u, -json, -debug 같은 것을 flag(옵션)이라 부름).
											// var는 옵션을 등록하는 함수
											// commendLine은 프로그램을 실행할 때, 터미널에서 입력한 명령어 한줄 전체를 파싱 대상(flagSet)으로 삼겠다는 뜻.

    flagSet.flagKeys.Set(long, flagData) // [Key : Value]를 사용하는 자료구조. Key만 입력해서 정보를 빨리 찾기 위해서 쓰임.

    return flagData

}
```
> Boolean Flag가 활성화됐을 때 지정된 함수를 실행하는 Callback 구현
- **타입 :** CallBackFunc, callBackVar
- **위치 :** $callbackPath
- **설명 :** Set()이 입력값을 Boolean으로 변환하고 	rue이면 등록된 Callback을 실행

## readConfig에서 사용하는 API #전지성
> `readConfig()`가 FlagSet을 구성할 때 호출하는 goflags API
- **설명 :**
  - goflags.NewFlagSet()으로 FlagSet 생성
  - CaseSensitive로 대소문자 구분 설정
  - SetDescription()으로 프로그램 설명 등록

`goflags.NewFlagSet()`, `.CaseSensitive`, `.SetDescription(...)` → 전부 **`github.com/projectdiscovery/goflags`** 패키지 API
실제 대입되는 값/문자열 내용은 main.go에서 작성

## CreateGroup #전지성
```go
func (flagSet *FlagSet) CreateGroup(groupName, description string, flags ...*FlagData) { // 매개변수(그룹내부 이름, 그룹의 표시이름). ...(가변인자)로,FlagData를 몇개든 받겠다는 뜻

    flagSet.SetGroup(groupName, description) // 그룹생성

    for _, currentFlag := range flags { // 들어온 모든 옵션 하나씩 꺼냄 ex. 목록에 Targets, TargetFile, Resume이 있다면,  currentFlag에 Targets, TargetFile, Resume을 하나씩 저장.

        currentFlag.Group(groupName)

    }

}
```
> 여러 FlagData를 하나의 CLI 도움말 그룹으로 묶는 함수
- **매개변수 :** groupName, description, lags ...*FlagData
- **반환 타입 :** 없음
- **위치 :** $createPath
- **설명 :** SetGroup()으로 그룹을 만든 뒤 모든 Flag에 같은 그룹 이름을 지정
- **참조 :** [main_Functions](../01_Main_Flow/main_Functions.md)

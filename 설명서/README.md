---
유형: 설명서
상태: false
상세: nuclei-analysis 전체 문서를 처음 읽는 사람을 위한 시작점
---

# Nuclei 코드 분석 설명서 #전지성

<!-- main-line-tags:start -->
> [!info]- 관련 main.go 줄 태그
> 이 문서가 직접 설명하거나 다음 단계로 연결하는 main.go 줄이다. 태그를 누르면 같은 줄을 다루는 다른 설명서도 함께 찾을 수 있다.
>
> #L58 #L60 #L67 #L69 #L74 #L80 #L118 #L166 #L168 #L170 #L177 #L185 #L204 #L237 #L245
<!-- main-line-tags:end -->

<!-- glossary-links:start -->
> [!info]- 명칭 참조
> 이 문서에서 사용하는 공통 명칭은 아래 링크를 눌러 명칭 사전에서 확인할 수 있다.
>
> [원본 소스](<../03_External_Packages/04_etc/sample.md#원본 소스>) · [패키지](<../03_External_Packages/04_etc/sample.md#패키지>) · [외부 라이브러리 또는 외부 패키지](<../03_External_Packages/04_etc/sample.md#외부 라이브러리 또는 외부 패키지>) · [내부 패키지](<../03_External_Packages/04_etc/sample.md#내부 패키지>) · [실행 파일](<../03_External_Packages/04_etc/sample.md#실행 파일>) · [선언](<../03_External_Packages/04_etc/sample.md#선언>) · [정의](<../03_External_Packages/04_etc/sample.md#정의>) · [구조체](<../03_External_Packages/04_etc/sample.md#구조체>)
> [객체 또는 인스턴스](<../03_External_Packages/04_etc/sample.md#객체 또는 인스턴스>) · [변수](<../03_External_Packages/04_etc/sample.md#변수>) · [필드](<../03_External_Packages/04_etc/sample.md#필드>) · [함수](<../03_External_Packages/04_etc/sample.md#함수>) · [Branch 또는 분기](<../03_External_Packages/04_etc/sample.md#Branch 또는 분기>) · [types.Options](<../03_External_Packages/04_etc/sample.md#types.Options>) · [Runner](<../03_External_Packages/04_etc/sample.md#Runner>) · [Catalog](<../03_External_Packages/04_etc/sample.md#Catalog>)
> [Loader](<../03_External_Packages/04_etc/sample.md#Loader>) · [Parser](<../03_External_Packages/04_etc/sample.md#Parser>) · [Engine](<../03_External_Packages/04_etc/sample.md#Engine>) · [Output Writer](<../03_External_Packages/04_etc/sample.md#Output Writer>) · [Matcher](<../03_External_Packages/04_etc/sample.md#Matcher>) · [Extractor](<../03_External_Packages/04_etc/sample.md#Extractor>) · [DSL](<../03_External_Packages/04_etc/sample.md#DSL>) · [-list-dsl-function](<../03_External_Packages/04_etc/sample.md#-list-dsl-function>)
<!-- glossary-links:end -->










## 설명서를 읽을 때 지키는 약속

이 설명서는 독자가 앞 문서를 읽었더라도 새 파일에서 중요한 이름이 다시 등장하면 짧게 뜻을 반복한다. 처음 등장하는 이름은 다음 다섯 가지를 가능한 한 함께 적는다.

```text
이름이 무엇인가?
→ 어느 파일에 선언되어 있는가?
→ 누가 값을 만들거나 채우는가?
→ 어느 함수가 사용하는가?
→ 다음 단계로 무엇을 전달하는가?
```

예를 들어 `Runner`라는 이름만 쓰지 않고 다음처럼 연결해서 설명한다.

```text
Runner
├─ 선언: internal/runner/runner.go의 Runner 구조체
├─ 생성: 같은 파일의 runner.New(options)
├─ 생성 요청자: cmd/nuclei/main.go의 main()
├─ 보관 값: Options, Catalog, Parser, Writer 등
└─ 다음 단계: RunEnumeration()에서 Engine 실행 환경 구성
```

### 문서에서 사용하는 표기

| 표기 | 뜻 |
|---|---|
| `cmd/nuclei/main.go` | 원본 소스의 파일 경로 |
| `main()` | 함수 이름 |
| `Runner` | 구조체 또는 타입 이름 |
| `runner.New(options)` | `runner` 패키지의 `New` 함수 호출 |
| `r.options` | Runner 객체 `r` 안의 `options` 필드 |
| `*types.Options` | `types.Options` 객체를 가리키는 참조 타입 |
| `A → B` | A의 결과 또는 참조가 B로 전달되거나 B가 다음에 호출됨 |

`→`는 항상 “새 복사본 생성”을 뜻하지 않는다. 같은 객체의 참조가 전달되는 경우와, 한 형식이 다른 형식으로 변환되는 경우를 본문에서 구분한다.

이 폴더는 `nuclei-analysis`에 흩어진 분석 문서를 **처음 보는 사람도 순서대로 이해할 수 있게 연결한 안내서**다. 기존 분석 문서는 실제 코드와 함수 단위의 세부 기록이고, 이 설명서는 그 기록을 읽기 위한 배경지식과 전체 관계를 설명한다.

Go 문법 자체보다는 다음 질문에 답하는 데 초점을 둔다.

- 이 폴더는 무엇을 분석한 곳인가?
- 이 파일은 전체 실행 과정에서 언제 사용되는가?
- 옵션, Runner, Catalog, Loader, Engine 같은 구성요소가 왜 필요한가?
- 템플릿과 DSL은 무엇이며 실제 스캔과 어떻게 연결되는가?
- 명령을 입력하면 어느 파일을 거쳐 어떤 결과가 나오는가?

## 권장 학습 순서

1. [[01_프로젝트와 폴더 지도]]
2. [[02_Main과 프로그램 시작 흐름]]
3. [[03_CLI와 readConfig]]
4. [[04_내부 패키지]]
5. [[05_외부 패키지]]
6. [[06_템플릿과 DSL]]
7. [[07_전체 스캔 실행 흐름]]
8. [[08_명령어별 동작]]
9. [[09_용어 사전]]
10. [[10_현재 문서 상태와 주의점]]
11. [[11_코드 구성요소를 읽는 방법]]
12. [[12_구성요소 연결 상세]]
13. [[13_main.go 분기별 상세 설명]]
14. [[14_-list-dsl-function 상세]]

명령을 실행했을 때 나타난 긴 원문은 [출력결과 폴더](<../출력결과/README.md>)에서 확인한다. 현재 `-list-dsl-function` 설명은 [-list-dsl-function 실행 결과](<../출력결과/-list-dsl-function.md>)로 연결되어 있다.

각 문서의 시작 부분에는 앞 문서에서 무엇을 이어받는지, 그 문서에서 새로 등장하는 이름은 무엇인지 적는다. 용어를 잊었을 때는 [[09_용어 사전]]을 함께 본다.

## main.go 줄 번호 태그

`main.go`의 특정 줄을 설명하는 항목에는 단순 줄 태그 `#L번호`와 세부 분류 태그 `#main/L번호/기능`을 함께 사용한다.

```text
## L80 — -sign 템플릿 서명
#전지성 #L80 #main/L80/template-sign
```

- `#L80`: 줄 번호만으로 빠르게 검색할 때 사용한다.
- `#main/L80/template-sign`: main의 기능 분류까지 좁혀서 검색할 때 사용한다.
- `상세 참조` 링크: 태그 검색 결과가 아니라 설명서의 L80 제목 위치로 직접 이동한다.

| main.go 위치 | 기능 | 태그 |
|---|---|---|
| L74 | DSL 함수 목록 | `#L74`, `#main/L74/dsl` |
| L80 | 템플릿 서명 | `#L80`, `#main/L80/template-sign` |
| L118 | 성능 Profiling | `#L118`, `#main/L118/profiling` |
| L166 | Execution ID | `#L166`, `#main/L166/execution-id` |
| L168 | ParseOptions | `#L168`, `#main/L168/parse-options` |
| L170 | Cloud 업로드 | `#L170`, `#main/L170/cloud-upload` |
| L177 | Runner 생성 | `#L177`, `#main/L177/runner-new` |
| L185 | Hang Monitor | `#L185`, `#main/L185/hang-monitor` |
| L204 | Graceful Shutdown | `#L204`, `#main/L204/graceful-shutdown` |
| L237 | RunEnumeration | `#L237`, `#main/L237/run-enumeration` |
| L245 | Resume 정리 | `#L245`, `#main/L245/resume-cleanup` |

공통 명칭의 기준 문서는 [Nuclei 분석 명칭 사전](../03_External_Packages/04_etc/sample.md)이다. 새 명칭은 이 파일에 먼저 정의하고, 설명서에서는 현재 코드에서 그 명칭이 어떻게 연결되는지를 설명한다.

`01_Main_Flow/main.md`는 줄 번호별 실행 순서를 빠르게 확인하는 요약 문서다. 요약만으로 이해되지 않는 분기는 [[13_main.go 분기별 상세 설명]]으로 연결하고, DSL을 처음 접하는 독자는 [[14_-list-dsl-function 상세]]부터 읽는다.

## 가장 먼저 기억할 핵심

```text
사용자 명령
  ↓
main.go가 옵션을 등록하고 입력값을 읽음
  ↓
Options에 실행 설정이 저장됨
  ↓
Runner가 필요한 구성요소를 준비함
  ↓
Catalog와 Loader가 템플릿을 찾고 불러옴
  ↓
Engine이 대상과 템플릿을 조합해 실행함
  ↓
Matcher가 탐지 여부를 판단하고 Extractor가 값을 추출함
  ↓
Output Writer가 화면 또는 파일로 결과를 보냄
```

Nuclei 실행 파일은 **공통 실행 엔진**이고, 템플릿은 **무엇을 요청하고 어떤 조건을 탐지할지 적은 검사 설명서**다. 같은 실행 파일을 사용해도 선택한 템플릿과 옵션에 따라 검사 내용이 달라진다.

## 원본 코드 기준

이 설명서는 다음 원본을 기준으로 작성했다.

- Nuclei: v3 계열 분석 소스
- DSL 모듈: `github.com/projectdiscovery/dsl v0.8.20`
- 주요 시작 파일: `cmd/nuclei/main.go`
- 주요 실행 파일: `internal/runner/runner.go`, `pkg/core/engine.go`

버전이 바뀌면 옵션 이름, 필드, 줄 번호, 함수 목록이 달라질 수 있다. 따라서 줄 번호보다 **함수 이름과 데이터 흐름**을 우선해서 보는 것이 좋다.

## 이 설명서의 분석 형식

구성요소를 설명할 때 다음 순서를 사용한다.

```text
파일 위치
→ 선언된 구조체·변수·함수
→ 값을 실제로 만드는 위치
→ 호출하는 코드
→ 전달되는 데이터
→ 다음 구성요소
→ 이렇게 분리한 이유
→ 코드에서 확인되는 사실과 설계에 대한 해석 구분
```

단순히 “Runner는 지휘자다”라고 외우는 것이 아니라, Runner의 어떤 필드가 어느 객체를 보관하고 어떤 함수가 그 객체를 생성하는지까지 추적하는 것이 목표다.


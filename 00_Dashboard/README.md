Nuclei 흐름 요약

1. CLI 입력 파싱 (cmd/nuclei/)
2. 옵션 및 환경 초기화 (pkg/types/options.go)
3. 템플릿 로드, 태그 필터링 (pkg/catalog/loader/loader.go)
4. 입력 타겟 파싱 (pkg/input -> pkg/protocols/common/contextargs/)
5. 프로토콜 검사 수행 (pkg/protocols/http/request.go)
6. 결과 출력 및 파일 저장 (pkg/output/output.go)



**명령어**

![](../05_attachments/Pasted%20image%2020260715184421.png)
- `nuclei -version` : `-version` 플래그를 감지하면(`goflags` 외부 라이브러리 사용) `config.Version` 출력 후 `os.Exit(0)` 호출하여 종료


![](../05_attachments/help.txt)
- `nuclei -h` : `-h` 플래그를 감지하면 `goflags` 라이브러리의 `Usage()`와 `CommandLineHelp` 메서드를 호출하여 도움말 출력


- `nuclei -u https://example.com` : 
	- `goflags`의 파싱 과정에서 `-u`를 인식하고 `&options.Targets` 주소에 `[]string{"https://example.com"}` 형태로 저장
	- 템플릿 로드 및 필터링 이후 `pkg/input`, `pkg/protocols` 에서 타겟 URL 을 파싱하여 스키마(`https`), 호스트(`example.com`), 포트(`443`)을 분리하여 저장. 
	- `https`가 식별되었으므로 내부 HTTP 클라이언트에 TLS 스키마 플래그를 활성화. 이후 입력 프로바이더(`InputProvider`)를 통해서 중복 제거 및 정규화를 거쳐 실제 실행 큐로 넘김
	- 대기 큐에서 타겟을 꺼내 프로토콜 검사를 위해 비동기 스캔 프로세스 실행.
	- 검사 결과(`ResultEvent`)를 채널에 넣고 `pkg/runner`의 채널 수신 루프에서 이벤트를 꺼내어 `pkg/output/standard_writer.go` 파일의 `Wirte()`함수로 출력

- `nuclei -u https://example.com -v` : 
	- `nuclei -u https://example.com`의 명령어와 동일 과정을 거쳐 `-v` 를 파싱하고 `options.Verbose` 변수에 `true` 저장
	- `pkg/protocols`내부에서 YAML 템플릿을 하나 실행하며 엔진 내부 코드에서 `options.Verbose` 값을 확인하고 `true`인 경우 `gologger` 외부 로깅 라이브러리를 사용하여 (`LevelVerbose`)즉시 출력

- `nuclei -u https://example.com -silent` :
	- `-silent` 플래그가 파싱되어 `options.Silent` 변수에 `true` 저장, `options` 객체 `runner`에 전달
	- `gologger` 외부 라이브러리에서 모든 출력 차단 (`LevelSilent`)
	- `ResultEvent`를 `StandardWriter` 가 출력할 때 `options.Silent`를 확인하여 `true`인 경우 핵심 결과만 출력

- `nuclei -u https://example.com -s low,medium,high,critical` :
	- `goflags`에서 `-s` 옵션을 읽고 `low,medium,high,critical` 문자열을 파싱하여 `options.Severity` 슬라이스에 저장
	- 템플릿 로드 과정에서 Severity 필터를 적용하여 각 템플릿의 YAML 헤더의 `severity` 값을 확인하고 슬라이스에 저장된 심각도와 비교하여 일치하는 경우만 스캔 큐에 대기
	- 위 `nuclei -u https://example.com` 명령어와 동일하게 타겟을 지정하고 필터링한 YAML 템플릿으로 `Engine`에서 비동기 스캔 실행
	- `ResultEvent` 출력

- `nuclei -u https://example.com -tags tech` :
	- `goflags`가 `-tags tech` 파싱 후 `tech` 단어를 `optinos.Tags` 슬라이스에 저장
	- `Runner`는 `pkg/catalog/loader`를 통해 템플릿을 불러올 때, 각 템플릿(YAML)의 `info` 섹션에 `tags` 필드 검사
	- 각 YAML 파일 태그에 `tech`가 포함되어 있으면 스캔 큐에 저장
	- `Engine`에서 비동기 스캔을 실행하고 `ResultEvent`를 채널로 전달하고 `StandardWriter`에 의해 출력



# CLI 입력 파싱, 옵션 및 환경 초기화

`cmd/nuclei/main.go`의 `main()`이 프로그램이 시작되면 가장 먼저 실행되며, `pkg/types/options.go`를 호출

`pkg/types/options.go`의 `ParseOptions()`함수에서 `goflags` 라이브러리를 이용하여 입력된 플래그들을 Go 객체인 `Options` 구조체 필드에 매핑


# 템플릿 로딩, 카탈로그 필터링

`pkg/catalog/loader/loader.go` 의 `New()`, `Store.Load()` 함수
- 사용자의 로컬 환경(`~/.nuclei-templates/`)에서 **모든 YAML 파일을 스캔하고 검증**

`pkg/catalog/loader/filter/` 디렉터리 내의 **필터 로직**
- `matchTags()` 등의 함수에서 각 YAML 파일의 `info.tags` 필드를 읽어 사용자가 입력한 값(`tech`, `exposure`, `cve` 등)과 문자열 교집합 비교 연산을 수행합니다.

`pkg/catalog/loader/filter/` 디렉터리 내의 **심각도 매칭 로직**
- `matchSeverity()` 함수에서 YAML에 표기된 `info.severity` 값과 사용자가 입력한 `-s low,medium...` 배열을 비교하여 유효한 템플릿만 남깁니다.

`pkg/catalog/loader/loader.go` 내부 분기 처리
- `TemplateList` 플래그(`-tl`)나 `TemplateTagList` 플래그(`-tgl`)가 `true`이면 스캔 엔진을 가동하지 않고, 수집된 템플릿 파일들의 ID와 경로 또는 태그 빈도 카운팅 맵(`map[string]int`)을 `os.Stdout`으로 덤프하고 프로그램을 종료시킵니다.

# 입력 타겟 파싱

`pkg/protocols/common/contextargs/contextargs.go` 의 `NewWithInput()` 함수
- 사용자가 입력한 `-u https://example.com` 문자열을 넘겨받아 URL을 구문분석(Parsing)합니다.
- URL 객체의 `Scheme` 필드를 검사합니다. `https`가 식별되면 TLS 연결용 설정 플래그를 켜고 기본 목적지 포트를 `443`으로 세팅합니다. `http`라면 기본 포트를 `80`으로 세팅합니다. 만약 프로토콜 스키마가 통째로 빠져있다면 기본 스키마를 보정하여 할당합니다.


`pkg/input/provider/` 내 파일 기반 프로바이더 구현체 (옵션으로 여러 값을 받는 경우)
- `-l targets.txt` 파일 핸들을 열고 `bufio.NewScanner`를 사용해 라인 단위로 타겟 목록을 읽어옵니다. 데이터 유효성 검사 및 정규화(Trim, 스키마 유무 확인)를 마친 후 가상 대기 큐(Target Queue)에 삽입합니다.


# 프로토콜 검사 수행

`pkg/core/engine.go` 의 `ExecuteTemplates()` 및 `pkg/core/executors.go`
- 로드된 템플릿 개수와 타겟 대기 큐를 바탕으로 Go 루틴 워커 풀(Worker Pool)을 가동하여 비동기 스캔 프로세스를 진행합니다.

`pkg/protocols/http/request.go` 의 `executeRequest()` 함수
- Nuclei 커스텀 HTTP 클라이언트 세션을 열어 요청을 생성합니다.
- 만약 전역 옵션의 `Debug`가 `true`이면, 실제 소켓 전송 바로 직전 단계에서 `httputil.DumpRequestOut()`과 수신 직후의 `httputil.DumpResponse()`를 호출하여 터미널 콘솔에 원본 그대로의 네트워크 패킷을 디버그 출력합니다.

# 결과 출력 및 파일 저장

`pkg/output/standard_writer.go` 의 `Write()` 함수
- 스캔 코어 엔진으로부터 취약점이 최종 매칭되었다는 이벤트(`ResultEvent`)가 발행되면 이를 가로채 전역 옵션(`-silent`, `-v` 등)에 맞게 화면 출력을 조절합니다.

`pkg/output/file_writer.go` (텍스트 파일 저장)
- `-o result.txt` 가 설정되어 있다면, 파일 스트림 버퍼에 취약점 탐지 텍스트 줄(ASCII 포맷)을 실시간으로 추가(`Write`) 및 동기화(`Sync`)합니다.

`pkg/output/json_writer.go`
- `-json-export result.json`이 활성화되어 있다면, 매칭된 `ResultEvent` 구조체를 `json.Marshal()` 함수를 통해 한 줄 단위의 JSON 데이터 형식으로 인코딩한 뒤 파일에 영구 기록합니다.





Compile : YAML 데이터를 구조체로 변환하여 메모리에 올림
Excute : 엔진이 템플릿을 실행할 때 호출하는 가장 중요한 함수. 여기서 실제 네트워크 통신이 시작
Match : 서버로부터 받은 응답 데이터를 우리가 정의한 템플릿(Matcher)와 비교하여 취약점 여부를 판별하는 로직
Extract : 필요시 응답에서 원하는 값을 정규식 등으로 뽑아내는 로직

# 모듈 공통 파이프라인 5단계

| **단계**  | **파이프라인 명칭**                                                                     | **주요 역할 및 동작 내용**                                                                                                |
| ------- | -------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **1단계** | **Input & Variable Substitution**<br><br>  <br><br>(입력 및 변수 치환)                  | 템플릿 내의 변수(`{{BaseURL}}`, `{{Hostname}}`, `{{IP}}` 등)를 스캔 대상의 정보로 치환하고 전송할 동적 페이로드를 준비합니다.                        |
| **2단계** | **Connection & Engine Setup**<br><br>  <br><br>(연결 및 엔진 가동)                      | 모듈 성격에 맞는 클라이언트/엔진을 준비합니다.<br><br>  <br><br>_(예: HTTP 소켓, Headless Chrome, Goja JS 엔진, OS Shell 등)_              |
| **3단계** | **Execution & Data Transmission**<br><br>  <br><br>(명령/패킷 전송 및 실행)               | 실제 데이터 패킷을 타깃으로 송신하거나, 스크립트/명령어를 직접 실행합니다.<br><br>  <br><br>_(예: HTTP Request, TCP Packet, JS Code, OS Command)_ |
| **4단계** | **Response Capture & Structuring**<br><br>  <br><br>(응답 수신 및 `internalEvent` 생성) | 타깃이나 실행 결과로부터 돌아온 데이터(Status, Header, Body, Log 등)를 `internalEvent` 맵 데이터 구조체로 파싱합니다.                            |
| **5단계** | **Operators Evaluation & Output**<br><br>  <br><br>(매칭/추출 평가 및 결과 출력)            | `internalEvent` 데이터를 바탕으로 **Matcher**(취약점 조건 판정)와 **Extractor**(민감 정보 추출)를 실행하여 터미널에 결과를 출력합니다.                  |

# HTTP/HTTPS
### 1단계: 템플릿 실행기 진입 (`pkg/tmplexec/generic/exec.go`)

`runner`에 의해 생성된 `Executer` 스레드가 실행되면, 이 파일에서 실제 스캔 루프가 돌아갑니다.

- **주요 구조체**:
    
    Go
    
    ```
    type Executer struct {
        requests []protocols.Request // 실행해야 할 프로토콜 Request 목록 (HTTP, DNS 등)
        operators *operators.Operators // 결과를 판별할 매처/익스트랙터
    }
    ```
    
- **핵심 함수**: `func (e *Executer) Execute(ctx *scan.ScanContext) (bool, error)`
    
    1. `ctx` (스캔 환경 객체)에서 타겟 정보(`ctx.Input.MetaInput.Input`)를 추출합니다.
        
    2. `e.requests` 배열을 순회하며 각 프로토콜의 `ExecuteWithResults`를 호출합니다.
        
    3. **실시간 콜백 실행**: `ExecuteWithResults`에 결과 이벤트 처리 콜백 함수(`func(event *output.InternalWrappedEvent)`)를 인자로 넘겨줍니다.
        

### 2단계: 프로토콜 패킷 빌드 및 전송 (`pkg/protocols/http/request.go` 기준)

HTTP 프로토콜을 예로 들면, 템플릿의 문자열을 진짜 네트워크 패킷으로 치환하고 전송합니다.

- **주요 구조체**: `type Request struct`
    
    - `Path []string`: 템플릿에 정의된 요청 경로 (예: `{{BaseURL}}/admin/login.php`)
        
    - `Raw []string`: Raw HTTP 패킷 템플릿
        
    - `Operators *operators.Operators`: 프로토콜 레벨 매처
        
- **핵심 함수**: `func (r *Request) ExecuteWithResults(ctx *scan.ScanContext, dynamicValues map[string]interface{}, previous output.InternalEvent, callback protocols.OutputEventCallback) error`
    

#### 내부 세부 동작

1. **변수 치환 (DSL Engine)**:
    
    - `protocols.MakePaths()` 및 `r.options.Values.Extract()` 함수가 호출됩니다.
        
    - `dynamicValues` 맵을 활용하여 `{{BaseURL}}`을 `[https://example.com](https://example.com)`으로, `{{randstr}}`을 랜덤 문자열로 동적 변환합니다.
        
2. **네트워크 요청 생성**:
    
    - Raw 요청인 경우: `pkg/protocols/http/raw/raw.go`의 `ParseRequest()`가 호출되어 raw bytes를 HTTP Request 구조체로 복원합니다. (Go 내부 통신 라이브러리 http.Request 형태로 변환)
        
    - 일반 요청인 경우: `retryablehttp.Client` (Nuclei 자체 커스텀 HTTP 클라이언트)를 이용해 HTTP Request 객체를 만듭니다.
        
3. **패킷 전송 및 응답 수신**:
    
    - `r.httpClient.Do(req)`를 호출하여 타겟에 패킷을 날리고 `http.Response`를 받아옵니다.
        
4. **InternalEvent 데이터 맵 작성**:
    
    - 응답이 오면 `r.makeResultEvent()`를 통해 서버의 응답 데이터를 아래와 같은 Go `map[string]interface{}` 형태(`InternalEvent`)로 가공합니다. (나중에 Matcher가 취약점인지 판별하기 위해 아래의 형태로 저장함)
        
        - `"status_code"`: 200
            
        - `"header"`: "HTTP/1.1 200 OK\r\nContent-Type: text/html..."
            
        - `"body"`: "..."
            
        - `"response"`: 전체 원문 패킷
            

### 3단계: 매칭 및 취약점 검증 (`pkg/operators/operators.go`)

수신한 응답 데이터(`InternalEvent`)가 템플릿이 요구하는 조건(취약점 조건)에 맞는지 판별합니다.

- **주요 구조체**: `type Operators struct`
    
    - `Matchers []*matchers.Matcher`: 조건 비교 목록 (Status, DSL, Regex, Words 등)
        
    - `Extractors []*extractors.Extractor`: 데이터 추출기
        
- **핵심 함수**: `func (o *Operators) Evaluate(data map[string]interface{}) (bool, bool)`
    
    1. `pkg/operators/matchers/match.go` 내부의 `Match()` 함수들이 호출됩니다.
        
    2. 예를 들어 Matcher 타입이 `status`이고 조건이 `200`이라면, `data["status_code"] == 200`을 검증합니다.
        
    3. Matcher 타입이 `regex`라면 `data["body"]`에 대해 정규식 컴파일된 연산을 수행합니다.
        
    4. 모든 조건(AND/OR 로직)이 참(`true`)으로 평가되면 매칭 성공 상태로 결정됩니다.
        

Nuclei 템플릿은 단순 단어 검색부터 복잡한 논리식까지 다양한 조건 유형(**Matcher Type**)을 제공합니다. `pkg/operators/matchers/` 내의 검증 로직은 `InternalEvent` 맵의 항목들을 대상으로 아래 기준들을 검사합니다.

|**검증 유형 (type)**|**실제 검증 대상**|**검증 동작 메커니즘**|
|---|---|---|
|**`status`**|`data["status_code"]`|서버의 응답 코드(예: `200`, `302`, `403`)가 템플릿에 명시된 숫자와 일치하는가?|
|**`word`**|`data["body"]`, `data["header"]` 등|지정한 영역에 특정 문자열(예: `root:x:0:0`, `Admin Dashboard`)이 포함되어 있는가?|
|**`regex`**|`data["body"]`, `data["response"]`|정규표현식 패턴(예: `vulnerable-v[0-9]\.[0-9]`)에 매칭되는 문자열이 있는가?|
|**`dsl`**|`InternalEvent` 맵 전체|Go 기반 산술/논리 표현식(예: `status_code == 200 && contains(body, 'admin')`)의 결과가 `true`인가?|
|**`binary`**|Hex / Raw Byte 데이터|헥사코드 형태의 패킷 바이너리 데이터가 응답 패킷과 일치하는가?|

### 4단계: 실시간 결과 반환 및 출력 (`pkg/scan/context.go` & `pkg/output/`)

매칭이 성공하면 대기하고 있던 콜백 함수가 즉시 발동합니다.

- **호출 매커니즘**:
    
    Go
    
    ```
    // pkg/protocols/http/request.go 내부
    if operators.Evaluate(data) {
        resultEvent := r.makeResultEvent(data) // 최종 ResultEvent 객체 생성
    
        // 2단계에서 Executer가 전달해 준 콜백 함수를 즉시 실행!
        callback(&output.InternalWrappedEvent{
            Results: []*output.ResultEvent{resultEvent},
        })
    }
    ```
    
- **출력 처리**:
    
    - 전달받은 콜백 함수는 `ctx.Output.ResultHook` 또는 `runner`에 등록된 `outputWriter.Write(resultEvent)`를 호출합니다.
        
    - `pkg/output/output.go`의 `Write()` 함수가 터미널에 색상이 들어간 알파벳 라인(`[cve-2023-xxx] [http] [critical] [https://example.com](https://example.com)`)을 실시간으로 보내고, `-o` 옵션이 설정되어 있다면 파일에 로그를 기록합니다.
        

## 4. 요약

### 1단계: 템플릿 변수 치환 (`dynamicValues`)

- **변수 매핑**: YAML 템플릿에 정의된 `{{BaseURL}}`, `{{Host}}`, `{{randstr}}` 등 가변 파라미터들을 `dynamicValues` 맵에 동적으로 등록합니다.
    
- **사용자 입력 적용**: 스캔 명령 시 사용자가 전달한 대상 도메인/IP 주소와 무작위 공격 문자열(Payload)을 템플릿의 변수 위치에 실제 값으로 치환합니다.
    

### 2단계: 요청 성능 최적화 & 세션 관리 (성능/상태 최적화)

- **템플릿 클러스터링 (Clustering)**: 동일한 주소/경로로 가는 여러 템플릿의 요청을 하나로 그룹화합니다. **단 1번만 네트워크 패킷을 보내고, 돌아온 응답을 여러 템플릿이 공유**하여 대상 서버 과부하를 막고 스캔 속도를 높입니다.
    
- **상태/세션 쿠키 추적 (Stateful Flow)**: 다단계 공격 시나리오 시, 앞선 요청의 응답에서 받은 **쿠키(`Set-Cookie`)나 CSRF 토큰**을 메모리에 보관했다가 다음 요청 패킷의 헤더에 자동으로 주입합니다.
    

### 3단계: HTTP 패킷 생성 (Raw vs Standard)

- **Standard (일반 요청)**: Go 언어의 `net/http` 표준 라이브러리 구조체 형태로 깔끔한 HTTP 패킷을 만듭니다.
    
- **Raw (원본 패킷 요청)**: 줄바꿈(`\r\n`), 헤더 순서, 비표준 불법 메서드 등 **패킷의 형태 자체를 1바이트 단위로 미세 조작**해야 하는 공격(HTTP Request Smuggling 등)을 위해 쌩 텍스트 형태의 패킷을 조립합니다.
    
- **OOB Payload 주입 (Interactsh)**: 응답 바디가 비어있는 취약점(Blind RCE/SSRF/Log4j)을 찾기 위해, 외부 감지 서버 주소인 `{{interactsh-url}}`을 패킷 내부에 심습니다.
    

### 4단계: Go 네트워크 암호화 통신 (`net` / `crypto/tls`)

Nuclei 내부의 Go 네트워크 통신 모듈이 OS 커널과 연동하여 보안 통로를 개설하고 데이터를 보냅니다.

1. **TCP Handshake (OS 소켓 연결)**:
    
    - OS 커널 레벨에서 대상의 IP:Port로 `SYN ➔ SYN-ACK ➔ ACK` (3-Way Handshake)를 수행하여 디지털 통신 파이프(소켓)를 개설합니다.
        
2. **TLS Handshake (HTTPS 암호화 통로)**:
    
    - 開設된 TCP 소켓 위에서 서버 인증서를 검증하고, **디피-헬만(DH) 수학 연산**을 통해 양쪽이 절대 도청당하지 않는 동일한 대칭키(Master Secret)를 각자 만듭니다.
        
3. **암호화 패킷 전송 및 복호화 수신**:
    
    - 3단계에서 조립된 HTTP 패킷을 TLS 대칭키로 **AES 암호화**하여 소켓으로 밀어 넣습니다.
        
    - 서버가 돌려준 암호화된 응답 패킷을 받아서 **TLS 대칭키로 다시 복호화**하여 쌩(Raw) 텍스트 데이터로 되돌립니다.
        

### 5단계: 응답 데이터 구조화 (`internalEvent`)

- 서버로부터 복호화되어 돌아온 **응답 상태 코드(Status Code), 응답 헤더(Headers), 응답 바디(Body), 수신 시간, OOB 서버 호출 여부** 등의 모든 통신 결과 데이터를 `internalEvent`라는 키-값(Key-Value) 형태의 Go 맵 구조체로 저장합니다.
    

### 6단계: Operators 매칭 & Extractors 가공 (최종 평가 및 출력)

1. **Matchers (취약점 판정)**:
    
    - **상태 코드**: `internalEvent["status_code"] == 200` 인지 확인.
        
    - **정규식 (Regex)**: `internalEvent["body"]` 안에서 비밀번호 패턴(`root:.*:0:0`), 특정 에러 메시지가 찍혔는지 패턴 돋보기로 검색.
        
    - **DSL (종합 판단)**: `status_code == 200 && contains(body, 'admin')` 과 같이 여러 조건을 논리 연산(AND/OR)으로 평가하여 Final True(취약점 발생)를 결정.
        
    - **OOB Check**: 응답 데이터와 상관없이, 3단계에서 보낸 `interactsh` 외부 서버로 DNS/HTTP 신호가 들어왔는지 확인하여 취약점 판정.
        
2. **Extractors (민감 정보 추출)**:
    
    - 취약함이 확인되면, 응답 바디나 헤더에서 **실제 유출된 API Key, 토큰, 비밀번호 문자열만 정규식으로 뽑아냅니다.**
        
3. **최종 출력**:
    
    - 판정 결과가 True이면 **(VULNERABILITY_NAME) (SEVERITY) (URL) (추출된 데이터)** 형태로 터미널 화면이나 로그 파일에 결과 텍스트를 출력합니다.

# Network

Network 모듈은 HTTP 헤더나 웹 브라우저 개념 없이, **소켓을 열고 원하는 바이너리/텍스트 페이로드를 직접 넣은 뒤 응답 바이트를 분석하는 5단계 파이프라인**으로 작동

### 1단계: 템플릿 변수 치환 & 타겟 포트 결정

- **주소/포트 매핑**: `{{Hostname}}`, `{{IP}}`, `{{Port}}` 변수를 사용자가 지정한 타겟 IP와 검사할 포트 번호(예: MySQL `3306`, Redis `6379`, SSH `22`)로 치환합니다.
    
- **프로토콜 지정**: YAML 템플릿의 `type` 정의에 따라 전송방식을 **TCP**로 할지, **UDP**로 할지 결정합니다.
    

### 2단계: L4 네트워크 소켓 직접 연결 (Go `net` 패키지)

HTTP 클라이언트를 거치지 않고, Go 언어의 `net.DialTimeout()` 함수를 통해 **OS 커널 소켓을 직접 엽니다.**

1. **TCP 모듈인 경우**:
    
    - 대상 서버의 `IP:Port`로 TCP 3-Way Handshake (`SYN ➔ SYN-ACK ➔ ACK`)를 수행하여 순수 소켓 파이프라인을 연결합니다.
        
2. **UDP 모듈인 경우**:
    
    - 연결 확인 절차 없이 즉시 패킷을 보낼 수 있는 비연결형 UDP 소켓을 준비합니다.
        
3. **TLS/SSL 옵션 (선택적)**:
    
    - DB나 특수 포트가 SSL/TLS로 암호화되어 있다면(`tls: true`), TCP 연결 직후 **TLS Handshake**를 수행하여 소켓을 암호화 파이프(`tls.Conn`)로 승격시킵니다.
        

### 3단계: Raw Payload 생성 & 파이프 직송 (Read/Write)

웹 패킷처럼 `GET / HTTP/1.1` 같은 구조를 만들지 않고, **서버가 알아듣는 고유한 프로토콜 명령어를 텍스트나 16진수(Hex) 바이트로 직접 전송**합니다.

- **Text/String 전송**:
    
    - Redis 미인증 점검예시: 소켓에 `INFO\r\n` 텍스트 문자열을 그대로 날림.
        
- **Hex (16진수 바이너리) 전송**:
    
    - 특수 바이너리 프로토콜 패킷을 조작할 때 사용. (예: `hex: "50494e47"` ➔ 메모리상에서 `PING` 바이너리 바이트로 변환 후 소켓에 `Write()`)
        
- **Banner Grabbing (대기)**:
    
    - 먼저 데이터를 보내지 않고, 소켓이 열리자마자 서버가 먼저 뱉어내는 인사말 문자열(예: SSH 서버의 `SSH-2.0-OpenSSH_8.9p1`)을 읽기만(`Read()`) 하는 방식도 수행합니다.
        

### 4단계: 소켓 데이터 읽기 & `internalEvent` 가공

- **응답 바이너리/텍스트 수집**: 소켓 통로에서 서버가 돌려준 Raw 바이트 배열(Byte Array)을 읽어옵니다.
    
- **데이터 구조화 (`internalEvent`)**:
    
    - `internalEvent["buffer"]`: 서버가 보내온 전체 쌩(Raw) 바이트 데이터/문자열.
        
    - `internalEvent["type"]`: `tcp` 또는 `udp`.
        
    - `internalEvent["ip"]`, `internalEvent["port"]`: 접속한 대상 정보.
        

### 5단계: Operators 매칭 & 판정 (Matchers)

HTTP 상태 코드(`200 OK` 등) 개념이 없기 때문에, 오직 **서버가 돌려준 쌩 바이트/텍스트 패턴**만 가지고 매칭을 진행합니다.

1. **Matchers (조건 검증)**:
    
    - **Text Matcher**: `contains(buffer, 'redis_version')` ➔ Redis 서버에 인증 없이 접근되어 `INFO` 결과 응답 데이터가 온 경우 **True (취약)**.
        
    - **Hex/Binary Matcher**: `hex: "0500"` ➔ SOCKS5 프록시 서버의 인증 성공 응답 바이너리가 돌아왔는지 매칭.
        
    - **Regex Matcher**: `regex: "OpenSSH_[0-7]"` ➔ 서버가 응답한 SSH 버전 문자열을 정규식으로 감지.
        
2. **OOB (Interactsh) 체크**:
    
    - Raw TCP/UDP 패킷 내부에도 `{{interactsh-url}}`을 실어 보내어, 취약한 서버가 외부로 역접속(Reverse Shell / Outbound Connection)을 시도하는지 감지.
        
3. **결과 출력**:
    
    - 조건이 충족되면 **[REDIS-UNAUTH] [CRITICAL] 192.168.1.100:6379** 형태로 취약점 탐지 결과를 출력합니다.

## Network 모듈이 탐지하는 핵심 취약점

|**취약점 종류**|**상세 설명**|
|---|---|
|**1. 미인증 접근 (Unauthenticated Access)**|**비밀번호 없이 열려있는 DB/캐시 서버 탐지**<br><br>  <br><br>• Redis, MongoDB, Memcached, Elasticsearch 등의 포트에 접속했을 때, 비밀번호 입력(인증) 요구 없이 즉시 내부 데이터나 제어권을 내주는지 검사합니다.|
|**2. 서비스 배너 노출 (Banner Grabbing)**|**서버의 이름과 구버전 정보 노출 탐지**<br><br>  <br><br>• SSH, FTP, Telnet 등의 포트에 연결되자마자 서버가 스스로 뱉어내는 "인사말 문자열(Banner)"을 읽어와, 보안 패치가 안 된 구버전 소프트웨어인지 확인합니다.|
|**3. 디폴트 / 약한 계정 (Default Credentials)**|**초기 설정 관리자 계정 방치 탐지**<br><br>  <br><br>• SSH, FTP, MySQL 등에 `root/root`, `admin/admin` 같은 제조사 초기 설정 계정으로 핸드셰이크 패킷을 보내 로그인 성공 여부를 탐지합니다.|
|**4. 프로토콜 파싱 및 메모리 오류 (RCE / DoS)**|**특수 바이너리로 인한 서버 다운 및 원격 코드 실행 탐지**<br><br>  <br><br>• 조작된 16진수(Hex) 바이너리 패킷을 소켓에 직접 밀어 넣었을 때, 서버가 메모리 에러를 내거나 뻗어버리는 취약점(예: EternalBlue, SMBGhost 등)을 검사합니다.|
|**5. Out-of-Band (OOB) 역접속 탐지**|**외부로 통신을 시도하는 눈먼(Blind) 취약점 탐지**<br><br>  <br><br>• Raw 패킷 속에 `{{interactsh-url}}` 미끼 주소를 심어 보내어, 대상 서버가 내부적으로 외부(Interactsh)로 DNS/TCP 역접속 신호를 날리는지 감지합니다.|

# DNS

DNS 모듈은 웹 서버나 DB 서버에 직접 접속하는 게 아니라, 도메인 이름(예: `example.com`)을 IP 주소로 바꿔주는 **네임서버(DNS Server)에 질의(Query) 패킷을 던져서 도메인 설정상의 취약점을 찾는 모듈**입니다.
### 1단계: 타겟 도메인 변수 치환 & 질의 타입(Type) 결정

- **변수 치환**: `{{FQDN}}`, `{{Hostname}}` 등의 변수를 스캔할 도메인 주소(예: `sub.example.com`)로 치환합니다.
    
- **DNS 레코드 타입 결정**: YAML 템플릿의 `dns:` 설정에 따라 네임서버에 조회할 **레코드 종류**를 정합니다.
    
    - `A` (IPv4 주소), `AAAA` (IPv6 주소)
        
    - `CNAME` (별칭 도메인)
        
    - `TXT` (텍스트 정보/인증 레코드)
        
    - `MX` (메일 서버 주소), `NS` (네임서버 주소), `PTR` (역방향 IP 조회)
        

### 2단계: DNS 전용 라이브러리로 레졸루션(Resolution) 패킷 생성

- HTTP나 일반 소켓을 쓰지 않고, Go 언어의 DNS 전용 라이브러리(`miekg/dns`)를 사용해 53번 포트(UDP/TCP)로 보낼 **DNS Query 패킷**을 바이너리 형태로 만듭니다.
    

### 3단계: 네임서버로 DNS Query 패킷 전송

- 지정된 **DNS 네임서버(예: `8.8.8.8` 또는 타겟의 전용 Authoritative DNS Server)의 53번 포트**로 UDP(기본) 또는 TCP 패킷을 쏩니다.
    

### 4단계: DNS 응답(Response) 수집 & `internalEvent` 구조화

- 네임서버가 돌려준 **DNS Response 패킷**을 수신합니다.
    
- 응답 코드와 레코드 값들을 `internalEvent` 맵에 구조화하여 저장합니다:
    
    - `internalEvent["rcode"]`: DNS 응답 상태 코드 (`NOERROR`, `NXDOMAIN`[존재하지 않는 도메인], `REFUSED` 등)
        
    - `internalEvent["a"]`, `internalEvent["cname"]`, `internalEvent["txt"]`: 네임서버가 돌려준 실제 레코드 값 리스트
        

### 5단계: Operators 매칭, Extractor 데이터 추출 & 결과 출력

- `internalEvent` 데이터를 기반으로 설정 오류나 취약한 상태를 평가합니다.
    
    - **Matcher 예시**: `cname` 레코드 응답 값이 `s3.amazonaws.com`을 가리키고, 응답 코드가 `NXDOMAIN`인 경우 ➔ **Subdomain Takeover 취약점 발생(True)**
        
    - **Extractor 예시**: TXT 레코드에 노출된 내부 서버 주소나 API 인증 키 문자열 파싱
        
- 결과가 True이면 터미널에 결과를 출력합니다.

## DNS 모듈이 탐지하는 핵심 취약점

### 1. Subdomain Takeover (서브도메인 탈취)

- **개념**: `sub.example.com`이라는 서브도메인이 외부 서비스(GitHub Pages, AWS S3, Zendesk 등)를 가리키도록 `CNAME` 레코드가 설정되어 있는데, 정작 **그 외부 서비스 계정이나 버킷이 삭제되어 방치된 상태**를 탐지합니다.
    
- **위험성**: 해커가 해당 외부 서비스에 똑같은 이름으로 계정/버킷을 새로 등록해 버리면, `sub.example.com`으로 들어오는 모든 사용자를 해커의 악성 페이지로 연결시킬 수 있습니다.
    

### 2. DNS Zone Transfer (영역 전송 설정 오류)

- **개념**: DNS 서버끼리 도메인 목록을 복사할 때 쓰는 `AXFR` 요청을 보안 검증 없이 외부인에게 허용해 주는지 탐지합니다.
    
- **위험성**: 공격자가 단 한 번의 요청으로 회사 내부의 모든 서브도메인 리스트(`admin.dev.example.com`, `db-internal.example.com` 등)를 싹 노출당하게 됩니다.
    

### 3. DNS TXT 레코드 내 민감 정보 노출

- **개념**: 도메인 소유권 인증이나 메일 보안(SPF, DKIM 등)을 위해 설정해 둔 `TXT` 레코드에 불필요한 테스트용 API 키, 내부 IP 주소, 시스템 정보가 텍스트로 남아있는지 검사합니다.
    

### 4. 댕글링 레코드 (Dangling DNS Records)

- **개념**: 더 이상 사용하지 않는 옛날 서버의 IP 주소를 `A` 레코드가 그대로 가리키고 있는지(PTR 레코드 불일치) 점검하여 IP 재해당에 따른 보안 리스크를 탐지합니다.

# SSL/TLS

SSL/TLS 모듈은 HTTP 패킷이나 웹 페이지 콘텐츠를 보지 않습니다. 대신 **서버와 클라이언트가 암호화 통로를 여는 'TLS Handshake' 단계** 자체를 검사하여, 인증서 결함이나 취약한 암호화 알고리즘(Cipher Suite) 사용 여부를 판별합니다.

## SSL/TLS 모듈 작동 파이프라인 (5단계)

### 1단계: 타겟 포트 및 SNI(Server Name Indication) 치환

- `{{Hostname}}`, `{{IP}}`, `{{Port}}` 변수를 대상 주소(기본 `443` 포트 또는 RDP/LDAPS/SMTPS 등 암호화 포트)로 치환합니다.
    
- SNI(Server Name Indication) 헤더에 스캔할 도메인 주소를 매핑하여 특정 가상 호스트의 TLS 설정을 타깃팅합니다.
    

### 2단계: TCP 소켓 접속 후 TLS Handshake 시도 (`Client Hello`)

- HTTP 통신을 시작하지 않고, Go의 `crypto/tls` 패키지를 통해 대상 서버로 **TLS `Client Hello` 패킷**을 보냅니다.
    
- 이때, Nuclei는 일부러 옛날 TLS 버전(SSL v3, TLS 1.0/1.1)이나 취약한 암호화 알고리즘 목록(Cipher Suites)을 패킷에 포함시켜 서버가 받아들이는지 테스트합니다.
    

### 3단계: 서버의 TLS 응답 (`Server Hello` & 인증서) 수신

- 서버가 보내온 **`Server Hello` 패킷**, **SSL/TLS 인증서(Certificate)**, **선택된 암호화 알고리즘(Cipher Suite) 정보**를 핸드셰이크 단계에서 가로채 수집합니다.
    

### 4단계: TLS 메타데이터 구조화 (`internalEvent`)

- 서버로부터 건져 올린 암호화 관련 메타데이터를 `internalEvent` 맵에 구조화하여 저정합니다:
    
    - `internalEvent["tls_version"]`: 서버가 최종 채택한 TLS 버전 (예: `tls10`, `tls12`, `tls13`)
        
    - `internalEvent["cipher"]`: 선택된 암호화 알고리즘 (예: `TLS_RSA_WITH_AES_128_CBC_SHA`)
        
    - `internalEvent["ip"]`, `internalEvent["port"]`
        
    - `internalEvent["tls_connection_issuer"]`: 인증서 발급 기관 (CA)
        
    - `internalEvent["tls_connection_subject_an"]`: SAN(Subject Alternative Name) 등록 도메인 목록
        
    - `internalEvent["not_after"]` / `not_before`: 인증서 유효기간 시작 및 만료일
        

### 5단계: Operators 매칭, Extractor 추출 & 결과 출력

- `internalEvent` 맵 데이터를 바탕으로 암호화 설정의 보안 수준을 평가합니다.
    
    - **Matcher 예시**: `tls_version == 'tls10'` ➔ 취약한 TLS 1.0 프로토콜 활성화되어 있음 (**True**)
        
    - **Extractor 예시**: 인증서 SAN 영역에서 노출된 숨겨진 내부 서브도메인 목록 추출
        
- 조건이 충족되면 터미널에 결과를 출력합니다.
    

## SSL/TLS 모듈이 탐지하는 핵심 취약점

### 1. 만료되거나 유효하지 않은 SSL 인증서 (Expired / Invalid Certificates)

- **개념**: 인증서의 유효기간(`not_after`)이 이미 지났거나, 아직 시작되지 않은 경우, 또는 발급 기관(CA)을 신뢰할 수 없는 자체 서명(Self-Signed) 인증서인지 탐지합니다.
    
- **위험성**: 브라우저 경고 창이 발생하여 사용자 신뢰도가 떨어지고, 중간자 공격(MitM)에 취약해집니다.
    

### 2. 취약한 암호화 프로토콜 활성화 (Outdated Protocols: SSLv3, TLS 1.0/1.1)

- **개념**: 이미 암호학적으로 기밀성이 깨진 옛날 SSL/TLS 버전을 서버가 여전히 허용하고 있는지 탐지합니다.
    
- **위험성**: POODLE, BEAST 같은 공개된 암호화 무력화 공격 기법에 노출되어 통신 내용이 도청될 수 있습니다.
    

### 3. 약한 암호화 알고리즘 (Weak Cipher Suites / NULL Cipher)

- **개념**: 암호화 키 길이가 너무 짧거나(RC4, DES, 3DES, 1024-bit RSA 등), 암호화를 아예 적용하지 않는 `NULL Cipher`를 지원하는지 검사합니다.
    
- **위험성**: 무작위 대입이나 수학적 연산으로 암호화 패킷을 복호화하여 실제 통신 데이터(아이디/비밀번호)가 유출될 수 있습니다.
    

### 4. SAN(Subject Alternative Name)을 통한 내부 서브도메인 노출

- **개념**: SSL 인증서 하나로 여러 도메인을 커버하기 위해 적어둔 **SAN 항목**에서 외부로 노출되지 않은 내부 개발/테스트용 서브도메인(`dev-db.internal.example.com` 등)을 수집 및 감지합니다.
    

### 5. Heartbleed 등 프로토콜 구현체 취약점 (CVE)

- **개념**: OpenSSL 등의 TLS 라이브러리 자체에 존재하는 유명 메모리 누수 취약점(Heartbleed - CVE-2014-0160 등)을 핸드셰이크 패킷 교환 과정에서 검사합니다.
# File

File 모듈은 네트워크 소켓을 열거나 IP/포트로 통신하지 않습니다. 대신 **스캐너가 돌아가는 로컬 컴퓨터/서버의 파일 시스템(디렉터리, 소스코드, 로그, 설정 파일 등)을 직접 열어서 분석하는 모듈**입니다.

## File 모듈 작동 파이프라인 (5단계)

### 1단계: 검색 대상 파일 경로 및 변수 치환

- 스캔 대상이 네트워크 주소(`http://...`)가 아니라 디렉터리 경로(예: `/app/src`, `C:\workspace`)가 됩니다.
    
- YAML 템플릿의 `file:` 섹션에 지정된 탐색 파일 확장자(`.env`, `.py`, `.js`, `.yaml` 등)와 변수를 매핑합니다.
    

### 2단계: 로컬 디렉터리 순회 및 파일 I/O 열기 (Go `os` / `filepath`)

- 네트워크 소켓 대신, Go 언어의 `os.Open()`과 `filepath.Walk()` 함수를 사용해 **지정된 로컬 디렉터리 안의 모든 파일과 하위 폴더를 직접 차례대로 열어 메모리에 로드**합니다.
    

### 3단계: 파일 라인/전체 텍스트 읽기 (Read)

- 대용량 파일로 인한 메모리 과부하를 막기 위해 파일 콘텐츠를 **한 줄씩(Line-by-Line) 스트리밍 방식으로 읽거나 메모리 버퍼에 저장**합니다.
    

### 4단계: 파일 데이터 구조화 (`internalEvent`)

- 파일에서 읽어온 데이터와 파일 메타정보를 `internalEvent` 맵 구조체에 저장합니다:
    
    - `internalEvent["path"]`: 읽은 파일의 로컬 전체 경로 (예: `/var/www/html/.env`)
        
    - `internalEvent["line"]` / `buffer`: 파일 내부의 실제 텍스트 내용
        
    - `internalEvent["file"]`: 파일 이름
        

### 5단계: Operators 매칭, Extractor 데이터 추출 & 결과 출력

- 파일 텍스트 버퍼를 상대로 정규식 패턴(Regex)이나 특정 단어 매칭을 수행합니다.
    
    - **Matcher 예시**: `regex: "AKIA[0-9A-Z]{16}"` ➔ 소스코드 파일 안에 AWS Access Key 정규식 패턴이 노출되어 있음 (**True**)
        
    - **Extractor 예시**: 소스코드나 `.env` 파일 내부의 실제 비밀번호/토큰 데이터 문자열을 정규식으로 쏙 추출
        
- 조건이 맞으면 **파일 경로와 함께 추출된 민감 정보를 터미널에 출력**합니다.
    

## File 모듈이 탐지하는 핵심 취약점

### 1. 하드코딩된 API 키 및 비밀 인증 정보 노출 (Hardcoded Secrets)

- **개념**: 개발 과정에서 소스코드, 설정 파일, 스크립트 내부에 실수로 직접 적어둔(Hardcoded) **AWS Access Key, Google API Key, Slack Webhook URL, JWT Secret** 등을 정규식으로 탐지합니다.
    
- **위험성**: 소스코드가 GitHub 등에 유출될 경우, 즉시 외부 공격자가 인프라 제어권이나 데이터베이스에 접근할 수 있게 됩니다.
    

### 2. 설정 파일 내 DB 비밀번호 및 민감 정보 방치

- **개념**: `.env`, `config.json`, `settings.py`, `web.config` 같은 프로젝트 설정 파일 안에 암호화되지 않은 쌩(Raw) **DB 아이디/비밀번호, OAuth Client Secret**이 남아있는지 점검합니다.
    

### 3. 개인정보 및 민감 데이터 유출 (PII Leakage)

- **개념**: 로그 파일(`.log`)이나 데이터베이스 백업 파일(`.sql`, `.csv`) 내에 이메일 주소, 주민등록번호, 전화번호, 개인 암호화 키(`.pem`, `.key`)가 그대로 평문 저장되어 있는지 탐지합니다.
    

### 4. 소스코드 내 위험한 함수 사용 및 보안 미비 (Static Code Analysis)

- **개념**: 소스코드(`*.py`, `*.js`, `*.php` 등) 내부에서 보안상 위험한 명령어 실행 함수(예: Python의 `eval()`, `exec()`, PHP의 `system()`)가 사용되었는지 정적 코드 분석을 수행합니다.
# Whois

Whois 모듈은 웹 서버나 특정 포트에 직접 공격 패킷을 보내지 않습니다. 대신 **전 세계 도메인 관리 기관(전 세계 WHOIS 서버)의 43번 포트로 질의를 보내, 도메인 소유자 정보, IP 대역(ASN), 등록/만료일 등 타겟 시스템의 '인프라 및 도메인 정보'를 수집하고 점검하는 모듈**입니다.

## Whois 모듈 작동 파이프라인 (5단계)

### 1단계: 타겟 도메인/IP 변수 치환

- `{{Hostname}}`, `{{IP}}`, `{{FQDN}}` 변수를 사용자가 지정한 도메인(예: `example.com`) 또는 IP 주소(예: `1.1.1.1`)로 치환합니다.
    

### 2단계: 최상위 WHOIS 서버 43번 포트 TCP 소켓 개설

- Go 언어의 `net` 패키지를 사용해 해당 TLD(Top-Level Domain, 예: `.com`, `.kr` 등)를 담당하는 **공식 WHOIS 서버(예: `whois.verisign-grs.com`, `whois.krnic.or.kr` 등)의 43번 포트**로 TCP 소켓 연결을 생성합니다.
    

### 3단계: WHOIS 질의(Query) 텍스트 전송 (`Write`)

- HTTP나 복잡한 바이너리 규격 없이, 소켓 파이프에 조회할 도메인 텍스트 문자열 + 줄바꿈(`example.com\r\n`)을 날것(Raw) 그대로 보냅니다.
    

### 4단계: WHOIS 응답 텍스트 수신 & `internalEvent` 구조화

- WHOIS 서버가 돌려준 전체 텍스트 응답(Raw Text)을 수신하여 `internalEvent` 맵 데이터로 구조화합니다:
    
    - `internalEvent["created_date"]`: 도메인 최초 등록일
        
    - `internalEvent["updated_date"]`: 최근 정보 변경일
        
    - `internalEvent["expiration_date"]`: **도메인 만료 예정일**
        
    - `internalEvent["nameservers"]`: 연결된 네임서버 목록
        
    - `internalEvent["emails"]`: 등록자/담당자 이메일 주소
        
    - `internalEvent["asn"]`: IP가 속한 자율 시스템 번호(ASN) 및 대역 정보
        

### 5단계: Operators 매칭, Extractor 데이터 추출 & 결과 출력

- `internalEvent`에 저장된 텍스트 데이터를 정규식이나 조건식(Matcher)으로 검증합니다.
    
    - **Matcher 예시**: `expiration_date < 30_days` ➔ 도메인 만료가 30일 이내로 다가옴 (**True**)
        
    - **Extractor 예시**: WHOIS 응답 텍스트 내에서 담당자 이메일, ASN 번호, 네임서버 주소만 정규식으로 쏙 추출
        
- 조건이 충족되면 모니터링 결과를 터미널에 출력합니다.
    

## Whois 모듈이 탐지하는 핵심 취약점 및 보안 위협

### 1. 도메인 만료 임박 (Domain Expiration Risk)

- **개념**: 도메인의 만료 예정일(`expiration_date`)이 얼마 남지 않았는지(예: 7일/30일 이내) 감지합니다.
    
- **위험성**: 관리자 실수로 도메인을 갱신하지 않아 만료되면, 공격자가 해당 도메인을 선점(Domain Hijacking)하여 서비스 마비 및 피싱 사이트로 악용할 수 있습니다.
    

### 2. 등록자 개인정보 노출 (Registrant PII Exposure)

- **개념**: WHOIS 보호 서비스(Privacy Shield/Proxy Service)가 적용되지 않아 도메인 소유자의 **실제 이름, 개인 이메일, 전화번호, 회사 주소**가 대외에 그대로 노출되었는지 탐지합니다.
    
- **위험성**: 노출된 담당자 이메일/전화번호를 바탕으로 타깃 스피어 피싱(Spear Phishing)이나 사회공학적 공격이 들어올 수 있습니다.
    

### 3. 네임서버(NS) 설정 변경 및 탈취 위험 감지

- **개념**: 도메인에 연결된 네임서버가 존재하지 않거나, 보안이 취약한 외부 네임서버 주소가 등록되어 있는지 검사합니다.
    

### 4. 자산 식별 및 ASN 기반 공격 표면(Attack Surface) 파악

- **개념**: IP/도메인이 속한 ASN(Autonomous System Number) 정보와 등록 기관 정보를 조회하여, 회사가 소유한 전체 IP 대역과 인프라 자산 규모를 식별합니다.
# Websocket

일반 HTTP 통신은 클라이언트가 요청을 보낼 때만 서버가 응답하고 연결을 끊지만, 웹소켓(WebSocket)은 한 번 연결을 해두면 통로를 계속 열어둔 채 양방향으로 데이터를 주고받는 실시간 통신 방식(`ws://`, `wss://`)입니다.

Websocket 모듈은 **핸드셰이크로 연결을 승격(Upgrade)시킨 후, 열린 실시간 데이터 통로(Frame)로 공격 페이로드를 밀어 넣어 보안 허점을 찾는 모듈**입니다.

## Websocket 모듈 작동 파이프라인 (5단계)

### 1단계: 웹소켓 주소(`ws://`, `wss://`) 및 변수 치환

- `{{Hostname}}`, `{{Path}}` 등의 변수를 스캔할 웹소켓 엔드포인트(예: `wss://[example.com/chat](https://example.com/chat)` 또는 `/socket.io/`)로 치환합니다.
    

### 2단계: HTTP Upgrade 핸드셰이크 요청 (연결 승격)

- 일반 HTTP 요청이 아니라, 서버에게 "나 이제 웹소켓 실시간 통신으로 전환할게!"라고 요청하는 헤더를 실어 보냅니다.
    
    - `Upgrade: websocket`
        
    - `Connection: Upgrade`
        
    - `Sec-WebSocket-Key: ...`
        
- 서버가 `101 Switching Protocols` 응답을 돌려주면, 지속적인 실시간 TCP 통신 채널이 생성됩니다.
    

### 3단계: 웹소켓 프레임(Frame) 메시지 송신 (`Write`)

- 열린 통로를 통해 텍스트(JSON/String) 또는 바이너리 웹소켓 프레임 메시지(Payload)를 연속해서 서버로 전송합니다.
    
- _예시_: 채팅 입력창이나 실시간 데이터 요청 프레임 속에 공격용 구문(`{"message": "' OR 1=1 --"}`)을 실어 보냅니다.
    

### 4단계: 웹소켓 스트림 응답 수신 & `internalEvent` 구조화

- 서버가 실시간 프레임으로 돌려주는 응답 메세지 스트림(Text/Binary Frame)을 읽어 수집합니다.
    
- 수신한 데이터와 웹소켓 헤더를 `internalEvent` 맵에 구조화하여 저장합니다:
    
    - `internalEvent["success"]`: 웹소켓 핸드셰이크 성공 여부 (True/False)
        
    - `internalEvent["response"]` / `buffer`: 서버가 실시간 프레임으로 돌려준 메시지 내용
        
    - `internalEvent["header"]`: 핸드셰이크 시 반환된 HTTP 헤더 목록
        

### 5단계: Operators 매칭, Extractor 추출 & 결과 출력

- `internalEvent`에 모인 실시간 응답 버퍼 데이터를 검증합니다.
    
    - **Matcher 예시**: `contains(buffer, 'SQL syntax error')` ➔ 웹소켓 메세지 처리 로직에 SQL Injection 발생 (**True**)
        
    - **Matcher 예시 (CSWSH)**: `Origin` 헤더를 조작해 요청했을 때 `101 Switching Protocols` 응답이 성공적으로 돌아옴 (**True**)
        
- 조건이 맞으면 터미널에 결과를 출력합니다.
    

## Websocket 모듈이 탐지하는 핵심 취약점

### 1. CSWSH (Cross-Site WebSocket Hijacking, 크로스 사이트 웹소켓 탈취)

- **개념**: 웹소켓 핸드셰이크 시 서버가 **`Origin` 헤더(요청을 보낸 출처)를 올바르게 검증하는지** 테스트합니다.
    
- **위험성**: 악성 웹사이트에 접속한 피해자의 브라우저가 사용자 쿠키/인증 정보를 실어 해커가 지정한 웹소켓으로 몰래 연결을 생성하고, 피해자의 실시간 데이터를 탈취할 수 있습니다.
    

### 2. 웹소켓 프레임 기반 취약점 (SQLi, XSS, Command Injection)

- **개념**: 웹소켓 메세지(`{"action": "search", "query": "..."}`) 입력값에 일반 웹 취약점 페이로드를 실어 보냈을 때, DB 에러가 발생하거나 자바스크립트가 반사되어 오는지 탐지합니다.
    
- **위험성**: 웹 응답 화면(HTTP Body)에는 나타나지 않더라도, 실시간 웹소켓 통신을 처리하는 백엔드 DB나 서버 명령어가 뚫릴 수 있습니다.
    

### 3. 미인증 웹소켓 엔드포인트 접근 (Unauthenticated Access)

- **개념**: 세션 쿠키나 JWT 토큰 없이 웹소켓 연결(`101` 응답)을 시도했을 때, 서버가 인증을 거부하지 않고 실시간 내부 메세지 스트림(주식 체결가, 타인의 채팅, 관리자 로그 등)을 그대로 내주는지 검사합니다.
    

### 4. 웹소켓 DoS (Denial of Service) 및 메시지 사이즈 제한 미비

- **개념**: 웹소켓 연결을 끊지 않고 유지한 상태에서 엄청나게 용량이 큰 프레임 패킷을 연속으로 발송하여 서버 메모리를 고갈시키거나 다운되는지 점검합니다.
# Headless 

Headless 모듈은 단순히 HTTP 패킷만 주고받는 일반 스캐너와 다릅니다. **실제 화면만 안 뜰 뿐, 실제 크롬(Chrome) 브라우저 엔진을 뒤에서 직접 실행(Headless Chrome)시켜 자바스크립트를 실행하고, DOM 요소를 직접 클릭하고 입력하며 정밀 스캔을 수행하는 모듈**입니다.

## Headless 모듈 작동 파이프라인 (5단계)

### 1단계: 타겟 URL 변수 치환 & 브라우저 제어 동작(Actions) 정의

- `{{BaseURL}}` 변수를 대상 웹페이지 주소로 치환합니다.
    
- YAML 템플릿의 `headless:` 섹션에 정의된 브라우저 자동화 시나리오(Click, Type, Wait, Navigate 등)를 읽어옵니다.
    

### 2단계: Headless Chrome 인스턴스 실행 & CDPSession 연결

- 스캐너 백그라운드에 실제 크롬 브라우저 프로세스(Headless Chrome)를 띄웁니다.
    
- CDP (Chrome DevTools Protocol)를 통해 브라우저와 통신 채널을 뚫고, 자바스크립트 실행 및 이벤트 감시 설정을 완료합니다.
    

### 3단계: 웹페이지 로드, DOM 렌더링 & 사용자 동작 시뮬레이션

- 브라우저로 대상 페이지에 접속하여 **React, Vue, Angular 등 싱글 페이지 애플리케이션(SPA)의 자바스크립트 코드까지 완전히 실행 및 렌더링**합니다.
    
- 정의된 시나리오에 따라 브라우저 동작을 직접 수행합니다:
    
    - `navigate`: 페이지 이동
        
    - `type`: 입력창에 공격 페이로드(`"<script>alert(1)</script>"`) 직접 타이핑
        
    - `click`: 버튼 클릭 이벤트 발생
        
    - `wait`: DOM 변화나 자바스크립트 실행 완료 대기
        

### 4단계: DOM 스냅샷, 자바스크립트 실행 결과 및 네트워크 수집 (`internalEvent`)

- 브라우저에서 동적으로 최종 완성된 **DOM HTML 스냅샷**, **자바스크립트 콘솔 로그/알림창(Dialog) 팝업 여부**, **네트워크 트래픽 목록**을 수집하여 `internalEvent` 맵에 구조화합니다:
    
    - `internalEvent["data"]`: 브라우저가 화면에 최종 렌더링한 완성본 HTML
        
    - `internalEvent["type"]`: 팝업 이벤트 발생 여부 (`alert`, `confirm` 등)
        
    - `internalEvent["history"]`: 이동한 브라우저 URL 경로 기록
        

### 5단계: Operators 매칭, Extractor 데이터 추출 & 결과 출력

- 렌더링된 최종 DOM 텍스트와 브라우저 이벤트 결과를 평가합니다.
    
    - **Matcher 예시**: `type == 'alert'` ➔ 브라우저 내에서 자바스크립트 `alert()` 창이 실제로 실행되어 XSS 성공 (**True**)
        
    - **Extractor 예시**: 브라우저 로컬 스토리지(LocalStorage)나 동적 DOM에 노출된 JWT 토큰/API 키 추출
        
- 조건이 충족되면 결과를 출력합니다.
    

## Headless 모듈이 탐지하는 핵심 취약점

### 1. DOM-based XSS (DOM 기반 크로스 사이트 스크립팅)

- **개념**: 단순 HTTP 응답 텍스트에는 나타나지 않고, 브라우저가 자바스크립트를 실행하는 과정에서 `document.location`, `element.innerHTML` 등을 거쳐 실행되는 취약점을 탐지합니다.
    
- **위험성**: 일반 HTTP 스캐너는 자바스크립트를 실행하지 못해 절대 찾을 수 없지만, Headless 모듈은 **진짜 `alert()` 창이 뜨는 순간을 잡아내어 완벽히 탐지**할 수 있습니다.
    

### 2. SPA (Single Page Application) 및 동적 웹앱 보안 취약점

- **개념**: React, Vue, Angular로 작성된 현대적인 웹 애플리케이션처럼 **사용자 클릭이나 입력 이벤트가 발생해야 비로소 API를 호출하거나 화면이 바뀌는 동적 페이지**의 보안 구멍을 탐지합니다.
    

### 3. CSRF (Cross-Site Request Forgery) 및 UI Redressing (클릭재킹)

- **개념**: 브라우저 세션 상태에서 동적으로 폼(Form) 요소가 생성되거나, `iframe` 내에 타겟 페이지가 렌더링되는 지점을 시뮬레이션하여 클릭재킹 및 CSRF 방어 미비점을 검사합니다.
    

### 4. DOM 소스 내 노출된 민감 정보 (LocalStorage / Cookie / DOM Leakage)

- **개념**: 자바스크립트 실행 완료 후 `window.localStorage`, `sessionStorage`, 동적으로 삽입된 `<script>` 태그 내에 노출되는 민감한 인증 토큰이나 비밀 정보를 수집합니다.
# Javascript

Javascript 모듈은 브라우저에서 화면을 띄우는 게 아닙니다. 대신 **Nuclei 내부에 탑재된 자바스크립트 엔진(Goja)을 이용해, 스캐너 자체가 고난도 다단계 복합 프로토콜 통신(커스텀 핸드셰이크, 복잡한 바이너리 암호화 연산 등)을 자유자재로 조합해 실행하는 모듈**입니다.

## Javascript 모듈 작동 파이프라인 (5단계)

### 1단계: 타겟 변수 치환 & JS 스크립트 코드 로드

- `{{Hostname}}`, `{{Port}}` 변수를 치환합니다.
    
- YAML 템플릿의 `javascript:` 섹션에 작성된 **인라인 자바스크립트 코드 스크립트**를 읽어옵니다.
    

### 2단계: 내장 자바스크립트 샌드박스 엔진(Goja) 가동

- 외부 Node.js를 설치할 필요 없이, Nuclei 내부의 Go 전용 임베디드 JS 엔진(Goja)을 가동합니다.
    
- 샌드박스 내부에서 안전하게 실행할 수 있도록 Nuclei 전용 통신 라이브러리(Network, SSH, FTP, Crypto, Struct 연산 등)를 JS 객체 형태로 로딩합니다.
    

### 3단계: 커스텀 JS 통신 로직 및 다단계 스크립트 실행

- 단순 패킷 전송이 아닌, JS 코드에 작성된 **조건문, 반복문, 복잡한 비트 연산, 암호화 패킷 조합 로직**을 순차적으로 수행합니다.
    
- _예시_: "1) SSH 소켓 연결 -> 2) 커스텀 인증 패킷 암호화 후 전송 -> 3) 응답받은 키 값 복호화 -> 4) 조건에 따라 2차 패킷 재전송" 같은 다단계 통신 시나리오를 직접 수행합니다.
    

### 4단계: JS 실행 결과값 반환 및 `internalEvent` 구조화

- 자바스크립트 스크립트가 실행을 마치고 `set("result", true)` 또는 데이터 객체를 리턴하면, 이를 `internalEvent` 맵 데이터로 구조화합니다:
    
    - `internalEvent["response"]`: JS 코드 실행 결과로 반환된 최종 텍스트/바이너리 데이터
        
    - `internalEvent["success"]`: JS 내부 로직 판단 결과 (True/False)
        

### 5단계: Operators 매칭, Extractor 추출 & 결과 출력

- JS 스크립트의 리턴값과 `internalEvent` 데이터를 바탕으로 조건식을 판정합니다.
    
    - **Matcher 예시**: `success == true` ➔ 다단계 커스텀 핸드셰이크 인증 우회 성공 (**True**)
        
- 최종 평가 후 취약점 결과를 터미널에 출력합니다.
    

## Javascript 모듈이 탐지하는 핵심 취약점

### 1. 복잡한 다단계 인증 및 커스텀 프로토콜 취약점

- **개념**: 표준 HTTP/TCP 규격이 아니고, 패킷을 여러 번 주고받으며 매번 난수(Nonce)나 세션 키를 암호화해서 응답해야 하는 특수 서비스(특정 VPN 장비, IoT 장비, 자체 제작 프로토콜)의 취약점을 탐지합니다.
    

### 2. SSH / FTP / RDP / LDAP 등 비-웹 네트워크 서비스 정밀 점검

- **개념**: 단순 소켓 텍스트 송신으로는 불가능한 **SSH Key 인증, LDAP 바인딩 테스트, SMB 복잡한 세션 협상** 등 고도화된 비-웹 서비스 취약점을 정밀 검사합니다.
    

### 3. 암호화 연산이 필요한 복잡한 RCE (원격 코드 실행) 페이로드 생성

- **개념**: 공격 패킷을 보낼 때 타겟 서버의 고유 Key로 AES/RSA 암호화를 하거나 CRC32 체크섬을 계산해서 붙여야만 동작하는 복잡한 CVE 취약점(CVE-2023-xxxx 등)을 직접 구현하여 스캔합니다.
    

### 4. 조건별 분기 탐지 (Stateful Vulnerability Scanning)

- **개념**: "A 응답이 올 경우 B 패킷을 보내고, C 응답이 올 경우 D 패킷을 보낸다"와 같이 서버의 반응에 따라 실시간으로 공격 시나리오를 바꾸는 **상태 기반(Stateful) 취약점**을 스캔합니다.
# Code

Code 모듈은 외부 원격 서버로 네트워크 패킷을 쏘는 모듈이 아닙니다. 대신 **Nuclei 스캐너가 실행 중인 로컬 운영체제(OS)의 쉘(Bash, PowerShell, Python 등) 명령어를 직접 호출하여, 로컬 시스템 환경 점검 및 권한 상승 취약점 등을 검사하는 모듈**입니다.

## Code 모듈 작동 파이프라인 (5단계)

### 1단계: 실행 언어(Engine) 및 실행 명령어(Code) 정의

- YAML 템플릿의 `code:` 섹션에 지정된 실행 환경(`engine: sh`, `bash`, `powershell`, `python` 등)과 실행할 명령어 스크립트를 읽어옵니다.
    
- 스캔 대상 변수가 필요한 경우(예: 로컬 네트워크 IP 대역 등) 해당 변수를 치환합니다.
    

### 2단계: 로컬 OS 서브프로세스(Subprocess) 생성

- Go의 `os/exec` 패키지를 사용해 Nuclei 프로세스 하위에 **실제 운영체제 쉘 프로세스(`cmd.exe`, `/bin/bash` 등)를 서브프로세스로 안전하게 생성**합니다.
    
- 안전을 위해 `-code` 옵션을 직접 부여해 CLI를 실행해야만 이 모듈이 동작하도록 시스템적 제어가 적용됩니다.
    

### 3단계: 로컬 시스템 Shell 명령어 실행 (`exec`)

- 템플릿에 작성된 로컬 스크립트 명령어를 쉘로 전달하여 실행합니다.
    
    - _예시 (Linux)_: `uname -a`, `cat /etc/os-release`, `sudo -l`
        
    - _예시 (Windows)_: `whoami /priv`, `systeminfo`
        

### 4단계: 표준 출력(stdout) / 표준 에러(stderr) 수집 & `internalEvent` 구조화

- OS 쉘이 실행 후 돌려준 표준 출력(stdout) 및 에러 메세지(stderr)를 수집하여 `internalEvent` 맵 데이터에 저장합니다:
    
    - `internalEvent["code_response"]` / `buffer`: 명령어가 화면에 출력한 텍스트 결과
        
    - `internalEvent["exit_code"]`: 명령어의 프로세스 종료 코드 (`0`: 성공, `1`: 에러 등)
        

### 5단계: Operators 매칭, Extractor 데이터 추출 & 결과 출력

- 명령어 결과 출력 버퍼(`code_response`)를 기반으로 취약점 조건을 평가합니다.
    
    - **Matcher 예시**: `uname -r` 결과 버전이 `5.10.0-14-amd64` 이하이고 `contains(buffer, 'Debian')` ➔ 해당 Linux 커널 권한 상승 취약점(CVE) 대상임 (**True**)
        
    - **Extractor 예시**: `sudo -l` 결과에서 비밀번호 없이 실행 가능한 명령어 목록만 정규식으로 추출
        
- 조건을 만족하면 결과를 터미널에 출력합니다.
    

## Code 모듈이 탐지하는 핵심 취약점 및 활용처

### 1. 로컬 권한 상승 취약점 (Local Privilege Escalation / LPE)

- **개념**: 침투 테스트 시 타겟 서버 내부 계정을 얻은 상태에서, 패치되지 않은 Linux 커널 취약점(Dirty Pipe, Pkexec 등)이나 **Windows 권한 상승 취약점**이 해당 서버에 존재하는지 탐지합니다.
    

### 2. OS 설정 오류 및 보안 위협 점검 (OS Misconfigurations)

- **개념**: Linux의 `SUID` / `SGID` 권한이 잘못 설정된 파일, `sudoers` 내 비밀번호 없는 명령어 실행(`NOPASSWD`) 허용 여부, Windows 서비스 권한 설정 오류 등을 내부에서 직접 명령어로 검사합니다.
    

### 3. 내부 외부 툴(External Security Tools) 연동

- **개념**: Go나 HTTP로 구현하기 힘든 특수 보안 검사 도구(예: `nmap`, `masscan`, `smbclient` 등)를 Nuclei가 직접 로컬 셸로 호출하여 실행하고, 그 실행 결과를 가져와 분석할 때 사용합니다.
    

### 4. 로컬 패치 상태 및 소프트웨어 버전 검사

- **개념**: 서버 내부에 설치된 주요 패키지 및 소프트웨어(OpenSSL, Nginx, Docker 등)의 설치 버전을 명령어로 조회하여 보관 및 패치 대상인지 점검합니다.

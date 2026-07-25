# Nuclei와 AI 기능 연결

#전지성 #Nuclei #AI #Matcher #ResultEvent #OutputWriter

> [!info]- 논문 용어 바로가기
> [[05_논문 전용 용어 사전#^term-ai-ml|AI·머신러닝]] · [[05_논문 전용 용어 사전#^term-ai-model|모델]] · [[05_논문 전용 용어 사전#^term-dataset|데이터셋]] · [[05_논문 전용 용어 사전#^term-label|레이블]] · [[05_논문 전용 용어 사전#^term-feature|특징]] · [[05_논문 전용 용어 사전#^term-preprocessing|전처리]] · [[05_논문 전용 용어 사전#^term-class-imbalance|데이터 불균형]]
> [[05_논문 전용 용어 사전#^term-accuracy|정확도]] · [[05_논문 전용 용어 사전#^term-precision|정밀도]] · [[05_논문 전용 용어 사전#^term-recall|재현율]] · [[05_논문 전용 용어 사전#^term-f1-score|F1-score]] · [[05_논문 전용 용어 사전#^term-generalization|일반화]] · [[05_논문 전용 용어 사전#^term-reproducibility|재현성]]
> [[05_논문 전용 용어 사전#^term-agent|Agent]] · [[05_논문 전용 용어 사전#^term-ai-tool|Tool]] · [[05_논문 전용 용어 사전#^term-guardrail|Guardrail]] · [[05_논문 전용 용어 사전#^term-prompt-injection|Prompt Injection]] · [[05_논문 전용 용어 사전#^term-confidence-score|Confidence Score]]
> [[05_논문 전용 용어 사전#^term-fixture|Fixture]] · [[05_논문 전용 용어 사전#^term-test-oracle|Test Oracle]] · [[05_논문 전용 용어 사전#^term-redaction|Redaction]] · [[05_논문 전용 용어 사전#^term-manifest|Manifest]] · [[05_논문 전용 용어 사전#^term-scan-coverage|Scan Coverage]]
> [[05_논문 전용 용어 사전#^term-code-generation-ai|Code Generation AI]] · [[05_논문 전용 용어 사전#^term-rpa|RPA]] · [[05_논문 전용 용어 사전#^term-selenium|Selenium]] · [[05_논문 전용 용어 사전#^term-prompt-engineering|Prompt Engineering]] · [[05_논문 전용 용어 사전#^term-negative-control|Negative Control]]

## 선행연구의 흐름

```text
Bitscanner
자동 스캐너의 기본 구조
    ↓
홈페이지 분석모델·진단 방법 개선
자동 결과에는 오탐과 미탐이 존재함
    ↓
정적·동적 크로스 체크
다른 분석 방법으로 결과를 재검증
    ↓
AI 동적 분석
고정 규칙의 한계를 머신러닝으로 보완
    ↓
LSTM HAST
SAST·DAST·AI를 하나의 구조로 통합
```

## Nuclei의 현재 판정 흐름

```text
InputProvider
검사 대상 공급
    ↓
Catalog·Loader
Template 선택·로드
    ↓
Parser·Compiler
Template을 실행 가능한 형태로 준비
    ↓
Engine
대상과 Template 실행
    ↓
HTTP·DNS·SSL 등의 Request
    ↓
Matcher
응답이 탐지 조건에 맞는지 판정
    ↓
ResultEvent
탐지 결과를 구조체로 정리
    ↓
Output Writer
터미널과 파일에 결과 출력
```

## 제안 기능: AI Evidence Explainer

Nuclei의 Matcher가 탐지 여부를 결정하고, AI는 이미 만들어진 결과의 근거를 설명한다.

```text
Matcher가 탐지
    ↓
ResultEvent 생성
    ↓
민감정보 제거
    ↓
Template·Matcher·탐지 근거 수집
    ↓
AI Evidence Analyzer
    ↓
탐지 이유·영향·수동 검증 방법 생성
    ↓
Output Writer가 함께 출력
```

### 예상 출력

```text
템플릿:
weak-cipher-suites

무엇을 검사했는가:
서버가 오래된 TLS 버전과 CBC 암호화 방식을 허용하는지 검사했다.

왜 탐지됐는가:
TLS 1.0에서 TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA가 허용되었다.

추가 확인:
1. 다른 TLS 검사 도구로 다시 확인
2. 서버 또는 로드 밸런서 TLS 설정 확인
3. TLS 1.0 비활성화 후 재검사
```

## AI가 최종 판정하면 안 되는 이유

```text
잘못된 설계
AI가 취약점인지 아닌지 최종 결정

권장 설계
Matcher가 탐지 여부 결정
AI는 탐지 근거와 검증 방법 설명
사람이 최종 판단
```

AI는 사실과 다른 설명을 생성할 수 있다. 따라서 원래 탐지 결과와 AI 설명을 분리해야 한다.

```json
{
  "matched": true,
  "matcher_result": "Nuclei가 결정한 결과",
  "ai_explanation": "AI가 만든 참고 설명",
  "ai_is_authoritative": false
}
```

## 선행연구별 활용 지점

| 논문 | Nuclei AI 기능에 활용할 내용 |
|---|---|
| Bitscanner | 요청·응답 기반 자동 스캐너와 보고서 구조 |
| 홈페이지 분석모델 | 자동 결과를 그대로 확정하지 않고 추가 검증 |
| AI 기반 동적 분석 | 특징 추출, 학습·평가 지표, 데이터 불균형 처리 |
| 정적·동적 크로스 체크 | 다른 분석 결과로 탐지 결과 재검증 |
| LSTM HAST | AI와 SAST·DAST를 연결하는 전체 아키텍처 |
| 웹사이트 구조화 | 검사하지 못한 페이지와 기능의 커버리지 관리 |
| 진단 방법 개선 | 단순 패턴보다 문맥과 데이터 흐름이 중요함 |

## 실험 설계에서 지켜야 할 점

AI 동적 분석 논문은 OWASP ZAP과 AI를 서로 다른 데이터와 환경에서 비교했다는 한계를 가진다.

Nuclei AI 기능을 실험할 때는 다음 조건을 동일하게 유지해야 한다.

```text
같은 대상
같은 취약점 데이터
같은 요청
같은 정답 레이블
같은 평가 기준
    ↓
기존 Nuclei 결과와 AI 보완 결과 비교
```

### 필요한 평가 지표

- 정밀도(Precision): 취약하다고 탐지한 것 중 실제 취약점 비율
- 재현율(Recall): 실제 취약점 중 도구가 찾아낸 비율
- F1-score: 정밀도와 재현율의 균형
- 오탐률(False Positive Rate)
- 미탐률(False Negative Rate)
- 평균 분석 시간
- AI 설명에 대한 전문가 평가

## 개인정보 보호

HTTP 요청과 응답에는 다음 정보가 포함될 수 있다.

- Authorization 헤더
- Cookie
- API Key
- 로그인 세션
- 사용자 개인정보

따라서 원본 요청과 응답을 외부 AI에 그대로 전달하면 안 된다.

```text
ResultEvent
    ↓
Authorization·Cookie·Token 제거
    ↓
Matcher와 관련된 부분만 추출
    ↓
AI에 전달
```

가능하면 로컬 AI 모델을 사용하고, 외부 AI를 사용하는 경우 Nuclei의 Redact 처리 이후 데이터만 전달하도록 설계한다.

## 코드 추가 위치 초안

```text
cmd/nuclei/main.go
└─ -ai-explain 옵션 등록

pkg/types/types.go
└─ Options에 AIExplain 필드 추가

pkg/output/output.go
└─ ResultEvent를 AI 분석기로 전달

pkg/aiexplain/
├─ analyzer.go
├─ evidence.go
├─ prompt.go
└─ result.go
```

## 연구 주제 문장

> Nuclei의 Template과 Matcher가 생성한 탐지 결과를 유지하면서, 탐지 요청·응답과 Matcher 조건을 AI가 추가 분석하여 탐지 근거, 오탐 가능성 및 수동 검증 방법을 제공하는 설명 모듈을 제안한다.

## 추가 AI 오픈소스에서 배울 점

### PentestGPT

[PentestGPT](https://github.com/GreyDGL/PentestGPT)는 긴 침투 테스트 작업을 추론, 명령 생성, 결과 해석으로 나눈다.

```text
Reasoning
현재 상황과 다음 목표 판단
    ↓
Generation
도구가 실행할 명령 생성
    ↓
Parsing
긴 결과에서 중요한 사실 추출
    ↓
Reasoning으로 다시 전달
```

Nuclei에 적용할 때도 Template 선택 설명, 결과 설명, 수동 검증 제안을 한 AI 함수에 모두 맡기지 않고 책임을 분리하는 편이 좋다.

### CAI

[CAI](https://github.com/aliasrobotics/CAI)는 AI Agent가 보안 Tool을 호출할 수 있게 만든 Framework다. 여기서 중요한 부분은 Agent의 능력뿐 아니라 `Guardrail`이다.

```text
사용자 요청
    ↓
Input Guardrail
Prompt Injection·금지된 행동 검사
    ↓
Agent가 Tool 선택
    ↓
실행 직전 명령과 Scope 재검사
    ↓
허용된 대상에서만 실행
```

Nuclei AI 기능이 명령을 만들 수 있다면 다음을 제한해야 한다.

- 사용자가 허용한 Target만 검사
- `dos`, `fuzz`, `intrusive` 성격의 검사를 자동 활성화하지 않음
- 비밀값을 다른 Target으로 전달하지 않음
- AI가 만든 Template을 검증 없이 실행하지 않음
- 실행 전 사용자가 실제 요청과 위험도를 확인할 수 있게 함

### Vulnhuntr

[Vulnhuntr](https://github.com/protectai/vulnhuntr)는 LLM이 Python 코드의 외부 입력부터 위험한 함수까지 호출 관계를 따라가도록 필요한 함수·클래스 문맥을 반복해서 제공한다.

Nuclei에 적용할 수 있는 부분은 다음과 같다.

```text
Template 전체를 무조건 AI에 전달
        X

탐지된 Matcher
관련 Request
일치한 Response 부분
관련 Extractor
        ↓
필요한 근거만 AI에 전달
```

Vulnhuntr의 Confidence Score는 AI가 계산한 참고 점수다. Nuclei의 Matcher 결과와 같은 확정적 사실로 저장하면 안 된다.

## 코드 생성 AI 웹 점검 논문과 Nuclei

[[02_논문별 상세 분석#17. 코드 생성 AI를 이용한 웹 취약점 점검 방법|코드 생성 AI 논문]]은 Cursor와 Continue에 자연어 Prompt를 입력하여 Python·Selenium 점검 코드를 생성한다.

```text
논문의 방식
자연어 Prompt
    ↓
Cursor·Continue
    ↓
Selenium Code
    ↓
Browser 자동화
    ↓
화면 Capture·결과 기록

Nuclei의 방식
자연어 Prompt
    ↓
Nuclei -ai
    ↓
YAML Template
    ↓
Nuclei Engine
    ↓
Matcher 판정·ResultEvent·Output Writer
```

두 방법의 가장 큰 차이는 생성된 코드를 누가 책임지고 실행하는가다.

- 논문의 Selenium Code는 Browser 초기화, Element 탐색, Timeout, 예외 처리와 판정 기준을 코드마다 구현한다.
- Nuclei Template은 Engine의 Protocol Client, Matcher, Extractor, Rate Limiter와 Output 구조를 재사용한다.

논문에서도 최초 생성 Code가 의도와 다르거나 오류가 발생해 전문가가 Prompt로 수정했다고 설명한다. 따라서 Nuclei에 필요한 새 기능은 AI Template 생성 자체보다 생성 결과의 검증이다.

```text
AI Template 생성
    ↓
문법 검증
    ↓
위험한 Payload·Protocol·Callback 검사
    ↓
정상·취약 Response Fixture 실행
    ↓
Matcher 오탐·미탐 측정
    ↓
전문가 승인
    ↓
허가된 Target에서 실행
```

### 논문 수치를 사용할 때 주의

- 결론은 점검 시간을 약 40% 단축했다고 주장한다.
- 본문 사례는 코드 생성 AI 약 20분, 기존 방식 약 1시간이라고 설명한다.
- 20분 대 60분을 계산하면 약 66.7% 단축이므로 두 수치가 일치하지 않는다.
- `오탐률 0%`를 제시하지만 TP·TN·FP·FN과 전체 Test Case 수가 없다.
- 정확한 AI Model과 Version, 반복 실행 횟수, 전문가 수정량도 공개하지 않는다.

따라서 연구 아이디어 참고자료로는 유용하지만 Nuclei AI 기능의 정확도 목표값으로 0% 오탐률이나 40% 단축을 그대로 사용하면 안 된다.

## Nuclei에 이미 있는 기능과 중복 검토

새로운 기능을 제안하기 전에 현재 Nuclei 기능과 비교해야 한다.

| 제안처럼 보이는 기능 | 현재 Nuclei에 있는 관련 기능 | 그대로 제안하면 생기는 문제 |
|---|---|---|
| AI가 Template 작성 | `-ai` AI Template 생성 | 이미 제공되는 기능 |
| 자연어로 보안 점검 코드 생성 | `-ai`와 Code·JavaScript Protocol | 생성 기능보다 품질·안전 검증으로 차별화 필요 |
| 여러 요청을 순서대로 실행 | Workflow·Flow·공유 실행 Context | 기존 기능의 이름만 바꿀 수 있음 |
| Parameter Fuzzing | `-dast`, Fuzzing Template | 이미 DAST/Fuzzing 지원 |
| 새 Template만 검사 | `-nt`, `-ntv` | 이미 새 Template 선택 가능 |
| 중단 후 이어서 검사 | `-resume` | 이미 Resume 지원 |
| OpenAPI 검사 | OpenAPI·Swagger Input Mode | 이미 입력 형식 지원 |
| 이전 결과 중복 제거 | Report Database와 Reporting Module | 고유 Issue 관리 기능과 일부 중복 |
| 기술을 보고 Template 선택 | Automatic Scan | 기술 탐지와 Tag Mapping 기능 존재 |
| JSON 보고서 생성 | JSONL·SARIF·Markdown 등 Output | 단순 출력 형식 추가만으로는 새 연구가 약함 |

이 표의 뜻은 관련 기능을 절대 연구하면 안 된다는 것이 아니다. 기존 기능이 하지 못하는 범위를 정확히 밝혀야 한다.

예:

```text
기존 -validate
└─ Template 문법과 구조가 올바른지 검사

새 Template 오탐 내성 시험기
└─ 문법은 올바르지만 정상·WAF·404 응답을 잘못 탐지하는지 검사
```

## WAVE 논문과 오탐 감소 기능의 연결

[[02_논문별 상세 분석#18. 웹 애플리케이션 취약점 분석 시스템|WAVE 논문]]은 2008년에 이미 룰 실행 전 대상 페이지의 적합성을 확인하는 방법을 제안했다.

```text
고정 경로와 파일 이름만 확인
    ↓
다른 프로그램에 잘못된 룰 적용 가능

WAVE
JavaScript 함수·변수·링크·이미지 특성 비교
    ↓
같은 프로그램의 페이지인지 확인
    ↓
관련 룰만 실행
```

따라서 `페이지 특징을 확인해 관련 Template만 실행한다`는 아이디어만으로는 완전히 새로운 연구라고 보기 어렵다. Nuclei에도 Automatic Scan이 있어 탐지한 기술과 Tag를 연결해 Template을 선택한다.

새 기능으로 차별화하려면 다음 범위를 포함해야 한다.

- WAVE의 페이지 특성 비교와 Nuclei Automatic Scan의 기술 기반 선택을 실험으로 비교
- 단순 제품 탐지가 아니라 각 Template이 기대하는 페이지·응답 특징을 선언
- Custom 404, WAF 차단, 로그인 Redirect 같은 정상·방어 응답을 Negative Control로 함께 실행
- 사전 필터로 줄인 요청 수뿐 아니라 필터 때문에 놓친 취약점 수도 측정
- 특징 일치 점수는 보조 정보로 사용하고, 최종 취약 판정은 Matcher 결과와 분리
- AI를 사용할 경우 AI의 판단만 믿지 않고 재현 가능한 특징과 근거를 함께 출력

## 새로운 기능 후보

### 1. 교육용 실행 추적 모드

사용자가 `왜 이 Template이 실행되었고 왜 결과가 나왔는가?`를 단계별로 확인한다.

```text
-tags cve 입력
    ↓
Loader가 cve Template 선택
    ↓
Protocol과 Target 형식 확인
    ↓
실제 Request 생성
    ↓
Response 수신
    ↓
각 Matcher의 true·false 표시
    ↓
matcher-condition으로 최종 결과 계산
```

예상 사용법:

```bash
nuclei -u https://example.com -t custom-template.yaml -explain
```

예상 출력:

```text
[1] custom-template.yaml 선택 이유: 사용자가 -t로 직접 지정
[2] HTTP Request #1 실행
[3] status matcher: true (응답 200)
[4] word matcher: false ("vulnerable" 없음)
[5] matcher-condition: and
[6] 최종 결과: 탐지하지 않음
```

예상 연결 지점:

- `internal/runner`: 옵션과 전체 실행 모드 연결
- `pkg/catalog/loader`: Template 선택·제외 이유 기록
- `pkg/templates`: Parse·Compile 단계 기록
- `pkg/operators`: Matcher·Extractor 판정 근거 기록
- `pkg/output`: 사람이 읽을 수 있는 Trace 출력

이 기능은 Scanner 성능을 직접 높이지 않지만 학습, Template Debugging, 오탐 원인 분석에 유용하다.

### 2. Template 오탐 내성 시험기

실제 Target을 공격하는 대신 저장된 여러 응답에 Matcher를 실행하여 Template의 탐지 조건을 시험한다.

```text
취약 응답 Fixture
정상 응답 Fixture
Custom 404 Fixture
WAF 차단 Fixture
로그인 Redirect Fixture
Proxy 오류 Fixture
        ↓
동일한 Matcher 실행
        ↓
정상 계열을 탐지하면 오탐 위험 기록
```

예상 결과:

```text
Template: example-rce
취약 응답 탐지: 성공
정상 응답 오탐: 없음
Custom 404 오탐: 발생
WAF 차단 오탐: 발생

권장:
status matcher만 사용하지 말고
제품 고유 문자열과 header 조건을 함께 사용
```

Semgrep Community Rule이 정상·취약 코드 예제로 규칙을 시험하는 방식과 비교할 수 있다.

### 3. 검사 범위와 미실행 원인 보고서

탐지 결과가 없을 때 “검사했지만 취약하지 않음”과 “검사를 완료하지 못함”을 구별한다.

```text
0 matches
├─ 실행 완료 후 Matcher 불일치
├─ 인증 실패
├─ WAF 차단 의심
├─ Timeout
├─ DNS 실패
├─ Template 조건 불일치
├─ Scope 밖이라 제외
└─ 안전 설정 때문에 실행하지 않음
```

보고서에 기록할 상태:

- 선택된 Template
- 실제 실행된 Template
- 선택·제외 이유
- 실행된 Request 수
- 실패한 Protocol과 원인
- 도달한 URL·Endpoint
- 인증이 필요한 미검사 영역
- 최종 판정이 `취약점 없음`, `미탐지`, `미검사`, `판정 불가` 중 무엇인지

### 4. 실행 재현 묶음

다른 사람이 같은 결과를 다시 확인하는 데 필요한 정보를 하나의 Manifest로 저장한다.

```text
Reproduction Bundle
├─ Nuclei Version
├─ Template Version·Hash
├─ 실행 Options
├─ Config 영향 항목
├─ 민감정보를 제거한 Request·Response
├─ 실제 일치한 Matcher 근거
├─ 실행 시각과 환경
└─ 재현 명령
```

`ResultEvent` 한 줄보다 많은 정보를 담지만 비밀값은 반드시 제거해야 한다.

### 5. 민감정보 자동 제거 Writer

Debug·JSON·AI 설명에 포함될 수 있는 값을 저장 전에 제거한다.

```text
Authorization: Bearer eyJ...
Cookie: session=abc...
X-API-Key: secret-value
        ↓
Authorization: Bearer [REDACTED]
Cookie: session=[REDACTED]
X-API-Key: [REDACTED]
```

단순 삭제뿐 아니라 어떤 필드가 어떤 규칙으로 제거되었는지도 기록해야 보고서 공유 가능 여부를 판단할 수 있다.

### 6. 결과 원인 그래프

여러 Template 결과가 같은 원인에서 나왔는지 연결한다.

```text
TLS 1.0 허용
    ├─ deprecated-tls 탐지
    └─ weak-cipher-suites 탐지

Cloudflare WAF 탐지
    ├─ 여러 Request의 동일 차단 응답
    └─ 일부 결과의 오탐 가능성 증가
```

비밀값을 다른 Target에 전달하는 Cross-Target 기능보다 먼저, 기술·WAF·TLS·인증 상태처럼 비밀이 아닌 사실을 연결하는 편이 안전하다.

### 7. 적응형 안전 제어기

고정 Rate Limit뿐 아니라 Target의 상태를 관찰하여 요청 강도를 낮춘다.

```text
정상 응답
    ↓
5xx 비율·응답시간 급증 감지
    ↓
Concurrency와 Rate 감소
    ↓
계속 불안정하면 해당 Target 일시 중단
    ↓
안전 중단 이유 기록
```

Spider-Scents가 애플리케이션 상태 손상을 감지하는 연구와 연결되지만, 데이터베이스 접근 없이 외부 응답만으로 안전하게 동작하도록 범위를 제한한다.

## 추천 순위

### 1순위: 교육용 실행 추적 모드

- 지금 분석 중인 Runner, Loader, Template, Matcher, Output 흐름을 모두 활용한다.
- AI나 대규모 학습 데이터가 없어도 구현할 수 있다.
- 처음 배우는 사람과 Template 작성자에게 사용 목적이 분명하다.

### 2순위: Template 오탐 내성 시험기

- 오탐 문제를 Template 자체의 품질 문제로 구체화한다.
- 실제 외부 Target 없이 Fixture로 반복 실험할 수 있다.
- 정밀도·오탐률 등 정량 평가가 가능하다.

### 3순위: 검사 범위와 미실행 원인 보고서

- `0 matches`를 안전하다는 뜻으로 오해하는 문제를 줄인다.
- Runner부터 Protocol과 Output까지 전체 구조를 이해해야 구현할 수 있다.

## 새로운 연구 주제 문장

> Nuclei의 Template 선택, Request 실행 및 Matcher 판정 과정을 단계별 근거와 함께 기록하여 사용자가 탐지와 미탐지 원인을 추적할 수 있는 설명 가능한 실행 추적 기능을 제안한다.

> Nuclei Template의 Matcher를 취약·정상·Custom 404·WAF 응답 Fixture에 반복 적용하여 탐지 규칙의 오탐 내성을 자동 평가하는 Template Robustness Tester를 제안한다.

> Nuclei 스캔에서 탐지 결과뿐 아니라 Template별 실행 여부와 미실행 원인을 기록하여 `취약점 없음`과 `검사하지 못함`을 구별하는 Coverage Report를 제안한다.

# Nuclei와 AI 기능 연결

#전지성 #Nuclei #AI #Matcher #ResultEvent #OutputWriter

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


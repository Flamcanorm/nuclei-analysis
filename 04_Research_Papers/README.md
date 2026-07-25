# Nuclei 관련 선행연구 정리

#전지성 #논문분석 #Nuclei #웹취약점 #DAST

## 이 폴더의 목적

이 폴더는 Nuclei와 유사한 웹 취약점 검사 프로그램을 조사하고, 기존 연구의 문제점과 해결 방법을 비교하기 위해 만든 선행연구 정리 폴더다.

논문들은 공통적으로 다음 문제를 다룬다.

```text
기존 취약점 진단
├─ 오탐: 취약하지 않은데 취약하다고 판단
├─ 미탐: 실제 취약점을 놓침
├─ 과탐: 같거나 관련된 결과를 지나치게 많이 출력
├─ 점검 누락: 일부 페이지·기능을 검사하지 못함
├─ 검사자 편향: 검사자의 경험에 따라 결과가 달라짐
└─ 시간·비용 문제: 전체 검사에 많은 자원이 필요함
```

## 문서 구성

1. [[01_논문 목록과 읽기 순서]]
   - 찾은 논문의 정확한 제목, 중복 여부와 우선순위를 정리한다.

2. [[02_논문별 상세 분석]]
   - 각 논문의 연구 목적, 처리 흐름, 결과, 한계와 Nuclei 관련성을 분석한다.

3. [[03_공통점과 차이점]]
   - 논문별 검사 방식, 자동화 정도, AI 사용 여부, 소스코드 필요 여부를 비교한다.

4. [[04_Nuclei와 AI 기능 연결]]
   - 논문 내용을 Nuclei 구조 및 새로운 AI 기능 아이디어와 연결한다.
   - Nuclei에 이미 존재하는 기능과 새 제안의 중복 여부도 구분한다.

5. [[05_논문 전용 용어 사전]]
   - 논문, 보안 분석, AI와 성능 평가 용어를 초보자 기준으로 설명한다.

> [!info]- 논문 용어 바로가기
> [[05_논문 전용 용어 사전#^term-architecture|아키텍처]] · [[05_논문 전용 용어 사전#^term-sast|SAST]] · [[05_논문 전용 용어 사전#^term-dast|DAST]] · [[05_논문 전용 용어 사전#^term-hast|HAST]] · [[05_논문 전용 용어 사전#^term-lstm|LSTM]] · [[05_논문 전용 용어 사전#^term-false-positive-rate|오탐률]] · [[05_논문 전용 용어 사전#^term-false-negative-rate|미탐률]]
> [[05_논문 전용 용어 사전#^term-plugin|Plugin]] · [[05_논문 전용 용어 사전#^term-stateful-testing|Stateful Testing]] · [[05_논문 전용 용어 사전#^term-openapi|OpenAPI]] · [[05_논문 전용 용어 사전#^term-scan-coverage|Scan Coverage]] · [[05_논문 전용 용어 사전#^term-agent|Agent]] · [[05_논문 전용 용어 사전#^term-guardrail|Guardrail]]

## 핵심 결론

확보한 국내 논문 중에서는 `Bitscanner`와 `WAVE`가 Nuclei와 가깝다. Bitscanner는 크롤링·공격 요청·응답 비교 흐름이 유사하고, WAVE는 취약점 룰을 실행하기 전에 대상 페이지가 실제로 관련 프로그램인지 확인한다. 추가로 조사한 실제 오픈소스까지 포함하면 `Jaeles`, `Nmap NSE`, `Google Tsunami`, `OpenVAS`도 중요한 비교 대상이다.

```text
Bitscanner
크롤링 → 매개변수 추출 → 공격 요청 → 응답 패턴 비교 → 보고서

WAVE
사이트 탐색 → 페이지 콘텐츠 특성 비교 → 관련 룰 선택 → 요청·응답 분석

Nuclei
대상 공급 → Template 로드 → 요청 실행 → Matcher 판정 → Output Writer

Jaeles
Signature 로드 → HTTP 요청 실행 → 탐지 조건 판정 → 결과 출력

Nmap NSE
포트·서비스 탐색 → 실행 규칙 확인 → Lua Script 실행 → 결과 출력

Google Tsunami
서비스 탐색 Plugin → 취약점 탐지 Plugin → 검증 결과 출력
```

AI 기능을 연구할 때는 다음 논문들을 함께 참고하는 것이 좋다.

```text
AI 기반 동적 분석
└─ 머신러닝으로 규칙 기반 탐지 보완

정적·동적 크로스 체크
└─ 다른 분석 결과로 탐지 결과 재검증

홈페이지 웹취약점분석모델
└─ 자동 점검 결과를 수동 검증하여 오탐 제거

LSTM HAST
└─ SAST·DAST·LSTM을 하나의 파이프라인으로 설계

코드 생성 AI 웹 점검
└─ 자연어 점검 절차를 Selenium 코드로 바꾸어 로그인·입력·증적 생성을 자동화

RESTler
└─ OpenAPI에서 요청 사이의 선후 관계를 추론하고 상태 기반 API 검사

Black Widow
└─ JavaScript 이벤트와 페이지 사이의 상태 관계를 추적하며 크롤링

PentestGPT·CAI·Vulnhuntr
└─ LLM의 추론, 도구 실행, 코드 문맥 수집 및 안전 제어 구조 참고
```

> [!important] 새 기능을 고를 때
> Nuclei에는 이미 AI Template 생성, DAST/Fuzzing, Workflow, 공유 변수, 신규 Template만 실행, Resume, OpenAPI 입력과 결과 중복 관리 기능이 있다. 이름만 다른 기존 기능을 새 아이디어로 제안하지 않도록 [[04_Nuclei와 AI 기능 연결#Nuclei에 이미 있는 기능과 중복 검토|중복 검토]]를 먼저 확인한다.

## 이번 추가 조사의 핵심 추천

1. `교육용 실행 추적 모드`: Template이 왜 선택되고 요청과 Matcher가 어떻게 결과를 만들었는지 단계별로 설명한다.
2. `Template 오탐 내성 시험기`: 취약·정상·Custom 404·WAF 응답을 이용해 Matcher의 오탐 가능성을 시험한다.
3. `검사 범위 보고서`: 결과가 없는 이유를 “취약점 없음”과 “검사하지 못함”으로 구별한다.

세 기능의 자세한 구조는 [[04_Nuclei와 AI 기능 연결#새로운 기능 후보|새로운 기능 후보]]에서 설명한다.

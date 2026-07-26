# Nuclei와 도구별 비교

#전지성 #Nuclei/비교 #Jaeles #Nikto #WAVE #Bitscanner

> [!info]- 용어 바로가기
> [[05_비교 용어 사전#^term-engine|Engine]] · [[05_비교 용어 사전#^term-template|Template]] · [[05_비교 용어 사전#^term-signature|Signature]] · [[05_비교 용어 사전#^term-upper-compatible|상위 호환]] · [[05_비교 용어 사전#^term-maintenance|유지보수 상태]]

## 비교 결과 요약

| 비교 대상 | Nuclei와 비슷한 핵심 | Nuclei가 더 넓은 부분 | 상대 도구에서 참고할 부분 |
|---|---|---|---|
| Jaeles | Engine과 YAML 검사 규칙 분리 | Protocol, Template 생태계, 출력·연동, 현재 유지보수 | 단순한 Web Signature 선택과 사용자 정의 흐름 |
| Nikto | 외부 검사 DB, HTTP 응답 조건 판정 | 여러 Protocol과 Template 표현력 | Web Server 전용 검사 DB와 Plugin 구조 |
| WAVE | 외부 Rule 선택 후 응답 판정 | 현재 실행 가능한 범용 Engine과 생태계 | Page 특성 유사도를 이용한 사전 검증 |
| Bitscanner | 자동 공격 요청, 응답 비교, 보고서 | 다양한 Protocol과 Template 기반 정밀 검사 | Crawler로 미리 알려지지 않은 입력 지점 탐색 |

## 1. Nuclei와 Jaeles

### 공통점

```text
Nuclei
└─ Engine + Template

Jaeles
└─ Engine + Signature
```

- Go로 작성된 자동화 스캐너다.
- 검사 규칙을 YAML 계열 외부 파일로 분리한다.
- 여러 Target과 규칙을 선택하여 실행할 수 있다.
- 동시 실행을 지원한다.
- 새 규칙 파일을 추가하여 기능을 확장한다.

### 차이점

| 항목 | Nuclei | Jaeles |
|---|---|---|
| 범위 | HTTP·DNS·SSL·TCP·WHOIS·File·Headless 등 | HTTP·Web Application 중심 |
| 규칙 | Nuclei Template | Jaeles Signature |
| 판정·추출 | Matcher와 Extractor를 구분 | Signature의 HTTP 응답 조건 중심 |
| 생태계 | 큰 공식·커뮤니티 Template 저장소 | 별도 Signature 저장소 |
| 현재 상태 | 활발히 개발 중 | 공식 저장소 보관 처리, 유지보수 중단 |

### Nuclei가 상위 호환인가?

기능 범위만 보면 Nuclei가 더 넓어 `상위에 가깝다`고 말할 수 있다. 그러나 엄밀한 [[05_비교 용어 사전#^term-upper-compatible|상위 호환]]은 아니다.

- Jaeles Signature를 Nuclei가 그대로 실행하지 못한다.
- CLI 옵션과 변수 문법이 호환되지 않는다.
- Jaeles용 자동화 작업을 수정 없이 Nuclei로 옮길 수 없다.

발표에서는 다음처럼 설명하는 것이 정확하다.

> Nuclei는 Jaeles와 비슷한 규칙 분리 구조를 더 많은 Protocol로 확장한 범용 스캐너다. 기능 범위는 더 넓지만 규칙과 인터페이스가 호환되지 않으므로 완전한 상위 호환은 아니다.

## 2. Nuclei와 Nikto

### 공통점

- 미리 정의된 검사 자료를 읽는다.
- HTTP 요청을 전송한다.
- 응답의 상태 코드, Header와 Body 등을 판정한다.
- 자동화된 CLI 검사와 파일 출력을 제공한다.

### 차이점

```text
Nikto
└─ Web Server의 알려진 위험 파일·설정·버전 검사

Nuclei
└─ 여러 Protocol에서 Template이 정의한 검사 시나리오 실행
```

Nikto의 검사 DB는 Web Server 점검에 최적화되어 있다. Nuclei Template은 Protocol 요청, Matcher, Extractor, Variable과 다단계 흐름을 한 체계에서 표현한다.

### 결론

Nikto는 Nuclei의 HTTP Template 일부와 비슷한 역할을 하지만 Nuclei 전체 범위와 같지는 않다. Web Server 점검만 설명할 때는 Nikto가 좋은 비교 대상이고, 범용 Template Engine 구조를 설명할 때는 Jaeles가 더 가깝다.

## 3. Nuclei와 WAVE

### 공통점

- 검사 Engine과 외부 검사 정의를 분리한다.
- HTTP 요청을 보내고 응답으로 결과를 판정한다.
- Target에 관련된 검사만 선택하여 불필요한 요청을 줄이려 한다.

### 차이점

```text
Nuclei Automatic Scan
기술 식별 결과
  ↓
관련 Tag의 Template 선택

WAVE
개별 Page의 JavaScript·변수·링크·이미지 특성
  ↓
취약 Page와 유사한지 확인
  ↓
관련 Rule 선택
```

WAVE는 Page 단위 대상 적합성 검사를 앞에 둔다. 이는 Nuclei에 오탐 검증이나 Template 적용 대상 확인 기능을 추가할 때 참고할 만한 아이디어다.

### 결론

WAVE는 Nuclei와 경쟁하는 현재 제품이라기보다 `Template 실행 전에 이 Target이 정말 검사 대상인지 확인하는 방법`을 제공하는 선행연구다.

## 4. Nuclei와 Bitscanner

### 공통점

- Target에 공격성 시험 요청을 자동 전송한다.
- 응답의 Pattern을 분석한다.
- 반복 검사를 자동화하고 결과를 보고서로 만든다.

### 차이점

```text
Nuclei
Template이 요청 경로와 조건을 미리 정의

Bitscanner
Crawler가 Page와 Parameter를 먼저 발견
```

Nuclei에도 Headless, Fuzzing과 여러 입력 형식이 있지만, 기본 설계의 중심은 웹사이트 전체를 Crawler로 탐색해 모든 입력 지점을 생성하는 것이 아니라 선택된 Template을 Target에 실행하는 것이다.

### 결론

Bitscanner는 Nuclei에 Crawler 기반 Coverage 표시, 미검사 Parameter 발견, 자동 입력 수집 기능을 추가할 때 참고하기 좋다.

## 비교 대상 선택 방법

발표 목적에 따라 비교 대상을 다르게 선택한다.

```text
Nuclei의 구조를 설명
└─ Jaeles

HTTP 응답 판정 방식 설명
└─ Nikto

오탐·불필요한 Rule 실행 감소 아이디어
└─ WAVE

Crawler와 자동 입력 발견 아이디어
└─ Bitscanner
```

## 최종 발표 문장

> 사진 속 도구들은 모두 외부 검사 자료를 이용해 요청을 보내고 응답을 분석한다는 공통점이 있습니다. 그중 Jaeles가 Nuclei와 구조적으로 가장 가깝지만 현재는 유지보수가 중단되었습니다. Nikto는 Web Server 검사에 특화되어 있고, WAVE와 Bitscanner는 각각 Page 적합성 검증과 Crawler 기반 입력 발견이라는 연구 아이디어를 제공합니다. Nuclei는 이들과 비슷한 요소를 가지면서 더 많은 Protocol과 큰 Template 생태계를 지원하지만, 다른 도구의 규칙을 그대로 실행할 수 없으므로 완전한 상위 호환이라고 단정할 수는 없습니다.


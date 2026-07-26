# 보안 스캐너 비교 분석

#전지성 #도구비교 #Nuclei #Jaeles #Nikto #WAVE #Bitscanner

이 폴더는 사진 속 비교표를 그대로 옮기는 데서 끝나지 않고, 비교 항목이 무엇을 의미하는지와 비교가 공정한지를 설명한다.

## 문서 읽기 순서

1. [[01_사진 속 비교항목 해설]]
   - `검사 정의`, `주요 대상`, `실행 흐름` 등 사진의 8개 행이 무엇을 비교하는지 설명한다.
2. [[02_도구별 분석]]
   - Nuclei, Jaeles, Nikto, WAVE, Bitscanner가 어떤 종류의 도구인지 구분한다.
3. [[03_Nuclei와 도구별 비교]]
   - 각 도구를 Nuclei와 일대일로 비교한다.
4. [[04_표의 정확성 검토와 수정안]]
   - 사진의 표현 중 맞는 부분, 단순화된 부분, 수정할 부분을 정리한다.
5. [[05_비교 용어 사전]]
   - 처음 등장하는 용어를 설명한다.

## 가장 먼저 알아야 할 결론

```text
현재 실행 가능한 오픈소스
├─ Nuclei
├─ Nikto
└─ Jaeles: 저장소가 보관 처리되어 더 이상 유지보수되지 않음

논문에서 제안한 연구 시스템
├─ WAVE
└─ Bitscanner
```

따라서 다섯 대상을 완전히 같은 제품처럼 비교하면 안 된다.

- Nuclei·Jaeles·Nikto는 실제 소프트웨어의 구조와 기능을 비교할 수 있다.
- WAVE·Bitscanner는 논문에서 제안한 아이디어와 실험 결과를 중심으로 비교해야 한다.
- WAVE·Bitscanner에 현재 배포판, 최신 출력 형식, 유지보수 상태가 있다는 뜻으로 해석하면 안 된다.

## 전체 결론

Nuclei와 구조적으로 가장 가까운 것은 Jaeles다. 두 도구 모두 실행 Engine과 외부 검사 규칙을 분리한다.

```text
Nuclei: Engine + Template
Jaeles: Engine + Signature
```

하지만 Nuclei는 HTTP뿐 아니라 DNS, SSL, TCP, WHOIS, File, JavaScript, Code 등 더 넓은 Protocol과 큰 Template 생태계를 지원한다. 기능 범위에서는 Jaeles보다 넓지만, Signature와 Template 문법이 서로 호환되지 않으므로 `완전한 상위 호환`이라고 부르는 것은 정확하지 않다.

Nikto는 외부 검사 DB와 Plugin을 이용해 HTTP 응답을 판정한다는 점에서 비슷하지만 Web Server 검사에 집중한다. WAVE는 Rule을 실행하기 전에 Page 특성 유사도를 확인하며, Bitscanner는 Crawler가 입력 지점을 찾은 뒤 공격 Pattern을 적용한다.

## 원본 표

![[assets/Nuclei_도구_비교표.png]]


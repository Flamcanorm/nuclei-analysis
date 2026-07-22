---
상태: false
유형: Main_Function
상세: main.go 함수 목차
---
# main.go 함수 #main/function #전지성

> `main()` 밖에 정의되어 있으며 `main.go`의 실행을 돕는 함수들의 목차

## 함수 목록

| 함수 | 역할 | 상세 문서 |
|---|---|---|
| `readConfig()` | CLI 옵션 등록·파싱·설정 파일 및 Template Profile 병합 | [readConfig](main_Functions/readConfig/README.md) |
| `cleanupOldResumeFiles()` | 10일보다 오래된 Resume 파일 삭제 | [cleanupOldResumeFiles](main_Functions/cleanupOldResumeFiles.md) |
| `readFlagsConfig()` | Config 파일 확인·복사·병합 | [readFlagsConfig](main_Functions/readFlagsConfig.md) |
| `disableUpdatesCallback()` | 자동 Update 확인 기능 비활성화 | [disableUpdatesCallback](main_Functions/disableUpdatesCallback.md) |
| `printVersion()` | 버전과 주요 디렉터리 정보를 출력하고 종료 | [printVersion](main_Functions/printVersion.md) |
| `printTemplateVersion()` | 설치된 Template 버전과 사용자 Template 경로 출력 | [printTemplateVersion](main_Functions/printTemplateVersion.md) |
| `resetCallback()` | 확인을 받은 뒤 Config·Cache·Template 폴더 초기화 | [resetCallback](main_Functions/resetCallback.md) |
| `findProfilePathById()` | ID와 같은 이름의 Template Profile YAML 검색 | [findProfilePathById](main_Functions/findProfilePathById.md) |
| `processInlineSecretsFromProfile()` | Profile의 Inline Secret을 임시 파일로 변환 | [processInlineSecretsFromProfile](main_Functions/processInlineSecretsFromProfile.md) |

## readConfig 구성

`readConfig()`는 내용이 가장 길기 때문에 옵션 그룹과 후처리 단계별로 나누었다.

```text
readConfig()
├─ FlagSet 준비
├─ CLI 옵션 그룹 18개 등록
├─ flagSet.Parse() 실행
├─ Config·Template Profile 병합
├─ Inline Target·Secret 처리
└─ 오래된 Resume 파일 정리 후 FlagSet 반환
```

### 옵션 그룹

| 순서 | 그룹 | 상세 문서 |
|---|---|---|
| 1 | Target | [01_Target](main_Functions/readConfig/01_Target.md) |
| 2 | Target-Format | [02_Target-Format](main_Functions/readConfig/02_Target-Format.md) |
| 3 | Templates | [03_Templates](main_Functions/readConfig/03_Templates.md) |
| 4 | Filtering | [04_Filtering](main_Functions/readConfig/04_Filtering.md) |
| 5 | Output | [05_Output](main_Functions/readConfig/05_Output.md) |
| 6 | Configurations | [06_Configurations](main_Functions/readConfig/06_Configurations.md) |
| 7 | Interactsh | [07_Interactsh](main_Functions/readConfig/07_Interactsh.md) |
| 8 | Fuzzing | [08_Fuzzing](main_Functions/readConfig/08_Fuzzing.md) |
| 9 | Uncover | [09_Uncover](main_Functions/readConfig/09_Uncover.md) |
| 10 | Rate-Limit | [10_Rate-Limit](main_Functions/readConfig/10_Rate-Limit.md) |
| 11 | Optimizations | [11_Optimizations](main_Functions/readConfig/11_Optimizations.md) |
| 12 | Headless | [12_Headless](main_Functions/readConfig/12_Headless.md) |
| 13 | Debug | [13_Debug](main_Functions/readConfig/13_Debug.md) |
| 14 | Update | [14_Update](main_Functions/readConfig/14_Update.md) |
| 15 | Honeypot | [15_Honeypot](main_Functions/readConfig/15_Honeypot.md) |
| 16 | Statistics | [16_Statistics](main_Functions/readConfig/16_Statistics.md) |
| 17 | Cloud | [17_Cloud](main_Functions/readConfig/17_Cloud.md) |
| 18 | Authentication | [18_Authentication](main_Functions/readConfig/18_Authentication.md) |

### 옵션 파싱 이후

[19_PostProcessing](main_Functions/readConfig/19_PostProcessing.md)에는 `flagSet.Parse()` 이후 Deprecated 옵션 보정, 인증, Logger 설정, Config 및 Template Profile 병합, Inline Target·Secret, Resume 정리 과정이 들어 있다.

## 관련 문서

- [main](main.md) — `main()` 함수 실행 흐름
- [main_Varialbes](main_Varialbes.md) — `main.go` 전역·지역변수
- [pkg_types](../02_Internal_Packages/pkg_types.md) — Options 구조체
- [goflags](../03_External_Packages/goflags.md) — CLI 플래그 등록과 파싱

---
상태: false
유형: Main_Function
상세: main.go processInlineSecretsFromProfile 함수
---
# processInlineSecretsFromProfile #전지성

```go
func processInlineSecretsFromProfile(
	profilePath string,
	options *types.Options,
) (string, error)
```

> Template Profile YAML 안에 직접 작성된 `secrets` 내용을 Nuclei 인증 기능이 읽을 수 있는 임시 YAML 파일로 변환하는 함수

| 매개변수 | 의미 |
|---|---|
| `profilePath` | 읽을 Template Profile YAML 경로 |
| `options` | 생성한 Secret 파일 경로를 추가할 실행 설정 |

| 반환값 | 의미 |
|---|---|
| `string` | 생성된 임시 Secret 파일 경로, Secret이 없으면 빈 문자열 |
| `error` | 읽기·YAML 변환·폴더 생성·파일 기록 과정의 오류 |

```text
Profile YAML 읽기
    ↓
profileSecrets 구조체로 secrets 항목 추출
    ↓
secrets가 없는가?
    ├─ 예: 빈 문자열 반환
    └─ 아니오: secrets를 YAML로 변환
        ↓
임시 전용 폴더·파일 생성
        ↓
Secret YAML 기록
        ↓
options.SecretsFile에 경로 추가
        ↓
임시 파일 경로 반환
```

임시 파일은 `main()`의 `inlineSecretsTempFiles` 목록에도 기록되어 프로그램 종료 시 삭제된다. 폴더 권한 `0700`은 다른 사용자가 Secret 폴더를 읽지 못하도록 소유자에게만 접근 권한을 주기 위한 설정이다.

**참조 :** [profileSecrets](../main_Varialbes.md#profileSecrets), [readConfig PostProcessing](readConfig/19_PostProcessing.md)

---
상태: false
유형: Main_Function
상세: main.go findProfilePathById 함수
---
# findProfilePathById #전지성

```go
func findProfilePathById(
	profileId string,
	templatesDir string,
) string
```

> Template Profile ID와 같은 파일 이름을 가진 YAML 파일을 Template 폴더에서 검색하는 함수

| 매개변수 | 의미 |
|---|---|
| `profileId` | 찾을 Profile ID 또는 파일 이름 |
| `templatesDir` | 검색을 시작할 Template 기준 폴더 |

- **반환값 :** 찾은 YAML 파일 경로, 찾지 못하면 빈 문자열

```text
templatesDir 내부 순회
    ↓
폴더와 YAML이 아닌 파일 제외
    ↓
확장자를 뺀 파일 이름과 profileId 비교
    ├─ 다름: 계속 검색
    └─ 같음: 해당 경로 저장 후 검색 중단
```

`FOUND` 오류 문자열은 실제 실패가 아니라 `filepath.WalkDir()`의 순회를 조기에 멈추기 위한 신호로 사용된다.

**참조 :** [readConfig PostProcessing](readConfig/19_PostProcessing.md)

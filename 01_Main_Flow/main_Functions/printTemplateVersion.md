---
상태: false
유형: Main_Function
상세: main.go printTemplateVersion 함수
---
# printTemplateVersion #전지성

```go
func printTemplateVersion()
```

> 설치된 공식 nuclei-templates 버전과 Custom Template 저장 위치를 출력하고 프로그램을 종료하는 함수

- **매개변수 :** 없음
- **반환 타입 :** 없음
- **실행 옵션 :** `-templates-version`, `-tv`

```text
DefaultConfig 읽기
    ↓
공식 Template 버전·폴더 출력
    ↓
S3·GitHub·GitLab·Azure Custom Template 폴더가 있으면 출력
    ↓
os.Exit(0)으로 정상 종료
```

`printVersion()`은 Nuclei 실행 파일의 버전과 Config·Cache 경로를 보여주고, `printTemplateVersion()`은 검사 규칙인 Template의 버전과 위치를 보여준다는 차이가 있다.

**참조 :** [printVersion](printVersion.md), [readConfig Update](readConfig/14_Update.md)

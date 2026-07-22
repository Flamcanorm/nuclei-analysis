---
상태: false
유형: Main_Function
상세: main.go resetCallback 함수
---
# resetCallback #전지성

```go
func resetCallback()
```

> 사용자의 확인을 받은 뒤 Nuclei Config, Cache, Template 폴더를 모두 삭제하여 초기 상태로 되돌리는 함수

- **매개변수 :** 없음
- **반환 타입 :** 없음
- **실행 옵션 :** `-reset`

```text
삭제 대상과 주의사항 출력
    ↓
y 또는 yes 입력 대기
    ├─ n·no·빈 입력: 삭제하지 않고 종료
    └─ y·yes: 삭제 진행
        ↓
Config 폴더 삭제
        ↓
Cache 폴더 삭제
        ↓
Template 폴더 삭제
        ↓
성공 메시지 출력 후 종료
```

이 함수는 복구하기 어려운 삭제 작업을 수행하므로 바로 삭제하지 않고 사용자에게 확인을 요구한다. Custom Template을 해당 폴더에 저장했다면 실행 전에 별도 백업이 필요하다.

**참조 :** [readConfig Configurations](readConfig/06_Configurations.md)

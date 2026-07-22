---
상태: false
유형: Main_Function
상세: main.go disableUpdatesCallback 함수
---
# disableUpdatesCallback #전지성

## disableUpdatesCallback #전지성
```go
func disableUpdatesCallback()
```
> Nuclei의 자동 Update 확인 기능을 비활성화하는 Callback 함수
- **매개변수 :** 없음
- **반환 타입 :** 없음
- **설명 :** `config.DefaultConfig.DisableUpdateCheck()`를 호출
- **참조 :** [readConfig](readConfig/README.md)

### 상세 분석

```
### 20. 자동 업데이트 확인 비활성화(768~771줄)

// disableUpdatesCallback disables the update check.
// disableUpdatesCallback은 자동 업데이트 확인 기능을 비활성화한다.
func disableUpdatesCallback() {

	config.DefaultConfig.DisableUpdateCheck()

} // [disableUpdatesCallback 함수 종료]
```

#### 해석
`-disable-update-check` 옵션이 입력되면 실행되는 콜백 함수이다.

내부적으로 `config.DefaultConfig.DisableUpdateCheck()`를 호출하여  
자동 업데이트 확인 기능을 끈다.


---


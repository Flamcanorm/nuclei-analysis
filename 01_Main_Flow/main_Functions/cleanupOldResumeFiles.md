---
상태: false
유형: Main_Function
상세: main.go cleanupOldResumeFiles 함수
---
# cleanupOldResumeFiles #전지성

## cleanupOldResumeFiles #전지성
```go
func cleanupOldResumeFiles()
```
> Cache 디렉터리에서 10일보다 오래된 Resume 파일을 정리하는 함수
- **매개변수 :** 없음
- **반환 타입 :** 없음
- **설명 :** `resume-` 접두사를 가진 파일 중 생성 후 10일이 지난 파일을 삭제
- **참조 :** [readConfig](readConfig/README.md)

### 상세 분석

```
### 16. 오래된 Resume 파일 삭제(729줄~737줄)

// cleanupOldResumeFiles cleans up resume files older than 10 days.
// cleanupOldResumeFiles는 10일보다 오래된 resume 파일을 정리한다.
func cleanupOldResumeFiles() {

	root := config.DefaultConfig.GetCacheDir()

	filter := fileutil.FileFilters{
		OlderThan: 24 * time.Hour * 10, // cleanup on the 10th day
		Prefix:    "resume-",
	}

	_ = fileutil.DeleteFilesOlderThan(root, filter)

} // [cleanupOldResumeFiles 함수 종료]
```

#### 해석
Nuclei의 cache 디렉터리에서 `resume-`으로 시작하고,  
10일보다 오래된 파일을 삭제한다.

`resume` 파일은 중단된 스캔을 이어서 실행할 때 사용되는 파일이다.

---


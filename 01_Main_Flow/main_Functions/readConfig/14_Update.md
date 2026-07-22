---
상태: false
유형: Main_Function
상세: Update 옵션 그룹
---
# readConfig - Update #전지성

```
	flagSet.CreateGroup("update", "Update",
		flagSet.BoolVarP(&updateNucleiBinary, "update", "up", false, "update nuclei engine to the latest released version"),
		flagSet.BoolVarP(&options.UpdateTemplates, "update-templates", "ut", false, "update nuclei-templates to latest released version"),
		flagSet.StringVarP(&options.NewTemplatesDirectory, "update-template-dir", "ud", "", "custom directory to install / update nuclei-templates"),
		flagSet.CallbackVarP(disableUpdatesCallback, "disable-update-check", "duc", "disable automatic nuclei/templates update check"),
	)
```

#### FlagSet.CreateGroup 해석

[흐름]  
update라는 내부에서 그룹을 식별하기 위한 이름, 사용자에게는 Update 그룹을 생성하고, Nuclei 엔진 및 nuclei-templates 업데이트와 관련된 CLI 옵션들을 하나의 그룹으로 등록.  
  
flagSet.BoolVarP(...)에서 -update 옵션을 등록하고, FlagData 반환.  
flagSet.BoolVarP(...)에서 -update-templates 옵션을 등록하고, FlagData 반환.  
  
아래 옵션들도 모두 등록됨.  
-update  
-update-templates  
-update-template-dir  
-disable-update-check  
  
CreateGroup 호출 후, 모두 update 그룹으로 지정.  
  
[의미]  
이해를 돕고자 예를 들어 설명했습니다.  
  
1. Nuclei 엔진을 최신 릴리스 버전으로 업데이트한다.  
( ex. nuclei -up -> updateNucleiBinary = true )  
  
2. nuclei-templates를 최신 릴리스 버전으로 업데이트한다.  
( ex. nuclei -ut -> options.UpdateTemplates = true )  
  
3. nuclei-templates를 설치하거나 업데이트할 디렉터리를 지정한다.  
( ex. nuclei -ud /home/user/nuclei-templates -> options.NewTemplatesDirectory )  
  
4. 자동 업데이트 확인(Update Check)을 비활성화한다.  
( ex. nuclei -duc -> disableUpdatesCallback() 실행 )  
  
[CLI 옵션 등록]  
-up  
-ut  
-ud  
-duc  
  
[입력된 값을 변수 또는 options 구조체와 연결(바인딩) 한다.]  
-up → updateNucleiBinary  
-ut → options.UpdateTemplates  
-ud → options.NewTemplatesDirectory  
-duc → disableUpdatesCallback()



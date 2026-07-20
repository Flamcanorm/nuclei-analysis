---
유형: Internal_Pack
상태: false
상세: pkg/catalog/loader/loader.go
---
# catalog/loader.go #pkg/catalog/loader

> Options와 Catalog를 이용하여 실제 실행할 템플릿을 선택하고 Template 객체로 불러오는 코드

## Catalog와 Loader의 차이

사진에서는 Catalog 내부에 Loader가 포함된 것처럼 표시되어 있지만 실제 코드에서는 역할이 분리되어 있다.

```text
Catalog
└─ 템플릿 파일 경로를 찾고 파일을 엶
        ↓
Loader
└─ 조건에 맞는 템플릿을 선택하고 불러옴
```

## Config struct

Loader Config에는 템플릿 선택에 필요한 설정이 들어간다.

```go
type Config struct {
	Templates  []string
	Tags       []string
	Authors    []string
	Severities severity.Severities
	Catalog    catalog.Catalog
}
```

실제 구조체에는 제외 경로, 포함 태그, 프로토콜, Workflow 등 더 많은 필드가 있다.

Runner는 Options의 값을 Loader Config로 옮긴다.

```go
loaderConfig := loader.NewConfig(
	r.options,
	r.catalog,
	executorOpts,
)
```

## Store struct

Store는 경로 검색과 필터링이 끝난 템플릿을 보관한다.

```text
Store
├─ finalTemplates: 선택된 템플릿 경로
├─ finalWorkflows: 선택된 Workflow 경로
├─ templates: 해석·준비된 Template 객체
└─ workflows: 해석·준비된 Workflow 객체
```

## 주요 함수

### Load

```go
func (store *Store) Load() error
```

> Options의 태그·심각도·작성자·경로 등의 조건을 적용하고 실제 실행할 템플릿을 불러온다.

### Templates

```go
func (store *Store) Templates() []*templates.Template
```

> Store가 불러온 일반 Template 객체 목록을 반환한다.

### Workflows

```go
func (store *Store) Workflows() []*templates.Template
```

> Store가 불러온 Workflow 객체 목록을 반환한다.

사진의 `Catalog.GetTemplates()`는 실제 함수가 아니다. 실제 흐름에서는 Catalog가 경로를 찾고, Loader Store가 `Load()`로 불러온 뒤 `Templates()`로 목록을 제공한다.

**참조 :** [pkg_types](pkg_types.md), [catalog_disk](catalog_disk.md), [runner_runner](runner_runner.md), [core_engine](core_engine.md)

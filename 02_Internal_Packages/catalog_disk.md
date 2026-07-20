---
유형: Internal_Pack
상태: false
상세: pkg/catalog/disk
---
# catalog/disk #pkg/catalog/disk

> 디스크에서 Nuclei 템플릿 경로를 찾고 템플릿 파일을 여는 코드

## DiskCatalog struct

```go
type DiskCatalog struct {
	templatesDirectory string
	templatesFS        fs.FS
}
```

| 필드 | 역할 |
|---|---|
| `templatesDirectory` | 템플릿을 검색할 기준 폴더 |
| `templatesFS` | 별도로 제공된 파일 시스템에서 템플릿을 읽을 때 사용 |

Runner는 다음 코드로 DiskCatalog를 만든다.

```go
runner.catalog = disk.NewCatalog(
	config.DefaultConfig.TemplatesDirectory,
)
```

## 주요 함수

### NewCatalog

```go
func NewCatalog(directory string) *DiskCatalog
```

> 템플릿 기준 폴더를 저장한 DiskCatalog 객체를 생성한다. 경로가 비어 있으면 Nuclei의 기본 템플릿 폴더를 사용한다.

### GetTemplatesPath

```go
func (c *DiskCatalog) GetTemplatesPath(
	definitions []string,
) ([]string, map[string]error)
```

> 사용자가 지정한 여러 파일·폴더·URL을 실제 템플릿 파일 경로 목록으로 변환한다.

### GetTemplatePath

```go
func (c *DiskCatalog) GetTemplatePath(
	target string,
) ([]string, error)
```

```text
입력 경로
├─ 파일: 해당 템플릿 파일 반환
├─ 폴더: 내부 템플릿 재귀 검색
└─ *.yaml: 와일드카드에 맞는 파일 검색
```

### OpenFile

```go
func (d *DiskCatalog) OpenFile(
	filename string,
) (io.ReadCloser, error)
```

> 찾은 템플릿 파일을 열어 Loader와 Parser가 내용을 읽을 수 있게 한다.

## Catalog가 하지 않는 일

Catalog는 태그·심각도를 필터링하거나 HTTP 요청을 실행하지 않는다.

```text
Catalog
└─ 템플릿 위치 검색·파일 열기

Loader
└─ 템플릿 필터링·해석·로드

Engine
└─ 로드된 템플릿 실행
```

**참조 :** [runner_runner](runner_runner.md), [catalog_loader](catalog_loader.md), [core_engine](core_engine.md)

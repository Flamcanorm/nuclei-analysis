---
유형: 실행 결과
상태: false
명령: ./nuclei -sign
상세: 템플릿 서명용 사용자 인증서와 개인키의 최초 생성 결과
---

# `-sign` 실행 결과 #전지성

<!-- glossary-links:start -->
> [!info]- 명칭 참조
> 이 문서에서 사용하는 공통 명칭은 아래 링크를 눌러 명칭 사전에서 확인할 수 있다.
>
> [정의](<../03_External_Packages/04_etc/sample.md#정의>) · [변수](<../03_External_Packages/04_etc/sample.md#변수>) · [nil](<../03_External_Packages/04_etc/sample.md#nil>) · [Branch 또는 분기](<../03_External_Packages/04_etc/sample.md#Branch 또는 분기>) · [Config](<../03_External_Packages/04_etc/sample.md#Config>) · [환경변수](<../03_External_Packages/04_etc/sample.md#환경변수>) · [Catalog](<../03_External_Packages/04_etc/sample.md#Catalog>) · [INF](<../03_External_Packages/04_etc/sample.md#INF>)
> [개인키](<../03_External_Packages/04_etc/sample.md#개인키>) · [공개키](<../03_External_Packages/04_etc/sample.md#공개키>) · [키 쌍](<../03_External_Packages/04_etc/sample.md#키 쌍>) · [인증서](<../03_External_Packages/04_etc/sample.md#인증서>) · [자체 서명 인증서](<../03_External_Packages/04_etc/sample.md#자체 서명 인증서>) · [Passphrase 또는 암호 구문](<../03_External_Packages/04_etc/sample.md#Passphrase 또는 암호 구문>) · [ECDSA](<../03_External_Packages/04_etc/sample.md#ECDSA>) · [P-256](<../03_External_Packages/04_etc/sample.md#P-256>)
> [PEM](<../03_External_Packages/04_etc/sample.md#PEM>)
<!-- glossary-links:end -->


## 입력

```bash
./nuclei -sign
```

## 출력

```text
[INF] Generating new key-pair for signing templates
[*] Enter User/Organization Name (exit to abort) : jiseong(내가 입력함.)
[*] Enter passphrase (exit to abort):
[*] Enter same passphrase again:
[INF] Successfully generated new key-pair for signing templates
```

## 먼저 알아야 할 결론

이번 출력은 **템플릿 서명이 끝났다는 뜻이 아니다.** 기존 사용자 인증서 또는 개인키를 찾지 못해 앞으로 서명에 사용할 새로운 키 쌍을 생성하고 저장한 결과다.

`NewTemplateSigner(nil, nil)`은 키 생성과 저장을 마친 뒤 `os.Exit(0)`을 호출한다. 따라서 다음 코드인 템플릿 순회와 `SignTemplate()`까지 진행하지 않는다.

```text
./nuclei -sign
  ↓
기존 인증서·개인키 탐색
  ↓ 찾지 못함
새 키 쌍과 인증서 생성
  ↓
설정 디렉터리의 keys 폴더에 저장
  ↓
os.Exit(0)
  ↓
이번 실행에서는 템플릿 서명 안 함
```

템플릿을 실제로 서명하려면 키 생성이 끝난 뒤 서명할 템플릿 경로를 지정해 명령을 다시 실행해야 한다.

```bash
./nuclei -t path/to/custom-template.yaml -sign
```

## 출력 문장별 의미

### `[INF] Generating new key-pair for signing templates`

환경변수와 Nuclei 설정의 `keys` 디렉터리에서 사용할 수 있는 사용자 인증서·개인키를 찾지 못해 새 키 쌍 생성을 시작했다는 뜻이다.

### `Enter User/Organization Name`

사용자 또는 조직을 구별할 식별자를 입력하는 단계다. 입력한 `jiseong`은 자체 서명 인증서의 `Subject.CommonName`에 저장된다.

이 이름은 템플릿의 실제 안전성을 보증하지 않는다. 인증서에서 서명자 이름을 표시하고 구분하기 위한 값이다.

### `Enter passphrase`

저장할 개인키를 암호화할 암호 구문을 입력하는 단계다. 입력 내용은 `term.ReadPassword()`가 받기 때문에 터미널 화면에 표시되지 않는다.

출력에서 입력값이 보이지 않는다고 해서 빈 암호를 입력했다고 단정할 수 없다. 실제로 사용자가 입력했더라도 화면에는 나타나지 않는다.

### `Enter same passphrase again`

처음 입력한 암호 구문과 같은 값을 다시 입력했는지 확인한다. 두 값이 다르면 `passphrase did not match try again` 오류로 종료한다.

### `[INF] Successfully generated new key-pair`

ECDSA P-256 개인키와 공개키가 생성되었고, 공개키와 `jiseong` 식별자를 포함하는 X.509 자체 서명 인증서가 만들어졌다는 뜻이다.

## 생성되는 파일

`SaveToDisk(config.DefaultConfig.GetKeysDir())`가 Nuclei 전역 설정 디렉터리 아래의 `keys` 폴더에 다음 파일을 저장한다.

| 파일 | 내용 | 보안상 취급 |
|---|---|---|
| `nuclei-user.crt` | 사용자 이름과 공개키를 가진 인증서 | 서명 검증에 사용하며 공개 가능 |
| `nuclei-user-private-key.pem` | 템플릿 서명에 사용하는 개인키 | 외부에 공개하면 안 됨 |

두 파일은 코드에서 `0600` 권한으로 기록된다. 이는 지원되는 운영체제에서 현재 사용자만 읽고 쓸 수 있도록 제한하려는 설정이다.

암호 구문을 입력했다면 개인키의 PEM 블록을 AES-256 방식으로 암호화한 뒤 저장한다.

## 사용한 암호 기술

```text
ECDSA P-256 키 쌍 생성
  ↓
SHA-256 기반 ECDSA 자체 서명 인증서 생성
  ↓
개인키를 PEM 형식으로 변환
  ↓ 암호 구문이 있으면
AES-256으로 개인키 PEM 암호화
  ↓
keys 폴더에 인증서와 개인키 저장
```

인증서의 유효기간은 생성 시각부터 코드상 약 4년으로 설정된다.

## 실제 템플릿 서명 시 나타나는 차이

키가 이미 존재하고 템플릿 경로를 지정한 다음 실행에서는 `main.go`가 `.yaml` 파일을 순회하면서 `templates.SignTemplate()`을 호출한다. 마지막에는 다음 형식의 집계 메시지가 나타난다.

```text
[INF] All templates signatures were elaborated success=<성공 수> failed=<실패 수>
```

제시된 출력에는 이 메시지가 없다. 따라서 이 결과만으로는 서명된 템플릿이 있다고 판단하면 안 된다.

## 코드 연결

```text
readConfig()
  ↓ -sign을 options.SignTemplates=true로 저장
main.go의 if options.SignTemplates
  ↓
signer.NewTemplateSigner(nil, nil)
  ↓
ReadCert()와 ReadPrivateKey()
  ↓ 키가 없으면
GenerateKeyPair()
  ↓
SaveToDisk()
  ↓
os.Exit(0)
```

## 참고할 코드 파일

- `cmd/nuclei/main.go`: `-sign` 분기와 템플릿 순회
- `pkg/templates/signer/tmpl_signer.go`: 기존 키 탐색, 새 키 생성 후 종료
- `pkg/templates/signer/handler.go`: 키 쌍, 인증서, 암호 구문, 파일 저장 처리
- `pkg/catalog/config/nucleiconfig.go`: `keys` 디렉터리 경로 생성

## 연결 문서

- 실행 위치: [main.md L80](<../01_Main_Flow/main.md>)
- 상세 코드 설명: [main.go 분기별 상세 설명](<../설명서/13_main.go 분기별 상세 설명.md#L80 — `-sign` 템플릿 서명 #전지성>)
- 모든 출력: [출력결과 폴더](<README.md>)

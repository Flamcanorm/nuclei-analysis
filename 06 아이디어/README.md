# Raspberry Pi Python Scanner 분석 노트

기존 Nuclei 구조를 참고해 Raspberry Pi OS에서 Python과 FastAPI로 새로운 스캐너를 설계하기 위한 조사 기록이다.

현재 단계에서는 구현을 시작하지 않고, Nuclei의 구조와 Python 대응 기술, Raspberry Pi 환경에서의 비동기 처리 및 저장 정책을 정리한다.

## 문서 목록

1. [[01 Nuclei 구조와 Python 분석]]
2. [[02 비동기 스캔과 로깅 저장 정책]]

## 현재 결론

- FastAPI는 스캔 엔진이 아니라 요청 접수, 상태 조회, 취소 명령을 담당한다.
- 실제 스캔은 별도의 `ScanRunner`가 수행한다.
- 네트워크 I/O는 `asyncio` 기반 비동기로 처리한다.
- 처음에는 HTTP/HTTPS 템플릿 스캐너만 구현한다.
- 운영 로그는 별도 파일보다 `systemd-journald`를 사용한다.
- 스캔 결과는 SQLite에 구조화해서 저장한다.
- 전체 요청·응답은 저장하지 않고, 탐지된 결과의 제한된 증거만 저장한다.
- 대상 범위 제한, 인증, timeout, 응답 크기 제한을 필수 보안 경계로 둔다.

## 조사 기준

- 로컬 Nuclei 소스: `C:\Users\anima\Documents\Obsidian Vault\nuclei-dev`
- 조사일: 2026-09-11
- 로컬 폴더에 독립적인 Git 메타데이터가 없으므로 정확한 공식 태그나 커밋은 식별하지 못했다.
- 분석은 위 폴더에 저장된 소스 스냅샷을 기준으로 한다.


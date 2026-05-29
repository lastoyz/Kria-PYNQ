# 코드 분석 문서 폴더

이 폴더는 `Kria-PYNQ` 저장소 코드를 분석한 Markdown 문서를 모아두는 공간입니다.

## 기본 구조

- `INDEX.md`: 전체 문서 인덱스
- `project-purpose.md`: 프로젝트 목적/범위 요약
- `guides/`: 보드 bring-up, PYNQ 설치, selftest/노트북 테스트, 트러블슈팅 가이드
- `structure/`: `install.sh`, `kv260/`, `dts/`, `setup.py` 구조 분석 문서

## 분류 기준

- `project`: 프로젝트 목적/배경 정리
- `guides`: 실행 절차/환경 구축/운영 트러블슈팅
- `structure`: 저장소 구조/설치 스크립트 연결 관계 분석

## 작성 규칙 (간단 버전)

- 파일명은 소문자-kebab-case 사용 (예: `board-setup-and-test-guide.md`)
- 문서 첫 부분에 목적/범위/관련 파일 경로를 짧게 기록
- 중복 내용은 `INDEX.md`에서 링크로 연결

## 분석 기준 저장소

- 경로: `/Users/mac/tmp/test_github/local_kria_pynq`
- 브랜치: `review_0529`
- submodule `pynq/`: 링크만 확보 (소스 미다운로드, 커밋 `a056b84`)

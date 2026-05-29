# guides

보드 bring-up, PYNQ 설치, selftest/노트북 검증 등 "실행 절차형 문서"를 저장한다.

## 문서 역할 분리

- `board-setup-and-test-guide.md`: 보드 셋업 + 테스트 통합 실행 가이드 (SSOT)
- `board-bringup-quickstart.md`: Ubuntu SD 이미지 부팅/접속 확인 전용
- `sw-setup-deep-dive.md`: Jupyter/노트북/selftest 기능 테스트 전용
- `common-troubleshooting.md`: 공통 장애 대응 전용
- `board-variants-notes.md`: KV260/KR260/KD240 차이/적용 주의사항 전용
- `kv260-board-test-checklist.md`: KV260 실물 점검 실행 체크리스트 (MIPI 구성)
- `kv260-mipi-camera-test-guide.md`: MIPI 카메라(Pcam 5C) 테스트·selftest 대체 전략

## RFSoC-PYNQ guides와의 차이

Kria-PYNQ에는 다음 문서가 **해당 없음**:

- `fpga-board-vivado-setup.md` — KV260 base 재빌드는 `structure/kv260-directory-notes.md` 참조
- `sd-image-build.md` — 공식 Ubuntu 이미지 사용, sdbuild 미사용
- `fw-setup-deep-dive.md` — pip/설치 스크립트가 overlay 배포를 담당

## 권장 진행 순서

1. `board-setup-and-test-guide.md` (통합 절차 요약)
2. `board-bringup-quickstart.md`
3. `sw-setup-deep-dive.md`
4. 문제 발생 시 `common-troubleshooting.md`

## 빠른 의사결정 트리

- 보드가 Ubuntu 22.04로 정상 부팅되는가?
  - 아니오: 공식 이미지/부트 펌웨어 확인 (`board-bringup-quickstart.md`)
  - 예: `install.sh` 진행
- PYNQ 설치 후 overlay/노트북만 검증하면 되는가?
  - 예: `sw-setup-deep-dive.md` + `selftest.sh`
  - 아니오: KV260 base overlay 재빌드 필요 여부 확인 (`structure/kv260-directory-notes.md`)

## Go/No-Go 게이트

- [ ] Ubuntu 22.04 정상 부팅
- [ ] `install.sh` 설치 완료
- [ ] JupyterLab 접속 성공 (`:9090/lab`)
- [ ] overlay 로딩 성공
- [ ] selftest 또는 핵심 노트북 테스트 통과
- [ ] 결과/이슈 기록 완료

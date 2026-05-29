# KV260 Board Test Checklist

## 문서 목적

KV260 Vision AI Starter Kit에 Kria-PYNQ를 설치하고 기능을 점검하기 위한 실행 체크리스트를 제공한다.

## 범위

- 포함: KV260 bring-up → install.sh → selftest → 노트북 pass/fail 기록
- 제외: KR260/KD240, Vivado overlay 재빌드 (→ `board-setup-and-test-guide.md` §5)

## 선행 문서

- 통합 절차: `board-setup-and-test-guide.md`
- 상세 테스트: `sw-setup-deep-dive.md`
- 보드 차이: `board-variants-notes.md`

## 필수 주변장치

selftest `test_apps.py` 실행 시 필요:

- HDMI 또는 DisplayPort 모니터
- USB 웹캠

## Phase 1 — Bring-up

- [ ] Canonical Ubuntu 22.04 SD 이미지 플래시
- [ ] 부트 펌웨어 2022.1+ 확인
- [ ] Ethernet/UART 연결, IP 확인
- [ ] `cat /etc/lsb-release` → 22.04 확인
- [ ] SSH 또는 콘솔 접속

## Phase 2 — PYNQ 설치

```bash
git clone https://github.com/lastoyz/Kria-PYNQ.git
cd Kria-PYNQ
git checkout review_0529
sudo bash install.sh -b KV260
```

- [ ] install.sh 약 25분 완료
- [ ] Jupyter URL 출력 확인 (`:9090/lab`)

## Phase 3 — 스모크 테스트

```bash
source /etc/profile.d/pynq_venv.sh
python3 -c "from kv260 import BaseOverlay; ol=BaseOverlay('base.bit'); print(ol.is_loaded())"
```

- [ ] JupyterLab 접속 (비밀번호 `xilinx`)
- [ ] overlay 로딩 True

## Phase 4 — Selftest

```bash
sudo ./selftest.sh
```

- [ ] `test_apps.py`
- [ ] `test_composable.py`
- [ ] `test_mmio_partial_bitstreams.py`
- [ ] `pynq_dpu/tests`

## Phase 5 — 노트북 (우선순위)

**video/**

- [ ] `display_port_introduction.ipynb`
- [ ] `opencv_filters_webcam.ipynb`
- [ ] `mipi_to_displayport.ipynb` (Pcam 5C 있을 때)

**microblaze/**

- [ ] `microblaze_programming.ipynb`

**pip 패키지**

- [ ] `pynq_helloworld` 예제
- [ ] `pynq-dpu` 예제 1개

## Phase 6 — 결과 기록

| 항목 | 결과 | 날짜/로그 |
|------|------|-----------|
| install.sh | | |
| overlay 로딩 | | |
| selftest | | |
| 노트북 | | |

## Go/No-Go

- [ ] Ubuntu 22.04 + install.sh OK
- [ ] Jupyter + overlay OK
- [ ] selftest pass (또는 실패 이슈 등록)
- [ ] 핵심 노트북 1개 이상 pass

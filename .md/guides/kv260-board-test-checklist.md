# KV260 Board Test Checklist

## 문서 목적

KV260 Vision AI Starter Kit에 Kria-PYNQ를 설치하고 기능을 점검하기 위한 실행 체크리스트를 제공한다.

## 테스트 구성 (본 프로젝트)

> **MIPI 카메라 모듈 장착** — USB 웹캠 없이 MIPI 경로로 검증한다.
> 상세: `kv260-mipi-camera-test-guide.md`

## 범위

- 포함: KV260 bring-up → install.sh → MIPI 검증 → selftest(변형) → 노트북 pass/fail 기록
- 제외: KR260/KD240, Vivado overlay 재빌드 (→ `board-setup-and-test-guide.md` §5)

## 선행 문서

- 통합 절차: `board-setup-and-test-guide.md`
- **MIPI 테스트**: `kv260-mipi-camera-test-guide.md`
- 상세 테스트: `sw-setup-deep-dive.md`
- 보드 차이: `board-variants-notes.md`

## 필수 주변장치 (MIPI 구성)

- MIPI 카메라 모듈 (Pcam 5C 등) → KV260 **Raspberry Pi camera** FFC 포트 **장착됨**
- HDMI 또는 DisplayPort 모니터
- USB 웹캠 **불필요**

## Phase 1 — Bring-up

- [ ] Canonical Ubuntu 22.04 SD 이미지 플래시
- [ ] 부트 펌웨어 2022.1+ 확인
- [ ] Ethernet/UART 연결, IP 확인
- [ ] MIPI FFC 체결·방향 확인
- [ ] DP/HDMI 모니터 연결
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

## Phase 3 — Overlay + MIPI 스모크 테스트

```bash
source /etc/profile.d/pynq_venv.sh
python3 -c "from kv260 import BaseOverlay; ol=BaseOverlay('base.bit'); print(ol.is_loaded())"
```

MIPI frame 캡처:

```python
from kv260 import BaseOverlay
from pynq.lib.video import VideoMode

base = BaseOverlay("base.bit")
mipi = base.mipi
mipi.configure(VideoMode(1280, 720, 24))
mipi.start()
frame = mipi.readframe()
print(frame.shape)
mipi.stop()
base.free()
```

- [ ] JupyterLab 접속 (비밀번호 `xilinx`)
- [ ] overlay 로딩 True
- [ ] MIPI `readframe()` 성공 (shape 출력)

## Phase 4 — Selftest (MIPI 변형)

`test_apps.py`는 OpenCV/USB 입력 전용 → **MIPI 환경에서는 제외**.

```bash
cd /usr/local/share/pynq-venv/lib/python3.10/site-packages/pynq_composable/runtime_tests
sudo python3 -m pytest test_composable.py test_mmio_partial_bitstreams.py
sudo python3 -m pytest /usr/local/share/pynq-venv/lib/python3.10/site-packages/pynq_dpu/tests
```

- [ ] `test_composable.py`
- [ ] `test_mmio_partial_bitstreams.py`
- [ ] `pynq_dpu/tests`
- [ ] `test_apps.py` → skip (MIPI-only, 이유 기록)

## Phase 5 — 노트북 (MIPI 우선)

**video/ — MIPI**

- [ ] **`mipi_to_displayport.ipynb`** (핵심 — MIPI→DP 실시간 출력)
- [ ] `display_port_introduction.ipynb`

**video/ — USB 전용 (skip)**

- [ ] ~~`opencv_filters_webcam.ipynb`~~ (USB 웹캠 필요)
- [ ] ~~`opencv_face_detect_webcam.ipynb`~~ (USB 웹캠 필요)

**microblaze/**

- [ ] `microblaze_programming.ipynb` (선택)

**pip 패키지**

- [ ] `pynq_helloworld` 예제
- [ ] `pynq-dpu` 예제 1개

## Phase 6 — 결과 기록

| 항목 | 결과 | 날짜/로그 |
|------|------|-----------|
| install.sh | | |
| overlay 로딩 | | |
| MIPI readframe | | |
| selftest (MIPI 변형) | | |
| mipi_to_displayport | | |

## Go/No-Go

- [ ] Ubuntu 22.04 + install.sh OK
- [ ] Jupyter + overlay OK
- [ ] **MIPI frame 캡처 OK**
- [ ] **mipi_to_displayport.ipynb pass**
- [ ] selftest (composable + DPU) pass 또는 이슈 등록
- [ ] `test_apps` skip 사유 기록 (MIPI-only)

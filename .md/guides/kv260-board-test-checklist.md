# KV260 Board Test Checklist

## 문서 목적

KV260 Vision AI Starter Kit에 Kria-PYNQ를 설치하고 기능을 점검하기 위한 실행 체크리스트를 제공한다.

## 테스트 구성 (본 프로젝트)

> **카메라:** onsemi **AR1335** IAS (13MP AF RGB) · **J7** 장착  
> **USB 웹캠 미사용** · 상세: `kv260-mipi-camera-test-guide.md`

## 범위

- 포함: bring-up → install.sh → **Smartcam AR1335 MIPI** → PYNQ selftest → 결과 기록
- 제외: KR260/KD240, Vivado overlay 재빌드

## 선행 문서

- `board-setup-and-test-guide.md`
- **`kv260-mipi-camera-test-guide.md`** (AR1335 필수)
- `sw-setup-deep-dive.md`

## 필수 주변장치

- **AR1335 IAS 모듈** → KV260 **J7** (장착됨)
- HDMI 또는 DisplayPort 모니터
- USB 웹캠 **불필요**

---

## Phase 1 — Bring-up

- [ ] Ubuntu 22.04 SD 이미지
- [ ] 부트 펌웨어 2022.1+
- [ ] Ethernet/UART, IP 확인
- [ ] **AR1335 → J7 IAS** FFC 체결 확인
- [ ] DP/HDMI 모니터 연결
- [ ] SSH/콘솔 접속

## Phase 2 — PYNQ 설치

```bash
git clone https://github.com/lastoyz/Kria-PYNQ.git
cd Kria-PYNQ
git checkout review_0529
sudo bash install.sh -b KV260
```

- [ ] install.sh 완료
- [ ] Jupyter `:9090/lab` 접속

## Phase 3 — PYNQ overlay 스모크

```bash
source /etc/profile.d/pynq_venv.sh
python3 -c "from kv260 import BaseOverlay; ol=BaseOverlay('base.bit'); print(ol.is_loaded())"
```

- [ ] overlay 로딩 True

> AR1335 MIPI 영상은 Phase 4에서 Smartcam으로 검증. `base.mipi`는 Pcam/RPi 경로용.

## Phase 4 — AR1335 MIPI 검증 (Smartcam)

```bash
sudo apt install xlnx-firmware-kv260-smartcam   # PPA: xilinx-apps
sudo xmutil unloadapp
sudo xmutil loadapp kv260-smartcam
ls /lib/firmware/ap1302_ar1335_single_fw.bin
smartcam --mipi -W 1920 -H 1080 --target dp --nodet
```

- [ ] smartcam firmware 설치
- [ ] `loadapp kv260-smartcam` 성공
- [ ] AP1302 firmware blob 존재
- [ ] **DP/HDMI에 AR1335 영상 출력**

선택:

```bash
smartcam --mipi -W 1920 -H 1080 --target rtsp --nodet
# 호스트: ffplay rtsp://<ip>:5000/test
```

- [ ] (선택) RTSP 스트림 확인

PYNQ 복귀:

```bash
sudo xmutil unloadapp
```

## Phase 5 — Selftest (PYNQ, test_apps 제외)

```bash
cd /usr/local/share/pynq-venv/lib/python3.10/site-packages/pynq_composable/runtime_tests
sudo python3 -m pytest test_composable.py test_mmio_partial_bitstreams.py
sudo python3 -m pytest /usr/local/share/pynq-venv/lib/python3.10/site-packages/pynq_dpu/tests
```

- [ ] composable tests pass
- [ ] DPU tests pass
- [ ] `test_apps.py` skip (OpenCV/USB — AR1335 대체 아님)

## Phase 6 — 노트북

**Smartcam (AR1335)**

- [ ] `02.mipi-dp.sh` 또는 smartcam Jupyter (`smartcam-install.py`)

**Kria-PYNQ (PYNQ stack)**

- [ ] `display_port_introduction.ipynb`
- [ ] ~~`mipi_to_displayport.ipynb`~~ → Pcam/RPi용, AR1335 pass 기준 **아님**
- [ ] ~~`opencv_*_webcam.ipynb`~~ → USB 필요, skip
- [ ] `pynq_helloworld` 또는 DPU 예제 1개

## Phase 7 — 결과 기록

| 항목 | 결과 | 로그 |
|------|------|------|
| install.sh | | |
| PYNQ overlay | | |
| **AR1335 smartcam DP** | | |
| composable selftest | | |
| DPU selftest | | |

## Go/No-Go

- [ ] Ubuntu 22.04 + install.sh OK
- [ ] **AR1335 MIPI → DP 출력 OK**
- [ ] composable + DPU selftest OK
- [ ] xmutil ↔ PYNQ overlay 전환 절차 확인

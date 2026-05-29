# KV260 Board Test Checklist

## 문서 목적

KV260 Vision AI Starter Kit에 Kria-PYNQ를 설치하고 기능을 점검하기 위한 실행 체크리스트를 제공한다.

## 범위

- 포함: bring-up → install.sh → **카메라 트랙별 검증** → selftest → 노트북 → 결과 기록
- 제외: KR260/KD240, Vivado overlay 재빌드 (→ `board-setup-and-test-guide.md` §5)

## 선행 문서

- 통합 절차: `board-setup-and-test-guide.md`
- 상세 테스트: `sw-setup-deep-dive.md`
- 보드 차이: `board-variants-notes.md`
- **AR1335 카메라 모듈 (Track B)**: `kv260-mipi-camera-test-guide.md`
- **모듈 P/N·구매 정보**: `kv260-peripherals-modules.md`

---

## 카메라 테스트 트랙 비교

공통 Phase 1~3(PYNQ 설치·overlay 스모크) 이후, **아래 두 트랙을 각각 독립 실행**하고 결과를 비교한다.

| 항목 | **Track A — USB 웹캠** (기본 Kria-PYNQ) | **Track B — AR1335 IAS** (카메라 모듈) |
|------|----------------------------------------|----------------------------------------|
| 입력 장치 | USB 웹캠 | onsemi AR1335 (13MP AF RGB) |
| 연결 | USB 포트 | KV260 **J7 IAS** FFC |
| SW 스택 | PYNQ base overlay | **kv260-smartcam** + `xmutil` |
| 검증 명령/도구 | `selftest.sh`, opencv 노트북 | `smartcam --mipi` |
| PL 점유 | PYNQ `base.bit` | smartcam firmware |
| 상세 가이드 | 본 문서 Phase 4A~6A | `kv260-mipi-camera-test-guide.md` |

> **참고 (Track C — Pcam 5C):** RPi camera 포트 + `mipi_to_displayport.ipynb` / `base.mipi` 경로. AR1335(J7)와 **다른 MIPI 경로**이므로 Track B와 혼동하지 않는다.

---

## 공통 — Phase 1 Bring-up

- [ ] Canonical Ubuntu 22.04 SD 이미지 플래시
- [ ] 부트 펌웨어 2022.1+ 확인
- [ ] Ethernet/UART 연결, IP 확인
- [ ] DP/HDMI 모니터 연결
- [ ] `cat /etc/lsb-release` → 22.04 확인
- [ ] SSH 또는 콘솔 접속

## 공통 — Phase 2 PYNQ 설치

```bash
git clone https://github.com/lastoyz/Kria-PYNQ.git
cd Kria-PYNQ
git checkout review_0529
sudo bash install.sh -b KV260
```

- [ ] install.sh 약 25분 완료
- [ ] Jupyter URL 출력 확인 (`:9090/lab`)

## 공통 — Phase 3 Overlay 스모크

```bash
source /etc/profile.d/pynq_venv.sh
python3 -c "from kv260 import BaseOverlay; ol=BaseOverlay('base.bit'); print(ol.is_loaded())"
```

- [ ] JupyterLab 접속 (비밀번호 `xilinx`)
- [ ] overlay 로딩 True

---

## Track A — USB 웹캠 (기본 Kria-PYNQ)

### 필수 주변장치

- USB 웹캠 (UVC 호환, **P/N 미지정** — Workshop 예: Logitech BRIO)
- HDMI 또는 DisplayPort 모니터

> 모듈 P/N·구매: `kv260-peripherals-modules.md` § Track A

### Phase 4A — Selftest (전체)

```bash
sudo ./selftest.sh
```

- [ ] `test_apps.py` (USB 웹캠 또는 `mountains.mp4` 필요)
- [ ] `test_composable.py`
- [ ] `test_mmio_partial_bitstreams.py`
- [ ] `pynq_dpu/tests`

### Phase 5A — 노트북 (USB / PYNQ stack)

**video/**

- [ ] `display_port_introduction.ipynb`
- [ ] `opencv_filters_webcam.ipynb`
- [ ] `opencv_face_detect_webcam.ipynb` (선택)
- [ ] `mipi_to_displayport.ipynb` (Pcam 5C 있을 때만)

**microblaze/**

- [ ] `microblaze_programming.ipynb` (선택)

**pip 패키지**

- [ ] `pynq_helloworld` 예제
- [ ] `pynq-dpu` 예제 1개

### Track A Go/No-Go

- [ ] `selftest.sh` 전체 pass (또는 실패 이슈 등록)
- [ ] opencv webcam 노트북 1개 이상 pass

---

## Track B — AR1335 IAS 카메라 모듈 (별도 추가)

> 상세: `kv260-mipi-camera-test-guide.md`  
> PYNQ overlay와 smartcam firmware는 **동시 PL 점유 불가** → Track A 완료 후 진행하거나, 전환 절차를 기록한다.

### 필수 주변장치

- onsemi **AR1335** IAS 모듈 (Avnet **CAVBA-000A**) → KV260 **J7**
- HDMI 또는 DisplayPort 모니터

> 모듈 P/N·구매: `kv260-peripherals-modules.md` § Track B

### Phase 4B — AR1335 MIPI 검증 (Smartcam)

```bash
sudo apt install xlnx-firmware-kv260-smartcam   # PPA: xilinx-apps
sudo xmutil unloadapp
sudo xmutil loadapp kv260-smartcam
ls /lib/firmware/ap1302_ar1335_single_fw.bin
smartcam --mipi -W 1920 -H 1080 --target dp --nodet
```

- [ ] J7 AR1335 FFC 체결 확인
- [ ] smartcam firmware 설치
- [ ] `loadapp kv260-smartcam` 성공
- [ ] AP1302 firmware blob 존재
- [ ] **DP/HDMI에 AR1335 영상 출력**

선택 (RTSP):

```bash
smartcam --mipi -W 1920 -H 1080 --target rtsp --nodet
# 호스트: ffplay rtsp://<ip>:5000/test
```

- [ ] (선택) RTSP 스트림 확인

PYNQ 복귀:

```bash
sudo xmutil unloadapp
# BaseOverlay 재로딩
```

### Phase 5B — Selftest (Track B 변형)

Track B에서는 `test_apps.py`(OpenCV/USB 전용)를 **제외**하고 composable/DPU만 수행한다.

```bash
cd /usr/local/share/pynq-venv/lib/python3.10/site-packages/pynq_composable/runtime_tests
sudo python3 -m pytest test_composable.py test_mmio_partial_bitstreams.py
sudo python3 -m pytest /usr/local/share/pynq-venv/lib/python3.10/site-packages/pynq_dpu/tests
```

- [ ] composable tests pass
- [ ] DPU tests pass
- [ ] `test_apps.py` skip (Track A에서 별도 검증)

### Phase 6B — 노트북 (Smartcam)

- [ ] `02.mipi-dp.sh` 또는 smartcam Jupyter (`smartcam-install.py`)

### Track B Go/No-Go

- [ ] **AR1335 MIPI → DP 출력 OK**
- [ ] composable + DPU selftest OK (test_apps 제외)
- [ ] xmutil ↔ PYNQ overlay 전환 절차 확인

---

## Phase 7 — 결과 기록 (트랙 비교)

| 항목 | Track A (USB) | Track B (AR1335) | 로그 |
|------|:-------------:|:----------------:|------|
| install.sh | | | |
| PYNQ overlay | | | |
| 카메라 영상 출력 | | | |
| selftest (전체/변형) | | | |
| 핵심 노트북 | | | |
| PL 전환 (xmutil) | N/A | | |

## 전체 Go/No-Go

- [ ] Ubuntu 22.04 + install.sh OK
- [ ] Jupyter + overlay OK
- [ ] **Track A**: selftest + USB webcam 노트북 OK
- [ ] **Track B**: AR1335 smartcam DP 출력 OK (해당 하드웨어 있을 때)

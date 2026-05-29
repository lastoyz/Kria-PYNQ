# KV260 AR1335 Camera Module Test Guide

## 문서 목적

KV260 **Track B** — onsemi **AR1335 IAS 모듈**(J7) 카메라 입력 검증 절차를 정리한다.

> **Track A (USB 웹캠)** 은 Kria-PYNQ 기본 흐름(`selftest.sh`, opencv 노트북)으로 `kv260-board-test-checklist.md` Phase 4A~5A를 따른다.  
> 본 문서는 Track B 전용이며, 두 트랙을 **비교 검증**하는 것을 전제로 한다.

## 본 가이드 대상 카메라

| 항목 | 값 |
|------|-----|
| 모델 | **AR1335** (13MP Auto-Focus RGB) |
| 제조/분류 | onsemi **IAS (Imager Access System)** |
| 연결 | KV260 carrier **J7 IAS** 커넥터 |
| ISP | onsemi **AP1302** (J7 전용) |
| Avnet P/N | **CAVBA-000A** |

> P/N·구매·참조 전체: `kv260-peripherals-modules.md`

## 범위

- 포함: AR1335 하드웨어 경로, Smartcam/xmutil 검증, Track A와의 비교, PL 전환
- 제외: IAS J8 직결 FPGA 커스텀 PL, Vitis overlay 개발 상세

---

## Track A vs Track B vs Track C

KV260에는 **서로 다른 카메라 입력 경로**가 있다.

| 트랙 | 입력 | 커넥터 | SW 스택 | 검증 |
|------|------|--------|---------|------|
| **A — USB 웹캠** | USB cam | USB | PYNQ base overlay | `selftest.sh`, `opencv_*_webcam.ipynb` |
| **B — AR1335 IAS** | AR1335 | **J7** | kv260-smartcam + xmutil | `smartcam --mipi` |
| **C — Pcam 5C** | Pcam 5C | RPi camera | PYNQ `base.mipi` | `mipi_to_displayport.ipynb` |

Kria-PYNQ `base.mipi` / `mipi_to_displayport.ipynb`는 **Track C (RPi 포트)** 용이다.  
`base.dtsi`에 AP1302/AR1335 노드가 있으나, 배포 bitstream과 Python `base.mipi`가 **J7 AR1335를 직접 구동한다고 가정하면 안 된다**.

### PL 점유 주의

PYNQ base overlay(`base.bit`)와 smartcam firmware는 **동시 PL 점유 불가**.

- Track A → Track B: `BaseOverlay.free()` 또는 재부팅 → `xmutil loadapp kv260-smartcam`
- Track B → Track A: `xmutil unloadapp` → `BaseOverlay("base.bit")` 재로딩

---

## 사전 조건 (공통)

Track B 실행 전 **Phase 1~3** 완료 (`kv260-board-test-checklist.md`):

- Ubuntu 22.04 + `install.sh -b KV260`
- Jupyter `:9090/lab`
- `BaseOverlay("base.bit")` 로딩 확인

Track A(selftest 전체)를 먼저 수행한 뒤 Track B로 전환하는 것을 권장한다.

---

## Phase B-1 — 하드웨어 준비

### AR1335 (J7)

- 전원 OFF 상태에서 **J7 IAS**에 AR1335 모듈 장착
- FFC 케이블 방향·체결 확인 ([IAS 센서 통합 가이드](https://xilinx.github.io/kria-apps-docs/kv260/2022.1/build/html/docs/integrating_new_sensors.html))
- KV260 V2: AR1335 auto-focus 지원 / V1: 거리에 따라 흐릴 수 있음

### 모니터

- DisplayPort 또는 HDMI (smartcam DP 출력 확인용)

### AP1302 firmware blob

```bash
ls /lib/firmware/ap1302_ar1335_single_fw.bin
```

없으면 아래 Smartcam 패키지 설치 후 확인.

---

## Phase B-2 — Smartcam 설치 및 MIPI 검증

공식 절차: [Smart Camera Application Deployment](https://xilinx.github.io/kria-apps-docs/kv260/2022.1/build/html/docs/smartcamera/docs/app_deployment.html)

### 1) Smartcam firmware 설치

```bash
sudo add-apt-repository ppa:xilinx-apps
sudo apt update
sudo apt install xlnx-firmware-kv260-smartcam
sudo xmutil listapps
```

`kv260-smartcam`이 목록에 표시되는지 확인.

### 2) Smartcam overlay 로드

```bash
sudo xmutil unloadapp
sudo xmutil loadapp kv260-smartcam
```

> `xmutil desktop_disable` 후 UART로 진행하는 것이 안정적일 수 있다 (DP blank 가능).

### 3) MIPI 스모크 — DP 출력

```bash
smartcam --mipi -W 1920 -H 1080 --target dp --nodet
```

또는:

```bash
bash /opt/xilinx/kv260-smartcam/bin/02.mipi-dp.sh
```

**pass 기준**: DP/HDMI 모니터에 AR1335 영상 표시.

### 4) MIPI RTSP (선택)

보드:

```bash
smartcam --mipi -W 1920 -H 1080 --target rtsp --nodet
```

호스트 PC:

```bash
ffplay rtsp://<board_ip>:5000/test
```

### 5) PYNQ overlay로 복귀 (Track A 재개 시)

```bash
sudo xmutil unloadapp
```

Jupyter에서 `BaseOverlay("base.bit")` 재로딩.

---

## Phase B-3 — Selftest (Track B 변형)

`test_apps.py`는 `VSource.OpenCV` (USB 또는 `mountains.mp4`) 전용 → **Track A에서 검증**.

Track B에서는 composable/DPU만:

```bash
cd /usr/local/share/pynq-venv/lib/python3.10/site-packages/pynq_composable/runtime_tests
sudo python3 -m pytest test_composable.py test_mmio_partial_bitstreams.py
sudo python3 -m pytest /usr/local/share/pynq-venv/lib/python3.10/site-packages/pynq_dpu/tests
```

| 항목 | Track A | Track B |
|------|---------|---------|
| USB/OpenCV (`test_apps`) | ✓ | skip |
| composable PL | ✓ | ✓ |
| DPU | ✓ | ✓ |
| MIPI 영상 | — | `smartcam --mipi` |

---

## Track A / B / C 비교 요약

| 비교 항목 | Track A (USB) | Track B (AR1335) | Track C (Pcam) |
|-----------|---------------|------------------|----------------|
| 해상도/품질 | 웹캠 의존 | 13MP AF (smartcam 설정) | Pcam 5C (720p 등) |
| PYNQ 통합 | ✓ (opencv 노트북) | ✗ (별도 smartcam) | ✓ (`base.mipi`) |
| selftest `test_apps` | ✓ | skip | skip (USB 전용) |
| PL 전환 필요 | — | xmutil ↔ PYNQ | PYNQ overlay 내 |

---

## 트러블슈팅

### Smartcam MIPI 인식 실패

- J7 체결·FFC 방향 재확인 (RPi 포트 아님)
- `ap1302_ar1335_single_fw.bin` 존재 확인
- `sudo xmutil loadapp kv260-smartcam` 재실행
- `dmesg | grep -i ap1302`

### PYNQ `base.mipi` / `mipi_to_displayport` 실패 (AR1335만 장착 시)

- **Track C 경로**이므로 AR1335-only 환경에서 fail은 예상 가능
- AR1335 검증은 Track B (`smartcam --mipi`) 결과를 기준으로 삼는다

### xmutil / PYNQ overlay 충돌

- smartcam 테스트 전 `BaseOverlay.free()` 또는 재부팅
- smartcam 종료 후 `xmutil unloadapp` → PYNQ overlay 재로딩

### Auto-focus (KV260 V1)

- V1 carrier는 AR1335 AF 미지원 — 특정 거리에서 흐릴 수 있음

---

## Go/No-Go (Track B)

- [ ] J7 AR1335 물리 장착 확인
- [ ] `xlnx-firmware-kv260-smartcam` 설치
- [ ] `xmutil loadapp kv260-smartcam` 성공
- [ ] `smartcam --mipi --target dp --nodet` 영상 출력
- [ ] composable + DPU selftest pass (test_apps 제외)
- [ ] PL 전환(xmutil ↔ PYNQ overlay) 절차 기록
- [ ] Track A 결과와 비교 기록 (`kv260-board-test-checklist.md` Phase 7)

## 관련 문서

- `kv260-board-test-checklist.md` (Track A + B 통합 체크리스트)
- `board-setup-and-test-guide.md`
- `structure/kv260-directory-notes.md`

## 참고 링크

- [Integrating New IAS Sensor Modules (KV260 J7)](https://xilinx.github.io/kria-apps-docs/kv260/2022.1/build/html/docs/integrating_new_sensors.html)
- [Smart Camera Deployment](https://xilinx.github.io/kria-apps-docs/kv260/2022.1/build/html/docs/smartcamera/docs/app_deployment.html)

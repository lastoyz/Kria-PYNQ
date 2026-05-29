# KV260 MIPI Camera Test Guide

## 문서 목적

KV260에 **onsemi AR1335 IAS 모듈**(J7)이 장착된 환경에서 카메라 입력을 검증하는 방법을 정리한다.

## 본 프로젝트 카메라

| 항목 | 값 |
|------|-----|
| 모델 | **AR1335** (13MP Auto-Focus RGB) |
| 제조/분류 | onsemi **IAS (Imager Access System)** |
| 연결 | KV260 carrier **J7 IAS** 커넥터 |
| ISP | onsemi **AP1302** (J7 전용) |
| Avnet P/N 예 | CAVBA-000A |

## 범위

- 포함: AR1335 하드웨어 경로, Smartcam/xmutil 검증, PYNQ install.sh와의 관계, selftest 대체
- 제외: IAS J8 직결 FPGA 커스텀 PL, Vitis overlay 개발 상세

---

## 중요: Pcam 5C / RPi 포트 vs AR1335 IAS (J7)

KV260에는 **서로 다른 MIPI 입력 경로**가 있다.

| 경로 | 커넥터 | 대표 모듈 | Kria-PYNQ base overlay |
|------|--------|-----------|------------------------|
| **IAS + AP1302** | **J7** | **AR1335**, AR0144 | `base.mipi` 노트북과 **PL 연결 불일치 가능** |
| RPi camera | RPi FFC 포트 | Digilent Pcam 5C | `mipi_to_displayport.ipynb` 대상 |

Kria-PYNQ `kv260/base` Vivado 설계의 top-level MIPI 포트(`mipi_phy_if`)와 노트북 `base.mipi` API는 **README상 Pcam 5C / RPi camera** 경로를 전제로 한다.

`base.dtsi`에 AP1302/AR1335 노드가 있으나, 배포 bitstream(`kv260_base_2.7.zip`)과 Python `base.mipi`가 **J7 AR1335를 직접 구동한다고 가정하면 안 된다**.

### 결론 (AR1335 장착 시)

| 검증 목적 | 권장 경로 |
|-----------|-----------|
| **AR1335 MIPI 영상 입출력** | **kv260-smartcam** + `xmutil` + `smartcam --mipi` |
| PYNQ overlay / composable / DPU | Kria-PYNQ `install.sh` (기존) |
| USB 웹캠 OpenCV | `opencv_*_webcam.ipynb` (별도 USB cam 필요) |

PYNQ base overlay와 smartcam firmware는 **동시 PL 점유 불가** → 테스트 시 `xmutil unloadapp` / overlay 재로딩으로 전환.

---

## 하드웨어 준비

### AR1335 (J7)

- 전원 OFF 상태에서 **J7 IAS**에 AR1335 모듈 장착
- FFC 케이블 방향·체결 확인 ([IAS 센서 통합 가이드](https://xilinx.github.io/kria-apps-docs/kv260/2022.1/build/html/docs/integrating_new_sensors.html))
- KV260 V2: AR1335 auto-focus 지원 / V1: 거리에 따라 흐릴 수 있음

### 모니터

- DisplayPort 또는 HDMI (smartcam DP 출력 확인용)

### AP1302 firmware blob

J7 AR1335는 ISP firmware가 필요하다.

```bash
ls /lib/firmware/ap1302_ar1335_single_fw.bin
```

없으면 smartcam firmware 패키지 설치 후 확인 (아래 § Smartcam 설치).

---

## Phase A — PYNQ 환경 (Kria-PYNQ)

기존 체크리스트대로 `install.sh -b KV260` 완료.

- Jupyter `:9090/lab`
- composable / DPU selftest (`test_apps.py` 제외)
- **카메라 검증은 Phase B에서 수행**

---

## Phase B — AR1335 MIPI 검증 (Smartcam)

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
sudo xmutil unloadapp          # 기존 accelerator 있으면
sudo xmutil loadapp kv260-smartcam
```

> `xmutil desktop_disable` 후 UART로 진행하는 것이 안정적일 수 있다 (DP blank 가능).

### 3) MIPI 스모크 — DP 출력 (AI 없음)

모니터 연결 후:

```bash
smartcam --mipi -W 1920 -H 1080 --target dp --nodet
```

또는:

```bash
bash /opt/xilinx/kv260-smartcam/bin/02.mipi-dp.sh
```

**pass 기준**: DP/HDMI 모니터에 AR1335 영상 표시.

### 4) MIPI RTSP (선택)

보드에서:

```bash
smartcam --mipi -W 1920 -H 1080 --target rtsp --nodet
```

호스트 PC:

```bash
ffplay rtsp://<board_ip>:5000/test
```

### 5) PYNQ overlay로 복귀 (필요 시)

```bash
sudo xmutil unloadapp
```

Jupyter에서 `BaseOverlay("base.bit")` 재로딩.

---

## Phase C — Kria-PYNQ `mipi_to_displayport.ipynb` (참고)

Pcam 5C / RPi 포트용 노트북. **AR1335@J7 전용 검증으로 사용하지 않는다.**

AR1335만 있는 경우 이 노트북 실패는 **예상 가능** — Smartcam Phase B 결과를 카메라 pass 기준으로 삼는다.

---

## Selftest (MIPI / AR1335 환경)

`test_apps.py`는 `VSource.OpenCV` (USB 또는 `mountains.mp4`) 전용.

**AR1335 MIPI 검증과 무관** → 제외.

```bash
cd /usr/local/share/pynq-venv/lib/python3.10/site-packages/pynq_composable/runtime_tests
sudo python3 -m pytest test_composable.py test_mmio_partial_bitstreams.py
sudo python3 -m pytest /usr/local/share/pynq-venv/lib/python3.10/site-packages/pynq_dpu/tests
```

| 항목 | AR1335 프로젝트 기준 |
|------|---------------------|
| MIPI 영상 | `smartcam --mipi` (Phase B) |
| composable PL | pytest (Phase A) |
| DPU | pytest (Phase A) |
| test_apps | skip |

---

## 트러블슈팅

### Smartcam MIPI 인식 실패

- J7 체결·FFC 방향 재확인
- `ap1302_ar1335_single_fw.bin` 존재 확인
- `sudo xmutil loadapp kv260-smartcam` 재실행
- `dmesg | grep -i ap1302` 로 ISP 드라이버/firmware 로드 확인

### PYNQ `base.mipi` readframe 실패 (AR1335 장착 시)

- **정상적일 수 있음** — base overlay가 RPi MIPI 경로용이기 때문
- AR1335 검증은 Smartcam 경로 사용

### xmutil / PYNQ overlay 충돌

- smartcam 테스트 전 `BaseOverlay.free()` 또는 재부팅
- smartcam 종료 후 `xmutil unloadapp` → PYNQ overlay 재로딩

### Auto-focus (KV260 V1)

- V1 carrier는 AR1335 AF 미지원 — 특정 거리에서 흐릴 수 있음

---

## Go/No-Go (AR1335)

- [ ] J7 AR1335 물리 장착 확인
- [ ] `xlnx-firmware-kv260-smartcam` 설치
- [ ] `xmutil loadapp kv260-smartcam` 성공
- [ ] `smartcam --mipi --target dp --nodet` 영상 출력
- [ ] PYNQ composable + DPU selftest pass
- [ ] PL 전환(xmutil ↔ PYNQ overlay) 절차 기록

## 관련 문서

- `kv260-board-test-checklist.md`
- `board-setup-and-test-guide.md`
- `structure/kv260-directory-notes.md`

## 참고 링크

- [Integrating New IAS Sensor Modules (KV260 J7)](https://xilinx.github.io/kria-apps-docs/kv260/2022.1/build/html/docs/integrating_new_sensors.html)
- [Smart Camera Deployment](https://xilinx.github.io/kria-apps-docs/kv260/2022.1/build/html/docs/smartcamera/docs/app_deployment.html)
- [KV260 Workshop — camera setup](https://github.com/Xilinx/Xilinx_Kria_KV260_Workshop/blob/main/Linux%20set-up.md)

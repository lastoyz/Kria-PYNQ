# KV260 Peripherals & Camera Modules

## 문서 목적

KV260 테스트에 사용하는 **카메라 모듈·주변장치**의 식별 정보(모델, P/N, 커넥터, 구매/참조 링크)를 한곳에 정리한다.

테스트 절차는 `kv260-board-test-checklist.md`, Track B 상세는 `kv260-mipi-camera-test-guide.md`를 따른다.

---

## 카메라 모듈 요약 (테스트 트랙별)

| 트랙 | 장치 | 센서/ISP | KV260 연결 | P/N / SKU | 검증 |
|------|------|----------|------------|-----------|------|
| **A** | USB 웹캠 | (장치 의존, UVC) | **U44 / U46** USB | **미지정** (UVC 호환) | `selftest.sh`, opencv 노트북 |
| **B** | AR1335 IAS | AR1335 + **AP1302** ISP | **J7** IAS FFC | **CAVBA-000A** (Avnet) | `smartcam --mipi` |
| **C** | Pcam 5C | Omnivision **OV5640** | **RPi camera** FFC | **410-358** (Digilent) | `mipi_to_displayport.ipynb` |

> J7(IAS+AP1302)과 RPi camera 포트는 **서로 다른 MIPI/PL 경로**이다.

---

## Track A — USB 웹캠

### 식별 정보

| 항목 | 값 |
|------|-----|
| 분류 | **UVC (USB Video Class)** 호환 USB 카메라 |
| Kria-PYNQ README | “A USB Webcamera” (특정 모델 **미지정**) |
| KV260 연결 | Carrier **U44** 또는 **U46** USB 포트 |
| 본 프로젝트 문서 | 특정 P/N **없음** — 범용 UVC 웹캠 사용 |

### AMD/KV260 Workshop 권장 예 (참고)

| 항목 | 값 |
|------|-----|
| 모델 | **Logitech BRIO** |
| 용도 | Smart Camera AA (`smartcam --usb`) 등 공식 워크숍 예시 |
| 비고 | Kria-PYNQ `test_apps` / opencv 노트북과 **동일 제품을 요구하지 않음** |

### SW / 테스트 연계

| 항목 | 설명 |
|------|------|
| `test_apps.py` | OpenCV 입력 (`VSource.OpenCV`) — USB 웹캠 **또는** `mountains.mp4` |
| 노트북 | `opencv_filters_webcam.ipynb`, `opencv_face_detect_webcam.ipynb`, `display_port_introduction.ipynb` |
| 대체 입력 | USB 없을 때 `mountains.mp4`로 `test_apps`만 우회 가능 (opencv 노트북은 웹캠 필요) |

### 구매 / 참조

- Kria-PYNQ upstream: [README § Selftest](https://github.com/lastoyz/Kria-PYNQ) — “USB Webcamera”
- KV260 Workshop: [Setup Board](https://github.com/Xilinx/Xilinx_Kria_KV260_Workshop/blob/main/Part%201:%20Setup%20Board.md) — USB cam → U44/U46, Logitech BRIO 예시

---

## Track B — onsemi AR1335 IAS (본 프로젝트 카메라 모듈)

### 식별 정보

| 항목 | 값 |
|------|-----|
| 모델 | **AR1335** (13MP Auto-Focus RGB) |
| 제조/분류 | onsemi **IAS (Imager Access System)** |
| Avnet 모듈 P/N | **CAVBA-000A** |
| 모듈 설명 | 4K imaging, 4208×3120 @ 30fps, VCM auto-focus, 74° FOV |
| KV260 연결 | Carrier **J7** IAS FFC (AP1302 ISP 경유) |
| ISP | onsemi **AP1302** (J7 전용) |
| Firmware blob | `/lib/firmware/ap1302_ar1335_single_fw.bin` |

### SW / 테스트 연계

| 항목 | 설명 |
|------|------|
| 검증 | `xlnx-firmware-kv260-smartcam` + `xmutil loadapp kv260-smartcam` + `smartcam --mipi` |
| PYNQ `base.mipi` | **Track B 대상 아님** (RPi MIPI 경로용) |
| PL 전환 | smartcam firmware ↔ PYNQ `base.bit` **동시 점유 불가** |

### 하드웨어 주의

- KV260 **V2**: AR1335 auto-focus 지원
- KV260 **V1**: AF 미지원 — 거리에 따라 흐릴 수 있음

### 구매 / 참조

- Avnet IAS Camera Modules: [CAVBA-000A product page](https://www.avnet.com/americas/products/avnet-boards/avnet-board-families/add-on-products/ias-camera-modules/)
- onsemi AR1335: [AR1335 product page](https://www.onsemi.com/products/sensors/image-sensors-processors/image-sensors/ar1335)
- KV260 IAS 통합: [Integrating New IAS Sensor Modules](https://xilinx.github.io/kria-apps-docs/kv260/2022.1/build/html/docs/integrating_new_sensors.html)
- Smartcam 배포: [Smart Camera Application Deployment](https://xilinx.github.io/kria-apps-docs/kv260/2022.1/build/html/docs/smartcamera/docs/app_deployment.html)
- KV260 Workshop (J7 AR1335): [Setup Board — peripheral table](https://github.com/Xilinx/Xilinx_Kria_KV260_Workshop/blob/main/Part%201:%20Setup%20Board.md)

---

## Track C — Digilent Pcam 5C (참고)

### 식별 정보

| 항목 | 값 |
|------|-----|
| 제품명 | **Pcam 5C** (5 MP Fixed-Focus Color Camera Module) |
| Digilent SKU | **410-358** |
| 센서 | Omnivision **OV5640** (5MP) |
| KV260 연결 | **Raspberry Pi camera** FFC 포트 (15-pin FFC) |
| 인터페이스 | 2-lane MIPI CSI-2 |

### SW / 테스트 연계

| 항목 | 설명 |
|------|------|
| Kria-PYNQ | `base.mipi`, `mipi_to_displayport.ipynb` |
| upstream README | [Digilent Pcam 5C](https://digilent.com/reference/add-ons/pcam-5c/start?redirect=1) 링크 |

### 구매 / 참조

- Digilent Store: [Pcam 5C (410-358)](https://digilent.com/shop/pcam-5c-5-mp-fixed-focus-color-camera-module/)
- Reference Manual: [Pcam 5C Reference](https://digilent.com/reference/add-ons/pcam-5c/start)

---

## 기타 KV260 주변장치 (선택)

KV260 Workshop / Kria-PYNQ에서 언급되는 추가 주변장치:

| 용도 | 장치 | P/N / SKU | KV260 연결 | 비고 |
|------|------|-----------|------------|------|
| 디스플레이 | HDMI 또는 DisplayPort 모니터 | — | **J5** (HDMI) / **J6** (DP) | selftest, smartcam DP 출력 |
| IAS (Defect Detection 예) | AR0144 1MP GS | **CAV10-000A** (Avnet) | **J7** | Smartcam/AA4용, AR1335와 **교체** 장착 |
| 오디오 (Smartcam RTSP) | PMOD I2S2 | **410-379** (Digilent) | **J2** PMOD | `smartcam --audio` (선택) |
| PMOD/Grove | Grove/PMOD 키트 | (키트별) | PMOD | microblaze 노트북 |

---

## KV260 Starter Kit (보드 본체)

| 항목 | 값 |
|------|-----|
| 키트명 | **KV260 Vision AI Starter Kit** |
| SOM | Kria K26 (예: XCK26-Si-EV-G) |
| 공식 Getting Started | [KV260 Ubuntu Setup](https://www.xilinx.com/products/som/kria/kv260-vision-starter-kit/kv260-getting-started-ubuntu/setting-up-the-sd-card-image.html) |

키트 기본 구성(워크숍 기준): 12V/3A 전원, microSD, micro-USB(UART), Ethernet 케이블 등. **카메라 모듈은 키트에 미포함** — Track A/B/C 장치는 별도 준비.

---

## 관련 문서

- `kv260-board-test-checklist.md` — Track A/B 실행 체크리스트
- `kv260-mipi-camera-test-guide.md` — Track B 절차
- `board-variants-notes.md` — 보드별 차이
- `sw-setup-deep-dive.md` — selftest/노트북 상세

## 관련 upstream

- `README.md` § Selftest, Included Overlays
- `kv260/base/notebooks/video/mipi_to_displayport.ipynb` (Pcam 5C)

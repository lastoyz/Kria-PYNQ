# KV260 MIPI Camera Test Guide

## 문서 목적

KV260 보드 테스트 시 **USB 웹캠 대신 MIPI 카메라 모듈**로 영상 입력 경로를 구성하고 검증하는 방법을 정리한다.

## 범위

- 포함: 하드웨어 연결, Kria-PYNQ base overlay MIPI 경로, 노트북/스모크 테스트, selftest 대체 전략
- 제외: IAS(J7) Avnet 센서 모듈용 Vitis smart-camera bitstream, composable DFX 커스텀 파이프라인 개발

## USB 웹캠 vs MIPI — 경로 차이

| 항목 | USB 웹캠 | MIPI (Pcam 5C) |
|------|----------|----------------|
| 입력 API | OpenCV `VideoCapture(0)` | PYNQ `base.mipi` (`pynq.lib.video`) |
| PL 사용 | 주로 PS USB + DisplayPort | MIPI CSI-2 RX + ISP 파이프라인 (base overlay) |
| 대표 노트북 | `opencv_filters_webcam.ipynb`, `opencv_face_detect_webcam.ipynb` | `mipi_to_displayport.ipynb` |
| selftest `test_apps.py` | **직접 사용** (OpenCV 소스) | **미사용** (MIPI 소스 아님) |
| KV260 커넥터 | USB (U44/U46 등) | **Raspberry Pi camera** FFC (15-pin) |

Kria-PYNQ README 기준, base overlay의 MIPI는 **Raspberry Pi camera 인터페이스 + Digilent Pcam 5C** 조합을 전제로 한다.

## 지원 MIPI 모듈 (Kria-PYNQ base overlay)

### 공식 권장

- [Digilent Pcam 5C](https://digilent.com/reference/add-ons/pcam-5c/start)
  - OV5640, 2-lane MIPI CSI-2
  - Raspberry Pi 호환 15-pin FFC
  - Kria-PYNQ `kv260/base` Vivado 설계의 `mipi_phy_if`, `cam_gpio`, I2C 경로와 매칭

### KV260 carrier 연결

- **Raspberry Pi camera 커넥터**에 Pcam 5C FFC 연결 (contacts facing up, carrier 측 가이드 참조)
- **DisplayPort 또는 HDMI** 모니터 (MIPI→DP 출력 확인용)
- 전원: carrier 정격 전원 (12V barrel jack 등, carrier 매뉴얼 준수)

### IAS(J7) 모듈과의 구분 (중요)

KV260 carrier의 **IAS 센서 커넥터(J7)** 용 Avnet AR1335/AR0144 등은 AMD Vitis smart-camera 앱용으로 문서화되어 있다 ([KV260 Workshop](https://github.com/Xilinx/Xilinx_Kria_KV260_Workshop/blob/main/Linux%20set-up.md)).

Kria-PYNQ **base overlay**의 `base.mipi` API는 이 IAS 모듈이 아니라 **Raspberry Pi camera 포트 + Pcam 5C** 경로를 대상으로 한다. IAS 모듈만 보유한 경우 base overlay MIPI 노트북과 **호환되지 않을 수 있다**.

## 선행 조건

- `install.sh -b KV260` 완료
- `kv260` base overlay 로딩 가능
- Pcam 5C + FFC 케이블 + DP/HDMI 모니터

참조: `board-setup-and-test-guide.md`, `kv260-board-test-checklist.md`

## 테스트 방법

### 1) MIPI 스모크 테스트 (Python 셀)

Jupyter 또는 SSH에서:

```python
from kv260 import BaseOverlay
from pynq.lib.video import VideoMode
import PIL.Image

base = BaseOverlay("base.bit")
mipi = base.mipi

videomode = VideoMode(1280, 720, 24)
mipi.configure(videomode)
mipi.start()

frame = mipi.readframe()
print("frame shape:", frame.shape)

# 채널 순서 보정 (PIL 표시용)
PIL.Image.fromarray(frame[:, :, [2, 1, 0]])

mipi.stop()
base.free()
```

**pass 기준**: 예외 없이 frame shape 출력 (예: `(720, 1280, 3)`)

### 2) 노트북 전체 테스트 (권장)

경로: `kv260/base/notebooks/video/mipi_to_displayport.ipynb`

설치 후 Jupyter 경로 (예):

```
/root/jupyter_notebooks/kv260/video/mipi_to_displayport.ipynb
```

실행 흐름:

1. `BaseOverlay("base.bit")` 로드
2. `base.mipi` configure/start
3. 단일 frame Jupyter 표시
4. `DisplayPort` configure (1280×720@24)
5. 200 frame MIPI→DP 루프 + FPS 출력
6. cleanup (`displayport.close()`, `mipi.stop()`, `base.free()`)

**pass 기준**:

- [ ] 노트북에서 still frame 표시
- [ ] DP/HDMI 모니터에 실시간 영상 출력
- [ ] FPS 로그 출력 (0 FPS 아님)

### 3) selftest — MIPI 전용 환경에서의 대체

`test_apps.py`는 composable 파이프라인에서 **`VSource.OpenCV`** (USB 웹캠 또는 `mountains.mp4`)만 사용한다.

```python
# pynq_composable/runtime_tests/test_apps.py (요지)
VSource.OpenCV  # USB cam (index 0) 또는 ../mountains.mp4
```

따라서 **MIPI만 연결된 환경**에서는:

| 옵션 | 방법 | 비고 |
|------|------|------|
| A (권장) | `test_apps.py` **제외**, MIPI 노트북으로 대체 | 실물 MIPI 검증에 부합 |
| B | `mountains.mp4`를 runtime_tests 경로에 배치 후 `test_apps` 실행 | USB/MIPI 미검증, composable만 확인 |
| C | USB 웹캠 추가 연결 | README 기본 selftest 조건 |

MIPI 프로젝트 권장 selftest:

```bash
cd /usr/local/share/pynq-venv/lib/python3.10/site-packages/pynq_composable/runtime_tests
sudo python3 -m pytest test_composable.py test_mmio_partial_bitstreams.py
sudo python3 -m pytest /usr/local/share/pynq-venv/lib/python3.10/site-packages/pynq_dpu/tests
```

MIPI 검증은 별도로 `mipi_to_displayport.ipynb` 또는 §1 스모크 테스트 수행.

## MIPI 전용 테스트 체크리스트

### 하드웨어

- [ ] Pcam 5C FFC가 **Raspberry Pi camera** 포트에 올바른 방향으로 체결
- [ ] DP/HDMI 모니터 연결
- [ ] (USB 웹캠 **미연결** — MIPI-only 테스트 시)

### 소프트웨어

- [ ] `source /etc/profile.d/pynq_venv.sh`
- [ ] `BaseOverlay("base.bit")` 로딩
- [ ] §1 MIPI 스모크 pass
- [ ] `mipi_to_displayport.ipynb` pass

### selftest (MIPI-only 변형)

- [ ] `test_composable.py` pass
- [ ] `test_mmio_partial_bitstreams.py` pass
- [ ] `pynq_dpu/tests` pass
- [ ] `test_apps.py` → **skip 또는 mountains.mp4 대체** (MIPI 대체 아님, 명시 기록)

## 트러블슈팅

### `base.mipi` 초기화 실패 / readframe hang

- FFC 방향·체결 상태 재확인 ([Pcam 5C Reference](https://digilent.com/reference/add-ons/pcam-5c/reference-manual))
- Raspberry Pi camera 포트 사용 확인 (IAS J7 모듈과 혼동 여부)
- overlay 재로딩: `base.free()` 후 `BaseOverlay("base.bit")` 재시도
- I2C/cam_gpio: base overlay가 cam enable 및 I2C switch를 제어 (`base.xdc`, `base.tcl` 참조)

### 노트북 frame은 보이나 DP 출력 없음

- 모니터 DP/HDMI 입력 소스 확인
- `displayport.configure(videomode, PIXEL_RGB)` 해상도가 MIPI mode(1280×720)와 일치하는지 확인

### OpenCV 웹캠 노트북을 MIPI에 그대로 적용 불가

`opencv_filters_webcam.ipynb` 등은 `cv2.VideoCapture(0)` 고정. MIPI frame을 OpenCV 파이프에 넣으려면 **별도 브릿지 코드**가 필요하며, Kria-PYNQ 기본 패키지에는 포함되지 않는다.

## 관련 문서

- `kv260-board-test-checklist.md` (MIPI 변형 체크리스트)
- `sw-setup-deep-dive.md`
- `structure/kv260-directory-notes.md`

## 관련 소스

- `kv260/base/notebooks/video/mipi_to_displayport.ipynb`
- `kv260/base/base.tcl` — `create_hier_cell_mipi`
- `kv260/base/vivado/constraints/base.xdc` — MIPI/I2C/cam_gpio
- `README.md` § Base Overlay

## 참고 링크

- [Kria-PYNQ mipi_to_displayport.ipynb](https://github.com/Xilinx/Kria-PYNQ/blob/main/kv260/base/notebooks/video/mipi_to_displayport.ipynb)
- [Digilent Pcam 5C](https://digilent.com/reference/add-ons/pcam-5c/start)
- [KV260 Workshop — camera options](https://github.com/Xilinx/Xilinx_Kria_KV260_Workshop/blob/main/Linux%20set-up.md)

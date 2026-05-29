# Board Variants Notes

## 목적

KV260, KR260, KD240 보드별 공통점과 차이를 한 페이지로 정리해, 설치/테스트 적용 시 혼선을 줄인다.

## 대상 보드

- `KV260`
- `KR260`
- `KD240`

## 공통점

- Ubuntu 22.04 LTS + `install.sh -b <BOARD>` 설치 흐름 동일
- PYNQ 3.0.1, Python 3.10 venv, Jupyter `:9090/lab` (비밀번호 `xilinx`)
- device tree overlay (`dts/pynq.dts` → `pynq.dtbo`) 공통 적용
- `pynq-v3.0-binaries.tar.gz`에서 gcc-mb, xclbinutil 설치

## 차이점 (실무 관점)

### 1) install.sh 보드별 pip 패키지

| 패키지 | KV260 | KR260 | KD240 |
|--------|:-----:|:-----:|:-----:|
| `pynq_helloworld` | ✓ | ✓ | — |
| `pip install .` (kv260 base) | ✓ | — | — |
| `PYNQ_Composable_Pipeline` | ✓ | — | — |
| `PYNQ_Peripherals` | ✓ | — | — |
| `pynq-dpu==2.5` | ✓ | ✓ | — |
| `MakarenaLabs/DPU-PYNQ` | — | — | ✓ (editable) |

### 2) selftest.sh

| 보드 | 생성 여부 | 테스트 범위 |
|------|-----------|-------------|
| KV260 | ✓ | composable runtime (3) + DPU |
| KR260 | ✓ | DPU only |
| KD240 | **✗** | 수동 노트북 테스트 필요 |

### 3) 로컬 저장소 자산

| 자산 | KV260 | KR260 | KD240 |
|------|:-----:|:-----:|:-----:|
| `kv260/` Python 패키지 | ✓ | — | — |
| `kv260/base` Vivado 소스 | ✓ | — | — |
| 로컬 노트북 | 7개 (video/microblaze) | — | — |

KR260/KD240 overlay/노트북은 **외부 pip/git 저장소**에서만 제공된다.

### 4) KV260 카메라 입력 (테스트 트랙별)

| 트랙 | 장치 | 연결 | 검증 |
|------|------|------|------|
| **A — USB 웹캠** (기본) | USB cam (UVC) | **U44/U46** USB | `selftest.sh`, `opencv_*_webcam.ipynb` |
| **B — AR1335 IAS** | onsemi AR1335 13MP AF | **J7** | `smartcam --mipi` (`kv260-mipi-camera-test-guide.md`) |
| **C — Pcam 5C** | Digilent Pcam 5C (410-358) | RPi camera | `mipi_to_displayport.ipynb`, `base.mipi` |

- 모듈 P/N·구매 링크: **`kv260-peripherals-modules.md`**
- AR1335(J7)와 Pcam(RPi)은 **다른 MIPI/PL 경로**
- PMOD/Grove — microblaze 노트북

### 5) 노트북 정리 (install.sh)

KV260에서 제거:

- `pynq_peripherals/app*`, `grove_joystick`

KR260에서 제거:

- `common/zynq_clocks.ipynb`, `common/overlay_download.ipynb`

## 적용 규칙

- `-b` 플래그 값은 반드시 대문자 (`KV260`, `KR260`, `KD240`)
- `$BOARD` 환경변수는 `/etc/profile.d/pynq_venv.sh`에 기록됨
- KV260 overlay/노트북을 KR260/KD240에 교차 사용하지 않음

## 관련 가이드

- `board-setup-and-test-guide.md`
- `sw-setup-deep-dive.md`
- `structure/install-sh-notes.md`

## 관련 소스

- `install.sh` (board 분기: L212~259, L279~306, L337~350)
- `README.md` § Included Overlays, Selftest

# SW Setup Deep Dive

## 목적

PYNQ 설치 후 Jupyter 환경, overlay 로딩, selftest, 노트북 기반 기능 검증 절차를 정리한다.

## 범위

- Jupyter/Python 실행 환경 확인
- overlay 로딩 스모크 테스트
- selftest 실행
- 보드별 노트북 테스트 시나리오
- 제외: Ubuntu SD 이미지 준비, `install.sh` 내부 상세 (→ `structure/install-sh-notes.md`)

## 선행 조건

- `install.sh` 설치 완료
- `board-setup-and-test-guide.md` §1~2 완료

## 1) 실행 환경 점검

```bash
source /etc/profile.d/pynq_venv.sh
echo $VIRTUAL_ENV
echo $BOARD
python3 -c "import pynq; print(pynq.__version__)"
```

기대값:

- venv: `/usr/local/share/pynq-venv`
- PYNQ: `3.0.1`
- 노트북 경로: `/root/jupyter_notebooks` (또는 `$PYNQ_JUPYTER_NOTEBOOKS`)

Jupyter 접속:

- `http://<ip>:9090/lab`
- 비밀번호: `xilinx`

## 2) Overlay 로딩 스모크 테스트

### KV260

```python
from kv260 import BaseOverlay
ol = BaseOverlay("base.bit")
ol.is_loaded()
```

`setup.py` 설치 시 pre-built overlay(`kv260_base_2.7.zip`)가 자동 다운로드된다.

### KR260 / KD240

HelloWorld 또는 DPU overlay 패키지 기준으로 테스트:

```python
from pynq import Overlay
# 패키지별 overlay 경로는 pip 설치 결과 확인
```

## 3) Selftest 실행

설치 디렉터리에서 root 권한으로:

```bash
sudo ./selftest.sh
```

### KV260 selftest 구성 (`install.sh` 생성)

| 테스트 | 경로/모듈 | 주변장치 |
|--------|-----------|----------|
| `test_apps.py` | `pynq_composable/runtime_tests` | HDMI/DP 모니터 + USB 웹캠 |
| `test_composable.py` | 동일 | — |
| `test_mmio_partial_bitstreams.py` | 동일 | — |
| DPU tests | `pynq_dpu/tests` | — |

### KR260 selftest 구성

- `pynq_dpu/tests` pytest만 실행

### KD240

- `install.sh`에 selftest 생성 로직 **없음**
- DPU 노트북 수동 실행 필요 (`/root/jupyter_notebooks/kd240_notebooks/`)

## 4) 로컬 노트북 테스트 (KV260)

저장소에 포함된 노트북 (`kv260/base/notebooks/`):

### video/

| 노트북 | 검증 내용 | 주변장치 |
|--------|-----------|----------|
| `display_port_introduction.ipynb` | DisplayPort 출력 | DP/HDMI 모니터 |
| `mipi_to_displayport.ipynb` | MIPI → DP 파이프라인 | Pcam 5C, 모니터 |
| `opencv_filters_webcam.ipynb` | OpenCV 필터 + 웹캠 | USB 웹캠 |
| `opencv_face_detect_webcam.ipynb` | 얼굴 검출 | USB 웹캠 |

### microblaze/

| 노트북 | 검증 내용 | 주변장치 |
|--------|-----------|----------|
| `microblaze_programming.ipynb` | MicroBlaze IOP 기본 | PMOD (선택) |
| `microblaze_python_libraries.ipynb` | Python 라이브러리 | PMOD (선택) |
| `microblaze_c_libraries.ipynb` | C 라이브러리 | PMOD (선택) |

## 5) pip 설치 노트북 (install.sh 경유)

KV260 추가 패키지:

- `pynq_helloworld` — 이미지 리사이저 overlay 데모
- `pynq-dpu==2.5` — Vitis-AI DPU + ML 예제
- `PYNQ_Composable_Pipeline` v1.1.0-dev — composable overlay
- `PYNQ_Peripherals` — Grove/PMOD 주변장치

공통:

- `pynq-get-notebooks` — PYNQ 공식 예제 노트북
- `pynq/pynq/notebooks/common/` — common 노트북 복사

노트북 패치 (install.sh):

- `pynq.overlays.base` → `kv260` 치환
- `PMODB` → `PMODA` 치환
- wifi.ipynb 등 Kria 비호환 문구 제거

## 6) 결과 정리

| 항목 | pass/fail | 로그/비고 |
|------|-----------|-----------|
| Jupyter 접속 | | |
| overlay 로딩 | | |
| selftest | | |
| 핵심 노트북 | | |

## 완료 기준

- Jupyter + overlay 로딩 확인
- selftest 통과 또는 실패 원인 이슈화
- 보드별 핵심 노트북 1개 이상 실행

## 관련 문서

- `board-setup-and-test-guide.md`
- `board-variants-notes.md`
- `common-troubleshooting.md`

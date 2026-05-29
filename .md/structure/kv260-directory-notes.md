# kv260 Directory Notes

## 목적

`kv260/` 디렉터리의 Python 패키지 구조, base overlay Vivado 자산, 노트북 배치를 정리한다.

## 범위

- 포함: `kv260/` 및 `kv260/base/` 하위 구조, 빌드/배포 연결
- 제외: 노트북 셀 단위 사용법

## 관련 파일

- `kv260/__init__.py`
- `kv260/base/` (Vivado + notebooks + dts)
- `setup.py` (패키징 진입점)

## 디렉터리 구조

```
kv260/
├── __init__.py              # BaseOverlay export, version 2.7.0
└── base/
    ├── base.py              # BaseOverlay class (PMOD alias)
    ├── base.tcl             # Vivado block design 생성
    ├── build_bitstream.tcl  # bitstream 빌드
    ├── check_timing.tcl
    ├── Makefile             # make → Vivado 빌드
    ├── README.md
    ├── LICENSE              # binary license
    ├── vivado/constraints/base.xdc
    ├── dts/
    │   ├── base.dtsi
    │   └── Makefile         # base.dtbo 컴파일
    └── notebooks/
        ├── video/           # 4 notebooks
        └── microblaze/      # 3 notebooks
```

## Python 패키지 계층

```python
# kv260/__init__.py
from .base.base import BaseOverlay
__version__ = '2.7.0'
```

```python
# kv260/base/base.py
class BaseOverlay(pynq.Overlay):
    # PMOD0/PMODA alias, iop_pmod0 MicroBlaze 설정
```

`setup.py` entry_points:

- `pynq.overlays`: `kv260 = kv260`
- `pynq.notebooks`: `kv260 = kv260.notebooks`

## overlay 배포 방식 (이중 경로)

### 1) pip 설치 시 (setup.py)

- Xilinx CDN에서 `kv260_base_2.7.zip` 다운로드 (md5 검증)
- `kv260/base/dts/` → `base.dtbo` 컴파일
- notebooks → `kv260/notebooks/`로 복사

### 2) Vivado 재빌드 (kv260/base)

```bash
cd kv260/base
make
```

- `base.tcl` → Vivado 프로젝트 생성 (board: `*:kv260:*`)
- `build_bitstream.tcl` → `base.bit`, `base.hwh` 생성
- 산출물이 `kv260/base/` 디렉터리에 복사됨

> pre-built binary는 OSI open source license 대상이 아님 (README LICENSE 참조)

## 노트북 분류

| 디렉터리 | 개수 | PL 의존 | 주변장치 |
|----------|------|---------|----------|
| `video/` | 4 | 높음 | DP/HDMI, Pcam, USB cam |
| `microblaze/` | 3 | 높음 | PMOD (선택) |

## RFSoC-PYNQ `boards/<BOARD>/base`와의 비교

| 항목 | kv260/base | RFSoC boards/base |
|------|------------|-------------------|
| Tcl/Makefile | ✓ | ✓ |
| petalinux_bsp | ✗ | ✓ |
| .spec (sdbuild) | ✗ | ✓ |
| overlay 배포 | pip + CDN zip | SD image / 직접 배치 |
| 보드 수 | KV260 only | RFSoC4x2, ZCU208, ZCU111 |

## 빌드 연계

```
setup.py (pip install .)
  ├─ download_overlay()  → kv260_base_2.7.zip
  ├─ compile_dtbo()      → kv260/base/dts/
  └─ copy_notebooks()    → kv260/base/notebooks/

kv260/base/Makefile (optional rebuild)
  └─ base.tcl → build_bitstream.tcl → base.bit/hwh
```

## 주의 포인트

- KR260/KD240용 로컬 overlay 소스 **없음** — 외부 pip/git만 사용
- `BOARD` 환경변수 없이 `setup.py` 실행 불가
- binary overlay md5 불일치 시 ImportWarning 발생

## 관련 문서

- `setup-py-notes.md`
- `guides/sw-setup-deep-dive.md`
- `repository-overview.md`

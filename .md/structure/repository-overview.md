# Repository Overview

## 목적

저장소의 핵심 축인 `install.sh`, `kv260/`, `dts/`, `setup.py`가 어떻게 연결되어 동작하는지 상위 관점에서 정리한다.

## 범위

- 포함: 디렉터리 역할, 설치/패키징 연결 관계, RFSoC-PYNQ 대비 차이
- 제외: 단계별 실행 절차 상세 (해당 내용은 `guides/` 문서에서 관리)

## 최상위 디렉터리 구조

```
local_kria_pynq/
├── install.sh          # 보드 on-target PYNQ 설치 오케스트레이터
├── setup.py            # kv260 Python 패키지 (overlay/노트북 배포)
├── MANIFEST.in
├── dts/                # PYNQ device tree overlay (zocl, uio, afi)
├── kv260/              # KV260 base overlay Python 패키지 + Vivado 소스
│   ├── __init__.py
│   └── base/           # overlay 빌드/노트북/dts
├── pynq/               # submodule (Xilinx/PYNQ @ a056b84, 현재 empty)
├── README.md
└── .md/                # 분석 문서 (로컬)
```

## 핵심 구조 요약

| 경로 | 역할 |
|------|------|
| `install.sh` | Ubuntu 22.04에서 PYNQ 전체 스택 설치 |
| `setup.py` | KV260 base overlay pip 패키지 빌드/배포 |
| `dts/` | 런타임 device tree overlay (zocl/uio) |
| `kv260/` | 유일한 로컬 보드 패키지 + Vivado 재빌드 자산 |
| `pynq/` | submodule — sdbuild 패키지 스크립트 참조용 |

## `pynq/` submodule 정의

- URL: `https://github.com/Xilinx/PYNQ.git`
- 고정 커밋: `a056b8455f80a145839177102288e1f1d2b8ebe3`
- `shallow = true`
- `install.sh` 실행 시 submodule init/update 또는 v3.0.1 shallow clone
- **용도**: SD image 전체 빌드가 아니라, sdbuild 하위 패키지 스크립트 참조
  - `python_packages_jammy`, `jupyter`, `libsds`, `clear_pl_statefile`

## RFSoC-PYNQ와의 구조 비교

| 관점 | Kria-PYNQ | RFSoC-PYNQ |
|------|-----------|------------|
| 진입점 | `install.sh` (on-board) | 루트 `Makefile` (host build) |
| 보드 자산 | `kv260/` 단일 패키지 | `boards/<BOARD>/` 다중 |
| SD 이미지 | 외부 Ubuntu 이미지 | `pynq/sdbuild` 빌드 |
| BSP/Petalinux | 없음 | `petalinux_bsp/` |
| overlay 배포 | pip + Xilinx CDN download | 빌드 산출물 직접 포함 |

## 설치/패키징 흐름

```
[Ubuntu 22.04 SD boot]
        │
        ▼
  install.sh -b KV260
        │
        ├─► pynq submodule → venv/jupyter/libsds 스크립트
        ├─► dts/ → pynq.dtbo + insert_dtbo.py
        ├─► pip install .  → setup.py
        │       ├─ download kv260_base_2.7.zip
        │       ├─ compile base.dtbo
        │       └─ copy notebooks
        ├─► pip: helloworld, composable, dpu, peripherals
        └─► selftest.sh 생성
```

## 외부 의존 저장소 (로컬 미포함)

| 패키지 | 저장소 | 대상 보드 |
|--------|--------|-----------|
| PYNQ | Xilinx/PYNQ | 공통 |
| Composable Pipeline | Xilinx/PYNQ_Composable_Pipeline | KV260 |
| DPU-PYNQ | pypi `pynq-dpu` | KV260, KR260 |
| DPU-PYNQ (fork) | MakarenaLabs/DPU-PYNQ | KD240 |
| Peripherals | Xilinx/PYNQ_Peripherals | KV260 |
| HelloWorld | pypi `pynq_helloworld` | KV260, KR260 |

## 문서 경계

- 절차/운영: `guides/`
- 구조/코드 맥락: `structure/` (이 문서 포함)
- 프로젝트 배경: `project-purpose.md`

## 관련 문서

- `install-sh-notes.md`
- `kv260-directory-notes.md`
- `dts-directory-notes.md`
- `setup-py-notes.md`

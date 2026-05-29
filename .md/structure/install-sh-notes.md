# install.sh Notes

## 목적

`install.sh`의 단계별 동작, 보드 분기, selftest 생성 로직을 구조 관점에서 정리한다.

## 범위

- 포함: 스크립트 단계, 환경변수, 보드별 분기, 외부 의존
- 제외: 실행 절차 (→ `guides/board-setup-and-test-guide.md`)

## 관련 파일

- `install.sh` (354 lines)

## 입력/전제조건

```bash
sudo bash install.sh -b { KV260 | KR260 | KD240 }
```

- root 권한 필수
- Ubuntu 22.04 (`/etc/lsb-release`)
- 네트워크 (wget, git, apt, pip)

## 주요 단계

### Phase 1: 환경 검증

| 단계 | 내용 |
|------|------|
| L29~42 | `-b` 플래그 파싱, 보드명 검증 |
| L49~62 | Ubuntu 22.04 확인 (20.04 거부) |

### Phase 2: 바이너리/소스 준비

| 단계 | 내용 |
|------|------|
| L73~82 | `pynq-v3.0-binaries.tar.gz` wget → `/tmp` |
| L90~99 | `pynq` submodule init/update 또는 v3.0.1 shallow clone |

### Phase 3: 시스템 패키지 + venv

| 단계 | 내용 |
|------|------|
| L102~110 | unattended-upgrades 중지/대기 |
| L113~121 | apt: python3.10-venv, opencv, i2c-tools 등 |
| L124~132 | `pynq/sdbuild/packages/python_packages_jammy` → venv 생성 |
| L135~138 | `/etc/profile.d/pynq_venv.sh` 환경변수 설정 |

환경변수:

- `PYNQ_VENV=/usr/local/share/pynq-venv`
- `PYNQ_JUPYTER_NOTEBOOKS=/root/jupyter_notebooks`
- `BOARD=<선택 보드>`
- `XILINX_XRT=/usr`

### Phase 4: PYNQ pip 설치

| 단계 | 내용 |
|------|------|
| L150~157 | pip constraint (numpy 1.26.4, pynq 3.0.1) |
| L169~178 | jupyter, libsds sdbuild 패키지 |
| L181 | `pip install pynq` |
| L186~191 | gcc-mb, xclbinutil → venv bin |

### Phase 5: Device Tree + 보드 설정

| 단계 | 내용 |
|------|------|
| L194 | `/etc/xocl.txt`에 보드명 기록 |
| L197~204 | `dts/` make → `pynq.dtbo` + `insert_dtbo.py` 배치 |

### Phase 6: 보드별 패키지 (분기)

**KV260** (L212~235):

- `pynq_helloworld`, `pip install .`, composable pipeline, peripherals, `pynq-dpu==2.5`

**KR260** (L237~246):

- `pynq_helloworld`, `pynq-dpu==2.5`

**KD240** (L248~259):

- `MakarenaLabs/DPU-PYNQ` editable install
- kd240_notebooks → jupyter_notebooks

### Phase 7: 노트북 배포/패치

| 단계 | 내용 |
|------|------|
| L262 | `pynq-get-notebooks` |
| L265 | common 노트북 복사 |
| L270~288 | 경로/import 패치 (kv260, PMODA) |
| L291 | microblaze rpc.py 경로 패치 |
| L297~306 | 보드별 불필요 노트북 삭제 |

### Phase 8: 서비스 + selftest

| 단계 | 내용 |
|------|------|
| L314 | jupyter.service 시작 |
| L317~319 | clear_pl_statefile systemd 등록 |
| L328~351 | pytest 설치 + `selftest.sh` 동적 생성 |

## selftest.sh 생성 로직

**KV260**:

```bash
pushd .../pynq_composable/runtime_tests
python3 -m pytest test_apps.py
python3 -m pytest test_composable.py
python3 -m pytest test_mmio_partial_bitstreams.py
popd
python3 -m pytest .../pynq_dpu/tests
```

**KR260**: DPU tests only

**KD240**: selftest 생성 **없음**

## install.sh가 생성/수정하는 시스템 파일

| 경로 | 용도 |
|------|------|
| `/etc/profile.d/pynq_venv.sh` | venv, BOARD, XRT, dtbo insert |
| `/etc/xocl.txt` | XRT platform name |
| `/usr/local/share/pynq-venv/pynq-dts/` | dtbo + insert script |
| `./selftest.sh` | 설치 디렉터리에 생성 |

## 관련 문서

- `guides/board-setup-and-test-guide.md`
- `guides/board-variants-notes.md`
- `setup-py-notes.md`

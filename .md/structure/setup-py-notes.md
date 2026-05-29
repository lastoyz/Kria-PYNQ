# setup.py Notes

## 목적

루트 `setup.py`가 KV260 base overlay Python 패키지를 어떻게 빌드/배포하는지 정리한다.

## 범위

- 포함: setup.py 함수, entry_points, overlay 다운로드, dtbo/notebook 처리
- 제외: pip install 실행 절차 (→ `install-sh-notes.md`)

## 관련 파일

- `setup.py`
- `MANIFEST.in`
- `kv260/__init__.py`

## 전제조건

```bash
export BOARD=KV260
pip install .
```

`BOARD` 환경변수 필수 — `os.environ["BOARD"]` (L26)

## 핵심 상수

| 변수 | 값 |
|------|-----|
| `module_name` | `"kv260"` |
| overlay URL | `kv260_base_2.7.zip` (Xilinx CDN) |
| overlay md5 | `b2a97221b04aead529a6a862d9d691ff` |
| package version | `2.7.0` (from `kv260/__init__.py`) |

## 주요 함수

### download_overlay(board, overlay_dest)

- BOARD가 `overlay` dict에 있을 때만 동작 (현재 KV260만)
- URL에서 zip 다운로드 → md5 검증 → unpack
- checksum 불일치 시 ImportWarning + overlay 미배포

### compile_dtbo(src_path, dst_path)

- `kv260/base/dts/`에서 `make` 실행
- `base.dtbo` → 패키지 data로 복사

### copy_notebooks(board_folder, module_name)

- `kv260/base/notebooks/*` → `kv260/notebooks/*`로 복사
- setup 시점에 노트북을 Python 패키지 내부로 이동

### extend_package(path)

- walk 결과를 `package_data`에 등록

## setup() 호출 흐름

```python
copy_notebooks("kv260/base", module_name)
download_overlay(board, module_name)
compile_dtbo("kv260/base/dts/", module_name)
extend_package(module_name)

setup(
    name="kv260",
    version="2.7.0",
    packages=find_packages(),
    entry_points={
        "pynq.notebooks": ["kv260 = kv260.notebooks"],
        "pynq.overlays": ["kv260 = kv260"],
    },
    install_requires=["pynq>=2.7.0"],
    cmdclass={"build_py": build_py},  # pynqutils.setup_utils
)
```

## MANIFEST.in

```
include pyproject.toml
recursive-include kv260 *
```

## install.sh와의 연결

KV260 설치 분기 (L218~219):

```bash
python3 -m pip install .
```

이 시점에:

- `$BOARD=KV260` (profile.d에 설정됨)
- venv 활성화 상태
- overlay zip 다운로드 + dtbo 컴파일 + notebooks 패키징 일괄 수행

## RFSoC-PYNQ boards packages와의 비교

| 항목 | Kria setup.py | RFSoC board packages |
|------|---------------|----------------------|
| overlay 소스 | CDN pre-built + 선택적 Vivado rebuild | 로컬 Vivado 빌드 중심 |
| sdbuild stage4 | 미사용 | `.spec`으로 stage4 정의 |
| 보드 수 | KV260 only | 보드별 packages/ |

## 주의 포인트

- KR260/KD240에는 해당 setup.py 분기 없음
- overlay URL/md5 하드코딩 — Xilinx CDN 변경 시 수정 필요
- `pynq>=2.7.0` vs install.sh constraint `pynq==3.0.1` — install.sh가 pip constraint로 실제 버전 고정

## 관련 문서

- `kv260-directory-notes.md`
- `install-sh-notes.md`
- `repository-overview.md`

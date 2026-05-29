# Kria-PYNQ Board Setup and Test Guide (SSOT)

## 문서 목적

`Kria-PYNQ` 보드를 실무 관점에서 빠르게 셋업하고, PYNQ 설치 및 selftest/노트북 기반 기능 테스트까지 완료하는 표준 절차를 제공한다.
이 문서는 `guides` 영역의 실행 순서를 정의하는 단일 기준(SSOT)으로 사용한다.

## 대상 보드

- `KV260` (Vision AI Starter Kit)
- `KR260` (Robotic Starter Kit)
- `KD240` (Drives Starter Kit)

## 전체 흐름

1. Bring-up (Ubuntu SD 이미지 부팅/접속 확인)
2. PYNQ 설치 (`install.sh`)
3. Jupyter/overlay 스모크 테스트
4. Selftest 또는 노트북 기반 기능 테스트
5. (선택) KV260 base overlay Vivado 재빌드
6. 결과 기록 (pass/fail, 로그, 이슈)

## 1) Bring-up

참조: `board-bringup-quickstart.md`

- 공식 Canonical Xilinx Ubuntu 22.04 SD 이미지 준비 및 부팅
- UART/LED/네트워크로 정상 부팅 확인
- SSH 또는 브라우저 접속 확인
- Ubuntu 22.04인지 확인 (`/etc/lsb-release`)

완료 기준:

- [ ] 보드 정상 부팅 (Ubuntu 22.04)
- [ ] 네트워크/SSH 접속 성공

## 2) PYNQ 설치

참조: `structure/install-sh-notes.md`

보드에서 저장소 clone 후 설치:

```bash
git clone https://github.com/lastoyz/Kria-PYNQ.git
cd Kria-PYNQ/
sudo bash install.sh -b KV260   # 또는 KR260, KD240
```

설치 스크립트가 수행하는 주요 작업:

- Ubuntu 22.04 호환성 검사
- `pynq-v3.0-binaries.tar.gz` 다운로드 (gcc-mb, xclbinutil 등)
- `pynq` submodule clone (v3.0.1 shallow)
- Python venv 생성 (`/usr/local/share/pynq-venv`)
- PYNQ 3.0.1 및 보드별 pip 패키지 설치
- device tree overlay (`pynq.dtbo`) 컴파일/등록
- Jupyter 서비스 시작
- `selftest.sh` 생성 (KV260/KR260)

예상 소요: **약 25분**

완료 기준:

- [ ] `install.sh` 오류 없이 완료
- [ ] 설치 완료 메시지 및 IP/hostname 출력 확인

## 3) Jupyter 및 Overlay 스모크 테스트

JupyterLab 접속:

- URL: `<ip_address>:9090/lab` 또는 `kria:9090/lab`
- 비밀번호: `xilinx`

Python 셀에서 overlay 로딩 확인 (KV260 예):

```python
from kv260 import BaseOverlay
ol = BaseOverlay("base.bit")
```

완료 기준:

- [ ] JupyterLab 접속 성공
- [ ] Python 셀 실행 성공
- [ ] overlay 로딩 성공 (해당 보드)

## 4) Selftest / 기능 테스트

참조: `sw-setup-deep-dive.md`, `board-variants-notes.md`, `kv260-board-test-checklist.md`, `kv260-mipi-camera-test-guide.md`, `kv260-peripherals-modules.md`

### Selftest 실행

설치 디렉터리에서:

```bash
sudo ./selftest.sh
```

| 보드 | selftest 내용 | 필수 주변장치 |
|------|---------------|---------------|
| KV260 | composable runtime tests + DPU tests | **Track A**: HDMI/DP + USB 웹캠 (`test_apps`); **Track B**: AR1335@J7 → `kv260-mipi-camera-test-guide.md` |
| KR260 | DPU tests only | (README 미기재) |
| KD240 | **생성 안 됨** | — |

### 노트북 테스트

KV260 로컬 포함 노트북 (`kv260/base/notebooks/`):

- `video/`: DisplayPort, MIPI, OpenCV 웹캠 예제 (4개)
- `microblaze/`: MicroBlaze IOP 예제 (3개)

pip으로 추가 설치되는 노트북:

- `pynq_helloworld`, `pynq-dpu`, `pynq_composable`, `pynq_peripherals` (KV260)
- `pynq-get-notebooks`로 common 노트북 배포

완료 기준:

- [ ] selftest 통과 (해당 보드)
- [ ] 핵심 노트북 1개 이상 실행 확인
- [ ] 실패 항목 이슈 등록

## 5) KV260 Base Overlay 재빌드 (선택)

참조: `structure/kv260-directory-notes.md`

pre-built binary 대신 직접 빌드할 때:

```bash
cd kv260/base
make
```

산출물: `base.bit`, `base.hwh`

완료 기준:

- [ ] `.bit` / `.hwh` 생성 확인
- [ ] 재빌드 overlay로 로딩 테스트

## 트러블슈팅

공통 이슈 대응은 `common-troubleshooting.md`를 우선 참고한다.

## 보드별 적용 주의사항

보드별 설치 패키지/selftest/노트북 차이는 `board-variants-notes.md`를 우선 참고한다.

## 최종 Go/No-Go 게이트

- [ ] Ubuntu 22.04 부팅/접속 안정
- [ ] `install.sh` 설치 완료
- [ ] JupyterLab 접속 및 overlay 로딩 성공
- [ ] selftest 또는 핵심 노트북 테스트 통과
- [ ] (KV260) 주변장치 연결 조건 충족 또는 테스트 범위 조정
- [ ] 결과 및 로그 기록 완료

## 관련 소스

- `README.md`
- `install.sh`
- `setup.py`

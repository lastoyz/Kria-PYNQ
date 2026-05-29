# Common Troubleshooting

## 목적

Kria-PYNQ 설치/테스트 중 자주 발생하는 문제와 대응 방법을 정리한다.

## 범위

- Ubuntu 호환성, install.sh, Jupyter, overlay, selftest 관련
- 제외: Vivado 빌드 상세 (→ `structure/kv260-directory-notes.md`)

## Ubuntu 버전 불일치

**증상**: `install.sh` 시작 시 즉시 종료

```
This version of Kria-PYNQ is not compatible with Ubuntu 20.04
```

**대응**:

- Ubuntu 22.04 이미지 사용, 또는
- `git checkout tags/v1.0` (구버전 Kria-PYNQ)

## pynq binaries 다운로드 실패

**증상**: `Could not extract pynq binaries`

**원인**: `pynq-v3.0-binaries.tar.gz` 다운로드가 HTML 오류 페이지로 저장됨

**대응**:

- 네트워크 연결 확인
- `/tmp/pynq-v3.0-binaries.tar.gz` 파일 타입 확인 (`file` 명령)
- Xilinx 다운로드 URL 접근 가능 여부 확인

## apt 잠금 / unattended-upgrades

**증상**: `install.sh`가 apt 단계에서 hang

**원인**: Ubuntu 자동 업데이트가 dpkg lock 점유

**대응**:

- `install.sh`가 자동으로 `unattended-upgrades` 중지 및 대기 (L102~110)
- 수동: `systemctl stop unattended-upgrades.service` 후 재시도

## venv 진입 실패

**증상**: `ERROR: could not enter the Pynq venv`

**대응**:

- `/usr/local/share/pynq-venv` 존재 확인
- `pynq/sdbuild/packages/python_packages_jammy` pre.sh/qemu.sh 로그 확인
- 디스크 공간 확인

## Jupyter 접속 불가

**증상**: `:9090/lab` 연결 거부

**대응**:

```bash
systemctl status jupyter.service
source /etc/profile.d/pynq_venv.sh
ip addr show eth0
```

- 방화벽/네트워크 케이블 확인
- 비밀번호: `xilinx`

## overlay 로딩 실패

**증상**: `BaseOverlay("base.bit")` 오류

**대응**:

- KV260: `pip show kv260` — overlay 파일 위치 확인
- device tree overlay 적용 확인:

```bash
ls /sys/kernel/config/device-tree/overlays/pynq
```

- `/etc/profile.d/pynq_venv.sh` sourcing 확인 (`insert_dtbo.py` 실행)

## selftest 실패 (KV260 test_apps)

**증상**: `test_apps.py` pytest fail

**원인**: OpenCV USB 웹캠 또는 `mountains.mp4` 필요 (`VSource.OpenCV`)

**대응 (AR1335 IAS 프로젝트)**:

- `test_apps.py` **skip** — MIPI 검증은 Smartcam Phase (`kv260-mipi-camera-test-guide.md`)
- composable/DPU pytest만 수행

## AR1335 Smartcam MIPI 실패

**증상**: `smartcam --mipi` 오류 또는 검은 화면

**대응**:

- J7 IAS FFC 체결 확인 (RPi 포트 아님)
- `ls /lib/firmware/ap1302_ar1335_single_fw.bin`
- `sudo xmutil loadapp kv260-smartcam`
- `dmesg | grep -i ap1302`

## PYNQ base.mipi 실패 (AR1335 장착 시)

**증상**: `base.mipi.readframe()` hang/오류

**원인**: Kria-PYNQ base overlay `base.mipi`는 Pcam 5C / RPi MIPI 경로용. J7 AR1335와 **다른 PL 경로**.

**대응**: Smartcam으로 AR1335 검증. `mipi_to_displayport.ipynb` fail은 AR1335-only 환경에서 예상 가능.

## KD240 selftest 없음

**증상**: `selftest.sh` 파일 없음

**원인**: `install.sh`에 KD240 selftest 생성 분기 없음 (의도적 또는 미구현)

**대응**:

- `/root/jupyter_notebooks/kd240_notebooks/` 노트북 수동 실행
- DPU overlay 수동 로딩 테스트

## pip constraint 충돌

**증상**: 패키지 버전 충돌 during install

**참고**: install.sh가 `/tmp/pynq_3.0.1_constraints.txt`로 numpy/pynq 버전 고정

**대응**:

- `$PIP_CONSTRAINT` 환경변수 확인
- 수동 pip install 시 constraint 파일 참조

## 관련 문서

- `board-setup-and-test-guide.md`
- `structure/install-sh-notes.md`

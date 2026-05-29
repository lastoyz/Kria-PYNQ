# 프로젝트 목적 정리

## 한 줄 요약

`Kria-PYNQ`는 AMD Kria K26 SOM 기반 스타터 키트(KV260, KR260, KD240)에 **공식 Ubuntu SD 이미지 위에 PYNQ 런타임을 추가 설치**하기 위한 스크립트/패키지 저장소다.

## 저장소에서 확인되는 목적

- Kria 보드용 PYNQ 환경 구축
  - Python venv, JupyterLab, PYNQ 3.0.1, device tree overlay, XRT 관련 도구 설치
- 보드별 overlay/노트북 패키지 배포
  - KV260: base overlay, composable pipeline, DPU, peripherals 등
  - KR260: HelloWorld, DPU
  - KD240: MakarenaLabs DPU-PYNQ fork 기반 DPU
- overlay 소스/재빌드 자산 제공 (KV260 base만 로컬 포함)

## RFSoC-PYNQ와의 핵심 차이

| 항목           | Kria-PYNQ                          | RFSoC-PYNQ                           |
| -------------- | ---------------------------------- | ------------------------------------ |
| PS 이미지      | Canonical Ubuntu 공식 이미지 사용  | `pynq/sdbuild`로 SD 이미지 직접 빌드 |
| 설치 방식      | 보드에서 `install.sh` 실행         | 호스트에서 `make BOARD=...`          |
| 보드 자산 위치 | `kv260/` Python 패키지             | `boards/<BOARD>/`                    |
| Vivado 빌드    | KV260 base만 선택적 (`kv260/base`) | 보드별 base overlay 빌드가 핵심 흐름 |

## README 기반 프로젝트 개요 요약

- 대상 OS: **Ubuntu 22.04 LTS** (20.04는 tag `v1.0` 사용)
- 설치 명령: `sudo bash install.sh -b { KV260 | KR260 | KD240 }`
- 설치 시간: 약 25분
- Jupyter 접속: `<ip>:9090/lab`, 비밀번호 `xilinx`
- PYNQ submodule: `Xilinx/PYNQ` @ `a056b84` (설치 시 shallow clone)

## 판단: 분석/테스트 우선순위

Kria-PYNQ는 RFSoC-PYNQ와 달리 **SD 이미지 빌드가 아니라 on-board 설치 + selftest/노트북 검증**이 중심이다.

권장 순서:

1. 공식 Ubuntu 이미지로 bring-up
2. `install.sh`로 PYNQ 설치
3. Jupyter 접속 및 overlay 로딩 스모크 테스트
4. `selftest.sh` (보드별) 또는 노트북 기반 기능 테스트
5. 필요 시 KV260 base overlay Vivado 재빌드 (`kv260/base`)

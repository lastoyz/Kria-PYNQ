# Board Bring-up Quickstart

## 목적

Kria 보드를 처음 켜는 시점부터 Ubuntu 정상 부팅 및 네트워크/SSH 접속 확인까지, PYNQ 설치 이전 bring-up 절차를 빠르게 점검한다.

## 대상

- `KV260`
- `KR260`
- `KD240`

## 준비물

- 전원 어댑터, USB/UART 케이블, Ethernet 케이블
- SD 카드 (Canonical Xilinx Ubuntu 22.04 이미지)
- 호스트 PC (브라우저 + SSH 클라이언트)

## 빠른 절차

### 1) SD 카드 이미지 준비

보드별 공식 Getting Started 문서를 따라 Ubuntu SD 이미지를 준비한다.

- [KV260 Ubuntu Setup](https://www.xilinx.com/products/som/kria/kv260-vision-starter-kit/kv260-getting-started-ubuntu/setting-up-the-sd-card-image.html)
- [KR260 Getting Started](https://www.xilinx.com/products/som/kria/kr260-robotics-starter-kit/kr260-getting-started/setting-up-the-sd-card-image.html)
- [KD240 Getting Started](https://www.xilinx.com/products/som/kria/kd240-drives-starter-kit/kd240-getting-started/getting-started.html)

이미지 다운로드: [Canonical Xilinx Ubuntu Images](https://ubuntu.com/download/xilinx)

### 2) 부트 펌웨어 확인 (Ubuntu 22.04)

Ubuntu 22.04 사용 시 부트 펌웨어 버전 불일치로 부팅 실패할 수 있다.
2022.1 이상 부트 펌웨어 권장.

- 참조: [Kria K26 SOM Wiki - Ubuntu LTS](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/1641152513/Kria+K26+SOM#Ubuntu-LTS)

### 3) 하드웨어 연결 및 부팅

- SD 카드 삽입, 전원/UART/Ethernet 연결
- 시리얼 콘솔에서 부팅 로그 확인
- 로그인 프롬프트까지 대기

### 4) OS 버전 확인

```bash
cat /etc/lsb-release
```

기대값: `DISTRIB_RELEASE=22.04`

> Ubuntu 20.04에서는 Kria-PYNQ v3.0이 거부된다. tag `v1.0` checkout 필요.

### 5) 네트워크/SSH 확인

```bash
ip addr show eth0
hostname
```

SSH 접속 또는 직접 콘솔에서 후속 작업 가능 여부 확인.

## 완료 기준

- Ubuntu 22.04 정상 부팅
- 네트워크 링크 업 또는 접속 가능
- `install.sh` 실행 준비 완료

## 다음 단계

- PYNQ 설치: `board-setup-and-test-guide.md` §2
- 기능 테스트: `sw-setup-deep-dive.md`

## 관련 소스

- `README.md` § Installation step 1

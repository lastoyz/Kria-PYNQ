# dts Directory Notes

## 목적

`dts/` 디렉터리의 device tree overlay 역할, 빌드 방법, 런타임 삽입 메커니즘을 정리한다.

## 범위

- 포함: `pynq.dts`, `Makefile`, `insert_dtbo.py`
- 제외: KV260 base overlay DT (`kv260/base/dts/` — 별도)

## 관련 파일

```
dts/
├── pynq.dts          # PYNQ 런타임 DT overlay 소스
├── Makefile          # dtc → pynq.dtbo
└── insert_dtbo.py    # 부팅 시 overlay 삽입
```

## pynq.dts fragment 구성

| fragment | target | 내용 |
|----------|--------|------|
| @1 | `&amba` | `afi0` — AFI FPGA 인터페이스 |
| @2 | `&amba` | `zocl` — ZOCL/XRT DRM (`xlnx,zocl`) |
| @3 | `&amba` | `fabric` — generic-uio @ 0xA0000000 |

목적: PYNQ overlay 로딩에 필요한 **zocl 드라이버** 및 **UIO fabric** 노드를 device tree에 추가.

## 빌드

```bash
cd dts/
make
# → pynq.dtbo
```

Makefile:

```makefile
dtc -I dts -O dtb -o pynq.dtbo pynq.dts -q
```

## install.sh에서의 처리

1. `dts/` make → `pynq.dtbo` 생성
2. `/usr/local/share/pynq-venv/pynq-dts/`에 `pynq.dtbo` + `insert_dtbo.py` 복사
3. `/etc/profile.d/pynq_venv.sh`에 insert 명령 추가:

```bash
python3 /usr/local/share/pynq-venv/pynq-dts/insert_dtbo.py
```

## insert_dtbo.py 동작

- sysfs 경로: `/sys/kernel/config/device-tree/overlays/pynq`
- 이미 삽입되어 있으면 skip
- `pynq.DeviceTreeSegment(path + dtbo).insert()` 호출

## kv260/base/dts/와의 관계

| 경로 | 용도 | 빌드 시점 |
|------|------|-----------|
| `dts/pynq.dts` | PYNQ 공통 (zocl/uio) | install.sh |
| `kv260/base/dts/base.dtsi` | KV260 base overlay HW | setup.py pip install |

두 overlay는 **서로 다른 목적** — 공통 PYNQ 런타임 vs KV260 PL 설계.

## RFSoC-PYNQ 대비

RFSoC-PYNQ는 petalinux BSP/device-tree를 sdbuild로 통합 빌드한다.
Kria-PYNQ는 Ubuntu 공식 이미지 + **런타임 DTBO 삽입** 방식으로 PYNQ 호환성을 확보한다.

## 관련 문서

- `install-sh-notes.md`
- `kv260-directory-notes.md`
- `guides/common-troubleshooting.md` (overlay 로딩 실패)

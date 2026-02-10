# Linux 6.17 가상화 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17 가상화는 KVM 디바이스 Posted IRQ 대규모 개편(IOMMU 통합), vhost-net VIRTIO_F_IN_ORDER 지원, virtio GSO over UDP 터널, vsock SIOCINQ ioctl 도입이 핵심이다. 6.16의 Intel TDX 초기화에 이어, 6.17에서는 KVM의 인터럽트 처리 성능 최적화에 집중했다.

---

## KVM 디바이스 Posted IRQ 개편

### 변경사항
- **디바이스 Posted IRQ 대규모 개편** — IOMMU 통합을 포함한 종합적 인터럽트 리매핑 재구현 (kernelnewbies.org)
- 디바이스에서 게스트 VM으로의 인터럽트 전달 경로 최적화
- **irqfd 전역 고유 등록** — KVM irqfd의 전역적으로 고유한 등록 보장

### 배경 및 영향
- Posted IRQ는 하드웨어가 직접 게스트 VM에 인터럽트를 전달하여 VMM(Virtual Machine Monitor) 개입 없이 인터럽트 처리하는 기술
- IOMMU와의 통합으로 디바이스 패스스루(VFIO) 환경에서의 인터럽트 처리 효율 향상
- 서버 가상화, 클라우드 VM, SR-IOV 환경에서 인터럽트 지연 감소
- 6.16의 TDX 기밀 VM과 결합 시 기밀 VM에서도 효율적 인터럽트 처리 가능

---

## Vhost-net VIRTIO_F_IN_ORDER

### 변경사항
- **VIRTIO_F_IN_ORDER 지원** — vhost-net에서 virtio 디스크립터 완료 순서 보장 기능 지원 (kernelnewbies.org)

### 배경 및 영향
- VIRTIO_F_IN_ORDER는 디스크립터가 제출 순서대로 완료됨을 보장하는 virtio 기능 비트
- 순서 보장으로 게스트 드라이버의 완료 처리 로직 단순화 및 배치 최적화 가능
- 네트워크 패킷 순서에 민감한 워크로드에서 성능 향상

---

## Virtio GSO over UDP 터널

### 변경사항
- **GSO over UDP 터널** — virtio에서 UDP 터널을 통한 Generic Segmentation Offload 지원 (kernelnewbies.org)

### 배경 및 영향
- VXLAN, Geneve 등 UDP 기반 오버레이 네트워크에서 GSO를 활용한 대용량 패킷 전송
- 클라우드 네트워크, 컨테이너 오버레이 네트워크의 처리량 향상

---

## Vsock SIOCINQ ioctl

### 변경사항
- **SIOCINQ ioctl 지원** — vsock(VM 소켓)에서 수신 버퍼의 데이터 크기를 조회하는 ioctl 도입 (kernelnewbies.org)

### 배경 및 영향
- 기존 TCP/UDP 소켓에서 사용되는 SIOCINQ(FIONREAD)를 vsock에서도 사용 가능
- 호스트-게스트 통신에서 유저스페이스 애플리케이션의 수신 버퍼 관리 편의성 향상
- Android의 pKVM(Protected KVM) 호스트-게스트 통신에 활용 가능성

---

## RISC-V KVM 개선

### 변경사항
- **링 기반 더티 메모리 추적** — 효율적 더티 페이지 추적 메커니즘
- **perf kvm stat 인터럽트 이벤트** — 인터럽트 이벤트 보고
- **불법 명령 트랩 위임** — VS-mode로의 트랩 위임
- **중첩 가상화 MMU 개선** — 중첩 가상화를 위한 MMU 강화

### 배경 및 영향
- RISC-V KVM이 6.16에서 실험적 → 정식으로 전환된 후, 6.17에서 기능 성숙화
- 링 기반 더티 메모리 추적은 라이브 마이그레이션 효율 향상
- 중첩 가상화 MMU는 RISC-V 하이퍼바이저-in-하이퍼바이저 시나리오 지원

---

## 이전 버전과의 연속성

- **KVM 인터럽트 경로**: 6.16 TDX 기밀 VM 초기화 → 6.17 Posted IRQ 대규모 개편으로 인터럽트 성능 최적화
- **virtio 기능**: 6.16 virtio_rtc 새 디바이스 → 6.17 VIRTIO_F_IN_ORDER, GSO over UDP
- **RISC-V KVM**: 6.16 실험적 → 정식 전환 → 6.17 더티 추적, 중첩 가상화 MMU
- **기밀 컴퓨팅**: 6.16 Intel TDX → 6.17 Posted IRQ IOMMU 통합 (기밀 VM의 I/O 성능 기반)

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [LWN.net 6.17 merge window](https://lwn.net/Articles/1031713/)

# Linux 6.17 보안 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17 보안은 CPU 취약점 완화를 위한 공격 벡터 제어(AVC) 프레임워크 도입, 코어 덤프 소켓 프로토콜 확장, CRC 코드 대규모 재작업 및 SHA-1/SHA-2 새 API가 핵심이다. 6.16의 SELinux 캐싱/와일드카드에 이어 6.17에서는 하드웨어 보안 완화 관리 체계를 혁신했다.

---

## 공격 벡터 제어 (Attack Vector Controls, AVC)

### 변경사항
- **CPU 취약점 완화 통합 프레임워크** — Spectre, Meltdown 등 CPU 하드웨어 취약점의 완화(mitigation) 옵션을 공격 벡터 단위로 관리하는 새 커널 커맨드라인 인터페이스 도입 (kernelnewbies.org, LWN.net)
- 새로운 CPU 취약점 발견 시 관리자가 개별 파라미터를 재설정할 필요 없이, 공격 벡터 기반으로 자동 적용
- 기존의 개별 완화 파라미터(mitigations=off, spectre_v2=off 등)를 대체하는 상위 수준 제어

### 배경 및 영향
- 지금까지 새로운 CPU 취약점이 발견될 때마다 별도의 커널 파라미터가 추가되어 관리 복잡성 증가
- AVC는 "어떤 공격 벡터를 방어할 것인가"라는 상위 수준 정책으로, 새 취약점 대응 시 자동으로 적절한 완화 적용
- 클라우드, 서버, 임베디드 환경에서 보안/성능 트레이드오프의 통일적 관리 가능
- Android SoC의 ARM CPU 취약점 완화에도 유사 프레임워크 적용 가능성 (현재 x86 전용)

---

## 코어 덤프 소켓 확장

### 변경사항
- **코어 덤프 소켓 프로토콜 확장** — 6.16에서 도입된 AF_UNIX 코어 덤프 소켓에 대한 다수의 후속 커밋 (kernelnewbies.org)
- 서버가 개별 코어 덤프에 대해 처리 지시를 지정할 수 있는 프로토콜
- 더 안전하고 세밀한 코어 덤프 관리 가능

### 배경 및 영향
- 기존 core_pattern 기반 코어 덤프 처리는 특권 헬퍼 프로세스가 필요했으나, 소켓 기반 접근으로 권한 분리 강화
- 컨테이너 환경에서 코어 덤프의 보안적 격리 및 선택적 처리 가능
- 6.16 도입 → 6.17 프로토콜 확장으로 실용성 강화

---

## 암호화 코드 재작업

### 변경사항
- **CRC 코드 대규모 재작업** — 커널의 CRC(Cyclic Redundancy Check) 구현 전면 리팩토링
- **SHA-1/SHA-2 새 API** — SHA-1 및 SHA-2 해시 생성을 위한 새로운 커널 API 도입

### 배경 및 영향
- CRC 재작업은 코드 중복 제거, 성능 최적화, 유지보수성 향상을 위한 구조적 변경
- SHA 새 API는 커널 내부에서 해시를 사용하는 다수 서브시스템(dm-verity, IMA, 모듈 서명 등)의 인터페이스 현대화
- 보안 관련 코드의 구조적 품질 향상

---

## Runtime Verification

### 변경사항
- **선형 시간 논리(LTL) 모니터** — 결정론적 오토마타 방식 대신 LTL 기반 런타임 검증 모니터 추가 (kernelnewbies.org)
- 실시간 애플리케이션의 제약 조건을 더 직관적으로 명세 가능

### 배경 및 영향
- LTL은 형식 검증(formal verification)에서 널리 사용되는 명세 언어. 런타임 검증에 LTL 도입은 명세 작성의 편의성 향상
- 자동차, 항공, 산업 제어 등 안전 필수(safety-critical) 시스템의 커널 검증에 활용
- Android Automotive의 기능 안전 요구사항과 관련 가능

---

## 이전 버전과의 연속성

- **보안 완화 관리**: 개별 취약점 파라미터 (기존) → 6.17 공격 벡터 제어(AVC)로 패러다임 전환
- **코어 덤프 보안**: 6.16 AF_UNIX 코어 덤프 소켓 도입 → 6.17 프로토콜 확장
- **SELinux**: 6.16 디렉터리 접근 캐시/와일드카드 → 6.17 (주요 SELinux 변경 없음, AVC에 집중)
- **기밀 컴퓨팅**: 6.16 Intel TDX KVM → 6.17에서는 기밀 컴퓨팅보다 CPU 완화 관리에 초점
- **Runtime Verification**: 6.17에서 LTL 모니터 추가로 검증 도구 확대

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [LWN.net 6.17 release announcement](https://lwn.net/Articles/1038266/)
- [LWN.net 6.17 merge window part 1](https://lwn.net/Articles/1031713/)

# 21. Fast Packet Processing Methods

### Software Packet Switching

- NFV

  - 고속 패킷 프로세싱을 위한 기술 필요

  <img src="images/image-20210303110718151.png" alt="image-20210303110718151" style="zoom:50%;" />



### DPDK

- DPDK(Data Plane Development Kit)

  - 리눅스 운영체제로 동작하는 서버에서 패킷 프로세싱에 리눅스 커널 내부에 존재하는 네트워킹 스택 사용
    - 패킷 단위로 인터럽트를 발생시킴
    - kernel networking stack과 실제 응용 프로그램이 존재하는 user space 사이를 오고가기 때문에 context switching이 많이 발생
    - 그에 따른 메모리 카피가 많이 발생
    - 일반적인 용도의 서버로 사용할 때는 동작할 때 성능에 큰 문제가 없음
    - NFV환경의 VNF를 사용하려면 VNF에 패킷을 빠르게 공급하거나 입출력하기에는 이러한 기존 방식에는 성능에 제약이 있음
  - 고속 패킷 프로세싱을 위한 유저 스페이스의 라이브러리 집합 혹은 DPDK 기반의 네트워크 응용 개발을 위한 프레임워크라고 정의할 수 있음
    - 패킷은 cpu interrupt를 발생시키지 않고 dpdk 전용 user space로 전달
    - 고속 패킷 프로세싱을 위한 CPU affinity, huge page, multi-queue와 같은 다양한 네트워크 커스터마이제이션 기술들을 지원

  ![image-20210303111340321](images/image-20210303111340321.png)

- **DPDK Features**

  - Environment Abstraction Layer(EAL)
    - 하드웨어 환경 추상화
  - Poll Mode Drivers(PMD)
    - 네트워크 I/O를 하기 위한 Direct Memory Access 기술
    - 네트워크 카드로 수신되는 패킷들이 기존 linux 네트워크에 커널 네트워크 스택을 거치지 않고 바로 user space의 dpdk 응용 프로그램이나 네트워크 인터페이스 카드로 전달되도록 하는 핵심적인 역할을 수행하는 컴포넌트
  - Memory Manager
  - Buffer Manager
  - Queue Manager
  - Packet Flow Classification
    - 수신된 패킷의 헤더를 분석해 그룹핑

  <img src="images/image-20210303125526955.png" alt="image-20210303125526955" style="zoom:50%;" />



### Open vSwitch

- Open vSwitch(OVS)
  - NFV 구성을 위해서 서버 가상화 환경에서 가장 일반적이고 많이 사용하는 가상 소프트웨어 스위치
  - VM간의 통신, VM과 외부와의 통신을 위해서 호스트 서버 내에서 기본적인 패킷 스위칭 기느을 수행
  - 관리 프로토콜을 통한 다양한 네트워킹 서비스 및 SDN 제어 매커니즘 지원
    - IPv6, tunneling, NIC bonding, 포트 미러링 등의 다양한 네트워크 기능들을 지원
    - 오픈플로우, OVSDB, NetFlow, sFlow, (R)SPAN, etc
  - Key Components
    - kernel module : 
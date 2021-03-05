# 22. Datapath Composition

### Datapath 구성 방안

#### (1) Original OVS in Host + Virtio in VM

- 일반적인 모드로 동작하는 OVS를 호스트 서버에 가상스위치로써 배치

- VM을 Virtio 기반의 가상 네트워크 인터페이스를 가지도록 구성

  - Virtio : VM의 가상 네트워크 인터페이스 카드를 위한 표준화된 가상 드라이버

- 일반적으로 가장 많이 사용되고 쉽게 구현 가능한 방법

- VM 마이그레이션, 특히 라이브 마이그레이션할때도 구성이 단순해 유리한 방법

  - 하지만 고속 패킷 프로세싱 기술이 사용되지 않기 때문에 VM에서 동작하는 VNF를 위한 속도 측면에서 한계점 존재 

  ![image-20210305110611927](images/image-20210305110611927.png)

#### (2) OVS-DPDK in Host + DPDK PMD in VM

- 가상스위치로써 OVS-DPDK를 사용

- DPDK 라이브러리를 이용해서 DPDK로 구현된 응용을 VM 동작하는 VNF로 구현한 datapath 구성

  <img src="images/image-20210305111114361.png" alt="image-20210305111114361" style="zoom:50%;" />

#### (3) SR-IOV in Host + VF Driver in VM

- 호스트 서버에서 SR-IOV 지원하는 카드를 이용해서 Virtual Function(VF)을 만들고 만든 VF를 VM에 가상 인터페이스에 바로 연결하는 방법

- 가상스위치를 경유하지 않고 패킷 프로세싱을 할 수 있어 패킷 전달 속도 면에서 가장 우수

- 가상스위치가 제공하는 여러 네트워크 기능들을 사용할 수 없음

-  VF과 VM간에 바인딩 정보가 존재하여 VM을 다른 호스트 서버로 마이그레이션할 때 fix된 정보들 때문에 마이그레이션하는데 제약사항이 있음

  <img src="images/image-20210305135933578.png" alt="image-20210305135933578" style="zoom:50%;" />

#### (4) SR-IOV in Host + DPDK in VM

- 가장 높은 수준의 고속 패킷 프로세싱을 요구하는 VNF에 적합한 데이터패스 구성

- 호스트서버에 물리 네트워크 인터페이스 카드를 SR-IOV로 지원하는 카드를 사용해서 VF를 구성

- VF를 VM에 Virtual Interface 혹은 VF Driver에 다이렉트하게 연결하는 SR-IOV 구성

- VM에서는 DPDK PMD thread를 이용해서 수신된 패킷을 DPDK 기반 응용으로 고속으로 패킷 프로세싱하는 구조

- 이러한 수준의 구성을 요구하는 VNF의 예

  - 패킷 헤더 및 페이로드를 인스펙션하는 DPI과 같은 네트워크 기능들이 VNF로 구현되었을 때 적합

  <img src="images/image-20210305140251646.png" alt="image-20210305140251646" style="zoom:50%;" />

#### Datapath Composition Comparison

![image-20210305140759362](images/image-20210305140759362.png)




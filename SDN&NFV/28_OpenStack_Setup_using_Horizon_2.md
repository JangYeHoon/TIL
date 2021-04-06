# 28. OpenStack Setup using Horizon - 2

- 오픈스택에 관리자 입장이 아닌 관리자가 제공한 인프라를 사용하는 tenant 입장에서 오픈스택을 설정하기 때문에 Demo 프로젝트로 변환해서 진행

  ![image-20210406150343583](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210406150343583.png)



## Tenant Network (Demo)

- tenant network란 오픈스택의 사용자들을 위한 Private Network

- 많은 테넌트들이 존재할 수 있는데 각각의 테넌트에 대해서 논리적으로 독립된 네트워크 환경을 제공

  - 내부적으로는 오픈스택 서비스 제공자가 Provider Network를 구축
  - 실제 사용자들은 그런 인프라를 공유하면서 각각 자신들 고유의 address space, network topology 등을 가지는 독립된 네트워크를 구성 가능
  - 구현 방식은 오픈 스택 설치 및 설정시 다양하게 설정 가능
    - 실습에서는 VLAN 기술을 기반으로 오버레이 네트워킹을 통해서 테넌트 네트워크를 구축

- Network - Networks - Create Network

- Subnet

  - 테넌트를 위한 Private Network를 설정하는 것이기 때문에 네트워크 주소는 맘대로 설정
  - 게이트웨이 IP도 임의로 설정할 수 있지만 일반적으로는 할당했던 네트워크 대역의 첫번째 주소를 사용

- Subnet Details

  - 테넌트 네트워크가 구축 이후에 테넌트가 생성하는 VM 인스턴스들에게 IP 대역에 속하는 IP를 자동으로 할당하기 위해 DHCP 활성화
  - Allocation pools
    - 테넌트에 의해서 생성될 VM 인스턴스들이 가질 수 있는 Private IP의 서브넷 어드레스 풀

  <img src="C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210406151325600.png" alt="image-20210406151325600" style="zoom:50%;" />

- 테넌트 네트워크 생성 완료 모습

  - Demo 프로젝트로 생성하지만 Demo 프로젝트로는 이러한 시스템에 있는 네트워크 패널에 정보가 안보임
  - 생성된 정보를 확인하기 위해서는 admin으로 변경 후 확인

  ![image-20210406151508106](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210406151508106.png)



## Virtual Router

- 뉴트론에 L3 에이전트에서 담당하는 기능

- Virtual Router는 컨트롤러 노드의 뉴트론의 L3에 설치

  - 실제는 네트워크 노드에 설치되지만 실습에서는 컨트롤러 노드가 역할을 동시에 수행되어 컨트롤러 노드에 설치

- Virtual Router는 Provider Network(Public Network)와 Tenant Network(Private Network)을 연결해주는 역할을 수행

- Virtual Router에서 NAT 기술 수행

  ![image-20210406152324816](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210406152324816.png)



## Virtual Router 생성 (Demo)

- NAT테이블을 가지고 floating IP와 private IP사이의 변환을 담당

- Neutron L3 agent를 통해 구현

- Network - Routers - Create Router

- External Network

  - provider network 선택

- Select "router1" - interfaces - Add Interface

  - Virtual Router의 private network 쪽 인터페이스를 생성하는 단계
  - tenant1 서브넷 설정

  <img src="C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210406152835141.png" alt="image-20210406152835141" style="zoom:50%;" />



## Key Pairs (Demo)

- 키페어 유닛 생성 단계

- Key Pair가 필요한 이유는 tenant가 VM 인스턴스를 생성했을 때 외부에서 이 인스턴스를 접근하기 위해서 필요한 private key와 public key의 pair

- VM을 실제로 생성하는 단계에서 어떠한 Key Pair를 사용하겠다고 선택을 해야하여 미리 생성

- tenant의 VM 인스턴스에 SSH 접속을 위해 필요

- public key는 Keystone 서비스에 등록

- private key는 생성된 VM 인스턴스에 shell을 통해서 접근하는데 이용될 머신에 저장

- Compute - Access & Security - Key Pairs - Create Key Pairs

  <img src="C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210406153457579.png" alt="image-20210406153457579" style="zoom:50%;" />



## Security Groups (Demo)

- Tenant-level에서의 방화벽 규칙

- 각각의 VM 인스턴스들은 여러 개의 Security Group 인스턴스 중에서 선택하여 적용

- Compute - Access & Security - Security Groups - Manage rules for "default" Security Group - Add Rule

- 들어오고 나가는(ingress/egress) 모든 ICMP와 SSH 트래픽을 허용

  ![image-20210406153928675](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210406153928675.png)
# GNS3 실습

> gns 장비를 이용하여 기본 네트워크 실습을 진행한다.

## 목차

- [스위치 기본 설정](#스위치-기본-설정)
- [Router와 PC ping 테스트](#router와-pc-ping-테스트)
- [VLAN 기본 구성](#vlan-기본-구성)
- [VLAN 실습-1](#vlan-실습-1)
- [VLAN 실습-2](#vlan-실습-2)
- [라우터 기본 명령어](#라우터-기본-명령어)
- [스태틱 라우팅 프로토콜](#스태틱-라우팅-프로토콜)
- [디폴트와 스태틱 라우팅 예제](#디폴트와-스태틱-라우팅-예제)
- [Cisco Discovery Protocol](#cisco-discovery-protocol)
- [Routing Information Protocol(RIP)](#routing-information-protocolrip)
- [IGRP 라우팅 프로토콜](#igrp-라우팅-프로토콜)
- [DHCP 구성](#dhcp-구성)
- [OSPF 라우팅 프로토콜](#ospf-라우팅-프로토콜)
- [Access List](#access-list)

---



## 스위치 기본 설정

### 기본 설정

- **스위치 추가**

  ![image-20210605215355066](images/image-20210605215355066.png)

- **스위치의 각 포트들의 현재 상황 확인**
  - `switch# show interface status`
  - ![image-20210605215443862](images/image-20210605215443862.png)

### IP 주소 세팅

- 프리빌리지드(Privileged) 모드 접속

  - `enable`
  - 처음 접속시에는 비밀번호를 묻지 않음
  - 현재 사용하는 설정에서는 이미 프리빌리지드 모드로 접속되어 있음

- 구성(configure) 모드 접속

  - `configure terminal` or `conf t`
  - ![image-20210605215937971](images/image-20210605215937971.png)

- IP 주소는 vlan1 인터페이스에서 세팅하기 때문에 vlan1 인터페이스에 접속해서 IP 세팅

  ```bash
  Switch(config)# interface vlan 1
  Switch(config-if)# ip address 192.168.100.1 255.255.255.0
  ```

  - ![image-20210605230202821](images/image-20210605230202821.png)

- 설정완료 종료

  ```bash
  Switch(config-if)# exit
  Switch(config)# exit
  Switch#
  ```

  - ![image-20210605230323943](images/image-20210605230323943.png)

- IP 구성 확인

  - `Switch# show interface vlan 1`
  - ![image-20210605230538091](images/image-20210605230538091.png)

### 디폴트 게이트웨이 설정

- 디폴트 게이트웨이란 라우터의 주소

- 이 스위치가 붙어있는 라우터 인터페이스의 IP 주소가 바로 디폴트 게이트웨이

  ```sh
  Switch# configure terminal
  Switch(config)# ip default-gateway 192.168.1.1
  Switch(config)# exit
  ```

  - ![image-20210605231022231](images/image-20210605231022231.png)

### 스위치 포트 속도와 Duplex 세팅

- 포트 속도와 Duplex 설정

  ```sh
  Switch# configure terminal
  Switch(config)# interface fastethernet 0/1
  Switch(config-if)# speed ?		// speed 옵션 확인
  Switch(config-if)# speed 10		// 10Mbps로 설정
  Switch(config-if)# duplex ?		// duplex 옵션 확인
  Switch(config-if)# duplex half	// half로 선택
  Switch(config-if)# exit
  Switch(config)# exit
  ```

  - 이유는 모르겠지만 현재 스위치에서는 speed 설정이 안되는 상태이다.
  - 추후에 다른 스위치를 설치하여 진행
  - ![image-20210605233454118](images/image-20210605233454118.png)

- 설정 확인
  - `Switch# show interface fastethernet 0/1`
  - 이유는 아직 모르는데 duplex 또한 설정이 안된다

### Mac Address Table

- Mac Table 확인
  - `Switch# show mac address-table`
  - ![image-20210607231218235](images/image-20210607231218235.png)

- static mac address 설정 

  ```sh
  Switch# configure terminal
  Switch(config)# mac address-table static aaaa.aaaa.aaaa vlan 1 interface ethernet 0/2
  Switch(config)# exit
  Switch# show mac address-table
  ```

  ![image-20210607231409753](images/image-20210607231409753.png)

---



## Router와 PC ping 테스트

### 실습 구성도

​	<img src="images/image-20210603222005069.png" alt="image-20210603222005069" style="zoom:60%;" />
- VPCS를 생성하면 구동시킬 호스트를 로컬과 VM 둘 중 하나를 선택할 수 있는데 저는 VM으로 생성하였습니다.

  > 연결한 Interface 보이게 하기 - VIew->Show/Hide interface labels 활성화

- <img src="images/image-20210603222221471.png" alt="image-20210603222221471" style="zoom:50%;" />

### 라우터 설정

- 라우터를 더블 클릭하거나 오른쪽 클릭 후 콘솔을 눌러 라우터 콘솔 창 진입

  <img src="images/image-20210603222418626.png" alt="image-20210603222418626" style="zoom:50%;" />

- 명령어를 이용한 라우터 설정

  ```shell
  IOU2# conf t		// configuration 진입
  IOU2(config)# int e0/0		// 장비와 연결된 인터페이스로 진입
  IOU2(config-if)# ip address 1.2.3.1 255.255.255.0		// 해당 인터페이스에 ip와 subnet mask 설정
  IOU2(config-if)# do show ip int br		// 인터페이스에 설정된 내역 확인
  ```

  <img src="images/image-20210603223053894.png" alt="image-20210603223053894" style="zoom:67%;" />

### PC 설정

- PC에 IP와 게이트웨이 설정

  ```sh
  PC1> ip 1.2.3.2/24 1.2.3.1
  PC1> show
  ```

  ![image-20210603223458974](images/image-20210603223458974.png)

  ![image-20210603223521672](images/image-20210603223521672.png)

- 라우터로 ping 전송

  `ping 1.2.3.1`

  ![image-20210603223601183](images/image-20210603223601183.png)

---



## VLAN 기본 구성

### VTP 설정

- VTP 도메인 이름 설정

  ```sh
  Switch# configure terminal
  Switch(config)# vtp ?
  Switch(config)# vtp domain cisco
  ```
  - ![image-20210608225759216](images/image-20210608225759216.png)

- VTP 모드 설정
  - 클라이언트는 안되고 서버 혹은 트랜스페어런트인데 서버 모드로 구성
  - `Switch(config)# vtp mode server`
  - ![image-20210608225948013](images/image-20210608225948013.png)

- VTP 구성 확인
  - `Switch# show vtp status`
  - ![image-20210608230105633](images/image-20210608230105633.png)

### 트렁크 포트 설정

- 트렁크 포트는 IEEEE 802.1Q 방식과 ISL 방식 존재

- 트렁크 포트 설정

  ````bash
  Switch# conf t
  Switch(config)# int ethernet 0/1
  Switch(config-if)# switchport trunk ?
  Switch(config-if)# switchport trunk encapsulation ?
  Switch(config-if)# switchport trunk encapsulation dot1q
  ````

  ![image-20210608230707296](images/image-20210608230707296.png)

- 트렁크 포트 설정 확인

  ```sh
  Switch# show interface ethernet 0/1 switchport
  Switch# show interface ethernet 0/1 trunk
  ```

  ![image-20210608230931193](images/image-20210608230931193.png)

  ![image-20210608231030267](images/image-20210608231030267.png)

### VLAN 설정

- VLAN에 대해 아무 설정을 하지 않아도 디폴트 VLAN은 이미 세팅되어 있습니다.
  - `Switch# show vlan`
  - ![image-20210608231147132](images/image-20210608231147132.png)
- 스위치의 만들 수 있는 VLAN 개수 확인
  - `show vtp status`
  - ![image-20210609230000507](images/image-20210609230000507.png)

### VLAN 생성

#### config-vlan 모드

- vlan 2 생성하고 이름 설정

  ```sh
  Switch# configure terminal
  Switch(config)# vlan 2		// VLAN 2생성
  Switch(config-vlan)# name CCNA	// CCNA 이름
  ```

  ![image-20210609230305884](images/image-20210609230305884.png)

- vlan 구성 확인
  - `show vlan`
  - ![image-20210609230343167](images/image-20210609230343167.png)

- vlan 삭제
  - `no vlan 2`
  - ![image-20210609230430649](images/image-20210609230430649.png)

#### vlan configuration 모드

- vlan 생성

  ```sh
  Switch# vlan database
  Switch(vlan)# vlan 2 name CCNA
  Switch(vlan)# exit
  ```

  ![image-20210609230747576](images/image-20210609230747576.png)

- vlan 삭제

  ```sh
  Switch# vlan database
  Switch(vlan)# no vlan 2
  Switch(vlan)# exit
  ```

  ![image-20210609230900931](images/image-20210609230900931.png)

- VTP Revision 넘버 확인
  - `show vtp status`
  - ![image-20210609230941345](images/image-20210609230941345.png)
    - VLAN이 추가, 삭제될 때마다 하나 씩 증가하기 때문에 4
    - VTP 모드가 Transparent 모드라면 0
    - 트랜스페어런트 모드에서는 증가하지 않음

### VLAN에 포트 배정

- ethernet 0/2에 vlan 2 설정

  ```sh
  Switch# configure terminal
  Switch(config)# interface Ethernet 0/2
  Switch(config-if)# switchport access vlan 2
  ```

  ![image-20210609231351680](images/image-20210609231351680.png)

- 구성 확인

  - `show vlan`
  - ![image-20210609231420210](images/image-20210609231420210.png)
  - vlan 2에 대한 정보 확인
    - `show vlan id 2`
    - ![image-20210609231512361](images/image-20210609231512361.png)

- vlan 설정 취소

  ```
  Switch# configure terminal
  Switch(config)# interface Ethernet 0/2
  Switch(config-if)# no switchport access vlan 2
  ```

  ![image-20210609231640016](images/image-20210609231640016.png)

---



## VLAN 실습-1

> https://gmldbd94.tistory.com/102?category=816429의 실습 1

- 실습 구성도

  ![image-20210612114259224](images/image-20210612114259224.png)

### 스위치 VLAN 설정

- SW1

  - VLAN 10과 VLAN 20 생성

    ```sh
    SW1# conf t
    SW1(config)# vlan 10
    SW1(config-vlan)# name ENG
    SW1(config-vlan)# exit
    SW1(config)# vlan 20
    SW1(config-vlan)# name SALES
    ```

    ![image-20210612114413820](images/image-20210612114413820.png)

- SW2

  ```sh
  SW1# conf t
  SW1(config)# vlan 10
  SW1(config-vlan)# name ENG
  SW1(config-vlan)# exit
  SW1(config)# vlan 20
  SW1(config-vlan)# name SALES
  ```

  ![image-20210612114449037](images/image-20210612114449037.png)

### Host IP 설정

- PC1
  - `ip 10.10.10.1/24`
- PC2
  - `ip 10.10.10.2/24`
- PC3
  - `ip 20.20.20.1/24`
- PC4
  - `ip 20.20.20.2/24`

- PC5
  - `ip 10.10.10.3/24`
- PC6
  - `ip 20.20.20.3/24`

### VLAN Interface 설정

- SW1

  ```sh
  SW1# conf t
  SW1(config)# int e0/0
  SW1(config-if)# switchport access vlan 10
  SW1(config-if)# exit
  SW1(config)# int e0/1
  SW1(config-if)# switchport access vlan 10
  SW1(config-if)# exit
  SW1(config)# int e0/2
  SW1(config-if)# switchport access vlan 20
  SW1(config-if)# exit
  SW1(config)# int e0/3
  SW1(config-if)# switchport access vlan 20
  SW1(config-if)# exit
  SW1(config)# exit
  SW1# show vlan
  ```

  ![image-20210612115333115](images/image-20210612115333115.png)

- SW2도 동일하게 구성

  - ![image-20210612115603395](images/image-20210612115603395.png)

- SW1과 SW2 트렁크 포트 설정

  ```sh
  SW1# conf t
  SW1(config)# vtp domain cisco
  SW1(config)# int e1/0
  SW1(config-if)# switchport trunk encapsulation dot1q
  SW2# conf t
  SW1(config)# vtp domain cisco
  SW2(config)# int e0/2
  SW2(config-if)# switchport trunk encapsulation dot1q
  ```

  ![image-20210612120024890](images/image-20210612120024890.png)

  ![image-20210612120131639](images/image-20210612120131639.png)

- 트렁크 포트 설정 확인

  - `SW1# show interface e1/0 trunk`
  - ![image-20210612120315607](images/image-20210612120315607.png)

### 연결 확인

- 같은 스위치와 같은 VLAN을 가진 PC끼리 확인
  - PC1(VLAN 10) -> PC2(VLAN 10)
  - ![image-20210613175134556](images/image-20210613175134556.png)

- 같은 스위치에 다른 VLAN을 가진 PC끼리 확인
  - PC1(VLAN 10) -> PC3(VLAN 20)
  - ![image-20210613175235027](images/image-20210613175235027.png)

- 다른 스위치에 같은 VLAN을 가진 PC끼리 확인
  - PC1(VLAN 10) -> PC5(VLAN 10)
  - ![image-20210613175331162](images/image-20210613175331162.png)

- 다른 스위치에 다른 VLAN을 가진 PC끼리 확인
  - PC1(VLAN 10) -> PC6(VLAN 20)
  - ![image-20210613175404013](images/image-20210613175404013.png)

---



## VLAN 실습-2

> 후니의 쉽게 쓴 시스코 네트워킹 책 VLAN 실습
>
> 참고 블로그 : https://pfe2000.tistory.com/m/206?category=757141

- 실습 구성도

  ![image-20210613180522099](images/image-20210613180522099.png)

### 스위치 설정

- 스위치 관리를 위해 VLAN 1에 IP 주소를 주고 디폴트 게이트웨이 설정

  ```sh
  SW1# configure terminal
  SW1(config)# interface vlan 1
  SW1(config-if)# ip address 10.10.10.2 255.255.255.0
  SW1(config-if)# no shutdown
  SW1(config-if)# exit
  SW1(config)# ip default-gateway 10.10.10.1
  ```

- VTP 모드 트랜스페어런트로 설정

  ```sh
  SW1# configure terminal
  SW1(config)# vtp mode transparent
  SW1(config)# vtp domain cisco
  SW1(config)# exit
  SW1# show vtp status
  ```

  ![image-20210614231146964](images/image-20210614231146964.png)

- vlan 2 생성

  ```sh
  SW1# configure terminal
  SW1(config)# vlan 2
  SW1(config-vlan)# name CCNA
  ```

  ![image-20210614231217796](images/image-20210614231217796.png)

- e0/2 트렁크 포트 설정

  ```sh
  SW1# configure terminal
  SW1(config)# interface e0/2
  SW1(config-if)# switchport trunk encapsulation dot1q
  SW1(config-if)# switchport mode trunk
  ```

  ![image-20210706223113622](images/image-20210706223113622.png)

- e0/1에 VLAN 2 설정

  ```sh
  SW1# configure terminal
  SW1(config)# interface e0/1
  SW1(config-if)# switchport access vlan 2
  ```

  ![image-20210614231600900](images/image-20210614231600900.png)

### 라우터 설정

- 하나의 인터페이스에서 두 개의 인터페이스가 필요하지만 현재 1개이기 때문에 서브 인터페이스 구성

  - VLAN이 2개이기 때문

- 서브 인터페이스 설정

  ```sh
  Router# conf t
  Router(config)# interface e0/0
  Router(config-if)# no shutdown
  Router(config-if)# exit
  Router(config)# interface Ethernet 0/0.1
  Router(config-subif)# encapsulation dot1Q 1 native
  Router(config-subif)# ip address 10.10.10.1 255.255.255.0
  Router(config-subif)# exit
  Router(config)# interface Ethernet 0/0.2
  Router(config-subif)# encapsulation dot1Q 2
  Router(config-subif)# ip address 10.10.20.1 255.255.255.0
  ```

  ![image-20210706223358875](images/image-20210706223358875.png)

### Host IP 설정

- PC1
  - `ip 10.10.1.11 /24 10.10.1.1`
  - ![image-20210708221211140](images/image-20210708221211140.png)
- PC2
  - `ip 10.10.2.11 /24 10.10.2.1`
  - ![image-20210708221232688](images/image-20210708221232688.png)

### 연결 확인

- PC1에서 PC2
  - `ping 10.10.2.11`
  - ![image-20210708221248177](images/image-20210708221248177.png)

- PC1에서 PC2
  - `ping 10.10.1.11`
  - ![image-20210708221303810](images/image-20210708221303810.png)

---



## 라우터 기본 명령어

### 라우터 설정 확인 명령어

- **show version**
  - 라우터의 기본적인 내용, 소프트웨어 버전, 인터페이스 종류 등에 관한 전반적인 내용을 볼 수 있는 명령어입니다.
  - ![image-20210708222354459](images/image-20210708222354459.png)

- **show interface**
  - 현재 라우터가 가지고 있는 모든 인터페이스를 보여주고 그 인터페이스의 현재 상황까지도 알 수 있는 명령어입니다.
  - `show interface` 인터페이스를 지정하지 않으면 모든 인터페이스 정보를 보여줍니다.
  - ![image-20210708222637047](images/image-20210708222637047.png)
  - `show interface serial 2/0`
  - ![image-20210708222702808](images/image-20210708222702808.png)

- **show running-configuration**
  - 라우터에서 현재 돌아가고 있는 구성 파일을 볼 수 있는 명령어입니다.
  - 짧게 `sh run`을 많이 사용합니다.
- **show processes cpu**
  - 라우터의 동작 상태를 보여주며 현재 라우터의 5분, 1분, 5초 동안의 CPU 로드가 퍼센트로 나타납니다.
  - ![image-20210708223112913](images/image-20210708223112913.png)

### 라우터 구성 명령어

- **configuration terminal**

  - 라우터의 구성을 콘솔이나 텔넷을 이용해서 할 때 사용하는 모드입니다.
  - `conf t`로 주로 사용

- **write memory**

  - 구성 모드에서 프리빌리지드 모드로 빠져 나온 다음에 구성 파일을 저장하는 명령어입니다.

- 호스트 네임과 secret과 password 설정

  ```sh
  Router# conf t
  Router(config)# hostname Cisco-R
  Cisco-R(config)# enable secret korea
  Cisco-R(config)# enable password cisco
  ```
  ![image-20210708224724790](images/image-20210708224724790.png)

- 텔넷 라인에 대한 패스워드 세팅

  ```sh
  Cisco-R(config)# line vty 0 4	# vty는 버추얼 라인을 0번부터 4번까지 총 5개의 텔넷이 동시에 가능
  Cisco-R(config-line)# password korea	# password 설정
  Cisco-R(config-line)# login		# 로그인시 패스워드를 체크
  ```

- 인터페이스에 IP주소 설정

  ```sh
  Cisco-R(config)# interface ethernet 0/0 or int e 0/0
  Cisco-R(config-if)# no shutdown
  Cisco-R(config-if)# ip addr 172.16.92.1 255.255.0.0
  ```

  ![image-20210711162730631](images/image-20210711162730631.png)

- 인터페이스에 IPX 네트워크 번호 설정

  ```sh
  Cisco-R(config)# ipx routing
  Cisco-R(config)# int e 0/0
  Cisco-R(config-if)# ipx network 1
  ```

  ![image-20210711163223459](images/image-20210711163223459.png)

- 시리얼 인터페이스 설정

  ```sh
  Cisco-R(config)# interface serial 2/0 or int s 2/0
  Cisco-R(config-if)# no shut or no shutdown
  Cisco-R(config-if)# ip addr 152.100.10.67 255.255.0.0
  ```

  ![image-20210711163604382](images/image-20210711163604382.png)

- 라우팅 프로토콜 설정

  ```sh
  Cisco-R(config)# router eigrp 200	# IP 라우팅 프로토콜 : EIGRP, AS 번호 : 200
  Cisco-R(config-router)# network 172.16.0.0
  Cisco-R(config-router)# network 150.100.0.0
  Cisco-R(config-router)# no network 150.100.0.0		# 네트워크 설정 취소
  Cisco-R(config-router)# network 152.100.0.0
  ```

  ![image-20210711163958137](images/image-20210711163958137.png)

- 라우팅 테이블 확인
  
  - `show ip route`
  
- 구성 파일 확인
  
  - `sh run`
  
- RAM에 있는 구성 파일 NVRAM으로 전송

  ```sh
  Cisco-R# copy running-config startup-config
  or
  Cisco-R# write memory
  ```

---



## 스태틱 라우팅 프로토콜

- 스태틱 라우팅 프로토콜은 라우터 운영자가 직접 경로를 입력한 라우팅 프로토콜입니다.
- 운영자가 입력해준 경로에 문제가 생겨도 다른 길을 자동으로 찾아내지 못하고 다시 운영자가 수정해줄 때까지 기다리는 단점이 존재합니다.
  - 따라서 스태틱 라우텅 프로토콜은 갈 수 있는 경로가 하나밖에 없는 Stub 라우터용으로 많이 사용합니다.

- 스태틱 라우팅 프로토콜 명령어
  - `Router(config)# ip route network [mask] {address | interface} [distance]`
  - 지정하는 방법은 두 가지로 address를 주는 방법과 interface를 주는 방법이 있다.
    - address : 목적지 네트워크를 가기 위해서는 여기 나오는 address로 전송(1홉을 건너뛴 상대방 라우터의 주소)
    - interface : 목적지 네트워크를 가기 위해서는 여기 나오는 interface로 전송(자기 라우터의 인터페이스)

### 스태틱 라우팅 개념 예제

> 출처 : 후니의 쉽게 쓴 Cisco 네트워킹 385p 예제

- 예제 구성도

  ![image-20210711172627487](images/image-20210711172627487.png)

- 지방 사무소의 라우터에 대한 스태틱 라우팅 구성
  - 지방 사무실 라우터는 경로가 하나만 존재하는데 무조건 본사 네트워크에 접속해야 한다.
  - 따라서 라우터 A에 대한 스태틱 라우팅을 구성한다.

- 인터페이스 IP 설정

  ```sh
  Router-A# conf t
  Router-A(config)# int e 0/0
  Router-A(config-if)# no shutdwon
  Router-A(config-if)# ip addr 210.240.10.1 255.255.255.0
  Router-A(config-if)# exit
  Router-A(config)# int s 2/0
  Router-A(config-if)# ip addr 203.210.100.2 255.255.255.0
  ```

  ![image-20210711172037372](images/image-20210711172037372.png)

- 스태틱 라우팅 설정

  ```sh
  Router-A(config)# ip route 150.150.0.0 255.255.0.0 203.210.100.1
  or
  Router-A(config)# ip route 150.150.0.0 255.255.0.0 serial 2/0
  ```

  ![image-20210711172305147](images/image-20210711172305147.png)

- 설정 확인 및 저장

  - `sh run`
  - `write memory`

  > 현재 실습에서는 라우터 A만 설정했기 때문에 라우터 B에도 라우팅 프로토콜을 구성해야 두 라우터간 통신이 가능하고 해당 실습에서는 **어떻게 스태틱 라우팅을 설정하는지에 대한 간단한 실습**이다.

---



## 디폴트 라우트

- 디폴트 라우트란 경로를 찾아내지 못한 모든 네트워크들은 모두 이곳으로 가라고 미리 정해놓은 길입니다.
- 특정 목적지를 지정하지 않고 그냥 모든 목적지가 모두 디폴트로 지정한 곳으로 간다.

### 디폴트 라우트 생성 규칙

- 디폴트 라우트의 네트워크 주소는 항상 클래스를 맞추어 지정해야 한다.
  - 네트워크 주소가 C 클래스라면 203.240.10.0, B 클래스라면 150.100.10.0으로 클래스에 맞게 지정해야 한다.
- RIP에서 디폴트 네트워크를 구성하는 경우 네트워크 주소는 RIP 프로토콜이 돌고 있는 네트워크여야 한다.

### 디폴트 라우트 생성 예제

- 예제 구성도
  - ![image-20210713203603873](images/image-20210713203603873.png)

- Router-A에 RIP 라우팅 프로토콜 설정

  ```sh
  Router-A(config)# router rip
  Router-A(config-router)# network 150.150.0.0
  Router-A(config-router)# network 203.240.10.0
  ```

- 디폴트 라우트 생성
  - `ip default-network` 생성 방법
    - `ip default-network 203.240.10.0`
  - 스태틱 라우팅 생성 방법
    - `ip route 0.0.0.0 0.0.0.0 203.240.10.1`
  - ![image-20210713203850276](images/image-20210713203850276.png)

- 디폴트 라우트 확인
  - `sh run`
  - ![image-20210713203914041](images/image-20210713203914041.png)

---



## 디폴트와 스태틱 라우팅 예제

### 예제 구성도 및 구성

- 예제 구성도
  - ![image-20210713205141877](images/image-20210713205141877.png)
- 라우터A 구성
  - 라우터 이름은 Router_A를 사용
  - Enable password는 cisco를 사용
  - 이더넷 인터페이스는 210.240.10.1 255.255.255.0 네트워크를 사용
  - 라우터 B로는 시리얼 2/0 인터페이스(203.210.100.1 255.255.255.0)을 사용하고 라우터 C로는 시리얼 2/1 인터페이스(203.210.200.1 255.255.255.0)을 사용
  - 모두 스태틱 라우팅만을 사용
  - 나머지 모든 트래픽은 디폴트로 인터넷 쪽으로 나갈 수 있도록 구성

### Router B 구성

```sh
Router-B(config)# int s 2/0
Router-B(config-if)# no shutdown
Router-B(config-if)# ip addr 203.210.100.2 255.255.255.0
```

![image-20210714103915176](images/image-20210714103915176.png)

### Router C 구성

```sh
Router-C(config)# int e 0/0
Router-C(config-if)# no shutdown
Router-C(config-if)# ip addr 150.150.1.1 255.255.0.0
Router-C(config-if)# exit
Router-C(config)# int s 2/0
Router-C(config-if)# no shutdown
Router-C(config-if)# ip addr 203.210.200.2 255.255.255.0
```

### Router A 구성

- 호스트 네임과 패스워드 설정

  ```sh
  Router-A(config)# hostname Router_A
  Router_A(config)# enable password cisco
  ```

  ![image-20210714104326503](images/image-20210714104326503.png)

- 이더넷 인터페이스 설정

  ```sh
  Router_A(config)# int e 0/0
  Router_A(config-if)# no shutdown
  Router_A(config-if)# ip addr 210.240.10.1 255.255.255.0
  ```

  ![image-20210714104454700](images/image-20210714104454700.png)

- 시리얼 인터페이스 설정

  ```sh
  Router_A(config)# int s 2/0
  Router_A(config-if)# no shutdown
  Router_A(config-if)# ip addr 203.210.100.1 255.255.255.0
  Router_A(config-if)# exit
  Router_A(config)# int s 2/1
  Router_A(config-if)# no shutdown
  Router_A(config-if)# ip addr 203.210.200.1 255.255.255.0
  ```

  ![image-20210714104618478](images/image-20210714104618478.png)

### 스태틱 라우팅 설정

- RIP 라우팅 프로토콜 설정

  ```sh
  Router_A(config)# router rip
  Router_A(config-router)# network 203.210.100.0
  Router_A(config-router)# network 203.210.200.0
  ```

  ![image-20210714104955252](images/image-20210714104955252.png)

- 스태틱 라우팅 설정

  ```sh
  Router_A(config)# ip route 150.150.0.0 255.255.0.0 203.210.200.2
  Router_A(config)# ip route 203.210.100.0 255.255.255.0 203.210.100.2
  ```

  ![image-20210714105422940](images/image-20210714105422940.png)

- 디폴트 라우팅 설정
  - `Router_A(config)# ip route 0.0.0.0 0.0.0.0 203.210.100.2`

### 연결 확인

- Router A와 Router C 확인

  ```sh
  Router_A# ping 150.150.1.1
  Router_A# ping 203.210.200.2
  ```

  ![image-20210714105647901](images/image-20210714105647901.png)
  ![image-20210714105656750](images/image-20210714105656750.png)

- Router A와 Router B 확인

  ![image-20210714105711334](images/image-20210714105711334.png)

---



## Cisco Discovery Protocol

- CDP는 시스코 라우터와 스위치에서 직접 연결된 시스코 장비를 찾아내는 기능입니다.
- CDP는 Data Link 계층에 올라가는 프로토콜이기 때문에 상위 네트워크 계층에 어떤 프로토콜이 올라가던 상관없이 실행이 가능합니다.
- CDP는 멀티캐스트를 이용해서 시스코 장비를 찾아냅니다.

### 예제 구성도

<img src="images/image-20210714113246933.png" alt="image-20210714113246933" style="zoom:67%;" />

- 라우터 A에서 CDP 진행

### CDP 명령어

- `show cdp`
  - cdp 관련 설정 확인 
  - ![image-20210714113653944](images/image-20210714113653944.png)

- `show cdp neighbors`

  - 현재 자기 라우터와 연결된 시스코 장비들의 정보 확인

  - ![image-20210714114011648](images/image-20210714114011648.png)

    > Router D는 직접적으로 연결되지 않고 스위치를 통해 연결되어 있어 나오지 않음

- `show cdp entry *` or `show cdp neighbors details`

  - `show cdp neighbors`보다 많은 정보 확인 가능
  - ![image-20210714114247013](images/image-20210714114247013.png)
  - ![image-20210714114310848](images/image-20210714114310848.png)

- `no cdp run`
  - 라우터 전체에 cdp를 disable하는 명령어
  - 다시 CDP를 살리고 싶으면 `cdp run`
  - ![image-20210714114507701](images/image-20210714114507701.png)
  - Router B에 대한 Holdtime이 계속 줄어들다가 사라짐
  - ![image-20210714114609433](images/image-20210714114609433.png)
  - ![image-20210714114707392](images/image-20210714114707392.png)

- `no cdp enable`
  - 특정 인터페이스에서만 CDP를 Disable
  - ![image-20210714114828133](images/image-20210714114828133.png)

---



## Routing Information Protocol(RIP)

- RIP는 OSI 7계층 중 Network 레이어에서 동작하는 라우팅(Routing Protocol)로 다이나믹 프로토콜입니다.
- 내부용 라우팅 프로토콜(IGP)로 디스턴스 벡터(Distance Vector) 라우팅 프로토콜입니다.
- RIP는 경로를 설정할 때 홉 카운트를 사용하고 최대 15 개이고 갱신 주기는 30초입니다.

#### RIP 설정 명령어

```sh
Router(config)# router rip
Router(config-router)# network 150.150.100.0
```

- ![image-20210815170831638](images/image-20210815170831638.png)
- `router rip`는 현재 라우터에서 RIP 라우팅을 사용하겠다는 명령어입니다.
- `network 150.150.100.0`은 라우팅을 사용할 네트워크를 지정하는 명령어로 서브넷 마스크를 입력하지 않고 Classful하게 모든 네트워크를 인식합니다.
  - 그렇기 때문에`show running-config`를 수행하면 네트워크가 `150.150.0.0`으로 설정되어 있는 것을 확인할 수 있습니다.
  - ![image-20210815170801907](images/image-20210815170801907.png)

### RIP 예제-1

> 후니의 쉽게 쓴 CISCO 네트워킹 Vol.2 16p 예제

#### 예제 구성도

![image-20210815171716250](images/image-20210815171716250.png)

#### RouterA 구성

- 서울 리전에 있는 RouterA에 이더넷 및 라우팅 구성입니다.

- 이더넷 설정

  ```sh
  RouterA# configure terminal
  RouterA(config)# interface ethernet 0/0
  RouterA(config-if)# ip address 203.240.100.1 255.255.255.0
  ```

  ![image-20210815172509829](images/image-20210815172509829.png)

- 시리얼 인터페이스 설정

  ```sh
  RouterA# configure terminal
  RouterA(config)# interface serial 2/0
  RouterA(config-if)# ip address 203.240.150.1
  ```

  ![image-20210815172748124](images/image-20210815172748124.png)

- RIP 구성

  ```sh
  RouterA# configure terminal
  RouterA(config)# router rip
  RouterA(config-router)# network 203.240.100.0
  RouterA(config-router)# network 203.240.150.0
  ```

  ![image-20210815172754817](images/image-20210815172754817.png)

#### RouterB 구성

- 이더넷 설정

  ```sh
  RouterA# configure terminal
  RouterA(config)# interface ethernet 0/0
  RouterA(config-if)# ip address 203.240.200.1 255.255.255.0
  ```

  ![image-20210815173939490](images/image-20210815173939490.png)

- 시리얼 인터페이스 설정

  ```sh
  RouterA# configure terminal
  RouterA(config)# interface serial 2/0
  RouterA(config-if)# ip address 203.240.150.2 255.255.255.0
  ```

  ![image-20210815174012790](images/image-20210815174012790.png)

- rip 설정

  ```sh
  RouterA# configure terminal
  RouterA(config)# router rip
  RouterA(config-router)# network 203.240.100.0
  RouterA(config-router)# network 203.240.150.0
  ```

  ![image-20210815174036249](images/image-20210815174036249.png)

#### 구성 확인

- RouterA
  - `show running-config`
  - ![image-20210815173009535](images/image-20210815173009535.png)![image-20210815173015051](images/image-20210815173015051.png)![image-20210815173020630](images/image-20210815173020630.png)
  - `show ip protocol`
    - ![image-20210815173524827](images/image-20210815173524827.png)
  - `show ip route`
  - ![image-20210815174243536](images/image-20210815174243536.png)

- RouterB
  - `show running-config`
  - ![image-20210815174516634](images/image-20210815174516634.png)![image-20210815174528610](images/image-20210815174528610.png)![image-20210815174550361](images/image-20210815174550361.png)
  - `show ip protocol`
    - ![image-20210815174625631](images/image-20210815174625631.png)
  - `show ip route`
    - ![image-20210815174650298](images/image-20210815174650298.png)

### RIP 예제-2

> 후니의 쉽게 쓴 CISCO 네트워킹 Vol.2 26p 예제

#### 예제 구성

![image-20211020221226727](images/image-20211020221226727.png)

- PC1에서 PC2로 통신이 가능하도록 설정

#### PC 설정

- PC 1 ip 설정
  - `ip 150.150.1.10 /16 150.150.1.1`
- PC 2 ip 설정
  - `ip 172.70.100.10 /24 172.70.100.1`

#### RouterC 구성

```sh
# conf t
# interface eth0/0
# no shutdown
# ip address 150.150.1.1 255.255.0.0
# interface Serial 2/0
# no shutdown
# ip address 203.210.200.2 255.255.255.0
# router rip
# network 150.150.0.0
# network 203.210.200.0
```

#### RouterA 구성

```sh
# conf t
# interface serial 2/1
# no shutdown
# ip address 203.210.200.1 255.255.255.0
# clockrate 128000    # 백투백 구성을 위한 설정
# interface serial 2/0
# no shutdown
# ip address 203.210.100.1 255.255.255.0
# clockrate 1007616
# router rip
# network 203.210.200.0
# network 203.210.100.0
```

#### Router C 구성

```sh
# conf t
# interface serial 2/0
# no shutdown
# ip address 203.210.100.2 255.255.255.0
# interface eth 0/0
# no shutdown
# ip address 172.70.100.1 255.255.255.0
# router rip
# network 203.210.100.0
# network 172.70.100.0
```

#### 디버그

- `debug ip rip`
- 디버그는 반드시 꺼야됨
  - `u all`

#### 연결 확인

- PC1에서 PC2로 핑
  - `ping 172.70.100.10`
  - ![image-20211020221811587](images/image-20211020221811587.png)

- RouterC에서 RouterB로 핑
  - `ping 172.70.100.1`
  - ![image-20211020221924468](images/image-20211020221924468.png)

---



## IGRP 라우팅 프로토콜

- IGRP 라우팅 프로토콜 특성
  - 라우팅 프로토콜
  - 다이나믹 프로토콜
  - 내부용 라우팅 프로토콜(IGP)
  - 디스턴스 벡터 라우팅 프로토콜
  - 시스코라우터에서만 사용 가능
  - VLSM을 지원하지 못함
- IGRP 라우팅 프로토콜에서 경로 결정 요소
  - 대역폭
  - Delay
  - Reliability - 목적지까지 제대로 도착한 패킷과 에러가 발생한 패킷의 비율
  - Load(부하)
  - MTU(Maximum Transmission Unit)
- IGRP 구성 명령어
  - `router igrp <autonomous system number(AS번호)>`
  - `network network-number`
- 해당 인터페이스로는 igrp를 제외하는 명령어
  - `passive-interface Ethernet 0/0`

### IGRP 예제

> 후니의 쉽게 쓴 CISCO 네트워킹 Vol.2 49p
>
> 현재 설치한 버전에서는 igrp를 지원하지 않는 것으로 보여 igrp를 완전히 대체하는 eigrp로 실습을 진행하기 때문에 만약 igrp가 가능한 라우터에서는 eigrp 대신 igrp로 변경하여 입력하시면 됩니다.

#### 예제 구성

- AS 번호는 200
- ![image-20211026225452117](images/image-20211026225452117.png)

#### host 설정

```sh
// PC 1
# ip 203.240.100.10 /24 203.240.100.1

// PC 2
# ip 203.240.200.10 /24 203.240.200.1
```

#### RouterA 설정

```sh
# conf t
# interface eth0/0
# no shutdown
# ip address 203.210.100.1 255.255.255.0
# interface serial 2/0
# no shutdown
# ip address 203.210.150.1 255.255.255.0
# router eigrp 200
# network 203.210.100.0
# network 203.210.150.0
```

#### RouterB 설정

```sh
# conf t
# interface eth0/0
# no shutdown
# ip address 203.210.200.1 255.255.255.0
# interface serial 2/0
# no shutdown
# ip address 203.210.150.1 255.255.255.0
# router eigrp 200
# network 203.210.200.0
# network 203.210.150.0
```

#### 연결 확인

- PC1에서 PC2로 핑
  - ![image-20211026225442415](images/image-20211026225442415.png)

### IGRP 예제-2

>  후니의 쉽게 쓴 CISCO 네트워킹 Vol.2 55p

#### 예제 구성

- AS 200
- ![image-20211102221142324](images/image-20211102221142324.png)

#### host 설정

```sh
// PC1
# ip 150.150.1.10 /16 150.150.1.1
// PC2
# ip 210.240.10.10 /24 210.240.10.1
// PC3
# ip 172.70.100.10 /24 172.70.100.1
```

#### RouterC 설정

```sh
# conf t
# interface eth 0/0
# no shutdown
# ip address 150.150.1.1 255.255.0.0
# interface serial 2/0
# no shutdown
# ip address 203.210.200.2 255.255.255.0
# router eigrp 200
# network 150.150.0.0
# network 203.210.200.0
```

#### RouterA 설정

```sh
# conf t
# interface eth 0/0
# no shutdown
# ip address 210.240.10.1 255.255.255.0
# interface serial 2/0
# no shutdown
# ip address 203.210.100.1 255.255.255.0
# interface serial 2/1
# no shutdown
# ip address 203.210.200.1 255.255.255.0
# router eigrp 200
# network 210.240.10.0
# network 203.210.100.0
# network 203.210.200.0
```

#### RouterB 설정

```sh
# conf t
# interface eth 0/0
# no shutdown
# ip address 172.70.100.1 255.255.255.0
# interface serial 2/0
# no shutdown
# ip address 203.210.100.2 255.255.255.0
# router eigrp 200
# network 203.210.100.0
# network 172.70.100.0
```

#### 설정 확인

- `show ip route`

- `show running-config`

- `show interface`

- `show ip protocol`

- `show ip interface`

- 최대 홉 카운트 변경

  ```
  # conf t
  # router eigrp 200
  # metric maximum-hops 255
  #
  ```

#### 디버그

- `debug ip igrp`
- 마지막에는 항상 디버그 명령을 꺼야됨
  - `u all`

#### 연결 확인

- PC1에서 PC2
  - `ping 210.240.10.10`
  - ![image-20211102222903856](images/image-20211102222903856.png)
- PC1에서 PC3
  - `ping 172.70.100.10`
  - ![image-20211102222921838](images/image-20211102222921838.png)

---



## DHCP 구성

- 라우터 DHCP 구성

  ```
  Router# conf t
  Router(config)# service dhcp
  ```

- DHCP Pool, default router 설정

  ```
  Router(config)# ip dhcp pool ccna   // pool 이름 ccna로 설정
  Router(dhcp-config)# network 210.240.10.0 255.255.255.0
  // IP Pool 지정 210.240.10.1 ~ 210.240.10.254
  Router(dhcp-config)# default-router 210.240.10.1
  // 자기 네트워크에 붙어있는 라우터의 주소
  ```

- 이미 사용하고 있는 주소 제외

  ```
  Router(config)# ip dhcp excluded-address 210.240.10.1
  Router(config)# ip dhcp excluded-address 210.240.10.100
  ```

- 현재 배정해준 IP 주소 확인 및 DHCP 상태 확인

  ```
  Router# show ip dhcp binding   // 현재 배정된 IP 주소 확인
  Router# show ip dhcp server statistics    // DHCP 서버 상태 확인
  ```

---



## OSPF 라우팅 프로토콜

- OSPF 라우팅 프로토콜의 장점
  - 빠른 업데이트
  - Area라는 개념을 사용해 전체 네트워크를 분리하여 효율적인 관리
  - VLSM 지원
  - 홉 카운트 제한 없음
  - 변화가 있을 때만 정보를 교환하고 멀티캐스트를 사용해 실용적
- OSPF의 특징
  - 표준 라우팅 프로토콜
  - 링크스테이트 라우팅 알고리즘
  - DR(Designated Router)과 BDR(Backup Designated Router)
    - DR은 Link State 정보를 관리하여 Sync를 일치시킴
    - BDR은 DR이 다운되면 DR의 역할을 수행
    - DR과 BDR의 Link State를 Sync(일치)해야 함(Adjacency)
    - DR과 BDR은 라우터의 ID와 Priority를 가지고 선출(Priority가 높은 라우터가 DR, 만약 Priority가 같으면 라우터 ID가 높은 라우터가 DR)
    - default priority는 1, DR, BDR 선거에서 제외시킬려면 0으로 설정
- OSPF 구성할 때 주의사항
  - Hello/dead intervals, Area-ID, password, Stub area flag 가 라우터들끼리 모두 동일해야 함
  - 라우터 ID로 사용할 인터페이스는 보통 Loopback 인터페이스를 사용

### OSPF 구성

- OSPF Enable
  - `Router(config)# router ospf <process-id>`
- 네트워크 설정
  - `Router(config-router)# network <wildcard-mask> area <area-id>`
  - `network 150.100.1.0 0.0.0255 area 0`

### OSPF 예제

> 후니의 쉽게 쓴 CISCO 네트워킹 Vol.2 86p

#### 예제 구성

- ![image-20211110221151839](images/image-20211110221151839.png)

- area 0
- process-id 100

#### 호스트 설정

```
PC1> ip 172.16.10.100 /24 172.16.10.1
PC2> ip 172.16.30.100 /24 172.16.30.1
```

#### RouterA 구성

```
RouterA# conf t
RouterA(config)# inter eth 0/0
RouterA(config-if)# no shutdown
RouterA(config-if)# ip address 172.16.10.1 255.255.255.0
RouterA(config)# inter serial 2/0
RouterA(config-if)# no shutdown
RouterA(config-if)# ip address 192.168.12.1 255.255.255.240
RouterA(config)# router ospf 100
RouterA(config-router)# network 192.168.12.0 0.0.0.15
RouterA(config-router)# network 172.16.10.0 0.0.0.255
```

#### RouterB 구성

```
RouterB# conf t
RouterB(config)# inter serial 2/0
RouterB(config-if)# no shutdown
RouterB(config-if)# ip address 192.168.12.2 255.255.255.240
RouterB(config)# inter serial 2/1
RouterB(config-if)# no shutdown
RouterB(config-if)# ip address 192.168.23.2 255.255.255.240
RouterB(config)# router ospf 100
RouterB(config-router)# network 192.168.12.0 0.0.0.15 area 0
RouterB(config-router)# network 192.168.23.0 0.0.0.15 area 0
```

#### RouterC 구성

```
RouterC# conf t
RouterC(config)# inter serial 2/0
RouterC(config-if)# no shutdown
RouterC(config-if)# ip address 192.168.23.3 255.255.255.240
RouterC(config)# inter eth 0/0
RouterC(config-if)# no shutdown
RouterC(config-if)# ip address 172.16.30.1 255.255.255.0
RouterC(config)# router ospf 100
RouterC(config-router)# network 192.168.23.0 0.0.0.15 area 0
RouterC(config-router)# network 172.16.30.1 0.0.0.255 area 0
```

#### 연결 확인

- PC1 -> PC2
  - `ping 172.16.10.100`
  - ![image-20211110223805092](images/image-20211110223805092.png)

#### OSPF 구성 확인

- `show ip ospf interface`
- `show ip ospf neighbor`

---



## Access List

- 접근을 허용할지 말지 미리 정해놓은 리스트

- 스탠더드 액세스 리스트(Standard Access List)

  - 출발지 주소만을 이용해 접근을 통제

- 익스텐디드 액세스 리스트(Extended Access List)

  - 출발지 주소, 목적지 주소, 프로토콜, 포트 번호 등을 이용해 접근을 통제

- 규칙

  - 액세스 리스트는 윗줄부터 하나씩 차례로 수행
  - 맨 마지막에 `permit any`를 넣지 않으면 default로 액세스 리스트와 매칭되지 않는 모든 address는 deny
  - 새로 추가하는 라인은 맨 마지막에 추가되므로 access-list line의 선택적 추가(selective add)나 제거(remove)가 불가능
  - interface에 대한 엑시스 리스트가 정의 되지 않은 경우 결과는 `permit any`

- 주의사항

  ```
  // 익스텐디드 엑세스 리스트 추가
  Router(config)# access-list 105 permit tcp any host 201.222.11.7 eq domain
  Router(config)# access-list 105 permit tcp any host 201.222.11.7 eq ftp
  
  // eq ftp에 대한 ip를 수정하기 위해 삭제하고 다시 추가
  Router(config)# no access-list 105 permit tcp any host 201.222.11.7 eq ftp // no를 이용해 삭제
  Router(config)# access-list 105 permit tcp any host 101.222.11.7 eq ftp
  ```

  - 만약 `no` 명령어를 이용해 삭제를 하면 정의되어 있는 모든 액세스 리스트가 삭제
  - 따라서 위에는 `domain`에 대한 리스트도 삭제되고 맨 마지막에 입력한 리스트만 남음

### Standard Access List

- 기본 설정 명령어

  ```
  Router(config)# access-list access-list-number(1~99) {permit | deny} {source-address [source-wildcard]} | any}   // wildcard는 생략 가능, 생략하면 0.0.0.0
  Router(config-if)# ip access-group access-list-number {in | out}   // 설정할 인터페이스에서 실행
  ```


#### Standard Access List 예제

> 후니의 쉽게 쓴 CISCO 네트워킹 Vol.2 110p

##### 예제 구성



- PC 2를 제외한 210.240.100.0 네트워크의 모든 PC는 PC1에 접속할 수 있다.
- 210.240.150.0 네트워크에서는 PC 4를 포함해서 나머지 모든 PC들이 PC A를 접속할 수 있다.
- 이 외의 모든 PC는 PC A를 접속할 수 없다.

##### 호스트 설정

```
PC1> ip 203.210.100.15 /24 203.210.100.1
PC2> ip 210.240.100.5 /24 210.240.100.1
PC3> ip 210.240.100.10 /24 210.240.100.2
PC4> ip 210.240.150.100 /24 210.240.15.1
```

##### RouterA IP 설정

```
RouterA# conf t 
RouterA(conf)# inter eth 0/0
RouterA(conf-if)# no shutdown
RouterA(conf-if)# ip address 203.210.100.1 255.255.255.0
RouterA(conf)# inter serial 2/0
RouterA(conf-if)# no shutdown
RouterA(conf-if)# ip address 150.100.100.1 255.255.0.0
RouterA(conf)# inter serial 2/1
RouterA(conf-if)# no shutdown
RouterA(conf-if)# ip address 150.200.100.1 255.255.0.0
RouterA(conf)# router rip
RouterA(conf-router)# network 203.210.100.0
RouterA(conf-router)# network 150.100.0.0
RouterA(conf-router)# network 150.200.0.0
```

##### RouterB 설정

```
RouterB# conf t
RouterB(conf)# inter eth 0/0
RouterB(conf-if)# no shutdown
RouterB(conf-if)# ip address 210.240.150.1 255.255.255.0
RouterB(conf)# inter serial 2/0
RouterB(conf-if)# no shutdown
RouterB(conf-if)# ip address 150.200.100.2 255.255.0.0
RouterB(conf)# router rip
RouterB(conf-router)# network 210.240.150.0
RouterB(conf-router)# network 150.200.0.0
```

##### RouterC 설정

```
RouterC# conf t
```


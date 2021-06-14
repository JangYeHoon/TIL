# GNS3 실습

> gns 장비를 이용하여 기본 네트워크 실습을 진행한다.

## 목차

- [스위치 기본 설정](#스위치-기본-설정)
  - [기본 설정](#기본-설정)
  - [IP 주소 세팅](#ip-주소-세팅)
  - [디폴트 게이트웨이 설정](#디폴트-게이트웨이-설정)
  - [스위치 포트 속도와 Duplex 세팅](#스위치-포트-속도와-duplex-세팅)
  - [Mac Address Table](#mac-address-table)
- [Router와 PC ping 테스트](#router와-pc-ping-테스트)
  - [실습 구성도](#실습-구성도)
  - [라우터 설정](#라우터-설정)
  - [PC 설정](#pc-설정)
- [VLAN 기본 구성](#vlan-기본-구성)
  - [VTP 설정](#vtp-설정)
  - [트렁크 포트 설정](#트렁크-포트-설정)
  - [VLAN  설정](#vlan-설정)
  - [VLAN 생성](#vlan-생성)
    - [config-vlan 모드](#config-vlan-모드)
    - [vlan configuration 모드](#vlan-configuration-모드)
  - [VLAN에 포트 배정](#vlan에-포트-배정)
- [VLAN 실습-1](#vlan-실습-1)
  - [스위치 VLAN 설정](#스위치-vlan-설정)
  - [Host IP 설정](#host-ip-설정)
  - [VLAN과 Host 연결](#vlan과-host-연결)
  - [VLAN Interface 설정](#vlan-interface-설정)
  - [연결 확인](#연결-확인)

- [VLAN 실습-2](#vlan-실습-2)



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



## VLAN 실습-1

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



## VLAN 실습-2

- 실습 구성도

  ![image-20210613180522099](images/image-20210613180522099.png)


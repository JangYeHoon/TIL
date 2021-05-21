# Trunk / Access Port

> Trunk Port와 Access Port의 기본 정리입니다.

### 목차

- [Access Port](#access-port)
- [Trunk Port](#trunk-port)
- [DTP(Dynamic Trunk Protocol)](#dtpdynamic-trunk-protocol)
- [문제 및 참고자료](#참고자료)



## Access Port

- Switch Port에 하나의 VLAN을 할당하여 해당 VLAN끼리만 트래픽이 흐를 수 있도록 설정하는 Port 입니다.
- VLAN tag를 제거하고 데이터를 전송합니다.

#### Cisco Switch에서 Access Port 할당 방법

```sh
Switch# conf t
Switch(config-if)# switchport	// Layer 2 Interface로 설정
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10	// Port가 속하게 되는 VLAN 설정
```



## Trunk Port

- 하나의 포트로 VLAN tag가 달린 모든 패킷을 전송할 수 있도록 설정하는 Port 입니다.
- 장비에 설정된 VLAN Segment data를 전송하기 위한 것, 즉 일반적인 Ethernet frame을 VLAN ID를 확인할 수 있돌 Encapsulation하거나 frame에 VLAN ID를 삽입하여 전송한다.
  - **즉 Trunk란 복수개의 VLAN 프레임을 전송할 수 있는 링크**
- Access Port와는 다르게 VLAN tag를 제거하지 않고 그대로 전송합니다.

#### Cisco Switch에서 Trunk Port 할당 방법

- **기본적인 Trunk Port 설정**

  ```bash
  Switch# conf t
  Switch(config-if)# switchport
  Switch(config-if)# switchport mode trunk
  ```

- **트렁크 프로토콜에 따른 트렁크 인캡슐레이션 지정**

  ```bash
  Switch(config-if)# switchport trunk encapsulation dot1q
  or
  Switch(config-if)# switchport trunk encapsulation isl
  ```

  - 802.1q 트렁크
    - 이더넷 프레임의 출발지 주소 다음에 4바이트 길이의 802.1q 태그를 추가하여 VLAN 정보를 표시하는 방식이다.
  - ISL 트렁크
    - 시스코에서 개발한 트렁크 인캡슐레이션 방식으로 확장 VLAN을 지원하지 못하고 1024개의 VLAN만을 지원한다.
    - 이더넷 프레임을 변경하지 않고 프레임 앞에 26바이트의 ISL 헤더와 프레임 끝에 ISL FCS 4바이트를 추가하는 방식이다.

- **특정 VLAN Trunk Port 설정**

  ```bash
  Switch(config-if)# switchport trunk allowed vlan 10-11,20 // vlan 10, 11, 20 사용
  Switch(config-if)# switchport trunk allowed vlan add 30-31,40 // 기존 vlan에 30, 31, 40 추가
  Switch(config-if)# switchport trunk allowed vlan all // 모든 VLAN 사용
  Switch(config-if)# switchport trunk allowed vlan except 101-200 // 101~200 VLAN 제외
  Switch(config-if)# switchport trunk allowed vlan none // 모든 VLAN 차단
  Switch(config-if)# switchport trunk allowed vlan remove 101-200 // 101~200 VLAN 삭제
  ```

  

## DTP (Dynamic Trunking Protocol)

- 시스코 스위치에서 상대방 스위치와 협상하기 위한 프로토콜

#### Access mode

- DTP 메시지를 전송하지 않으며, 상대방이 보낸 DTP를 무시한다.
- 상대 포트와 상관없이 자신은 Access Port로 동작한다.

#### Dynamic(Auto) mode

- DTP를 먼저 전송하지 않고 상대방에게 DTP를 받은 경우 DTP를 전송한다.
- 상대 포트에 따라 자신의 모드를 지정한다.
  - 상대 포트가 Trunk나 Desirable이면 Trunk mode로 동작
  - 상대 포트가 Auto나 Access이면 Access mode로 동작
- `Switch(config-if)# switchport mode dynamic auto`

#### Dynamic(Desirable) mode

- DTP를 먼저 전송한다.
- 상대 포트에 따라 자신의 모드를 지정한다.
  - 상대 포트가 Trunk나 Desirable이면 Trunk mode로 동작
  - 상대 포트가 Access이면 Access mode로 동작
- `Switch(config-if)# switchport mode dynamic desirable`

#### Trunk mode

- DTP를 먼저 전송하며, 상대방이 보낸 DTP는 무시한다.



### 참고자료

- https://blog.naver.com/PostView.nhn?blogId=sung_mk1919&logNo=221393785227&parentCategoryNo=&categoryNo=71&viewDate=&isShowPopularPosts=true&from=search
  - 기본적인 개념과 예시를 통해 VLAN Port에 대한 동작 과정 이해
- https://jeongzzang.com/38
- https://m.blog.naver.com/PostView.nhn?blogId=lunarispars&logNo=221440105402&categoryNo=20&proxyReferer=https:%2F%2Fwww.google.com%2F
- https://blog.naver.com/hanoolee/221345231920
  - 문제를 통해 VLAN 동작 개념 잡기 좋음


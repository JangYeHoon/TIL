# OpenStack에서 OpenVSwitch 작동 방식

> https://aptira.com/openstack-rules-how-openvswitch-works-inside-openstack/

### OpenFlow 규칙 이해

- OpenVSwitch(OVS)는 가상 링크 및 포트를 사용하여 가상 머신을 함께 연결하는 가상 스위치
- OpenStack에서 OVS는 가상화된 네트워크 서비스를 제공하고 Neutron 노드와 Computing 노드가 OpenVSwitch를 실행하는 역할
- OVS에서 중요한 것은 룰을 기반으로 패킷이 들어오고 나가는 것을 처리하는 역할

=> **해당 문서에서는 OpenStack Mitaka 내부의 OVS에 설치된 Flow Rule 을 설명**

- Mitaka 노드에 로그인하고 computing 노드에 로그인
- 그리고 OpenStack의 br-tun 정보를 출력하여 플로우 룰 확인
  - `ovs-ofctl dump-flows br-tun`



## 플로우 룰 설명

### Table 0

```bash
1- cookie=0xbb7b3cdd52626a01, duration=13003.029s, table=0, n_packets=183, n_bytes=28498, idle_age=4, priority=1,in_port=1 actions=resubmit(,2)
2- cookie=0xbb7b3cdd52626a01, duration=9917.985s, table=0, n_packets=198, n_bytes=36045, idle_age=4, priority=1,in_port=2 actions=resubmit(,4)
3- cookie=0xbb7b3cdd52626a01, duration=13003.030s, table=0, n_packets=0, n_bytes=0, idle_age=13003, priority=0 actions=drop
```

- 1 : 우선순위는 1이고 in_port가 patch-int(1)에서 들어온 패킷인지 확인하고 맞으면 **table 2**로 전송합니다.
- 2 : in_port가 vxlan-c0a80202(2)에서 들어온 패킷인지 확인하고 맞으면 **table4**로 전송합니다
- 3 : 가장 낮은 우선순위를 갖고 1번과 2번 룰에 매칭이 안되는 패킷이면 **drop**합니다.

### Table 2

```bash
4- cookie=0xbb7b3cdd52626a01, duration=13003.029s, table=2, n_packets=1, n_bytes=42, idle_age=9913, priority=1,arp,dl_dst=ff:ff:ff:ff:ff:ff actions=resubmit(,21)
5- cookie=0xbb7b3cdd52626a01, duration=13003.029s, table=2, n_packets=168, n_bytes=26780, idle_age=4, priority=0,dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,20)
6- cookie=0xbb7b3cdd52626a01, duration=13003.029s, table=2, n_packets=14, n_bytes=1676, idle_age=9904, priority=0,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,22)
```

- 4 : 들어온 패킷이 ARP 패킷인지와 목적지 MAC 주소가 브로드 캐스트인지 확인하여 맞으면 **table 21**로 전송합니다
- 5 : 들어온 패킷이 dl_dst=00:00:00:00:00:00/01:00:00:00:00:00(모든 유니캐스트 이더넷 패킷)인지 확인하여 맞으면 **table 20**으로 전송합니다.
- 6 : 들어온 패킷이 dl_dst=01:00:00:00:00:00/01:00:00:00:00:00(모든 멀티캐스트(브로드캐스트 이더넷 포함))인지 확인하여 맞으면 **table 22**로 전송합니다.

### Table 3

```bash
7- cookie=0xbb7b3cdd52626a01, duration=13003.029s, table=3, n_packets=0, n_bytes=0, idle_age=13003, priority=0 actions=drop
```

- 7 : table 3으로 들어오는 모든 패킷은 **drop**합니다.

### Table 4

```bash
8- cookie=0xbb7b3cdd52626a01, duration=9921.166s, table=4, n_packets=198, n_bytes=36045, idle_age=4, priority=1,tun_id=0x2 actions=mod_vlan_vid:1,resubmit(,10)
9- cookie=0xbb7b3cdd52626a01, duration=13003.029s, table=4, n_packets=0, n_bytes=0, idle_age=13003, priority=0 actions=drop
```

- 8 : tun_id=0x20 패킷인지 확인하여 맞으면 vlan_vid 1을 추가하고 **table 10**으로 전송합니다.
- 9 : 8번 룰에 매칭되지 않으면 패킷을 **drop**합니다.

### Table 6

```bash
10- cookie=0xbb7b3cdd52626a01, duration=13003.029s, table=6, n_packets=0, n_bytes=0, idle_age=13003, priority=0 actions=drop
```

- 10 : table 6으로 들어오는 모든 패킷은 **drop**합니다.

### Table 10

```bash
11- cookie=0xbb7b3cdd52626a01, duration=13003.029s, table=10, n_packets=198, n_bytes=36045, idle_age=4, priority=1 actions=learn(table=20,hard_timeout=300,priority=1,cookie=0xbb7b3cdd52626a01,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:1
```

- 11번 룰은 **action이 두 부분으로 구성**됩니다.
- 11 - 1 : **table 20**에 룰을 설치하는데 table20은 MAC 학습 테이블입니다.
  - `learn` action은 현재 **table 4**에서 처리중인 flow의 내용을 기반으로 flow table을 수정합니다.
  - learn의 각 항목에 대한 해석
    - `table=20` : table 20을 수정하는 의미
    - `hard_timeout=300` : 300초 후에 해당 flow entry를 삭제
    - `priority=1` : 우선순위를 1로 설정
    - `cookie=0xbb7b3cdd52626a01,NXM_OF_VLAN_TCI[0..11]` : table 20에 추가하는 flow가 현재 처리중인 패킷에 포함된 VLAN ID와 일치하도록 설정
      - 이렇게 하면 MAC learning entry가 VLAN 스위치의 일반적인 동작인 단일 VLAN을 효과적으로 범위를 지정
    - `NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[]` : table 20에 추가하는 flow가 현재 처리중인 패킷의 ehternet src와 ethernet dst를 일치하도록 설정
    - `load:0->NXM_OF_VLAN_TCI[]` : 0을 VLAN ID로 로드하여 VLAN ID를 제거
    - `load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[]` : 현재 처리중인 패킷의 터널 ID를 패킷의 터널 ID로 로드
    - `output:NXM_OF_IN_PORT[]` : **현재 들어온 in_port로 패킷을 전송**
- 11 - 2 : patch-int(1)로 패킷을 전송합니다.

### Table 20

```bash
12- cookie=0xbb7b3cdd52626a01, duration=9917.984s, table=20, n_packets=102, n_bytes=14108, idle_age=9435, priority=2,dl_vlan=1,dl_dst=fa:16:3e:0b:cf:10 actions=strip_vlan,set_tunnel:0x2,output:2
13- cookie=0xbb7b3cdd52626a01, duration=9917.984s, table=20, n_packets=66, n_bytes=12672, idle_age=4, priority=2,dl_vlan=1,dl_dst=fa:16:3e:4a:10:2b actions=strip_vlan,set_tunnel:0x2,output:2
14- cookie=0xbb7b3cdd52626a01, duration=9913.613s, table=20, n_packets=0, n_bytes=0, hard_timeout=300, idle_age=9913, hard_age=4, priority=1,vlan_tci=0x0001/0x0fff,dl_dst=fa:16:3e:4a:10:2b actions=load:0->NXM_OF_VLAN_TCI[],load:0x2->NXM_NX_TUN_ID[],output:2
15- cookie=0xbb7b3cdd52626a01, duration=13003.029s, table=20, n_packets=0, n_bytes=0, idle_age=13003, priority=0 actions=resubmit(,22)
```

- 12, 13 : 들어온 패킷이 VLAN ID가 1이고 특정 dl_dst address와 일치한다면 VLAN ID를 제거하고 터널 ID를 0x2로 로드하여 **vxlan-c0a80202**로 패킷을 전송합니다.
- 14 : 해당 룰과 비슷한 룰든은 **table 10**의 `learn action`을 통해 설치됩니다. 패킷에 vlan_tci=0x0001/0x0fff(VLAN id = 1)이고 dl_dst=fa:16:3e:4a:10:2b인지 확인합니다. 만약 매칭된다면 VLAN ID를 제거하고 터널 ID 를 0x2로 로드하여 vxlan-c0a80202(2)로 패킷으 전송합니다.
- 15 : 위의 12번, 13번, 14번 룰에 모두 매칭되지 않는 패킷은 **table 22**로 전송합니다.

### Table 21

```bash
16- cookie=0xbb7b3cdd52626a01, duration=9917.985s, table=21, n_packets=1, n_bytes=42, idle_age=9913, priority=1,arp,dl_vlan=1,arp_tpa=192.168.111.1 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],mod_dl_src:fa:16:3e:0b:cf:10,load:0x2->NXM_OF_ARP_OP[],move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],move:NXM_OF_ARP_SPA[]->NXM_OF_ARP_TPA[],load:0xfa163e0bcf10->NXM_NX_ARP_SHA[],load:0xc0a86f01->NXM_OF_ARP_SPA[],IN_PORT
17- cookie=0xbb7b3cdd52626a01, duration=9917.984s, table=21, n_packets=0, n_bytes=0, idle_age=9917, priority=1,arp,dl_vlan=1,arp_tpa=192.168.111.2 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],mod_dl_src:fa:16:3e:4a:10:2b,load:0x2->NXM_OF_ARP_OP[],move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],move:NXM_OF_ARP_SPA[]->NXM_OF_ARP_TPA[],load:0xfa163e4a102b->NXM_NX_ARP_SHA[],load:0xc0a86f02->NXM_OF_ARP_SPA[],IN_PORT
18- cookie=0xbb7b3cdd52626a01, duration=13003.028s, table=21, n_packets=0, n_bytes=0, idle_age=13003, priority=0 actions=resubmit(,22)
```

- 16, 17

  - 패킷이 ARP 패킷이고 특정 VLAN ID(예 : dl_vlan=1)인지 또한 특정 대상 IP 주소(예 : arp_tpa=192.168.111.1)인지 확인합니다.

  - 만약 맞으면 action은 다음과 같습니다.

  - `move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[]` : processing packet의 이더넷 dst를 플로우의 이더넷 src 주소로 이동

  - `mod_dl_src:fa:16:3e:0b:cf:10` : 이더넷 src 주소를 특정 값으로 변경

  - `load:0x2->NXM_OF_ARP_OP[]` : 터널 ID 0x2 로드

  - `move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[]` : processing packet의 arp src mac address를 플로우의 arp target mac address로 이동

  - `move:NXM_OF_ARP_SPA[]->NXM_OF_ARP_TPA[]` : processing packet의 arp src ip address를 플로우의 arp target ip address로 이동

  - `load:0xfa163e0bcf10->NXM_NX_ARP_SHA[]` : 0xfa163e0bcf10을 arp src mac address로 로드

  - `load:0xc0a86f01->NXM_OF_ARP_SPA[]` : 0xc0a86f01을 arp ip address로 로드

  - `IN_PORT` : **들어온 포트로 패킷 전송**

    > 16번과 17번의 flow는 호스트에 가까운 스위치가 arp mac address에 대한 응답함을 나타냅니다.

- 18 : 테이블 내의 다른 룰에 매칭되지 않는 패킷이면 **table 22**로 전송합니다.

### Table 22

```bash
19-  cookie=0xbb7b3cdd52626a01, duration=9917.956s, table=22, n_packets=10, n_bytes=1336, idle_age=9904, dl_vlan=1 actions=strip_vlan,set_tunnel:0x2,output:2
20-  cookie=0xbb7b3cdd52626a01, duration=13003.002s, table=22, n_packets=4, n_bytes=340, idle_age=9920, priority=0 actions=drop
```

- 19 : VLAN ID가 1인 패킷이면 vlan id를 없애고 0x2의 터널 id를 로드하고 **vxlan-c0a80202(2)**로 패킷을 전송합니다.
- 20 : 19번 룰에 매칭되지 않는 패킷은 모두 **drop**


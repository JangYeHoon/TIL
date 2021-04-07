# 29. OpenStack Setup using Horizon - 3

- tenant network위에서 실제 사용자가 VM 인스턴스를 생성하고 생성된 VM 인스턴스들 간에 네트워크를 구축



## VM Instance (Demo)

- 오픈스택의 테넌트 입장에서 VM을 생성

- 2개의 VM 인스턴스를 Demo tenant에 생성하고 실행

- 생성한 compute node의 VM위에 VM 인스턴스를 생성

- Compute - Instances - Launch Instance

  1. Details : 인스턴스 이름 설정
  2. Source : Glance에 업로드했던 VM 이미지를 선택
  3. Flavor : 생성했던 flavor 1을 선택
  4. Networks : 생성한 tenant network 선택

  ![image-20210407101704491](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210407101704491.png)

  ![image-20210407101746999](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210407101746999.png)



## Floating IP Allocation (Demo)

- 생성한 VM 인스턴스들이 외부와의 통신을 위해 사용하는 public ip인 Floating IP 설정

- Compute - Instances - Drop down the "Actions" tap - Associate Floating IP - Push "+" button in IP Address

- "Port to be associated" : 현재 선택된 VM 인스턴스의 네트워크 인터페이스 중에서 tenant1 네트워크와 연결된 IP가 할당된 인터페이스를 선택

  ![image-20210407102157596](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210407102157596.png)



## Port Forwarding Rules for Remote Access to VMs

- 오픈스택 외부에서 생성한 VM 접속

- 접속하기 위해서는 Floating IP 대역이 호스트 서버의 NAT 네트워크로 구현되어 있기 때문에 오픈스택 설치 후에 대시보드에 접근하기 위해서 Virtual Box를 이용해서 PortFowarding Rule을 설정한 것 처럼 동일하게 VM 인스턴스들에 대해서도 룰 추가

  ![image-20210407102439082](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210407102439082.png)



## OpenStack Setup using Horizon

- 구성완료 모습

  ![image-20210407102608923](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210407102608923.png)



## SSH Connection to VM Instances

- Virtual Box가 구동되는 호스트 서버를 클라이언트 머신이라고 하고 접근

- 생성한 Key Pairs에서 다운로드한 private key를 이용해 접근

  - demo.pem

  ```bash
  $ sudo chmod 600 demo.pem
  $ sudo ssh -i demo.pem -p 2222 cirros@{HOST_IP}
  $ ip a
  $ sudo ssh -i demo.pem -p 2223 cirros@{HOST_IP}
  $ ip a
  ```

  ![image-20210407103306349](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210407103306349.png)



## Networking test on VM instance

- ping test

  <img src="C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210407103608818.png" alt="image-20210407103608818" style="zoom:50%;" />



### Network Topology

- 지금까지 설정했던 내용이 표현됨

  ![image-20210407103847457](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210407103847457.png)
# 30. OpenStack Additional Setup - 1

## Environment Overview

![image-20210408162907248](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210408162907248.png)



## Add a New Compute Node

![image-20210408163015372](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210408163015372.png)

- Virtual Box 인터페이스

  - File - Import Appliance - select "compute1.ova"

- Rename compute1 - compute2

- Check "Reinitialize the MAC address of all network cards"

  ![image-20210408163219359](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210408163219359.png)

- compute2 노드의 VM을 시작하고 login

  - USER: root, PW: stack

- 네트워크 인터페이스 설정

  - eth0 : 192.168.11.22

  - eth1 : 192.168.56.22

  - eth2 : 10.10.1.22

    ![image-20210408163436714](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210408163436714.png)

- 호스트 서버에서 해당 VM으로 ssh 접근

  - 컨트롤러 노드와 compute1 노드랑 연결확인

    ![image-20210408163514339](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210408163514339.png)

- compute2 노드에도 오픈스택을 설치하기 위해 기본적인 설정

  ```bash
  $ service NetworkManager stop
  $ service firewalld stop
  $ setenforce 0
  $ hostnamectl set-hostname compute2.openstack && su -
  ```

  ![image-20210408163609680](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210408163609680.png)

- 컨트롤러 노드에서 compute2 노드에도 오픈스택의 컴퓨트 노드로 동작하게 오픈스택 재설치

  ```bash
  $ cp packstack-sample-anser.txt packstack-adding-compute.txt
  $ vim multi-nodes-2computes.txt
  $ packstack --answer-file=packstack-adding-compute.txt
  ```

  ![image-20210408163752729](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210408163752729.png)

- 설치 완료 모습

  ![image-20210408164057084](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210408164057084.png)

  ![image-20210408164241571](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210408164241571.png)

  ![image-20210408164310844](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210408164310844.png)



### Compute Node 추가 모습

![image-20210408164418928](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210408164418928.png)
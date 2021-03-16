# 25. OpenStack Installation

## Environment Overview

​	![image-20210316100758024](images/image-20210316100758024.png)

### Preparation

- Host machine
  - 12 core, 24 GB memory, 1TB HDD
  - X11 forwarding support for VirtualBox GUI in Linux
- Linux shell client
  - 호스트 머신에 접속하거나 VM위에서 설치될 컨트롤 노드, 컴퓨트 노드에 접속하기 위한 목적
  - Window : Putty (+ Xming), Xshell (+ Xmanager)
  - Mac OS : Xquartz
- VirtualBox
  - 호스트 머신에 설치되며 가장 최근 버전 설치
- VM Images
  - 강의에서 제공되는 VM
  - CentOS 7.3
  - Controller node : 2 vCPU, 4GB memory, 10GB HDD
  - Compute node : 4 vCPU, 4GB memory, 10GB HDD
  - 사용하는 호스트 서버의 자원이 충분하다면 이것보다 더 많은 리소스를 할당



### VM Image Import

- Virtual Box에 Controller VM, Compute VM 차례대로 임포트
- Reinitialize the MAC address of all network cards 선택



### NAT Network Setup

- File - Preferences - Network - NAT Networks

- Add a new NAT network and edit it

- Uncheck "Supports DHCP"

  ![image-20210316124903084](images/image-20210316124903084.png)



### Host-only Networks Setup

- Create two host-only networks

  - vboxnet0 (192..168.56.0/24) - 오픈스택 설치 이후에 VM 인스턴스들 간에 통신을 위한 커뮤니케이션 링크로써 게스트 네트워크 구성(VLAN)
  - vboxnet1 (10.10.1.0/24) - 오픈스택과는 별개로 호스트서버에서 컨트롤 노드와 컴퓨트 노드의 제어를 하기 위한 SSH connection을 위한 목적

- File - Preferences - Network - Host-only Networks

- Edit the selected network

- Disable DHCP server

  <img src="images/image-20210316125205907.png" alt="image-20210316125205907" style="zoom:50%;" />



### VM Network Setting

- Machine - Settings - Network

- Check the adapter 1~3 of the given two VMs

  - Adapter 1 : NAT Network (192.168.11.0/24), Adapter Type (virtio-net), Promiscuous Mode(Allow All, only for Controller VM)

    ![image-20210316125436586](images/image-20210316125436586.png)

  - Adapter 2 : Host-only Adapter (vboxnet0, 192.168.56.0/24), Adapter Type(virtio-net), Promiscuous Mode (Allow All for both nodes)

    ![image-20210316125532877](images/image-20210316125532877.png)

  - Adapter 3 : Host-only Adapter (vboxnet1, 10.10.1.0/24), Adapter Type (virtio-net)

    ![image-20210316125616477](images/image-20210316125616477.png)
# 27. OpenStack Setup using Horizon - 1

## VM Image (admin)

- VM의 이미지들은 등록이 되면 오픈스택에 Glance 서비스에 등록

- 대시보드에서 설정

  - System-Images-Create Image

- Image Source

  - http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img

  <img src="images\image-20210402102227040.png" alt="image-20210402102227040" style="zoom:50%;" />



## VM Flavor (admin)

- VM 인스턴스를 위한 리소스 명세서

- System-Flavors-Create Flavor

- Flavor를 이용해서 생성되는 VM이 cpu, 메모리, 디스크 등의 자원 값을 지정

  ![image-20210402102349830](images\image-20210402102349830.png)



## Project (admin)

- 프로젝트는 OpenStack 서비스를 제공하는 서비스 제공자가 사용자들에게 고유의 OpenStack 네임 스페이스를 제공

  - 네임 스페이스가 제공되는 단위가 프로젝트
  - 클라우드의 테넌트

- Identity-Projects-Create Project

  <img src="C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210402102649266.png" alt="image-20210402102649266" style="zoom:50%;" />

  <img src="C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210402102709029.png" alt="image-20210402102709029" style="zoom:50%;" />



## Provider Network (admin)

- 프로바이더는 오픈스택 환경을 구성하는 서비스 프로바이더에 의해서 이미 구축이 되어있는 인프라스트럭쳐 네트워크 혹은 언더레이 네트워크

  - VirtualBox NAT Network (192.168.11.0/24)
  - 내부적인 구현이 되어 있는데 이것을 오픈스택 수준에서 프로바이더 네트워크라는 이름으로 불림

- System-Networks-Create Network

  - Flat과 extnet은 Packstack에서 설정함

  ![image-20210402103022704](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210402103022704.png)



## Floating IP Pool (admin)

- Floating IP란 오픈스택 환경에서 생성되는 VM 인스턴스들이 외부와 통신을 하기 위한 오픈스택에서 사용하는 퍼블릭 IP

- 프로바이더 네트워크에 할당된 IP 대역에서 그 일부를 Floating IP를 위한 Pool로써 활용 가능

  - 프로바이더 네트워크의 서브넷 생성

- System-Networks-"provider network"-Create Subnet

  ![image-20210402103851867](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210402103851867.png)



## Floating IP

- 일종의 Connectivity를 제공하는 수단

- External -- OpenStack Provider network -- Virtual router (NAT) -- VM

  ![image-20210402104906517](C:\Users\jkjk1\바탕 화면\TIL\SDN&NFV\images\image-20210402104906517.png)
## GNS3 2.2.25 설치

### GNS3 다운로드

- https://www.gns3.com/software/download 에서 Free Download 버튼을 눌러 자신의 운영체제에 맞는 gns3 설치 파일을 받습니다.
  - 로그인 필요
- 설치 파일을 실행하여 설치를 진행합니다.
  - 모두 다음을 눌러 설치
- 다음 창이 나오면 사용하는 email 입력

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_1.jpg)

- 다음 창에서는 No를 선택하고 다음을 클릭합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_3.jpg)

- 완료

### GNS3 VM 설정

- GNS3 설치가 완료되어 실행되면 아래와 같은 창이 나오는데 해당 실습에서는 virtual machine을 사용하기 때문에 맨 위의 `Run appliances in a virtual machine` 을 선택합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_4.jpg)

- 다음 화면에서는 기본으로 입력되어 있는 설정을 사용하기 때문에 그대로 두고 다음으로 넘어갑니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_5.jpg)

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_6.jpg)

- 만약 Virtual Machine인 VMware나 VirtualBox가 설치되어 있지 않으면 아래와 같은 오류가 발생합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_7.jpg)

- 해당 오류가 발생하신 분들은 아래 단계에서 VirtualBox를 설치하고 다음으로 넘어갑니다.

### VirtualBox 설치

- https://www.virtualbox.org/ 에서 VirtualBox 설치 파일을 다운받습니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_8.jpg)

- 다음을 선택하고 설치를 진행하는데 설치는 모두 다음을 선택하여 진행합니다.
- 설치가 완료되면 다음과 같은 화면이 나오는데 여기서 가져오기 버튼을 클릭합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_9.jpg)

- 잠시 설치를 멈추고 먼저 https://www.gns3.com/software/download-vm 에서 VirtualBox를 선택하여 다운로드합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_11.jpg)

- 다시 VirtualBox로 돌아와 오른쪽의 폴더를 선택합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_10.jpg)

- 위에서 받은 VirtualBox이미지를 선택하여 가져옵니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_12.jpg)

- 이제 모두 다음을 눌러 VM 생성을 완료합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_13.jpg)

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_14.jpg)

- 생성된 VM을 선택하고 시작 버튼을 클릭해 실행합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_15.jpg)

- 실행 중에 아래와 같은 네트워크 인터페이스 설정 오류가 발생하면 `네트워크 설정 변경` 을 선택합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_16.jpg)

- 호스트 전용 어댑터, VirtualBox Host-Only Ethernet Adapter를 선택하여 아래와 같이 설정하고 확인을 눌러 완료합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_17.jpg)

- gns3 vm 실행이 완료되었습니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_18.jpg)

### GNS3 VM 설정 완료

- 다시 GNS3로 돌아와 오류난 창을 닫고 GNS3를 재실행합니다.
- 실행된 GNS3에서 위의 메뉴의 Edit-Preferences를 선택합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_22.jpg)

- 아래와 같은 화면에서 왼쪽에 VirtualBox를 선택하고 Virtualization engine에서 VirtualBox를 선택하고 아래 Refresh 버튼을 클릭해 설정을 완료하고 OK를 눌러 종료합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_20.jpg)

- 그럼 GNS3 초기화면으로 돌아오고 오른쪽에 GNS3 VM 서버에 초록불이 생기면 오류없이 설정이 완료된 것입니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_21.jpg)

이렇게 GNS3 설치와 VM 설정을 모두 마쳤습니다.

다음에는 실습을 위한 스위치와 라우터를 추가하기 위해 IOS와 IOU/IOL 설정 방법을 알려드리겠습니다.
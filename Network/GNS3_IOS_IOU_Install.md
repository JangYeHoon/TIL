## GNS3 IOS/IOU 설정

- GNS3 2.2.25 version
- IOS와 IOU 이미지는 구글에서 검색해서 다운로드 받으시면 됩니다.

### Cisco IOS 설치

- 설치된 gns3에서 Edit - References 선택

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_22-16505466290161.jpg)

- 왼쪽 메뉴에서 Dynamips - IOS routers - New 클릭

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_20-16505466534433.jpg)

- `Run this IOS router on the GNS3 VM` 선택하고 Next

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_23.jpg)

- Browse를 눌러 다운로드한 IOS 이미지 선택

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_24.jpg)

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_25.jpg)

- 아래 그림처럼 설정하고 모두 Next

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_27.jpg)

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_28.jpg)

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_29.jpg)

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_30.jpg)

- 아래에서 Idle-PC finder 선택

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_31-165054676621213.jpg)

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_32.jpg)

- 모두 완료하면 다음과 같이 IOS이미지의 라우터가 생성됩니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_33.jpg)

- gns3 초기 화면에서 오른쪽에 라우터 모양을 선택하면 아래와 같이 라우터가 추가된 것을 확인하실 수 있습니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_34.jpg)

### Switch 사용을 위한 IOU/IOL 추가

- router IOS와 같이 Edit - References 클릭
- IOS on UNIX - IOU Devices - Next 클릭

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_35.jpg)

- 이름은 알아서 적절한 이름으로 입력하고 Type은 L2 image를 선택하고 다운로드한 이미지를 불러옵니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_36.jpg)

- Finish를 눌러 L2 Switch에 대한 설정을 완료합니다.
- L3 이미지는 아래와 같이 설정하시면 됩니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_37.jpg)

### IOS 장비 라이센스 추가

- 현재 설치한 switch는 라이센스 설정이 안되어 동작을 안하기 때문에 라이센스를 추가하여 설정을 완료합니다.
- gns3를 실행하면 VirtualBox의 VM도 실행되는데 해당 VM을 여시면 아래와 같은 화면입니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_2.png)

- 위에서 Enter를 누르면 아래와 같이 메뉴를 선택하는 화면이 나오는데 방향키 위, 아래로 움직여 Shell로 두고 Enter를 입력합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_3.png)

- 그리고 다음 명령을 입력하여 라이센스를 받습니다.

```bash
$ sudo gns3restore
$ wget http://www.ipvanquish.com/download/CiscoIOUKeygen3f.py
$ python3 CiscoIOUKeygen3f.py
$ vi iourc.txt
[license]
gns3vm = 73635fd3b0a13ad0;		// 정확하게 기억은 안나지만 만약 입력이 되어 있는 경우에는 바꾸지 말고 바로 종료합니다.
:wq를 입력하여 종료
```

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_1.png)

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_38.jpg)

- 다시 gns3로 돌아와 Edit - References에서 `IOS on UNIX`를 클릭하고 위의 값을 똑같이 입력합니다.

![img](images/%EC%9D%B4%EB%AF%B8%EC%A7%80_40.jpg)

- 이렇게 하시면 모든 설정이 안료되어 스위치를 사용할 수 있습니다.
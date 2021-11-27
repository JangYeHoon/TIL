# rhcsa

### DNS

- DNS 주소 확인 : `nslookup`
- DNS 주소 변경 : `/etc/resolve.conf`
  - `nameserver 8.8.8.8`
- 도메인의 IP를 찾을 때가장 먼저 조사하는 파일(DNS)
  - `/etc/hosts`

### SElinux

- Security Enhanced Linux의 약자로 커널을 보호하기 위한 도구
- 3가지 상태 존제
  - enforcing : 강제
  - permissive : 허용
  - disabled : 비활성화
- 현재 상태 확인 명령어 : `getenforce`
- 상태 변경 명령어 : `setenforce <상태값>`(상태값 0 : Permissive, 1 : Enforcing)
- 영구적인 변경
  - `/etc/selinux/config` 혹은 `/etc/sysconfig/selinux`에서 변경

### Firewall

- 사용 가능한 모든 서비스/포트 목록 출력
  - `firewall-cmd --list-all`
- 서비스나 포트 추가
  - `firewall-cmd --add-service=ftp`
  - `firewall-cmd --add-port=21/tcp`
- 서비스나 포트 제거
  - `firewall-cmd --remove-service=ftp`
  - `firewall-cmd --remove-port=21/tcp`

### 아파치 설치

- `yum -y install httpd`
- `systemctl restart httpd`
- `systemctl enable httpd`
- `firewall-cmd --permanent --add-service=http`
- `/var/www/html`로 이동해서 `index.html` 파일 변경

### ps 명령어

- 현재 실행중인 프로세스 목록 출력
- 옵션
  - `-e` : 모든 프로세스 출력
  - `-f` : UID, PID 출력
  - `-l` : 긴 포맷으로 출력
  - `-p` : 특정 PID 프로세스 출력
  - `-u` : 특정 사용자의 프로세스 출력

- 프로세스를 죽이는 명령어 : `kill [options] <pid>`

### SSH 접속

- sshd 설치 확인

  - `ps -ef | grep openssh-server`

- ssh 설치

  - ```sh
    $ yum install -y openssh-server
    $ systemctl restart sshd
    $ systemctl enable sshd
    ```

- `vi /etc/ssh/sshd_config`

  - Port 22 주석 제거

- 방화벽 해제

  - `firewall-cmd --zone=public --add-port=22/tcp --permanent`

- ssh 재시작

  - `systemctl restart sshd`

### 런레벨 변경

- 현재 런레벨 확인
  - `who -r`
- 런레벨 변경
  - `systemctl isolate runlevel4`
- 런레벨
  - 0 runlevel0.target poweroff.target : 시스템 셧다운
  - 1 runlevel1.target rescue.target : 복구 쉘
  - 2 runlevel2.target multi-user.target 커맨드 환경의 다중 사용자모드 CLI
  - 3 runlevel3.target multi-user.target 커맨드 환경의 다중 사용자모드 CLI
  - 4 runlevel4.target multi-user.target 커맨드 환경의 다중 사용자모드 CLI
  - 5 runlevel5.target graphical.target : 그래픽 환경의 다중 사용자모드
  - 6 runlevel6.target reboot.target : 재부팅

### rpm, yum, repository

- RPM(Redhat Package Manager)
  - 레드햇에서 만든 프로그램 설치 패키지
  - 자동으로 업그레이드
  - 설치 : `rpm -Uvh <패키지>`
  - 삭제 : `rpm -e <패키지>`
  - 설치확인 : `rpm -qa <패키지>`
  - 상세정보 : `rpm -qi <패키지>`
  - 의존성 문제가 존재
- YUM(Yellodog Updater Modified)
  - RPM 기반의 시스템을 위한 자동 업데이터 및 패키지 설치/제거 프로그램
  - RPM과 달리 인터넷 사용이 필수여서 패키지간 의존성 문제 해결 가능
  - 설치 : `yum install <패키지>`
  - 삭제 : `yum remove <패키지>`
  - 업그레이드 : `yum update <패키지>`
  - 목록 : `yum list <패키지>`
- Yum Repository
  - 패키지를 모아놓은 저장소
  - yum을 통해 패키지를 설치할때 활성화된 Yum Repository에 패키지가 없으면 설치 불가

### Root 계정 비밀번호 복구

- 부팅시 OS 버전 선택하는 곳에서 `e` 입력

- `linux 16`으로 시작하는 줄을 찾아`UTF-8` 뒤에 `rd.brak` 입력

- `ctrl + x`로 저장

- 부팅이 되면 다음 입력

- ```shell
  $ mount | grep -w '/sysroot'
  $ mount -o remount, rw /sysroot
  $ chroot /sysroot
  ```

- `passwd` 입력하고 설정할 비밀번호 입력

  ```sh
  $ touch / .autorelabel
  $ exit
  $ exit
  ```

### 스케줄링 명령어

- `at`, `crontab`

#### at

- 1회성 예약 작업

- `at <설정 시간>`

  - 3분 뒤에 ls 명령어 실행

  - ```sh
    $ at now +3min
    at> ls
    at> <EOT>
    ```

- 예약 작업 목록 출력

  - `at -l`
  - `atq`

### Disk 관련 명령어

- 디스크 확인 : `fdisk -l`
- 파티션 생성
  - `fdisk /dev/<디스크>`
  - `Command(m for help) : n` 새로운 파티션 추가 의미인 n 입력
  - partition type을 `p`(primary 기본값) 입력
  - partition number `1` 입력
  - sector에 설정할 크기 입력, ex) `4G`
- 스왑 메모리 생성하고 마운트
  - ext4파일로 포맷을 먼저 실행 : `mkfs.ext4 /dev/sdb1`
  - mount 실행 : `mount /dev/sdb1 /disk1`
  - 마운트되었는지 확인 : `df -Th`, `lsblk`
  - `mount` 명령어로 마운트를 하면 재부팅시 마운트가 **해제**됨
- 영구적으로 마운트 하는 방법
  - `vi /etc/fstab`
  - `/dev/sdb1 /disk1 ext4 defaults 1 1` 양식에 맞게 편집
  - 오타가 발생하면 부팅이 되지 않기 때문에 주의 필요

- PVS - VGS - LVM 실습

  - 디스크의 파티션을 나누고 LVM으로 설정

  - pvcreate 명령어를 통해 물리 볼륨 생성

    - `pvcreate /dev/sdb{1..4}`

  - 물리 볼륨 생성 확인 : `pvscan`

  - 볼륨 그룹 생성

    - `vgcreate <group_name> <partition_name>`
    - `vgcreate vg0 /dev/sdb2 /dev/sdb4`

  - 볼륨 그룹 확인 : `vgdisplay`

  - 논리 볼륨 생성

    - `lvcreate <vg_name> -n <lv_name> -L <size>`
    - `lvcreate vg0 -n lv01 -L 2G`
    - `lvcreate vg0 -l 100%FREE -n lv03`

  - 논리 볼륨 확인 : `lvscan`

  - 포맷과 마운트 실행

    ```sh
    $ mkfs.xfs /dev/vg0/lv02
    $ mount /dev/vg0/lv01 /mnt/disk2
    ```

  - 설정 확인 : `lsblk`

- PVS - VGS - LVM 볼륨값 수정

  - `pvcreate /dev/sdb3`
  - `vgextend vg0 /dev/sdb3` : sdb3을 vg0 볼륨그룹에 새로 추가
  - 논리 볼륨 확장
    - `lvextend <lv_경로> -L <size>`
    - `lvextend -L 5G /dev/vg0/lv01`
  - 설정 확인 : `lsblk`

### NFS

- 리눅스, 유닉스 시스템끼리 저장공간을 공유하게 해주는 서비스

- 서버의 리소스를 클라이언트 상에서 마치 자신의 리소스를 사용하는 것처럼 사용할 수 있도록 제공하고 네트워크가 가능한 곳이면 리눅스, 유닉스에서 NFS를 사용하여 파일 시스템 공유가 가능

- NFS 설치 확인 : `rpm -qa nfs-utils`

- NFS 설치 : `yum -y install nfs-utils`

- 공유할 디렉토리 설정

  ```sh
  $ vi /etc/exports nfs
  /share 10.0.2.*(rw,sync)
  # share 디렉토리에 10.0.2.* IP 주소는 읽기와 쓰기에 대한 권한과 실시간 동기화를 부여
  $ echo 'helloworld' > /share/hello.txt  # 확인을 위한 임의의 파일
  ```

- NFS 설정

  ```sh
  $ systemctl restart nfs-server
  $ systemctl enable nfs-server
  $ exportfs -v
  # 방화벽 설정
  $ firewall-cmd --permanent --add-service=nfs
  $ firewall-cmd --permanent --add-service=rpc-bind
  $ firewall-cmd --permanent --add-service=mountd
  $ firewall-cmd --reload
  ```

- 클라이언트에서 서버 공유 폴더 마운트

  ```sh
  $ showmount -e 10.0.2.9   # 해당 서버의 공유 디렉토리를 확인하는 명령어
  $ mount -t nfs 10.0.2.9:/share /nfs_share     # 서버의 디렉토리와 클라이언트 디렉토리 마운트
  ```

### AutoFS

- 자동 마운트 데몬의 작동을 제어하는 프로그램

  - 자동 마운트 데몬은 자동으로 파일 시스템을 마운트하고 파일 시스템이 사용되지 않고 일정 시간이 흐르면 자동으로 언마운트

- AutoFS와 맵(MAP)

  - 맵은 AutoFS 모듈이 동작하는데 필요한 정보가 저장된 설정 파일
  - 종류
    - master map : AutoFS 파일시스템에서 가장 기준이 되는맵, `/etc/auto.master.d/`에서 `이름.autofs`파일에 설정 저장, direct map이나 indirect map의 이름과 포인터를 정의하는 역할
    - direct map(직접 맵 마운트) : `/etc/auto.이름`으로 관습적으로 파일을 만들어서 설정 저장, 맵 내부의 마운트 포인터로 절대 경로명을 사용
    - indirect map(간접 맵 마운트) : `/etc/auto.이름`으로 관습적으로 파일을 만들어서 설정 저장, 맵 내부의 마운트 로 상대 경로명을 사용

- direct map 사용 방법

  - `yum install -y autofs` autofs 다운로드

  - ```sh
    $ vi /etc/auto.master.d/direct.autofs
    /- /etc/auto.direct
    $ vi /etc/auto.direct
    /nfs_share -rw,sync 10.0.2.9:/share
    ```

  - `systemctl start autofs.service`

- indirect map 사용 방법

  - ```sh
    $ vi /etc/auto.master.d/indirect.autofs
    /indirect /etc/auto.indirect
    $ vi /etc/auto.indirect
    share -rw,sync 10.0.2.9:/share
    ```

  - `systemctl restart autofs.service`

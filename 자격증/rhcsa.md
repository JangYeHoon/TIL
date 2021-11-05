# rhcsa

### 기본 명령어

- `cal` : 달력
- `date`
- `free` : 메모리
- `lscpu` : cpu 상태 확인
- `df`, `lsblk` : 디스크 사용량 확인
- `du` : 디렉토리 디스크 사용량
- `whatis` : 명령어에 대한 설명
- `type`
- `last` : 로그인 및 재부팅 로그 출력
- `w` : 서버에 접속한 사용자 정보 
- `hostname` : 호스트 이름 출력

### 유저 및 그룹 관련 명령어

#### 유저 관련

- 유저 생성 : `useradd <이름>`
  - 옵션
    - -d : 홈디레토리 경로 지정
    - -u : uid 지정
    - -g : gid 지정
    - -G : 보조 그룹 지정
    - -s : 쉘 지정
    - -c : 주석 추가
    - -D : 기본 설장 값 확인, 기본 설정은 `/etc/default/useradd`에 존재

- `/etc/skel` : 계정 생성시 제공되는 파일 및 디렉토리가 들어 있는 파일

- 생성한 유저 정보 확인 : `/etc/passwd`
- 유저 삭제 : `userdel <이름>`
- 유저와 홈디렉토리 모두 삭제 : `userdel -r <이름>`
- 사용자 패스워드 만기일 수정 : `chage <이름>`
  - 옵션
    - -d : 패스워드를 마지막으로 변경한 일자
    - -E : 계정을 사용할 수 있는 기간
    - -i : 잠금날짜
    - -m : 패스워드 변경할 수 있는 최소 일수
    - -M : 마지막 변경 이후의 패스워드 유효 일수
    - -W : 패스워드 만료기간 몇일 전에 안내메시지 출력
    - -l : 현재 사용자의 패스워드 만기 정보 보여줌

#### 그룹 관련

- 그룹 생성 : `groupadd <이름>`
- 생성한 그룹 정보 확인 : `/etc/group`
- 그룹에 유저 추가 : `gpasswd -a <유저명> <그룹명>`
- 그룹 삭제 : `groupdel <이름>`

### 퍼미션 및 소유자

- r, w, x, s(SetUID, SetGID 권한), t(Sticky Bit 권한)
- r=4, w=2,x=1
- s=4,s=2,t=1
- 권한 변경 방법
  - `chmod ugo+rwx file`, `chmod u-r file`
  - `chmod 777 file`, `chmod 744 file`
- 소유자 변경 방법
  - `sudo chown user01 file`
  - `sudo chgrp group01 file`
- 디렉토리와 그 안에 들어있는 모든 파일의 소유자 변경
  - `sudo chown -R user01 dir/`

- 파일이나 디렉토리가 생성됨가 동시에 권한 지정

  - `umask`

  - | umask | 파일 | 디렉토리 |
    | ----- | ---- | -------- |
    | 000   | 666  | 777      |
    | 001   | 665  | 776      |
    | 002   | 664  | 775      |
    | 022   | 644  | 755      |

  - umask 설정 : `umask 022`

### DNS

- DNS 주소 확인 : `nslookup`
- DNS 주소 변경 : `/etc/resolve.conf`
  - `nameserver 8.8.8.8`
- 도메인의 IP를 찾을 때가장 먼저 조사하는 파일(DNS)
  - `/etc/hosts`

### 리눅스 특수 권한

- SetUID

  - 일시적으로 파일 소유자의 권한을 얻어 실행하 수 있도록 함

  - 퍼미션 4

  - ```sh
    $ touch setuid
    $ chmod 4644 setuid
    ```

- SetGID

  - 파일을 실행할 때 일시적으로 파일 소유 그룹의 권한을 얻어 실행

  - 퍼미션 2

  - ```sh
    $ touch setgid
    $ chmod 2644 setgid
    ```

- Sticky Bit

  - 파일을 누구나 생성할 수 있지만 삭제는 본인과 관리자만 가능

  - 퍼미션 1

  - ```sh
    $ touch stickybit
    $ chmod 1644 stickybit
    ```


### 파일 속성 관련 명령어

- `lsattr <옵션> <파일명>`
  - 파일 속성 확인 명령어
  - `A` : atime 레코드가 수정되지 않음, `a` : 파일을 쓰는 것은 가능하나 삭제 불가, `c` : 파일이 커널에 의해 자동적으로 압축, `D` : 디렉토리 변경 사항이 디스크에 동기식으로 저장됨, `d` : dump 프로그램을 실행 중일 때에는 해당 파일이 백업되지 않음
  - `E` : 압축된 데이터가 오류를 가지고 있음, `e` : 파일이 디스크 블록에 매핑될 때 Extents를 사용, `I` : 디렉토리가 htree로 인덱싱 중, `h` : 파일을 저장할 때 블록사이즈 단위로 저장, `i` : 파일을 수정할 수 없음, `j` : 파일에 데이터를 쓰기 전에 ext3 journal에 먼저 씀
  - `s` : 파일을 제거했을 때 해당 블록은 zeroed되고 디스크에 쓰여짐, `S` : 파일의 변경 사항이 디스크에 동기식으로 저장됨, `T` : 가장 상위 디렉토리, `t` : 파일에 Partial block fragmentation 발생하지 않음, `u` : 파일이 삭제되더라도 내용은 저장되어 있어 복구 가능, `x` : 압축된 파일의 실제 내용을 직접 접근 가능, `Z` : 압축된 데이터가 손상됨
- `chattr <옵션> <+/-/=> <mode> <파일명>`
  - 파일 속성 부여 명령어
  - `chattr +i test`

### 사용자 sudo 권한 부여

- `/etc/sudoers` 수정
  - 읽기 권한밖에 없기 때문에 권한을 수정하거나 `visudo -f /etc/sudoers`로 수정
  - `Allows people in group wheel to run all command`아래에 입력
    - `<계정명> ALL=(ALL) ALL`

### Access Control List

- ACL은 파일과 디렉토리의 확장 속성 중 하나로 `chmod` 보다 다양한 권한을 부여할 수 있음
  - ACL은 특정 사용자에게만 권한 부여가 가능
- `setfacl -m u:<user_name>:<부여할 권한> <file_name>`
- `getfacl <file_name>`을 통해 정보 확인

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

#### crontab

- 작업 리스트 확인
  - `crontab -l`
- 작업 수정
  - `crontab -e`
- 작업 삭제
  - `crontab -r`
- 작업을 지정하는 사용자
  - `crontab -u 사용자명`
- `* * * * * /bin/echo "hello"` 입력하면 매 분마다 echo “hello” 명령어 출력
  - `* * * * *`는 각각 분, 시간, 일, 월, 요일을 나타냄
    - 요일에서 0과 7은 일요일 1~6은 월요일~토요일


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

- 유저 생성 : `useradd <이름>`
- 생성한 유저 정보 확인 : `/etc/passwd`
- 그룹 생성 : `groupadd <이름>`
- 생성한 그룹 정보 확인 : `/etc/group`
- 그룹에 유저 추가 : `gpasswd -a <유저명> <그룹명>`
- 유저 삭제 : `userdel <이름>`
- 유저와 홈디렉토리 모두 삭제 : `userdel -r <이름>`
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






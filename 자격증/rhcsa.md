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
- `setfacl -m u: <user_name>:<부여할 권한> <file_name>`

- `getfacl <file_name>`을 통해 정보 확인

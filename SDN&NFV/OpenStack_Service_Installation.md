# OpenStack Service Installation



## Keystone Installation

### Identity Service 개요

- 인증, 권한 부여 및 서비스 카탈로그를 관리하기 위한 서비스
- 사용자가 상호 작용하는 첫 번째 서비스로 인증을 통해 사용자는 자신의 ID를 사용해 다른 OpenStack 서비스에 접근할 수 있습니다.

- 다른 OpenStack 서비스는 Identity 서비스를 활용하여 사용자가 접근 권한이 있는 사용자인지 확인하고 배포 내에서 다른 서비스가 어디에 있는지 확인합니다.
- 사용자와 서비스들은 Identity 서비스에서 관리하는 서비스 카탈로그를 사용하여 다른 서비스들을 찾고 사용할 수 있습니다.
  - 서비스 카탈로그 : OpenStack 배포에서 사용 가능한 서비스 모음

#### Identity 서비스 구성 요소

- **Server**
  - RESTful 인터페이스를 사용하여 인증 및 권한 부여 서비스를 제공합니다.
- **Drivers**
  - Driver or Service Backend는 Server와 통합됩니다.
  - OpenStack 외부 저장소의 ID 정보에 접근하는데 사용되며 OpenStack이 배포된 인프라(ex. SQL 데이터베이스 또는 LDAP 서버)에 이미 존재할 수 있습니다.
- **Modules**
  - Identity 서비스를 사용하는 OpenStack 구성 요소의 주소 공간에서 실행됩니다.
  - 모듈은 서비스 요청을 가로채 사용자 자격 증명을 추출한 다음 인증을 위해 서버로 보냅니다.
  - 미들웨어 모듈과 OpenStack 구성 요소 간의 통합은 Python 웹 서버 게이트웨이 인터페이스를 사용합니다.



### 설치 및 구성

- 확장성을 위해 Fernet 토큰과 Apache HTTP 서버를 배포하여 요청을 처리합니다.
- **Controller Node**에서 진행합니다.

#### 전제 조건

- 데이터베이스 액세스 클라이언트를 사용하여 root 사용자로 데이터베이스 서버에 연결

  `mysql -u root -p`

- Keystone Database 생성

  `MariaDB [(none)]> CREATE DATABASE keystone;`

- Keystone Database에 접근 권한 부여

  ```bash
  # KEYSTONE_DBPASS를 적절한 암호로 변경
  # keystone 데이터베이스의 모든 스키마에 대해 keystone 계정의 로컬 접근 권한 허용
  MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
  IDENTIFIED BY 'KEYSTONE_DBPASS';
  # keystone 데이터베이스의 모든 스키마에 대해 keystone 계정의 외부 접근 권한 허용
  MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
  IDENTIFIED BY 'KEYSTONE_DBPASS';
  ```

- 데이터베이스 액세스 클라이언트 종료

  `exit`

#### 구성요소 설치 및 구성

- 필요한 패키지 설치

  `yum install openstack-keystone httpd mod_wsgi`

- `/etc/keystone/keystone.conf`  파일 수정

  - `[database]` 섹션에서 데이터베이스 액세스 구성

    ```bash
    ...
    [database]
    connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
    ...
    ```

    ![image-20210604151253990](images/image-20210604151253990.png)

  - `[token]` 섹션에서 Fernet 토큰 공급자를 구성

    ```bash
    ...
    [token]
    provider = fernet
    ...
    ```

    ![image-20210604151437726](images/image-20210604151437726.png)

- Identity 서비스 데이터베이스 내용 채우기

  `su -s /bin/sh -c "keystone-manage db_sync" keystone`

- Fernet 키 저장소 초기화

  ```sh
  root@controller$ keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone	# 저장소 초기화
  root@controller$ keystone-manage credential_setup --keystone-user keystone --keystone-group keystone	# 암호화
  ```

- Identity 서비스 부트 스트랩

  ```sh
  root@controller$ keystone-manage bootstrap-password ADMIN_PASS \
  	--bootstrap-admin-url http://controller:5000/v3/ \
  	--bootstrap-internal-url http://controller:5000/v3/ \
  	--bootstrap-public-url http://controller:5000/v3/ \
  	--bootstrap-region-id RegionOne
  ```

#### Apache HTTP 서버 구성

- `/etc/httpd/conf/httpd.conf` 파일 편집
  - `ServerName controller`
  - <img src="images/image-20210604152742996.png" alt="image-20210604152742996" style="zoom:67%;" />

- `/usr/share/keystone/wsgi-keystone.conf` 파일에 대한 링크 생성
  - `ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/`

#### 설치 완료

- Apache HTTP 서비스를 시작하고 시스템이 부팅될 때 시작되도록 구성

  ```sh
  root@controller$ systemctl enable httpd.service
  root@controller$ systemctl start httpd.service
  ```

- 관리 계정을 구성하기 위한 환경변수 설정

  - `vi admin-openrc`
  
  ```bash
  export OS_USERNAME=admin
  export OS_PASSWORD=ADMIN_PASS		# keystone-manage bootstrap에서 사용한 비밀번호
  export OS_PROJECT_NAME=admin
  export OS_USER_DOMAIN_NAME=Default
  export OS_PROJECT_DOMAIN_NAME=Default
  export OS_AUTH_URL=http://controller:5000/v3
  export OS_IDENTITY_API_VERSION=3
  ```
  
  - `. admin-openrc` 관리 계정 스크립트 실행
  
- admin 유저에 대한 검증

  - `openstack token issue`
  - ![image-20210607115550731](images/image-20210607115550731.png)

### 도메인, 프로젝트, 사용자 및 역할 생성

- keystone-manage  bootstrap 단계에서 기본 도메인이 생성되지만 새 도메인을 생성하는 방법
  - `openstack domain create --description "An Example Domain" example`
  - ![image-20210607115559005](images/image-20210607115559005.png)

- service 프로젝트 생성
  - `openstack project create --domain default --description "Service Project" service`
  - ![image-20210607132403249](images/image-20210607132403249.png)

- 관리자가 아닌 일반 작업은 권한이 없는 프로젝트 및 사용자를 사용해야 합니다.

  - 이를 위해 `myproject`와 `myuser`를 생성합니다.

  - `openstack project create --domain default --description "Demo Project" myproject`

  - ![image-20210607132617010](images/image-20210607132617010.png)

  - `openstack user create --domain default --password-prompt myuser`

  - ![image-20210607132827332](images/image-20210607132827332.png)

  - `openstack role create myrole`

  - ![image-20210607132910810](images/image-20210607132910810.png)

  - 프로젝트 및 사용자에 `myrole` 추가

  - `openstack role add --project myproject --user myuser myrole`

    > 해당 명령은 출력을 제공하지 않습니다.



### 작동 확인

- 임시 OS_AUTH_URL 및 OS_PASSWORD 환경변수를 설정 해제
  - `unset OS_AUTH_URL OS_PASSWORD`

- admin 사용자 인증 토큰 요청
  - `openstack --os-auth-rul http://controller:5000/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name admin --os-username admin token issue`
  - ![image-20210607133649639](images/image-20210607133649639.png)

- myuser 사용자 인증 토큰 요청
  - `openstack --os-auth-rul http://controller:5000/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name myproject --os-username myuser token issue`
  - ![image-20210607134950285](images/image-20210607134950285.png)



### OpenStack 클라이언트 환경 스크립트 생성

#### 스크립트 생성

- `admin` 및 `demo` 프로젝트와 사용자를 위한 클라이언트 환경 스크립트를 작성한다.

- `admin-openrc`파일을 생성 및 편집

  ```bash
  export OS_PROJECT_DOMAIN_NAME=Default
  export OS_USER_DOMAIN_NAME=Default
  export OS_PROJECT_NAME=admin
  export OS_USERNAME=admin
  export OS_PASSWORD=ADMIN_PASS		// admin 사용자 비밀번호로 변경
  export OS_AUTH_URL=http://controller:5000/v3
  export OS_IDENTITY_API_VERSION=3
  export OS_IMAGE_API_VERSION=2
  ```

- `demo-openrc` 파일 생성 및 편집

  ```bash
  export OS_PROJECT_DOMAIN_NAME=Default
  export OS_USER_DOMAIN_NAME=Default
  export OS_PROJECT_NAME=myproject
  export OS_USERNAME=myuser
  export OS_PASSWORD=DEMO_PASS
  export OS_AUTH_URL=http://controller:5000/v3
  export OS_IDENTITY_API_VERSION=3
  export OS_IMAGE_API_VERSION=2
  ```

#### 스크립트 사용

- 클라이언트를 특정 프로젝트 및 사용자로 실행하려면 실행하기 전에 연결된 클라이언트 환경스크립트를 로드하여 사용합니다.
- admin 사용자를 로드하려면 `admin-openrc`파일을 로드
  - `. admin-openrc`
- 인증 토큰 요청
  - `openstack token issue`



## Glance Installation

### Image Service 개요

- 사용자는 Image Service(Glance)를 통해 가상 머신 이미지를 검색 및 등록할 수 있습니다.
- 가상 머신 이미지 메타 데이터를 쿼리하고 실제 이미지를 검색할 수 있는 REST API를 제공합니다.
- 가상 머신 이미지는 간단한 파일 시스템부터 객체 스토리지 시스템에 이르기 까지 다양한 위치에 저장할 수 있습니다.
- 오픈스택 이미지 서비스는 캐싱을 지우너하기 위해 여러 periodic processes를 실행합니다.

#### 구성 요소

- **glance-api**
  - 이미지 검색 및 저장을 위한 Image API를 호출
- **glance-registry**
  - 이미지에 대한 메타 데이터를 저장, 처리 및 검색
  - 메타 데이터에는 크기 및 유형과 같은 항목이 포함
- **database**
  - 이미지 메타 데이터를 저장합니다.
  - 기본 설정에 따라 데이터베이스를 선택할 수 있습니다.
- **Storage repository for image files**
  - 일반 파일 시스템, Object Storage, RADOS 블록 장치, VMware 데이터 저장소 및 HTTP를 포함한 다양한 저장소 유형 지원
  - 일부 저장소는 읽기 전용
- **Metadata definition service**
  - 공급 업체, 관리자, 서비스 및 사용자가 고유한 사용자 지정 메타 데이터를 의미있게 정의하기 위한 공통 API
  - 메타 데이터는 이미지, 아티팩트, 볼륨, 플레이버 및 집계와 같은 다양한 유형의 리소스에서 사용



### 설치 및 구성

- 단순화를 위해 로컬 파일 시스템에 이미지를 저장합니다.
- **Controller Node**에서 진행합니다.

#### 전제 조건

- 데이터베이스 작성

  - 데이터 베이스 액세스 클라이언트를 사용하여 root 사용자로 데이터베이스 서버에 연결

    - `mysql -u root -p`

  - glance 데이터베이스 생성

    - `MariaDB [(none)]> CREATE DATABASE glance;`

  - glance 데이터베이스에 접근 권한 부여

    ```sh
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
    IDENTIFIED BY 'GLACE_DBPASS';
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
    IDENTIFIED BY 'GLANCE_DBPASS';
    ```

  - 설정 종료
  - ![image-20210607142515693](images/image-20210607142515693.png)

- admin 자격 증명 관리자 액세스

  - `. admin-openrc`

- Service credentials 생성

  - glance 사용자 생성
    - `openstack user create --domain default --password-prompt glance`
    - ![image-20210607143240916](images/image-20210607143240916.png)
  - glance 서비스에 admin role 추가
    - `openstack role add --project service --user glance admin`
  - glance service entity 생성
    - `openstack service create --name glance --description "OpenStack Image" image`
    - ![image-20210607143351705](images/image-20210607143351705.png)

- 이미지 서비스 API 엔드 포인트 생성

  ```sh
  root@controller$ openstack endpoint create --region RegionOne image public http://controller:9292
  root@controller$ openstack endpoint create --region RegionOne image internal http://controller:9292
  root@controller$ openstack endpoint create --region RegionOne image admin http://controller:9292
  ```

#### 구성 요소 설치 및 구성

- 패키지 설치

  - `yum install openstack-glance`

- `/etc/glance/glance-api.conf` 수정

  - `[database] ` 섹션에서 데이터베이스 액세스 구성

    ```bash
    [database]
    ...
    connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
    ```

    ![image-20210607152815673](images/image-20210607152815673.png)

  - `[keystone_authtoken]`와 `[paste_deploy]` 섹션 구성

    ```bash
    [keystone_authtoken]
    # ...
    www_authenticate_uri  = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = Default
    user_domain_name = Default
    project_name = service
    username = glance
    password = GLANCE_PASS
    
    [paste_deploy]
    # ...
    flavor = keystone
    ```

    ![image-20210607153424553](images/image-20210607153424553.png)

  - [glance_store] 섹션에 이미지 파일의 로컬 파일 시스템 저장소 위치 구성

    ```bash
    [glance_store]
    # ...
    stores = file,http
    default_store = file
    filesystem_store_datadir = /var/lib/glance/images/
    ```

    ![image-20210607153637039](images/image-20210607153637039.png)

- 이미지 서비스 데이터베이스 채움
  - `su -s /bin/sh -c "glance-manage db_sync" glance`

#### 설치 완료

- 이미지 서비스를 시작하고 시스템이 부팅될 때 시작되도록 구성합니다.

  ```sh
  root@controller$ systemctl enable openstack-glance-api.service
  root@controller$ systemctl start openstack-glance-api.service
  ```



### 작동 확인

- Linux 이미지인 CirrOS를 사용하여 이미지 서비스의 작동을 확인합니다.

- admin 자격 증명 관리자

  - `. admin-openrc`

- 소스 이미지 다운로드

  ```
  root@controller$ yum install wget
  root@controller$ wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
  ```

  <img src="images/image-20210607161915931.png" alt="image-20210607161915931" style="zoom:60%;" />

- 모든 프로젝트에서 접근할 수 있도록 QCOW2 디스크 형식, 베어 컨테이너 형식 및 공개 가시성을 사용하여 이미지 서비스에 이미지를 업로드

  - `glance image-create --name "cirros" --file cirros-0.4.0-x86-64-disk.img --disk-format qcow2 --container-format bare --visibility public`
  - 

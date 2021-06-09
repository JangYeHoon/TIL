# OpenStack Service Installation

- [Keystone Installation](#keystone-installation)
- [Glance Installation](#glance-installation)
- [Placement Installation](#placement-installation)



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

  ```sql
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

    ```sql
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
  - ![image-20210608114917384](images/image-20210608114917384.png)
  - ![image-20210608114938738](images/image-20210608114938738.png)

- 업로드 이미지 확인
  - `glance image-list`
  - ![image-20210608115157519](images/image-20210608115157519.png)



## Placement Installation

### Steps Overview

#### Deploy the API service

- Placement는 Apache, nginx 또는 기타 WSGI 지원 웹 서버에서 서비스를 실행하기 위한 placement-api WSGI 스크립트를 제공합니다.
- 표준 WSGI 스크립트인 placement-api는 다양한 배포 시나리오에서 유연성을 제공하여 다양한 서버에서 실행할 수 있습니다.
  - 일반적인 시나리오
    - apachd2 with mod_wsgi
    - apache2 with mod_proxy_uwsgi
    - nginx with uwsgi
    - nginx with gunicorn
- placement API 서비스는 자체적으로 stateless (모든 state가 데이터베이스에 저장)이고 간단한 확장을 위해 로드 밸런싱 솔루션 뒤에 원하는 만큼 서버를 배포할 수 있습니다.

#### Synchronize the database

- 배치 서비스는 configuration의 placement_database 섹션에 정의된 자체 데이터베이스를 사용합니다.
  - placement_database.connection 옵션을 설정해야 하며 설정하지 않으면 서비스가 시작되지 않습니다.
- 동기화를 위한 또 다른 옵션은 placement_database.sync_on_startup을 True로 설정하는 것입니다.
  - 이렇게하면 배치 웹 서비스가 시작될 때 누락된 데이터베이스 마이그레이션을 수행합니다.

#### Create accounts and update the service catalog

- Keystone에 관리자 역할을 하는 placement service user를 생성합니다.
- placement API는 별도의 서비스이므로 서비스 카탈로그에서 placement service 유형을 등록해야합니다.
- Devstack은 독립 포트를 사용하는 대신 placement prefix를 사용하여 기본 HTTP port(80)에 placement service를 설정합니다.



### 설치 및 구성

#### 데이터베이스 생성

- root 사용자로 데이터베이스 서버에 연결

  - `mysql -u root -p`

- placement 데이터베이스 생성

  - `CREATE DATABASE placement`

- 데이터베이스에 접근 권한 부여

  ```sql
  MariaDB [(none)]> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' \ IDENTIFIED BY 'PLACEMENT_DBPASS';	#PACEMENT_DBPASS 적절한 암호로 변경
  MariaDB [(none)]> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' \ IDENTIFIED BY 'PLACEMENT_DBPASS';	#PACEMENT_DBPASS 적절한 암호로 변경
  ```

- 종료
  - `exit`

#### 사용자 및 Endpoint 구성

- admin 자격 증명
  - `. admin-openrc`
- placement 사용자 생성
  - `openstack user create --domain default --password-prompt placement`
  - ![image-20210608142327478](images/image-20210608142327478.png)

- 관리자 역할이 있는 서비스 프로젝트에 placement 사용자 추가
  - `openstack role add --project service --user placement admin`
- 서비스 카탈로그에서 placement API 항목 생성
  - `openstack service create --name placement --description "Placement API" placement`
  - ![image-20210608142539170](images/image-20210608142539170.png)

- placement API 서비스 Endpoint 생성

  ```sh
  root@controller$ openstack endpoint --region RegionOne placement public http://controller:8778
  root@controller$ openstack endpoint create --region RegionOne placement internal http://controller:8778
  root@controller$ openstack endpoint create --region RegionOne placement admin http://controller:8778
  ```

  ![image-20210608142923551](images/image-20210608142923551.png)

  ![image-20210608142940801](images/image-20210608142940801.png)

#### 구성 요소 설치 및 구성

- 패키지 설치

  - `yum install openstack-placement-api`

- `/etc/placement/placement.conf` 파일 수정

  - `[placement_database]` 섹션

    ```bash
    [placement_database]
    # ...
    connection = mysql+pymysql://placement:PLACEMENT_DBPASS@controller/placement	# PLACEMENT_DBPASS는 placement 사용자 password
    ```

  - `[api]`와 `[keystone_authtoken]` 섹션

    ```bash
    [api]
    # ...
    auth_strategy = keystone
    
    [keystone_authtoken]
    # ...
    auth_url = http://controller:5000/v3
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = Default
    user_domain_name = Default
    project_name = service
    username = placement
    password = PLACEMENT_PASS		// placement 사용자 password
    ```

- placement 데이터베이스 채움

  - `su -s /bin/sh -c "placement-manage db sync" placement`

#### 설치 완료

- httpd 서비스 다시 시작
  - `systemctl restart httpd`



### 설치 확인

- admin 자격 증명
  - `. admin-openrc`
- 상태 확인 수행
  - `placement-status upgrade check`
  - ![image-20210608144254311](images/image-20210608144254311.png)



## Nova Installation

### Compute Service 개요

- 클라우드 컴퓨팅 시스템을 호스팅하고 관리합니다.
- IaaS(Infrastructure-as-a-Service) 시스템의 주요 부분으로 주요 모듈은 Python으로 구현되어 있습니다.
- Compute는 인증을 위한 Identity, 리소스 인벤토리 추적 및 선택을 위한 Placement, 디스크 및 서버 이미지를 위한 Image, 사용자 및 관리 인터페이스를 위한 Dashboard와 상호 작용합니다.

#### 구성 요소

- **nova-api**
  - 사용자 컴퓨팅 API 호출을 수락하고 이에 응답합니다.
- **nova-api-metadata service**
  - 인스턴스의 메타 데이터 요청을 수락합니다.
  - nova-api-metadata 서비스는 일반적으로 nova-network 설치와 함께 다중 호스트 모드에서 실행할 때 사용됩니다.
- **nova-compute service**
  - 하이퍼 바이저 API를 통해 가상 머신 인스턴스를 생성하고 종료하는 작업자 데몬입니다.
    - XenServer / XCP 용 XenAPI
    - KVM / QEMU 용 libvirt
    - VMware 용 VMwareAPI
- **nova-scheduler service**
  - 대기열에서 가상 머신 인스턴스 요청을 받아 실행할 컴퓨팅 서버 호스트를 결정합니다.

- **nova-conductor-module**
  - nova-compute 서비스와 데이터베이스 간의 상호 작용을 조정합니다.
  - nova-compute service에서 만든 클라우드 데이터베이스에 대한 직접 접근을 제거합니다.
- **nova-novncproxy daemon**
  - VNC 연결을 통해 실행 중인 인스턴스에 접근하는 프록시를 제공합니다.
  - 브라우저 기반 novnc 클라이언트를 지원합니다.
- **nova-spicehtml5proxy daemon**
  - SPICE 연결을 통해 실행 중인 인스턴스에 접근하는 프록시를 제공합니다.
  - 브라우저 기반 HTML5 클라이언트를 지원합니다.
- **nova-xvpvncproxy daemon**
  - VNC 연결을 통해 실행 중인 인스턴스에 접근하는 프록시를 제공합니다.
  - OpenStack-specific Java 클라이언트를 지원합니다.
  - *버전 19.0.0(Stein) 이후 더 이상 사용하지 않음*
- **The queue**
  - 데몬 간의 메시지를 전달하는 중앙 허브입니다.
  - 일반적으로 RabbitMQ로 구현되지만 다른 옵션을 사용할 수 있습니다.
- **SQL Database**
  - 다음을 포함하여 클라우드 인프라의 대부분의 빌드 시간 및 런타임 상태를 저장합니다.
    - 사용 가능한 인스턴스 유형
    - 사용 중인 인스턴스
    - 사용 가능한 네트워크
    - 프로젝트
  - 이론적으로 OpenStack Compute는 SQLAlchemy에서 지원하는 모든 데이터베이스를 지원합니다.
    - 공통 데이터베이스는 테스트 및 개발 작업을 위한 SQLite3, MySQL, MariaDB 및 PostgreSQL 입니다.



### 설치 및 구성(Controller Node)

- 컨트롤러 노드에서 nova를 설치하고 구성하는 방법을 설명합니다.

#### 전제 조건

- Compute 노드를 설치 및 구성하기 전에 데이터베이스, 서비스 자격 증명 및 API 엔드 포인트를 생성해야 합니다.

- 데이터베이스 생성

  - 데이터베이스 액세스 클라이언트를 사용하여 root 사용자로 데이터베이스 서버에 연결

    - `mysql -u root -p`

  - nova, nova_api 및 nova_cell0 데이터 베이스 생성

    ```sql
    MariaDB [(none)]> CREATE DATABASE nova_api;
    MariaDB [(none)]> CREATE DATABASE nova;
    MariaDB [(none)]> CREATE DATABASE nova_cell0;
    ```

  - 데이터베이스에 접근 권한 부여

    ```sql
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
      IDENTIFIED BY 'NOVA_DBPASS';
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
      IDENTIFIED BY 'NOVA_DBPASS';
    
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
      IDENTIFIED BY 'NOVA_DBPASS';
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
      IDENTIFIED BY 'NOVA_DBPASS';
    
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
      IDENTIFIED BY 'NOVA_DBPASS';
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
      IDENTIFIED BY 'NOVA_DBPASS';
    ```

  - 데이터베이스 액세스 클라이언트 종료
    - `exit`

- admin 자격 증명 관리자

  - `. admin-openrc`

- Compute 서비스 자격 증명 생성

  - nova 사용자 생성

    - `openstack user create --domain default --password-prompt nova`
    - ![image-20210609161049354](images/image-20210609161049354.png)

  - nova 사용자에게 admin role 추가

    - `openstack role add --project service --user nova admin`

      > 출력 결과 없음

  - nova service entity 생성

    - `openstack service create --name nova --description "OpenStack Compute" compute`
    - ![image-20210609161630845](images/image-20210609161630845.png)

- Compute API 서비스 엔드 포인트 생성

  ```sh
  root@controller$ openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
  root@controller$ openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
  root@controller$ openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
  ```

  ![image-20210609161947788](images/image-20210609161947788.png)

  ![image-20210609162006951](images/image-20210609162006951.png)

#### 구성 요소 설치 및 구성

- 패키지 설치

  - `yum install openstack-nova-api openstack-nova-conductor openstack-nova-novncproxy openstack-nova-scheduler`

- `/etc/nova/nova.conf` 수정

  - [DEFAULT] 섹션에 컴퓨팅 및 메타데이터 API를 활성화

    ```bash
    [DEFAULT]
    # ...
    enabled_apis = osapi_compute,metadata
    ```

  - [api_database]와 [database] 섹션에 데이터베이스 액세스 구성

    ```bash
    [api_database]
    # ...
    connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api
    
    [database]
    # ...
    connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova
    ```

    > NOVA_DBPASS는 Compute 데이터베이스 암호로 변경

  - [DEFAULT] 섹션에 RabbitMQ 메시지 큐 구성

    ```bash
    [DEFAULT]
    # ...
    transport_url = rabbit://openstack:RABBIT_PASS@controller:5672/
    ```

    > RABBIT_PASS는 openstack 계정 암호로 변경

  - [api]와 [keystone_authtoken] 섹션에 Identity 서비스 접근 구성

    ```bash
    [api]
    # ...
    auth_strategy = keystone
    
    [keystone_authtoken]
    # ...
    www_authenticate_uri = http://controller:5000/
    auth_url = http://controller:5000/
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = Default
    user_domain_name = Default
    project_name = service
    username = nova
    password = NOVA_PASS
    ```

    > NOVA_PASS를 Identity 서비스에서 사용자의 비밀번호로 변경

  - [DEFAULT] 섹션에 my_ip 구성

    ```bash
    [DEFAULT]
    # ...
    my_ip = 10.0.0.11
    ```

  - [DEFAULT] 섹션에 네트워킹 서비스에 대한 지원을 활성화

    ```bash
    [DEFAULT]
    # ...
    use_neutron = true
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    ```

    > 기본적으로 Compute는 내부 방화벽 드라이버를 사용합니다. 네트워킹 서비스에 방화벽 드라이버가 포함되어 있으므로 Compute 방화벽 드라이버를 비활성화합니다.

  - [vnc] 섹션에 컨트롤러 노드의 IP 주소 인터페이스 관리를 사용하도록 VNC 프록시를 구성

    ```bash
    [vnc]
    enabled = true
    # ...
    server_listen = $my_ip
    server_proxyclient_address = $my_ip
    ```

  - [glance] 섹션에 이미지 서비스 API의 위치를 구성

    ```bash
    [glance]
    # ...
    api_servers = http://controller:9292
    ```

  - [oslo_concurrency] 섹션에 잠금 경로를 구성

    ```bash
    [oslo_concurrency]
    # ...
    lock_path = /var/lib/nova/tmp
    ```

  - [placement] 섹션에 배치 서비스에 대한 접근 구성

    ```bash
    [placement]
    # ...
    region_name = RegionOne
    project_domain_name = Default
    project_name = service
    auth_type = password
    user_domain_name = Default
    auth_url = http://controller:5000/v3
    username = placement
    password = PLACEMENT_PASS	# placement 서비스 사용자의 비밀번호
    ```

- nova-api 데이터베이스 채움
  - `su -s /bin/sh -c "nova-manage api_db sync" nova`
- cell0 데이터베이스 등록
  - `su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova`
- cell1 셀 만들기
  - `su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova`
- nova 데이터베이스 채움
  - `su -s /bin/sh -c "nova-manage db sync" nova`
- nova cell0 및 cell1이 올바르게 등록되었는지 확인
  - `su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova`
  - ![image-20210609164946421](images/image-20210609164946421.png)

#### 설치 완료

- Compute 서비스를 시작하고 시스템이 부팅될 떄 시작되도록 구성

  ```sh
  root@controller$ systemctl enable \
      openstack-nova-api.service \
      openstack-nova-scheduler.service \
      openstack-nova-conductor.service \
      openstack-nova-novncproxy.service
  root@controller$ systemctl start \
      openstack-nova-api.service \
      openstack-nova-scheduler.service \
      openstack-nova-conductor.service \
      openstack-nova-novncproxy.service
  ```



### 설치 및 구성(Compute Node)

- 이 서비스는 인스턴스 또는 가상 머신을 배포하기 위해 여러 하이퍼 바이저를 지원합니다.
- 이 구성에서는 단순화를 위해 가상 머신에 대한 하드웨어 가속을 지원하는 **커널 기반 VM(KVM)**과 **QEMU(Quick EMUlator)** 하이퍼 바이저를 사용합니다.


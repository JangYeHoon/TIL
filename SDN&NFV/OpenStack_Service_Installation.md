# OpenStack Service Installation

- [Keystone Installation](#keystone-installation)
- [Glance Installation](#glance-installation)
- [Placement Installation](#placement-installation)
- [Nova Installation](#nova-installation)
- [Neutron Installation](#neutron-installation)



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

#### 구성 요소 설치 및 구성

- 패키지 설치

  - `yum install openstack-nova-compute`

- `/etc/nova/nova.conf` 파일 수정

  - [DEFAULT] 섹션에서 컴퓨팅 및 메타 데이터 API 활성화

    ```bash
    [DEFAULT]
    # ...
    enabled_apis = osapi_compute,metadata
    ```

  - [DEFAULT] 섹션에서 RabbitMQ 구성

    ```bash
    [DEFAULT]
    # ...
    transport_url = rabbit://openstack:RABBIT_PASS@controller	// openstack 계정 암호
    ```

  - [api]와 [keystone_authtoken] 섹션에 아이덴티티 서비스 액세스 구성

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
    password = NOVA_PASS	// NOVA 사용자 암호
    ```

  - [DEFAULT] 섹션 my_ip 옵션 구성

    ```bash
    [DEFAULT]
    # ...
    my_ip = MANAGEMENT_INTERFACE_IP_ADDRESS	// 컴퓨팅 노드에 있는 관리 네트워크 인터페이스의 IP 주소(예제 아키텍처는 10.0.0.31)
    ```

  - [DEFAULT] 섹션에 네트워킹 서비스에 대한 지원을 활성화

    ```bash
    [DEFAULT]
    # ...
    use_neutron = true
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    ```

  - [vnc] 섹션에 원격 콘솔 액세스를 활성화

    ```bash
    [vnc]
    # ...
    enabled = true
    server_listen = 0.0.0.0
    server_proxyclient_address = $my_ip
    novncproxy_base_url = http://controller:6080/vnc_auto.html
    ```

    - 서버는 모든 IP 주소를 수신하고 프록시는 compute 노드의 관리 인터페이스 IP 주소만 수신
    - 기본 URL은 웹 브라우저를 사용하여 이 컴퓨팅 노드에 있는 인스턴스의 원격 콘솔에 액세스할 수 있는 위치를 나타냄

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

  - [placement] 섹션에 placement API를 구성

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
    password = PLACEMENT_PASS	// placement 암호로 변경
    ```

#### 설치 완료

- 컴퓨팅 노드가 가상 머신에 대한 하드웨어 가속을 지원하는지 확인

  - `egrep -c '(vmx|svm)' /proc/cpuinfo`

  - ![image-20210610172215701](images/image-20210610172215701.png)

  - `one or greater`를 리턴하면 하드웨어 가속을 지원, 추가 설정 필요 없음

  - `zero`를 리턴하면 하드웨어 가속을 지원하지 않으므로  KVM 대신 QEMU를 사용하도록 `libvirt` 구성

    - `/etc/nova/nova.conf` 수정

      ```bash
      [libvirt]
      # ...
      virt_type = qemu
      ```

- compute 서비스를 시작하고 시스템이 부팅될 때 자동으로 시작되도록 구성

  ```sh
  root@controller$ systemctl enable libvirtd.service openstack-nova-compute.service
  root@controller$ systemctl start libvirtd.service openstack-nova-compute.service
  ```

#### 셀 데이터베이스에 컴퓨팅 노드 추가

- **Controller Node에서 명령 실행**

- 관리자 자격 증명

  - `. admin-openrc`

- 데이터베이스에 컴퓨팅 호스트가 있는지 확인

  - `openstack compute service list --service nova-compute`
  - ![image-20210610172956757](images/image-20210610172956757.png)

-  Discover compute hosts

  - `su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova`

  - ![image-20210610173143843](images/image-20210610173143843.png)

    > 새 compute 노드를 추가 할 때 컨트롤러 노드에서 `nova-manage cell_v2 discover_hosts`를 실행하여 새 compute 노드를 등록해야합니다.



### 작동 확인

- compute 서비스의 작동 확인

- **Controller Node에서 수행**

  - admin 자격 증명

    - `. admin-openrc`

  - 서비스 구성 요소를 출력하여 등록이 잘 되어 있는지 확인

    - `openstack compute service list`

    - ![image-20210610173709679](images/image-20210610173709679.png)

      > 위 그림처럼 3개의 서비스가 활성화되어 있어야 합니다.

  - Identity 서비스에 API end-point를 나열하여 Identity 서비스와의 연결 확인

    - `openstack catalog list`
    - ![image-20210610173759217](images/image-20210610173759217.png)

  - 이미지 서비스에 이미지를 나열하여 이미지 서비스와의 연결 확인

    - `openstack image list`
    - ![image-20210610173818591](images/image-20210610173818591.png)

  - cell 및 placement API가 성공적으로 작동하고 다른 필수 전제 조건이 있는지 확인

    - `nova-status upgrade check`

    - `nova-status upgrade check forbidden (HTTP 403)` 에러 발생

      - 해결방법

      - `vi /etc/httpd/conf.d/00-placement-api.conf`

        ```bash
        <Directory /usr/bin>
            <IfVersion >= 2.4>
                Require all granted
            </IfVersion>
            <IfVersion < 2.4>
                Order allow,deny
                Allow from all
            </IfVersion>
        </Directory>
        ```

      - ![image-20210610174513558](images/image-20210610174513558.png)

      - `systemctl restart httpd`

      - 해결 완료

    - ![image-20210610174756036](images/image-20210610174756036.png)



## Neutron Installation

### 네트워킹 서비스 개요

- OpenStack Networking(Neutron)을 사용하면 다른 OpenStack 서비스에서 관리하는 인터페이스 장치를 생성하고 네트워크에 연결할 수 있습니다.
- 플러그인을 구현하면 다양한 네트워킹 장비 및 소프트웨어를 수용할 수 있습니다.

#### 구성 요소

- **neutron-server**
  - API를 수신하고 OpenStack Networking 플러그인으로 라우팅합니다.
- **OpenStack Networking plug-ins and agents**
  - 포트 추가/삭제, 네트워크 또는 서브넷을 만들고 IP 주소를 제공합니다.
  - 플러그인 및 에이전트는 특정 클라우드에서 사용되는 공급 업체 및 기술에 따라 다릅니다.
    - OpenStack Networking
      - Cisco virtual and physicla switches
      - NEC OpenFLow products
      - Open vSwitch
      - Linux bridging
      - VMware NSX product
  - 공통 에이전트는 L3, DHCP, plug-in agent 입니다.
- **Messaging Queue**
  - neutron-server와 다양한 에이전트간에 정보를 라우팅하는데 사용합니다.
  - 또한 특정 플러그인에 대한 네트워킹 상태를 저장하는 데이터베이스 역할을 합니다.



### 네트워킹(Neutron) 개념

- OpenStack Networking(Neutron)은 OpenStack 환경에서 VNI(가상 네트워킹 인프라) 및 PNI(물리적 네트워킹 인프라)의 액세스 계층 측면에 대한 모든 네트워킹 패싱을 관리합니다.
- OpenStack Networking을 사용하면 프로젝트에서 방화벽 및 VPN 등을 만들 수 있습니다.
- 네트워킹 설정에는 하나 이상의 외부 네트워크가 있습니다.
  - 외부 네트워크는 단순히 가상으로 정의된 네트워크가 아닌 OpenStack 외부에서 접근할 수 있는 물리적 외부 네트워크입니다.
  - 외부 네트워크의 IP 주소는 외부의 모든 사람이 물리적으로 접근할 수 있습니다.
- 외부 네트워크 외에도 하나 이상의 내부 네트워크가 존재합니다.
  - 이러한 소프트웨어 정의 네트워크(내부 네트워크)는 VM에 직접 연결됩니다.
  - 같은 내부 네트워크의 VM 또는 라우터에 연결된 서브넷의 VM만 해당 네트워크에 연결된 VM에 직접 접근할 수 있습니다.
- 외부 네트워크에서 내부 네트워크에 연결된 VM에 액세스하거나 그 반대의 경우 네트워크 간 라우터가 필요합니다.
  - 각 라우터에는 외부 네트워크에 연결된 게이트웨이 하나와 내부 네트워크에 연결된 인터페이스가 하나 이상 있습니다.
  - 물리적 라우터와 마찬가지로 라우터에 연결된 다른 서브넷의 시스템에 접근할 수 있으며, 시스템은 라우터의 게이트웨이를 통해 외부 네트워크에 접근할 수 있습니다.
- 외부 네트워크의 IP 주소를 내부 네트워크나 VM의 포트에 연결할 수 있습니다.
  - 이렇게 하면 외부 네트워크의 entity가 VM에 접근할 수 있습니다.
- 네트워킹은 보안 그룹을 지원합니다.
  - 관리자는 Security Group을 사용하여 방화벽 규칙을 그룹으로 정의할 수 있습니다.
  - VM은 하나 이상의 Security Group에 속할 수 있으며, Neutron은 이러한 Security Group의 규칙을 적용하여 해당 VM의 포트, 포트 범위 또는 트래픽 유형을 차단하거나 허용합니다.
- 네트워킹에서 사용하는 플러그인
  - 코어 플러그인, 보안 그룹 플러그인, FWaaS(Firewall-as-a-Service)



### 설치 및 구성(Controller Node)

#### 전제 조건

- OpenStack Networking(Neutron) 서비스를 구성하기 전에 데이터베이스, 서비스 자격 증명 및 API 엔드 포인트를 생성해야 합니다.

1. 데이터베이스 작성

   - root 사용자로 데이터베이스 서버 연결

     - `mysql -u root -p`

   - neutron 데이터베이스 생성

     - `CREATE DATABASE neutron;`

   - neutron 데이터베이스에 대한 접근 권한을 부여하고 `NEUTRON_DBPASS`를 적절한 암호로 변경

     ```mariadb
     MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
       IDENTIFIED BY 'NEUTRON_DBPASS';
     MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
       IDENTIFIED BY 'NEUTRON_DBPASS';
     ```

2. 관리자 자격 증명을 등록하여 관리자 전용 CLI 명령에 액세스

   - `. admin-openrc`

3. 서비스 자격 증명 생성

   - neutron 유저 생성

     - `openstack user create --domain default --password-prompt neutron`
     - ![image-20210614150202993](images/image-20210614150202993.png)

   - neutron 유저에 admin role 추가

     - `openstack role add --project service --user neutron admin`

       > 이 명령은 출력이 없습니다.

   - neutron 서비스 엔터티 생성

     - `openstack service create --name neutron --description "OpenStack Networking" network`
     - ![image-20210614150322610](images/image-20210614150322610.png)

4. 네트워킹 서비스 API 앤드 포인트 생성

   ```sh
   root@controller$ openstack endpoint create --region RegionOne \
     network public http://controller:9696
   root@controller$ openstack endpoint create --region RegionOne \
     network internal http://controller:9696
   root@controller$ openstack endpoint create --region RegionOne \
     network admin http://controller:9696
   ```

   ![image-20210614150621212](images/image-20210614150621212.png)

   ![image-20210614150656690](images/image-20210614150656690.png)

#### 네트워킹 옵션 구성

- 네트워킹 서비스를 배포할 때 2가지 옵션이 존재합니다.
- 옵션 1은 외부 네트워크에 인스턴스 연결만을 지원하는 가장 간단한 아키텍처를 배포합니다.
  - 셀프 서비스(사설) 네트워크, 라우터 또는 Floating IP 주소가 없습니다.
- 옵션 2는 셀프 서비스 네트워크에 인스턴스 연결을 지원하는 L3 서비스로 옵션 1을 보강합니다.
  - 사용자는 셀프 서비스 및 provider 네트워크 사이의 연결을 제공하는 라우터를 포함하여 셀프 서비스 네트워크를 관리할 수 있습니다.
  - Floating IP는 인터넷과 같은 외부 네트워크의 셀프 서비스 네트워크를 사용하여 인스턴스에 대해 외부 네트워크와의 연결을 제공합니다.
  - 일반적으로 오버레이 네트워크를 사용하는데 이때 VXLAN을 사용합니다.

#### 셀프 서비스 네트워크 구성

- **컨트롤러 노드에서 진행합니다.**

##### 구성 요소 설치

- `yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables`

##### 서버 구성 요소 구성

- `/etc/neutron/neutron.conf` 수정

  - [database] 섹션에 데이터베이스 액세스 구성

    ```bash
    [database]
    # ...
    connection = mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron
    ```

    - `NEUTRON_DBPASS`는 데이터베이스에 대해 선택한 암호로 바꿉니다 .

  - [DEFAULT] 섹션에 ml2 플러그인, 라우터 서비스 및 겹치는 IP 주소를 사용하도록 설정

    ```bash
    [DEFAULT]
    # ...
    core_plugin = ml2
    service_plugins = router
    allow_overlapping_ips = true
    ```

  - [DEFAULT] 섹션에서 Rabbit MQ을 구성

    ```bash
    [DEFAULT]
    # ...
    transport_url = rabbit://openstack:RABBIT_PASS@controller
    ```

    - RABBIT_PASS는 RabbitMQ의 openstack 계정에 대한 암호로 변경

  - [DEFAULT]와 [keystone_authtoken]에 Identity 서비스 접근 구성

    ```bash
    [DEFAULT]
    # ...
    auth_strategy = keystone
    
    [keystone_authtoken]
    # ...
    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = NEUTRON_PASS
    ```

    - NEUTRON_PASS는 Identity service에서 neutron 사용자에 대한 암호로 변경

  - [DEFAULT] 및 [nova] 섹션에서 네트워크 토폴로지 변경 사항을 컴퓨팅에 알리도록 네트워킹을 구성

    ```bash
    [DEFAULT]
    # ...
    notify_nova_on_port_status_changes = true
    notify_nova_on_port_data_changes = true
    
    [nova]
    # ...
    auth_url = http://controller:5000
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = nova
    password = NOVA_PASS
    ```

    - NOVA_PASS는 nova 사용자 암호로 변경

  - [oslo_concurrency] 섹션에서 잠금 경로를 구성

    ```bash
    [oslo_concurrency]
    # ...
    lock_path = /var/lib/neutron/tmp
    ```

##### ML2 플러그인 구성

- ML2 플러그인은 Linux 브리지 메커니즘을 사용하여 인스턴스 용 레이어 2 (브리징 및 스위칭) 가상 네트워킹 인프라를 구축합니다.

- `/etc/neutron/plugins/ml2/ml2_conf.ini` 수정

  - [ml2] 섹션에서 flat, vlan, vxlan 네트워크 활성화

    ```bash
    [ml2]
    # ...
    type_drivers = flat,vlan,vxlan
    ```

  - [ml2] 섹션에서 VXLAN self-service networks 활성화

    ```bash
    [ml2]
    # ...
    tenant_network_types = vxlan
    ```

  - [ml2] 섹션에서 Linux bridge와 L2 population mechanism 활성화

    ```bash
    [ml2]
    # ...
    mechanism_drivers = linuxbridge,l2population
    ```

    - ML2 플러그인을 구성한 후 type_drivers 옵션의 값을 제거하면 데이터베이스 불일치가 발생할 수 있습니다.

    > Linux bridge agent는 오직 VXLAN overlay network만 지원합니다.

  - [ml2] 섹션에서 port security extension driver 활성화

    ```bash
    [ml2]
    # ...
    extension_drivers = port_security
    ```

  - [ml2_type_flat] 섹션에서 provider 가상 네트워크를 flat네트워크로 구성

    ```bash
    [ml2_type_flat]
    # ...
    flat_networks = provider
    ```

  - [ml2_type_vxlan] 섹션에서 셀프 서비스 네트워크의 VXLAN 네트워크 식별자 범위를 구성

    ```bash
    [ml2_type_vxlan]
    # ...
    vni_ranges = 1:1000
    ```

  - [security group] 섹션에서 ipset을 활성화하여 Security group 규칙의 효율성을 높임

    ```bash
    [securitygroup]
    # ...
    enable_ipset = true
    ```

##### Linux bridge agent 구성

- 리눅스 브리지 에이전트는 인스턴스를 위한 L2(브릿지 및 스위칭) 가상 네트워킹 인프라를 구축하고 보안 그룹을 처리합니다.

- `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` 수정

  - [linux_bridge] 섹션에서 provider 가상 네트워크를 provider 물리적 네트워크 인터페이스에 매핑

    ```bash
    [linux_bridge]
    physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME
    ```

    - PROVIDER_INTERFACE_NAME은 기본 provider 물리 네트워크 인터페이스로 수정

  - [vxlan] 섹션에서 VXLAN 오버레이 네트워크를 사용하도록 설정하고 오버레이 네트워크를 처리하는 물리적 네트워크 인터페이스의 IP 주소를 구성하고 L2 population를 사용하도록 설정

    ```bash
    [vxlan]
    enable_vxlan = true
    local_ip = OVERLAY_INTERFACE_IP_ADDRESS
    l2_population = true
    ```
    - OVERLAY_INTERFACE_IP_ADDRESS를 오버레이 네트워크를 처리하는 기본 물리적 네트워크 인터페이스의 IP 주소로 변경
    - 예제 아키텍처는 관리 인터페이스를 사용하여 트래픽을 다른 노드로 터널링합니다. 따라서 OVERLAY_INTERFACE_IP_ADDRESS를 컨트롤러 노드의 관리 IP 주소로 교체

  - [security group] 섹션에서 Security group을 사용하도록 설정하고 Linux bridge iptables 방화벽 드라이버를 구성

    ```bash
    [securitygroup]
    # ...
    enable_security_group = true
    firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
    ```

  - `/usr/sbin/modprobe br_netfilter` 명령을 통해 bridge 모듈 로딩

    - `/usr/sbin/modprobe br_netfilter`

  - 다음 sysctl 값이 모두 1로 설정되어 있는지 확인하여 Linux 운영 체제 커널이 네트워크 브리지 필터를 지원하는지 확인

    ```sh
    root@controller$ sysctl -a | grep net.bridge.bridge-nf-call-ip6tables
    root@controller$ sysctl -a | grep net.bridge.bridge-nf-call-iptables
    ```

    ![image-20210615110816520](images/image-20210615110816520.png)

##### L3 Agent 구성

- L3 에이전트는 self-service 가상 네트워크에 라우팅과 NAT 서비스를 제공합니다.

- `/etc/neutron/l3_agent.ini` 수정

  - [DEFAULT] 섹션에서 Linux 브리지 인터페이스 드라이버를 구성

    ```bash
    [DEFAULT]
    # ...
    interface_driver = linuxbridge
    ```

##### DHCP Agent 구성

- DHCP 에이전트는 가상 네트워크에 DHCP 서비스를 제공합니다.

- `/etc/neutron/dhcp_agent.ini` 수정

  - [DEFAULT] 섹션에서 Linux 브리지 인터페이스 드라이버, DNSmasq DHCP 드라이버를 구성하고 provider 네트워크의 인스턴스가 네트워크를 통해 메타데이터에 액세스할 수 있도록 격리된 메타데이터를 사용하도록 설정

    ```bash
    [DEFAULT]
    # ...
    interface_driver = linuxbridge
    dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
    enable_isolated_metadata = true
    ```

#### Metadata Agent 구성

- Metadata 에이전트는 자격 증명과 같은 구성 정보를 인스턴스에 제공합니다.

- `/etc/neutron/metadata_agent.ini` 수정

  - [DEFAULT] 섹션에서 메타 데이터 호스트를 구성하고 데이터 프록시 암호 공유

    ```bash
    [DEFAULT]
    # ...
    nova_metadata_host = controller
    metadata_proxy_shared_secret = METADATA_SECRET
    ```

    - METADATA_SECRET을 메타 데이터 프록시에 대한 적절한 암호로 변경

#### 네트워킹 서비스를 사용하기 위한 Compute Service 구성

- 해당 과정을 수행하기 위해서는 Nova가 필수로 설치되어 있어야 합니다.

- `/etc/nova/nova.conf` 파일 수정

  - [neutron] 섹션에서 액세스 매개 변수를 구성하고 메타데이터 프록시를 사용하도록 설정한 다음 암호를 구성

    ```bash
    [neutron]
    # ...
    auth_url = http://controller:5000
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = neutron
    password = NEUTRON_PASS
    service_metadata_proxy = true
    metadata_proxy_shared_secret = METADATA_SECRET
    ```

    - NEUTRON_PASS는 neutron 사용자의 암호
    - METADAT_SECRET은 metadata proxy의 암호

#### 설치 완료

- 네트워킹 서비스 초기화 스크립트인 `/etc/neutron/plugin.ini`에 ML2 플러그인 구성 파일을 가리키는 심볼릭 링크가 필요

  - `/etc/neutron/plugins/ml2/ml2_conf.ini` 파일에 대한 심볼링 링크 설정
  - `ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini`

- 데이터베이스를 채움

  - `su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron`

- Compute API 서비스 재시작

  - `systemctl restart openstack-nova-api.service`

- 네트워킹 서비스를 시작하고 시스템이 부팅될 때 같이 시작되도록 구성

  - 옵션 1과 옵션 2 공통

    ```sh
    root@controller$ systemctl enable neutron-server.service \
      neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
      neutron-metadata-agent.service
    root@controller$ systemctl start neutron-server.service \
      neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
      neutron-metadata-agent.service
    ```

  - 옵션 2의 경우 L3 서비스도 활성화

    ```sh
    root@controller$ systemctl enable neutron-l3-agent.service
    root@controller$ systemctl start neutron-l3-agent.service
    ```



### 설치 및 구성(Compute Node)

- Compute Node는 인스턴스에 대한 연결 및 보안 그룹을 처리합니다.

#### 구성 요소 설치

- `yum install openstack-neutron-linuxbridge ebtables ipset`

#### 공통 구성 요소 설치

- 네트워킹 공통 구성 요소 구성에는 인증 메커니즘, 메시지 대기열 및 플러그인이 포함됩니다.

- `/etc/neutron/neutron.conf` 수정

  - [database] 섹션에서 compute node가 데이터베이스에 직접 액세스하지 않으므로 연결 옵션에 대해 설정

  - [DEFAULT] 섹션에서 RabbitMQ 메시지 큐 접근 구성

    ```bash
    [DEFAULT]
    # ...
    transport_url = rabbit://openstack:RABBIT_PASS@controller
    ```

    - RABBIT_PASS는 RabbitMQ에서 openstack 계정 비밀번호

  - [DEFAULT]와 [keystone_authtoken] 섹션에서 Identity service 접근 구성

    ```bash
    [DEFAULT]
    # ...
    auth_strategy = keystone
    
    [keystone_authtoken]
    # ...
    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = NEUTRON_PASS
    ```

    - NEUTRON_PASS는 Identity 서비스에서 neutron 사용자의 비밀번호

  - [oslo_concurrency] 섹션에서 lock path 구성

    ```bash
    [oslo_concurrency]
    # ...
    lock_path = /var/lib/neutron/tmp
    ```

#### 네트워킹 옵션 구성

- 네트워킹 옵션 2인 Self-service networks 구성

##### Linux Bridge Agent 구성

- Linux bridge agent는 인스턴스를 위한 L2(bridging and switching) 가상 네트워킹 인프라를 구축하고 보안 그룹을 처리합니다.

- `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` 수정

  - [linux_bridge] 섹션에서 provider 가상 네트워크를 제공자 provider 네트워크 인터페이스에 매핑

    ```bash
    [linux_bridge]
    physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME
    ```

    - PROVIDER_INTERFACE_NAME을 기본 provider 물리적 네트워크 인터페이스의 이름으로 변경

  - [vxlan] 섹션에서 VXLAN 오버레이 네트워크를 사용하도록 설정하고 오버레이 네트워크를 처리하는 물리적 네트워크 인터페이스의 IP 주소를 구성하고 L2 population를 사용하도록 설정

    ```bash
    [vxlan]
    enable_vxlan = true
    local_ip = OVERLAY_INTERFACE_IP_ADDRESS
    l2_population = true
    ```
    - OVERLAY_INTERFACE_IP_ADDRESS를 오버레이 네트워크를 처리하는 기본 물리적 네트워크 인터페이스의 IP 주소로 변경
    - 예제 아키텍처는 관리 인터페이스를 사용하여 트래픽을 다른 노드로 터널링합니다. 따라서 OVERLAY_INTERFACE_IP_ADDRESS를 컨트롤러 노드의 관리 IP 주소로 교체

  - [security group] 섹션에서 Security group을 사용하도록 설정하고 Linux bridge iptables 방화벽 드라이버를 구성

    ```bash
    [securitygroup]
    # ...
    enable_security_group = true
    firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
    ```

  - `/usr/sbin/modprobe br_netfilter` 명령을 통해 bridge 모듈 로딩

    - `/usr/sbin/modprobe br_netfilter`

  - 다음 sysctl 값이 모두 1로 설정되어 있는지 확인하여 Linux 운영 체제 커널이 네트워크 브리지 필터를 지원하는지 확인

    ```sh
    root@controller$ sysctl -a | grep net.bridge.bridge-nf-call-ip6tables
    root@controller$ sysctl -a | grep net.bridge.bridge-nf-call-iptables
    ```

    ![image-20210616144629896](images/image-20210616144629896.png)

#### 네트워킹 서비스를 사용하기 위한 Compute Service 구성

- `/etc/nova/nova.conf` 파일 수정

  - [neutron] 섹션에서 액세스 매개 변수를 구성

    ```bash
    [neutron]
    # ...
    auth_url = http://controller:5000
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = neutron
    password = NEUTRON_PASS
    ```

    - NEUTRON_PASS는 Identity 서비스에서 neutron 사용자의 비밀번호

#### 설치 완료

- Compute service 재시작

  - `systemctl restart openstack-nova-compute.service`

- linuxbridge agent를 시작하고 시스템이 부팅될 때 시작되도록 구성

  ```sh
  root@controller$ systemctl enable neutron-linuxbridge-agent.service
  root@controller$ systemctl start neutron-linuxbridge-agent.service
  ```




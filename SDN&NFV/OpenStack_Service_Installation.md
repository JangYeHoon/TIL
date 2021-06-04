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
    [database]
    # ... 원래 있던 connection 줄 주석 처리
    connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
    ```

    ![image-20210604151253990](images/image-20210604151253990.png)

  - `[token]` 섹션에서 Fernet 토큰 공급자를 구성

    ```bash
    [token]
    # ... 원래 있던 provider 줄 주석 처리
    provider = fernet
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

- 인증에 사용할 환경 변수 설정

  ```sh
  user@controller$ export OS_USERNAME=admin
  user@controller$ export OS_PASSWORD=ADMIN_PASS
  user@controller$ export OS_PROJECT_NAME=admin
  user@controller$ export OS_USER_DOMAIN_NAME=Default
  user@controller$ export OS_PROJECT_DOMAIN_NAME=Default
  user@controller$ export OS_AUTH_URL=http://controller:5000/v3
  user@controller$ export OS_IDENTITY_API_VERSION=3
  ```


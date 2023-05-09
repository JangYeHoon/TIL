# 유데미 AWS 강의

- [IAM](#iam)
- [EC2](#ec2)
- [Storage](#storage)

## IAM

- 글로벌 서비스
- 루트 계정은 자동으로 생성
- 유저를 생성해 회사나 그룹 포함할 수 있음
- 그룹에는 사용자만 추가 가능

### IAM Permissions

- 사용자나 그룹에 JSON을 이용해 정책 부여
- JSON의 정책을 통해 사용자에게 권한 부여
- AWS는 최소 권한 원칙(least privilege principle) 사용


### IAM Policies inheritance

- version number
- id : 선택사항
- statement : 하나 혹은 여러개
  - Sid : 문장의 Id, 선택사항
  - Effect : 문장이 특정 API 접근하는 걸 허용할지 거부할지에 대한 내용
  - Principal : 특정 정책이 적용될 사용자, 계정 역할이 정의
  - Action : 허용, 거부되는 API 호출 목록
  - Resource : 적용될 action의 리소스 목록
  - condition : statement가 언제 적용될지 정의, 선택사항

#### IAM - Password Policy

- 비밀번호를 이용해 계정을 보호하는 방법
- 비밀번호에 정책을 부여해 안전하게 보호

#### Multi Factor Authentication - MFA

- root 계정과 전체 IAM 유저들은 모두 보호해야함
- MFA = passowrd + security device
- MFA devices options
  - virtual MFA device = google auth(phone only), Authy(multi-device)
  - Universal 2nd Factor(U2F) Security Key - YubiKey by Yubico
  - Hardware Key Fod MFA Device - Provided by Gemalto
  - Hardware Key Fod MFA Device for AWS GovCloud(US) - Provided by SurePassID

### access AWS

- AWS Management Consolke - password + MFA
- AWS Command Line Interface(CLI) - access key
- AWS Software Developer Kit(SDK) - access key

#### IAM Roles for Services

- AWS서비스에 권한을 부여하는 기능
  - ex) ec2 인스턴스를 생성해서 다른 aws 서비스에 접근하려면 role이 필요
- Common roles
  - EC2
  - Lambda Function
  - Roles for CloudFormation

### IAM Security Tools

- IAM Credentials Report(account-level)
  - 유저들의 모든 정보를 볼 수 있음
- IAM Access Advisor(user-level)
  - 사용자에게 부여한 서비스의 권한과 해당 서비스에 마지막으로 접근한 시간이 보임

### IAM Guidelines & Best Practices

- 루트 계정은 AWS 계정을 설정하는 것을 제외하고 사용하지 않는다.
- 하나의 AWS 사용자 = 한명의 실제 사용자
- 그룹 수준에서 보안을 관리
- 강한 비밀번호 정책
- MFA 정책 사용
- AWS에 권한을 줄 때 role을 만들어 사용
- CLI/SDK를 사용하려면 Access Key 사용
- 계정의 권한을 체크할 때는 IAM Credentials Report & IAM Access Advisor를 사용
- 절대 IAM 사용자와 Access Key를 공유하지 않는다.




## EC2

###	 EC2 User Data

- EC2 User data 스크립트를 통해 bootstrap 구성 가능
- bootstrap은 EC2 인스턴스 처음 생성 시 정의한 스크립트를 수행

### Security Group

- 허용 규칙만 설정 가능
- 허용 규칙 외의 모든 ip나 포트는 차단
- 보안 그룹 끼리 참조 가능
- EC2 외부에 있음
- 기본적으로 모든 인바운드 트래픽은 차단
- 기본적으로 모든 아웃바운드 트래픽은 허용

### Classic Port

- 22 = SSH(Secure Shell)
- 21 = FTP(File Transfer Protocol)
- 22 = SFTP(Secure File Transfer Protocol)
- 80 = HTTP
- 443 = HTTPS
- 3389 = RDP(Remote Desktop Protocol)


### EC2 Instances Purchasing Options

- On-Demand Instances
- Reserved(1 & 3 years)
  - Reserved Instances - long workloads
  - Convertible Reserved Instances - long workloads with flexible instnaces
- Savings Plans(1 & 3 years) - 달러 단위로 사용량 약정, long workloads
- Spot Instances - short workloads, 저렴하지만 인스턴스가 손실될 수 있어 신뢰성이 낮음
  - 데이터베이스나 중요한 작업에 적절하지 않음
  - 배치 작업, 데이터 분석, 실패해도 복원할 수 있는 워크로드에서 사용
  - spot instances 종료는 먼저 request를 취소해 새로운 인스턴스를 생성하지 않게 하고 인스턴스를 종료
  - Spot Fleets - Spot Instaces + (optional) On-Demand Instaces
    - 한 세트의 스팟 인스턴스에 선택적으로 온디맨드 인스턴스를 조합
    - 정의된 비용 내에서 용량을 맞추려 노력
      - 사용가능한 런치풀을 통해서 실행(인스턴스 유형, OS, AZ)
      - 가장 적합하고 좋은 런치풀을 선택
      - 정해진 예산 혹은 원하는 용량에 달한 경우 인스턴스 실행을 멈춤
    - 스팟 인스턴스 할당 전략
      - lowestPrice : 스팟 플릿이 가장 적은 비용을 가진 풀에서 인스턴스를 실행(비용 최적화, shot workload)
      - diversified : 기존에 정의한 모든 풀에 걸쳐 분산(greate for availability, log workloads)
      - capacityOptimized : 인스턴스의 개수에 따라서 최적 용량으로 실행되고 적절한 풀을 찾아줌
- Dedicated Hosts - 전용 물리서버를 빌려 인스턴스 배치를 제어
- Dedicated Instances - 다른 고객과 하드웨어를 공유하지 않음
- Capacity Reservations - 특정한 AZ에 용량을 예약

### Elastic IP

- EC2 인스턴스를 종료하고 시작할 때 바뀌지 않는 IP
- 계정 당 5개를 가질 수 있음
- 사용하지 않는 것이 좋음

### Elastic Network Interfaces(ENI)

- VPC에서 사용하는 가상의 네트워크 카드
- 생성할 때 지정한 AZ에서만 사용 가능
- failover에 유용

### Placement Groups

- EC2 인스턴스를 어디에 배치할 지 제어
- 배치 그룹 전략
  - Cluster - 단일 가용 영역 내에서 지연 시간이 짧은 하드웨어 설정
    - 동일한 하드웨어, 동일한 가용영역에 존재
  - Spread - 인스턴스가 다른 하드웨어에 분산
    - 다른 하드웨어, 다른 가용영역에 존재
  - Partition - 여러 파티션에 인스턴스가 분할되어 있음
    - 다른 랙, 다른 가용 영역에 존재

### EC2 Hibernate

- EC2 stop, terminate
  - stop - EBS 볼륨 유지
  - terminate - EBS(root) 볼륨이 삭제되게 했으면 인스턴스 삭제
- 재시작
  - OS boot & EC2 User Data script run
  - OS boots up
  - application 시작, 캐시 구성
- Hibernate(절전모드)
  - RAM에 있던 인메모리 그대로 유지
  - boot가 빠름
  - RAM에 기록된 인메모리는 root 볼륨에 유지되기 때문에 암호화 필요
  - bare metal 인스턴스에서는 사용 불가
  - EBS 볼륨에서만 사용 가능

## Storage

### EBS

- 인스턴스에서 동작 중 사용되는 네트워크 드라이브
- 한번에 하나의 EC2 인스턴스에만 마운트 가능(일부 EBS 다중 연결 가능)
- EBS는 생성한 가용 영역에서만 사용 가능
  - 스냅샷을 이용해 다른 가용 영역에서 똑같이 생성 가능
- EC2 인스턴스 생성 과정 중 인스턴스 종료 시 EBS 볼륨 삭제 옵션이 존재
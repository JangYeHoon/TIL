# 유데미 AWS 강의

- [IAM](#identity-and-access-management(iam))
- [EC2](#elastic-compute-cloud(ec2))
- [AMI](#amazon-machine-image(ami))
- [EBS](#elastic-block-store(ebs))
- [EFS](#elastic-file-system(efs))
- [ELB](#elastic-load-balancer(elb))
- [ASG](#auto-scaling-group(asg))

## Identity and Access Management(IAM)

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

#### Password Policy

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

### AWS 접근 방법

- AWS Management Console - password + MFA
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




## Elastic Compute Cloud(EC2)

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


### EC2 Instances 구매 옵션

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

### EC2 Hibernate(절전모드)

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

## Amazon Machine Image(AMI)

### AMI Overview

- Amazon Machine Image
- EC2 인스턴스를 통해 만든 이미지
- AMI을 통해 다른 리전에서 인스턴스 생성 가능
- AMI 종류
  - Public AMI : AWS 제공
  - Your own AMI : 자체적으로 생성한 AMI
  - AWS Marketplace AMI : 기업에서 자체적으로 만든 AMI

### AMI Process

- 커스텀 EC2 인스턴스 시작 가능
- 데이터를 보존하면서 인스턴스 stop 가능
- AMI 빌드 - EBS snapshots 생성
- AMI를 통한 인스턴스 생성

## Elastic Block Store(EBS) 

### EBS

- 인스턴스에서 동작 중 사용되는 네트워크 드라이브
- 한번에 하나의 EC2 인스턴스에만 마운트 가능(일부 EBS 다중 연결 가능)
- EBS는 생성한 가용 영역에서만 사용 가능
  - 스냅샷을 이용해 다른 가용 영역에서 똑같이 생성 가능
- EC2 인스턴스 생성 과정 중 인스턴스 종료 시 EBS 볼륨 삭제 옵션이 존재

### EBS Volume Types

- gp2/gp3(SSD) : 가장 일반적인 성능과 비용의 SSD
- io1/io2(SSD) : 높은 성능의 SSD
  - multi attach 기능 제공
- st1(HDD) : 저비용의 HDD
- sc1(HDD) : 가장 비용이 작은 HDD
- Size | Throughput | IOPS 를 통해 EBS 볼륨을 나눔
- 인스턴스의 boot volume으로는 gp2/gp3, io1/io2만 사용 가능

#### General Purpose SSD

- 짧은 지연 시간과 비용 효율적인 볼륨
- 1G ~ 16T 지원
- gp3
  - 기본 3000IOPS, 125MiB/s 처리량
  - 최대 16,000 IOPS, 최대 1000MB/s 처리량
- gp2
  - gp3보다 볼륨이 작고 최대 3000IOPS 지원
  - 볼륨과 IOPS가 연결되어 있어서 볼륨 크기를 늘리면 최대 16,000 IOPS 지원

#### Provisioned IOPS SSD

- IOPS 성능을 유지할 필요가 있을 때 적합
- 16,000 IOPS 이상을 요할 때 적합
- io1/io2(4G ~ 16T)
  - Nitro EC2 인스턴스인 경우 최대 64000 IOPS 지원, 이외의 인스턴스는 최대 32,000 IOPS 지원
  - IOPS를 스토리지 크기와 상관없이 늘리 수 있음
  - io2는 동일한 비용으로 io보다 더 노퓨은 내구성과 기가바이트 당 IOPS가 더 높음
- io2 Block Express(4G ~ 64TB)
  - 고성능 유형의 볼륨
  - 지연시간이 밀리초 미만
  - IOPS:GiB 비율이 1000:1 일 때 최대 256,000 IOPS 지원

#### Hard Disk Drives

- 부트 볼륨 사용 불가능
- 125MB ~ 16TB
- 볼륨 유형
  - Throughput Optimized HDD (st1)
    - 최대 500MB 처리량, 최대 500 IOPS
  - Cold HDD(sc1)
    - 접근 빈도가 낮은 데이터에 적합
    - 최저 비용
    - 최대 처리량은 250M, 최대 250 IOPS

#### EBS Multi-Attach - io1/io2 family

- 하나의 EBS 볼륨을 같은 AZ에 있는 여러 EC2 인스턴스에 연결해주는 기능
- 각 인스턴스는 읽기/쓰기 권한을 가짐
- 16개의 인스턴스만 연결 가능
- 클러스터 인식 파일 시스템에서만 사용 가능(XFS, EX4 등의 파일 시스템에서 사용 불가)

### EBS Encryption

- 볼륨 내부, 데이터 이동, 스냅샷, 스냅샷을 통해 생성한 모든 볼륨 암호화  가능
- KMS의 키를 통해 EBS 볼륨 암호화(AES-256)

### EBS 스냅샷

- EBS의 특정 시점의 백업
- 다른 AZ나 리전에 복사 가능

#### EBS Snapshots Features

- 스냅샷 아카이브는 최대 75% 저렴하게 스냅샷을 보관
  - 복원하는데 24에서 72시간 정도 소요
- EBS 스냅샷 휴지통
  - EBS 스냅샷을 영구 삭제하지 않고 휴지통에 삭제
  - 보관 기간은 1일에서 1년 중 선택 가능
- Fast Snapsshot Restore(FSR)
  - EBS나 인스턴스를 빠르게 초기화할 때 유용
  - 매우 비쌈

### EC2 Instance Store

- EBS 볼륨은 좋지만 성능에 제한이 있음
- EC2 Instance Store를 사용하면 높은 성능의 하드웨어 디스크를 사용할 수 있음
  - 물리적으로 연결되어 있는 디스크
- Better I/O performance
- 인스턴스를 중지하거나 종료하면 스토리지 손실
  - 임시 스토리지

## Elastic File System(EFS)

### Amazon EFS(Elastic File System)

- EFS는 관리형 NFS로 여러 EC2 인스턴스에 마운트 가능
- EFS는 여러 AZ에 있는 EC2 인스턴스에 마운트 가능
- HA, 확장성이 좋고 가격이 비쌈
- EFS는 접근 제어를 위해 보안그룹이 존재
- 윈도우는 지원하지 않고 Linux 기반 AMI만 호환
- KMS를 사용해 EFS 드라이브의 데이터 암호화 가능
- Linux의 표준 파일 시스템인 POSIX 파일 시스템이고 standard file API 사용
- 용량을 지정하지 않고 사용하는 만큼 용량이 증가되며 사용한 GB 당 데이터 비용 지불

### Performance

- EFS Scale
  - 수천 개의 NFS 클라이언트에서 EFS에 동시 접근 가능하고 처리량은 10GB 이상
  - PB 규모로 자동 확장
- Performance mode
  - general purpose(default) : 지연시간에 민감할 때 유용
  - Max I/O : 지연 시간, 처리량, 병렬 처리 성능 향상
- Throughput mode
  - Bursting : 1TB 파일시스템 = 50MB/s + 100MB/s burst
  - Provisioned : 스토리지 크기에 상관없이 처리량을 설정 가능

### Storage Classes

- Storage Tiers (일정 기간 후 파일을 다른 계층으로 옮김)
  - Standard : 접근이 빈번한 파일들 저장
  - Infrequent access(EFS-IA) : 저장 비용은 작지만 파일 검색 시 비용 발생, 수명 주기 정책 설정 필요
- Availability and durability
  - Standard : 다중 AZ 설정
  - One Zone : 단일 AZ, 기본적으로 백업 활성화, EFS-IA와 호환 가능(EFS One Zone-IA)

## EBS vs EFS

#### EBS

- EBS 볼륨은 하나의 인스턴스에만 연결
- 하나의 AZ에만 존재
- 다른 AZ로 옮기기 위해서는 snapshot 사용
- 인스턴스가 종료되면 EBS root 볼륨도 삭제(삭제 안되 게 설정 가능)

#### EFS

- EFS는 여러 가용영역에 걸쳐 많은 인스턴스에 연결 가능
  - Mount Target을 이용해 연결
- Linux 인스턴스에만 연결 가능
- EBS보다 비쌈
- EFS-IA를 사용하면 저렴하게 사용 가능
- 사용한 만큼 요금을 지불

## Elastic Load Balancer(ELB)

### High Availability

- 동일 애플리케이션의 동일 인스턴스를 다수의 AZ 걸쳐 실행
- 고가용성은 애플리케이션이나 시스템을 두 개의 AZ나 데이터 센터에서 가동하는 것을 의미
- 손실이나 장애를 막기 위해 사용

### Load Balancing

- 네트워크 트래픽을 여러 서버에 분산해주는 서비스
- 부하를 분산하기 위해 사용
- 애플리케이션의 단일 액세스 지점(DNS) 노출
- 인스턴스의 장애를 원할히 처리
- ELB는 managed load balancer
  - aws가 관리하며 어떤 경우에도 작동할 것을 보장

#### Health Checks

- 연결된 인스턴스의 상태를 계속해서 확인

#### Types of load balancer on AWS

- Application Load Balancer (v2)
  - HTTP, HTTPS, WebSocket 지원
- Network Load Balancer(v2)
  - TCP, TLS, UDP 지원
- Gateway Load Balancer
  - 3계층(Network Layer)에서 IP Protocol 사용

#### Load Balancer Security Groups

- 유저는 HTTP, HTTPS를 통해 LB에 접근 가능
- 인스턴스는 LB만 접근 가능하도록 보안그룹 설정

### Application Load Balancer(v2)

- Application Load Balancer는 7계층에서 동작(HTTP)
- 다수 HTTP 어플리케이션으로 라우팅되어 부하분산
- 동일 머신의 다수의 어플리케이션을 부하분산(ex. containers)
- HTTP/2, Web Socket 지원
- 라우팅 테이블 지원
  - URL의 경로 기반으로 라우팅
  - URL의 호스트 네임을 기반으로 라우팅
  - 쿼리와 헤더를 기반으로 라우팅
- 포트 매핑 기능을 지원하기 때문에 동적 포트로의 리다이렉션 가능

#### Target Groups

- EC2 인스턴스들의 그룹(Auto Scaling Group을 통해 관리 가능)
- ECS task
- Lambda functions
- IP Addresses
- ALB는 여러 target group으로 라우팅 가능

#### Good to Know

- 호스트네임은 고정(XXX.region.elb.amazonaws.com)
- 애플리케이션 서버는 클라이언트의 IP를 직접 확인하지 못함
  - 클라이언트 IP는 X-Forwarded-For 헤더에 삽입
  - X-Forwarded-Port에서 사용 포트를 X-Forwarded-Proto에서 사용 프로토콜을 확인 가능

### Network Load Balancer(v2)

- 네트워크 로드 밸런서는 L4 로드 밸런서
  - TCP & UDP 트래픽을 제어
  - 초당 수백만건의 요청을 처리 가능
  - ALB에 비해 지연 시간이 짧음(NLB - 100ms, ALB - 400ms)
- 네트워크 로드 밸런서는 가용 영역별로 하나의 고정 IP를 가짐

#### Target Groups

- EC2 인스턴스
- IP 주소 - private IP만 사용 가능
- Application Load Balancer
- TCP, HTTP와 HTTPS 프로토콜 헬스체크 지원

### Gateway Load Balancer

- 배포 및 확장과 AWS의 타사 네트워크 가상 어플라이언스의 플릿 관리에 사용
- 네트워크의 모든 트래픽이 방화벽, IDS, IPS, Deep Packet Inspection, payload 수정(네트워크 레벨) 등의 모든 곳을 통과하도록 설계 가능
- 레이어 3인 네트워크 계층에서 동작 - IP packet
- GWLB의 기능
  - Transparent Network Gateway - VCP의 모든 트래픽이 GWLB를 지나기 때문
  - Load Balancer - target group인 자신의 가상 어플라이언스 집합(방화벽, IPS 등)에 트래픽을 분산 
- 6081번 포트의 GENEVE 프로토콜을 사용

#### Target Groups

- EC2 Instance
- IP Addresses - private IP만 사용 가능

### Sticky Sessions

- Client가 처음 통신을 시도한 EC2 인스턴스로 계속해서 통신을 유지할 수 있도록 세션 유지
- Application Load Balancers에서 사용 가능
- 쿠키를 이용하여 제어

#### Cookie Names

- Application-based Cookies
  - Custom Cookie
    - 애플리케이션에서 생성한 Custom Cookie
    - 애플리케이션에 필요한 모든 사용자 정의 속성을 포함
    - 쿠키 이름은 대상 그룹별로 개별적으로 지정
  - Application cookie
    - 로드밸런서에서 생성
    - 생성한 쿠키 이름은 AWSALBAPP
- Duration-based Cookie
  - 로드밸런서에서 생성한 쿠키
  - ALB의 쿠키 이름은 AWSALB

### Cross-Zone Load Balancing

- 다른 AZ에 존재하는 모든 EC2 인스턴스에 트래픽을 분배
- Aplication Load Balancer
  - 기본적으로 Cross-Zone Load Balancing 활성화(Target Group 레벨에서 비활성화 가능)
  - 다른 AZ로 옮기는 비용 발생하지 않음
- Network Load Balancer & Gateway Load Balancer
  - 기본적으로 비활성화
  - AZ간 데이터 옮기는 비용 발생

### SSL/TLS

#### Basic

- SSL 인증서는 클라이언트와 로드 밸런서 사이에서 트래픽이 이동하는 동안 암호화(in-flight 암호화)
- SSL은 보안 소켓 계층(Secure Sockets Layer)을 의미하고 연결을 암호화하는데 사용
- TLS는 새로운 버전의 SSL로 전송 계층 보안(Transport Layer Security)을 의미
- 최근에는 TLS 인증서가 많이 사용

#### Load Balancer - SSL Certificates

- 로드밸런서는 X.509 인증서를 사용(SSL/TLS 서버 인증서)
- AWS에서는 ACM(AWS Certificate Manager)에서 인증서들을 관리할 수 있음
- 개인이 가지고 있는 인증서를 ACM에 업로드 가능
- HTTPS 리스너
  - 기본 인증서로 지정
  - 다중 도메인을 지원하기 위해 다른 인증서를 추가 가능
  - 클라이언트는 SNI(Server Name Indication)을 통해 호스트의 이름을 알릴 수 있음

#### SNI(Server Name Indication)

- SNI는 여러 개의 SSL 인증서를 하나의 웹서버에 로드해 여러 개의 웹사이트를 지원
- 확장된 프로토콜로 클라이언트가 대성 서버의 호스트 이름을 지정
- ALB & NLB(신버전), CloudFront에서만 지원

### Deregistration Delay

- 인스턴스가 등록 취소, 혹은 비정상인 상태일 때 인스턴스에 어느정도 시간을 주어 'in-flight request'(기존 연결 및 요청)를 완료할 수 있도록 하는 기능
- 등록 취소(de-registering)중인 EC2 인스턴스에 새로운 요청을 보내지 않음
- 1~3600초 사이로 설정 가능(기본 300초)

## Auto Scaling Group(ASG)

### Scalability

- Vertical Scalability(수직 확장)
  - 인스턴스 크기 확장(scale up / down)
  - t2.micro -> t2.large로 확장
- Horizontal Scalability
  - 애플리케이션이나 인스턴스의 수를 확장
  - 인스턴스의 수 확장(scale out / in)

### Auto Scaling Group

- 실시간으로 변하는 로드에 맞춰 서버를 생성하거나 종료
- ASG의 목표
  - 증가한 로드에 맞춰 Scale out(EC2 인스턴스 추가)
  - 감소한 로드에 맞춰 Scale in(EC2 인스턴스 제거)
  - 실행하는 EC2 인스턴스에 대한 최대, 최소 크기 지정
  - 로드밸런서와 페어링 시 새로 생성되는 인스턴스에 대해 자동으로 로드밸런서에 연결
  - 비정상 인스턴스가 생기면 종료하고 새로운 EC2 인스턴스 생성
- ASG는 무료(EC2 인스턴스와 같은 ,생성된 하위 리소스에 대한 비용만 지불)
- Auto Scaling Group과 Load Balancer를 함께 사용

#### Auto Scaling Group Attributes

- Launch Template을 생성
  - AMI + Instance Type
  - EC2 User Data
  - EBS Volumes
  - Security Groups
  - SSH Key Pair
  - IAM 역할
  - VPC + Subnets
  - 로드밸런서 정보
- 최소/최대/초기 용량 정의
- Scaling Policies

#### CloudWatch Alarms & Scaling

- CloudWatch 경보를 기반으로 ASG를 Scale in & out 가능
- metric(지표)를 모니터링해 경보 발생(cpu, memory, 사용자 지정 지표 등)
- ​
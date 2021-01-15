# 01. Overview

## 클라우드의 역사

- 1960년대 가상화라는 용어 사용
  - 서버 가상화 기술이 퍼블릭하게 발전한게 클라우드
- 다양한 하이퍼바이저의 출현
  - IBM의 Logical Partition
    - IBM의 유닉스 머신에 사용되는 하이퍼 바이저
  - VMWare, Xen, Hyper-V, KVM
  - 하이퍼바이저의 발전으로 클라우드의 기술들도 같이 발전



## 공공 클라우드

- 공공 클라우드 인증을 받은 클라우드
  - 국내에서 공공 클라우드 인증을 받은 6개의 기업 중 하나
- 국내 최초의 전문 금융 퍼블릭 클라우드
  - 금융 및 핀테크 기업들이 믿고 맡기는 클라우드



## 왜 클라우드를 사용하는가?

- 비용 절감
  - 데브옵스,  TCO, ROI 측면에서 리소스를 효율적으로 사용하여 리소스를 줄임
  - 서비스 트래픽에 맞게 탄력적으로 인프라를 운영할 수 있음
  - IaaS:오토스케일링, SaaS:쿠버네티스, PaaS:클라우드 funtion
  - 다양한 어플리케이션과 서비스를 제공하고 있음
- 빠른 Deploy
  - 서버 한대 만드는데 5분
- 글로벌 진출시 용이
  - 어플리케이션들은 국내뿐만 아니라 해외에서도 많이 사용
- 보안



## 네이버 클라우드 플랫폼 구성 요소

- 홈페이지
  - 사용하기 쉬운 UX
    - 접근하기 편하고 직관적이도록 개선
- 블로그
  - 공식 블로그를 통해 다양한 기술, 트랜드, 신상품 등을 확인 가능
- 유튜브
  - 신규 상품 소개, Webinar 내용

 

## 네이버 클라우드 플랫폼 전체 상품 라인업

- 하드웨어
  - 인프라
  - 하이브리드 클라우드, 클라우드 커넥트(Cloud Connect),  Vmware on NCP
  - 레거시 인프라와 연동할 수 있는 상품 존재
- 6개의 리전
  - 서버, 스토리지, 네트워크
  - 데이터베이스, 아날리티스, 미디어, 게임
  - AI 서비스, DEV TOOLS, Application Service
  - 전체 상품을 관리하는 MANAGEMENT
  - 외부로부터 위협을 막는 SECURITY



## 네이버 클라우드 플랫폼 인프라 상품군

- Compute
  - 서버 상품
  - CPU와 메모리, 하드디스크를 제공하는 서버
  - GPU를 제공하는 GPU 서버
  - 하나의 물리 서버에 단일 VM만 생성해서 사용하는 VDS
  - 물리 서버 자체를 제공해주는 베어메탈(Bare-metal)
  - 컨테이너와 관련된 Container Registry, Kubernetes Service
  - Auto Scailing, NPC
- Storage
  - Block Storage, Object Storage, NAS
  - 데이터들을 복원해주는Backup
  - 다른 데이터들을 좀 더 오래 보관하고자 할 때 사용하는 Archive Storage
  - 레거시 데이터에서 빠르게 네이버 클라우드로 전환해주는 Data Teleporter
- Networking
  - 서비스의 트래픽을 분산시켜주는 LoadBalancer
  - 레거시 인프라와 클라우드 플랫폼을 사설 IP로 연결해주는IPsec VP
  - DNS, NAT Gateway, Global Route Manager, CDN, Global CDN
- Global
- Hybrid
  - 기존 레거시 인프라도 유지해야 되면 사용
  - Hybrid Cloud Hosting : 사용자의 서버 리소스를 네이버 클라우드의 IDC에 위치시켜 사용자의 레거시 인프라와 클라우드의 다양한 상품을 같이 사용
  - Cloud Connect : 레거시 인프라와 네이버 클라우드 플랫폼을 사설 IP로 연결
  - Vmware on NCP



## 네이버 클라우드 플랫폼 플랫폼 상품군

- Database
  - 설치형 DB : Maria DB, MySQL, MSSQL, Cubrid
    - 서버 이미지에 데이터베이스를 같이 설치
    - 사용자가 데이터 베이스를 설치할 필요없이 바로 사용
  - 관리형 DB : Cloud DB for MySQL, Cloud DB for MSSQL, Cloud DB for Redis
    - 관리 자체, 구성 자체를 네이버 클라우드에서 함
    - 사용자는 인스턴스에 접속할 수 있는 권한만 받음
    - HA, Backup, Monitoring을 지원
- Analytics
  - CLA, ELSA, RUA, Cloud Hadoop, Cloud Search
- Media
  - Live Transcoder, VOD Transcoder, Image Optimizer
- Game
  - Gamepot



## 네이버 클라우드 플랫폼 어플리케이션 상품군

- AI Service
  - 파파고와 클로바를 b2b 형식으로 제공
- DevTools
  - SourceCommit, SourceBuild, SourceDeploy
    - CICD를 구성할 수 있음
- Application Service
  - 독립적인 서비스를 제공
  - APIGateway, GeoLocation, Maps, Search Trend, 단축 URL, CAPTCHA
- Biz Application



## 상품 의존성

- 클라우드에서는 기본적으로 존과 리전이 생기면서 상품들이 구분됨
- 존 의존 상품
  - 서버, 클라우드 디비
  - KR1존과 KR2존이 존재하는데, 이 둘을 선택하는 상품들에 대해서만 존 의존 상품
- 리전 의존 상품
  - 로드밸런서, 오브젝트 스토리지
  - 존을 변경해도 설정된 내용이 동일하게 보이면 리전 의존 상품
- 리전 통합 상품
  - DNS, ACG
  - 리전을 선택하고 상품을 선택했을 때 다른 곳에서 설정한 내용이 그대로 보이면 리전 통합 상품
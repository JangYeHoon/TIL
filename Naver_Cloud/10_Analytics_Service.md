# 10. Analytics Service

- Analytics는 사용자의 데이터를 분석하는데 가장 큰 목적이 있고 분석을 하는데 있어서 데이터의 유형에 따라 분석하는 방법이 다름



## Analytics

- Analytics는 데이터를 분석하고 서버의 지표를 분석하고 서버의 서비스 지표를 확인하고 분석하는 도구
- 서비스와 관련된 다양한 로그를 수집하고 분석
  - ELSA를 통해 모바일 앱에서 발생하는 크래시를 수집하고 분석
- 시스템의 다양한 로그를 수집하고 분석
  - CLA를 통해 시스템의 로그를 수집하고 분석
- 웹 페이지에 대한 통계 분석
  - RUA를 통해 웹페이지의 방문자 통계를 확인



## ELSA

- Effective Log Search & Analytics
  - Application Crash 모니터링 서비스
  - ELSA의 SDK/API를 이용하여 어플리케이션을 쉽게 저장하고 검색, 분석할 수 있는 로그 분석 툴
  - 특정 로그 발생시 알람을 보내는 이벤트 기능과 App Crash Report도 제공 예정
    - 사용자가 어플리케이션에서 Crash를 피할 수 있는 방법을 가이드



## CLA

- 시스템 로그 수집 분석 플랫폼
  - NCP안에 있는 서버들의 다양한 로그들을 수집하고 분석해주는 서비스
  - Agent 기반으로 동작
  - Syslog, Apache Log, MySQL Log, Tomcat Log, Windows Event Log, MS-SQL error Log 수집
  - 커스텀 로그 기능을 통해 직접 로그 대상 지정 가능
    - 커스텀 로그 기능을 이용할 때는 반드시 텍스트 파일이여야 하며 해당 텍스트 파일이 쌓이게 되면 CLA에서는 CLA Agent가 CLA 서버에 전송
  - Object Storage와 연계되어 로그파일 보관 기능 제공
  - CLA는 기본적으로 100GB 제공
- CLA의 정첵
  - CLA는 사용자별로 100GB 용량을 할당
  - 사용자의 로그가 많이 쌓이면 CLA는 가장 오래된 30%의 로그를 삭제



## RUA

- 웹페이지 접속자 분석 도구
  - 웹서비스에서 웹 페이지로 누가 들어왔는지, PV는 얼마고 UV는 얼마인지 데이터를 확인할 필요가 있음.
    - 그리고 시간대 별로 유저가 어떤 브라우저를 쓰고 있는지 좀 더 자세한 클라이언트에 대한 정보를 뽑아낼 필요가 있음
    - 이때 사용하는 도구가 **RUA**
  - 구글 애널리틱스와 유사한 상품으로 웹페이지 접속자에 대한 분석, 통계
    - 구글 애널리틱스와 네이버 애널리틱스는 각각 구글과 네이버로 데이터를 전송하게 됨
    - 타 벤더로 데이터를 전송하는게 우려되면 NCP의 RUA를 사용하여 NCP에 데이터를 쌓고 사용자가 분석할 수 있게 사용자만의 페이지 제공



## Cloud Hadoop

- 빅데이터 분석 도구
  - 하둡 클러스터를 보다 쉽고 편리하게 생성 및 관리
  - Apache Ambari를 기본 제공하여 하둡 클러스터와의 관리 및 모니터링을 효율적
  - 4가지 타입의 Cloud Hadoop 제공
    - Core Hadoop
      - Hadoop의 가장 큰 기본 베이스 컴포넌트인 HDFS, YARN, HIVE, Hue
    - HBase
      - HDFS, YARN, HIVE, HBase, Zookeeper
    - Presto(페이스북)
      - Presto, HDFS, YARN, HUE, Zeppelin Notebook
    - Spark
      - HDFS, YARN, spark, spark2, HIVE, Hue, Zepplin Notebook



## Cloud Search

- 사용자의 웹사이트에 검색 기능 제공
  - 다양한 인덱싱 구성 옵션 제공
  - 다국어 및 불용어, 동의어 처리 기능 제공
  - 네이버 형태소 분석 처리기를 기반으로 한국어 처리
  - 네이버의 검색 엔진을 쓰는 것과 동일한 효과를 사이트에서 이용할 수 있음
  - NCP의 API를 통해 검색



## Elasticsearch Service

- Elasticsearch 클러스터를 손쉽게 배포, 보호, 운영 및 확장하여 로그분석, 검색, app 모니터링 등을 수행할 수 있도록 제공하는 완전 관리형 서비스
  - Elasticsearch service 클러스터는 1대의 매니저 노드와 3대 이상의 데이터 노드로 구성(데이터 노드 수는 설치 시 원하는 만큼 증설 가능)
    - 사용자가 원하는 성능에 맞춰 데이터 노드 설치 가능
  - Elasticsearch service는 데이터 분석 및 시각화 플랫폼인Kibana와 연계되어 데이터를 빠르고 정확하게 분석 가능



## Global Region

- 네이버 클라우드 플랫폼 리전간 전용선 응답시간을 실시간으로 제공
  - 전용선은 응답시간의 편차가 거의 없음
  - https://www.ncloud.com/product/globalRegion/globalLatencyStatus
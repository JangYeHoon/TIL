## AWS Summit Seoul 참석 정리

- 2023년 5월 4일에 참석한 AWS Summit Seoul 발표 정리 내용 입니다.

**1. 가격은 저렴, 성능은 최대로! 확 달라진 Amazon EC2 알아보기**

- AWS EC2의 독점 가상화 인프라인 Nitro 시스템 아키텍처 소개

- 새롭게 출신하는 인스턴스 유형 소개

   => HPC 고성능 컴퓨팅 특화, 기계학습 특화, Graviton 3세대 등 새로운 유형 소개

- 삼성전자에서 ARM 기반 프로세서인 AWS Graviton 3세대로 변경하여 서비스 절감 및 성능 향상 사례 소

**2. Amazon EKS 데이터 전송 비용 절감 및 카오스 엔지니어링 적용 사례**

- Istio Locality LB와 Topology Aware Hints를 활용해 비용 절감

- 다른 가용영역의 pod로 전송하는 비용을 절감
- 카오스 엔지니어링 서비스인 AWS Fault Injection Simulator를 활용해 EKS 안전성 검증 소개

**3. 성공적인 AWS RDS 마이그레이션을 위한 여정과 필수 고려사항**

- DB 마이그레이션을 위한 단계별 계획 수립 방법 소개
- Assess -> Plan -> Test -> Execute
- DB 마이그레시션 시 고려 사항 및 마이그레이션 툴 소개
    -> 고려사항 : 백업, 호환성, 제약사항, 3rd파티 SW 이관 대체 방안, 시간
    -> 마이그레이션 툴 : OS commands, DBMS Provided, Change Data Capture

- 마이그레이션 케이스 소개
    -> MS-SQL을 AWS My SQL RDS로 이관, 온프레미스의 Oracle RAC DB를 AWS RDS로 이관

**4. 지속적인 혁신과 발전, AWS 네트워킹이 이끄는 미래**

- VPC, 리전, 가용 영역에 대한 간단한 소개
- VPC 엔드포인트, VPC 피어링, AWS Transit Gateway 서비스 소개
- 글로벌 연결을 위한 Direct Connect, Cloud WAN, Accelerated S2S VPN 서비스 소개
- 새로운 서비스인 AWS Verified Access, Amazon VPC Lattice 서비스 소개
    -> Verified Access : VPN 없이 간편한 연결을 지원하는 보안 서비스
    -> VPC Lattice : 다중 VPC 연결 및 배포 일관성을 지원하는 서비스

**5. KINX CloudHub를 통한 AWS Direct Connect 연동 : 네트워크 확장 방안**

- KINX에서 Direct Connect와 연동을 위해 제공하는 기술들 소개

**6. 지능화되는 랜섬웨어 위협으로부터 지킬 것인가? 당할 것인가?**

- AWS 랜섬웨어 방어를 위한 전략 소개
    -> 서브넷 분리, NACL 및 보안그룹 활용
    -> 방화벽 서비스 인 Route 53 Resolver DNS Firewall, Network Firewall, WAF 활용
    -> System Manager 서비스를 이용한 자동 패치 및 공격 탐지
    -> CloudWatch, Config, GuardDuty를 이용한 지속적인 모니터링
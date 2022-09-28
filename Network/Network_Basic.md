# 네트워크 기본 정리

- [OSI 7계층과 TCP/IP 계층](#osi-7계층과-tcp/ip-계층)

## OSI 7계층과 TCP/IP 계층

### OSI 7계층

| 계층 번호 | 계층                 | 데이터(PDU) | 구분        |
| ----- | ------------------ | -------- | --------- |
| 7     | Application Layer  | Data     | 애플리케이션 계층 |
| 6     | Presentation Layer | Data     | 애플리케이션 계층 |
| 5     | Session Layer      | Data     | 애플리케이션 계층 |
| 4     | Transport Layer    | Segments | 데이터 플로 계층 |
| 3     | Network Layer      | Packets  | 데이터 플로 계층 |
| 2     | DataLink Layer     | Frames   | 데이터 플로 계층 |
| 1     | Physical Layer     | Bits     | 데이터 플로 계층 |

### TCP/IP 프로토콜 스택

| 계층 번호 | TCP/IP 모델            |
| ----- | -------------------- |
| 4     | Application Layer    |
| 3     | Transport Layer      |
| 2     | Internet Layer       |
| 1     | Network Access Layer |

- TCP/IP 모델은 현실에 쉽게 반영하도록 간단히 4계층으로 구분
- 상위 3개 계층(Session, Presentation, Application)을 하나의 Application Layer로 묶고 1, 2계층을 하나의 Network Access Layer로 묶음

### OSI 계층 별 특징

####Physical Layer

- 전기 신호를 전달하는데 초점이 맞추어져 있다.
- 주요 장비 - 허브, 리피터, 케이블, 커넥터, 트랜시버, 탭
- 들어온 전기 신호를 그대로 전달하는 것이 목적
- 주소 개념이 없기 때문에 들어온 포트를 제외하고 모든 포트에 같은 전기 신호를 전송

#### DataLink Layer

- 전기 신호를 모아 사람이 알아볼 수 있는 데이터 형태로 처리
- **MAC이라는 주소**를 가지고 있기 때문에 출발지와 도착지 주소를 확인해 자신한테 온게 맞는지 확인 후 데이터 처리
- 동시에 여러 명과 통신할 수 있으므로 받는 사람이 현재 데이터를 받을 수 있는지 확인하고 전송
  - **Flow Control**
- 주요 장비 - 네트워크 인터페이스 카드(NIC), 스위치

#### Network Layer

- 논리적인 주소인 IP 주소를 가짐
- MAC 주소와 달리 IP 주소는 사용자가 환경에 맞게 변경 가능
- IP 주소를 통해 3계층 장비는 자신이 속한 네트워크와 원격지 네트워크를 구분
- 주요 장비 - 라우터

#### Transport Layer

- 데이터가 정상적으로 목적지까지 잘 보내졌는지 확인하는 역할
- 패킷 네트워크는 데이터를 분할하여 패킷 단위로 보내기 때문에 중간에 유실되거나 순서가 바뀌는 경우가 발생하는데 이러한 문제를 해결
- 주요 용어
  - 시퀀스 번호(Sequence Number) - 패킷에 순서를 명시
  - ACK 번호(Acknowledgement Number) - 받는 순서를 명시
  - 포트 번호(Port Number) - 장치 내의 Application 구분
- 주요 장비 - 로드 밸런서, 방화벽

#### Session Layer

- 끝단의 응용 프로세스가 연결이 성립되도록 돕고 안정적으로 유지되도록 관리하고 통신이 끝나면 해당 연결을 끊는 역할
- TCP/IP 세션을 만들고 없애고 에러로 중단된 통신에 대한 에러 복구와 재전송도 수행

#### Presentation Layer

- 표현 방식이 다른 Application이나 시스템 간의 통신을 하나의 통일된 구문 형식으로 변환하는 기능을 수행
- MIME 인코딩이나 암호화, 압축, 코드 변환과 같은 동작을 수행

#### Application Layer

- Application 프로세스를 정의하고 Application 서비스 동작

### OSI 계층 별 주요 프로토콜

| 계층                 | 주요 프로토콜                                  | 장비                |
| ------------------ | ---------------------------------------- | ----------------- |
| Application Layer  | HTTP, SMP, SMTP, STUN, TFTP, TELNET, HTTPS | ADC, NGFW, WAF    |
| Presentation Layer | TLS, AFP, SSH                            |                   |
| Session Layer      | L2TP, PPTP, NFS, RPC, RTCP, SIP, SSH     |                   |
| Transport Layer    | TCP, UDP, SCTP, DCCP, AH, AEP            | 로드밸런서, 방화벽        |
| Network Layer      | ARP, IPv4, IPv6, NAT, IPSec, VRRP, Routing | 라우터, L3 스위치       |
| DataLink Layer     | IEEE 802.2, FDDI                         | 스위치, 브릿지, 네트워크 카드 |
| Physical Layer     | RS-232, RS-449, V.35, S 등의 케이블           | 케이블, 허브, 탭(TAP)   |


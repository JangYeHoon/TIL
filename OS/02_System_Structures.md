# 02. System Structures

## Operating-System Services

- 운영체제는 사용자와 시스템에게 다양한 서비스를 제공
  - UI(User Interface)
    - 사용자와 컴퓨터 시스템이 만나는 지점
    - 인터페이스는 크게 CLI(Command-Line Interface)와 배치 인터페이스(Batch Interface), GUI(Graphical User Interface)로 나뉨
      - CLI : 사용자가 텍스트 명령을 통해 명령을 내리는 인터페이스(셸 : shll)
      - 배치 인터페이스 : 명령을 파일에 포함시켜 파일이 실행되면 명령을 실행하는 인터페이스
      - GUI : 현재 가장 많이 쓰는 인터페이스로 사용자는 키보드, 마우스, 터치 등 다양한 방법으로 그래픽을 조작
  - 프로그램 실행(Program Execution)
    - 시스템은 프로그램을 메모리에 로드하고 이를 실행할 수 있어야함
  - 입출력 명령(I/O Operations)
    - 운영체제는 입출력 명령을 수행하는데 이때 효율과 보안을 위해 사용자가 직접입출력 장치를 조작하지 않고 자신을 거치게 함
  - 파일 시스템 조작(File-System Manipulation)
    - 파일을 쓰고, 읽고, 만들고 지움
  - 통신(Communications)
    - 프로세스끼리 정보를 교환할 때 운영체제는 공유 메모리(Shared Memory)나 메시지 패싱(Message Passing) 사용
    - 공유 메모리는 메모리의 한 부분을 공유
    - 메시지 패싱은 패킷을 주고받는 것(메시지 패싱 방식이 더 느림)
  - 에러 탐지(Error Detection)
- 시스템을 위한 서비스
  - 자원 할당(Resource Allocation)
  - 회계(Accounting)
    - 시스템은 어떤 유저가 어떤 종류의 자원을 얼마나 사용하고 있는지 계속 추적해야 함
  - 보호와 보안(Protection and Security)



## System Calls

- 시스템 콜은 커널과 사용자 프로그램을 이어주는 인터페이스
- OS에서 제공하는 서비스에 대한 프로그래밍 인터페이스
- 사용자 프로그램이 시스템에 접근하기 위해서는 커널 모드로 전환할 때 시스템 콜을 사용
- 일반적으로 고급 언어로 작성(C/C++)
- 시스템 호출을 직접 호출하지 않고 API를 통해 프로그램에서 주로 액세스
  - Win32 API, POSIX API, Java API
- 시스템 호출보다 API를 사용하는 이유
  - 프로그램 이식성
  - 사용하기 어려움
- 메모리의 특정 주소 범위에는 동작들이 할당됨
  - 이를 시스템 콜 테이블(System call table)이라고 부르며, 인터럽트 벡터(Interrupt vector)라고도 부름
- 시스템 콜은 메모리 주소의 모음인데 메모리 주소는 인터럽트 서비스 루틴(Interrupt Service Routine)을 가리키고 있음
  - 인터럽트 서비스 루틴은 시스템 콜 테이블이 가리키는 특정 메모리 주소가 어떤 동작을 할지 정의해 놓은 것
- 시스템 콜 함수를 직접 조작은 불편하고 위험해 표준 라이브러리 사용
- 사용자 프로그램이 운영체제에게 매개변수를 넘기는 방법
  - Call by Value
  - Call by Reference
  - 프로그램은 스택에 매개변수 추가, 운영체제를 통해 값을 뺌
- 일반적으로 각 시스템 호출과 관련된 번호
  - 시스템 호출 인터페이스는 이 숫자에 따라 색인된 테이블을 유지
- 시스템 호출 인터페이스는 OS 커널에서 의도한 시스템 호출을 호출하고 시스템 호출 상태 및 반환 값을 반환

*참고*

- 시스템 콜 구현 코드
- https://blog.naver.com/jkjk010jkjk/222082419801

![API-System Call-OS Relationship](images/API-System%20Call-OS%20Relationship.png)



## System Call Parameter Passing

- OS에 매개변수를 전달하는데 사용되는 세 가지 일반적인 방법
  - 매개변수는 레지스터에 전달
    - 경우에 따라 레지스터보다 더 많은 매개변수가 필요
  - 메모리의 블록(또는 테이블)에 저장된 매개변수와 블록의 주소는 레지스터에 매개변수로 전달
  - 매개변수는 프로그램에 의해 스택에 푸시되고 운영체제에 의해 스택에서 팝됨
- 블록 및 스택 방법은 전달할 매개변수의 수 또는 길이를 제한하지 않음

![System Call Parameter Passing](images/System%20Call%20Parameter%20Passing.png)



## Types of System Calls

- 시스템 콜 구분
  - 프로세스 제어 : end, abort, load, execute
  - 파일 관리 : create, delete, open, close, read, write
  - 장치 관리 : read, write, request, release
  - 정보 유지 : get/set time or date
  - 통신 : send/receive messages, transfer status
  - 보호



## Operating System Structure

- 현대 운영체제는 계층을 나눠서 시스템을 관리
- 범용 OS는 매우 큰 프로그램
- 구조화하는 다양한 방법
  - Simple Structure : MS-DOS
  - More Complex : UNIX
  - Layered : an abstraction
  - Microkernel : Mach



### Simple Structure

- MS-DOS에서는 사용자 프로그램이 입출력 루틴에 접근해 직접 쓰기를 할 수 있었음
- 만약 사용자 프로그램에 문제가 생기면 전체 시스템에 문제가 생김
- UNIX 시스템은 이르 개선해 기능을 분리하였지만 여전히 하나의 계층이 일을 많이 함
- 하드웨어 계층 위, 사용자 계층 아래에 있는 커널이 모든 기능을 제공
- 이러한 모놀리틱(Monolithic) 구조는 구현과 유지보수가 어려움
- 최소한의 공간에서 가장 많은 기능을 제공하도록 작성됨
  - 모듈로 분리되지 않음
  - 일부 구조가 있지만 인터페이스와 기능 수준이 잘 분리되어 있지 않음



### Not Simple Structure - UNIX

- UNIX
  - 하드웨어 기능에 의해 제한됨
  - 원래 UNIX 운영체제는 구조화가 제한적
- UNIX OS는 분리 가능한 두 부분으로 구성
  - 시스템 프로그램
  - 커널
    - 시스템 호출 인터페이스 아래 및 물리적 하드웨어 위의 모든 것으로 구성
    - 파일 시스템, CPU 스케줄링, 메모리 관리 및 기타 운영체제 기능을 제공
    - 한 수준에 많은 기능이 있음



### Layered Approach

- 운영체제를 더 세분화해 계층을 분리한 것
- 가장 아래 계층이 하드웨어, 높은 계층은 사용자  인터페이스
- 유지보수가 편하고 하나의 계층을 수정해도 다른 계층까지 신경쓰지 않아도 됨
- **Benefits**
  - 구성 및 디버깅의 단순성
- **Detriments**
  - 다양한 레이어를 적절하게 정의하기 어려움
  - 다른 유형보다 덜 효율적
    - 시스템 호출은 모든 하위 계층을 거쳐야 하므로 각 계층에서 추가되는 오버헤드가 발생



### Microkernels

- 마이크로커널은 커널에서 핵심적인 요소만 남긴 가벼운 커널
- 커널에서 "사용자" 공간으로 이동
- 메시지 전달을 사용하여 사용자 모듈 간에 통신이 이루어짐
- **Benefits**
  - 코드 양이 적어 컴파일, 테스트 시간이 짧고 다른 시스템에 이식하기 쉬움
  - 마이크로 커널 확장이 쉬움
  - 안전함
- **Detriments**
  - 시스템 프로그램을 추가해 기능을 늘리면 속도가 느려짐
  - 커널 공간 통신에 대한 사용자 공간의 오버 헤드



### Modules

- 많은 최신 운영체제는 로드 가능한 커널 모듈을 구현
- 객체 지향 접근 방식 사용
- 각 핵심 구성 요소는 분리되어 있음
- 모듈은 커널을 확장하기 위한 기술
  - 장치드라이버는 모두 모듈로 구현되어 있음
- 커널 전체를 컴파일할 필요없이 모듈만 컴파일하면 되고 필요할 때만 모듈을 동적으로 링크시켜 커널의 일부로 사용할 수 있어 효율적
- 모듈을 적용하면 자주 사용하지 않는 기능은 메모리에 상주시키지 않아도 됨
- 모듈의 사용은 확장성과 재사용성을 높일 수 있음
- 계층 접근 방식(Layered Approach)과 비교
  - 각 커널 섹션에 인터페이스가 정의되고 보호된다는 점에서 계층화된 접근 방식과 유사
  - 모든 모듈이 다른 모듈을 호출할 수 있어 더 유연
- 마이크로 커널 방식과 비교
  - 기본 모듈이 다른 모듈을 로드하고 통신하는 방법에 대한 지식과 핵심 기능ㅁ나 있다는 점에서 마이크로 커널 접근 방식과 유사
  - 모듈 간 통신에 메시지 전달이 필요하지 않다는 점에서 마이크로 커널 방식보다 우수



### Hybrid Systems

- 커널의 핵심만 남기고 나머지는 따로 구현한 시스템
- OS X
  - BSD가 핵심이지만 나머지는 모두 애플이 구현
- 안드로이드
  - 리눅스 커널 위에 자체 구현한라이브러리를 올린 시스템



##### Reference

- https://parksb.github.io/article/6.html
- Provided by Operating Systems Concepts, 10th Edition
# 17. ONOS Code Walkthrough

- ONOS Northbound API
  - Web GUI
    - http://onos_ip:8181/onos/ui
  - CLI
    - local : karaf clean
    - remote : onos_ip
  - REST API
    - http://onos_ip:8181/onos/v1



## OSGi Bundles & Karaf Features

- OSGi Bundles
  - 번들 자체적으로 이름과 버전을 가짐
  - 번들 자체적으로 필요한 자바 라이브러리 임포트 가능
  - 번들에서 구현된 자바 패키지를 export 가능
- Karaf Features
  - 번들들을 그룹으로 설치/삭제를 쉽게하기 위해 만들어짐
  - XML 파일 형식



## Service Component Runtime(SCR)

- 싱글톤을 쉽게 구현하기 위해 annotation을 이용해서 구현
  - maven-scr-plugin이 컴파일 단계에서 OSGI-INF/*.xml 파일들을 추출해 관리
- `@Services`를 이용해 컴포넌트 싱글톤으로 생성
- `@Reference` 다른 싱글톤 인스턴스에서 참조하고 싶으면
- `@Activate`, `@Modified`, `@Deactivate` 해당 액션이 수행될 때 메소드 수행



## ONOS Application

- Maven archetypes
  - 어플리케이션을 자동으로 만들어줌
  - `onos-api-archetype`
  - `onos-bundle-archetype`
  - `onos-cli-archetype`
  - `onos-ui-archetype`
  - `onos-uitab-archetype`
  - `onos-uitopo-archetype`
- `mvn archetype:generate`를 이용해 생성
  - `onos-create-app` 같은 명령어

- 어플리케이션은 .oar 형식을 가짐
- ONOS 어플리케이션 관리 방법
  - REST API
  - shell tool
  - CLI
  - GUI



## BYON Application

- Bring Your Own Network

  - 스스로 가상 네트워크를 spawn할 수 있는 어플리케이션
    - 모든 디바이스가 연결된 full mesh 네트워크라고 가정
  - CLI commands를 이용해 BYON과 연결
    - `list-networks` : 생성된 가상 네트워크 나열
    - `create-network` : 가상 네트워크 생성(샘플코드로 제공)
    - `add-host` : 호스트 추가(샘플코드로 제공)
    - `remove-host` : 호스트 제거(구현)
    - `remove-network` : 네트워크 제거(구현)

  ![image-20210223135909563](images/image-20210223135909563.png)



### Import & Build BYON App

- Follow Lab#1 of the Distributed Tutorial on the ONOS Wiki
  - https://goo.gl/5ezwoI


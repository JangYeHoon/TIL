# Auto Scaling Group and Application Load Balancer(NGINX)

- EC2에 NGINX를 이용한 웹을 구성하고 로드 밸런서와 오토 스케일링으로 처리



### EC2에 NGINX 설치 및 웹 구성

- 인스턴스 구성

  - ubuntu server 20.04 선택
  - ![image-20210311104621320](images/image-20210311104621320.png)

  - 인스턴스 구성

    ![image-20210311104649156](images/image-20210311104649156.png)

  - 보안 그룹에 HTTP 추가

    ![image-20210311104730967](images/image-20210311104730967.png)

  - 키페어 설정하고 인스턴스 생성

    ![image-20210311104759787](images/image-20210311104759787.png)

- nginx 설치 및 웹 구성

  - ip를 이용해 ssh 접속

    ![image-20210311104940369](images/image-20210311104940369.png)

  - web 폴더 생성

    `mkdir web`

  - react에서 생성한 build 폴더 옮기기

    ![image-20210311105036373](images/image-20210311105036373.png)

    ![image-20210311105147559](images/image-20210311105147559.png)

  - nginx 설치

    `sudo apt-get update`

    `sudo apt-get install nginx`

  - 옮긴 build에 있는 파일들 /var/www/html/ 밑으로 복사

    `sudo cp -r ./ /var/www/html/`

    ![image-20210311105401533](images/image-20210311105401533.png)

  - nginx 실행

    `sudo service nginx start`

  - ip 접속 확인

    ![image-20210311105515010](images/image-20210311105515010.png)

### 생성한 인스턴스를 이용해 AMI 이미지 생성 및 시작 템플릿 설치

- 이미지 생성

  - 이미지 생성 클릭

  ![image-20210311105610714](images/image-20210311105610714.png)
  - 이미지 이름 설정하고 생성

  ![image-20210311105644123](images/image-20210311105644123.png)
  - 생성한 이미지 상태 available 확인

  ![image-20210311110023683](images/image-20210311110023683.png)

- 시작 템플릿 생성

  - 메뉴의 시작 템플릿 - 생성

    ![image-20210311110146256](images/image-20210311110146256.png)

  - 템플릿 설정

    ![image-20210311110243844](images/image-20210311110243844.png)

  - AMI는 만들어놓은 이미지 선택

    ![image-20210311110319305](images/image-20210311110319305.png)

  - 키페어, 보안그룹 모두 인스턴스와 동일하게

    ![image-20210311110401573](images/image-20210311110401573.png)

  - 템플릿 생성 완료

    ![image-20210311110538768](images/image-20210311110538768.png)



### 로드밸런서 생성

- 로드밸런서 생성 클릭

  <img src="images/image-20210311110612615.png" alt="image-20210311110612615" style="zoom:50%;" />

- Application Load Balancer 선택

  <img src="images/image-20210311110632552.png" alt="image-20210311110632552" style="zoom:50%;" />

- 기본 구성

  ![image-20210311110657721](images/image-20210311110657721.png)

  ![image-20210311110709828](images/image-20210311110709828.png)

- 보안 그룹 설정

  ![image-20210311110728020](images/image-20210311110728020.png)

- 라우팅 구성

  <img src="images/image-20210311110812015.png" alt="image-20210311110812015" style="zoom:50%;" />

  ![image-20210311110825917](images/image-20210311110825917.png)

- 로드밸런서 생성 완료

  ![image-20210311110851963](images/image-20210311110851963.png)

- active 상태 확인

  ![image-20210311111012301](images/image-20210311111012301.png)



### 오토 스케일링 그룹 생성

- 그룹 생성 클릭

  ![image-20210311111054462](images/image-20210311111054462.png)

- 이름과 시작 템플릿 선택

  ![image-20210311111118788](images/image-20210311111118788.png)

- 시작 템플릿 준수 선택, 네트워크 설정

  <img src="images/image-20210311111143378.png" alt="image-20210311111143378" style="zoom:50%;" />

- 생성한 로드밸런서 선택

  ![image-20210311111229978](images/image-20210311111229978.png)

- 모니터링 활성화 선택하고 다음

  ![image-20210311111245501](images/image-20210311111245501.png)

- 그룹 크기 선택

  <img src="images/image-20210311111309921.png" alt="image-20210311111309921" style="zoom:50%;" />

- 정책 설정

  <img src="images/image-20210311111327899.png" alt="image-20210311111327899" style="zoom:50%;" />

- Auto Scaling 그룹 생성 완료

  ![image-20210311111359401](images/image-20210311111359401.png)



### 테스트

- 오토 스케일링 인스턴스 생성 완료 확인

  ![image-20210311111424365](images/image-20210311111424365.png)

  ![image-20210311111813659](images/image-20210311111813659.png)

- 로드밸런서 DNS 주소 확인

  <img src="images/image-20210311111736073.png" alt="image-20210311111736073" style="zoom:50%;" />

- DNS 주소로 접속, 웹 화면 구동 확인

  ![image-20210311111836287](images/image-20210311111836287.png)
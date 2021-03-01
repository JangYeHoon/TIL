# 04. Creating an Auto Scaling Group and Application Load Balancer in AWS

- Elastic Load Balancer와 Auto Scaling 그룹을 이용한 웹 서비스 설계

- 스트레스 테스트를 이용해 Auto Scaling 작동 테스트

  ![image-20210301220120248](images/image-20210301220120248.png)



### Create an Application Load Balancer

- EC2 콘솔 - Load Balancer - 로드밸런서 생성

  <img src="images/image-20210301220328010.png" alt="image-20210301220328010" style="zoom:50%;" />

- Application Load Balancer 선택

  ![image-20210301220409660](images/image-20210301220409660.png)

- 다음과 같이 구성

  <img src="images/image-20210301220522387.png" alt="image-20210301220522387" style="zoom:50%;" />![image-20210301220537846](images/image-20210301220537846.png)<img src="images/image-20210301220522387.png" alt="image-20210301220522387" style="zoom:50%;" />![image-20210301220537846](images/image-20210301220537846.png)

- 새 보안 그룹 구성하고 다음
- ![image-20210301220645910](images/image-20210301220645910.png)

- 라우팅 구성과 고급상태설정하고 생성

  ![image-20210301220737409](images/image-20210301220737409.png)

  ![image-20210301220915286](images/image-20210301220915286.png)

- 로드밸런서 생성 완료

  ![image-20210301220950748](images/image-20210301220950748.png)



### Create a Launch Template

#### Create SSH Key Pair

- 키페어 생성

  ![image-20210301221051420](images/image-20210301221051420.png)

- pem으로 설정하고 생성

  <img src="images/image-20210301221128523.png" alt="image-20210301221128523" style="zoom:50%;" />

- 키 저장

#### Create Security Group for EC2 Instances

- 보안 그룹 생성

  ![image-20210301221231519](images/image-20210301221231519.png)

- 보안 그룹 이름과 인바운드 규칙 설정

  <img src="images/image-20210301221407155.png" alt="image-20210301221407155" style="zoom:50%;" />![image-20210301221425128](images/image-20210301221425128.png)

- 생성 완료

#### Create EC2 Instance Launch Template

- 시작 템플릿 생성

  ![image-20210301221555547](images/image-20210301221555547.png)

- 아래와 같이 설정

  <img src="images/image-20210301221832607.png" alt="image-20210301221832607" style="zoom:50%;" />

  <img src="images/image-20210301222043646.png" alt="image-20210301222043646" style="zoom:50%;" />

  <img src="images/image-20210301222151550.png" alt="image-20210301222151550.png" style="zoom:50%;" />

- 마지막 맨밑에 Advanced details section - user data box

  ```
  #!/bin/bash
  yum update -y
  yum install -y httpd
  yum install -y wget
  cd /var/www/html
  wget https://raw.githubusercontent.com/ACloudGuru-Resources/Course-Certified-Solutions-Architect-Associate/master/labs/creating-an-auto-scaling-group-and-app-load-balancer-aws/index.html
  wget https://raw.githubusercontent.com/ACloudGuru-Resources/Course-Certified-Solutions-Architect-Associate/master/labs/creating-an-auto-scaling-group-and-app-load-balancer-aws/acg.jpg
  service httpd start
  ```

  - 다음 내용 추가

- 생성완료



### Create an Auto Scaling Group

- 생성한 로드밸런서 active 상태인지 확인

  ![image-20210301222638138](images/image-20210301222638138.png)

- Auto Scaling 그룹 생성

  ![image-20210301222725045](images/image-20210301222725045.png)

- 이름과 시작 템플릿 설정

  ![image-20210301222806831](images/image-20210301222806831.png)

- 시작 템플릿 준수 선택 및 2개 서브넷 모두 선택

  <img src="images/image-20210301222900205.png" alt="image-20210301222900205" style="zoom:50%;" />

- 생성한 로드밸런서와 연결

  ![image-20210301223019309](images/image-20210301223019309.png)

  ![image-20210301223038202](images/image-20210301223038202.png)

- 그룹 크기와 조정 사항 선택

  <img src="images/image-20210301223128942.png" alt="image-20210301223128942" style="zoom:50%;" /><img src="images/image-20210301223157329.png" alt="image-20210301223157329" style="zoom:50%;" />

- 로드밸런서 DNS 주소 복사

  ![image-20210301223347370](images/image-20210301223347370.png)

- 접속 확인

  <img src="images/image-20210301223409178.png" alt="image-20210301223409178" style="zoom:50%;" />



### Test Horizontal Scaling

- 생성된 인스턴스 중 하나 접속

  ![image-20210301223634968](images/image-20210301223634968.png)

- 테스트를 위한 스트레스 어플리케이션 설치

  ```
  $ sudo amazon-linux-extras install epel -y
  $ sudo yum install -y stress
  ```

- 스트레스 테스트

  ```
  $ stress --cpu 2 --timeout 300
  ```

  ![image-20210301223857247](images/image-20210301223857247.png)

- CPU 사용률 높이기 전 인스턴스 2개

  ![image-20210301223928101](images/image-20210301223928101.png)

- 오토 스케일링을 통해 인스턴스 증가

  ![image-20210301224803220](images/image-20210301224803220.png)
# 8. Creating an EC2 Instance with Lambda in AWS

- Lambda : 함수만으로 서비스를 구축할 수 있는 일종의 Serverless Architecture

- EC2 인스턴스를 생성하는 람다 함수를 파이썬으로 작성

- IAM 역할에 대한 사용자 지정 람다 실행 정책 생성 

  <img src="images/image-20210314204451184.png" alt="image-20210314204451184" style="zoom:50%;" />



## Create EC2 Key Pair

- EC2 - 키 페어 - 키페어 생성 클릭

  ![image-20210314204537160](images/image-20210314204537160.png)

- 아래와 같이 설정하고 생성

  ![image-20210314204611456](images/image-20210314204611456.png)



## Create a Lambda Function

- Lambda - 함수 - 함수 생성

  ![image-20210314204717647](images/image-20210314204717647.png)

- 아래와 같이 설정하고 함수 생성

  ![image-20210314204909781](images/image-20210314204909781.png)

  ![image-20210314204923641](images/image-20210314204923641.png)

- 함수 생성 완료

  ![image-20210314205026260](images/image-20210314205026260.png)

- 새 탭을 열어서 IAM - 역할에서 생성한 역할 이름을 검색해 역할 선택

  ![image-20210314205203400](images/image-20210314205203400.png)

- 권한에 연결된 정책 선택

  ![image-20210314205254365](images/image-20210314205254365.png)

- `{} JSON` 선택하고 정책 편집 선택

  ![image-20210314205341099](images/image-20210314205341099.png)

- JSON 선택하고 밑에 내용 복붙

  ```
  {
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Action": [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ],
        "Resource": "arn:aws:logs:*:*:*"
      },
      {
        "Action": [
          "ec2:RunInstances"
        ],
        "Effect": "Allow",
        "Resource": "*"
      }
    ]
  }
  ```

  ![image-20210314205438013](images/image-20210314205438013.png)

- 정책 저장 완료

  ![image-20210314205527432](images/image-20210314205527432.png)

- 다시 Lambda 콘솔 페이지로 와서 새로고침

- CreateEC2 function 페이지에서 스크롤을 내려 아래 코드로 수정하고 Deploy

  ```python
  import os
  import boto3
  
  AMI = os.environ['AMI']
  INSTANCE_TYPE = os.environ['INSTANCE_TYPE']
  KEY_NAME = os.environ['KEY_NAME']
  SUBNET_ID = os.environ['SUBNET_ID']
  
  ec2 = boto3.resource('ec2')
  
  
  def lambda_handler(event, context):
  
      instance = ec2.create_instances(
          ImageId=AMI,
          InstanceType=INSTANCE_TYPE,
          KeyName=KEY_NAME,
          SubnetId=SUBNET_ID,
          MaxCount=1,
          MinCount=1
      )
  
      print("New instance created:", instance[0].id)
  ```

  ![image-20210314205741929](images/image-20210314205741929.png)

- 다음으로 구성 - 환경변수 - 편집 선택

  ![image-20210314210009833](images/image-20210314210009833.png)

- 입력하기 전에 새 탭을 열어 EC2 - 인스턴스 시작 - Amazon Linux2의 ami 값 복사

  ![image-20210314210215523](images/image-20210314210215523.png)

- 다음으로 VPC - 서브넷에서 서브넷 ID 복사

  ![image-20210314210414006](images/image-20210314210414006.png)

- 복사한 값드를 채워 환경변수 추가

  ![image-20210314210435132](images/image-20210314210435132.png)



## Test Lambda Function

- Test 선택

  ![image-20210314210601662](images/image-20210314210601662.png)

- 다음과 같이 입력하고 생성 

  ![image-20210314210655605](images/image-20210314210655605.png)

- 테스트클릭

  ![image-20210314210727294](images/image-20210314210727294.png)

- 테스트 결과

  ![image-20210314210830314](images/image-20210314210830314.png)

- EC2 - 인스턴스 생성 확인

  ![image-20210314210921200](images/image-20210314210921200.png)



## Connect to the Instance via SSH 

- 생성된 인스턴스 IP로 SSH 접속

  ![image-20210314211029564](images/image-20210314211029564.png)

- 처음 생성한 키로 접속

  ![image-20210314211121967](images/image-20210314211121967.png)

- 접속 확인

  ![image-20210314211148752](images/image-20210314211148752.png)


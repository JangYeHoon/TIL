# 9. Creating a Basic Lambda Function to Shut Down an EC2 Instance

- lambda function을 이용해 실행 중인 EC2 인스턴스 중지
  ![image-20210314212321583](images/image-20210314212321583.png)





## Create a Basic Lambda Function and a Custom IAM Policy for an IAM Role

- EC2 - 인스턴스 - 실행중인 인스턴스 확인

  ![image-20210314212450658](images/image-20210314212450658.png)

- 실행중인 인스턴스 ID 주소 복사

  ![image-20210314214611941](images/image-20210314214611941.png)

- Lambda - 함수 - 함수 생성 클릭

  ![image-20210314212608636](images/image-20210314212608636.png)

- 함수 설정전에 새 탭열어서 IAM 콘솔에서 역할 생성 

  ![image-20210314212819374](images/image-20210314212819374.png) 

- AWS 서비스 - Lambda 선택

  ![image-20210314212839549](images/image-20210314212839549.png)

- 태그에 아래와 같이 설정

  ![image-20210314213338754](images/image-20210314213338754.png)

- 역할 이름 설정하고 생성 

  ![image-20210314212944802](images/image-20210314212944802.png)

- 생성한 역할 선택

  ![image-20210314213014537](images/image-20210314213014537.png)

- 권한 - 정책연결

  ![image-20210314213029687](images/image-20210314213029687.png)

- 정책 생성 선택

  ![image-20210314213117311](images/image-20210314213117311.png)

- JSON 선택하고 아래 스크립트 넣기

  ```
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ],
        "Resource": "arn:aws:logs:*:*:*"
      },
      {
        "Effect": "Allow",
        "Action": [
          "ec2:Stop*"
        ],
        "Resource": "*"
      }
    ]
  }
  ```

  <img src="images/image-20210314213155370.png" alt="image-20210314213155370" style="zoom:50%;" />

- 정책 이름 설정하고 생성 

  ![image-20210314213219685](images/image-20210314213219685.png)

- 생성한 정책 연결 

  ![image-20210314213711219](images/image-20210314213711219.png)

- 생성중이던 Lambda 함수로 돌아와서 아래와 같이 기존 역할 사용 

  ![image-20210314213829812](images/image-20210314213829812.png)

  ![image-20210314213840981](images/image-20210314213840981.png)

- 함수 생성 완료

  ![image-20210314213907443](images/image-20210314213907443.png)



## Create a Test in the Lambda Console

- 테스트 선택

  ![image-20210314214041379](images/image-20210314214041379.png)

- 이벤트 이름 설정하고 생성

  ![image-20210314214116847](images/image-20210314214116847.png)

- 함수 소스 코드 변경

  - `instanes = ['<instance id>']` <instance id>에 복사한 id 넣기

  ```python
  import boto3
  #This simple lambda function is available from AWS with instructions on starting and stopping an instance at regular intervals using Lambda and CloudWatch: https://aws.amazon.com/premiumsupport/knowledge-center/start-stop-lambda-cloudwatch/
  # Enter the region your instances are in. Include only the region without specifying Availability Zone; e.g., 'us-east-1'
  region = 'us-east-1'
  # Enter your instances here: ex. ['X-XXXXXXXX'] you can comma separate the instance IDs for more than one instance: i.e. ['X-XXXXXXXXX', 'X-XXXXXXXXX"]
  instances = ['<instance id>']
  
  def lambda_handler(event, context):
      ec2 = boto3.client('ec2', region_name=region)
      ec2.stop_instances(InstanceIds=instances)
  ```

- 변경하고 Deploy

  ![image-20210314214401906](images/image-20210314214401906.png)

- 인스턴스 실행 중 확인 

  ![image-20210314214435944](images/image-20210314214435944.png)

- 다시 Test

  ![image-20210314214459402](images/image-20210314214459402.png)

- 테스트 성공

  ![image-20210314214518145](images/image-20210314214518145.png)

- 인스턴스 확인

  ![image-20210314214633887](images/image-20210314214633887.png)
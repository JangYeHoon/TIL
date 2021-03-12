# 7. Using EC2 Roles and Instance Profiles in AWS

- 보안 자격증명을 관리할 필요없는 IAM을 이용하여 수동 관리없이 인스턴스 tempory credential을 자동으로 부여

  ![image-20210312112048287](images/image-20210312112048287.png)



## Create a Trust Policy and Role Using the AWS CLI

### Obtain the labreferences.txt File

- S3 콘솔에서 버킷 리스트 중 이름 중간에 *s3bucketlookupfiles* 들어간 버킷 선택

  ![image-20210312112452006](images/image-20210312112452006.png)

- labreferences.txt 선택

  ![image-20210312112519322](images/image-20210312112519322.png)

- 다운로드

  ![image-20210312112550788](images/image-20210312112550788.png)

### Log in to Bastion Host and Set the AWS CLI Region and Output Type

- EC2 콘솔 - 인스턴스 - Bastion Host 인스턴스 IP 확인

  ![image-20210312112825903](images/image-20210312112825903.png)

- ssh로 접속

  <img src="images/image-20210312112945842.png" alt="image-20210312112945842" style="zoom:50%;" />

- 비밀번호는 lab 페이지에 있는 번호

  ![image-20210312113031175](images/image-20210312113031175.png)

- 접속 완료

  <img src="images/image-20210312113108752.png" alt="image-20210312113108752" style="zoom:50%;" />

- AWS 설정 변경

  `$ aws configure`

  ![image-20210312113204337](images/image-20210312113204337.png)

### Create IAM Trust Policy for an EC2 Role

- trust_policy_ec2.json 파일 생성(EC2 인스턴스에 대한 신뢰 정책)

  `$ vim trust_policy_ec2.json`

  ```
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {"Service": "ec2.amazonaws.com"},
        "Action": "sts:AssumeRole"
      }
    ]
  }
  ```

  <img src="images/image-20210312113403993.png" alt="image-20210312113403993" style="zoom:50%;" />

### Create the DEV IAM Role

- DEV IAM 생성

  `$ aws iam create-role --role-name DEV_ROLE --assume-role-policy-document file://trust_policy_ec2.json`

  <img src="images/image-20210312113812799.png" alt="image-20210312113812799" style="zoom:50%;" />

### Create an IAM Policy Defining Read-Only Access Permissions to an S3 Bucket

- S3 Bucket에 대한 읽기 권한을 가진 IAM 정책 생성

- dev_s3_read_access.json 파일 생성

  - <DEV_S3_BUCKET_NAME>은 labreferences.txt파일에 있는 내용으로 변경

  `$ vim dev_s3_read_access.json`

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
            "Sid": "AllowUserToSeeBucketListInTheConsole",
            "Action": ["s3:ListAllMyBuckets", "s3:GetBucketLocation"],
            "Effect": "Allow",
            "Resource": ["arn:aws:s3:::*"]
          },
          {
              "Effect": "Allow",
              "Action": [
                  "s3:Get*",
                  "s3:List*"
              ],
              "Resource": [
                  "arn:aws:s3:::<DEV_S3_BUCKET_NAME>/*",
                  "arn:aws:s3:::<DEV_S3_BUCKET_NAME>"
              ]
          }
      ]
  }
  ```

- DevS3ReadAccess 정책 생성

  `$ aws iam create-policy --policy-name DevS3ReadAccess --policy-document file://dev_s3_read_access.json`

  ![image-20210312114517255](images/image-20210312114517255.png)

- ARN 복사해서 labreferencess.txt에 붙여넣기



## Create Instance Profile and Attach Role to an EC2 Instance

### Attach Managed Policy to Role

- 관리 정책과 Role 연결

  - <DevS3ReadAccess_POLICY_ARN>은 마지막에 복사한 ARN 넣기

  `$ aws iam attach-role-policy --role-name DEV_ROLE --policy-arn "<DevS3ReadAccess_POLICY_ARN>"`

- 정책 연결 확인

  `$ aws iam list-attached-role-policies --role-name DEV_ROLE`

  ![image-20210312114851973](images/image-20210312114851973.png)

### Create the Instance Profile and Add the `DEV_ROLE` via the AWS CLI

- 인스턴스 프로파일 DEV_PROFILE 생성 

  `$ aws iam create-instance-profile --instance-profile-name DEV_PROFILE`

  ![image-20210312115346062](images/image-20210312115346062.png)

- role 추가

  `$ aws iam add-role-to-instance-profile --instance-profile-name DEV_PROFILE --role-name DEV_ROLE`

- 설정 수정

  `$ aws iam get-instance-profile --instance-profile-name DEV_PROFILE`

  <img src="images/image-20210312115444862.png" alt="image-20210312115444862" style="zoom:50%;" />

### Attach the DEV_PROFILE Role to an Instance

- EC2 - 인스턴스에서 Web Server 인스턴스 ID 복사
- ![image-20210312115643958](images/image-20210312115643958.png)

- 인스턴스와 DEV_PROFILE 연결

  - <LAB_WEB_SERVER_INSTANCE_ID>에는 복사한 인스턴스 ID 넣기

  `$ aws ec2 associate-iam-instance-profile --instance-id <LAB_WEB_SERVER_INSTANCE_ID> --iam-instance-profile Name="DEV_PROFILE"`

  ![image-20210312115749198](images/image-20210312115749198.png)

- 설정 확인

  `$ aws ec2 describe-instances --instance-ids <LAB_WEB_SERVER_INSTANCE_ID>`

- 출력에서 IamInstanceProfile 섹션을 찾고 Arn이 DEV_PROFILE로 끝나는지 확인

  ![image-20210312115952236](images/image-20210312115952236.png)



## Test S3 Permissions via the AWS CLI

- Web Server 인스턴스 IP 복사

  ![image-20210312125414173](images/image-20210312125414173.png)

- ssh 접속

  - 비밀번호는 lab 페이지

  ![image-20210312125516639](images/image-20210312125516639.png)
  <img src="images/image-20210312125527844.png" alt="image-20210312125527844" style="zoom:50%;" />

- 인스턴스가 DEV-ROLE 역할을 하고있는지 확인

  `$ aws sts get-caller-identity`

  ![image-20210312125631254](images/image-20210312125631254.png)

- buckets 리스트 확인

  `$ aws s3 ls`

  ![image-20210312125730798](images/image-20210312125730798.png)

- 파일 리스트 중 s3bucketdev-가 들어간 버킷의 이름 복사해서 <s3bucketdev-123>에 넣기

  `$ aws s3 ls s3://<s3bucketdev-123>`

- 버킷 접속

  ![image-20210312125920599](images/image-20210312125920599.png)



## Create an IAM Policy and Role Using the AWS Management Console

### Create Policy

- IAM - 정책 - 정책 생성

  <img src="images/image-20210312130550294.png" alt="image-20210312130550294" style="zoom:50%;" />

- JSON 탭 선택해서 밑에 내용 복사 붙여넣기

  - 이때 PROD_S3_BUCKET_NAME에는 labreferences.txt파일 내용  넣기

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
            "Sid": "AllowUserToSeeBucketListInTheConsole",
            "Action": ["s3:ListAllMyBuckets", "s3:GetBucketLocation"],
            "Effect": "Allow",
            "Resource": ["arn:aws:s3:::*"]
          },
          {
              "Effect": "Allow",
              "Action": [
                  "s3:Get*",
                  "s3:List*"
              ],
              "Resource": [
                  "arn:aws:s3:::<PROD_S3_BUCKET_NAME>/*",
                  "arn:aws:s3:::<PROD_S3_BUCKET_NAME>"
              ]
          }
      ]
  }
  ```

  ![image-20210312130816919](images/image-20210312130816919.png)

- 정책 이름 설정하고 정책 생성

  ![image-20210312130856000](images/image-20210312130856000.png)

### Create Role

- 역할 - 역할 만들기 선택

  <img src="images/image-20210312130939484.png" alt="image-20210312130939484" style="zoom:50%;" />

- EC2 선택하고 다음

  ![image-20210312131017460](images/image-20210312131017460.png)

- ProdS3ReadAccess 선택하고 다음

  ![image-20210312131057736](images/image-20210312131057736.png)

- 역할 이름 설정하고 생성 

  ![image-20210312131125046](images/image-20210312131125046.png)



## Attach IAM Role to an EC2 Inastance Using the AWS Management Console

- EC2 - 인스턴스에서 Web Server 인스턴스 선택

  ![image-20210312131306187](images/image-20210312131306187.png)

- 작업 - 보안 - IAM 역할 수정 선택

  ![image-20210312131343704](images/image-20210312131343704.png)

- PROD_ROLE 선택하고 저장

  ![image-20210312131414678](images/image-20210312131414678.png)



### Test the Configuration

- Web Server 인스턴스 ssh 접속

  ![image-20210312131549976](images/image-20210312131549976.png)

- 현재 사용중인 id와 Arn에 PROD_ROLE이 있는지 확인

  `$ aws sts get-caller-identity`

  ![image-20210312131651655](images/image-20210312131651655.png)

- 버킷 리스트 확인

  `$ aws s3 ls`

  ![image-20210312131820526](images/image-20210312131820526.png)

- 버킷 리스트 중에 s3bucketprod-이 들어간 버킷 확인

  - 접속 가능

  `$ aws s3 ls s3://<s3bucketprod-123>`

  ![image-20210312131831538](images/image-20210312131831538.png)

- 버킷 리스트 중 s3bucketsecret의 이름 확인해서 접속

  - 권한이 없어 접속 불가

  `$ aws s3 ls s3://<s3bucketsecret-123>`

  ![image-20210312132001625](images/image-20210312132001625.png)
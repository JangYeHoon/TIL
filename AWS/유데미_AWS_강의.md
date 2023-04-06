# 유데미 AWS 강의

- [IAM](#iam)

## IAM

- 글로벌 서비스
- 루트 계정은 자동으로 생성
- 유저를 생성해 회사나 그룹 포함할 수 있음
- 그룹에는 사용자만 추가 가능

### IAM Permissions

- 사용자나 그룹에 JSON을 이용해 정책 부여
- JSON의 정책을 통해 사용자에게 권한 부여
- AWS는 최소 권한 원칙(least privilege principle) 사용


### IAM Policies inheritance

- version number
- id : 선택사항
- statement : 하나 혹은 여러개
  - Sid : 문장의 Id, 선택사항
  - Effect : 문장이 특정 API 접근하는 걸 허용할지 거부할지에 대한 내용
  - Principal : 특정 정책이 적용될 사용자, 계정 역할이 정의
  - Action : 허용, 거부되는 API 호출 목록
  - Resource : 적용될 action의 리소스 목록
  - condition : statement가 언제 적용될지 정의, 선택사항

#### IAM - Password Policy

- 비밀번호를 이용해 계정을 보호하는 방법
- 비밀번호에 정책을 부여해 안전하게 보호

#### Multi Factor Authentication - MFA

- root 계정과 전체 IAM 유저들은 모두 보호해야함
- MFA = passowrd + security device
- MFA devices options
  - virtual MFA device = google auth(phone only), Authy(multi-device)
  - Universal 2nd Factor(U2F) Security Key - YubiKey by Yubico
  - Hardware Key Fod MFA Device - Provided by Gemalto
  - Hardware Key Fod MFA Device for AWS GovCloud(US) - Provided by SurePassID

### access AWS

- AWS Management Consolke - password + MFA
- AWS Command Line Interface(CLI) - access key
- AWS Software Developer Kit(SDK) - access key

#### IAM Roles for Services

- AWS서비스에 권한을 부여하는 기능
  - ex) ec2 인스턴스를 생성해서 다른 aws 서비스에 접근하려면 role이 필요
- Common roles
  - EC2
  - Lambda Function
  - Roles for CloudFormation

### IAM Security Tools

- IAM Credentials Report(account-level)
  - 유저들의 모든 정보를 볼 수 있음
- IAM Access Advisor(user-level)
  - 사용자에게 부여한 서비스의 권한과 해당 서비스에 마지막으로 접근한 시간이 보임

### IAM Guidelines & Best Practices

- 루트 계정은 AWS 계정을 설정하는 것을 제외하고 사용하지 않는다.
- 하나의 AWS 사용자 = 한명의 실제 사용자
- 그룹 수준에서 보안을 관리
- 강한 비밀번호 정책
- MFA 정책 사용
- AWS에 권한을 줄 때 role을 만들어 사용
- CLI/SDK를 사용하려면 Access Key 사용
- 계정의 권한을 체크할 때는 IAM Credentials Report & IAM Access Advisor를 사용
- 절대 IAM 사용자와 Access Key를 공유하지 않는다.


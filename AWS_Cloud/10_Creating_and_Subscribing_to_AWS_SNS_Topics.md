# 10. Creating and Subscribing to AWS SNS Topics

- AWS Simple Notification Service 사용
- Lambda Function을 이용해 구독하고 메일을 받음



## Create an SNS Topic

- SNS 서비스에서 주제 이름 설정하고 다음 단계 선택

  ![image-20210318154954589](images/image-20210318154954589.png)

- 수정없이 바로 주제 생성 클릭

  ![image-20210318155051727](images/image-20210318155051727.png)

- 주제 생성하고 구독 생성 클릭

  ![image-20210318155125505](images/image-20210318155125505.png)

- 프로토콜 이메일, 엔드포인트 메일 주소 입력하고 구독 생성

  ![image-20210318155216640](images/image-20210318155216640.png)

- 주제에서 생성한 토픽 선택

  ![image-20210318155330409](images/image-20210318155330409.png)

- 확인 대기 중

  ![image-20210318155409837](images/image-20210318155409837.png)

- 이메일에서 구독 수락

- ![image-20210318155452021](images/image-20210318155452021.png)

  ![image-20210318155506016](images/image-20210318155506016.png)



## Create a Lambda Function

- Lambda 콘솔에서 함수 생성

  ![image-20210318155600752](images/image-20210318155600752.png)

- 아래와 같이 설정하고 함수 생성

  ![image-20210318155701402](images/image-20210318155701402.png)

  ![image-20210318155717492](images/image-20210318155717492.png)

- 새 창열어서 SNS 서비스에서 구독 새로 생성

  ![image-20210318155918444](images/image-20210318155918444.png)

- 아래로 설정하고 구독 생성

  ![image-20210318155947982](images/image-20210318155947982.png)

- 다시 Lambda 탭으로 이동해서 아래 코드 입력하고 Deploy

  ```python
  def lambda_handler(event, context):
      message = event['Records'][0]['Sns']['Message']
      print("From SNS: " + message)
      return message
  ```

  ![image-20210318160151414](images/image-20210318160151414.png)



## Send Your SNS Topic to Multiple Endpoints

- SNS - 주제에서 생성한 주제 선택하고 메시지 게시 선택

  ![image-20210318160329831](images/image-20210318160329831.png)

- 제목과 내용 입력하고 메시지 게시

  ![image-20210318160437721](images/image-20210318160437721.png)

- 메일이 온 것을 확인

  ![image-20210318160518540](images/image-20210318160518540.png)

- Lambda 탭에서 모니터링에서 CloudWatch에서 로그 보기 선택

  ![image-20210318160658677](images/image-20210318160658677.png)

- 로그 스트림에 로그 확인

  ![image-20210318160745426](images/image-20210318160745426.png)

  ![image-20210318160829480](images/image-20210318160829480.png)
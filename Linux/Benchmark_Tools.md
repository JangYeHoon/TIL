## 1. Sysbench

### 1.1. 설치 방법

### 1.2. 명령어 사용 방법

- 기본 사용 방법


- command
  - prepare : 테스트 데이터 생성 과정
  - run : 테스트 실행
  - cleanup : 테스트 데이터 삭제 과정
  - prepare -> run -> cleanup 과정으로 실행
- 제공 testname
  - oltp_*.lua : OLTP 유형의 데이터베이스 벤치마크
  - cpu
  - memory
  - fileio
  - threads
  - mutex
- 기본 options

### 1.3. 명령어 예제

- cpu 벤치마킹



- memory 벤치마킹

![img](https://wiki.daoucloud.com/download/attachments/45154715/image2023-4-28_13-53-8.png?version=1&modificationDate=1682657588000&api=v2)

- fileio 벤치마킹

![img](https://wiki.daoucloud.com/download/attachments/45154715/image2023-4-28_13-53-33.png?version=1&modificationDate=1682657614000&api=v2)

- mysql 벤치마킹

![img](https://wiki.daoucloud.com/download/attachments/45154715/image2023-4-28_13-53-44.png?version=1&modificationDate=1682657624000&api=v2)

## 2. Stream

- 호스트의 전체 메모리 대역폭의 크기를 측정하는 벤치마크 툴
- 인위적으로 생성한 데이터를 CPU로 전송하여 메모리의 대역폭을 정량적으로 측정
- 벡터 커널의 계산 속도를 측정

### 2.1. 설치 방법

### 2.2. 사용 방법

### 2.3. 실행 결과

![img](https://wiki.daoucloud.com/download/attachments/45154715/image2023-4-28_13-54-17.png?version=1&modificationDate=1682657657000&api=v2)

다음 4가지 항목에 대한 메모리 대역폭이 나옴

| name  | kernel                     | 설명                                       |
| ----- | -------------------------- | ---------------------------------------- |
| Copy  | b[:] = a[:]                | measures transfer rates in the absence of arithmetic. |
| Scale | b[:] = const * a[:]        | adds a simple arithmetic operation.      |
| Add   | c[:] = a[:] + b[:]         | adds a third operand to allow multiple load/store ports on vector machines to be tested. |
| Triad | c[:] = a[:] + const * b[:] | allows chained/overlapped/fused multiply/add operations. |

- 보통 가장 복잡한 시나리오인 Triad의 대역폭으로 성능 비교
  - Triad는 복사, 곱셈, 덧셈이 모두 적용된 벡터 커널

### 2.4. 관련 자료

- <https://www.cs.virginia.edu/stream/ref.html>
- <https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=alice_k106&logNo=221317365992>
- <https://kbench.com/?q=node/1414>
- <https://mirhenge.tistory.com/entry/Memory-Becnchmark-stream>
- <https://documentation.sigma2.no/jobs/arm-perf/stream.html>
- <https://repository.kisti.re.kr/bitstream/10580/15548/1/%28%EA%B8%B0%EC%88%A0%29%EB%8B%A8%EC%9D%BC%20%EA%B0%80%EC%83%81%ED%99%94%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%ED%94%8C%EB%9E%AB%ED%8F%BC%20%EB%B2%A4%EC%B9%98%EB%A7%88%ED%81%AC%20%ED%85%8C%EC%8A%A4%ED%8A%B8%EB%B2%A0%EB%93%9C%20%EA%B5%AC%EC%B6%95%20%EB%B0%8F%20%EC%8B%9C%ED%97%98.pdf>

## 3. K6

- API endpoint 테스트 툴

### 3.1. 설치 방법

### 3.2. 사용 방법

- 웹 콘솔이나 직접 설치하여 자바스크립트를 이용해 테스트 가능
- 직접 설치하여 자바스크립트 실행

### 3.3. 결과

- 테스트 결과는 `--out`을 통해 grafana, prometheus, datadog 등으로 export 가능
- 총 생성하여 처리된 request 수와 request에 걸린 시간(http_req_duration)으로 비교 가능


- 다음은 nodejs를 통해 간단한 api 테스트 진행 결과

![img](https://wiki.daoucloud.com/download/attachments/45154715/image2023-4-28_13-54-32.png?version=1&modificationDate=1682657672000&api=v2)

## 4. iperf3

- 서버/클라이언트 간 네트워크 부하 테스트 툴

### 4.1. 설치 방법

### 4.2. 사용 방법

- 옵션


- 서버 실행


- 클라이언트 실행

### 4.3. 결과

- 5G의 데이터를 보내는데 총 걸리는 시간 체크

![img](https://wiki.daoucloud.com/download/attachments/45154715/image2023-4-28_13-54-58.png?version=1&modificationDate=1682657699000&api=v2)

- 서버와 클라이언트 간의 badwidth 측정 및 전송 속도 체크 가능
- 데이터 처리에 걸린 시간과 대역폭을 이용해 비교 가능
- udp 모드 사용 시 packet loss 확인 가능

![img](https://wiki.daoucloud.com/download/attachments/45154715/image2023-4-28_13-55-9.png?version=1&modificationDate=1682657710000&api=v2)

## 5. FIO

- 하드디스크의 read, write IOPS와Bandwidth를 측정할 수 있는 툴

### 5.1. 설치 방법

### 5.2. 사용 방법

- fio 옵션


- 사용 예제 명령어

### 5.3. 결과

![img](https://wiki.daoucloud.com/download/attachments/45154715/image2023-4-28_13-55-24.png?version=1&modificationDate=1682657725000&api=v2)

- 처리 속도이기 때문에 높을수록 좋음

### 5.4. 관련 자료

- <https://portal.nutanix.com/page/documents/kbs/details?targetId=kA07V000000LX7xSAG>

## 6. gzip compression

### 6.1.  설치 방법

### 6.2. 사용 방법

### 6.3. 실행 결과

- linux 4.13 kernel 폴더 두 개를 압축/해제하는데 필요한 시간 측정
- 한 번 실행 시 3번의 gzip 테스트를 진행하여 평균 값 도출
- 다른 CPU와의 결과 차이 확인 가능


- <https://openbenchmarking.org/result/2304286-NE-GZIPTEST258>

![img](https://wiki.daoucloud.com/download/attachments/45154715/image2023-4-28_16-1-9.png?version=1&modificationDate=1682665270000&api=v2)
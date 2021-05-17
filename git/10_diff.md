# git diff 명령어 정리

> git diff 명령어에 대한 정리입니다.

git diff 명령어는 커밋하거나 추가한 파일에 대한 차이점을 알 수 있는 명령어입니다. 커밋한 내용과 현재 내용이나 다른 브랜치와의 변경된 내용을 확인할 수 있습니다.



## diff 사용법
- **워킹 디렉토리와 Staging Area 비교**
	
	- `git diff`
	
	- add한 내용과 현재 수정한 내용의 비교는 간단하게 `git diff`를 통해 확인이 가능하다.
	
	  ![img](https://blogfiles.pstatic.net/MjAyMTA1MTJfMjAx/MDAxNjIwNzg1NDM0NTQ2.Ap1IsOMwIquF5lcKgpJ695B5F6PXyAA7D4E37PUOz_og.ZV3RRTqsN6qUh4cb4h0avy1zL_-QgUQRiePLoccNHxcg.JPEG.jkjk010jkjk/gitdiff1.PNG?type=w1)
- **Staging Area와 Commit한 내용 비교**
	
	- `git diff --staged` or `git diff --cached`
	
	- commit한 내용과 현재 수정해서 add한 파일의 비교는 `--staged`나 `--cached` 옵션을 이용해 비교가 가능하다.
	
	  ![img](https://blogfiles.pstatic.net/MjAyMTA1MTJfNDYg/MDAxNjIwNzg1NDY2Mjk2.Nk2bGem8o09o2fT7wBJgGHBRQdNkJWsibvdyJSazDHkg.P1R7LMawhWvuf0yBoXpqn7hCZhYOX8X7kRND1DMSt1og.JPEG.jkjk010jkjk/gitdiff2.jpg?type=w1)
- **Branch간 비교**
	- `git diff {branch} {다른 branch}`
	
	  ![img](https://blogfiles.pstatic.net/MjAyMTA1MTJfNTMg/MDAxNjIwNzg1NTIwNjgw.oSKpkBL7yi9DFLowPQwZtlJgtBH3dXbhW9E-mJzyOisg.-D3qq2d1AMBvG460ZODNaXIkdBZUL6036zSqRT7NQUwg.JPEG.jkjk010jkjk/gitdiff7.PNG?type=w1)
- **현재 Branch와 다른 Branch 비교**
	- `git diff {다른 branch}`
	
	  ![img](https://blogfiles.pstatic.net/MjAyMTA1MTJfMjk5/MDAxNjIwNzg1Njg3NDIy.t4ZJSGdJo4u_kSRU5QzDYxNG-1SqlQc3pYVa9xEY7hEg.LnWgK2wwIB8Ci5O4TTyVPp3L-_bKxvn_LVoGjIySom8g.JPEG.jkjk010jkjk/gitdiff8.PNG?type=w1)
- **Commit한 내용 비교**
	
	- `git diff {commit hash} {commit hash}`
	
	- commit 넘버를 이용하여 commit한 내용을 비교한다.
	
	  ![img](https://blogfiles.pstatic.net/MjAyMTA1MTJfMTk0/MDAxNjIwNzg1ODU0MDk0.jjTqQlPk_7SxH8DC4O0_4doLrOKUdTLXJJqXxQk7okwg.YbIMwXs8P31bzScWjektYPij2Wi8u2qJmHGt1dvQTe0g.JPEG.jkjk010jkjk/gitdiff5.PNG?type=w1)
- **마지막 Commit과 이전 Commit 비교**
	
	- `git diff HEAD^ HEAD`
	
	  ![img](https://blogfiles.pstatic.net/MjAyMTA1MTJfNzEg/MDAxNjIwNzg1ODcyMjI0.WVZ9o5U1GzRi2eD_TdnGEszbKBJIZLh4eGLmiyHRVJsg.2kKmCNilbfq_iPEf2I4FRYDx2rYtn3LTquNoZhdnX4gg.JPEG.jkjk010jkjk/gitdiff6.PNG?type=w1)
- **마지막 Commit과 현재 수정사항 비교**
	
	- `git diff HEAD`
	
	- untracking 파일을 제외하고 add한 내용과 아닌 내용 모두를 비교한다.
	
	  ![img](https://blogfiles.pstatic.net/MjAyMTA1MTJfMjY2/MDAxNjIwNzg1ODg4MzEz.WJcrvHQ5Q-yPADenBA_2htoBjngCBk-hxFvfoZieHwcg.m6g0sorcbPqtTdSZ3LqD4fnEVkn_v9RI2y39NYK8tHog.JPEG.jkjk010jkjk/gitdiff3.PNG?type=w1)



## git diff 패치 적용

- **git diff를 이용한 패치 파일 생성**
  - `git diff > patchfile`
  - `git diff src/main.c ../../temp/src/main.c > patchfile`
  - `--no-prefix`옵션을 주어 a/b patch prefix가 제거된 패치 파일 생성
    - `git diff --no-prefix > patchfile`
- **생성한 패치 파일 적용**
  - `patch -p1 < patchfile`
  - `--no-prefix` 옵션을 사용하여 생성한 패치 파일인 경우 적용 방법
    - `patch -p0 < patchfile`
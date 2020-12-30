# 06. gitignore

- 우리가 git으로 관리하고 싶지 않은 파일들을 제외시키는 방법

  - 우리가 원하지 않는 파일을 제외
  - 외부에 공개되면 안되는 파일들 (key, 개인정보...)

- `.gitignore`

  - `.gitignore`에 작성된 파일들은 git으로 관리하지 않음. 무시

  ```bash
  .DS_Store # Mac OS에서만 사용하는 파일
  ```

- 작성법

  ```bash
  f.txt # 특정 파일
  secret/ # 특정 폴더, 폴더안의 파일도 다 제외
  *.png # .png로 끝나는 파일들
  !profile.png # 모든 png는 빼고, profile.png는 넣고
  ```

- 처음부터, git으로 관리한 적이 없을때 부터 관리를 해줘야 함

- 그렇다면 이미 commit을 한 파일 제외 방법

  1. `.gitignore`에다가 파일 명시
  2. `git rm -cached [파일명]`
     - git에서 더 이상 관리하지 않음
     - WD에서 삭제해서 더 이상 관리하지 않음

- `.DS_Store` : Mac OS

- `Thumbs.db` : Windows
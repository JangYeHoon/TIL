# 05. 취소하기

1. git add 취소하기

   ```bash
   $ git status
   On branch main
   Changes to be committed:
     (use "git restore --staged <file>..." to unstage)
           new file:   f.txt
   ```

   ```bash
   $ git restore --staged f.txt
   ```

   - 구버전

     ```bash
     $ git reset HEAD f.txt
     ```

   ```bash
   $ git status
   On branch main
   Untracked files:
     (use "git add <file>..." to include in what will be committed)
           f.txt
   
   nothing added to commit but untracked files present (use "git add" to track)
   ```

2. git commit 취소 (되돌아가기)

   ```bash
   $ git reset [돌아갈 commit의 hash]
   ```

   ```bash
   $ git reset bda4343
   ```

   ```bash
   $ git log --oneline
   20cc748 (HEAD -> main) f 추가
   bda4343 (origin/main) Merge branch 'main' of https://github.com/JangYeHoon/TIL-test
   a1476ac e 추가
   3bd0341 d 추가
   90c1cd0 c 추가
   3911f2f b 추가
   d357ae7 a 추가
   
   jyh@DESKTOP-QB2HHTP MINGW64 ~/강의장/TIL-test (main)
   $ git reset bda4343
   
   jyh@DESKTOP-QB2HHTP MINGW64 ~/강의장/TIL-test (main)
   $ git log --oneline
   bda4343 (HEAD -> main, origin/main) Merge branch 'main' of https://github.com/JangYeHoon/TIL-test
   a1476ac e 추가
   3bd0341 d 추가
   90c1cd0 c 추가
   3911f2f b 추가
   d357ae7 a 추가
   ```

   - 옵션

     ```bash
     $ git reset --mixed [hash]
     ```

     - staging 없앰 + **작업한 것은 남아있음**
     - 아무 옵션 안주면 mixed 사용

     ```bash
     $ git reset --soft [hash]
     ```

     - staging 그대로 유지 + **작업한 것 남아있음**

     ```bash
     $ git reset --hard [hash]
     ```

     - **[주의]** 완전히 취소하는 것 (작업한 것도 삭제!)

   - hash

     - ```bash
       git reset bda4343e3b56d1d9a96b769ba3429d753469cb76
       ```

   - HEAD의 상대적 위치

     - HEAD~(숫자)
       - HEAD~1 (`HEAD~`)
       - HEAD~2

     ```bash
     $ git reset HEAD~
     ```

     - commit  되돌리기
       - HEAD~1 (1단계 전으로 되돌리고)
       - `--mixed` : staging은 취소, 작업 내용은 유지

3. WD(Working Directory) 변경 내용 취소(삭제), 원상 복구

   - WD
     - 지금 우리가 작업 중인 공간 
     - Git이 관리하고 있는 공간
     - 기록이 한번이라도 기록된 파일들이 있는 공간

   ```bash
   $ git restore [파일명]
   $ git restore a.txt
   ```

   - 구버전

     ```bash
     $ git checkout -- [파일명]
     ```
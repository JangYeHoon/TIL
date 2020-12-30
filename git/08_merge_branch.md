# 08. Merge Branch

- 병합
  - `git merge {브랜치이름}`

1. Fast-Forward

   ```bash
   (main) $ git switch -c login
   (login) $ touch login txt
   (login) $ touch add, git commit
   (login) $ git switch main
   ```

   - `git merge login` : `login` 브랜치 병합

     ```bash
     (main) $ git merge login
     Updating bebf6f1..4ea1935
     Fast-forward
      login.txt | 0
      1 file changed, 0 insertions(+), 0 deletions(-)
      create mode 100644 login.txt
     ```

     ```bash
     $ git log --oneline --graph --all
     * 4ea1935 (HEAD -> main, login) login 추가
     * bebf6f1 b 추가
     * 3b54907 a 추가
     ```

   - `git branch -d login` : `login` 브랜치 삭제

2. 3-Way Merge

   - 브랜치 생성뒤에 main에서 파일만들고 다른 브랜치에서 파일만들고 브랜치를 병합
   - 이때 파일들은 충돌이 없는 상황

3. Merge Conflict

   - main과 다른 브랜치에서 파일을 만들고 병합
   - 근데 두 파일이 충돌했을 경우

   ```bash
   jyh@DESKTOP-QB2HHTP MINGW64 ~/git-merge (main)
   $ git merge signup
   CONFLICT (add/add): Merge conflict in README.md
   Auto-merging README.md
   Automatic merge failed; fix conflicts and then commit the result.
   
   jyh@DESKTOP-QB2HHTP MINGW64 ~/git-merge (main|MERGING)
   $
   ```

   ```bash
   $ git status
   On branch main
   You have unmerged paths.
     (fix conflicts and run "git commit")
     (use "git merge --abort" to abort the merge)
   
   Unmerged paths:
     (use "git add <file>..." to mark resolution)
           both added:      README.md
   
   no changes added to commit (use "git add" and/or "git commit -a")
   ```

   ```bash
   jyh@DESKTOP-QB2HHTP MINGW64 ~/git-merge (main|MERGING)
   $ git commit -m '합의함'
   [main db9a5e6] 합의함
   
   jyh@DESKTOP-QB2HHTP MINGW64 ~/git-merge (main)
   $
   
   ```

   
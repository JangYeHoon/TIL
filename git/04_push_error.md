# 04. Push Error

## Pull을 빠뜨린 경우

- e를 commit

- 그리고 push

  ```bash
  $ git push origin main
  To https://github.com/JangYeHoon/TIL-test.git
   ! [rejected]        main -> main (fetch first)
  error: failed to push some refs to 'https://github.com/JangYeHoon/TIL-test.git'
  # push 실패
  hint: Updates were rejected because the remote contains work that you do
  # remote와 local에 가지고 있는 작업이 다름
  hint: not have locally. This is usually caused by another repository pushing
  # 다른 Repo로 push하려고 시도한 것이거나
  hint: to the same ref. You may want to first integrate the remote changes
  # push를 하기전에 remote에서 발생한 변경 사항을 먼저 통합(반영)을 해야 할 수 있다.
  hint: (e.g., 'git pull ...') before pushing again.
  hint: See the 'Note about fast-forwards' in 'git push --help' for details.
  ```

- `git pull origin main`

  ```bash
  Merge branch 'main' of https://github.com/JangYeHoon/TIL-test # <- 이미 메세지가 작성되어 있음
  # Please enter a commit message to explain why this merge is necessary,
  # especially if it merges an updated upstream into a topic branch.
  #
  # Lines starting with '#' will be ignored, and an empty message aborts
  # the commit.
  ```

  - vim : 에디터
  - esc -> :wq -> enter

  ```bash
  $ git pull origin main
  remote: Enumerating objects: 3, done.
  remote: Counting objects: 100% (3/3), done.
  remote: Compressing objects: 100% (1/1), done.
  remote: Total 2 (delta 1), reused 2 (delta 1), pack-reused 0
  Unpacking objects: 100% (2/2), 247 bytes | 24.00 KiB/s, done.
  From https://github.com/JangYeHoon/TIL-test
   * branch            main       -> FETCH_HEAD
     90c1cd0..3bd0341  main       -> origin/main
  Merge made by the 'recursive' strategy.
  # Merge를 생성
   d.txt | 0
   1 file changed, 0 insertions(+), 0 deletions(-)
   create mode 100644 d.txt
  ```

- `git log --oneline --graph`

  ```bash
  $ git log --oneline --graph
  *   bda4343 (HEAD -> main) Merge branch 'main' of https://github.com/JangYeHoon/TIL-test
  |\
  | * 3bd0341 (origin/main) d 추가
  * | a1476ac e 추가
  |/
  * 90c1cd0 c 추가
  * 3911f2f b 추가
  * d357ae7 a 추가
  ```

  
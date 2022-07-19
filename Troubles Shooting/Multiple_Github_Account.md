### [ Issue ]
하나의 PC에서 `개인 github 계정` 과 `회사 github 계정` 을 모두 사용하고 싶었다.
따라서 특정 `project` 나 `repository` 에서 원하는 계정을 사용하고자하였다.

### [ Problem Solution Approach ]
- 보통의 경우 git 설치 이후 `global` 하게 본인의 `github 계정` 을 설정한다.
  - ```
    git config --global user.name "이름"
    git config --global user.email "이메일"
    ```
  - 왜냐하면 `git`에 `commit`, `push` 할 때마다 계정 정보를 입력하지 않아도 되기 때문이다.
- 하지만 `global`로 설정한 계정과 `다른 계정으로` `commit`과 `push`를 하고 싶다면, `local` 하게 `github 계정` 을 설정하면 된다.
- 특정 `project` 나 `repository` 에서 `--local` 을 이용하여 `원하는 계정`으로 `commit`, `push` 를 한다.
  - ```
    git config --local user.name "현재 레포에서 사용할 이름"
    git config --local user.email "현재 레포에서 사용할 이메일"
    ```
- `현재 사용하고 있는 git 계정의 설정을 확인` 하고 싶다면 아래 명령어로 확인할 수 있다.
  - ```
    git config --list
    ```
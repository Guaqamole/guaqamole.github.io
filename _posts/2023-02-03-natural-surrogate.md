---
title: 자연키(Natural key) vs 대체키(Surrogate Key)
author: guaqamole
date: 2023-02-03 18:32:00 -0500
categories: [DB, Design]
tags: [DBMS, Design, Schema, TBE]
---

![default_post_image](/common/dbms.jpg){: width="400" height="300" }

<br>

## Primary Key를 꼭 대체키로 설정해야할까?
### Use Case
과거에 주민번호를 PK로 사용하는 것이 일반적이던 시절이 있었다. 주민번호는 모든 사람에게 반드시 부여되며(NOT NULL), 유니크하고(UNIQUE), 절대 변하지 않을 것이므로 PK로 삼기에 적합해보였다.

그런데 문제는 정부 정책이 변경되면서, 법적으로 PK를 저장할 수 없게 된 것이다. 결국 PK와 FK 등을 주민번호로 사용하던 모든 데이터베이스 구성을 변경해주어야 했고, 이러한 변경은 비즈니스 로직에도 영향을 주게 되었다.

PK를 변경하는 것은 단순히 번거로움 뿐만 아니라 데이터베이스의 성능에도 악영향을 줄 수 있다. 특히 MySQL과 같이 PK가 레코드의 저장 위치가 결정하는 키라면 성능에 엄청난 악영향을 주게 된다. MySQL에서는 PK가 레코드의 물리적인 저장 위치를 결정하기 때문에, 인덱스는 PK에 의존한다. 그래야 인덱스를 타고 들어와서 PK를 통해 저장된 위치에서 레코드를 읽어올 수 있기 때문이다.




<br>

#### .git의 초기구성

```bash
HEAD
config
description
/branches
/hooks
/objects
/refs
```

<br>

#### .gitignore 생성

예로, `.env` 파일은 많은 자바스크립트 프로젝트에서 개발자들이 로컬 컴퓨터에 임의의 환경 변수를 설정하기 위한 용도로 사용합니다. 따라서 이 파일은 `.gitignore` 파일에 등록을 해야 보안적으로 안전하고 개발자 간에 불필요한 코드 충돌을 피할 수 있습니다.

`.gitignore` 파일을 생성하고, 위에서 디렉토리를 생성할 때 함께 생성해놓은 `.env` 파일을 등록하겠습니다.

```bash
echo .env > .gitignore

cat .gitignore
.env
```

<br>


### 2. To Github (remote)



#### SSH

~~~bash
ssh-keygen -t rsa -C “avoholo@github.com” # or
ssh-keygen -m PEM -t rsa -C "avoholo@github.com" # PEM 설정은 골라서 사용하자.
cat ~/.ssh/id_rsa.pub
~~~

Settings > SSH and GPG Keys > New SSH key > add

~~~bash
$ vi ~/.ssh/config

Host github.com
    HostName github.com
    User git
    IdentityFile /c/Users/avoholo/.ssh/id_rsa
~~~

local에서 remote 접근시 `private key`를 사용하고, github에는 `public key`(id_rsa.pub)를 등록 해야합니다.

<br>

[공식 가이드](https://docs.github.com/en/authentication/troubleshooting-ssh/error-permission-denied-publickey#always-use-the-git-user)에는 `User` 항목에 `git` 을 쓰라고 되어있는데,  절반 이상의 개인 블로그나 StackOverflow에는 자신의 **github username**을 사용하라고 하네요. 이것 때문에 삽질을 30분 동안 했는데, 인터넷에는 **검증되지 않은 정보가 많은것 같습니다.**

아래 명령어로 **인증 테스트**를 할 수 있습니다.

~~~bash
ssh -vT github.com
~~~

`~/.ssh/config`에 등록 해놨던 설정대로 접속이 된걸 확인 할 수 있습니다.

~~~bash
$ ssh -vT git@github.com
Hi avoholo! You've successfully authenticated, but GitHub does not provide shell access.
debug1: channel 0: free: client-session, nchannels 1
Transferred: sent 3216, received 2704 bytes, in 0.4 seconds
Bytes per second: sent 7578.5, received 6371.9
debug1: Exit status 1
~~~

<br>

#### Remote Url

만약 url이 `https` url로 설정되어 있다면, 인증시 `username`과 `password`를 요구합니다. `ssh url`로 바꿉니다.

~~~bash
git remote set-url git@github.com:avoholo/avoholo.github.io.git
~~~

<br>

#### Status (항상 먼저 체크하자)

~~~ bash
git status

On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
~~~

<br>

#### Staging Area

~~~ bash
git add .
~~~

현재까지 작업중이던 **모든** 파일들을 staging area(무대)에 올립니다.

<br>

#### Commit

```bash
git commit -m "commit message"
```

이 상태가 중요한 것은 나중에 Github에 `push`한 것을 `revert`하고 싶을 때

원래상태로 복구하고 싶은 이 커밋을 찾아서 `push`해야 하기 때문입니다.

"commit message" 부분에는 이 커밋에 대한 보조 설명이라던지 그냥 단순하게 당시 날짜를 적어도 무방합니다.

<br>

#### Remote & Origin (초기 설정에 필요)

~~~bash
git remote add origin git@github.com:avoholo/avoholo.github.io.git
git config user.name "avoholo"
git config user.email "avoholo9@gmail.com"
~~~

만약 local repo에서 remote repo 설정이 안되어 있다면 주소를 명시해서 설정할 수 있습니다.

 ***tip:*** `.git` 을 확인해서 remote repo가 어디로 설정되어있는지 확인하세요.

<br>

#### Branch

~~~bash
git branch -M main
~~~

만약 `master`외에 `main` Branch로 변경하고 싶다면 해당 명령어로 바꿀 수 있습니다. *Default*는 `master` 입니다.

<br>

#### Push

```bash
git push -u origin master
```

`origin`은 처음 git 설정시에 지금 이 **remote** **repository**(원격 저장소)에 붙여준 이름입니다.

단순히 `origin`을 가장 많이 사용하는 것 같지만, github, gitlab 같이 해당 사이트 이름을 붙여줄 수 있습니다.

그러면 한번에 두 가지 사이트에 `origin` 이름만 바꿔서 `push`할 수 있습니다.

<br>

### 3. Remote vs Origin

#### Remote update

remote 환경에서 변경되었던 부분을 up-to date로 업데이트 합니다. 

~~~bash
git remote update && git status
~~~

- `git status -uno` : Tracking 중인 브랜치가 remote 환경과 얼마나 다른지 알려줍니다.
- `git remote -v update` : 어떤 브랜치가 업데이트 되었는지 확인 할 수 있습니다.
- `git git show-branch *master` : 

<br>

### 4. Errors

#### Unsafe Repository

해당 오류를 수정하는 방법은 간단합니다. 이미 로그에도 어떻게 해야할지 나와있습니다.

~~~bash
git config --global --add safe.directory /directory
~~~

~~~bash
git config --global --add safe.directory '%(prefix)///172.*.*.*/Avoholo/Workspace/JeKyll/avoholo.github.io/avoholo_blog'
~~~

<br>
---
title: Git Fundamentals
author: guaqamole
date: 2022-11-09 18:32:00 +0900
categories: [Dev, Git]
tags: [Git, Github, Version Control]
image:
  path: /common/git.png
---
회사에서 Git Kraken을 사용하는데 아무래도 엔지니어다 보니 Git을 활용할 일이 개발팀 보다 상대적으로 적다. 하지만 그렇다고 Git을 덜 알아도 된다는 얘기는 아니니 이번 기회에 확실히 Branch 개념 부터 명령어 까지 기본을 잡고 가자.

<br>

****

## Git Branch
Git은 commit을 할 때마다 파일이 존재하는 순간을 중요하게 생각한다. 만약 파일이 수정되지 않았다면, Git은 빠른 성능을 위해서 파일을 새로 저장하지 않고 이전 상태의 파일에 대한 링크만 저장한다. 이런 경우에는 수정사항을 저장하지 않고, 파일의 스냅샷을 시간순으로 저장한다.

![default_post_image](/230124/1.png)

위와 같이 저장될 때 스냅샷으로 저장되기 때문에 개발자가 중앙 서버에서 새로운 프로젝트를 받으면(git checkout) 해당하는 repository의 특정 시점의 스냅샷을 받게 된다. 일반적으로는 가장 최신 버전을 받는다.

<br>

## Branch
이렇게 개발자들이 각각의 스냅샷을 checkout 받고 개발을 하면, 최종적으로 각자의 변경사항을 반영한 스냅샷을 만드는 과정이 필요하게 된다. 그러기 위해서 특정한 기준이 필요한데, 그럴 때 필요한 것이 branch다.

나뭇가지 혹은 분점을 말하는 branch는 말 그대로, 기준이 되는 큰 줄기가 있고 그 줄기에서 옆으로 나오는 가지와 같은 구조이다. 기준이 되는 큰 줄기를 master branch라고 하고, 각각의 개발자는 master branch에서 checkout하고 자신만의 branch를 만든다. 

<br>

## Feature Branch
각각의 개발자로부터 나온 줄기들을 feature branch라고 한다. feature branch에서 개발자들은 각자 개발을 하다가 개발이 끝나고 commit을 하면, 자신의 feature branch를 master branch로 합치게 된다. 이렇게 feature branch에서 master branch로 합치는 과정을 merge라고 한다:
1. master branch를 checkout.
2. feature branch를 각자 만든다.
3. feature branch에서 각자 개발한다.
4. 개발이 끝나면 commit 한다.
5. feature branch를 master branch와 합친다(merge).

<br>

## Git Head
HEAD는 현재 체크아웃된 브랜치의 가장 최신커밋을 가리킨다. 따라서 branch를 변경하게되면, 변경한branch의 가장 최신commit을 가리키게 된다. 이렇게 될 경우, **보통 HEAD는 브랜치 이름을 가리키도록 표시**된다.

> HEAD는 현재 내가 바라보고있는 commit을 가리킨다.

### 예시
1. main 브랜치에서 "상품정보" 커밋
2. `feature-1` 브랜치를 생성
3. feature 브랜치로 checkout
4. 이후 `Add add_to_cart.py` 커밋

HEAD는 현재 과거 커밋을 바라보고 있으므로, `Add add_to_cart.py` 커밋이 보이지 않는다. 하지만 `git log --all` 을 통해, 전체 커밋을 볼 순 있다.

<br>

## 1. Initialize

### Init

`git init`는 새로운 Git 저장소(repository)를 생성할 때 사용하는 Git 명령어다.

~~~bash
git init
Initialized empty Git repository in /Users/dale/temp/our-project/.git/
~~~

<br>

### .git의 초기구성

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

### .gitignore 생성

예로, `.env` 파일은 많은 자바스크립트 프로젝트에서 개발자들이 로컬 컴퓨터에 임의의 환경 변수를 설정하기 위한 용도로 사용한다. 따라서 이 파일은 `.gitignore` 파일에 등록을 해야 보안적으로 안전하고 개발자 간에 불필요한 코드 충돌을 피할 수 있다.

`.gitignore` 파일을 생성하고, 위에서 디렉토리를 생성할 때 함께 생성해놓은 `.env` 파일을 등록하겠다.

```bash
echo .env > .gitignore

cat .gitignore
.env
```

<br>


## 2. To Github (remote)

### SSH

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

local에서 remote 접근시 `private key`를 사용하고, github에는 `public key`(id_rsa.pub)를 등록 해야한다.

<br>

[공식 가이드](https://docs.github.com/en/authentication/troubleshooting-ssh/error-permission-denied-publickey#always-use-the-git-user)에는 `User` 항목에 `git` 을 쓰라고 되어있는데,  절반 이상의 개인 블로그나 StackOverflow에는 자신의 **github username**을 사용하라고 하네요. 이것 때문에 삽질을 30분 동안 했는데, 인터넷에는 **검증되지 않은 정보가 많은것 같다.**

아래 명령어로 **인증 테스트**를 할 수 있다.

~~~bash
ssh -vT github.com
~~~

`~/.ssh/config`에 등록 해놨던 설정대로 접속이 된걸 확인 할 수 있다.

~~~bash
$ ssh -vT git@github.com
Hi avoholo! You've successfully authenticated, but GitHub does not provide shell access.
debug1: channel 0: free: client-session, nchannels 1
Transferred: sent 3216, received 2704 bytes, in 0.4 seconds
Bytes per second: sent 7578.5, received 6371.9
debug1: Exit status 1
~~~

<br>

### Remote Url

만약 url이 `https` url로 설정되어 있다면, 인증시 `username`과 `password`를 요구한다. `ssh url`로 바꾸자.

~~~bash
git remote set-url origin git@github.com:avoholo/avoholo.github.io.git
~~~

<br>

### Status (항상 먼저 체크하자)

~~~ bash
git status

On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
~~~

<br>

### Staging Area

~~~ bash
git add .
~~~

현재까지 작업중이던 **모든** 파일들을 staging area(무대)에 올린다.

<br>

### Commit

```bash
git commit -m "commit message"
```

이 상태가 중요한 것은 나중에 Github에 `push`한 것을 `revert`하고 싶을 때

원래상태로 복구하고 싶은 이 커밋을 찾아서 `push`해야 하기 때문다.

"commit message" 부분에는 이 커밋에 대한 보조 설명이라던지 그냥 단순하게 당시 날짜를 적어도 무방한다.

<br>

### Remote & Origin (초기 설정에 필요)

~~~bash
git remote add origin git@github.com:avoholo/avoholo.github.io.git
git config user.name "avoholo"
git config user.email "avoholo9@gmail.com"
~~~

만약 local repo에서 remote repo 설정이 안되어 있다면 주소를 명시해서 설정할 수 있다.

 ***tip:*** `.git` 을 확인해서 remote repo가 어디로 설정되어있는지 확인하세요.

<br>

### Branch

~~~bash
git branch -M main
~~~

만약 `master`외에 `main` Branch로 변경하고 싶다면 해당 명령어로 바꿀 수 있다. *Default*는 `master` 다.

<br>

### Push

```bash
git push -u origin master
```

`origin`은 처음 git 설정시에 지금 이 **remote** **repository**(원격 저장소)에 붙여준 이름다.

단순히 `origin`을 가장 많이 사용하는 것 같지만, github, gitlab 같이 해당 사이트 이름을 붙여줄 수 있다.

그러면 한번에 두 가지 사이트에 `origin` 이름만 바꿔서 `push`할 수 있다.

<br>

## 3. Remote vs Origin

### Remote update

remote 환경에서 변경되었던 부분을 up-to date로 업데이트 한다. 

~~~bash
git remote update && git status
~~~

- `git status -uno` : Tracking 중인 브랜치가 remote 환경과 얼마나 다른지 알려준다.
- `git remote -v update` : 어떤 브랜치가 업데이트 되었는지 확인 할 수 있다.
- `git git show-branch *master` : 

<br>

## 4. Errors

### Unsafe Repository

해당 오류를 수정하는 방법은 간단한다. 이미 로그에도 어떻게 해야할지 나와있다.

~~~bash
git config --global --add safe.directory /directory
~~~

~~~bash
git config --global --add safe.directory '%(prefix)///172.*.*.*/Avoholo/Workspace/JeKyll/avoholo.github.io/avoholo_blog'
~~~

<br>
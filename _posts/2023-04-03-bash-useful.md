---
title: 유용한 Bash Commands 모음
author: guaqamole
date: 2023-04-02 18:32:00 -0500
categories: [Infra, Scripting Language]
tags: [Script, Infra, Bash]
---
<br>

![Desktop View](/common/bash.png){: width="300" height="200" }
_Fig 1. The Bourne-Again Shell_

<br>

이번 포스트에선 자주 사용하는 Bash Command들을 정리 해보려 합니다.

<br>

___

<br>

## Bash - ls

### 파일과 디렉터리 구분

`-F` 옵션을 사용하면 디렉터리 이름 뒤에는 `/` 기호가 붙습니다. 또한 실행가능한 파일 뒤에는 `*` 기호가 붙습니다.

```bash
$ ls -F
README.md     dist/         node_modules/ package.json  run.sh*       src/
```

### 시간 순으로 나열

`-t` 옵션을 사용하면 최근에 수정한 파일이 먼저 나오고 예전에 수정한 파일은 나중에 나옵니다

```bash
$ ls -t
run.sh       README.md    package.json node_modules dist         src
```

### 크기 순으로 나열

`-S` 옵션을 사용하면 크기가 큰 파일이 먼저 나오고 크기가 작은 파일이 나중에 나옵니다.

```bash
$ ls -S
node_modules package.json dist         src          README.md    run.sh
```

### 역순으로 나열

`-r` 옵션을 사용하면 알파멧 역순으로 나열됩니다. `-t`나 `-S` 조합해서 사용하면 시간 역순, 크기 역순으로도 정렬할 수 있습니다.

```bash
$ ls -r
src          run.sh       package.json node_modules dist         README.md

$ ls -tr
src          dist         node_modules package.json README.md    run.sh

$ ls -Sr
run.sh       README.md    src          dist         package.json node_modules
```

`-r` 옵션을 사용하지 않고 `ls`만 사용한다면 **알파벳 순으로 정렬합니다.**

### 크기를 쉽게 알아보기

`-hl` 옵션을 사용하시면 파일이나 디렉터리 크기에 단위가 붙어서 읽기 편한 상태로 표시됩니다. (예. 1K 234M 2G)

```bash
$ ll -hl
total 8
-rw-r--r--    1 dale  staff    51B 10 Aug 16:37 README.md
drwxr-xr-x    4 dale  staff   128B  7 Aug 16:45 dist
drwxr-xr-x  507 dale  staff    16K  7 Aug 16:46 node_modules
-rw-r--r--    1 dale  staff   371B  7 Aug 16:46 package.json
-rwxr--r--    1 dale  staff    33B 10 Aug 16:43 run.sh
drwxr-xr-x    3 dale  staff    96B  7 Aug 09:15 src
```

여기서 `-l` 옵션은 상세 정보 확인입니다.

### 패턴 매칭하기

`ls` 커맨드는 Globs 패턴 매칭도 지원합니다.

```bash
$ ls src/*.{ts,tsx}
src/App.test.tsx       src/App.tsx            src/index.tsx          src/react-app-env.d.ts src/reportWebVitals.ts src/setupTests.ts
```

<br>

## Bash - find

<br>

## Bash - sed

<br>

## Bash - Combination

### 크기 순 나열 후 보기 쉽게 정렬

`ls -lhS` or `ls -lS`

~~~bash
$ ls -lhS
total 3.5M
-rw-r--r-- 1 root root 3.3M Oct 25 17:17 libruby.so.2.7.6
-rw-r--r-- 1 root root 219K Oct 25 17:17 libjemalloc.so.1
drwxr-xr-x 2 root root 4.0K Apr  4 14:41 pkgconfig
drwxr-xr-x 6 root root 4.0K Apr  4 14:41 ruby
lrwxrwxrwx 1 root root   16 Oct 25 17:17 libjemalloc.so -> libjemalloc.so.1
lrwxrwxrwx 1 root root   16 Oct 25 17:17 libruby.so -> libruby.so.2.7.6
lrwxrwxrwx 1 root root   16 Oct 25 17:17 libruby.so.2.7 -> libruby.so.2.7.6

$ ls -lS
total 3512
-rw-r--r-- 1 root root 3359656 Oct 25 17:17 libruby.so.2.7.6
-rw-r--r-- 1 root root  224000 Oct 25 17:17 libjemalloc.so.1
drwxr-xr-x 2 root root    4096 Apr  4 14:41 pkgconfig
drwxr-xr-x 6 root root    4096 Apr  4 14:41 ruby
lrwxrwxrwx 1 root root      16 Oct 25 17:17 libjemalloc.so -> libjemalloc.so.1
lrwxrwxrwx 1 root root      16 Oct 25 17:17 libruby.so -> libruby.so.2.7.6
lrwxrwxrwx 1 root root      16 Oct 25 17:17 libruby.so.2.7 -> libruby.so.2.7.6
~~~


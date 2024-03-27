---
title: Git Branch
author: guaqamole
date: 2023-01-24 18:32:00 -0500
categories: [Dev, Git]
tags: [Git, Github, Version Control, TBE]
---

## What is it?

Git은 commit을 할 때마다 파일이 존재하는 순간을 중요하게 생각합니다. 만약 파일이 수정되지 않았다면, Git은 빠른 성능을 위해서 파일을 새로 저장하지 않고 이전 상태의 파일에 대한 링크만 저장합니다. 이런 경우에는 수정사항을 저장하지 않고, 파일의 스냅샷을 시간순으로 저장합니다.

![default_post_image](/230124/1.png)

위와 같이 저장될 때 스냅샷으로 저장되기 때문에 개발자가 중앙 서버에서 새로운 프로젝트를 받으면(git checkout) 해당하는 repository의 특정 시점의 스냅샷을 받게 됩니다. 일반적으로는 가장 최신 버전을 받습니다.

### Branch

이렇게 개발자들이 각각의 스냅샷을 checkout 받고 개발을 하면, ***최종적으로 각자의 변경사항을 반영한 스냅샷을 만드는 과정이 필요하게 됩니다*. 그러기 위해서 특정한 기준이 필요한데, 그럴 때 필요한 것이 branch입니다.**

나뭇가지 혹은 분점을 말하는 branch는 말 그대로, 기준이 되는 큰 줄기가 있고 그 줄기에서 옆으로 나오는 가지와 같은 구조입니다. 기준이 되는 큰 줄기를 master branch라고 하고, 각각의 개발자는 master branch에서 checkout하고 자신만의 branch를 만듭니다. 

### Feature Branch

각각의 개발자로부터 나온 줄기들을 feature branch라고 합니다. feature branch에서 개발자들은 각자 개발을 하다가 개발이 끝나고 commit을 하면, 자신의 feature branch를 master branch로 합치게 됩니다. 이렇게 feature branch에서 master branch로 합치는 과정을 merge라고 합니다.

![default_post_image](/230124/2.png)

1. master branch를 checkout 합니다.
2. feature branch를 각자 만듭니다.
3. feature branch에서 각자 개발합니다.
4. 개발이 끝나면 commit 합니다.
5. feature branch를 master branch와 합칩니다(merge).



# Git Head

HEAD는 현재 체크아웃된 브랜치의 가장 최신커밋을 가리킵니다. 따라서 branch를 변경하게되면, 변경한branch의 가장 최신commit을 가리키게 됩니다. 이렇게 될 경우, **보통 HEAD는 브랜치 이름을 가리키도록 표시**됩니다.

> #### HEAD는 현재 내가 바라보고있는 commit을 가리킨다.

### 예시

1. main 브랜치에서 "상품정보" 커밋
2. elice 브랜치를 생성
3. elice브랜치로 checkout
4. 이후 "Add m2.js" 커밋

HEAD는 현재 과거 커밋을 바라보고 있으므로, "Add m2.js" 커밋이 보이지 않습니다. 하지만 `git log --all` 을 통해, 전체 커밋을 볼 순 있습니다.
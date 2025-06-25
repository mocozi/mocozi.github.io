---
title: 자원 봉사에 대한 이야기
excerpt: 자원 봉사에서 배려는 어디까지 인가?
tags:
  - 일상
categories:
  - blog
toc: true
toc_sticky: true
date: 
last_modified_at: 
comments: true
author_profile: 
sidebar:
---
## 



덕분에 Obsidian에서 노트가 Github에 자동으로 동기화되는 것이 가능해졌다.

이제 동기화된 노트 중에 특정 폴더가 동기화되면, 동기화된 특정 폴더의 파일을 Github blog의 _posts 폴더로 자동으로 옮기고 싶다. 
1. 특정 폴더의 이름은 "6. blog"이며, 이 폴더가 갱신이 되면 https://github.com/mocozi/mocozi.github.io의 _posts 폴더와 자동으로 동기화가 이루어져야 한다.
2. Github Action을 이용해서 자동화하는 방법을 알려달라.

Github Action을 처음 사용해 보기 때문에 쉽고 자세히 설명해 달라.


테스트를 했는데 작동하지 않아. 예상되는 원인은 아래와 같은데 검토해 줘.
1. YAML 코드의 내용에 마지막 부분이 fi로 끝나고 있는데 맞는 코드인지 확인해 줘.
2. YAML 코드의 내용에 on.push.branches와 on.push.paths가 존재하지 않아.




>[!caution]
>반박시 당신의 말이 옳다.

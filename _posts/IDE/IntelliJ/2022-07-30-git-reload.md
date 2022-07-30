---
layout: single
title: "[IntelliJ] 깃허브 로컬데이터와 인텔리제이 탐색기 데이터가 다른 경우"
categories: github_blog
typora-root-url: ..\..\images\
toc: true
---



# [IntelliJ] 깃허브 로컬데이터와 인텔리제이 탐색기 데이터가 다른 경우



인텔리제이에 깃허브를 연동해서 사용하고 있다.

- 단축키
  - Ctrl + K : 커밋
  - Ctrl + Shift + K : 푸시

잘 사용중이었는데, 어쩐 일인지 fork한 remote Repository와 IntelliJ 탐색기 데이터가 달랐다.



## 문제상황 1: 인텔리제이 브랜치 체크아웃 시점

![IntelliJ-branch](..\..\images\IntelliJ-branch.PNG)

나는 이 상황은 아니었지만, 이런 상황도 가능성이 있을 것 같아서 작성한다.



## 문제상황 2: 로드 오류

일단 상황이 발생하고 나고, 가장 먼저 하고싶었던 작업은 '프로젝트 새로고침' 혹은 '프로젝트 다시 열기'였다. 탐색기에서 새로고침 하고 싶은 디렉토리를 우클릭한 후 'Reload from disk'를 작동시킨다.

![reload-from-disk](..\..\images\reload-from-disk.jpg)

다시 잘 보인다. 인텔리제이는 간혹 이런 경우가 있다고 한다.



별거 아니지만, 미래에 또 이러한 문제를 겪을 나를 위해.

------

> 마지막 수정일시: 2022-07-30 15:33
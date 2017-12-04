---
title: Git 기본 명령어 정리
date: 2017-12-01 14:56:48
---

Git은 분산 버전 관리 시스템으로, 리누스 토르발스가 리눅스 커널을 관리하기 위해 개발하였다. Git을 만드는 2주간의 기간이 제일 행복했다는 후문이 있다... ~~레알 Geek스럽다.~~ 이러한 Git을 사용하는 프로젝트를 지원하는 웹호스팅 서비스 중 Github ~~이곳은 코더들의 성지~~와 관련해서 포스팅을 하고자 한다.
 
### 1. 일단 Git 설치를 하자.

아래 주소에 접속해서 OS에 맞는 Git을 설치한다.
[Git 설치](https://git-scm.com/downloads)

### 2. Github에 repository를 만든다.

아래 주소에 접속해서 가입을 한다.
[Git 사이트](https://github.com/)

![새로운 Repsitories 만들기 1](/uploads/GitNewBtn.jpg) 
Repsitories 탭에서 위와 같이 생긴 New버튼을 클릭한다. 
![새로운 Repsitories 만들기 2](/uploads/GitCreateRepo.jpg)
New 버튼을 클릭하면 위와 같은 창이 보이게 된다. Repository name 부분에 Repo 이름을 등록하고, Description 부분과 README 부분은 만들어질 repo에 설명을 다는 부분이다. ~~나중에 적어도 되므로 안 적어도 그만이다.~~ Public은 전체 공개 프로젝트이고, Private은 비공개 프로젝트이다. ~~쉽게 말해 무료, 유료~~ 적절하게 작성 후 Create repository를 클릭하면 새로운 repository가 생성된다.

### 3.Github와 local repository 연동하기.

Github 페이지와 내 컴퓨터에 저장된 폴더를 연동하기 전에 Git 시스템에 대해 간단하게 정리를 하고자 한다.


---
layout: post
title: "[Trouble Shooting] Jekyll chirpy 템플릿 최신버전 사용하기 (NodeJS + Ruby)"
date: 2023-10-22
categories:
  - Trouble Shooting
  - Jekyll
tags:
  [
    blog,
    jekyll,
    chirpy,
    jekyll theme,
    NexT theme,
    지킬 테마,
    지킬 블로그 포스팅,
    GitHub Pages,
  ]
---

## Github 블로그 with Chirpy Jekyll Theme

github 블로그를 시작할 때 Jekyll을 많이 사용한다.
그럼 Jekyll이 무엇일까?

### Jekyll?

- 텍스트 변환 엔진으로, Markup 언어로 글을 작성하면 이를 통해 웹사이트를 만들어줌.
- 서버 소프트웨어가 필요 없어 매우 빠르고 가볍다.

-> Jekyll을 사용한 템플릿을 땡겨와서 블로그를 시작하면 기본 틀을 잡기 매우 간편하다.

### Chirpy

- Jekyll을 이용한 여러 템플릿을 제공해준다.
- Git : https://github.com/cotes2020/jekyll-theme-chirpy
- Tutorial : https://chirpy.cotes.info/categories/tutorial/

### 블로그 동작 방식
로컬 : 누군가 잘 만들어 놓은 블로그 테마 소스코드를 Bundle을 이용하여 빌드하고, 웹서버를 구동하는 것 같다.
Github : 소스 코드 Push -> Github action(빌드, 배포 명령어 수행) -> Git Page 배포
* nodejs를 이용하여 웹페이지를 개발하고 Github Action 경험이 있으면 누구나 쉽게 이해할 것 같다. 하지만, 경험이 없으면 이해하기 다소 어려움

### 개요 (정적에서 동적으로)
기존 깃헙 블로그 테마들에 대해서는 참고할만한 레퍼런스나 가이드가 많다. 하지만 해당 자료들은 단순 bundle을 이용해서 빌드하고 배포하는 방식의 테마들 설명밖에 없었다.
 * 정적 페이지 예시 :화이트 모드와 다크 모드를 설정하려면 다시 빌드해서 배포해야 함. 

나는 이왕 내 블로그를 개설하는 데 정적보단 동적인게 더 이쁘기 때문에 자료를 찾던 중 Chirpy 테마를 찾을 수 있었다. 해당 테마는 정적 웹소스 뿐만 아니라 상태 관리를 통해 동적인 페이지를 구성하고 있었다.


### (로컬) 빌드 및 배포
해당 소스코드를 clone하고 타 블로그에 나와있는 방식으로 bundler를 이용하여 빌드하고 구동하였다. 그러나, 왼쪽 하단의 다크모드와 화이트모드를 변환하는 토글이 먹히지 않았다. 로그를 살펴보니 해당 js파일이 없다는 로그들이 계속 찍혀있었다. 토글을 변경하는 js 코드가 있었음에도 불구하고 해당 파일을 찾지 못하는 건 빌드가 되지 않았다는 뜻이다.

그래서 나는 기존에 설치되어있던 npm을 이용하여 npm install && npm run build 명령어를 수행해주고 다시 bundle을 이용해 서버를 구동시켰더니 배경이 잘 변경되는 것을 확인할 수 있었다.

* 해당 소스코드의 github action script 중 ci.yml을 확인하니 npm build가 포함되어 있었음. (잡담) 소스코드에 빌드/배포 스크립트가 공존해 어떻게 수행되어야 하는지 알려주는 가이드를 해주는 점도 github action의 장점 같다.

### (깃허브) 배포
이제, 로컬에서 동적 페이지가 잘 배포되는 것을 확인했으니 깃허브에 올려서 나만의 블로그를 가질 차례다!
* 하지만... 커밋을 올리자마자 ci를 수행하는 도중 에러가 발생. (img 태그에 alt 설명을 안 달아서 에러 발생...)
CI(통합)도 잘 됐고, Deploy도 잘되었다고 한다. 하지만 banggeunho.github.io에 접속했더니 이상한 흰 배경에 텍스트 한 줄이 달랑 나온다... 뭔가 잘못됐다..

### 배포가 안되네;;
Chripy 레포지토리의 master branch를 clone한 경우 소스코드를 푸시하면 ci.yml과 cd.yml이 수행되게 된다. 당연히 해당 스크립트가 있으니 소스코드를 올리기만 하면 제대로 배포될 것이라 생각했다. 하지만 분석해보자...
* ci.yml -> 소스코드 클론 -> npm ruby 등 설치 -> 빌드 -> 테스트 -> 끝
* cd.yml -> github page에 post 요청을 날린다 -> 끝

분석해보니 두 파일의 연관성이 없었고, ci에서 빌드한 결과물을 cd에서 전혀 사용하고 있지 않았다. 배포가 제대로 안되는게 당연했음.
그러던 중, .github 폴더를 뒤지다가 pages-deploy.yml.hook 이라는 파일이 있었음. 해당 파일은 정적 페이지 버전에서 사용하던 스크립트 같았다. (node를 설치하는 부분이 없었기 때문)

### 해결 방법
1. 먼저 해당 pages-deploy.yml.hook의 파일명을 github action이 인식할 수 있도록 hook 확장자를 제거해준다.
2. 빌드 과정에서 빠진 node 설치와 의존성 설치, npm 빌드 등 스크립트를 추가해준다. (ci.yml에서 필요한 부분만 가져오면 된다.)
3. 끝

앞선 과정을 거쳤고 코드를 다시 push 하였고, pages-deploy workflow가 성공적으로 잘 수행됐고, 접속을 해보니 로컬에서 보았던 페이지와 똑같은 것을 확인할 수 있다.



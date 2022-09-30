---
layout: post
title: '[DevOps] AWS EC2에 Spring Boot 프로젝트 로컬에서 배포하기'
subtitle: 'AWS, EC2, Spring Boot, CyberDuck, Git Bash'
date: 2022-09-05 12:00:00 +0900
categories: 'DevOps'
background: '/img/posts/etc/git.png'
published: false
---

- 시작하기 전 준비물
> - AWS 회원가입(첫 가입이라면 1년 프리티어 사용 가능)
> - Spring Boot 프로젝트
> > - maven / gradle 빌드툴을 이용해 만든 jar 파일
> - CyberDuck
> > - [다운로드!](https://cyberduck.softonic.kr/)
> > - 서버에 파일을 업로드하거나 다운로드해주는 FTP 작업을 쉽게 처리해준다.
> - Git Bash
> > - 실습은 Git Bash에서 진행됩니다.

<br>

## AWS 설정


1 . EC2 - 인스턴스 시작

- EC2(Amazon Elastic Compute Cloud) : 아마존의 클라우드 가상 컴퓨터를 임대 받아 컴퓨터 어플리케이션을 실행할 수 있게 해준다. 

2 . 인스턴스 설정



Reference:

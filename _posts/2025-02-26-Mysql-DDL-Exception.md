---
layout: post
title:  "MySQL DDL Exception"
author: 악어새62
categories: [ TIL, Backend ]
image: assets/images/1.jpg
tags: []
---
## 개요

사이드프로젝트를 진행하던 중 하나의 테이블이 생성되지 않는 문제가 생겼다.

## 원인

얼핏 살펴봤을 때 에러메시지가 나오지 않아서 당황했지만 더 자세히 읽어보니 desc 부분에서 문법 에러가 났다고 알려주었다.  
컬럼명을 바꿔주었을 때 테이블이 정상적으로 생성되는 것을 보아 칼럼 이름으로 Mysql의 명령어(예약어)인 DESC를 사용한 것이 문제였다.  

칼럼 이름을 변경하여 해결
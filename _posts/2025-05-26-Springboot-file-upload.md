---
layout: post
title:  "Springboot file upload"
author: 악어새62
categories: [ TIL, WEB, Backend ]
tags: [ ]
image: assets/images/5.jpg
---
## 개요

파일을 다운받아 처리한 뒤 업로드하는 웹 서버를 작성하다가 S3 관련 의존성이 여러가지인 걸 보고 정리해본다.

## 종류

실제 코드에서 구분할 수 있는 종류에는 AmazonS3, S3Client, S3Operations, TransferManager가 있다.
이것들은 하나의 의존성에 담겨있지 않고 각각 메인으로 사용하는 클래스마다 별도의 의존성을 사용해야한다.  
이유가 뭘까?  

이는 AWS SDK 버전 차이, 기능 추상화 수준의 차이 때문이다.  

예를 들어 S3Operations는 스프링과 AWS의 결합으로 Spring Cloud AWS에서 제공하는 인터페이스다.  
위에서 살펴본 클래스들 중에서 가장 추상화되어있어서 버킷 이름, 키, 파일과 같은 정보들만 있으면 메소드를 호출하여 바로 업로드하고 삭제하고 다운로드 할 수 있다.  
내부적으로는 AmazonS3(AWS SDK v1)을 사용하고 AmazonS3Client가 기본 구현체다.

**참고**  
SDK는 소프트웨어 개발 키트의 약자로 특정 플랫폼이나 서비스용 애플리케이션을 쉽게 만들 수 있도록 도와주는 도구 모음이다. 예를 들어 AWS SDK는 AWS의 서비스를 자바에서 사용할 수 있도록 도와주는 킷이라고 보면 된다.  

AmazonS3는 AWS SDK for Java v1에 해당한다.

S3Client는 AWS SDK for Java v2로 비동기 방식과 모듈화를 지원한다.  

마지막으로 TransferManager는 대용량 파일 업로드 전용 도구다.

따라서 이러한 차이점을 확인하고 현재 목적에 맞는 구현체를 선택하면 된다.  

예를 들어 NCP는 별도의 의존성이 있지 않고 AWS와 동일한 의존성을 사용한다. 그러나 Spring Cloud AWS는 NCP에 맞는 설정을 하기에 부족하다. 따라서 조금 덜 추상화된 구현체를 사용하여 직접 설정해주어야하니 S3Client나 AmazonS3Client를 사용하는 것이 좋다.
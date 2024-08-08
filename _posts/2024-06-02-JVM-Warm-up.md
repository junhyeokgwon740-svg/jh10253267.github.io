---
layout: post
title:  "JVM 웜업"
author: 악어새62
categories: [ TIL, WEB, Backend ]
image: assets/images/3.jpg
tags: [Java, Spring, Backend]
---
## 개요

프로젝트를 하며 성능개선의 목적으로 데이터베이스의 인덱스를 걸고 안걸고의 응답속도 차이를 보려고 했다. 그러다가 이상한 점을 발견했는데 응답속도가 점점 빨라지는 것이였다.  
왜 이런 일이 발생하는지 알아보자

## 자바의 특성

![image](https://github.com/user-attachments/assets/9f25f569-511f-4aa1-9085-ecfa72f5e5b8)
작성된 자바 코드(.java)는 1차적으로 byte code로 컴파일된다.(.class)  
이를 그대로 사용하지 않고 묶어서 배포하는 것이 일반적인데 이를 위해서 자바는 아카이빙 작업을 수행한다.
.jar, .war와 같은 형식으로 여러 클래스 파일과 리소스 파일을 하나의 파일로 묶어준다. .jar파일은 메인 클래스 정보를 포함하고 있고(MANIFEST.MF 파일에 기록) `java -jar app.jar`와 같이 실행할 수 있다.
war파일은 웹 어플리케이션을 배포할 때 사용되는 아카이브 형식으로 클래스 파일 뿐만 아니라 jsp, html, web.xml등을 포함한다.

아무튼 이 파일을 실행하면 JVM은 이를 기계언어로 번역하여 CPU에서 처리하는 과정을 거친다.  
이렇게 자바는 컴파일과 인터프리트 스텝을 거쳐서 실행되는 언어이다.

그러다보니 바로 기계어로 변환되는 컴파일 언어에 비해 일반적으로 성능이 낮다.  
컴파일 언어는 컴파일 과정을 거치며 코드최적화도 수행하기 때문에 일반적으로 인터프리터 언어보다 좋은 성능을 기대할 수 있다.

그러나 이는 컴파일 언어의 장점이자 단점이라고 볼 수 있는다 왜냐하면 이렇게 컴파일된 언어는 빌드된 해당 CPU 아키텍쳐에 종속적이기 때문에 만약 다른 CPU 아키텍쳐에서 실행하고 싶다면 다시 빌드를 거쳐야한다.

이러한 성능 차이를 보완하기 위해 JIT Compiler를 도입했다.

JIT 컴파일러는 바이트 코드를 기계어로 변환시킬 때 기계어를 캐싱하여 재사용하게 된다. 이를 통해 반복되는 기계어 변환 과정을 줄일 수 있고 런타임 환경에 맞추어 최적화를 시켜 성능 향상을 이룰 수 있다.

즉, 런타임 시점에 바이트 코드를 기계어로 동적으로 번역하는 것이다.

JIT 컴파일러는 애플리케이션에서 자주 실행된다고 판단되는 부분만 기계어로 컴파일한다. 이 부분을 핫스팟이라고 한다. 또한 JIT는 실행중인 애플리케이션의 동작을 분석하고 코드 실행 횟수, 메소드 호출 등의 정보를 측정하고 기록한다. 이를 프로파일링이라고 
한다.  
JIT 컴파일러는 프로파일링 결과를 토대로 핫스팟을 식별한다. 그런 뒤 메소드 단위로 바이트코드를 기계어로 번역한다.

이렇게 번역된 기계어를 코드 캐시라는 공간에 저장한다. 저장해놓으면 핫스팟으로 판단된 코드는 다시 컴파일하지 않고 캐싱된 정보를 사용할 수 있다.

그러나 이러한 대안은 자바 성능 향상에는 도움이 되지만 애플리케이션이 처음 실행될 시점에서는 캐싱되어있는 정보가 없다보니 성능 이슈가 발생할 수 있다.

다른 이유가 하나 더 있는데 바로 클러스로더다.

JVM에서 자바의 클래스를 읽어오기 위해 클래스 로더가 사용된다. 이는 클래스 파일을 찾고 메모리에 로드해 실행 가능한 상태로 만드는 역할을 한다.

1. Class Loading: 클래스 파일을 가져와 JVM 메모리에 적재한다.
2. Class Linking: 클래스가 참조하는 다른 클래스, 메소드, 필드등을 확인하고 필요하면 메모리 상에 연결한다.
3. Class Initialization: 클래스 변수를 초기화하거나 static 블록 내의 코드를 실행하는 등 초기화 작업을 한다.

문제는 클래스 로더가 지연 로딩 방식으로 동작한다는 것이다.  
클래스 로더는 최초로 클래스가 필요해진 시점에 클래스를 로드한다.  
애플리케이션이 배포된 직후에는 대부분의 클래스들이 한번도 사용되지 않은 상태로 클래스 로더가 클래스를 매모리에 적재하지 않은 상태이다. 이런 상황에서 요청이 들어오면 그때서야 클래스 로더가 클래스를 메모리에 적재하게 된다.

따라서 애플리케이션이 배포된 직후에 다음의 영향으로 
1. JIT 컴파일러
2. 클래스 로더
응답의 지연이 발생하는 것이다.

## 해결 방법

해결 방법은 의외로 간단하다. 캐싱된 정보가 없어서, 메모리에 적재되지 않아서 생긴 문제이니 배포된 직후에 웜업을 수행하면 된다.

1. CommandLineRunner : 스프링 부트 1.0에 추가된 함수형 인터페이스로 스프링 애플리케이션이 구동된 후 실행되어야 하는 빈을 정의한다.
2. ApplicationRunner : 스프링 부트 2.0에 추가된 인터페이스로 스프링 애플리케이션이 구동된 후에 실행되어야 하는 빈을 정의할 수 있다.

```java
@Component
public class WarmupCommandLineRunner implements CommandLineRunner{

  private final ClubController clubController;

  public WarmupCommandLineRunner(final ClubController clubController) {
      this.clubController = clubController;
  }

  @Override
  public void run(String... args) throws Exception {
      clubController.getAllClub();
  }
}
```
```java
@Component
public class WarmupRunner implements ApplicationRunner {
  private final ClubController clubController;

  public WarmupRunner(final ClubController clubController) {
      this.clubController = clubController;
  }

  @Override
  public void run(final ApplicationArguments args) throws Exception {
      try {
          clubController.getAllClub();
      } catch (Exception e) {
          ...
      }
  }
}
```
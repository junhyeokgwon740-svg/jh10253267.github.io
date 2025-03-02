---
layout: post
title:  "Mapstruct"
author: 악어새62
categories: [ TIL, Backend ]
image: assets/images/1.jpg
tags: []
---
## 개요

프로젝트를 하며 객체 변환을 할 때 애매한 경우가 생겼다.  
객체 책임을 신경쓰자니 의존성이 생기고 고민하다가 잘못된 결론을 내기도 했다.
고민해봤을 때 별도의 클래스 계층을 사용하는 것도 좋은 방법이 될 수 있을 것 같았다.
이번 기회에 알아보자.

## ModelMapper

일단 관련 기능을 할 수 있는 의존성중에는 ModelMapper가 있다. 모델 매퍼는 DTO와 엔티티 사이의 상호 변환을 도와주는 의존성이다. 사용법도 굉장히 단순하다.  
그러나 ModelMapper에는 문제가 있는데 다음과 같다.
1. 성능 저하 문제

모델 매퍼는 내부적으로 리플렉션을 사용한다. 런타임에 클래스 정보를 탐색하고 매핑을 수행하기 때문에 느리고 리소스를 많이 사용할 수 있다. 따라서 많은 요청이 왔을 때 병목현상이 발견되었다고 한다.

2. 디버깅 어려움

자동으로 매핑을 수행하기 때문에 오류가 발생할 때 원인을 파악하기 어려울 수 있다.


## MapStruct

맵스트럭트는 자바에서 객체 간 매핑을 컴파일 시점에 자동으로 생성해주는 라이브러리다.  
두 가지 의존성이 필요하다.  
맵스트럭트 코어와 프로세서가 필요하다.  
```gradle
implementation 'org.mapstruct:mapstruct:1.6.3'
annotationProcessor 'org.mapstruct:mapstruct-processor:1.6.3'
```

annotationProcessor란 컴파일 타임에 애노테이션을 분석하고 특정한 코드를 자동 생성하는 기능을 수행하는 도구로 주로 애노테이션 기반의 코드 생성 라이브러리에서 사용된다. 롬복도 보면...
```gradle
compileOnly 'org.projectlombok:lombok'
annotationProcessor 'org.projectlombok:lombok'
```
이렇게 되어있다.  
컴파일 시점에 실행되므로 성능이 뛰어나고 오류를 미리 감지할 수 있다.

기본 사용 방법
1. Mapper인터페이스를 만든다.
```java
@Mapper
public interface RoomMapper {
	TestMapper INSTANCE = Mappers.getMapper(TestMapper.class);
	Test toTest(TestDTO testDTO);
}
```
이렇게 인터페이스에 @Mapper 애노테이션을 붙이면 맵스트럭트가 자동으로 Impl가 붙은 구현체를 생성해준다.

```java
TestMapper INSTANCE = Mappers.getMapper(TestMapper.class);
```
이 부분은 매퍼 클래스에서 해당하는 매퍼를 찾을 수 있도록 하여 매퍼에 접근할 수 있게 하는 것이다.  
필드값이 동일한 경우 별도의 설정 없이 메소드를 정의할 수 있다,
```java
Test toTest(TestDTO testdto);
```

이렇게 정의하고 빌드(실행)한 뒤 빌드 패키지를 보면 다음과 같은 구현체가 생성된 것을 확인할 수 있다.
```java
public class TestMapperImpl implements TestMapper {
    public TestMapperImpl() {
    }

    public Test toTest(TestDTO testDTO) {
        if (testDTO == null) {
            return null;
        } else {
            Test.RoomBuilder test = test.builder();
            test.id(testDTO.getId());
            test.name(testDTO.getName());
            return test.build();
        }
    }
}
```
종종 두 개의 엔티티를 합쳐서 하나의 DTO로 구성하거나 반대의 상황도 있을 수 있다. 혹은 상호변환하는 객체의 필드명이 다를 수도 있다. 이런 경우 모델매퍼가 알 수 없으니 다음과 같이 알려주어야한다.
```java
@Mapping(source="testDTO.id", target="testId")
@Mapping(source="testDTO2.name", target="testName")
Test toTest(TestDTO testDTO, TestDTO2 testDTO2);
```

일반적인 자바 프로그램이라면 이렇게 하면 되고 만약 스프링과 같은 프레임워크를 사용한다면 이를 주입받아서 사용하게된다.  
그럴 땐 그런 이름의 빈을 찾을 수 없다는 에러가 발생한다. 이럴 땐 
```java
@Mapper(componentModel = "spring")
```
위와 같이 설정해서 빈으로 인식될 수 있도록 해야한다.
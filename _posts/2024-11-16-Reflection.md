---
layout: post
title:  "Reflection API"
author: 악어새62
categories: [ TIL ]
tags: [ ]
image: assets/images/5.jpg
---
## Reflection API

자바에선 타입이라는 개념을 사용한다.  
예를 들어 Apple apple = new Apple()과 같은 식이다.  
또한 자바에선 상위 타입의 참조로 하위 타입을 사용할 수 있는데 다음과 같다.

Fruit apple = new Apple();

이런 구조는 Apple 클래스가 Fruit를 상속받고 있거나 구현하고 있을 때 가능하다.
그렇다면 모든 자바 클래스의 부모 요소인 Object를 타입으로 선언하고 사용할 수 있을까?
다음과 같이 말이다.
```java
Object apple = new Apple();  
apple.doSomething();
```
이와 관련된 설명을 하자면  

자바에서 객체를 생성할 때 가상 머신은 코드를 읽고 필요한 만큼의 공간을 힙에 할당하고 힙에 있는 공간을 가르키는 4바이트 포인터를 만든다.
그리고 상속받는 클래스의 공간을 함께 할당한다.  
예를 들어 Student가 People클래스를 상속받고 있다면 Student와 People에 들어있는 모든 변수도 함께 힙에 들어간다.
따라서 People s = new Student();와 같이 선언한다면 s가 가리키는 힙에는 Student뿐만 아니라 People가 가진 정보들도 같이 들어간다.

그러나 People로 Student에 정의되어있는 메소드를 실행시키는 것은 불가능하다.  
왜냐하면 같은 타입은 호환해서 사용할 수 있는데 예를 들어
```java
People p = new People();
People s = new Student();
```
이렇게 정의해서 People에서 일관된 방식으로 접근할 수 없기 때문이다.

그렇다면 반대로 하면 어떨까? 
Student s = new People();와 같은 식이다.  
s가 가리키는 힙에는 People과 그 상위 클래스에 대한 정보만 알 수 있다. 따라서 이런 식으로 사용할 수 없다.

마찬가지로 Object o = new Student();는 실행은 되지만 실제 동작은 불가능하다.

그러나 Replection API를 이용하면 가능하다.

위와 같이 선언된 객체는 컴파일 타임에 Object 타입으로 취급되어 Student에 정의된 고유 메서드에는 접근할 수 없다. 그러나 Reflection을 사용하면 런타임 시점에 해당 객체의 실제 타입을 확인하고 해당 클래스에 정의된 메서드나 필드에 동적으로 접근할 수 있다.

한번 사용해보자.
```java
public class Student {
    private String name;

    public Student(String name) {
        this.name = name;
    }

    public void doSomething() {
        System.out.println(name + " is doing something!");
    }
}
```
```java
import java.lang.reflect.Method;

public class ReflectionExample {
    public static void main(String[] args) {
        try {
            // Step 1: Object 타입으로 객체 생성
            Object o = new Student("John");

            // Step 2: 실제 객체의 Class 정보를 가져옴
            Class<?> clazz = o.getClass();  // Student 클래스의 Class 객체를 얻음

            // Step 3: 메서드 정보 가져오기
            Method doSomethingMethod = clazz.getDeclaredMethod("doSomething");

            // Step 4: 메서드 실행
            doSomethingMethod.invoke(o);  // "John is doing something!" 출력

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

1. o.getClass()를 통해 실제 객체의 클래스 정보를 가져온다.: o가 실제로 참조하는 객체가 무엇인지 알아낸다.
2. getDeclaredMethod로 특정 메서드를 조회한다.: Class객체를 통해 해당 클래스의 doSomething메서드를 가져온다.
3. invoke(o)를 통해 메서드를 호출한다.: Method 객체의 invoke 메서드를 사용해서 런타임에 동적으로 doSomething 메서드를 실행한다.
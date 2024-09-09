---
layout: post
title:  "Stream API"
author: 악어새62
categories: [ TIL, WEB, Backend ]
tags: [ Java ]
image: assets/images/5.jpg
---
## 개요

자바를 책으로 배웠다면 for, if문의 코드 블럭에 익숙할 것이다. 그러나 강의를 들어보거나 자바를 메인으로 다루는 것이 아닌 책을 보면 for, if문 만큼이나 `stream()`키워드를 사용하여 처리하는 코드를 많이 보게 된다. 이번 기회에 공부해보려한다.

## 함수형 프로그래밍

> "함수형 프로그래밍은 자료 처리를 수학적 함수의 계산으로 취급하고 상태와 가변 데이터를 멀리하는 프로그래밍 패러다임의 하나이다." - 위키백과

다음과 같은 코드를 예를 들어보자
```java
int total = 0;

public void addToTotal(int number) {
  total += number;
}
```
`addToTotal`함수는 외부 상태인 total을 변경한다.  
```java
public int add(int a, int b) {
  return a + b;
}
```
반면 `add`함수는 외부 상태에 영향을 미치지 않고 입력값에 따라 결과를 반환하는 순수 함수다.
이 함수는 프로그램의 외부 상태를 변경하지 않고 단순히 입력값을 받아 처리한 후 결과를 반환한다.

함수형 프로그래밍은 여러 주요 개념들이 있다.

1. 순수 함수
2. 부작용 없음
3. 고차함수
4. 불변성
5. 일급 함수
6. 함수 합성

위에서 본 예제가 바로 순수 함수와 부작용 함수의 차이다. 

## Stream API 사용법

Stream API는 자바가 계속 발전하고 새롭게 등장하고있는 프로그래밍 언어와 개발 방식에 대응하기 위해 만든 스펙이 아닐까 생각한다.  
이외에도 인터페이스 디폴트 메소드라던지 `var`타입 변수라던지 여러가지가 있다.

Stream API는 특별히 Collecitons패키지의 자료구조들을 일관된 방식으로 다루기 위한 API로 
사용법은 이렇다.
```java
List<Integer> integerList = new ArrayList<>();

integerList.stream()...;
```
이렇게 자료구조 뒤에 stream()메소드를 사용하는 것이다.

예를 들어 리스트의 모든 자료를 출력하는 코드를 작성한다면 반복문을 이용하여 다음과 같이 작성할 수 있다.

```java
for(int i = 0; i < integerList.size(); i++) {
  System.out.println(integerList.get[i]);
}
```
Stream을 이용한다면...
```java
integerList.stream().forEach(item  -> {
  System.out.println(item);
})
```
이렇게 Stream을 사용하지 않은 코드는 어떻게 동작하는지를 자세히 명시하고 있고 루프 인덱스인 i가 계속해서 증가한다.

반면 Stream을 사용한 코드는 무엇을 해야하는지를 표현하고있다. `stream()`메소드를 사용해 리스트를 데이터 스트림으로 변환한 뒤 각 요소를 직접 처리할 함수를 전달하고 있다.  
i와 같은 변수를 사용하여 인덱스를 직접 관리하지 않고 고차함수인 `forEach()`를 사용하여 함수를 전달하고 있다.

이렇게 Stream이 어떤 의도로 등장했는지 간단히 알아보았다.   
실제 사용하는 입장에서는 사실 for문이나 if문을 어떻게 대체할 수 있는지만 알면 되기 때문에 for, if문을 stream으로 대체하는방법에 대해 알아보자.

```java
Integer targetNum = null;

for(int i = 0; i < integerList.size(); i++) {
  System.out.println(integerList.get[i]);

  if(integerList.get(i).equals(100)) {
    targetNum = integerList.get(i);
    break;
  }
}
```
반복문을 돌며 원하는 숫자를 찾으면 반복문을 종료하는 코드다.  
이걸 Stream을 사용하는 바꿔보면...
```java
Integer targetNum = integerList.stream().filter(integer -> {
  System.out.println(integer);

  if(integer.equals(100)) {
    return true;
  }

  return false;
}).findAny().get();
```
`filter()`메소드는 조건에 따라 걸러서 반환하는 메소드로 위의 코드에선 `findAny().get()`메소드를 사용하여 처음으로 true가 등장하면 가장 처음으로 걸러진 데이터를 반환하는 코드다.

만약 반복문을 돌며 기존 값에서 +10을 시키고싶다면 어떨까?
```java
List<Integer> plusTenIntegerList = new ArrayList<>();

for(int i=0; i<integerList.size(); i++) {
  plusTenIntegerList.add(integerList.get(i) + 10);
}
```
이처럼 어떤 데이터를 가공하고 싶은 경우 사용하는 메소드는 `map()`이다. 다음과 같이 사용한다.
```java
List<Integer> plusTenIntegerList = integerList.stream().map(integer -> integer + 10).toList();
```
map메소드 내에 넣어준 함수대로 실행되고 이들을 스트림 형태로 반환하고 `toList()`메소드를 통해 List타입으로 변경하는 코드다.

filter와 map을 동시에 사용할수도 있는데 예를 들어 홀수들만 뽑아서 이들을 +10 시킬 수 있다.

```java
List<Integer> oddNumPlusTem = integerList.stream().filter(integer -> integer % 2 !=0).map(integer -> integer + 10).toList();
```
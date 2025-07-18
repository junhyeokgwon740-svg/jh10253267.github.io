---
layout: post
title:  "Java final 키워드"
author: 악어새62
categories: [ TIL]
image: assets/images/2.jpg
tags: [ ]
---
## 개요

Java의 final은 어떻게 동작하길래 불변성을 지킬 수 있는걸까? 한번 알아보자.

## final 키워드

Java의 final은 주로 한번 선언하고 더 이상 수정할 일이 없을 때 쓰인다.  
원주율 같은 예시가 있겠다.  

자바 코드는 자바 컴파일러를 거쳐 바이트 코드로 해석된다.  
이 때 컴파일러는 final이 붙은 변수나 클래스, 메소드를 확인하고 재할당이 검출되면 에러를 발생시킨다.

final 변수는 여기서 플래그를 붙인다. 
```java
Field ...
  flags: (0x0010) ACC_FINAL
```
JVM은 바이트코드 로딩 시 final 플래그를 해석하고 
상속이나 오버라이드 시 에러를 발생시킨다.  
JIT는 final 키워드를 보고 오버라이드가 없음을 보장하니 인라인 최적화를 공격적으로 적용시킨다.

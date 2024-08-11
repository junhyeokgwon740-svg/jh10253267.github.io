---
layout: post
title:  "Spring profiles"
author: 악어새62
categories: [ TIL, WEB, Backend]
image: assets/images/4.jpg
tags: [Web, Backend, Spring]
---
## 개요

스프링 부트 프로젝트를 할 때 application.properties파일이나 application.yml파일에 프로젝트의 기본 설정을 하곤한다.  
그러나 개발 환경에 따라 다른 설정을 사용해야하는 경우가 생긴다.

## Spring Profile

프로젝트의 설정을 개발 환경에 따라 다르게 적용할 수 있는 방법이다.

spring boot 2.4버전부터 .properties와 .yml파일이 동시에 있다면 properties가 나중에 로딩되도록 되어있어서 예상한대로 동작하지 않을 가능성이 있기 때문에 주의해야한다.

yml파일을 사용할 경우 하나의 profile에서 `---`로 구분하면 하나의 파일을 여러개로 나누어 사용하는 효과를 볼 수 있다.
```yml
spring:
  profiles:
    activate: local
---
spring:
  profiles: local
---
spring:
  profiles: dev
---
spring:
  profiles: prod
---
```
또한 하나의 설정파일에서 다른 profile을 포함시킬 수도 있다.
```yml
spring:
  profies:
    active: local
---
spring:
  profies: local
  include:
    - common
```
스프링 2.4 이후부턴 문법이 조금 바뀌었다.
```yml
spring:
  profiles:
    activate: local
---
spring:
  config:
    activate:
      on-profile: local
---
spring:
  config:
    activate:
      on-profile: prod
```
include도 group이라는 문법을 사용하도록 바뀌었다.
```yml
spring:
  profiles:
    activate: local
    group:
      local: 
        = common
      prod:
        -common
---
spring:
  config:
    activate:
      on-profile: common
---
spring:
  config:
    activate:
      on-profile: local
---
spring:
  config:
    activate:
      on-profile: prod
```

```yml
profiles:
  activate: local
  group:
    local: 
      - common
    prod:
      - common
```
이 부분은 디폴트로 local profile을 사용하고 local의 경우 local과 common을 그룹지어서 함께 사용, prod의 경우 prod, common을 그룹지어서 함께 사용한다는 의미의 설정이다.

이렇게 사용하는 것도 가능하고 별도의 파일로 분리해서 사용할 수도 있다.
application-local.yml과 같이 application-{profile}.yml과 같은 형식으로 사용해도 된다. 다른 설정 파일에서 다음과 같이 사용하면 된다.
```yml
spring:
  profiles:
    active: prod
```

인텔리제이 기준으로 run/debug configurations설정의 Active profies에서 어떤 profile을 사용할 것인지 설정할 수 있다.
따로 설정해주지 않으면 비어있는데 위에서 설정한 default설정을 사용하게 된다.
<img width="729" alt="스크린샷 2024-08-11 시간: 20 22 32" src="https://github.com/user-attachments/assets/2c2a90fb-e900-4aed-b0ec-d7e360dc07ef">

아니면 `java -jar -Dspring.profiles.active=local app.jar`와 같이 명령어를 주어 사용할 수도 있다.
이를 System Properties라고한다. 실행한 JVM 내부에서 접근 가능한 외부 설정으로 -D옵션을 사용하고 키와 벨류형식으로 입력하면 된다.
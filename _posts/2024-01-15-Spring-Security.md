---
layout: post
title:  "Spring Security"
author: 악어새62
categories: [ TIL, WEB, Backend ]
tags: [Spring, Security]
---
## 개요

사용자의 로그인과 세션 트래킹은 웹 애플리케이션에서 필수적인 기능이다.<br>
과거에는 HttpSession과 Cookie를 이용해서 직접 구현했지만 스프링을 적용할 때는 Spring Security와 약간의 설정을 적용하는 것 만으로도 구현이 가능하다.<br>

스프링 시큐리티는 원래는 별도의 프레임워크로 시작되었다고 한다. 그 후 스프링으로 프로젝트가 통합되었고 이를 사용하면 개발자는 약간의 코드와 설정만으로 로그인 처리와 자동 로그인, 로그인 후에 페이지 이동등을 처리할 수 있기 때문에 개발의 생산성을 높일 수 있다.<br>
만약 이러한 프레임워크가 없었다면 세션을 체크하는 로직을 구현해야할 것이고 리다이렉션을 시키는 로직을 개발자가 직접 구현해야할 것이다.

## 개념

스프링 시큐리티는 필터 기반으로 동작한다. 즉 스프링의 외부에서 동작하는 것이다.

클라이언트가 요청을 보내고 그 요청을 서블릿이나 JSP가 처리한다. 처음 응답을 받는 서블릿은 디스패쳐 서블릿이다. 필터는 요청이 디스패쳐 서블릿에 들어오기 전에 여러겹의 필터를 거치는데 이를 필터 체인이라고 한다. 마치 체인처럼 연결되어 작동하는 느낌이다.

우리가 스프링 시큐리티의 존재를 모르고 로그인 로그아웃을 구현한다면 아마 이렇게 할 것이다.

아이디와 비밀번호를 DB에 저장한다.  
사용자의 로그인 요청이 들어오면 작동할 쿼리를 작성한다.  
```sql
select * from user where id = 입력받은 아이디 and pw = 입력받은 비밀번호;
```
이렇게 로그인이 성공하면 정보를 세션에 담아 쿠키를 발행한다.  
로그인이 필요한 경로에서 요청과 함께 전송된 세션 쿠키를 받아 세션을 불러오고 세션이 존재하는지 체크한다. 만약 없다면 로그인 페이지로 리다이렉션 시킨다. 검증이 필요한 경로에서 매번 로직을 작성하니 중복된 코드를 개발자가 기계적으로 작성해야한다.

이를 해결하기 위해 필터를 활용한다.  
특정한 경로에 접근할 때 필터가 동작하도록 해서 동일한 로직을 필터로 분리시킨다.

로그인 서블릿 클래스
```java
@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
  String id = req.getParameter("id");
  String pw = req.getParameter("pw");

  HttpSession session = req.getSession();

  session.setAttribute("loginInfo", id);

  resp.sendRedirect("/mainpage");
}
```
로그인 체크 서블릿 클래스
```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
  //사용자의 세션 불러오기
  HttpSession session = req.getSession();

  if(session.isNew()) {
    resp.sendRedirect("/login");
    return;
  }
  
  if(session.getAttribute("loginInfo")) {
    resp.sendRedirect("/login");
    return;
  }

  resp.sendRedirect("/mainpage");
}
```

필터 적용
```java
@WebFilter(urlPatterns = {"/board/*"})
public class loginCheckFilter() implements Filter{
  @Override
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    HttpServletRequest req = (HttpServletRequest)request;
    HttpServletResponse resp = (HttpServletResponse)response;

    HttpSession session = req.getSession();

    if(session.getAttribute("loginInfo") == null) {
      resp.sendRedirect("/login");
      return;
    }

    //다음 필터나 목적지로 갈 수 있도록 함.
    chain.doFilter(request, response);
  }
}
```

이러한 기능들을 스프링 시큐리티를 사용한다면 간단히 설정만해도 사용할 수 있다.  
`(프레임워크 == 반제품)`

## 스프링 시큐리티 필터 체인

![필터 체인](https://github.com/jh10253267/TIL/assets/108499717/3ec08e0c-a091-46eb-9869-11f719a40e89)
* `SecurityContextPersistenceFilter` : SecurityContextRepository에서 SecurityContext를 가져오거나 저장한다.
* `LogoutFilter` : 로그아웃 요청을 감시하고 해당 유저를 로그아웃처리한다.
* `UsernamePasswordAuthenticationFilter` : 설정한 경로로 오는 요청을 감시하고 유저 인증처리를 한다. 인증 과정은 다음과 같다. 1. AuthenticationManager를 이용한 인증 실행 2. 인증 성공시 Authentication 객체를 SecurityContext에 저장후 AuthenticationSuccessHandler 실행 3. 인증 실패시 AuthenticationFailureHandler 실행 
* `DefaultLoginPageGeneratingFilter` : 인증을 위한 로그인폼 경로를 감시한다.
* `BasicAuthenticationFilter` : HTTP 기본 인증 헤더를 감시하여 처리
* `RequestCacheAwareFilter` : 로그인 성공후 원래 요청 정보를 재구성하기위해 사용
* `SecurityContextHolderAwareRequestFilter` : 필터 체인상의 다음 필터들에게 부가 정보를 제공한다.
* `AnonymousAuthenticationFilter` : 이 필터가 호출되는 시점까지 사용자 정보가 인증되지 않으면 인증 토큰에 사용자가 익명 사용자로 나타난다.
* `SessionManagementFilter` : 인증된 사용자와 관련된 모든 세션을 추적한다.
* `ExceptionTranslationFilter` : 보호된 요청을 처리하는 중에 발생할 수 있는 예외를 위임하거나 전달하는 역할을 한다.
* `FilterSecurityInterceptor` : AccessDecisionManager로 권한부여 처리를 위임해 접근 제어결정을 돕는다.
![mceclip2](https://github.com/jh10253267/TIL/assets/108499717/aaccd38d-1582-4c8e-b5b3-130de7b2b365)
실제 로그인을 처리하는 필터는 `UsernamePasswordAuthenticationFilter`이다. 이는 `AbstractAuthenticationProcessionFilter`를 구현한 것이다.  
(과거에는 `AuthenticationFilter`를 사용했다고 한다.)

필터의 동작 순서는 이러하다.

유저가 로그인을 시도한다.  
* AuthenticationManager
* AuthenticationProvider
* UserDetailsService

이들을 통해 유저의 정보를 읽어오고 인증을 처리한다.  
이중 가장 중요한 요소는 실제로 인증을 처리하는 `UserDetailsService이다`.  
`UserDetailsService`는 인터페이스로 `loadUesrByUsername`이라는 단 하나의 메소드만 가진다. 인터페이스니 개발자가 구현해서 사용하면 되는데 이렇게 설계된 이유는 유저의 정보는 db에 저장되어있고 데이터베이스와 시큐리티는 별개의 관심사기 때문이 아닐까 생각한다.  
(여기서 username은 스프링 시큐리티에서 사용하는 명칭으로 우리가 흔히 사용하는 아이디를 말한다.)

반환 타입은 `UserDetails`라는 이름의 인터페이스 타입이다.  
사용자 인증과 관련된 정보들을 저장하는 역할을한다.

따라서 해당 인터페이스를 직접 구현하여 사용하는 것도 가능하고 이렇게 만들어진 클래스는 UserDetails 타입을 필요로하는 부분에 끼워 넣을 수 있다.

나는 주로 User클래스를 사용하곤 하는데 이는 `UserDetails`의 구현체로 스프링 시큐리티에서 제공하고있다. 빌더타입을 지원하고 username, password, authorities를 넣어 생성하면 된다. 아니면 직접 UserDetails를 구현하여 UserDetailsImpl와 같이 사용해도 상관없다.

스프링 시큐리티는 인메모리 세션 저장소인 `SecurityContextHolder`에 `UserDetails`정보를 저장한다.  
클라이언트에게 세션ID와 함께 응답을 한다.

이후 요청이 들어왔을 때 쿠키의 세션ID정보를 통해 로그인된 정보가 존재하는지 확인한다.  
유효한 정보라면 인증처리를 해준다.

## 사용

스프링 시큐리티는 단순히 `applicatioin.properties`를 이용하는 설정보다 코드를 이용해서 설정을 조정하는 경우가 더 많다.

Config파일을 하나 작성해서 사용하면 된다.
`CustomSecurityConfig`클래스를 추가해서 작성한다.

처음 적용을 하면 내 프로젝트의 모든 경로에 대해서 로그인을 필요로 한다. 로그인이 필요한 경로와 그렇지 않은 경우가 있기에 따로 셋팅을 해주는 게 좋다.
해당 Config파일에 `SecurityFilterChain` 객체를 반환하는 메소드를 작성한다.

일단 별도의 인증절차를 거치지 않도록 구성한다.

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) {
 return http.build(); 
}
```
이렇게 설정하면 별도의 인증절차없이 사용이 가능하다.

스프링 시큐리티의 동작은 웹에서 사용하는 필터를 통해서 동작하고 많은 수의 필터들이 단계별로 동작한다. 따라서 문제가 발생하면 어떤 필터에서 문제가 생겼는지 알 수 있도록 로그 설정을 최대한 낮게 설정해서 관련된 에러를 볼 수 있게 설정하는 것이 좋다.

정적자원의 처리:  
어떤 파일에 접근하는 것은 필터를 적용하지 않아도 된다. 예를 들면
정적 파일 css나 js와 같은 파일에도 필터가 적용되곤 한다. 
이를 Config파일에서 설정해 줄 수 있다.
다음과 같은 메소드를 추가해준다.
```java
@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
  return (web) -> web.ignoring().requestMatchers(PathRequest.toStaticResources(.atCommonLocations());
}
```

## 인증과 인가

인증 : 스스로를 증명하다라는 뜻이며 로그인의 개념이다.  
인가 : 허가나 권한이라는 개념으로 특정한 자원에 접근할 수 있는 권한이 있는지를 확인하는 개념이다.

놀이공원을 생각해보면 이해하기 쉽다.  
놀이공원에 들어가기 위해서는 일단 티켓이 필요하다. 그러나 티켓이 있다고 해서 모든 시설을 이용할 수는 없다.

동물원을 이용할 계획이라면 돈을 지불하고 동물원 티켓을 사야하고 탈 수 있는 놀이기구도 티켓의 등급에 따라서 다르다.  
인증은 놀이공원 입장 티켓이다. 인가는 내가 접근하려는 리소스를 소지한 티켓으로 이용할 수 있는지 확인하는 작업이다.

 스프링 시큐리티에서 로그인에 해당하는 인증 단계는 다음과 같은 과정을 거친다.
 1. 사용자 아이디만으로 사용자의 정보를 로딩한다.
 2. 로딩된 사용자의 정보를 이용해서 패스워드를 검증한다.

 인증처리는 `인증 제공자(Authentication Provider)` 라는 존재를 이용해서 처리되는데 인증 제공자와 그 이하의 흐름은 일반적으로 커스터마이징해서 사용하는 경우는 거의 없다.

 인증 처리를 위한 `UserDetailService`  
 위에서 살펴본 것 처럼 가장 중요한 객체는 실제로 인증을 처리하는 `UserDetailService`라는 인터페이스의 구현체이다.  
 개발자가 구현해야하는 부분은 해당 인터페이스를 구현해서 username이라는 사용자의 아이디 인증을 코드로 구현하는 것이다.

 `loadUserByUsername`의 반환 타입은 `UserDetails`라는 인터페이스 타입으로 지정되어있다. 이는 사용자인증과 관련된 정보들을 저장하는 역할을 한다.  
 이 클래스의 세부 사항은 직접 작성해주어야한다. 스프링 시큐리티를 사용하는 사용자가 어떤 이름으로 유저의 정보를 저장하고 있는지, 예를 들면 어떤 서비스에서는 로그인할 때 아이디가 아닌 email을 사용할 것이고 이건 프레임워크를 제공하는 입장에서는 알 수 없기 때문이다.  

 또한 시큐리티와 데이터베이스는 별개의 관심사기 때문에 이렇게 인터페이스로 제공하고 있다.
 스프링 시큐리티는 내부적으로 `UserDetails`타입의 객체를 이용해서 패스워드를 검사하고 사용자 권한을 확인하는 방식으로 동작한다.

 스프링 시큐리티의 api에는 `userDetails`라는 인터페이스를 구현한 User라는 클래스를 제공하고 빌더방식을 지원하므로 앞에서 사용한 loadUserByUsername()에 약간의 코드를 추가해줄 수 있다.
 ```java
 UserDetails userDetails = User.builder()
        .username("tester1")
        .password(passwordEncoder.encode(("tester1"))
        .authorities("ROLE_USER")
        .build();

return userDetails;
 ```
유저의 정보는 데이터베이스에 있으니 입력받은 username을 가지고 레포지토리에서 조회해온 후 위와 같은 식으로 UserDetails타입의 클래스로 리턴해주면 된다.

 ## PasswordEncoder

스프링 시큐리티는 기본적으로 `PasswordEncoder`를 필요로 한다. 이는 보안상의 이유로 만약 비밀번호가 암호화되지 않고 저장되어있다면 데이터베이스를 열람할 수 있는 사람이 모든 유저의 비밀번호를 알 수 있기 때문이다.  
따라서 비밀번호를 알아보기 어렵게 만들어 데이터베이스에 저장한다.

`PasswordEncoder`는 인터페이스로 제공되는데 이를 구현하거나 스프링 시큐리티 api에서 제공하는 클래스를 지정할 수 있다. 해당 타입의 클래스중 `BCryptPasswordEncoder`는 해시 알고리즘으로 암호화 처리되는데 같은 문자열이라도 매번 해시 처리된 결과가 다르다는 특징이 있다. 결과가 매번 다르게 나오는 이유는 레인보우 어택과 같은 해킹 기법을 방어하기 위해서다. 만약 A를 암호화하면 `%`가 나오고 AB를 암호화하면 `%^`, ABC를 암호화하면 `%^&`가 된다고 해보자. 이런 상황이라면 암호문을 가지고 테이블을 만들어 유추하여 암호를 해독할 수 있게 된다. 이것을 레인보우 어택이라고 한다. 그래서 임의의 문자열을 추가하여 입력값이 같더라도 매번 다른 암호문이 나오게된다.  

BCrypt는 대표적인 단방향 암호화 알고리즘이다. 즉, 평문을 암호화할 수 있지만 암호문을 평문으로 복호화할 수는 없다. 그러니 암호문이 유출되더라도 그 의미를 알 수 없어서 의미없는 값이 되는 것이다.  

여기서 궁금했다.  
매번 암호화 된 값이 달라지고 복호화할 수 없는데 어떻게 사용자가 입력한 값과 데이터베이스에 있는 값이 일치하는 지 확인해서 로그인 처리를 해줄 수 있을까?  
이번 기회에 한 번 알아보자.

`$2b$12$EXAMPLESALTzUO1zZ1XpF.Ej5H2HpFGABY.BC53J2RXVVN`  
BCrypt알고리즘으로 암호화된 암호문은 위와 같은 형태를 가지고있다.  
이는 의미를 가지는 각각의 부분으로 나눠볼 수 있다.    
* Version : 알고리즘의 버전을 나타낸다.
* Salt : 랜덤하게 생성되는 값으로 매번 암호화 결과가 달라진다. 
* Costfactor : 공격자가 해킹 시도 시 크래킹을 어렵고 시간 소모적으로 만든다. 

위의 문자열의 경우 `$2b$`는 Bcrypt의 버전을 의미하고 2b는 현재 표준 버전 중 하나라고 한다.  
뒤의 `$12$`는 Cost Factor(작업 비용)를 나타낸다.  
비용 인자는 해시를 계산하는데 걸리는 시간을 결정한다. 이 숫자는 2의 제곱으로 표현되며 위의 문자열의 경우 2의 12제곱을 의미한다. 비용인자가 높을수록 해시를 생성하는데 더 많은 시간이 걸리며 이는 해시를 역산하려는 공격을 더 어렵게, 시간 소모적으로 만든다. 이 경우 일반적으로 수 밀리초에서 몇 초 정도가 된다고 한다.  
EXAMPLESALT는 해시화시 사용된 Salt값이다. `$EXAMPLESALTzUO1zZ1XpF.` 솔트는 고정된 길이의 랜덤 문자열로 입력 비밀번호와 결합되어 해시를 생성한다.
그 외의 문자열은 비밀번호의 해시 값이며 이는 비밀번호와 salt 그리고 작업 비용이 결합되어 생성된 고유의 해시 값이다.

PasswordEncoder가 로그인 요청시 입력받은 비밀번호와 데이터베이스의 비밀번호가 일치하는지 확인하는 시나리오는 다음과 같다.

1. 사용자가 로그인 시 비밀번호를 입력하면 데이터베이스에서 암호문을 가져온다.  
2. 여기서 `salt`와 `work factor`를 추출한다.  
3. 사용자가 입력한 비밀번호와 데이터베이스의 비밀번호 해시값에서 추출한 `salt`, `workfactor`를 사용하여 입력한 비밀번호를 해시화하고 이 두 가지를 비교하여 일치한다면 같은 문자열로 판단한다.

로그인 기능이 제대로 동작하려면 Config파일에 PasswordEncoder를 Bean으로 등록하고 이를 주입해줘야한다. 

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

## 인가

권한 체크는 특정한 경로에 지정하는 경우가 일반적인데 예를 들어 게시글 목록은 모두가 볼 수 있어야하지만 게시글 작성은 로그인이 되어야한다. 이런 권한 설정은 코드로 작성할 수도 있고 어노테이션을 이용해서 지정할 수도 있다. 어노테이션을 통해 지정하는 방식은 설정 관련 클래스에
`@EnableGlobalMethodSecurity` 를 추가해줘야 한다.
그리고 `@PostAuthrize`어노테이션을 통해 사전, 사후 권한을 체크할 수 있다. 여기서 자주 사용하는 표현식은 다음과 같다.
* `authenticated`
* `permitAll`
* `anonymous`
* `hasRole`
* `hasAnyRole`

## JWT를 이용한 인증/인가

### 개요

로그인 방법은 세션만 있는 것이 아니다. **JWT**(**J**SON **W**eb **T**oken)는 json문자열로 이루어진 문자열로 인증/인가를 요청한다. JWT를 사용하는 이유는 다음과 같다.
* 최근의 프론트엔드는 React나 Vue를 이용하여 서버와 클라이언트의 개발을 분리시켜 진행한다. 프론트엔드는 Ajax를 이용해서 서버를 호출하고 데이터를 받아 처리하게된다.  
이러한 이유로 HttpSession이나 쿠키를 사용한 인증방식에 제약이 생긴다. 
* 세션을 이용한 인증 방법의 경우 세션을 유지하는데에 서버의 자원이 소모되기 때문에 사용자가 많아진다면 서버의 메모리 사용량이 증가하고 문제로 이어질 수 있다.

이를 해결하는 방법은 여러가지가 있다.

* API서버를 호출하는 프로그램의 IP를 서버에 저장해두고 요청이 들어왔을 때 호출한 IP와 서버에 저장된 IP를 비교하여 허용된 IP에 대해서만 서버에서 결과를 만들어주는 방식. IP와 더불어 정해진 키 값을 이용하는 것이 일반적이라고 한다. 
* 인증에 성공하면 JWT토큰을 발급해준다. API를 호출할 때 토큰을 같이 전달해 API서버에서 토큰을 확인하고 결과를 만들어주는 방식이다. 이러한 토큰을 Access token이라고 부른다.

유효한 Access token을 가지고있다면 인증된 사용자로 판단한다. 그러나 토큰 자체가 인증 기능을 하고 단순한 문자열이라는 특징, 토큰이 클라이언트 측에 보관된다는 점 때문에 탈취당할수 있고 보안상 취약점이 있다. 따라서 Access token의 유효기간을 최대한 짧게 설정하고 Access token을 재발급받을 수 있는 Refresh token을 사용하여 인증처리를 하는 것이 일반적이다.  
이렇게 한다면 악의적 사용자가 Access token을 탈취하더라도 토큰의 유효기간이 짧으니 악의적 행동을 할 수 있는 시간이 짧아지고 그로 인해 예상되는 피해도 줄어들 것이다.

그러나 만약 Refresh token을 탈취당한다면 어떨까?  
해커가 이 토큰을 가지고 얼마든지 Access token을 재발급 받을 수 있게된다. Refresh token은 유효기간도 길기 때문에 사실상 비밀번호를 누군가 알아낸 것과 같다.   
이런 상황을 대비하여 토큰을 발급할 때 Refresh token을 데이터베이스에 저장하는 방법을 사용할 수 있다.  
사용자가 로그인하면 Access token과 Refresh token을 발급해주고 Refresh token을 데이터베이스에 저장한다.  
이렇게 한다면 악의적 사용자가 탈취한 Refresh token을 사용하고 있는 중에 원래의 사용자가 토큰을 새로 발급받는다면 데이터베이스에 새로 발급받은 토큰 정보로 새롭게 갱신되어 탈취당한 Refresh token이 아무 효력이 없게된다.  

이것은 마치 우리가 신용카드를 잃어버렸을 때 카드를 중지시키고 새롭게 발급받는 것과 같은 아이디어로 이러면 누군가 신용카드를 주워서 사용하려 하더라도 카드만 있을 뿐이지 사용할 수 없으니 아무 쓸모가 없어진다.

### 스프링 시큐리티 + JWT를 사용하는 방법

구현 방법은 크게 두 가지가 있다.
 * 컨트롤러단에서 로그인을 하고 서비스단에서 토큰을 발급해주는 방법
 * 필터를 사용해 로그인을 하는 방법.(스프링 시큐리티를 활용한 방법)
개인적으론 필터를 통한 로그인 방식을 선호한다. 스프링 시큐리티를 사용하며 이미 구현되어있는 프레임워크의 기능을 이용하는 것이 효율적이라는 생각이 들고 비즈니스 로직과 시큐리티를 분리하여 비즈니스 로직에만 집중할 수 있도록 하기 위해서다.

위에서 살펴봤듯이 스프링 시큐리티에선 이미 로그인을 처리하는 필터와 방식이 존재한다.  
`UsernamePasswordAuthenticationFilter`에서 로그인을 처리한다.    

스프링 시큐리티에서 말하자면 커스텀해서 사용할 수 있는 로그인 필터를 제공한다.  
바로 `AbstractAuthenticationProcessingFilter`이다.  
이를 상속받은 클래스가 바로 `UsernamePasswordAuthenticationFilter`이다.  
따라서 위의 추상 클래스를 상속받아 JWT에 맞게 구현한다면 얼마든지 커스텀 로그인 필터를 직접 만들 수 있다.  
`AbstractAuthenticationProcessingFilter`는 `AuthenticationManager`를 필수적으로 설정해줘야한다. 위에서 살펴본 이유이다.

이는 `SecurityConfig` 설정 클래스에서 할 수 있다.  
인증을 하는데 `UserDetailsService`도 필요하기 때문에 같이 설정해준다.

```java
AuthenticationManagerBuilder authenticationManagerBuilder =
                http.getSharedObject(AuthenticationManagerBuilder.class);

        authenticationManagerBuilder
                .userDetailsService(apiUserDetailsService)
                .passwordEncoder(passwordEncoder());

        AuthenticationManager authenticationManager =
                authenticationManagerBuilder.build();

        http.authenticationManager(authenticationManager);
```
그리고 LoginFilter를 만들어준다.  
위에서 말한대로 `AbstractAuthenticationProcessingFilter`를 상속받아 구현한다.

필요한 로직은 크게 두 가지가 있다.
1. json을 파싱하여 id와 pw를 알아내는 로직
2. 인증정보를 만드는 부분.

1번의 로직은 어렵지 않고 인증 정보를 만드는 부분은 UsernamePasswordAuthenticationToken을 만들어서 설정해둔 AuthenticationManager를 사용해 Authentication타입의 객체를 리턴해주면 된다.

```java
public class APILoginFilter extends AbstractAuthenticationProcessingFilter {
    public APILoginFilter(String defaultFilterProcessesUrl) {
        super(defaultFilterProcessesUrl);
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {
        if(request.getMethod().equalsIgnoreCase("GET")) {
            log.info("GET METHOD NOW ALLOWED");
            return null;
        }
        //사용자로 부터 입력받은 로그인 정보를 파싱
        Map<String, String> jsonData = parseRequestJSON(request);

        //인증정보를 만들어 다음 필터로 넘겨줌
        UsernamePasswordAuthenticationToken authenticationToken
                = new UsernamePasswordAuthenticationToken(
                        jsonData.get("mid"),
                        jsonData.get("mpw"));

        return getAuthenticationManager().authenticate(authenticationToken);
    }

    private Map<String, String> parseRequestJSON(HttpServletRequest request) {
        try(Reader reader = new InputStreamReader(request.getInputStream())) {
            ObjectMapper objectMapper = new ObjectMapper();

            return objectMapper.readValue(reader, Map.class);
        } catch (Exception e) {
            log.error(e.getMessage());
        }
        return null;
    }
}
```
```java
//Security Config
http.addFilterBefore(apiLoginFilter, UsernamePasswordAuthenticationFilter.class);
```

테스트해보면 로그인 처리가 완료되어 루트 경로로 리다이렉션되는 것을 볼 수 있다.  
로그인이 성공한 뒤 스프링 시큐리티는 디폴트로 / 경로로 리다이렉션을 시켜준다. 이는 `AuthenticationSuccessHandler`의 역할로 만약 변경하고 싶다면 이 부분을 커스텀해주면 된다.
(여담으로 내가 누군가에게 오버라이드를 쉽게 설명해줄 때 오버라이딩은 커스텀이라고 한다. 일반화되어있는 클래스를 상속받아 기본적인 골격과 기능을 하나하나 직접 구현할 필요없이 내 필요에 맞게 조금씩 수정해서 사용하는 것이다.)

`AuthenticationSuccessHandler`를 상속받아 커스텀 `LoginSuccessHandler`클래스를 작성해준다. jwt를 이용한 인증/인가 방식을 사용하고 있으니 로그인이 성공한 뒤 Access Token을 발급해주어야한다.

jwt토큰을 생성하고 검증하는 등의 로직은 직접 구현할 필요없이 `jjwt`라이브러리를 사용하면 된다.  
개발자가 작성해야할 부분은 어떤 정보들을 claim으로 넣어줄 것인지, 어떤 알고리즘을 사용할지, 어떤 암호화, 복호화 키를 사용할 것인지 정도가 된다.

지금까지의 과정을 정리해보면 아래와 같다.
1. 클라이언트가 "`/로그인 필터에서 지정해둔 경로`" 요청을 보낸다.
2. 필터를 거쳐 커스텀한 로그인 필터에 도착한다.
3. 로그인을 시도하고 성공한다면 `커스텀 AuthenticationSuccessHandler`가 낚아채서 토큰을 생성한다.

### JWT 토큰 검증

필터를 사용해 인증이 필요한 경로에 대한 요청에 대해 검증해주면 된다.  
베이직한 검증 과정은 유효 기간과 서명 부분을 확인하는 것이다.  
만약 유효기간이 지났다면 인증처리를 해줄 수 없고 서명이 다르다는 것은 토큰에 변조, 위조가 일어났다는 것을 의미하기 때문에 인증처리를 해줄 수 없다.  
이 때 사용하는 필터는 `OncePerRequestFilter`이다.
이름 그대로 하나의 요청에 대해서 한번씩 동작하는 필터로 서블릿의 필터와 유사하다.  
이 클래스를 상속받아 토큰을 검증해주면 된다.  

이러한 방식으로 인증을 수행할 때 스프링 시큐리티에서 사용하는 `SecurityContextHolder`와 관련된 기능들을 사용할 수 없다는 단점이 있다.  
그래서 토큰 체크 필터에서 토큰을 체크하고 유효한 토큰이라면 인증 정보를 `SecurityContextHolder`에 저장하여 사용하도록 설정해줄 수 있다.
```java
SecurityContextHolder.getContext().setAuthentication(authentication);
```
이렇게 하면 컨트롤러단에서 유저의 인증정보를 받아서 사용할 수 있다.  
예를 들면 게시글의 작성자만 수정할 수 있게하기 위해 컨트롤러에서 요청한 유저의 정보를 받아서 비즈니스 로직에서 요청자와 작성자를 비교한다던지 ADMIN 권한을 가진 사용자만 접근할 수 있게 위에서 살펴본 표현식을 이용한다던지 할 수 있다.

생각해보면 JWT토큰을 이용한 방식은 세션을 사용하지 않는다. 세션 방식의 경우 하나의 세션이 생성되고 사용자 정보를 SecurityContext에 담아 사용한다. 로그인 이후 요청에서는 세션에 저장된 정보를 이용해 사용자의 정보를 읽어온다.  
유효한 JWT토큰을 가지고 있다면 이미 인증을 완료한 사용자기 때문에 필수적인 것은 토큰 자체의 유효성 검사뿐이다. 

그러나 JWT의 경우 세션을 사용하지 않기 때문에 위와 같이 구현할 경우 매 요청마다 새롭게 인증정보를 구성하게 된다. 이는 매 요청을 보낼 때마다 로그인을 하는 것과 다름이 없게된다. 이 과정에서 데이터베이스 호출 역시 피할 수 없게되고 그로인해 성능상의 손해를 보게된다.

다른 대안으로는 데이터베이스를 조회하지 않고 토큰을 파싱하여 인증정보를 구성하고 위와같이 시큐리티 컨텍스트에 넣어주는 방법도 있다.  

Spring OAuth 라이브러리의 경우 JWT토큰 정보를 파싱하여 인증 객체를 만드는 방법을 제공하고 있다. `JwtAuthenticationConverter`는 Jwt타입을 인자로 받아 인증정보를 인증정보를 추출하고 이를 SecurityContext로 넘겨줄 수 있다.

## 레거시 스프링 시큐리티

레거시 프로젝트를 하며 공부했던 내용도 같이 정리해본다.  
기본 인증/인가 시나리오는 스프링 부트와 크게 다르지 않다.

일단 의존성을 설정해야한다. 일반적으로 pom.xml파일에서 의존성을 관리할 때 호환성 문제가 생긴다던지 버전 변경을 해야한다던지 하는 경우가 생길 수 있기 때문에 Spring 버전을 일종의 변수로 지정하여 사용하곤 한다.

```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.target>1.8</maven.compiler.target>
  <maven.compiler.source>1.8</maven.compiler.source>
  <junit.version>5.9.1</junit.version>
  <spring.version>5.2.2.RELEASE</spring.version>
</properties>
```

이런 식으로.  
이렇게 하면 버전을 변경해야할 때에도 간단하게 properties의 한 부분만 변경해주면 모든 스프링 의존성의 버전을 관리할 수 있다.

나는 스프링 5.3점대 버전을 사용하고 있었는데 오류가나서 봤더니 스프링 시큐리티는 내가 사용하던 버전이 없었다. 그래서 버전을 스프링 시큐리티에 맞춰서 적용했다.

앞의 부분들은 이전 Web 03포스팅에서 찾아볼 수 있다.

스프링 시큐리티를 사용하려면 `AbstractSecurityWebApplicationInitializer`를 상속받는 클래스를 작성해주어야한다. 이렇게 하면 스프링 시큐리티가 제공하는 필터들을 사용할 수 있게 활성화 시켜준다.

다음으로는 어떤 경로들에 대해 인증을 거치도록 할 것인지 그리고 암호를 처리할 때 어떤 알고리즘을 사용할 것인지등을 설정해줘야한다.

`WebSecurityConfigurerAdapter`클래스를 상속받은 시큐리티 컨피그 클래스를 작성하고 여기서 설정해준다.

이렇게 생성한 클래스에 `@EnableWebSecurity` 애노테이션을 붙여서 스프링 시큐리티의 기본적인 빈이 자동으로 구성될 수 있도록한다.

주된 설정 부분은 다음과 같다.
```java
// 인증 인가 처리를 거치지 않도록 설정.
// 예를 들어 favicon.ico나 css와 같은 정적 파일들에 대해서는 인증을 거치지 않아도 되기 때문에 이 부분을 적어준다.
@Override
public void configure(WebSecurity web) {
  web.ignoring().antMatchers("경로1", "경로2");
}

// 우리가 웹 서비스를 이용할 때 로그인하지 않아도 볼 수 있는 페이지가 있고 아닌 페이지가 있다. 여기서 설정해주면 된다. 아래의 코드는 루트 경로와 /main경로에 대해 허가하고있고 그 외의 경로들에 대해서는 인증을 거치도록 설정하고 있다.
//
@Override
protected void configure(HttpSecurity http) {
  http
    .csrf().disable()
    .authorizeRequests()
    .antMatchers("/", "/main").permitAll()
    .anyRequest().authenticated();
}

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(customUserDetailsService);
}

@Bean
public PasswordEncoder encoder() {
  return new BCryptPasswordEncoder();
}
```
```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(customUserDetailsService);
}
```
이 부분은 인증 인가 처리를 UserDetailsService를 구현한 구현체로 로그인 처리를 하도록 설정하는 부분이다.

http.으로 시작하는 메소드 체이닝 설정에서 로그인 정보가 넘어오는 경로가 어떤 경로인지, 프론트에서 어떤 이름으로 아이디와 비밀번호가 넘어오는지 등을 설정한다.

그리고 로그인 처리를 어떤 url에서 할 것인지, 로그인이 성공한 다음 어떤 경로로 리다이렉트시킬지, 로그인에 실패했을 때 어떤 페이지를 보여줄 것인지등을 설정할 수 있다.

```java
.and()
  .formLogin()
  .loginPage("/members/loginform")
  // 프론트의 input태그의 name속성을 적어준다.
  .usernameParameter("loginEmail")
  .passwordParameter("password")
  // form의 전송 경로를 아래에 적어준다.
  .loginProcessingUrl("/authenticate")
  .failureForwardUrl("/members/loginerror?login_error=1")
  .defaultSuccessUrl("/", true)
  .permitAll()
.and()
  .logout()
  .logoutUrl("/logout")
  .logoutSuccessUrl("/");
```

이렇게 설정하면 AuthenticationFilter에서 아이디와 암호를 입력받아 로그인을 처리하게된다.

logoutUrl을 설정해주면 여기 명시된 경로로 요청이 오면 스프링 시큐리티에서는 세션에서 로그인 정보를 삭제하여 로그아웃처리한다.

위에서 살펴보았듯이 로그인을 처리하는 필터는 AuthenticationFilter로 내부에서 UserDetailsService 타입의 클래스를 사용한다.

로그인 처리를 하려면 데이터베이스에서 해당하는 유저의 정보를 가져와서 사용해야한다.  
이 부분을 구현할 때 주의해야하는 점은 시큐리티와 데이터베이스는 별개의 계층이기 때문에 구조적으로 분리되어야한다는 것이다.

유저의 정보를 불러오기 위한 유저 서비스 클래스를 작성하고 내부에서 dao를 통해 유저의 정보와 권한을 불러온다.

UserDetailsService의 `loadUserByUsername`메소드는 UserDetails타입을 리턴하고 필터에서 적절한 인증 정보가 넘어오면 로그인처리하는 식이다.
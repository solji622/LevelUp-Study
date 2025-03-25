# Spring Security (스프링 시큐리티)
&nbsp;

## 📌 스프링 시큐리티란
Spring 기반 애플리케이션의 **인증, 인가** 및 보안을 담당하는 프레임워크 <br>
**필터** 기반으로 동작하기에 Spring MVC와 분리되어 관리 및 동작이 되고 <br>
데이터 보호 기능을 포함하고 있어서 사용자 관리 기능 구현에 도움을 준다.

> **인증** : 해당 사용자가 본인이 맞는지 확인하는 것, '증명하다'와 같은 의미 <br>
> **인가** : 인증된 사용자가 요청된 자원에 접근 가능한가를 결정하는 것, '권한부여' 또는 '허가'와 같은 의미

<br>
<br>
<br>
<br>

## ❓ 필터(Filter) 기반으로 동작한다
필터는 디스패처 서블릿(Dispatcher Servlet)에 요청이 전달되기 전/후에 요청에 대한 인증, 인가 처리를 하는 것을 의미한다. <br>
스프링 범위 밖에서 처리가 되기에 스프링 컨테이너가 아닌 웹 컨테이너에 의해 관리가 된다. <br>
![필터 과정](https://github.com/solji622/LevelUp-Study/blob/ab83e8ad3888371a120a378a99c8b6adadb39734/25.03/Spring%20Security/asset/%ED%95%84%ED%84%B0%20%EA%B3%BC%EC%A0%95.png)
> 필터를 그럼 **왜** 사용하는가?

요청으로 들어온 정보에 그대로 노출되면 위험한 정보들이 들어있을 수 있다. <br>
이를 디스패처 서블릿이 처리하기 전에 필터에서 인증, 인가 처리를 해야한다. <br>

<br>
<br>
<br>
<br>

## ❓ 스프링 시큐리티, 왜 사용해야할까?
1. **스프링 프로젝트에서 적용하기 쉬운 프레임워크** <br>
    스프링 부트(Spring Boot)를 예시로 들었을 때, gradle 파일에서 Dependencies에 스프링 시큐리티를 추가해주면 간단하게 적용이 가능하다.
<br>

2. **개발을 위한 참고자료가 많다** <br>
  검색만 하면 모든 정보가 나오는 시대에서 참고자료가 많다는 것은 그만큼 빠른 개발이 가능하다는 것이다.

<br>

3. **개발 작업 효율성 증가** <br>
  보안에 필요한 기능들을 제공해주기 때문에 개발자 입장에서는 일일이 보안 관련 로직을 작성하지 않아도 된다. <br>
  만약 스프링 시큐리티를 사용하지 않을 경우 직접 세션을 체크하고 redirect 등을 해야만 하기에 비효율적이게 된다. <br>

<br>
<br>
<br>
<br>

## 📌 스프링 시큐리티 인증 처리 과정
![스프링 아키텍쳐](https://github.com/solji622/LevelUp-Study/blob/ab83e8ad3888371a120a378a99c8b6adadb39734/25.03/Spring%20Security/asset/%EC%8A%A4%ED%94%84%EB%A7%81%20%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90.png)
1. **Http Request 수신** <br>
   사용자(클라이언트)가 로그인 정보와 함께 인증 요청을 한다. <br>
   <br>

2. **유저 자격 기반 인증 토큰 생성** <br>
   AuthenticationFilter가 인증 요청을 가로채서 UsernamePasswordAuthenticationToken(이하 UserToken) 이라는 인증용 객체를 생성한다. <br>
   해당 객체는 요청을 처리할 수 있는 Provider을 찾는데 사용된다. <br>
   <br>

3. **필터를 통해 UserToken을 AuthenticationManager로 위임** <br>
   AuthenticationManager의 구현체인 ProviderManager에게 생성한 UserToken 객체를 전달한다. <br>
   <br>

4. **AuthenticationProvider의 목록으로 인증 시도** <br>
   AuthenticationManager는 AuthenticationProvider에게(실제 인증을 할 역할) UserToken을 다시 전달하고 <br>
   등록된 AuthenticationProvider들을 조회하여 인증을 요구한다. <br>
   <br>

5. **UserDetailsService에서 조회** <br>
   데이터베이스에서 사용자 인증 정보를 가져올 UserDetailsService에게 <br>
   AuthenticationProvider가 사용자 아이디를 전달하여 사용자 정보를 조회한다. <br>
   <br>
   
6. **UserDetails를 이용해 User객체에 대한 정보 탐색** <br>
   UserDetailsService을 통해 넘겨받은 사용자 정보로 UserDetails 객체를 만든다. <br>
   <br>

7. **User 객체의 정보들을 UserDetails가 UserDetailsService(LoginService)로 전달** <br>
   AuthenticaitonProvider들은 UserDetails를 넘겨받고 사용자 정보와 화면에서 입력한 로그인 정보를 비교한다. <br>
   <br>

8. **비교 후 Authentication 반환** <br>
    인증이 완료 후 권한 등의 사용자 정보를 담은 Authentication 객체를 반환한다. <br>
   <br>

9. **인증 완료, Filter에게 반환** <br>
    다시 최초의 AuthenticationFilter에 Authentication 객체가 반환된다. <br>
   <br>

10. **SecurityContext에 인증 객체를 설정** <br>
    필터로 반환된 Authentication 객체를 Security Context에 저장한다. <br>
   <br>

<br>
<br>
<br>
<br>

## 📌 스프링 시큐리티의 주요 모듈
### 




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
![스프링 아키텍쳐](https://github.com/solji622/LevelUp-Study/blob/546f9b3c7d28ea33167ebc41f7f647e2837a0130/25.03/Spring%20Security/asset/%EC%8A%A4%ED%94%84%EB%A7%81%20%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0%20%EB%8F%99%EC%9E%91%20%EA%B3%BC%EC%A0%95_%ED%95%9C%EA%B5%AD%EC%96%B4.png)
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

### ▪️SecurityContextHolder
보안 주체의 세부 정보를 포함하여 응용 프로그램의 현재 보안 context에 대한 세부 정보를 저장한다. <br>
기본적으로 아래 두가지의 방법이 제공된다.
~~~
SecurityContextHolder.MODE_INHERITABLETHREADLOCAL  # 스레드당 SecurityContext 객체 할당 (default)
SecurityContextHolder.MODE_THREADLOCAL  # 메인과 자식 스레드에 관하여 동일한 SecurityContext 유지
~~~

<br>
<br>

### ▪️SecurityContext
Authentication 보관 및 Authentication 객체 가져오기 역할
~~~
SecurityContextHolder.getContext().setAuthentication(authentication);
SecurityContextHolder.getContext().getAuthentication(authentication);
~~~

<br>
<br>

### ▪️Authentication
현재 접근하는 주체의 정보와 권한을 담는 인터페이스, Authentication 객체는 Security Context에 저장되며 <br>
SecurityContextHolder를 통해 SecurityContext에 접근하고, SecurityContext를 통해 Authentication에 접근할 수 있다. <br>
~~~ java
public interface Authentication extends Principal, Serialzable {
    // 현재 사용자의 권한(ROLE_ADMIN 등) 목록을 가져옴
    Collection<? extends GrantedAuthority> getAuthorities();
    
    // Principal(현재 인증된 사용자) 객체
    Object getPrincipal();
    
    // credentials(자격, 증명) 객체
    Object getCredentials();
    
    Object getDetails();
    
    // 인증 여부
    boolean isAuthenticated();
    
    // 인증 여부를 설정 (강제 설정 가능)
    void setAuthenticated(boolean isAuthenticated) thrwos IllegalArgumentException;
 }
~~~

<br>
<br>

### ▪️UsernamePasswordAuthenticationToken
Authentication을 implements(구현)한 AbstractAuthenticationToken의 하위 클래스 <br>
사용자 아이디가 Principal 역할을, 비밀번호가 Credential 역할을 한다. <br>
첫 번째 생성자는 인증 전의 객체를, 두 번째 생성자는 인증 완료 객체를 생성한다. <br>
~~~ java
public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {

    private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

    // 사용자의 Id
    private final Object principal;
    // 사용자의 Password
    private Object credentials;
    
    // 인증 전 객체 생성
    public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
        super(null);
        this.principal = principal;
        this.credentials = credentials;
        setAuthenticated(false); // 인증 전이기에 false
    }
    
    // 인증 완료 객체 생성
    public UsernamePasswordAuthenticationToken(
    	Object principal,
        Object credentials,
        Collection<? extends GrantedAuthority> authorities
    ) {
        super(authorities);
        this.principal = principal;
        this.credentials = credentials;
        setAuthenticated(true);
    }
}

public abstract class AbstractAuthentiacationToken implements Authentication, CredentialsContainer {
}
~~~

<br>
<br>

### ▪️AuthenticationManager
인증에 대한 부분 처리를 담당한다. 실질적으로는 AuthenticationManager에 등록된 AuthenticationProvider에 의해 인증이 처리된다. <br>
인증 성공 시 UsernamePasswordAuthenticationToken의 두 번째 생성자를 이용해 인증 완료 객체를 생성하여 Security Context에 저장한다. <br>
더불어 인증 상태 유지를 위해 세션에 보관하며 인증 실패 시 AuthenticationException을 발생시킨다.
~~~ java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
~~~

<br>
<Br>

### ▪️AuthenticationProvider
실질적인 인증을 처리한다. 인증 전의 객체를 전달받아 인증 완료 객체를 반환하는 역할을 한다. <br>
인터페이스를 구현해서 custom AuthenticationProvider를 작성하고 AuthenticationManager에 등록하면 된다. <br>
~~~ java
public interface AuthenticationProvider {
    // 인증 전의 Authentication 객체를 받아서 인증된 Authentication 객체를 반환
    Authentication authenticate(Authentication var1) throws AuthenticationException;

    // 지원하는 Authentication 타입인지 확인인
    boolean supports(Class<?> var1);
}
~~~

<br>
<br>

### ▪️UserDetails
인증에 성공하여 생성된 객체, Authentication 객체를 구현한 UsernamePasswordAuthenticationToken을 생성하기 위해 사용된다. <br>
``` java
public interface UserDetails extends Serializable {

    Collection<? extends GrantedAuthority> getAuthorities();

    String getPassword();
    String getUsername();

    boolean isAccountNonExpired();
    boolean isAccountNonLocked();

    boolean isCredentialsNonExpired();
    boolean isEnabled();
    
}
```

<br>
<br>

### ▪️UserDetailService
UserDetails 객체를 반환하는 하나의 메소드만을 가진다. <br>
일반적으로 UserDetailService 클래스 내부에 UserRepository을 주입 받아 DB와 연결하여 처리한다. <br>
``` java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String var1) throws UsernameNotFoundException;
}
```

<br>
<br>

### ▪️PasswordEncoder
AuthenticationManagerBuilder.userDetailsService().passwordEncoder()를 통해 패스워드 암호화에 사용될 PasswordEncoder 구현체를 지정한다.
``` java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userDetialsService).passwordEncoder(passwordEncoder()); // 사용자 정보 조회하여 비밀번호 비교
}

@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(); // 비밀번호 암호화
}
```

<br>
<br>

### ▪️GrantedAuthority
현재 사용자가 가지고 있는 권한이다. ROLE_*의 형태로 사용하며 보통은 'roles' 라고 칭한다. <br>
객체는 UserDetailsService에 의해 불러올 수 있고, 특정 자원에 대한 권한 유무를 검사하여 접근 허용 여부를 결정한다. <br>

<br>
<br>
<br>
<br>

## 📌 스프링 시큐리티 사용하기
### 1. 의존성 추가
build.gradle 파일에 spring security를 추가한다. 

``` gradle
implementation 'org.springframework.boot:spring-boot-starter-security'
```

<br>
<br>

### 2. 시큐리티 활성화용 SecurityConfig 추가
<img src="https://github.com/solji622/LevelUp-Study/blob/01750cfec9aed37002f2a8e0355fd6d4f63743b3/25.03/Spring%20Security/asset/security_login_page.png" width="70%"/>
기본적으로 사용을 하게 되면 모든 페이지에 대해서 로그인을 하도록 되어있기에 매우 번거로워진다. <br>
이때 Config 파일을 통해서 모든 사용자의 접속을 허용시킬 수 있다. <br>

``` java
package com.example.demo;

// import 생략

@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true) // 메서드 수준에서 보안 설정 가능
public class SecurityConfig {
	@Bean
	SecurityFilterChain filterChain(HttpSecurity http) throws Exception { // 시큐리티의 필터 체인 설정
		http
			.authorizeHttpRequests((authorizeHttpRequests) -> authorizeHttpRequests
			.requestMatchers(new AntPathRequestMatcher("/**")).permitAll())
			.csrf((csrf) -> csrf.ignoringRequestMatchers // CSRF 보안 검사 제외
					(new AntPathRequestMatcher("/h2-console/**")))
			.headers((headers) -> headers
					.addHeaderWriter(new XFrameOptionsHeaderWriter(
							XFrameOptionsHeaderWriter.XFrameOptionsMode.SAMEORIGIN))) // 클릭재킹 공격 방지(같은 도메인에서만 접근 가능)
			.formLogin((formLogin) -> formLogin
					.loginPage("/user/login") 
					.defaultSuccessUrl("/")) // 로그인 성공 시 페이지 리다이렉트
			.logout((logout) -> logout
				.logoutRequestMatcher(new AntPathRequestMatcher("/user/logout"))
				.logoutSuccessUrl("/")
				.invalidateHttpSession(true)) // 로그아웃 시 세션 무효화
			;
		
		return http.build();
	}
	
	// 비밀번호 암호화
	@Bean
	PasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}

	// 사용자 인증 관리
	@Bean
	AuthenticationManager authenticationManager(AuthenticationConfiguration
	authenticationConfiguration) throws Exception {
		return authenticationConfiguration.getAuthenticationManager();
	}
}
```
**`.requestMatchers(new AntPathRequestMatcher("/**")).permitAll()`** 이 코드를 통해 <br>
모든 경로 (/**)에 대해 접근이 허용되었다. 즉, 로그인 필요 없이 모든 페이지에 접근이 가능해진다. <br>


<br>
<br>
<br>
<br>


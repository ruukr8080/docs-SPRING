

## D I

```null
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'com.auth0:java-jwt:4.4.0'
```

---
## Securityconfig class 생성

Securityconfig class 위에 @Configuration , **@EnableWebSecurity**를 박아준다.

#### @EnableWebSecurity
- 스프링 시큐리티 설정 활성화.
- Spring FilterChain에 Security FilterChain이 등록됨.

## SecurityFilterChain method 작성

```java
@Bean //  
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {  
    http.csrf(CsrfConfigurer::disable);  
    http.authorizeHttpRequests(authorize -> authorize  
  
            .anyRequest().authenticated()  
    )  
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))  
            .formLogin(FormLoginConfigurer::disable);
              
return http.build();
```
###  BCryptPasswordEncoder 빈 생성
```java
@Bean //비번  
public BCryptPasswordEncoder encodePWD(){  
    return new BCryptPasswordEncoder();  
}
```





### FormLogin 설정 추가.

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
	...
    http.formLogin((form) ->
    	form
        	.loginPage("/loginForm")
            .permitAll()
            .loginProcessingUrl("/login")
            .defaultSuccessUrl("/")
	);
    ...
}
```


---

#### fillterChain method
-  `SecurityFilterChain` 타입의 빈을 생성하는 메서드.
- `HttpSecurity` 객체를 파라미터로 받아, HTTP 보안 설정을 커스터마이징 한다.
    - `http.csrf(CsrfConfigurer:disable)` : CSRF(크로스 사이트 요청 위조) 보호 기능을 비활성화. 테스트나 내부 시스템에서 CSRF 공격 위험이 낮을 때 사용.
    - `http.authorizeHttpRequests(authorize -> ...)`  
        HTTP 요청에 대한 접근 제어 설정. 람다 표현식 내에서 세부적인 접근 제어 규칙 정의.
    - `requestMatchers("/user/**")`  
        "/user/**" 패턴의 URL에 대한 접근은 인증된 사용자에게만 허용함.
    - `requestMatchers("/manager/**").hasAnyRole("ADMIN", "MANAGER")`  
        "/manager/**" 패턴의 URL에 대한 접근은 "ADMIN", "MANAGER" 역할을 가진 사람에게만 허용함
    - `.anyRequest().permitAll()` : 위 조건에 해당하지 않는 요청은 제한 없이 허용됨.
- `return http.build()` : 설정한 HTTP 보안 설정을 바탕으로 `SecurityFilterChain` 객체를 생성하여 반환.

이렇게 해주면 `/user`, `/manager`, `/admin` 주소로 입력 시 권한이 없으면 403 오류를 띄운다.

---
- `http.formLogin((form) ->` 폼 로그인을 사용하며, 로그인 페이지와 관련된 설정을 한다.
    - `.loginpage("/loginForm")` : 사용자 정의 로그인 페이지의 경로를 해당 경로로 설정한다.
    - `permitAll()` : 모든 사용자가 로그인 페이지에 접속하게 한다.
    - `.loginProcessingUrl("/login")` : 시큐리티가 해당 주소를 낚아채서 대신 로그인을 진행함.
    - `defaultSuccessUrl("/")` : 로그인 완료 시 리턴 주소

사용자가 로그인을 진행 시 `.loginProcessingUrl()` 메소드가 동작하여 시큐리티가 로그인을 진행해준다.

## Security 동작 흐름

Start!➡️ 로그인 요청 
➡️ AuthenticationFilter 
➡️ AuthenticationManager  
➡️ AuthenticaitonProvider
➡️ UserDetailsService 도착

- UserDetailsService를 구현한 ProviderService 생성 후 아래 구현 메소드 생성.

```java
@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    User user = userRepository.findByUsername(username);
    // 유저가 있으면 시큐리티 session안에 들어갈 Authentication 인증 객체 안에 들어감.
    // Security Session <- Authentication <- PrincipalDetails(UserDetail)
    if(user != null) {
      return new PrincipalDetails(user); //User 타입을, UserDetails를 상속받는 PrincipalDetailsService 객체로 변환하여 리턴
    }
    return null;
}
```



# 결론

Security는 자기만의 세션을 갖고 있음.
그 세션의 객체 타입이 `Authentication` 임
그렇기 때문에 `User객체` 가 요청 들어와서 `Authenticaiton` 로 변환됨.

`User` 객체➡️ `UserDetails` ➡️`Authenticaiton`



---


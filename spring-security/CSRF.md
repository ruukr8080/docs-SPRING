cross-site Request forgey
![[Pasted image 20240830173026.png]]
요청을 위조하여 사용자의 권한을 이용해 서버에 대한 악성공격을 하는 것

사용자는 로그인 후 어플리케이션에서 인증 토큰을 받는다.  
만약 사용자가 공격자의 사이트를 방문하면, 공격자는 사용자의 토큰을 이용하여 서버에 요청을 보낼 수 있고, 사용자가 원치 않은 작업들을 수행하게 되버린다.
(악성 게시글 작성등.. )

Spring Security에서는 이러한 CSRF에 의한 공격을 방지하기 위해서 Form형식의 전송들에대해서는 `csrf` 라는 hidden 타입으로 값을 설정하고 이값과 서버에서 받는 값이 일치해야 요청 전송을 허가한다.  
하지만 Server가 REST API 용도로만 사용된다면 다음과 같은 설정으로 사용하지 않을 수도 있다.

```java
@EnableWebSecurity
@Configuration
public class WebSecurityConfig extends
   WebSecurityConfigurerAdapter {

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http
      .csrf().disable();
  }
}
```
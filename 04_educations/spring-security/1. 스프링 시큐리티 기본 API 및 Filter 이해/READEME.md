
## 사용자 정의 보안 기능 구현


## Form Login 인증


## Form Login 인증 필터 : UsernamePasswordAuthenticationFilter

- 사용자가 로그인 요청을 한다.
- 요청 정보가 매칭되는지 확인한다.
  - `AntPathRequestMatcher` 에서 `http.formLogin().loginProcessingUrl("/login")` 일치하는지 확인한다.
- 일치하면 사용자가 입력한 로그인 정보를(`Username + Password`) 이용해서 `Authentication` 객체를 생성한다.
- `AuthenticationManager` 를 통해서 인증 절차를 진행한다.
  - 내부적으로 `AuthenticationProvder` 에게 인증을 위임한다.
  - 인증 실패 시 `AuthenticationException` 을 발생하고 `UsernamePasswordAuthenticationFilter` 에서 처리한다.
  - 인증 성공 시 `Authenctication` 객체를 생성하여, 인증의 성공한 결과(`사용자 정보 및 권한 정보`)를 저장하고 `AuthenticationProvder` 에게 반환한다.
- 최종적인 인증 객체를 `Authentication` 를 반환한다.
  - 최종적으로 성공한 인증 정보의 사용자 정보 및 권한 정보를 가지고 있다.
- 인증 객체를 `SecurityContext` 에 저장한다.
  - 인증 객체를 저장하는 저장소이다.
  - `Session` 에도 저장된다.
- 인증 성공한 이후에 `SuccessHandler` 가 동작한다.

### 핵심 클래스 & 메소드 살펴보기

`SecurityContext` 안에 저장이 되면 어느 곳에서든지 아래 메소드를 이용해서 인증 객체를 확인할 수 있다.ㄴ

```java
SecurityContextHolder.getContext().getAuthentication();
```

Spring Security 가 초기화 되면서 설정되는 Filter 목록

- FilterChainProxy.java

```java
nextFilter.doFilter(request, response, this);
```

## Logout 처리, LogoutFilter

- 로그아웃 기본 URL : `/logout`
- 프로세스
  - 사용자가 로그아웃을 요청한다.
  - 세션 무효화, 인증토큰 삭제, 쿠키정보 삭제, 로그인 페이지로 리다이렉트
- Spring Security 기능 설정
  - 로그아웃 처리
  - 로그아웃 처리 URL
  - 로그아웃 성공 후 이동페이지
  - 로그아웃 후 쿠키 삭제
  - 로그아웃 핸들러
  - 로그아웃 성공 후 핸들러
- 샘플 소스

  ```java
  http.logout()                                     // 로그아웃 처리
    .logoutUrl("/logout")                           // 로그아웃 처리 URL
    .logoutSuccessUrl("/login")                     // 로그아웃 성공 후 이동페이지
    .deleteCookies("JSESSIONID", "remember-me")     // 로그아웃 후 쿠키 삭제
    .addLogoutHandler(logoutHandler())              // 로그아웃 핸들러
    .logoutSuccessHandler(logoutSuccessHandler())   // 로그아웃 성공 후 핸들러
  ```

## 인증 API - Logout

- 사용자가 로그아웃을 요청한다.
- `LogoutFilter` 가 받아서 처리한다.
- 요청 정보가 매칭되는지 확인한다.
  - `AntPathRequestMatcher` 에서 `http.logout().logoutUrl("/logout")` 일치하는지 확인한다.
- `SecurityContext` 객체에서 인증 정보(`Authentication`) 를 꺼내온다.
- 꺼내온 정보를 `SecurityContextLogoutHandler` 로 전달한다.
  - 세션 무효화
  - 쿠키 삭제
  - SecurityContextHolder.clearContext(); // 객체 삭제
  - 인증 객체 `NULL` 초기화
- `SecurityContextLogoutHandler` 가 성공적으로 종료가 되면 `LogoutFilter` 가 `SimpleUrlLogoutSuccessHadler` 가 처리한다.

### 핵심 클래스 & 메소드 살펴보기


## Remember Me 인증 필터 : RememberMeAuthenticationFilter

- 세션이 만료되고 웹 브라우저가 종료된 후에도 어플리케이션이 사용자를 기억하는 기능
- Remember-Me 쿠키에 대한 Http 요청을 확인한 후 토큰 기반 인증을 사용해 유효성을 검사하고 토큰이 검증되면 사용자는 로그인 한다.
- 사용자 라이프 사이클
  - 인증 성공(Remember-Me 쿠키 설정)
  - 인증 실패(쿠키가 존재하면 쿠키 무효화)
  - 로그아웃(쿠키가 존재하면 쿠키 무효화)
- 샘플 소스

  ```java 
  http.rememberMe()                                 // rememberMe 기능이 작동함
    .rememberMeParameter("remember")                // 기본 파라미터명은 remember-me
    .tokenValiditySeconds("3600")                   // Default 는 14일
    .alwaysRemember(true)                           // 리멤버 미 기능이 활성화되지 않아도 항상 실핼
    .userDetailsService(userDetailsService)         // 사용자 조회
  ```

### Remember Me 실습

[editThisCookie](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg/related?hl=ko) 를 활용해서 `Remember-Me` 테스트 진행

#### Remember Me 선택하지 않고 테스트

- `Remember-Me` 를 선택하지 않고 아이디와 패스워드 입력 후 로그인
- `editThisCookie` 를 활용하여 로그인 후 `JSESSIONID` 확인
- `editThisCookie` 를 활용하여 `JSESSIONID` 삭제
- 해당 URL 새로고침 시 로그인 화면으로 이동

#### Remember Me 선택하고 테스트

- `Remember-Me` 를 선택하고 아이디와 패스워드 입력 후 로그인
- `editThisCookie` 를 활용하여 로그인 후 `JSESSIONID` 확인
- `editThisCookie` 를 활용하여 로그인 후 `remember-me` 확인
- `editThisCookie` 를 활용하여 `JSESSIONID` 삭제
- 해당 URL 새로고침 시 로그인 화면으로 이동하지 않고 그대로 유지
- `editThisCookie` 를 활용하여 `JSESSIONID` 확인

`remember-me` 쿠키를 가지고 있다면 이 값을 다시 추출한 UserId와 Password를 통해서 다시 인증을 시도합니다.

## Remember Me 인증

필터가 사용자 요청을 처리하는 요건

- Authentication == null 인 경우
  - 인증 객체가 Security Context에 저장
    - 인증을 받은 사용자는 Security Context에 인증 객체가 존재
  - Security Context 안에 Authentication 존재하지 않는 경우
    - 사용자 세션이 만료된 경우
    - 사용자 세션이 끊긴 경우

Remember Me 작동 방식

- RememberMeAuthenticationFilter 작동
  - 세션이 만료되었으나 Form 인증을 받기전 Remember Me 기능을 활성화
- RememberMeServices 의 구현체
  - TokenBasedRememberMeServices
    - 메모리에서 저장된 토큰과 사용자의 요청으로 온 쿠키를 비교하여 인증 처리
    - 14일 만료기간
  - PersistentTokenBasedRememberMeservices
    - 토큰 영속화
    - 서비스에서 발급한 토큰을 데이터베이스에 저장
    - 저장된 토큰과 클라이언트에서 발급한 토큰과 비교
- Token Cookie 추출
- Token 이 존재하는가
  - 존재하지 않으면 그 다음 필터로 이동 (chain.doFilter)
- Decode Token
  - 현재 토큰의 형식이 정상적으로 규칙을 지키고 있는지 확인
- Token 이 서로 일치하는가?
  - 서버에 저장된 토큰과 사용자가 요청한 토큰 비교
  - 토큰이 일치하지 않을 경우 Exception
- User 계정이 존재하는가?
  - 서버에 저장된 사용자를 조회
  - 존재하지 않을 경우 Exception
- 새로운 Authentication 생성
  - 인증 처리를 위해 AuthenticationManager 

## 인증 API - AnonymousAuthenticationFilter

- 익명 사용자가 요청한다.
- AnonymousAuthenticationFilter 에서 처리한다.
  - 현재 요청을 하고 있는 사용자가 인증 객체가 존재하는지 체크한다.
    - 인증 객체는 SecurityContext에 저장되어 있음
  - 존재하지 않는다면 인증을 받지 않은 사용자로 판단
- 익명 사용자로 판단하여 인증 객체 생성
   - Anonymous AuthenticationToken 용 인증 객체 생성

### 특징

- 익명사용자 인증 처리 필터
- 익명사용자와 인증 사용자를 구분해서 처리하기 위한 용도로 사용
- 화면에서 인증 여부를 구현할 때 isAnonymous() 와 isAuthenticated() 로 구분해서 사용
  - isAnonymous() : 로그인 용도
  - isAuthenticated() : 로그아웃 용도
- 인증객체를 세션에 저장하지 않는다.

AnonymousAuthenticationFilter

- doFilter
- createAuthentication

AbstractSecurityInterceptor

- beforeInvocation

## 인증 API - 동시 세션 제어 / 세션 고정 보호 / 세션 정책

### 인증 API - 동시 세션 제어

- 현재 동일한 계정으로 인증을 받을 때, 생성되는 세션의 대한 허용개수가 초괴되었을 경우 세션을 초과하지 않고 유지하는 방법
  - 이전 사용자 세션 만료
  - 현재 사용자 인증 실패

```java
http.sessionManagement()            // 세션 관리 기능이 작동함
  .maximumSessions(1)               // 최대 허용 가능 세션 개수, -1 : 무제한 로그인 세션 허용
  .maxSessionsPreventsLogin(true)   // 동시 로그인 차단함, false : 기존 세션 만료(Default)
  .invalidSessionUrl("/invalid")    // 세션이 유효하지 않을 때 이동 할 페이지
  .expiredUrl("/expired")           // 세션이 만료된 경우 이동 할 페이지
```

### 인증 API - 세션 고정 보호

- 사용자, 공격자, WebApplication
- 공격자가 서버에 접속하여서 JSessionID를 발급받는다.
- 공격자는 사용자에게 자신이 발급받은 JSessionID를 심어놓는다.
- 사용자는 공격자가 심어놓은 JSessionID를 갖고 있는다.
- 사용자는 공격자가 심어놓은 JSessionID로 로그인을 시도하고 성공한다.
- 공격자 쿠키 값으로 인증되어 있기 때문에 공격자는 사용자 정보를 공유하게 된다.

```java
http.sessionManagement()                  // 세션 관리 기능이 작동함
  .sessionFixation().changeSessionId()    // 기본값, none, migrateSession, newSession
```

### 인증 API - 세션 정책

```java
http.sessionManagement()                  // 세션 관리 기능이 작동함
  .sessionCreationPolicy(SessionCreationPolicy.If_Required)
```

- SessionCreationPolicy.Always : 스프링 시큐리티가 항상 세션 생성
- SessionCreationPolicy.If_Required : 스프링 시큐리티가 필요 시 생성(기본값)
- SessionCreationPolicy.Never : 스프링 시큐리티가 생성하지 않지만 이미 존재하면 사용
- SessionCreationPolicy.Stateless : 스프링 시큐리티가 생성하지 않고 존재해도 사용하지 않음
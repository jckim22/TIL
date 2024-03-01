[개발자 유미](https://www.youtube.com/watch?v=ZTaZOCqTez4&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ) 님의 강의를 듣고 [JWT란 무엇인가?](https://velog.io/@hahan/JWT%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80)와 [[SpringBoot] Spring Security란?](https://mangkyu.tistory.com/76)을 참고해 정리한 내용입니다.

# Spring Security


Spring Security는 Spring 기반의 애플리케이션의 보안(인증과 권한, 인가 등)을 담당하는 스프링 하위 프레임워크이다. Spring Security는 '인증'과 '권한'에 대한 부분을 Filter 흐름에 따라 처리하고 있다. Filter는 Dispatcher Servlet으로 가기 전에 적용되므로 가장 먼저 URL 요청을 받지만, Interceptor는 Dispatcher와 Controller사이에 위치한다는 점에서 적용 시기의 차이가 있다. Spring Security는 보안과 관련해서 체계적으로 많은 옵션을 제공해주기 때문에 개발자 입장에서는 일일이 보안관련 로직을 작성하지 않아도 된다는 장점이 있다.

# JWT(Json Web Token)
>정보를 비밀리에 전달하거나 인증할 때 주로 사용하는 토큰으로, Json객체를 이용함

JWT는 Json Web Token의 약자로 일반적으로 클라이언트와 서버 사이에서 통신할 때 권한을 위해 사용하는 토큰이다. 웹 상에서 정보를 Json형태로 주고 받기 위해 표준규약에 따라 생성한 암호화된 토큰으로 복잡하고 읽을 수 없는 string 형태로 저장되어있다.



![](https://velog.velcdn.com/images/jckim22/post/b367b2eb-a7d5-41d7-9474-d214d00d6e47/image.png)
- 헤더 (Header)
어떠한 알고리즘으로 암호화 할 것인지, 어떠한 토큰을 사용할 것 인지에 대한 정보가 들어있다.

- 정보 (Payload)
전달하려는 정보(사용자 id나 다른 데이터들, 이것들을 클레임이라고 부른다)가 들어있다.
payload에 있는 내용은 수정이 가능하여 더 많은 정보를 추가할 수 있다. 그러나 노출과 수정이 가능한 지점이기 때문에 인증이 필요한 최소한의 정보(아이디, 비밀번호 등 개인정보가 아닌 이 토큰을 가졌을 때 권한의 범위나 토큰의 발급일과 만료일자 등)만을 담아야한다.

- 서명 (Signature)
가장 중요한 부분으로 헤더와 정보를 합친 후 발급해준 서버가 지정한 secret key로 암호화 시켜 토큰을 변조하기 어렵게 만들어준다.
한가지 예를 들어보자면 토큰이 발급된 후 누군가가 Payload의 정보를 수정하면 Payload에는 다른 누군가가 조작된 정보가 들어가 있지만 Signatute에는 수정되기 전의 Payload 내용을 기반으로 이미 암호화 되어있는 결과가 저장되어 있기 때문에 조작되어있는 Payload와는 다른 결과값이 나오게 된다.
이러한 방식으로 비교하면 서버는 토큰이 조작되었는지 아닌지를 쉽게 알 수 있고, 다른 누군가는 조작된 토큰을 악용하기가 어려워진다👍


![](https://velog.velcdn.com/images/jckim22/post/215b9b1d-784a-434e-84cf-f42e37402725/image.png)


1. 사용자가 id와 password를 입력하여 로그인 요청을 한다.

2. 서버는 회원DB에 들어가 있는 사용자인지 확인을 한다.

3. 확인이 되면 서버는 로그인 요청 확인 후, secret key를 통해 토큰을 발급한다.

4. 이것을 클라이언트에 전달한다.

5. 서비스 요청과 권한을 확인하기 위해서 헤더에 데이터(JWT) 요청을 한다.

6. 데이터를 확인하고 JWT에서 사용자 정보를 확인한다.

7. 클라이언트 요청에 대한 응답과 요청한 데이터를 전달해준다.


이와 같이 토큰 기반 인증방식은 사용자의 인증이 완료된 이후에 토큰을 발급한다. 클라이언트쪽에서는 전달받은 토큰을 저장해두고 서버에 요청을 할 때마다 해당 토큰을 서버에 함께 전달한다. 그 이후 서버는 토큰을 검증하고 응답하는 방식으로 작동한다.


# 목표

- 스프링 시큐리티 6 프레임워크를 활용하여 JWT 기반의 인증/인가를 구현하고 회원 정보 저장(영속성) h2 데이터베이스를 활용한다.

- 서버는 API 서버 형태로 구축한다.

- 토큰 발급의 경우 단일 토큰으로 진행한다.
>(Access, Refresh로 나누는 경우도 있지만 기본 강의라 간단하게 한개로 진행한다.

## JWT 의존성 추가
#### 0.12.3
```
dependencies {

    implementation 'io.jsonwebtoken:jjwt-api:0.12.3'
    implementation 'io.jsonwebtoken:jjwt-impl:0.12.3'
    implementation 'io.jsonwebtoken:jjwt-jackson:0.12.3'
}
```

#### 0.11.5
```
dependencies {

    implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
    implementation 'io.jsonwebtoken:jjwt-impl:0.11.5'
    implementation 'io.jsonwebtoken:jjwt-jackson:0.11.5'
}


```

## 기본 Controller 생성
#### MainController
```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@ResponseBody
public class MainController {

    @GetMapping("/")
    public String mainP() {

        return "main Controller";
    }
}
```
#### AdminController
```java

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@ResponseBody
public class AdminController {

    @GetMapping("/admin")
    public String adminP() {

        return "admin Controller";
    }
}
```

## SecurityConfig 클래스 설명

스프링 시큐리티의 인가 및 설정을 담당하는 클래스이다. Security Config 구현은 스프링 시큐리티의 세부 버전별로 많이 상이한다. (이 프로젝트는 스프링 시큐리티 6.2.1 버전으로 구현한다)

자주 접할 수 있는 버전에 대한 구현 차이는 아래 영상을 통해 확인할 수 있다.

- **스프링 시큐리티 시리즈 : 버전별 Security Config 구현 방법**
>https://youtu.be/NdRVhOccuOs?si=FufI38WMhTs3TVr8

## Security Config 클래스 기본 요소 작성

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    //비밀번호를 해싱
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {

        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {



        //csrf disable
        http
                .csrf((auth) -> auth.disable());


        //Form 로그인 방식 disable
        http
                .formLogin((auth) -> auth.disable());

        //http basic 인증 방식 disable
        http
                .httpBasic((auth) -> auth.disable());

        //경로별 인가 작업
        //로그인이나 회원가입은 인가 x
        //admin은 ADMIN만 가능
        //다른 요청에 대해서는 로그인이 되어야만 가능
        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
                        .requestMatchers("/admin").hasRole("ADMIN")
                        .anyRequest().authenticated());

        //세션 설정
        //JWT에서는 Session을 무상태성으로 관리
        http
                .sessionManagement((session) -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}

```

PostMan으로 테스트 해보면 이 config를 하기 전까지는 아무 응답이 오지 않았을 것이다.
왜냐면 permit이 되어있지 않으니까.

하지만 이제 /login, /, /join 경로에 permitAll을 줬기 때문에 응답이 올 것이다.

![](https://velog.velcdn.com/images/jckim22/post/c0902a65-8cfd-4069-bca7-9709e2d56b30/image.png)

하지만 아래처럼 /admin 경로는 ADMIN이라는 권한이 없기 때문에 아무 응답을 받지 못한 것을 볼 수 있다.
![](https://velog.velcdn.com/images/jckim22/post/5e986277-bac4-47e7-8934-d55d22bb61fd/image.png)


## DB연결 및 Entity 작성

회원 정보를 저장하기 위한 데이터베이스는 h2 데이터베이스를 사용한다. 그리고 접근은 Spring Data JPA를 사용한다.


### 회원 테이블 Entity 작성 : UserEntity

- **UserEntity**

```java
@Entity
@Setter
@Getter
public class UserEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String username;
    private String password;

    private String role;
}
```

---

### 회원 테이블 Repository 작성 : UserRepository

- **UserRepository**

```java
public interface UserRepository extends JpaRepository<UserEntity, Integer> {

}
```

---

# 회원가입 로직

#### JoinDTO

```java
@Setter
@Getter
public class JoinDTO {

    private String username;
    private String password;
}
```

#### JoinController

```java
@Controller
@ResponseBody
public class JoinController {
    
    private final JoinService joinService;

    public JoinController(JoinService joinService) {
        
        this.joinService = joinService;
    }

    @PostMapping("/join")
    public String joinProcess(JoinDTO joinDTO) {

        System.out.println(joinDTO.getUsername());
        joinService.joinProcess(joinDTO);

        return "ok";
    }
}
```

#### JoinService

```java
@Service
public class JoinService {

    private final UserRepository userRepository;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;

    public JoinService(UserRepository userRepository, BCryptPasswordEncoder bCryptPasswordEncoder) {

        this.userRepository = userRepository;
        this.bCryptPasswordEncoder = bCryptPasswordEncoder;
    }

    public void joinProcess(JoinDTO joinDTO) {

        String username = joinDTO.getUsername();
        String password = joinDTO.getPassword();

        Boolean isExist = userRepository.existsByUsername(username);

        if (isExist) {

            return;
        }

        UserEntity data = new UserEntity();

        data.setUsername(username);
        data.setPassword(bCryptPasswordEncoder.encode(password));
        data.setRole("ROLE_ADMIN");

        userRepository.save(data);
    }
}
```

SecurityConfig에서 스프링 빈으로 주입받은 BCryptPasswordEncoder를 사용해서 패스워드를 해싱한다.
![](https://velog.velcdn.com/images/jckim22/post/39d7d04f-5aeb-4ccb-94a0-77b73ea69fec/image.png)
위 정상처리 응답을 받았고
![](https://velog.velcdn.com/images/jckim22/post/bff3b0ae-e6ed-47da-9d8e-5a0fe7545bfd/image.png)
패스워드도 해쉬값으로 잘 암호화가 되었다.

#### UserRepository

```java
public interface UserRepository extends JpaRepository<UserEntity, Integer> {

    Boolean existsByUsername(String username);
}

```

# 로그인 필터 구현

### 자료

- **스프링 시큐리티 필터 참고 자료**

[Architecture :: Spring Security](https://docs.spring.io/spring-security/reference/servlet/architecture.html)

---

### 로그인 모식도

![](https://velog.velcdn.com/images/jckim22/post/c1b39c71-14fb-48b2-a25a-1630e8275ecc/image.png)

session 방식을 사용할 때는 UsernamePasswordAuthenticationFilter와 AuthenticationManager를 따로 구현하지 않아도 되었지만, JWT 방식을 사용하게 되면 form 방식을 disable로 했기 때문에 커스터마이징을 해주어야 한다.

1. 포스트 요청이 들어오면 UsernamePasswordAuthenticationFilter로 인해 로그인을 진행한다는 것이 AuthenticationManager에게 넘어가서 데이터베이스의 정보와 일치하는지 확인하게 된다.

2. 성공하면 successfulAuth가 실행되고 JWToken이 반환된다.(실패시 401을 반환)

### 스프링 시큐리티 필터 동작 원리

스프링 시큐리티는 클라이언트의 요청이 여러개의 필터를 거쳐 DispatcherServlet(Controller)으로 향하는 중간 필터에서 요청을 가로챈 후 검증(인증/인가)을 진행한다.

- 클라이언트 요청 → 서블릿 필터 → 서블릿 (컨트롤러)

- **Delegating Filter Proxy**
    
    서블릿 컨테이너 (톰캣)에 존재하는 필터 체인에 DelegatingFilter를 등록한 뒤 모든 요청을 가로챈다. (필터 중 하나)
    
    ![](https://velog.velcdn.com/images/jckim22/post/5dd74094-b50c-49d3-bfce-ffea1626445e/image.png)

    
- **서블릿 필터 체인의 DelegatingFilter → Security 필터 체인 (내부 처리 후) → 서블릿 필터 체인의 DelegatingFilter**
    
    ![](https://velog.velcdn.com/images/jckim22/post/ac411719-54bd-4f09-86fe-577575ae2229/image.png)

    
    가로챈 요청은 SecurityFilterChain에서 검증 후 상황에 따른 거부, 리디렉션, 서블릿으로 요청 전달을 진행한다.
    
 정리하면 많은 필터 중 하나인 DelegatingFilter를 등록하고 이로 인해 모든 요청을 가로채게 되고 SecurityFilter에서 처리하게 되는 것이다.




- SecurityFilterChain의 필터 목록과 순서
![](https://velog.velcdn.com/images/jckim22/post/af843a4b-ba47-4094-b2c8-6ff04ec53eb2/image.png)


### Form 로그인 방식에서 UsernamePasswordAuthenticationFilter

이전에도 말했듯이 Form 로그인 방식에서는 클라이언트단이 username과 password를 전송한 뒤 Security 필터를 통과하는데 UsernamePasswordAuthentication 필터에서 회원 검증을 진행을 시작한다. 

(회원 검증의 경우 UsernamePasswordAuthenticationFilter가 호출한 AuthenticationManager를 통해 진행하며 DB에서 조회한 데이터를 UserDetailsService를 통해 받음)

우리의 JWT 프로젝트는 SecurityConfig에서 formLogin 방식을 disable 했기 때문에 기본적으로 활성화 되어 있는 해당 필터는 동작하지 않는다.

따라서 로그인을 진행하기 위해서 필터를 커스텀하여 등록해야 한다.

### 로그인 로직 구현 목표

- 아이디, 비밀번호 검증을 위한 커스텀 필터 작성
- DB에 저장되어 있는 회원 정보를 기반으로 검증할 로직 작성
- 로그인 성공시 JWT를 반환할 success 핸들러 생성
- 커스텀 필터 SecurityConfig에 등록

## 로그인 요청 받기 : 커스텀 UsernamePasswordAuthentication 필터 작성

```java
LoginFilter
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;

    public LoginFilter(AuthenticationManager authenticationManager) {

        this.authenticationManager = authenticationManager;
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {

				//클라이언트 요청에서 username, password 추출
        String username = obtainUsername(request);
        String password = obtainPassword(request);

				//스프링 시큐리티에서 username과 password를 검증하기 위해서는 token에 담아야 함
        UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(username, password, null);

				//token에 담은 검증을 위한 AuthenticationManager로 전달
        return authenticationManager.authenticate(authToken);
    }

		//로그인 성공시 실행하는 메소드 (여기서 JWT를 발급하면 됨)
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) {

    }

		//로그인 실패시 실행하는 메소드
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) {

    }
}
```
AuthenticationManager를 주입 받고 이로 검증을 진행한다.
검증을 할 때는 UsernamePasswordAuthenticationToken라는 DTO에 담아서 보내준다.

검증이 성공하면 successfulAuthentication가 실행되고 실패하면 unsuccessfulAuthentication가 실행된다.


#### SecurityConfig 설정
##### SecurityConfig : 커스텀 로그인 필터 등록

이제 필터를 등록하자(addFilterAt): addFilterAt을 사용해서 UsernamePasswordAuthenticationFilter 자리에 대체한다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {

        return new BCryptPasswordEncoder();
    }
   @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {


        http
                .csrf((auth) -> auth.disable());

        http
                .formLogin((auth) -> auth.disable());

        http
                .httpBasic((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
                        .anyRequest().authenticated());

				//필터 추가 LoginFilter()는 인자를 받음 (AuthenticationManager() 메소드에 authenticationConfiguration 객체를 넣어야 함) 따라서 등록 필요
        http
                .addFilterAt(new LoginFilter(), UsernamePasswordAuthenticationFilter.class);

        http
                .sessionManagement((session) -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}
```

#### SecurityConfig : AuthenticationMananger Bean 등록과 LoginFilter 인수 전달


새로운 커스텀 필터에 authenticationManager를 주입해주는데 그 안에 또 authenticationConfiguration를 주입해주어야 한다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

		//AuthenticationManager가 인자로 받을 AuthenticationConfiguraion 객체 생성자 주입
		private final AuthenticationConfiguration authenticationConfiguration;

    public SecurityConfig(AuthenticationConfiguration authenticationConfiguration) {

        this.authenticationConfiguration = authenticationConfiguration;
    }

		//AuthenticationManager Bean 등록
		@Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration configuration) throws Exception {

        return configuration.getAuthenticationManager();
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {

        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {


        http
                .csrf((auth) -> auth.disable());

        http
                .formLogin((auth) -> auth.disable());

        http
                .httpBasic((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
                        .anyRequest().authenticated());

				//필터 추가 LoginFilter()는 인자를 받음 (AuthenticationManager() 메소드에 authenticationConfiguration 객체를 넣어야 함) 따라서 등록 필요
        http
                .addFilterAt(new LoginFilter(authenticationManager(authenticationConfiguration)), UsernamePasswordAuthenticationFilter.class);

        http
                .sessionManagement((session) -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}
```

#### 로그인 성공시 JWT 반환
로그인 성공시 successfulAuthentication() 메소드를 통해 JWT를 응답해야 한다. 따라서 JWT 응답 구문을 작성해야 하는데 JWT 발급 클래스를 아직 생성하지 않았기 때문에 DB 기반 회원 검증 구현을 진행한 뒤 JWT 발급 및 검증을 진행하는 클래스를 생성해야한다.

## DB기반 로그인 검증 로직
#### 로그인 모식도
![](https://velog.velcdn.com/images/jckim22/post/c1b39c71-14fb-48b2-a25a-1630e8275ecc/image.png)
이제 위처럼 Manager를 통한 검증 로직을 커스텀하여 구현한다.

구현은 UserDetails, UserDetailsService, UserRepository의 회원 조회 메소드를 진행한다.

### UserRepository

- **UserRepository**

```java
public interface UserRepository extends JpaRepository<UserEntity, Integer> {

    Boolean existsByUsername(String username);
		
		//username을 받아 DB 테이블에서 회원을 조회하는 메소드 작성
    UserEntity findByUsername(String username);
}
```

---

### UserDetailsService 커스텀 구현



- **CustomUserDetailsService**

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {

        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
				
				//DB에서 조회
        UserEntity userData = userRepository.findByUsername(username);

        if (userData != null) {
						
						//UserDetails에 담아서 return하면 AutneticationManager가 검증 함
            return new CustomUserDetails(userData);
        }

        return null;
    }
}
```

Repository에서 가져온 값이 null이 아니면 CustomUserDetails를 던져준다.


---

### UserDetails 커스텀 구현

UserDetails는 DTO에 해당한다.

- **CustomUserDetails**

```java
public class CustomUserDetails implements UserDetails {

    private final UserEntity userEntity;

    public CustomUserDetails(UserEntity userEntity) {

        this.userEntity = userEntity;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {

        Collection<GrantedAuthority> collection = new ArrayList<>();

        collection.add(new GrantedAuthority() {

            @Override
            public String getAuthority() {

                return userEntity.getRole();
            }
        });

        return collection;
    }

    @Override
    public String getPassword() {

        return userEntity.getPassword();
    }

    @Override
    public String getUsername() {

        return userEntity.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {

        return true;
    }

    @Override
    public boolean isAccountNonLocked() {

        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {

        return true;
    }

    @Override
    public boolean isEnabled() {

        return true;
    }
}
```
getAuthorities만 사용방법에 맞게 잘 신경써서 구현해주면 된다.
나머지는 메서드는 이름에 맞게 설정하되 일단 true로 바꾸어주자, 계정이 Expired되지도 않았고 Locked되지도 않았기 때문이다.

# JWT 발급과 검증

- 로그인시 → 성공 → JWT 발급
- 접근시 → JWT 검증

JWT에 관해 발급과 검증을 담당할 클래스가 필요하다. 따라서 JWTUtil이라는 클래스를 생성하여 JWT 발급, 검증 메소드를 작성하자.

---

### JWT 생성 원리

[JWT.IO](https://jwt.io/)

JWT는 Header.Payload.Signature 구조로 이루어져 있다. 각 요소는 다음 기능을 수행한다.

- Header
    - JWT임을 명시
    - 사용된 암호화 알고리즘
- Payload
    - 정보
- Signature
    - 암호화알고리즘((BASE64(Header))+(BASE64(Payload)) + 암호화키)

JWT의 특징은 내부 정보를 단순 BASE64 방식으로 인코딩하기 때문에 외부에서 쉽게 디코딩 할 수 있다. 

외부에서 열람해도 되는 정보를 담아야하며, 토큰 자체의 발급처를 확인하기 위해서 사용한다.

(지폐와 같이 외부에서 그 금액을 확인하고 금방 외형을 따라서 만들 수 있지만 발급처에 대한 보장 및 검증은 확실하게 해야하는 경우에 사용한다. 따라서 토큰 내부에 비밀번호와 같은 값 입력 금지)

---

### JWT 암호화 방식

- 암호화 종류
    - 양방향
        - 대칭키 - 이 프로젝트는 양방향 대칭키 방식 사용 : HS256
        - 비대칭키
    - 단방향

---

### 암호화 키 저장

암호화 키는 하드코딩 방식으로 구현 내부에 탑재하는 것을 지양하기 때문에 변수 설정 파일에 저장한다.

- **application.properties**

```
spring.jwt.secret=vmfhaltmskdlstkfkdgodyroqkfwkdbalroqkfwkdbalaaaaaaaaaaaaaaaabbbbb
```

---

### JWTUtil

- 토큰 Payload에 저장될 정보
    - username
    - role
    - 생성일
    - 만료일
- JWTUtil 구현 메소드
    - JWTUtil 생성자
    - username 확인 메소드
    - role 확인 메소드
    - 만료일 확인 메소드

- **JWTUtil : 0.12.3**

```java
@Component
public class JWTUtil {

    private SecretKey secretKey;

    public JWTUtil(@Value("${spring.jwt.secret}")String secret) {

        secretKey = new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), Jwts.SIG.HS256.key().build().getAlgorithm());
    }

    public String getUsername(String token) {

        return Jwts.parser().verifyWith(secretKey).build().parseSignedClaims(token).getPayload().get("username", String.class);
    }

    public String getRole(String token) {

        return Jwts.parser().verifyWith(secretKey).build().parseSignedClaims(token).getPayload().get("role", String.class);
    }

    public Boolean isExpired(String token) {

        return Jwts.parser().verifyWith(secretKey).build().parseSignedClaims(token).getPayload().getExpiration().before(new Date());
    }

    public String createJwt(String username, String role, Long expiredMs) {

        return Jwts.builder()
                .claim("username", username)
                .claim("role", role)
                .issuedAt(new Date(System.currentTimeMillis()))
                .expiration(new Date(System.currentTimeMillis() + expiredMs))
                .signWith(secretKey)
                .compact();
    }
}
```

- **JWTUtil : 0.11.5**

```java
@Component
public class JWTUtil {

    private Key key;

    public JWTUtil(@Value("${spring.jwt.secret}")String secret) {

				byte[] byteSecretKey = Decoders.BASE64.decode(secret);
        key = Keys.hmacShaKeyFor(byteSecretKey);
    }

    public String getUsername(String token) {

        return Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token).getBody().get("username", String.class);
    }

    public String getRole(String token) {

        return Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token).getBody().get("role", String.class);
    }

    public Boolean isExpired(String token) {

        return Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token).getBody().getExpiration().before(new Date());
    }

    public String createJwt(String username, String role, Long expiredMs) {

				Claims claims = Jwts.claims();
        claims.put("username", username);
        claims.put("role", role);

        return Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + expiredMs))
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();
    }
}
```

---

---

### 로그인 성공

로그인 로직, JWTUtil 클래스를 생성했다. 이제 로그인이 성공 했을 경우 JWT를 발급하기 위한 구현을 진행해보자.

LoginFilter는 로그인처리, 검증을 하는 것이고 검증이 성공했다면 LoginFilter에서 JWTUtil을 주입 받아 JWT를 발급할 것이다.

---

### **JWTUtil 주입**

- **LoginFilter : JWTUtil 주입**

```java
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;
		//JWTUtil 주입
		private final JWTUtil jwtUtil;

    public LoginFilter(AuthenticationManager authenticationManager, JWTUtil jwtUtil) {

        this.authenticationManager = authenticationManager;
				this.jwtUtil = jwtUtil;
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {

				//클라이언트 요청에서 username, password 추출
        String username = obtainUsername(request);
        String password = obtainPassword(request);

				//스프링 시큐리티에서 username과 password를 검증하기 위해서는 token에 담아야 함
        UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(username, password, null);

				//token에 담은 검증을 위한 AuthenticationManager로 전달
        return authenticationManager.authenticate(authToken);
    }

		//로그인 성공시 실행하는 메소드 (여기서 JWT를 발급하면 됨)
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) {

    }

		//로그인 실패시 실행하는 메소드
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) {

    }
}
```

- **SecurityConfig에서 Filter에 JWTUtil 주입**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

		private final AuthenticationConfiguration authenticationConfiguration;
		//JWTUtil 주입
		private final JWTUtil jwtUtil;

    public SecurityConfig(AuthenticationConfiguration authenticationConfiguration, JWTUtil jwtUtil) {

        this.authenticationConfiguration = authenticationConfiguration;
				this.jwtUtil = jwtUtil;
    }

		@Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration configuration) throws Exception {

        return configuration.getAuthenticationManager();
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {

        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http
                .csrf((auth) -> auth.disable());

        http
                .formLogin((auth) -> auth.disable());

        http
                .httpBasic((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
                        .anyRequest().authenticated());

				//AuthenticationManager()와 JWTUtil 인수 전달
        http
                .addFilterAt(new LoginFilter(authenticationManager(authenticationConfiguration)), UsernamePasswordAuthenticationFilter.class);

        http
                .sessionManagement((session) -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}
```

---

### LoginFilter 로그인 성공 successfulAuthentication 메소드 구현

- **LoginFilter**

```java
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) {
				
				//UserDetailsS
        CustomUserDetails customUserDetails = (CustomUserDetails) authentication.getPrincipal();

        String username = customUserDetails.getUsername();

        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
        Iterator<? extends GrantedAuthority> iterator = authorities.iterator();
        GrantedAuthority auth = iterator.next();

        String role = auth.getAuthority();

        String token = jwtUtil.createJwt(username, role, 60*60*10L);

        response.addHeader("Authorization", "Bearer " + token);
    }
}
```

HTTP 인증 방식은 RFC 7235 정의에 따라 아래 인증 헤더 형태를 가져야 한다.

```
Authorization: 타입 인증토큰

//예시
Authorization: Bearer 인증토큰string
```

---

### LoginFilter 로그인 실패 unsuccessfulAuthentication 메소드 구현

- **LoginFilter**

```java
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;
    private final JWTUtil jwtUtil;

    public LoginFilter(AuthenticationManager authenticationManager, JWTUtil jwtUtil) {

        this.authenticationManager = authenticationManager;
        this.jwtUtil = jwtUtil;
    }

    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) {
				
				//로그인 실패시 401 응답 코드 반환
        response.setStatus(401);
    }
}
```

---

### 발급 테스트

/login 경로로 username과 password를 포함한 POST 요청을 보낸 후 응답 헤더에서 Authorization 키에 담긴 JWT를 확인한다.



### LoginFilter 최종

```java
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;
    private final JWTUtil jwtUtil;

    public LoginFilter(AuthenticationManager authenticationManager, JWTUtil jwtUtil) {

        this.authenticationManager = authenticationManager;
        this.jwtUtil = jwtUtil;
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {

        String username = obtainUsername(request);
        String password = obtainPassword(request);

        UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(username, password, null);

        return authenticationManager.authenticate(authToken);
    }

    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) {

        CustomUserDetails customUserDetails = (CustomUserDetails) authentication.getPrincipal();

        String username = customUserDetails.getUsername();

        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
        Iterator<? extends GrantedAuthority> iterator = authorities.iterator();
        GrantedAuthority auth = iterator.next();

        String role = auth.getAuthority();

        String token = jwtUtil.createJwt(username, role, 60*60*10L);

        response.addHeader("Authorization", "Bearer " + token);
    }

    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) {

        response.setStatus(401);
    }
}
```

### JWT 검증 필터

스프링 시큐리티 filter chain에 요청에 담긴 JWT를 검증하기 위한 커스텀 필터를 등록해야 한다.

해당 필터를 통해 요청 헤더 Authorization 키에 JWT가 존재하는 경우 JWT를 검증하고 강제로SecurityContextHolder에 세션을 생성한다. (이 세션은 STATLESS 상태로 관리되기 때문에 해당 요청이 끝나면 소멸 된다.)

---

### JWTFilter 구현

- **JWTFilter**

```java
public class JWTFilter extends OncePerRequestFilter {

    private final JWTUtil jwtUtil;

    public JWTFilter(JWTUtil jwtUtil) {

        this.jwtUtil = jwtUtil;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
				
				//request에서 Authorization 헤더를 찾음
        String authorization= request.getHeader("Authorization");
				
				//Authorization 헤더 검증
        if (authorization == null || !authorization.startsWith("Bearer ")) {

            System.out.println("token null");
            filterChain.doFilter(request, response);
						
						//조건이 해당되면 메소드 종료 (필수)
            return;
        }
			
        System.out.println("authorization now");
				//Bearer 부분 제거 후 순수 토큰만 획득
        String token = authorization.split(" ")[1];
			
				//토큰 소멸 시간 검증
        if (jwtUtil.isExpired(token)) {

            System.out.println("token expired");
            filterChain.doFilter(request, response);

						//조건이 해당되면 메소드 종료 (필수)
            return;
        }

				//토큰에서 username과 role 획득
        String username = jwtUtil.getUsername(token);
        String role = jwtUtil.getRole(token);
				
				//userEntity를 생성하여 값 set
        UserEntity userEntity = new UserEntity();
        userEntity.setUsername(username);
        userEntity.setPassword("temppassword");
        userEntity.setRole(role);
				
				//UserDetails에 회원 정보 객체 담기
        CustomUserDetails customUserDetails = new CustomUserDetails(userEntity);

				//스프링 시큐리티 인증 토큰 생성
        Authentication authToken = new UsernamePasswordAuthenticationToken(customUserDetails, null, customUserDetails.getAuthorities());
				//세션에 사용자 등록
        SecurityContextHolder.getContext().setAuthentication(authToken);

        filterChain.doFilter(request, response);
    }
}
```

---

### SecurityConfig JWTFilter 등록

- **SecurityConfig**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    private final AuthenticationConfiguration authenticationConfiguration;
    private final JWTUtil jwtUtil;

    public SecurityConfig(AuthenticationConfiguration authenticationConfiguration, JWTUtil jwtUtil) {

        this.authenticationConfiguration = authenticationConfiguration;
        this.jwtUtil = jwtUtil;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http
                .csrf((auth) -> auth.disable());

        http
                .formLogin((auth) -> auth.disable());

        http
                .httpBasic((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
                        .anyRequest().authenticated());
				
				//JWTFilter 등록
        http
                .addFilterBefore(new JWTFilter(jwtUtil), LoginFilter.class);

        http
                .addFilterAt(new LoginFilter(authenticationManager(authenticationConfiguration), jwtUtil), UsernamePasswordAuthenticationFilter.class);

        http
                .sessionManagement((session) -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}
```

---

### JWT 요청 인가 테스트

요청 헤더에 JWT를 첨부하고 로그인이 권한이 필요한 /admin에 요청을 보내보자

#### /join
먼저 회원가입을 한다.
username을 admin으로 회원가입 했기 때문에 role은 ADMIN이 된다.
![](https://velog.velcdn.com/images/jckim22/post/20ad631b-bad0-49cd-8445-cd8326f3eeb3/image.png)

#### /login
![](https://velog.velcdn.com/images/jckim22/post/af2335c4-5a1e-4245-9ab1-52e4f471a4e2/image.png)
이제 회원가입했던 username과 password로 로그인을 진행하게 되면 응답으로 위처럼 Token을 받게 될 것이다.

#### /admin

이제 클라이언트에서 헤더에 이 토큰을 넣고 admin에 접근한다.

![](https://velog.velcdn.com/images/jckim22/post/83833936-546c-4415-baaa-8b80a303519e/image.png)
![](https://velog.velcdn.com/images/jckim22/post/b74f7188-b829-4d3f-80cc-3702d2de7aa9/image.png)

성공적으로 접근이 된 것을 볼 수 있다.

---

# 세션 정보

---


### JWTFilter를 통과한 뒤 세션 확인

```java
@Controller
@ResponseBody
public class MainController {

    @GetMapping("/")
    public String mainP() {

        String name = SecurityContextHolder.getContext().getAuthentication().getName();

        return "Main Controller : "+name;
    }
}
```

---

### 세션 현재 사용자 아이디

```java
SecurityContextHolder.getContext().getAuthentication().getName();
```

---

### 세션 현재 사용자 role

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
Iterator<? extends GrantedAuthority> iter = authorities.iterator();
GrantedAuthority auth = iter.next();
String role = auth.getAuthority();
```

---

# CORS 설정


---

### 자료

[CORS :: Spring Security](https://docs.spring.io/spring-security/reference/servlet/integrations/cors.html)

---

### CORS란

[교차 출처 리소스 공유](https://ko.wikipedia.org/wiki/교차_출처_리소스_공유)

---

### 발생 원리



---

### CORS 설정

- **SecurityConfig**

```java
@Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
				
				http
                .cors((corsCustomizer -> corsCustomizer.configurationSource(new CorsConfigurationSource() {

                    @Override
                    public CorsConfiguration getCorsConfiguration(HttpServletRequest request) {

                        CorsConfiguration configuration = new CorsConfiguration();

                        configuration.setAllowedOrigins(Collections.singletonList("http://localhost:3000"));
                        configuration.setAllowedMethods(Collections.singletonList("*"));
                        configuration.setAllowCredentials(true);
                        configuration.setAllowedHeaders(Collections.singletonList("*"));
                        configuration.setMaxAge(3600L);

												configuration.setExposedHeaders(Collections.singletonList("Authorization"));

                        return configuration;
                    }
                })));

        return http.build();
    }
```

- **config>CorsMvcConfig**

```java
@Configuration
public class CorsMvcConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry corsRegistry) {
        
        corsRegistry.addMapping("/**")
                .allowedOrigins("http://localhost:3000");
    }
}
```

---

# JWT의 궁극적인 목표

> 출처: 개발자 유미

### JWT를 사용한 이유

- **모바일 앱**
    
    JWT가 사용된 주 이유는 결국 모바일 앱의 등장입니다. 모바일 앱의 특성상 주로 JWT 방식으로 인증/인가를 진행합니다. 결국 STATLESS는 부수적인 효과였습니다.
    
- **모바일 앱에서의 로그아웃**
    
    모바일 앱에서는 JWT 탈취 우려가 거의 없기 때문에 앱단에서 로그아웃을 진행하여 JWT 자체를 제거해버리면 서버측에선 추가 조치도 필요가 없습니다. (토큰 자체가 확실하게 없어졌다는 보장이 되기 때문에)
    
- **장시간 로그인과 세션**
    
    장기간 동안 로그인 상태를 유지하려고 세션 설정을 하면 서버 측 부하가 많이 가기 때문에 JWT 방식을 이용하는 것도 한 방법입니다.
    

---

### API 서버에서 Refresh 토큰을 저장하여 어느 정도의 상태(STATE)를 만드는 이유

이제는 상태를 남기는것에 대해서 큰 고민은 없어졌습니다. JWT의 목적이 STATELESS가 아니기 때문이니까요.

---

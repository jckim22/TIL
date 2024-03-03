#### LoginCheckInterceptor

```java
@Slf4j
public class LoginCheckInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();

        log.info("인증 체크 인터셉터 실행 {}", requestURI);

        HttpSession session = request.getSession();

        if (session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
            log.info("미인증 사용자 요청");
            //로그인으로 redirect
            response.sendRedirect("/login?redirectURL=" + requestURI);
            return false;
        }

        return true;
    }
}

```

인터셉터로 인가를 처리한다.
HandlerInterceptor를 구현하는데 인가를 하기 위해서 컨트롤러 전에 실행되는 preHandle만 구현하면 된다.
특정 요청에서 세션이 없다면 login으로 리다이렉트하게 한다.


#### WebCondig

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
//    @Override
//    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
//        resolvers.add(new LoginMemberArgumentResolver());
//    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginCheckInterceptor())
                .order(2)
                .addPathPatterns("/**")
                .excludePathPatterns("/", "/members/new", "/login", "/logout",
                        "/css/**","/assets/**", "/*.ico", "/error");
    }
}


```

그리고 컨피그에서 화이트리스트로 제외할 URL들을 정한다.
이제 해당 URL에서 인가처리가 진행된다.


# 공통 관심사처리
```java
@SessionAttribute(name = SessionConst.LOGIN_MEMBER Member sessionMember
```
원래는 위처럼 세션의 sessionMember를 받고 해당 sessionMembmer로 인가처리를 진행하지만, ArgumentResolver로 이것을 편하게 해결할 수 있다.


#### @Login
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface Login {
}
```

#### LoginMemberArgumentResolver
커스텀 어노테이션을 만들어준다.

```java
@Slf4j
public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        log.info("supportsParameter 실행");

        boolean hasLoginAnnotation = parameter.hasParameterAnnotation(Login.class);
        boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());

        return hasLoginAnnotation && hasMemberType;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {

        log.info("resolveArgument 실행");

        HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
        HttpSession session = request.getSession(false);
        if (session == null) {
            return null;
        }

        return session.getAttribute(SessionConst.LOGIN_MEMBER);
    }
}

```

그리고 HandlerMethodArgumentResolver를 구현한 ArgumentResolver를 만들어주는데 supportsParameter가 실행되고 충족 되어야 resolveArgument가 실행이 되기 떄문에 support에서는 @Login 어노테이션을 달고 있는지, 그에 대한 파라미터가 Member 클래스인지를 체크한다.

둘 다 충족이 되면 resolveArgument에서 인가처리를 진행한다.

이제 @Login만 달아줘도 모든 세션처리를 해결할 수 있다.


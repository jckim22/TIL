## MVC 컨트롤러의 단점

이전에 MVC를 적용해봤지만 그 한계를 살펴보자.

**포워드 중복**
View로 이동하는 코드가 항상 중복 호출되어야 한다. 물론 이 부분을 메서드로 공통화해도 되지만, 해당 메서드도 항상
직접 호출해야 한다. ```java
 RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response); ```
  
**ViewPath에 중복** ```
 String viewPath = "/WEB-INF/views/new-form.jsp";
```
prefix: `/WEB-INF/views/` suffix: `.jsp`
그리고 만약 jsp가 아닌 thymeleaf 같은 다른 뷰로 변경한다면 전체 코드를 다 변경해야 한다. **사용하지 않는 코드**
다음 코드를 사용할 때도 있고, 사용하지 않을 때도 있다. 특히 response는 현재 코드에서 사용되지 않는다. ```
 HttpServletRequest request, HttpServletResponse response
```
그리고 이런 `HttpServletRequest` , `HttpServletResponse` 를 사용하는 코드는 테스트 케이스를 작성하기도 어렵다.

**공통 처리가 어렵다.**
기능이 복잡해질 수 록 컨트롤러에서 공통으로 처리해야 하는 부분이 점점 더 많이 증가할 것이다. 단순히 공통 기능을 메서드로 뽑으면 될 것 같지만, 결과적으로 해당 메서드를 항상 호출해야 하고, 실수로 호출하지 않으면 문제가 될 것이 다. 그리고 호출하는 것 자체도 중복이다.

**정리하면 공통 처리가 어렵다는 문제가 있다.**
이 문제를 해결하려면 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다. 소위 **수문장 역할**을 하는 기능이 필요하다. **프론트 컨트롤러(Front Controller) 패턴**을 도입하면 이런 문제를 깔끔하게 해결할 수 있다.(입구를 하나로!)
스프링 MVC의 핵심도 바로 이 프론트 컨트롤러에 있다.


# 프론트 컨트롤러 패턴

#### 프론트 컨트롤러 적용 전

![](https://velog.velcdn.com/images/jckim22/post/4ab9a3cf-c529-4fb9-94dc-dd85c836a8e8/image.png)

#### 프론트 컨트롤러 적용 후

![](https://velog.velcdn.com/images/jckim22/post/c71a8c67-de4c-4d0c-a5a1-1e182d4733a0/image.png)


**FrontController 패턴 특징**
- 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받음
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출
- 입구를 하나로!
- 공통 처리 가능
- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨
	
**스프링 웹 MVC와 프론트 컨트롤러**

스프링 웹 MVC의 핵심도 바로 **FrontController**
스프링 웹 MVC의 **DispatcherServlet**이 FrontController 패턴으로 구현되어 있음



# 프론트 컨트롤러 도입 - v1

![](https://velog.velcdn.com/images/jckim22/post/5dc8a95e-e10b-4cec-a761-ad157dff1348/image.png)




```java
public interface ControllerV1 {

    void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

위처럼 서블릿과 비슷한 모양의 컨트롤러 인터페이스를 도입한다. 각 컨트롤러들은 이 인터페이스를 구현하면 된다. 프론트 컨 트롤러는 이 인터페이스를 호출해서 구현과 관계없이 로직의 일관성을 가져갈 수 있다.

이제 이 인터페이스를 컨트롤러로 구현해보자
v1에서는 프론트 컨트롤러를 사용할 것이기 때문에 뒤 컨트롤러는 서블릿을 사용하지 않아도 되지만 포워드는 뒤 컨트롤러에서 계속 담당할 것이다.


```java
public class MemberFormControllerV1 implements ControllerV1 {

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }

}

```




```java
public class MemberListControllerV1 implements ControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        List<Member> members = memberRepository.findAll();

        request.setAttribute("members", members);

        String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}

```

```java
public class MemberSaveControllerV1 implements ControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        request.setAttribute("member", member);

        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}

```

아래는 프론트 컨트롤러이다.

```java
@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {

    private Map<String, ControllerV1> controllerMap = new HashMap<>();

    public FrontControllerServletV1() {
        controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
        controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("FrontControllerServletV1.service");

        String requestURI = request.getRequestURI();

        ControllerV1 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        controller.process(request, response);
    }
}

```

프론트 컨트롤러에서 urlPatterns를 /front-controller/v1/\*로 했기 때문에 v1 하위의 URI은 전부 이 프론트 컨트롤러로 먼저 들어온다.

MAP으로 URL을 키로하고 구현한 컨트롤러를 매핑한 매핑 정보를 담아놓는다.

그리고 service로직에는 request객체에서 URI를 받고 그 URI로 매핑한 객체를 인터페이스의 다형성을 활용하여 controller에 담는다.
그리고 그 컨트롤러를 가동한다.



# View 분리 - v2

![](https://velog.velcdn.com/images/jckim22/post/a9aba1dd-7737-45b2-85e7-1993757f8ffc/image.png)


이번에는 랜더링하는 과정을 분리한다.
아래를 보자

```java
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
```
위처럼 랜더링하는 과정을 따로 빼준다.


```java
    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        request.setAttribute("member", member);

        return new MyView("/WEB-INF/views/save-result.jsp");
    }
```
위와 같이 각 컨트롤러에서는 MyView를 새로 생성해서 리턴한다.


```java
@WebServlet(name = "frontControllerServletV2", urlPatterns = "/front-controller/v2/*")
public class FrontControllerServletV2 extends HttpServlet {

    private Map<String, ControllerV2> controllerMap = new HashMap<>();

    public FrontControllerServletV2() {
        controllerMap.put("/front-controller/v2/members/new-form", new MemberFormControllerV2());
        controllerMap.put("/front-controller/v2/members/save", new MemberSaveControllerV2());
        controllerMap.put("/front-controller/v2/members", new MemberListControllerV2());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String requestURI = request.getRequestURI();

        ControllerV2 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyView view = controller.process(request, response);
        view.render(request, response);
    }
}
```

이제 프론트 컨트롤러에서 다형성을 통해 받은 컨트롤러를 실행시켜서 view를 받는다.

view를 랜더링한다.


프론트 컨트롤러의 도입으로 `MyView` 객체의 `render()` 를 호출하는 부분을 모두 일관되게 처리할 수 있다. 각각의 컨트롤러는 `MyView` 객체를 생성만 해서 반환하면 된다.


개발자들은 컨트롤러의 반환 타입을 보고 view를 반환해야되는걸 인지한다.


# Model 추가 - v3

![](https://velog.velcdn.com/images/jckim22/post/4c32d39b-993f-4ecd-938a-63f6f0bc14db/image.png)


**서블릿 종속성 제거**
컨트롤러 입장에서 HttpServletRequest, HttpServletResponse이 꼭 필요할까?
요청 파라미터 정보는 자바의 Map으로 대신 넘기도록 하면 지금 구조에서는 컨트롤러가 서블릿 기술을 몰라도 동작할 수 있다.
그리고 request 객체를 Model로 사용하는 대신에 별도의 Model 객체를 만들어서 반환하면 된다.
우리가 구현하는 컨트롤러가 서블릿 기술을 전혀 사용하지 않도록 변경해보자.
이렇게 하면 구현 코드도 매우 단순해지고, 테스트 코드 작성이 쉽다.

**뷰 이름 중복 제거**
컨트롤러에서 지정하는 뷰 이름에 중복이 있는 것을 확인할 수 있다.
컨트롤러는 **뷰의 논리 이름**을 반환하고, 실제 물리 위치의 이름은 프론트 컨트롤러에서 처리하도록 단순화 하자. 이렇게 해두면 향후 뷰의 폴더 위치가 함께 이동해도 프론트 컨트롤러만 고치면 된다.


```java
public class ModelView {
    private String viewName;
    private Map<String, Object> model = new HashMap<>();

    public ModelView(String viewName) {
        this.viewName = viewName;
    }

    public String getViewName() {
        return viewName;
    }

    public void setViewName(String viewName) {
        this.viewName = viewName;
    }

    public Map<String, Object> getModel() {
        return model;
    }

    public void setModel(Map<String, Object> model) {
        this.model = model;
    }
}

```
이제 리퀘스트 객체를 모델로 사용하지 않고 따로 모델 객체를 만들어서 거기에 데이터를 담을 것이다.
위와 같은 ModelView 클래스를 만든다.

```java
public class MemberSaveControllerV3 implements ControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelView mv = new ModelView("save-result");
        mv.getModel().put("member", member);
        return mv;
    }
}
```

이제 컨트롤러를 구현한다.
위 코드를 보면 이전과는 다르게 Servlet을 사용하지 않는다.
paraMap이라는 정보를 받아서 paraMap으로 그에 대한 정보를 모델에 담아서 리턴한다.

컨트롤러에서는 서블릿에 종속당하지 않을 뿐 아니라 ModelView를 생성하면서 파라미터로 save-result라는 간단한 뷰 네임만 보내기 때문에 물리적인 URL에도 관여하지 않게 된다.


```java
@WebServlet(name = "frontControllerServletV3", urlPatterns = "/front-controller/v3/*")
public class FrontControllerServletV3 extends HttpServlet {

    private Map<String, ControllerV3> controllerMap = new HashMap<>();

    public FrontControllerServletV3() {
        controllerMap.put("/front-controller/v3/members/new-form", new MemberFormControllerV3());
        controllerMap.put("/front-controller/v3/members/save", new MemberSaveControllerV3());
        controllerMap.put("/front-controller/v3/members", new MemberListControllerV3());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

        String requestURI = request.getRequestURI();

        ControllerV3 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        String viewName = mv.getViewName();
        MyView view = viewResolver(viewName);

        view.render(mv.getModel(), request, response);
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```

프론트 컨트롤러는 더 복잡해졌다.
하지만 이것이 프레임워크를 만드는 과정임을 아는 것이 중요하다.
이 프론트 컨트롤러에 복잡한 과정들은 후에 Spring에서 알아서 해줄 것이다.

우리는 컨트롤러를 다루는게 얼마나 편해졌는지에 대해 생각한다.

위 프론트 컨트롤러에서는 request객체를 paramMap이라는 모델에 request의 내용을 담는다.
그리고 컨트롤러에 모델에 대한 정보를 보내고 ModelView를 전달 받는다.
ModelView에서는 URL을 직접 건들지 않았기 때문에 그것을 해결해주는 viewResolver를 사용해서 물리적 주소로 변경한다.

그리고 view에 모델에 대한 정보를 보냄으로써 렌더링까지 끝낸다.


# 단순하고 실용적인 컨트롤러 - v4

앞서 만든 v3 컨트롤러는 서블릿 종속성을 제거하고 뷰 경로의 중복을 제거하는 등, 잘 설계된 컨트롤러이다. 그런데 실제 컨트톨러 인터페이스를 구현하는 개발자 입장에서 보면, 항상 ModelView 객체를 생성하고 반환해야 하는 부분이 조금은 번거롭다.
좋은 프레임워크는 아키텍처도 중요하지만, 그와 더불어 실제 개발하는 개발자가 단순하고 편리하게 사용할 수 있어야 한다. 소위 실용성이 있어야 한다.
이번에는 v3를 조금 변경해서 실제 구현하는 개발자들이 매우 편리하게 개발할 수 있는 v4 버전을 개발해보자.

![](https://velog.velcdn.com/images/jckim22/post/8fd857ef-8714-42ce-9a4c-3d32a2be67cf/image.png)


```java
public interface ControllerV4 {

    /**
     * @param paramMap
     * @param model
     * @return viewName
     */
    String process(Map<String, String> paramMap, Map<String, Object> model);
}

```

위처럼 인터페이스를 새로 정의한다.
인자로 model을 아예 받는다.


```java
public class MemberSaveControllerV4 implements ControllerV4 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.put("member", member);
        return "save-result";
    }
}
```
인터페이스를 구현한다.
이번에는 ModelView를 생성하지 않고 viewname을 리턴하기 때문에 개발자는 훨씬 더 편리하고 깔끔하게 코드를 볼 수 있다.
model을 직접 받아서 세팅도 한다.

```java
@WebServlet(name = "frontControllerServletV4", urlPatterns = "/front-controller/v4/*")
public class FrontControllerServletV4 extends HttpServlet {

    private Map<String, ControllerV4> controllerMap = new HashMap<>();

    public FrontControllerServletV4() {
        controllerMap.put("/front-controller/v4/members/new-form", new MemberFormControllerV4());
        controllerMap.put("/front-controller/v4/members/save", new MemberSaveControllerV4());
        controllerMap.put("/front-controller/v4/members", new MemberListControllerV4());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String requestURI = request.getRequestURI();

        ControllerV4 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>(); //추가

        String viewName = controller.process(paramMap, model);

        MyView view = viewResolver(viewName);
        view.render(model, request, response);
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}

```

프론트 컨트롤러에서도 ModelView 없이 viewname을 직접 리턴받아서 뷰 리졸버로 물리적 URL을 만들고 랜더링한다.


# 유연한 컨트롤러1 - v5

만약 어떤 개발자는 `ControllerV3` 방식으로 개발하고 싶고, 어떤 개발자는 `ControllerV4` 방식으로 개발하고 싶다면 어떻게 해야할까?

**어댑터 패턴**
지금까지 우리가 개발한 프론트 컨트롤러는 한가지 방식의 컨트롤러 인터페이스만 사용할 수 있다.
      `ControllerV3` , `ControllerV4` 는 완전히 다른 인터페이스이다. 따라서 호환이 불가능하다. 마치 v3는 110v이
고, v4는 220v 전기 콘센트 같은 것이다. 이럴 때 사용하는 것이 바로 어댑터이다.
어댑터 패턴을 사용해서 프론트 컨트롤러가 다양한 방식의 컨트롤러를 처리할 수 있도록 변경해보자.


```java
 public interface ControllerV3 {
     ModelView process(Map<String, String> paramMap);
}
```
```java
 public interface ControllerV4 {
     String process(Map<String, String> paramMap, Map<String, Object> model);
}
```


![](https://velog.velcdn.com/images/jckim22/post/d4dd9ea7-8784-427d-9aa2-081523486062/image.png)


- **핸들러 어댑터**: 중간에 어댑터 역할을 하는 어댑터가 추가되었는데 이름이 핸들러 어댑터이다. 여기서 어댑터 역할을 해주는 덕분에 다양한 종류의 컨트롤러를 호출할 수 있다.

- **핸들러**: 컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경했다. 그 이유는 이제 어댑터가 있기 때문에 꼭 컨트롤러 의 개념 뿐만 아니라 어떠한 것이든 해당하는 종류의 어댑터만 있으면 다 처리할 수 있기 때문이다.


**MyHandlerAdapter**
어댑터는 이렇게 구현해야 한다는 어댑터용 인터페이스이다.

```java
public interface MyHandlerAdapter {

    boolean supports(Object handler);

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```

- oolean supports(Object handler)`
   - handler는 컨트롤러를 말한다.
   - 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드다.
- `ModelView handle(HttpServletRequest request, HttpServletResponse response,
Object handler)`
   - 어댑터는 실제 컨트롤러를 호출하고, 그 결과로 ModelView를 반환해야 한다.
   - 실제 컨트롤러가 ModelView를 반환하지 못하면, 어댑터가 ModelView를 직접 생성해서라도 반환해야 한다.
   - 이전에는 프론트 컨트롤러가 실제 컨트롤러를 호출했지만 이제는 이 어댑터를 통해서 실제 컨트롤러가 호출 된다.
   
실제 어댑터를 구현해보자.
먼저 ControllerV3를 지원하는 어댑터를 구현하자.

**ControllerV3HandlerAdapter**

```java
public class ControllerV3HandlerAdapter implements MyHandlerAdapter {

    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV3);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        //MemberFormControllerV3
        ControllerV3 controller = (ControllerV3) handler;

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        return mv;
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }


}
```

**FrontControllerServletV5**
    
```java
@WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {

    private final Map<String, Object> handlerMappingMap = new HashMap<>();
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();

    public FrontControllerServletV5() {
        initHandlerMappingMap();
        initHandlerAdapters();
    }

    private void initHandlerMappingMap() {
        handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());

        //V4 추가
        handlerMappingMap.put("/front-controller/v5/v4/members/new-form", new MemberFormControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members/save", new MemberSaveControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members", new MemberListControllerV4());
    }

    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdapter());
        handlerAdapters.add(new ControllerV4HandlerAdapter());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        Object handler = getHandler(request);
        if (handler == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyHandlerAdapter adapter = getHandlerAdapter(handler);

        ModelView mv = adapter.handle(request, response, handler);

        String viewName = mv.getViewName();
        MyView view = viewResolver(viewName);

        view.render(mv.getModel(), request, response);

    }

    private Object getHandler(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        return handlerMappingMap.get(requestURI);
    }

    private MyHandlerAdapter getHandlerAdapter(Object handler) {
        //MemberFormControllerV4
        for (MyHandlerAdapter adapter : handlerAdapters) {
            if (adapter.supports(handler)) {
                return adapter;
            }
        }
        throw new IllegalArgumentException("handler adapter를 찾을 수 없습니다. handler=" + handler);
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}

```
**컨트롤러(Controller) 핸들러(Handler)**
이전에는 컨트롤러를 직접 매핑해서 사용했다. 그런데 이제는 어댑터를 사용하기 때문에, 컨트롤러 뿐만 아니라 어댑터
가 지원하기만 하면, 어떤 것이라도 URL에 매핑해서 사용할 수 있다. 그래서 이름을 컨트롤러에서 더 넒은 범위의 핸들러로 변경했다.


**생성자**
```java
public FrontControllerServletV5() { 
	initHandlerMappingMap(); //핸들러 매핑 초기화 initHandlerAdapters(); //어댑터 초기화
}
```
생성자는 핸들러 매핑과 어댑터를 초기화(등록)한다.


핸들러를 처리할 수 있는 어댑터를 찾는다.

```java
        MyHandlerAdapter adapter = getHandlerAdapter(handler);

        ModelView mv = adapter.handle(request, response, handler);
```
그리고 해당 어댑터의 handle 메서드로 해당 컨트롤러를 실행해서 ModelView를 반환 받는다.

```java
ublic class ControllerV3HandlerAdapter implements MyHandlerAdapter {

    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV3);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        //MemberFormControllerV3
        ControllerV3 controller = (ControllerV3) handler;

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        return mv;
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }


}
```

위처럼 v3컨트롤러의 로직을 실행해서 ModelView에 받고 리턴한다.

**정리**
지금은 V3 컨트롤러를 사용할 수 있는 어댑터와 `ControllerV3` 만 들어 있어서 크게 감흥이 없을 것이다.
`ControllerV4` 를 사용할 수 있도록 기능을 추가해보자.


## V4Adapter


```java
public class ControllerV4HandlerAdapter implements MyHandlerAdapter {

    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV4);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV4 controller = (ControllerV4) handler;

        Map<String, String> paramMap = createParamMap(request);
        HashMap<String, Object> model = new HashMap<>();

        String viewName = controller.process(paramMap, model);
        
        //Modelview로 만들어주면서 220v를 110v로 사용할 수 있는 어댑터 역할을 해줌
        ModelView mv = new ModelView(viewName);
        mv.setModel(model);

        return mv;
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }

}
```

위처럼 V4 Adapter만 추가되면 FrontController의 세팅을 조금만 바꾸면 확장이 쉽게 가능해서 OCP를 거의 지켰다고 볼 수 있다.

v4Adapter에서 중요한 부분은 ModelView로 바꿔주는 것이다.
ModelView로 바꿔서 보내주므로서 어댑터의 역할을 확실히 한다.

잘 보면 이전에 배웠던 것처럼 기능과 역할이 잘 분리되어 있는 것을 볼 수 있다.
기능(인터페이스)위주로 코드를 짜는 것이 중요하다.


# 정리
지금까지 v1 ~ v5로 점진적으로 프레임워크를 발전시켜 왔다. 지금까지 한 작업을 정리해보자.

**v1: 프론트 컨트롤러를 도입**
기존 구조를 최대한 유지하면서 프론트 컨트롤러를 도입
**v2: View 분류**
단순 반복 되는 뷰 로직 분리
**v3: Model 추가**
서블릿 종속성 제거
  뷰 이름 중복 제거
**v4: 단순하고 실용적인 컨트롤러** v3와 거의 비슷
구현 입장에서 ModelView를 직접 생성해서 반환하지 않도록 편리한 인터페이스 제공 **v5: 유연한 컨트롤러**
어댑터 도입
어댑터를 추가해서 프레임워크를 유연하고 확장성 있게 설계


여기에 애노테이션을 사용해서 컨트롤러를 더 편리하게 발전시킬 수도 있다. 만약 애노테이션을 사용해서 컨트롤러를 편리하게 사용할 수 있게 하려면 어떻게 해야할까? 바로 애노테이션을 지원하는 어댑터를 추가하면 된다!
다형성과 어댑터 덕분에 기존 구조를 유지하면서, 프레임워크의 기능을 확장할 수 있다.

**스프링 MVC**
여기서 더 발전시키면 좋겠지만, 스프링 MVC의 핵심 구조를 파악하는데 필요한 부분은 모두 만들어보았다.
사실은 여러분이 지금까지 작성한 코드는 스프링 MVC 프레임워크의 핵심 코드의 축약 버전이고, 구조도 거의 같다.
스프링 MVC는 지금까지 우리가 학습한 내용과 거의 같은 구조를 가지고 있다.

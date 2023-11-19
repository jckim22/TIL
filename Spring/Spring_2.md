# 정적 컨텐츠
Spring은 정적 컨텐츠를 제공한다.
static에서 정적 파일을 던져준다. (프로그래밍 불가)


![](https://velog.velcdn.com/images/jckim22/post/6df4ea21-ddae-4831-8215-1e9801738f74/image.png)
localhost:8080/hello.html -> controller에서 매핑이 된 url이 있는지 확인 -> 없으면 static에서 해당 정적 파일을 찾아서 반환

# MVC와 템플릿 엔진

Model-View-Controller

View는 화면을 그리는데 모든 역량을 집중해야한다.
Model-Controller는 내부적인 비즈니스 로직을 처리해야한다.

![](https://velog.velcdn.com/images/jckim22/post/51293c2b-f836-4428-a25f-199c94397a29/image.png)

### controller

```java
    @GetMapping("hello-mvc")
    public String helloMvc(@RequestParam(value = "name") String name, Model model){
        model.addAttribute("name",name);
        return "hello-template";
    }
```

### template 파일
```html
<html xmlns:th="http://www.thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}">hello! empty</p>
</body>
</html>
```

1. @RequestParam의 옵션으로 required를 false 값으로 설정하지 않았기 떄문에 default값이 존재하지 않는다.
2. name이라는 value에 파라미터로 값을 받게 되고 
3. 그것은 name 변수에 담긴다.
4. name이라는 이름으로 Model을 view에 넘긴다.
5. hello-template를 리턴함으로서 template에서 hello-template.html 파일을 찾는다.
6. 타임리프(템플릿 엔진)은 받은 데이터를 사용해 html을 랜더링 한 후 화면에 출력한다.

![](https://velog.velcdn.com/images/jckim22/post/9401999a-b925-4b59-b2b4-a4198e8fd65b/image.png)




# API

### Hello 클래스
```java
    static class Hello {
        private String name;
        private String aa = "asdfasdf";

        public void setName(String name) {
            this.name = name;
        }

        public String getAa() {
            return aa;
        }

        public String getName() {
            return name;
        }
    }
```

### controller

```java
   	@GetMapping("hello-api")
    @ResponseBody
    public Hello helloAPi(@RequestParam("name") String name){
        Hello hello = new Hello();
        hello.setName(name);
        return hello; //객체를 반환
```

위처럼 @ResponseBody 어노테이션을 설정하게 되면 아래와 같은 구조로 동작한다.

![](https://velog.velcdn.com/images/jckim22/post/025d3eaa-4d20-4ecc-8f01-da14e8e92ea0/image.png)

controller 메소드에 ResponseBody가 붙어있는걸 확인하고 뷰리졸버 대신 HttpMessageConveter가 동작한다.

만약 객체 형태라면 기본으로 탑재된 잭슨 라이브러리로 Json으로 파싱한 뒤 클라이언트에게 반환한다.

위 코드에서는 파라미터로 값을 받은 뒤 객체를 생성하고 세팅하고 객체를 반환한다.

그럼 아래처럼 객체가 Json 형태의 파일로 반환된 것을 볼 수 있다.

![](https://velog.velcdn.com/images/jckim22/post/d7de53c4-d8c8-4bda-b9a4-760c8cae1999/image.png)




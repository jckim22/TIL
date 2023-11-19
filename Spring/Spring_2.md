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

![](https://velog.io/215790dd-9765-4c40-ab7c-44d40a2057a1)


# API

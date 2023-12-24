## 메시지
기획자가 화면에 보이는 문구가 마음에 들지 않는다고, **상품명**이라는 단어를 모두 **상품이름**으로 고쳐달라고 하면 어떻게 해야할까?
여러 화면에 보이는 상품명, 가격, 수량 등, `label` 에 있는 단어를 변경하려면 다음 화면들을 다 찾아가면서 모두 변경 해야 한다. 지금처럼 화면 수가 적으면 문제가 되지 않지만 화면이 수십개 이상이라면 수십개의 파일을 모두 고쳐야 한다.

왜냐하면 해당 HTML 파일에 메시지가 하드코딩 되어 있기 때문이다.
이런 다양한 메시지를 한 곳에서 관리하도록 하는 기능을 메시지 기능이라 한다.


#### messages.properties
```
hello=ìë
hello.name=ìë {0}

label.item=ìí
label.item.id=ìí ID
label.item.itemName=ìíëª
label.item.price=ê°ê²©
label.item.quantity=ìë

page.items=ìí ëª©ë¡
page.item=ìí ìì¸
page.addItem=하이 하이
page.updateItem=ìí ìì 

button.save=ì ì¥
button.cancel=ì·¨ì
```

#### messages_en.properties
```
hello=hello
hello.name=hello {0}

label.item=Item
label.item.id=Item ID
label.item.itemName=Item Name
label.item.price=price
label.item.quantity=quantity

page.items=Item List
page.item=Item Detail
page.addItem=Item Add
page.updateItem=Item Update

button.save=Save
button.cancel=Cancel

```

위에 한글은 깨져있지만 이렇게 메시지 설정을 해놓으면 클라이언트의 locale에 따라서 다른 언어 셋을 준다.

```java
@SpringBootTest
public class MessageSourceTest {

    @Autowired
    MessageSource ms;

    @Test
    void helloMessage() {
        String result = ms.getMessage("hello", null, null);
        assertThat(result).isEqualTo("안녕");
    }

    @Test
    void notFoundMessageCode() {
        assertThatThrownBy(() -> ms.getMessage("no_code", null, null))
                .isInstanceOf(NoSuchMessageException.class);
    }

    @Test
    void notFoundMessageCodeDefaultMessage() {
        String result = ms.getMessage("no_code", null, "기본 메시지", null);
        assertThat(result).isEqualTo("기본 메시지");
    }

    @Test
    void argumentMessage() {
        String message = ms.getMessage("hello.name", new Object[]{"Spring"}, null);
        assertThat(message).isEqualTo("안녕 Spring");
    }

    @Test
    void defaultLang() {
        assertThat(ms.getMessage("hello", null, null)).isEqualTo("안녕");
        assertThat(ms.getMessage("hello", null, Locale.KOREA)).isEqualTo("안녕");
    }

    @Test
    void enLang() {
        assertThat(ms.getMessage("hello", null, Locale.ENGLISH)).isEqualTo("hello");
    }
}

```

MesseageSource라는 자동 등록되는 스프링 빈을 확인해보면 위와 같은 테스트를 모두 통과한다.
자연스럽게 messsage.properties의 내용을 가져오기 때문이다.

>argumentMessage 테스트를 보면 인자를 넣어주는 것을 볼 수 있다.
인자를 넣는 것도 가능하다.
```
`hello.name=안녕 {0}`
`<p th:text="#{hello.name(${item.itemName})}"></p>`
```


이걸 프로젝트에 적용하면 아래처럼 타임리프를 작성하면된다.

```java
    <div class="py-5 text-center">
        <h2 th:text="#{page.addItem}">상품 등록</h2>
    </div>
```

![](https://velog.velcdn.com/images/jckim22/post/76acfc7e-726a-4a51-882b-3bc8b8e186cd/image.png)
![](https://velog.velcdn.com/images/jckim22/post/a62e583d-3798-4ec7-8a72-f7d0934d316f/image.png)


위처럼 세팅을 "하이 하이"로 바꾸면 인코딩이 맞지 않아 깨졌지만 ?? ??로 보이게 된다.
아마 아스키 코드로 인코딩이 되어 한글이 인식하지 못해 ?로 치환된 것 같다.

locale에 따라 메시지도 국제화가 된다.

브라우저 설정에서 언어 설정을 영로 바꾸면 en.properties의 셋을 가져오게 된다.

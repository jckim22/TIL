# Bean Validation이란
검증 기능을 지금처럼 매번 코드로 작성하는 것은 상당히 번거롭다. 특히 특정 필드에 대한 검증 로직은 대부분 빈 값인
지 아닌지, 특정 크기를 넘는지 아닌지와 같이 매우 일반적인 로직이다. 다음 코드를 보자.

```java
 public class Item {
     private Long id;
     @NotBlank
     private String itemName;
     @NotNull
     @Range(min = 1000, max = 1000000)
     private Integer price;
     @NotNull
     @Max(9999)
     private Integer quantity;
```

**Bean Validation 이란?**
먼저 Bean Validation은 특정한 구현체가 아니라 Bean Validation 2.0(JSR-380)이라는 기술 표준이다. 쉽게 이야 기해서 검증 애노테이션과 여러 인터페이스의 모음이다. 마치 JPA가 표준 기술이고 그 구현체로 하이버네이트가 있는 것과 같다.
Bean Validation을 구현한 기술중에 일반적으로 사용하는 구현체는 하이버네이트 Validator이다. 이름이 하이버네이 트가 붙어서 그렇지 ORM과는 관련이 없다.

# Bean Validation - 시작
Bean Validation 기능을 어떻게 사용하는지 코드로 알아보자. 먼저 스프링과 통합하지 않고, 순수한 Bean Validation 사용법 부터 테스트 코드로 알아보자.

**의존관계 추가**
Bean Validation을 사용하려면 다음 의존관계를 추가해야 한다.
`build.gradle` 
```
 implementation 'org.springframework.boot:spring-boot-starter-validation'
  ```
  
 **Item - Bean Validation 애노테이션 적용**
 ```java
 @Data
 public class Item {
     private Long id;
     
     @NotBlank
     private String itemName;
     
     @NotNull
     @Range(min = 1000, max = 1000000)
     private Integer price;
     
     @NotNull
     @Max(9999)
     private Integer quantity;
     
     public Item() {
     }
     public Item(String itemName, Integer price, Integer quantity) {
         this.itemName = itemName;
         this.price = price;
                  this.quantity = quantity;
     }
}        
```

위는 스프링과 통합하지 않은 순수한 Bean validation을 기존 Item 클래스에 적용한 것이다.

**검증 애노테이션**
`@NotBlank` : 빈값 + 공백만 있는 경우를 허용하지 않는다.
`@NotNull` : `null` 을 허용하지 않는다.
`@Range(min = 1000, max = 1000000)` : 범위 안의 값이어야 한다. `@Max(9999)` : 최대 9999까지만 허용한다.

>`jakarta.validation.constraints.NotNull` `org.hibernate.validator.constraints.Range`
`jakarta.validation` 으로 시작하면 특정 구현에 관계없이 제공되는 표준 인터페이스이고,
`org.hibernate.validator` 로 시작하면 하이버네이트 validator 구현체를 사용할 때만 제공되는 검증 기 능이다. 실무에서 대부분 하이버네이트 validator를 사용하므로 자유롭게 사용해도 된다



```java
    @Test
    void beanValidation() {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        Validator validator = factory.getValidator();

        Item item = new Item();
        item.setItemName(" "); //공백
        item.setPrice(0);
        item.setQuantity(10000);

        Set<ConstraintViolation<Item>> violations = validator.validate(item);
        for (ConstraintViolation<Item> violation : violations) {
            System.out.println("violation = " + violation);
            System.out.println("violation = " + violation.getMessage());
        }

    }
```

위처럼 ValidationFactory에서 Validator를 가져오고 해당 객체를 검사하면 된다.
그러면 violations에 에러에 대한 정보가 담긴다.

**검증기 생성**
다음 코드와 같이 검증기를 생성한다. 이후 스프링과 통합하면 우리가 직접 이런 코드를 작성하지는 않으므로, 이렇게
사용하는구나 정도만 참고하자. 
```java
 ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
 Validator validator = factory.getValidator();
```
**검증 실행**
검증 대상( `item` )을 직접 검증기에 넣고 그 결과를 받는다. `Set` 에는 `ConstraintViolation` 이라는 검증 오류가
담긴다. 따라서 결과가 비어있으면 검증 오류가 없는 것이다.
```java
Set<ConstraintViolation<Item>> violations = validator.validate(item);
```


**정리**
이렇게 빈 검증기(Bean Validation)를 직접 사용하는 방법을 알아보았다. 아마 지금까지 배웠던 스프링 MVC 검증 방 법에 빈 검증기를 어떻게 적용하면 좋을지 여러가지 생각이 들 것이다. 스프링은 이미 개발자를 위해 빈 검증기를 스프 링에 완전히 통합해두었다.


# Bean Validation - 스프링 적용

**스프링 MVC는 어떻게 Bean Validator를 사용?**
스프링 부트가 `spring-boot-starter-validation` 라이브러리를 넣으면 자동으로 Bean Validator를 인지하 고 스프링에 통합한다.

**스프링 부트는 자동으로 글로벌 Validator로 등록한다.**
`LocalValidatorFactoryBean` 을 글로벌 Validator로 등록한다. 이 Validator는 `@NotNull` 같은 애노테이션을
보고 검증을 수행한다. 이렇게 글로벌 Validator가 적용되어 있기 때문에, `@Valid` , `@Validated` 만 적용하면 된다. 검증 오류가 발생하면, `FieldError` , `ObjectError` 를 생성해서 `BindingResult` 에 담아준다.


결과적으로 의존관계를 추가하고 @Validator를 등록을 하게 되면 널 기반에 Validator가 수행되고 BindingResult에 담기게 되는 것이다.


**바인딩에 성공한 필드만 Bean Validation 적용**
BeanValidator는 바인딩에 실패한 필드는 BeanValidation을 적용하지 않는다.
생각해보면 타입 변환에 성공해서 바인딩에 성공한 필드여야 BeanValidation 적용이 의미 있다. (일단 모델 객체에 바인딩 받는 값이 정상으로 들어와야 검증도 의미가 있다.)
`@ModelAttribute` 각각의필드타입변환시도 변환에성공한필드만BeanValidation적용 
**예)**
- `itemName` 에 문자 "A" 입력 타입 변환 성공 `itemName` 필드에 BeanValidation 적용
- `price` 에 문자 "A" 입력 "A"를 숫자 타입 변환 시도 실패 typeMismatch FieldError 추가 `price` 필 드는 BeanValidation 적용 X

가격에 문자인데 범위를 보는 validation을 보는 것은 의미가 없음
그래서 properties에 설정해 놓았던 typemismatch 메시지가 던져진다.


# Bean Validation - 에러 코드

Bean Validation이 기본으로 제공하는 오류 메시지를 좀 더 자세히 변경하고 싶으면 어떻게 하면 될까?
Bean Validation을 적용하고 `bindingResult` 에 등록된 검증 오류 코드를 보자. 오류 코드가 애노테이션 이름으로 등록된다. 마치 `typeMismatch` 와 유사하다.
`NotBlank` 라는 오류 코드를 기반으로 `MessageCodesResolver` 를 통해 다양한 메시지 코드가 순서대로 생성된 다.

**@NotBlank** 
- NotBlank.item.itemName 
- NotBlank.itemName 
- NotBlank.java.lang.String
- NotBlank

**@Range** 
- Range.item.price
- Range.price 
- Range.java.lang.Integer 
- Range

`errors.properties` 
```
#Bean Validation 추가 
NotBlank={0} 공백X 
Range={0}, {2} ~ {1} 허용
Max={0}, 최대 {1}
```


`{0}` 은 필드명이고, `{1}` , `{2}` ...은 각 애노테이션 마다 다르다.


**BeanValidation 메시지 찾는 순서**
1. 생성된 메시지 코드 순서대로 `messageSource` 에서 메시지 찾기
2. 애노테이션의 `message` 속성 사용 `@NotBlank(message = "공백! {0}")`
3. 라이브러리가 제공하는 기본 값 사용 공백일 수 없습니다.
**애노테이션의 message 사용 예** 
```java
@NotBlank(message = "공백은 입력할 수 없습니다.") private String itemName;
```

# Bean Validation - 오브젝트 오류

Bean Validation에서 특정 필드( `FieldError` )가 아닌 해당 오브젝트 관련 오류( `ObjectError` )는 어떻게 처리할 수 있을까?
다음과 같이 `@ScriptAssert()` 를 사용하면 된다.

```java
 @Data
 @ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >=
 10000")
 public class Item {
//...
}
```



그런데 실제 사용해보면 제약이 많고 복잡하다. 그리고 실무에서는 검증 기능이 해당 객체의 범위를 넘어서는 경우들도
종종 등장하는데, 그런 경우 대응이 어렵다.

```java
        //특정 필드가 아닌 복합 룰 검증
        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }
```

그냥 자바 코드로 하자


# Bean Validation - 수정에 적용


이미 Item은 어노테이션으로 validation이 되어 있으므로 오브젝트 에러만 검증하면 된다.

```java
        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }
```


# Bean Validation - 한계
#### 수정시 검증 요구사항

데이터를 등록할 때와 수정할 때는 요구사항이 다를 수 있다.

**등록시 기존 요구사항**

타입 검증
- 가격, 수량에 문자가 들어가면 검증 오류 처리 필드 검증
상품명: 필수, 공백X
- 가격: 1000원 이상, 1백만원 이하 수량: 최대 9999
특정 필드의 범위를 넘어서는 검증
- 가격 * 수량의 합은 10,000원 이상


**수정시 요구사항**
- 등록시에는 `quantity` 수량을 최대 9999까지 등록할 수 있지만 **수정시에는 수량을 무제한으로 변경**할 수 있다.
- 등록시에는 `id` 에 값이 없어도 되지만, **수정시에는 id 값이 필수**이다.

이걸 수정을 위한 어노테이션으로 설정을 바꾸어 버리면 등록에 문제가 생긴다.

# Form 전송 객체 분리 - 소개

위 문제를 해결하기 위해 validation의 groups라는 기능이 있지만, 실무에서는 잘 사용하지 않는다.

그 이유가 다른 곳에 있다. 바로 등록시 폼에서 전달하는 데이터가 `Item` 도메인 객체와 딱 맞지 않기 때문이다.

소위 "Hello World" 예제에서는 폼에서 전달하는 데이터와 `Item` 도메인 객체가 딱 맞는다. 하지만 실무에서는 회원 등록시회원과관련된데이터만전달받는것이아니라,약관정보도추가로받는등 `Item` 과관계없는수많은부가데 이터가 넘어온다.
그래서 보통 `Item` 을 직접 전달받는 것이 아니라, 복잡한 폼의 데이터를 컨트롤러까지 전달할 별도의 객체를 만들어서 전달한다. 예를 들면 `ItemSaveForm` 이라는 폼을 전달받는 전용 객체를 만들어서 `@ModelAttribute` 로 사용한다. 이것을 통해 컨트롤러에서 폼 데이터를 전달 받고, 이후 컨트롤러에서 필요한 데이터를 사용해서 `Item` 을 생성한다.


**폼 데이터 전달에 Item 도메인 객체 사용**
`HTML Form -> Item -> Controller -> Item -> Repository`
- 장점: Item 도메인 객체를 컨트롤러, 리포지토리 까지 직접 전달해서 중간에 Item을 만드는 과정이 없어서 간단하다.
- 단점: 간단한 경우에만 적용할 수 있다. 수정시 검증이 중복될 수 있고, groups를 사용해야 한다.


**폼 데이터 전달을 위한 별도의 객체 사용**
`HTML Form -> ItemSaveForm -> Controller -> Item 생성 -> Repository`
- 장점: 전송하는 폼 데이터가 복잡해도 거기에 맞춘 별도의 폼 객체를 사용해서 데이터를 전달 받을 수 있다. 보통 등록과, 수정용으로 별도의 폼 객체를 만들기 때문에 검증이 중복되지 않는다.
- 단점: 폼 데이터를 기반으로 컨트롤러에서 Item 객체를 생성하는 변환 과정이 추가된다.


수정의 경우 등록과 수정은 완전히 다른 데이터가 넘어온다. 생각해보면 회원 가입시 다루는 데이터와 수정시 다루는 데 이터는 범위에 차이가 있다. 예를 들면 등록시에는 로그인id, 주민번호 등등을 받을 수 있지만, 수정시에는 이런 부분이 빠진다. 그리고 검증 로직도 많이 달라진다. 그래서 `ItemUpdateForm` 이라는 별도의 객체로 데이터를 전달받는 것이 좋다.

`Item` 도메인 객체를 폼 전달 데이터로 사용하고, 그대로 쭉 넘기면 편리하겠지만, 앞에서 설명한 것과 같이 실무에서 는 `Item` 의 데이터만 넘어오는 것이 아니라 무수한 추가 데이터가 넘어온다. 그리고 더 나아가서 `Item` 을 생성하는데
        
필요한 추가 데이터를 데이터베이스나 다른 곳에서 찾아와야 할 수도 있다.

따라서 이렇게 폼 데이터 전달을 위한 별도의 객체를 사용하고, 등록, 수정용 폼 객체를 나누면 등록, 수정이 완전히 분
리되기 때문에 `groups` 를 적용할 일은 드물다.



**Q: 이름은 어떻게 지어야 하나요?**
이름은 의미있게 지으면 된다. `ItemSave` 라고 해도 되고, `ItemSaveForm` , `ItemSaveRequest` ,
`ItemSaveDto` 등으로 사용해도 된다. 중요한 것은 일관성이다.

**Q: 등록, 수정용 뷰 템플릿이 비슷한데 합치는게 좋을까요?**
한 페이지에 그러니까 뷰 템플릿 파일을 등록과 수정을 합치는게 좋을지 고민이 될 수 있다. 각각 장단점이 있으므로 고 민하는게 좋지만, 어설프게 합치면 수 많은 분기문(등록일 때, 수정일 때) 때문에 나중에 유지보수에서 고통을 맛본다. 이런 어설픈 분기문들이 보이기 시작하면 분리해야 할 신호이다.



# Form 전송 객체 분리 - 개발


```java
 @Data
 public class Item {
     private Long id;
     private String itemName;
     private Integer price;
     private Integer quantity;
}
```

위처럼 Item을 다시 Validation을 하지 않고 원상 복구 시킨다.

```java
@Data
public class ItemSaveForm {

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
    @Max(value = 9999)
    private Integer quantity;
}

```

```java
@Data
public class ItemUpdateForm {

    @NotNull
    private Long id;

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    //수정에서는 수량은 자유롭게 변경할 수 있다.
    private Integer quantity;
}

```

그리고 위처럼 등록과 수정에 관한 DTO를 만든다.

```java
    @PostMapping("/add")
    public String addItem(@Validated @ModelAttribute("item") ItemSaveForm form, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

        //특정 필드가 아닌 복합 룰 검증
        if (form.getPrice() != null && form.getQuantity() != null) {
            int resultPrice = form.getPrice() * form.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }

        //검증에 실패하면 다시 입력 폼으로
        if (bindingResult.hasErrors()) {
            log.info("errors={} ", bindingResult);
            return "validation/v4/addForm";
        }

        //성공 로직
        Item item = new Item();
        item.setItemName(form.getItemName());
        item.setPrice(form.getPrice());
        item.setQuantity(form.getQuantity());

        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v4/items/{itemId}";
    }
```

위처럼 등록 컨트롤러를 수정한다.
>```java
 @ModelAttribute("item") ItemSaveForm form,
```
여기서 주의할 점으로는 @ModelAttribute("item")으로 이름을 명시해줘야한다.
자동으로 자료형 이름으로 담기기 때문이다.
또한 DTO에 값을 가져와 세팅할 때는 생성자로 하는 것이 좋다.
위 코드에서는 가독성을 위해 세터로 했다.


```java
    @PostMapping("/{itemId}/edit")
    public String edit(@PathVariable Long itemId, @Validated @ModelAttribute("item") ItemUpdateForm form, BindingResult bindingResult) {

        //특정 필드가 아닌 복합 룰 검증
        if (form.getPrice() != null && form.getQuantity() != null) {
            int resultPrice = form.getPrice() * form.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }

        if (bindingResult.hasErrors()) {
            log.info("errors={}", bindingResult);
            return "validation/v4/editForm";
        }

        Item itemParam = new Item();
        itemParam.setItemName(form.getItemName());
        itemParam.setPrice(form.getPrice());
        itemParam.setQuantity(form.getQuantity());

        itemRepository.update(itemId, itemParam);
        return "redirect:/validation/v4/items/{itemId}";
    }
```

수정도 위처럼 만들 수 있다.

**정리**
Form 전송 객체 분리해서 등록과 수정에 딱 맞는 기능을 구성하고, 검증도 명확히 분리했다.


# Bean Validation - HTTP 메시지 컨버터

`@Valid` , `@Validated` 는 `HttpMessageConverter` ( `@RequestBody` )에도 적용할 수 있다.


>`@ModelAttribute` 는 HTTP 요청 파라미터(URL 쿼리 스트링, POST Form)를 다룰 때 사용한다. `@RequestBody` 는 HTTP Body의 데이터를 객체로 변환할 때 사용한다. 주로 API JSON 요청을 다룰 때 사용
한다.

**ValidationItemApiController**

```java
@Slf4j
@RestController
@RequestMapping("/validation/api/items")
public class ValidationItemApiController {

    @PostMapping("/add")
    public Object addItem(@RequestBody @Validated ItemSaveForm form, 
    BindingResult bindingResult) {

        log.info("API 컨트롤러 호출");

        if (bindingResult.hasErrors()) {
            log.info("검증 오류 발생 errors={}", bindingResult);
            return bindingResult.getAllErrors();
        }

        log.info("성공 로직 실행");
        return form;
    }
}

```

테스트니까 form을 그대로 반환하고 responseBody(RestController)를 통해 JSON 형태로 리턴 된다.

여기서 잘못된 값으로 post 요청하게 되면 컨트롤러가 호출 되지 않고 그냥 에러가 난다.

HttpConveter가 @RequestBody를 보고 JSON을 객체로 못 만들고 있기 때문이다.

아예 잘못된 값이 아닌 검증의 범위를 넘어서는 값을 보내게 되면 리턴한 에러들이 JSON으로 파싱되어 보내지게 된다.

**API의 경우 3가지 경우를 나누어 생각해야 한다.** 
- 성공 요청: 성공
- 실패 요청: JSON을 객체로 생성하는 것 자체가 실패함
- 검증 오류 요청: JSON을 객체로 생성하는 것은 성공했고, 검증에서 실패함

form에서는 객체로 만들 필요 없이 에러 결과를 바로 보내면 되었지만, API에서는 일단 객체를 만들어야하기 때문에 이러한 결과 차이가 있다.
-> 이것은 예외 처리로 해결한다.

`return bindingResult.getAllErrors();` 는 `ObjectError` 와 `FieldError` 를 반환한다. 스프링이 이 객 체를 JSON으로 변환해서 클라이언트에 전달했다. 여기서는 예시로 보여주기 위해서 검증 오류 객체들을 그대로 반환 했다. 실제 개발할 때는 이 객체들을 그대로 사용하지 말고, 필요한 데이터만 뽑아서 별도의 API 스펙을 정의하고 그에 맞는 객체를 만들어서 반환해야 한다.

**@ModelAttribute vs @RequestBody**
HTTP 요청 파리미터를 처리하는 `@ModelAttribute` 는 각각의 필드 단위로 세밀하게 적용된다. 그래서 특정 필드 에 타입이 맞지 않는 오류가 발생해도 나머지 필드는 정상 처리할 수 있었다.
`HttpMessageConverter` 는 `@ModelAttribute` 와 다르게 각각의 필드 단위로 적용되는 것이 아니라, 전체 객체 단위로 적용된다.
따라서 메시지 컨버터의 작동이 성공해서 `ItemSaveForm` 객체를 만들어야 `@Valid` , `@Validated` 가 적용된다.
- `@ModelAttribute` 는 필드 단위로 정교하게 바인딩이 적용된다. 특정 필드가 바인딩 되지 않아도 나머지 필드 는 정상 바인딩 되고, Validator를 사용한 검증도 적용할 수 있다.
- `@RequestBody` 는 HttpMessageConverter 단계에서 JSON 데이터를 객체로 변경하지 못하면 이후 단계 자 체가 진행되지 않고 예외가 발생한다. 컨트롤러도 호출되지 않고, Validator도 적용할 수 없다.

>`HttpMessageConverter` 단계에서 실패하면 예외가 발생한다. 예외 발생시 원하는 모양으로 예외를 처리하는 방법
은 예외 처리 부분에서 다룬다.

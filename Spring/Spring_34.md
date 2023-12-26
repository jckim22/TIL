지금까지 만든 웹 애플리케이션은 폼 입력시 숫자를 문자로 작성하거나해서 검증 오류가 발생하면 오류 화면으로 바로 이동한다. 이렇게 되면 사용자는 처음부터 해당 폼으로 다시 이동해서 입력을 해야 한다. 아마도 이런 서비스라면 사용 자는 금방 떠나버릴 것이다. 웹 서비스는 폼 입력시 오류가 발생하면, 고객이 입력한 데이터를 유지한 상태로 어떤 오류 가 발생했는지 친절하게 알려주어야 한다.

**컨트롤러의 중요한 역할중 하나는 HTTP 요청이 정상인지 검증하는 것이다.** 그리고 정상 로직보다 이런 검증 로직을 잘 개발하는 것이 어쩌면 더 어려울 수 있다.

**참고: 클라이언트 검증, 서버 검증**
- 클라이언트 검증은 조작할 수 있으므로 보안에 취약하다.
- 서버만으로 검증하면, 즉각적인 고객 사용성이 부족해진다.
- 둘을 적절히 섞어서 사용하되, 최종적으로 서버 검증은 필수
- API 방식을 사용하면 API 스펙을 잘 정의해서 검증 오류를 API 응답 결과에 잘 남겨주어야 함


![](https://velog.velcdn.com/images/jckim22/post/96fab8ab-5fbc-443e-826a-99237528fb2e/image.png)

위처럼 수량, 형식 등 검증 로직에 걸리게 되고 고객에게 다시 상품 폼을 보여준 뒤 모델에 담아 놓은 값으로 어떤 값이 잘못되었던건지 고객에게 알려줘야 한다.

# 검증 직접 처리 - 개발
### 상품 등록 검증

```java
    @PostMapping("/add")
    public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes, Model model) {

        //검증 오류 결과를 보관
        Map<String, String> errors = new HashMap<>();

        //검증 로직
        if (!StringUtils.hasText(item.getItemName())) {
            errors.put("itemName", "상품 이름은 필수입니다.");
        }
        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            errors.put("price", "가격은 1,000 ~ 1,000,000 까지 허용합니다.");
        }
        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            errors.put("quantity", "수량은 최대 9,999 까지 허용합니다.");
        }

        //특정 필드가 아닌 복합 룰 검증
        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                errors.put("globalError", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice);
            }
        }

        //검증에 실패하면 다시 입력 폼으로
        if (!errors.isEmpty()) {
            log.info("errors = {} ", errors);
            model.addAttribute("errors", errors);
            return "validation/v1/addForm";
        }

        //성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v1/items/{itemId}";
    }
```

위처럼 해쉬맵에 검증 데이터를 담을 자료구조를 선언하고 오류가 발생한 필드명을 key로 설정한다.

만약 검증에서 오류 메시지가 하나라도 있으면 오류 메시지를 출력하기 위해 `model` 에 `errors` 를 담고, 입력 폼이 있 는 뷰 템플릿으로 보낸다.


이제 잘못 입력되었을 때 다시 입력 폼이 뜰 것이다.
근데 값이 그대로 남아있다.
이유는 @ModelAttribute로 값이 담겨있는 상태에서 다시 호출하기 때문에 값은 그대로 남아있는 것이다.

```html
        <div th:if="${errors?.containsKey('globalError')}">
            <p class="field-error" th:text="${errors['globalError']}">전체 오류 메시지</p>
        </div>

        <div>
            <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
            <input type="text" id="itemName" th:field="*{itemName}"
                   th:class="${errors?.containsKey('itemName')} ? 'form-control field-error' : 'form-control'"
                   class="form-control" placeholder="이름을 입력하세요">
            <div class="field-error" th:if="${errors?.containsKey('itemName')}" th:text="${errors['itemName']}">
                상품명 오류
            </div>
        </div>
        <div>
            <label for="price" th:text="#{label.item.price}">가격</label>
            <input type="text" id="price" th:field="*{price}"
                   th:class="${errors?.containsKey('price')} ? 'form-control field-error' : 'form-control'"
                   class="form-control" placeholder="가격을 입력하세요">
            <div class="field-error" th:if="${errors?.containsKey('price')}" th:text="${errors['price']}">
                가격 오류
            </div>
        </div>

        <div>
            <label for="quantity" th:text="#{label.item.quantity}">수량</label>
            <input type="text" id="quantity" th:field="*{quantity}"
                   th:class="${errors?.containsKey('quantity')} ? 'form-control field-error' : 'form-control'"
                   class="form-control" placeholder="수량을 입력하세요">
            <div class="field-error" th:if="${errors?.containsKey('quantity')}" th:text="${errors['quantity']}">
                수량 오류
            </div>
```

위처럼 타이림프 로직을 수정한다.
errors가 해당 필드의 키를 포함하고 있으면 div 텍스트를 출력한다.
또한 input 태그의 클래스도 if 조건식으로 수정해서 색깔도 변경할 수 있다.

![](https://velog.velcdn.com/images/jckim22/post/6f2786aa-e224-4e84-be30-c96d9ca39775/image.png)

결과적으로 위처럼 검증을 하게된다.

**정리**
- 만약 검증 오류가 발생하면 입력 폼을 다시 보여준다.
- 검증 오류들을 고객에게 친절하게 안내해서 다시 입력할 수 있게 한다.
- 검증 오류가 발생해도 고객이 입력한 데이터가 유지된다.

**남은 문제점**
- 뷰 템플릿에서 중복 처리가 많다. 뭔가 비슷하다.
- 타입 오류 처리가 안된다. `Item` 의 `price` , `quantity` 같은 숫자 필드는 타입이 `Integer` 이므로 문자 타입 으로 설정하는 것이 불가능하다. 숫자 타입에 문자가 들어오면 오류가 발생한다. 그런데 이러한 오류는 스프링 MVC에서 컨트롤러에 진입하기도 전에 예외가 발생하기 때문에, 컨트롤러가 호출되지도 않고, 400 예외가 발생 하면서 오류 페이지를 띄워준다.
- `Item` 의 `price` 에 문자를 입력하는 것 처럼 타입 오류가 발생해도 고객이 입력한 문자를 화면에 남겨야 한다. 만약 컨트롤러가 호출된다고 가정해도 `Item` 의 `price` 는 `Integer` 이므로 문자를 보관할 수가 없다. 결국 문 자는 바인딩이 불가능하므로 고객이 입력한 문자가 사라지게 되고, 고객은 본인이 어떤 내용을 입력해서 오류가 발생했는지 이해하기 어렵다.
- 결국 고객이 입력한 값도 어딘가에 별도로 관리가 되어야 한다.


# BindingResult1
```java
    public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

        //검증 로직
        if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수 입니다."));
        }
        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            bindingResult.addError(new FieldError("item", "price", "가격은 1,000 ~ 1,000,000 까지 허용합니다."));
        }
        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.addError(new FieldError("item", "quantity", "수량은 최대 9,999 까지 허용합니다."));
        }

        //특정 필드가 아닌 복합 룰 검증
        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
            }
        }

        //검증에 실패하면 다시 입력 폼으로
        if (bindingResult.hasErrors()) {
            log.info("errors={} ", bindingResult);
            return "validation/v2/addForm";
        }

        //성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v2/items/{itemId}";
    }

```

```
      <div th:if="${#fields.hasGlobalErrors()}">
            <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">글로벌 오류 메시지</p>
        </div>

        <div>
            <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
            <input type="text" id="itemName" th:field="*{itemName}"
                   th:errorclass="field-error" class="form-control" placeholder="이름을 입력하세요">
            <div class="field-error" th:errors="*{itemName}">
                상품명 오류
            </div>
        </div>
        <div>
            <label for="price" th:text="#{label.item.price}">가격</label>
            <input type="text" id="price" th:field="*{price}"
                   th:errorclass="field-error" class="form-control" placeholder="가격을 입력하세요">
            <div class="field-error" th:errors="*{price}">
                가격 오류
            </div>
        </div>

        <div>
            <label for="quantity" th:text="#{label.item.quantity}">수량</label>
            <input type="text" id="quantity" th:field="*{quantity}"
                   th:errorclass="field-error" class="form-control" placeholder="수량을 입력하세요">
            <div class="field-error" th:errors="*{quantity}">
                수량 오류
            </div>

        </div>
```



필드에 오류가 있으면 `FieldError` 객체를 생성해서 `bindingResult` 에 담아두면 된다.
- `objectName` : `@ModelAttribute` 이름
- `field` : 오류가 발생한 필드 이름
- `defaultMessage` : 오류 기본 메시지


특정 필드를 넘어서는 오류가 있으면 `ObjectError` 객체를 생성해서 `bindingResult` 에 담아두면 된다. 
- `objectName` : `@ModelAttribute` 의 이름
- `defaultMessage` : 오류 기본 메시지

**타임리프 스프링 검증 오류 통합 기능**
타임리프는 스프링의 `BindingResult` 를 활용해서 편리하게 검증 오류를 표현하는 기능을 제공한다.
- `#fields` : `#fields` 로 `BindingResult` 가 제공하는 검증 오류에 접근할 수 있다.
- `th:errors` : 해당 필드에 오류가 있는 경우에 태그를 출력한다. th:if` 의 편의 버전이다. 
- `th:errorclass` : `th:field` 에서 지정한 필드에 오류가 있으면 `class` 정보를 추가한다.

# BindingResult2

#### BindingResult2

- 스프링이 제공하는 검증 오류를 보관하는 객체이다. 검증 오류가 발생하면 여기에 보관하면 된다. `
- BindingResult` 가 있으면 `@ModelAttribute` 에 데이터 바인딩 시 오류가 발생해도 컨트롤러가 호출된다!

** @ModelAttribute에 바인딩 시 타입 오류가 발생하면?**
- `BindingResult` 가 없으면 400 오류가 발생하면서 컨트롤러가 호출되지 않고, 오류 페이지로 이동한다. 
- `BindingResult` 가 있으면 오류 정보( `FieldError` )를 `BindingResult` 에 담아서 컨트롤러를 정상 호출한다.

**BindingResult에 검증 오류를 적용하는 3가지 방법**
- `@ModelAttribute` 의 객체에 타입 오류 등으로 바인딩이 실패하는 경우 스프링이 `FieldError` 생성해서 `BindingResult` 에 넣어준다.
- 개발자가 직접 넣어준다.
- `Validator` 사용 이것은 뒤에서 설명


바인딩 자체가 실패하는 오류와 비즈니스 로직 상에서 생긴 오류 2개가 있다.


**정리**
`BindingResult` , `FieldError` , `ObjectError` 를 사용해서 오류 메시지를 처리하는 방법을 알아보았다.
그런데 오류가 발생하는 경우 고객이 입력한 내용이 모두 사라진다. 이 문제를 해결해보자.


# FieldError, ObjectError
**목표**
- 사용자 입력 오류 메시지가 화면에 남도록 하자.
   - 예) 가격을 1000원 미만으로 설정시 입력한 값이 남아있어야 한다. - `FieldError` , `ObjectError` 에 대해서 더 자세히 알아보자.

```java
//    @PostMapping("/add")
    public String addItemV2(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

        //검증 로직
        if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, null, null, "상품 이름은 필수 입니다."));
        }
        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            bindingResult.addError(new FieldError("item", "price", item.getPrice(), false, null, null, "가격은 1,000 ~ 1,000,000 까지 허용합니다."));
        }
        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.addError(new FieldError("item", "quantity", item.getQuantity(), false, null ,null, "수량은 최대 9,999 까지 허용합니다."));
        }

        //특정 필드가 아닌 복합 룰 검증
        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.addError(new ObjectError("item",null ,null, "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
            }
        }

        //검증에 실패하면 다시 입력 폼으로
        if (bindingResult.hasErrors()) {
            log.info("errors={} ", bindingResult);
            return "validation/v2/addForm";
        }

        //성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v2/items/{itemId}";
    }
```



위처럼 FieldErrolr의 오버로딩된 또 다른 생성자를 사용하면 된다.
3번째 인자로 거절된 값 그러니까 잘못 입력된 값을 넣어주고 binding 에러 여부를 false로 넣어주면 된다.
> 비즈니스 로직에 들어온 것이니 binding은 된 것이기 때문이다.

사용자의 입력 데이터가 컨트롤러의 `@ModelAttribute` 에 바인딩되는 시점에 오류가 발생하면 모델 객체에 사용자 입력 값을 유지하기 어렵다. 예를 들어서 가격에 숫자가 아닌 문자가 입력된다면 가격은 `Integer` 타입이므로 문자를 보관할 수 있는 방법이 없다. 그래서 오류가 발생한 경우 사용자 입력 값을 보관하는 별도의 방법이 필요하다. 그리고 이 렇게 보관한 사용자 입력 값을 검증 오류 발생시 화면에 다시 출력하면 된다.
`FieldError` 는 오류 발생시 사용자 입력 값을 저장하는 기능을 제공한다.
여기서 `rejectedValue` 가 바로 오류 발생시 사용자 입력 값을 저장하는 필드다.
`bindingFailure` 는 타입 오류 같은 바인딩이 실패했는지 여부를 적어주면 된다. 여기서는 바인딩이 실패한 것은 아
니기 때문에 `false` 를 사용한다.

**타임리프의 사용자 입력 값 유지** `th:field="*{price}"`
타임리프의 `th:field` 는 매우 똑똑하게 동작하는데, 정상 상황에는 모델 객체의 값을 사용하지만, 오류가 발생하면 `FieldError` 에서 보관한 값을 사용해서 값을 출력한다.

**스프링의 바인딩 오류 처리**
타입 오류로 바인딩에 실패하면 스프링은 `FieldError` 를 생성하면서 사용자가 입력한 값을 넣어둔다. 그리고 해당 오류를 `BindingResult` 에 담아서 컨트롤러를 호출한다. 따라서 타입 오류 같은 바인딩 실패시에도 사용자의 오류 메 시지를 정상 출력할 수 있다.


> ObjectError도 유사하게 2가지 생성자를 제공하지만, 일단 필드끼리의 조합에 의해 나온 결과기 때문에 바인딩 자체는 성공한 것이다.

# 오류 코드와 메시지 처리1

**목표**
오류 메시지를 체계적으로 다루어보자.
>오류 메시지도 어디서든 일관성이 있는 것이 좋다.


FieldError 파라미터 목록
`objectName` : 오류가 발생한 객체 이름
`field` : 오류 필드
`rejectedValue` : 사용자가 입력한 값(거절된 값)
`bindingFailure` : 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값 
`codes` : 메시지 코드
`arguments` : 메시지에서 사용하는 인자
`defaultMessage` : 기본 오류 메시지

여기서 codes와 arguments를 사용할 수 있다.


```
spring.messages.basename=messages,errors
```

application.properties에서 위처럼 설정을 바꾸어 준다.
그렇게 되면 	errors.properties까지 참조하게 된다.
그리고 errors.properties를 건드려보자.

```
#required.item.itemName=ìí ì´ë¦ì íììëë¤.
#range.item.price=ê°ê²©ì {0} ~ {1} ê¹ì§ íì©í©ëë¤.
#max.item.quantity=ìëì ìµë {0} ê¹ì§ íì©í©ëë¤.
#totalPriceMin=ê°ê²© * ìëì í©ì {0}ì ì´ìì´ì´ì¼ í©ëë¤. íì¬ ê° = {1}

#==ObjectError==
#Level1
totalPriceMin.item=ìíì ê°ê²© * ìëì í©ì {0}ì ì´ìì´ì´ì¼ í©ëë¤. íì¬ ê° = {1}

#Level2 - ìëµ
totalPriceMin=ì ì²´ ê°ê²©ì {0}ì ì´ìì´ì´ì¼ í©ëë¤. íì¬ ê° = {1}


#==FieldError==
#Level1
required.item.itemName=ìí ì´ë¦ì íììëë¤.
range.item.price=ê°ê²©ì {0} ~ {1} ê¹ì§ íì©í©ëë¤.
max.item.quantity=ìëì ìµë {0} ê¹ì§ íì©í©ëë¤.

#Level2 - ìëµ

#Level3
required.java.lang.String = íì ë¬¸ììëë¤.
required.java.lang.Integer = íì ì«ììëë¤.
min.java.lang.String = {0} ì´ìì ë¬¸ìë¥¼ ìë ¥í´ì£¼ì¸ì.
min.java.lang.Integer = {0} ì´ìì ì«ìë¥¼ ìë ¥í´ì£¼ì¸ì.
range.java.lang.String = {0} ~ {1} ê¹ì§ì ë¬¸ìë¥¼ ìë ¥í´ì£¼ì¸ì.
range.java.lang.Integer = {0} ~ {1} ê¹ì§ì ì«ìë¥¼ ìë ¥í´ì£¼ì¸ì.
max.java.lang.String = {0} ê¹ì§ì ì«ìë¥¼ íì©í©ëë¤.
max.java.lang.Integer = {0} ê¹ì§ì ì«ìë¥¼ íì©í©ëë¤.

#Level4
required = íì ê° ìëë¤.
min= {0} ì´ìì´ì´ì¼ í©ëë¤.
range= {0} ~ {1} ë²ìë¥¼ íì©í©ëë¤.
max= {0} ê¹ì§ íì©í©ëë¤.

#ì¶ê°
typeMismatch.java.lang.Integer=ì«ìë¥¼ ìë ¥í´ì£¼ì¸ì.
typeMismatch=íì ì¤ë¥ìëë¤.

#Bean Validation ì¶ê°

#NotBlank.item.itemName=ìí ì´ë¦ì ì ì´ì£¼ì¸ì.

#NotBlank={0} ê³µë°±X
Range={0}, {2} ~ {1} íì©
Max={0}, ìµë {1}
#required.item.itemName=ìí ì´ë¦ì íììëë¤.
#range.item.price=ê°ê²©ì {0} ~ {1} ê¹ì§ íì©í©ëë¤.
#max.item.quantity=ìëì ìµë {0} ê¹ì§ íì©í©ëë¤.
#totalPriceMin=ê°ê²© * ìëì í©ì {0}ì ì´ìì´ì´ì¼ í©ëë¤. íì¬ ê° = {1}

#==ObjectError==
#Level1
totalPriceMin.item=ìíì ê°ê²© * ìëì í©ì {0}ì ì´ìì´ì´ì¼ í©ëë¤. íì¬ ê° = {1}

#Level2 - ìëµ
totalPriceMin=ì ì²´ ê°ê²©ì {0}ì ì´ìì´ì´ì¼ í©ëë¤. íì¬ ê° = {1}


#==FieldError==
#Level1
required.item.itemName=ìí ì´ë¦ì íììëë¤.
range.item.price=ê°ê²©ì {0} ~ {1} ê¹ì§ íì©í©ëë¤.
max.item.quantity=ìëì ìµë {0} ê¹ì§ íì©í©ëë¤.

#Level2 - ìëµ

#Level3
required.java.lang.String = íì ë¬¸ììëë¤.
required.java.lang.Integer = íì ì«ììëë¤.
min.java.lang.String = {0} ì´ìì ë¬¸ìë¥¼ ìë ¥í´ì£¼ì¸ì.
min.java.lang.Integer = {0} ì´ìì ì«ìë¥¼ ìë ¥í´ì£¼ì¸ì.
range.java.lang.String = {0} ~ {1} ê¹ì§ì ë¬¸ìë¥¼ ìë ¥í´ì£¼ì¸ì.
range.java.lang.Integer = {0} ~ {1} ê¹ì§ì ì«ìë¥¼ ìë ¥í´ì£¼ì¸ì.
max.java.lang.String = {0} ê¹ì§ì ì«ìë¥¼ íì©í©ëë¤.
max.java.lang.Integer = {0} ê¹ì§ì ì«ìë¥¼ íì©í©ëë¤.

#Level4
required = íì ê° ìëë¤.
min= {0} ì´ìì´ì´ì¼ í©ëë¤.
range= {0} ~ {1} ë²ìë¥¼ íì©í©ëë¤.
max= {0} ê¹ì§ íì©í©ëë¤.

#ì¶ê°
typeMismatch.java.lang.Integer=ì«ìë¥¼ ìë ¥í´ì£¼ì¸ì.
typeMismatch=íì ì¤ë¥ìëë¤.

#Bean Validation ì¶ê°

#NotBlank.item.itemName=ìí ì´ë¦ì ì ì´ì£¼ì¸ì.

#NotBlank={0} ê³µë°±X
Range={0}, {2} ~ {1} íì©
Max={0}, ìµë {1}

```


```java
        if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, new String[]{"required.item.itemName"}, null, null));
        }
        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            bindingResult.addError(new FieldError("item", "price", item.getPrice(), false, new String[]{"range.item.price"}, new Object[]{1000, 1000000}, null));
        }
        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.addError(new FieldError("item", "quantity", item.getQuantity(), false, new String[]{"max.item.quantity"} ,new Object[]{9999}, null));
        }

```
위처럼 codes 인자에 new Stirng으로 properties의 설정 문장을 읽어오고 argument가 존재하는 문장이라면 arg까지 채워준다.

- `codes` : `required.item.itemName` 를 사용해서 메시지 코드를 지정한다. 메시지 코드는 하나가 아니라 배 열로 여러 값을 전달할 수 있는데, 순서대로 매칭해서 처음 매칭되는 메시지가 사용된다.
- `arguments` : `Object[]{1000, 1000000}` 를 사용해서 코드의 `{0}` , `{1}` 로 치환할 값을 전달한다.


실행해보면 `MessageSource` 를 찾아서 메시지를 조회하는 것을 확인할 수 있다.

# 오류 코드와 메시지 처리2

**목표**
`FieldError` , `ObjectError` 는 다루기 너무 번거롭다.
오류 코드도 좀 더 자동화 할 수 있지 않을까? 예) `item.itemName` 처럼?


컨트롤러에서 `BindingResult` 는 검증해야 할 객체인 `target` 바로 다음에 온다. 따라서 `BindingResult` 는 이 미 본인이 검증해야 할 객체인 `target` 을 알고 있다.


```java
 log.info("objectName={}", bindingResult.getObjectName());
 log.info("target={}", bindingResult.getTarget());
```
위처럼 로그를 찍어보면 첫번째 target 객체가 나온다.

## `rejectValue()` , `reject()`
`BindingResult` 가 제공하는 `rejectValue()` , `reject()` 를 사용하면 `FieldError` , `ObjectError` 를 직
접 생성하지 않고, 깔끔하게 검증 오류를 다룰 수 있다.

```java
        if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.rejectValue("itemName", "required");
        }
        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            bindingResult.rejectValue("price", "range", new Object[]{1000, 10000000}, null);
        }
        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.rejectValue("quantity", "max", new Object[]{9999}, null);
        }

        //특정 필드가 아닌 복합 룰 검증
        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }

        //검증에 실패하면 다시 입력 폼으로
        if (bindingResult.hasErrors()) {
            log.info("errors={} ", bindingResult);
            return "validation/v2/addForm";
        }

        //성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v2/items/{itemId}";
```

오류 메시지가 정상 출력된다. 그런데 `errors.properties` 에 있는 코드를 직접 입력하지 않았는데 어떻게 된것일 까?
**rejectValue()**
```java
 void rejectValue(@Nullable String field, String errorCode,
         @Nullable Object[] errorArgs, @Nullable String defaultMessage);
```
`field` : 오류 필드명
`errorCode` : 오류 코드(이 오류 코드는 메시지에 등록된 코드가 아니다. 뒤에서 설명할 messageResolver를 위한 오류 코드이다.)
`errorArgs` : 오류 메시지에서 `{0}` 을 치환하기 위한 값
`defaultMessage` : 오류 메시지를 찾을 수 없을 때 사용하는 기본 메시지
```java
bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null)
```

내가 생각엔 에러 코드.객체이름.필드의 정보로 조회하는 것 같다.

앞에서 `BindingResult` 는 어떤 객체를 대상으로 검증하는지 target을 이미 알고 있다고 했다. 따라서 target(item)에 대한 정보는 없어도 된다. 오류 필드명은 동일하게 `price` 를 사용했다.

`FieldError()` 를 직접 다룰 때는 오류 코드를 `range.item.price` 와 같이 모두 입력했다. 그런데 `rejectValue()` 를 사용하고 부터는 오류 코드를 `range` 로 간단하게 입력했다. 그래도 오류 메시지를 잘 찾아서 출
력한다. 무언가 규칙이 있는 것 처럼 보인다. 이 부분을 이해하려면 `MessageCodesResolver` 를 이해해야 한다. 왜 이런식으로 오류 코드를 구성하는지 바로 다음에 자세히 알아보자.


# 오류 코드와 메시지 처리
오류 코드를 만들 때 다음과 같이 자세히 만들 수도 있고, 
`required.item.itemName` : 상품 이름은 필수 입니다. 
`range.item.price` : 상품의 가격 범위 오류 입니다.

또는 다음과 같이 단순하게 만들 수도 있다. 
`required` : 필수 값 입니다.
`range` : 범위 오류 입니다.


단순하게 만들면 범용성이 좋아서 여러곳에서 사용할 수 있지만, 메시지를 세밀하게 작성하기 어렵다. 반대로 너무 자세 하게 만들면 범용성이 떨어진다. 가장 좋은 방법은 범용성으로 사용하다가, 세밀하게 작성해야 하는 경우에는 세밀한 내 용이 적용되도록 메시지에 단계를 두는 방법이다.


예를 들어서 `required` 라고 오류 코드를 사용한다고 가정해보자.
다음과 같이 `required` 라는 메시지만 있으면 이 메시지를 선택해서 사용하는 것이다.
```
properties
required: 필수 값 입니다. 
```

              
그런데 오류 메시지에 `required.item.itemName` 와 같이 객체명과 필드명을 조합한 세밀한 메시지 코드가 있으면
 
이 메시지를 높은 우선순위로 사용하는 것이다. 
```properties

#Level1
required.item.itemName: 상품 이름은 필수 입니다.
#Level2
required: 필수 값 입니다. 

```

```java
new String[]{"required.item.itemName", "required"};
```

위처럼 우선순위를 정하는 것이다.
properties에서 첫번째 요소를 찾고 없으면 두번째 더 범용성 있는 문구를 출력하게 될 것이다.

물론 이렇게 객체명과 필드명을 조합한 메시지가 있는지 우선 확인하고, 없으면 좀 더 범용적인 메시지를 선택하도록 추가 개발을 해야겠지만, 범용성 있게 잘 개발해두면, 메시지의 추가 만으로 매우 편리하게 오류 메시지를 관리할 수 있을 것이다.



스프링은 `MessageCodesResolver` 라는 것으로 이러한 기능을 지원한다.

# 오류 코드와 메시지 처리4

### MessageCodesResolver

**동작 방식**
- `rejectValue()` , `reject()` 는 내부에서 `MessageCodesResolver` 를 사용한다. 여기에서 메시지 코드들
을 생성한다.
- `FieldError` , `ObjectError` 의 생성자를 보면, 오류 코드를 하나가 아니라 여러 오류 코드를 가질 수 있다. `MessageCodesResolver` 를 통해서 생성된 순서대로 오류 코드를 보관한다.
- 이 부분을 `BindingResult` 의 로그를 통해서 확인해보자.
>`codes [range.item.price, range.price, range.java.lang.Integer, range]`

**FieldError** `rejectValue("itemName", "required")` 다음 4가지 오류 코드를 자동으로 생성
- `required.item.itemName` 
- `required.itemName`
- `required.java.lang.String` 
- `required`

**ObjectError** `reject("totalPriceMin")` 다음 2가지 오류 코드를 자동으로 생성
- `totalPriceMin.item` 
- `totalPriceMin`

**오류 메시지 출력**
타임리프 화면을 렌더링 할 때 `th:errors` 가 실행된다. 만약 이때 오류가 있다면 생성된 오류 메시지 코드를 순서대 로 돌아가면서 메시지를 찾는다. 그리고 없으면 디폴트 메시지를 출력한다

아무튼

```properties

#Level1
required.item.itemName: 상품 이름은 필수 입니다.
#Level2
required: 필수 값 입니다. 

```

```java
new String[]{"required.item.itemName", "required"};
```
위처럼 디테일 한것부터 우선순위를 잡고 반환한다.

# 오류 코드와 메시지 처리5

### 오류 코드 관리 전략
#### **핵심은 구체적인 것에서! 덜 구체적인 것으로!**

`MessageCodesResolver`는 `required.item.itemName` 처럼 구체적인 것을 먼저 만들어주고, `required` 처럼 덜 구체적인 것을 가장 나중에 만든다.
이렇게 하면 앞서 말한 것 처럼 메시지와 관련된 공통 전략을 편리하게 도입할 수 있다.


**왜 이렇게 복잡하게 사용하는가?**
모든 오류 코드에 대해서 메시지를 각각 다 정의하면 개발자 입장에서 관리하기 너무 힘들다.
크게 중요하지 않은 메시지는 범용성 있는 `requried` 같은 메시지로 끝내고, 정말 중요한 메시지는 꼭 필요할 때 구체적으로 적어서 사용하는 방식이 더 효과적이다.

이제 그 전략을 적용한 properties를 작성해보자
```
#required.item.itemName=상품 이름은 필수입니다. 
#range.item.price=가격은 {0} ~ {1} 까지 허용합니다. 
#max.item.quantity=수량은 최대 {0} 까지 허용합니다. 
#totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다.
현재 값 = {1}
#==ObjectError==
#Level1
totalPriceMin.item=상품의 가격 * 수량의 합은 {0}원 이상이어야 합니다. 
현재 값 = {1}
#Level2 - 생략
totalPriceMin=전체 가격은 {0}원 이상이어야 합니다.
현재 값 = {1}
#==FieldError==
#Level1
required.item.itemName=상품 이름은 필수입니다. 
range.item.price=가격은 {0} ~ {1} 까지 허용합니다. 
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
#Level2 - 생략
#Level3
required.java.lang.String = 필수 문자입니다. 
required.java.lang.Integer = 필수 숫자입니다. 
min.java.lang.String = {0} 이상의 문자를 입력해주세요. 
min.java.lang.Integer = {0} 이상의 숫자를 입력해주세요.
range.java.lang.String = {0} ~ {1} 까지의 문자를 입력해주세요.
range.java.lang.Integer = {0} ~ {1} 까지의 숫자를 입력해주세요. 
max.java.lang.String = {0} 까지의 문자를 허용합니다.
 max.java.lang.Integer = {0} 까지의 숫자를 허용합니다.
#Level4
required = 필수 값 입니다.
min= {0} 이상이어야 합니다.
range= {0} ~ {1} 범위를 허용합니다.
max= {0} 까지 허용합니다.
```

`itemName` 의 경우 `required` 검증 오류 메시지가 발생하면 다음 코드 순서대로 메시지가 생성된다.
1. `required.item.itemName`
2. `required.itemName`
3. `required.java.lang.String`
4. `required`



그리고 이렇게 생성된 메시지 코드를 기반으로 순서대로 `MessageSource` 에서 메시지에서 찾는다.
구체적인 것에서 덜 구체적인 순서대로 찾는다. 메시지에 1번이 없으면 2번을 찾고, 2번이 없으면 3번을 찾는다. 이렇게 되면 만약에 크게 중요하지 않은 오류 메시지는 기존에 정의된 것을 그냥 **재활용** 하면 된다!

>### ValidationUtils
**ValidationUtils 사용 전** 
```java
if (!StringUtils.hasText(item.getItemName())) { 
bindingResult.rejectValue("itemName", "required", 
"기본: 상품 이름은 필수입니다.");
}
```
**ValidationUtils 사용 후**
```java
 ValidationUtils.rejectIfEmptyOrWhitespace(bindingResult, 
 "itemName", "required");
```

**정리**
1. `rejectValue()` 호출
2. `MessageCodesResolver` 를 사용해서 검증 오류 코드로 메시지 코드들을 생성
3. `new FieldError()` 를 생성하면서 메시지 코드들을 보관
4. `th:erros` 에서 메시지 코드들로 메시지를 순서대로 메시지에서 찾고, 노출

# 오류 코드와 메시지 처리6


#### 스프링이 직접 만든 오류 메시지 처리

검증 오류 코드는 다음과 같이 2가지로 나눌 수 있다.
- 개발자가 직접 설정한 오류 코드 `rejectValue()` 를 직접 호출
- 스프링이 직접 검증 오류에 추가한 경우(주로 타입 정보가 맞지 않음)

`price` 필드에 문자 "A"를 입력해보자.
로그를 확인해보면 `BindingResult` 에 `FieldError` 가 담겨있고, 다음과 같은 메시지 코드들이 생성된 것을 확인 할 수 있다. `codes[typeMismatch.item.price,typeMismatch.price,typeMismatch.java.lang.Integer,ty peMismatch]`
다음과 같이 4가지 메시지 코드가 입력되어 있다. 
- `typeMismatch.item.price` 
- `typeMismatch.price` 
- `typeMismatch.java.lang.Integer` 
- `typeMismatch`

그렇다. 스프링은 타입 오류가 발생하면 `typeMismatch` 라는 오류 코드를 사용한다. 이 오류 코드가 `MessageCodesResolver` 를 통하면서 4가지 메시지 코드가 생성된 것이다

실행해도 아직 `errors.properties` 에 메시지 코드가 없기 때문에 스프링이 생성한 기본 메시지가 출력된다.
`Failed to convert property value of type java.lang.String to required type java.lang.Integer for property price; nested exception is java.lang.NumberFormatException: For input string: "A"`

그래서 error.propites에 level3, level4 오류 메시지를 추가해서 직접 설정하면 된다.

```java
typeMismatch.java.lang.Integer=숫자를 입력해주세요.
typeMismatch=타입 오류입니다.
```

**정리**
메시지 코드 생성 전략은 그냥 만들어진 것이 아니다. 조금 뒤에서 Bean Validation을 학습하면 그 진가를 더 확인할 수 있다.

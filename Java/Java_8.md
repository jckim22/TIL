# 🔆java.util.function 패키지

- 자바는 람다식을 위한 함수형 인터페이스가 정의되어 있어야 함
    - 필요할 때마다 정의해야 하므로 번거로움
    - 자주 사용하는 인터페이스가 `java.util.function` 패키지에 제공
    - 스트림에서 유용하게 사용
    
| 함수형 인터페이스 | 메서드 | 인자(들) 타입 | 반환값 타입 |
| --- | --- | --- | --- |
| Runnable (java.lang 패키지) | run |  |  |
| Supplier<T> | get |  | T |
| Consumer<T> | accept | T |  |
| BiConsumer<T, U> | accept | T, U |  |
| Function<T, R> | apply | T | R |
| BiFunction<T, U, R> | apply | T, U | R |
| Predicate<T> | test | T | boolean |
| BiPredicate<T, U> | test | T, U | boolean |
| UnaryOperator<T> | apply | T | T |
| BinaryOperator<T> | apply | T, T | T |
  
  ❗ 그래서 자바에서는 위와 같은 인터페이스들을 미리 구현해두었다.
  외워서 사용해야한다는 불편함이 있다.
  
  그래도 사용할 수 있다면 매우 용이하게 사용할 수 있다.
  아래를 보자

  `☕ Main.java`

```java
				Runnable dogButtonFunc = () -> System.out.println("멍멍");
        Runnable catButtonFunc = () -> System.out.println("야옹");
        Runnable cowButtonFunc = () -> System.out.println("음메");

        dogButtonFunc.run();
        catButtonFunc.run();
        cowButtonFunc.run();
```
위는 Runnable 인터페이스를 람다식으로 사용한 것이다.
`☕ Button.java`

```java
public class Button {
    private String text;
    public Button(String text) { this.text = text; }
    public String getText() { return text; }

    private Runnable onClickListener;
    public void setOnClickListener(Runnable onClickListener) {
        this.onClickListener = onClickListener;
    }
    public void onClick () {
        onClickListener.run();
    }
}
```
  위 코드를 보면 Runable이 필드로 있고 setOnClickListener를 통해 함수를 입력 받을 수 있다.
  

`☕ Main.java`
  ```java
				System.out.println("\n- - - - -\n");

				Button dogButton = new Button("강아지");
        dogButton.setOnClickListener(dogButtonFunc);

        Button catButton = new Button("고양이");
        catButton.setOnClickListener(() -> {
            System.out.println("- - - - -");
            System.out.println(catButton.getText() + " 울음소리: 야옹야옹");
            System.out.println("- - - - -");
        });

        dogButton.onClick();
        catButton.onClick();
```

위 코드로 자세히 보자면 Button 클래스의 인스턴스인 dogButton을 생성하고 setOnClickListenr라는 setter를 통해 위에서 정의했던 Runnable인 dogButtonFunc를 세팅해준다.
또 고양이 인스턴스를 생성해주고 setOnClickListenr의 인자로 람다식을 넣어준다.

그럼 자동으로 람다식을 통해 Runnable 인터페이스의 추상메소드인 run()에 오버라이딩이 된다.
  
onClick() 핸들러를 통해 run() 메서드를 실행할 수 있다.
  
  
출처: [제대로 파는 자바(얄코)](https://www.yalco.kr/)

# 🔆 함수형 프로그래밍이란
명령형 프로그래밍을 기반으로 개발했던 개발자들은 개발하는 소프트웨어의 크기가 커짐에 따라, 복잡하게 엉켜있는 스파게티 코드를 유지보수하는 것이  매우 힘들다는 것을 깨닫게 되었다. 그리고 이를 해결하기 위해 함수형 프로그래밍이라는 프로그래밍 패러다임에 관심을 갖게 되었다. 함수형 프로그래밍은 거의 모든 것을 순수 함수로 나누어 문제를 해결하는 기법으로, 작은 문제를 해결하기 위한 함수를 작성하여 가독성을 높이고 유지보수를 용이하게 해준다.

 
유명한 책인 클린 코드(Clean Code)의 저자 Robert C.Martin은 함수형 프로그래밍을 대입문이 없는 프로그래밍이라고 정의하였다.

 

>Functional Programming is programming without assignment satements
- Rober C.Martin -

 

 

그 동안 명령형 프로그래밍으로 개발을 해왔던 사람들에게 대입문이 없는 프로그래밍은 상당히 생소할 수 밖에 없다. 왜냐하면 다음과 같이 간단한 코드에서도 변수가 할당되고, 값이 대입되기 때문이다.

```java
// 1 ~ 10까지의 값이 i에 할당된다
for(int i = 1 ; i < 10; i++){
    System.out.println(i);
}
```
 

## 🔱이해
앞서 설명하였든 함수형 프로그래밍은 대입문을 사용하지 않는 프로그래밍이며, 작은 문제를 해결하기 위한 함수를 작성한다고 설명하였다. 그렇기 때문에 함수형 프로그래밍에서는 위와 같은 코드를 다음과 같이 해결할 수 있다.

(아래의 코드는 실제 동작 여부와 무관하게 함수형 프로그래밍에 대한 이해를 돕기 위해 작성된 수도 코드이다.)

```java
process(10, print(num));
```
 
process 함수는 첫 번째 인자로 몇까지 iteration을 돌 것인가를 매개변수로 받고 있고, 두 번째 인자로 전달받은 값을 출력하라는 함수를 매개변수로 받고 있다. 앞서 설명하였듯 함수형 프로그래밍은 무엇을(What)에 포커스를 두는 프로그래밍이라고 하였다. 그렇기 때문에 함수형 프로그래밍에서는 '출력을 하는 함수'를 파라미터로 넘길 수 있으며, 이는 함수형 프로그래밍의 기본 원리 中 함수를 1급 시민(First-Class Citizen) 또는 1급 객체(First-Class Object)로 관리하는 특징 때문인데, 이에 대해서는 뒤에서 자세히 설명하도록 하겠다.

 

명령형 프로그래밍에서는 메소드를 호출하면 상황에 따라 내부의 값이 바뀔 수 있다. 즉, 우리가 개발한 함수 내에서 선언된 변수의 메모리에 할당된 값이 바뀌는 등의 변화가 발생할 수 있다. 하지만 함수형 프로그래밍에서는 대입문이 없기 때문에 메모리에 한 번 할당된 값은 새로운 값으로 변할 수 없다.

출처: [망나니 개발자](https://mangkyu.tistory.com/111)

---
# 🔆람다식 *lambda expression*

- 자바8에 추가된 기능
- 메서드를 간략히 식 *expression* 으로 표현
- **익명 함수** *anonymous function* 이라고도 부름
- 자바에서는 인터페이스의 익명 클래스를 간략히 표현하는데 사용됨
    - IntelliJ 기능으로 기존의 코드 람다로 대체해보기
        - `이후 배울 람다` 라고 주석 표시한 코드 검색
            - `sec06/chap04/ex02/Main.java`
            - `sec08/chapa05/ex01/Main.java`
            - ⭐ IDE의 도움을 적극 받아 학습해나갈 것
            

---

# 🔆 함수형 인터페이스 *FunctionalInterface*

- 람다식 형태로 익명 클래스가 만들어질 수 있는 인터페이스
    - 조건 : 추상 메소드가 **하나(만)** 있어야 함
        - 람다식과 1:1로 대응될 수 있어야 하므로
        - `@FunctionalInterface` 로 강제
        - 클레스 메소드와 `default` 메소드는 여럿 있을 수 있음
        - 예외 있음 *(하단에서 다룰 것)*


```java
button1.setOnClickListener(new OnClickListener() {

	@Override
    public void onClick() {
    	System.out.println("줄바꿈");
        System.out.println("커서를 다음 줄에 위치");
   }
});
```
위처럼 익명 클래스가 작성되어 있는 부분을

아래와 같이 람다식으로 표현할 수 있다.
인터페이스를 인식하고 ()로 표현하고 람다식으로 표현하기 위해 인터페이스에 추상 메소드가 단 1개만 필요하다고 했으므로 자동으로 찾아가게 된다.

```java
@FunctionalInterface
button1.setOnClickListener(() -> {
    	System.out.println("줄바꿈");
        System.out.println("커서를 다음 줄에 위치");   
});
```
>❗ @FunctionalInterface 어노테이션으로 만들어진 인터페이스는 그 자체로 일급 객체인 함수가 된다.
❗ 자바에서는 독립적으로 존재할 수 있는 함수가 없기 때문에 (메서드 이기 때문) 인터페이스를 이용한다.

그냥 인터페이스가 오버라이딩된 것 이지만, 변수에 함수가 들어간 것을 표현한 자바만의 람다식 방식이다.

```java
				Returner returner1 = () -> {return 1;};
        Returner returner2 = () -> "반환 코드만 있을 시 { }와 return 생략가능";

        var returned1 = returner1.returnObj();
        var returned2 = returner2.returnObj();
```

❗ 위처럼 반환만 할 시 return이 생략 가능하다.

```java
				SingleParam square = (i) -> i * i;
        SingleParam cube = i -> i * i * i; // 인자가 하나일 시 괄호 생략 가능

        var _3_squared = square.func(3);
        var _3_cubed = cube.func(3);
```
❗ 인자가 하나일 시 위처럼 괄호도 생략 가능하다.
❗ 코드가 한 줄일 경우 중괄호도 생략 가능하다.


```java
				DoubleParam add = (a, b) -> a + b;
        DoubleParam multAndPrint = (a, b) -> {
            var result = a * b;
            System.out.printf("%d * %d = %d%n", a, b, result);
            return result;
        };

        var added = add.func(2, 3);
        var multiplied = multAndPrint.func(2, 3);
```
❗ 조합해보면 위와 같은 람다식들을 표현할 수 있다.


# 🔆 메소드 참조
아래 코드를 보자.

`☕ Main.java`

```java
				//  클래스의 클래스 메소드에 인자 적용하여 실행
        Function<Integer, String> intToStrLD = (i) -> String.valueOf(i);
        Function<Integer, String> intToStrMR = String::valueOf;
        var intToStr = intToStrMR.apply(123);
```
위의 윗줄 코드를 보면 인터페이스의 추상 메소드를 String.valueOf로 오버라이딩 해주는 경우를 보여준다.
이렇게 되면 intToStrLd 메서드는 사실상 String.valueOf나 다름 없는 기능하게 된다.
스위치를 누르는 기계의 스위치를 누르는 사람이나 스위치를 누르는 사람이나 똑같은 일을 하게 된다.

그럴 때는 intToStrMr = String::valueOf  형식으로 메소드 참조를 하면 된다.

> (인터페이스의 추상 메소드) = (메서드)

⭐  IDE의 자동 수정 기능을 적극 활용할 것
    
```java
				//  인자로 받은 인스턴스의 메소드 실행
        UnaryOperator<String> toLCaseLD = s -> s.toLowerCase();
        UnaryOperator<String> toLCaseMR = String::toLowerCase;
        var toLCase = toLCaseMR.apply("HELLO");
```

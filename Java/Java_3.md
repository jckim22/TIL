# 🔆Object 클래스
- 필드 없이 메소드들만 갖고 있음
    - 모든 클래스들에 상속됨
    - 필요에 따라 오버라이드하여 사용

- `Object` 인스턴스 선언하여 클래스 살펴볼 것
    - `@IntrinsicCandidate` : HotSpot VM *(현재 대다수 JVM)* 에 의한 최적화
        - 작성된 코드를 보다 효율적인 내부적 동작으로 덮어씀
    - `native` : C, C++ 등 다른 언어로 작성된 코드를 호출하여 성능 향상
        - Java Natice Interface 사용
        
  >toString, equals, hashCode, clone 등 오버라이드 해보면서 사용

# 🔆Wrapper 클래스
- 각 원시 자료형에는 그에 해당하는 **래퍼 클래스**가 있음
    - 해당 자료형에 관련된 클래스/인스턴스 기능들을 제공
    - 클래스 인스턴스를 받는 곳에 활용
        - 다음 강에서 배울 제내릭 등…
- 각 자료형의 원시값은 해당 래퍼 클래스의 인스턴스와 서로 변환 가능
- 💡 원시값의 존재 이유 : 더 높은 성능
    - 대신 순수한 객체지향 언어는 아니게 됨…
    
```java
				//  원시 자료형
        int int1 = 123;
        double dbl1 = 3.14;
        char chr1 = 'A';
        boolean bln1 = true;

        //  ⭐ 해당 래퍼 클래스의 인스턴스
        //  기존의 생성자 방식
        //  ⚠️ 오늘날에는 deprecated - 성능상 좋지 않음
        //  @Deprecated가 생김(안 쓸거다)
        Integer int2 = new Integer(123);
        Double dbl2 = new Double(3.14);
        Character chr2 = new Character('A');
        Boolean bln2 = new Boolean(true);

        //  💡 아래의 클래스 메소드들이 권장됨
        //  이렇게 써라
        Integer int3 = Integer.valueOf(123);
        Double dbl3 = Double.valueOf(3.14);
        Character chr3 = Character.valueOf('A');
        Boolean bln3 = Boolean.valueOf(true);
```

# 🔆박싱과 언박싱
- 원시값을 래퍼 클래스의 인스턴스로 **boxing**
- 래퍼 클래스의 인스턴스를 원시값으로 **unboxing**

```java
				//  💡 박싱 : 원시값을 래퍼 클래스의 인스턴스로
        //  ⭐ 과거에는 생성자를 사용했으나 deprecated
        int intPrim1 = 123;
        Integer intInst1 = Integer.valueOf(intPrim1);

        char chrPrim1 = 'A';
        Character chrInst1 = Character.valueOf(chrPrim1);

        //  💡 언박싱 : 래퍼 클래스의 인스턴스를 원시값으로
        Double dblInst1 = Double.valueOf(3.14);
        double dblPrim1 = dblInst1.doubleValue();

        Boolean blnInst1 = Boolean.valueOf(true);
        boolean blnPrim1 = blnInst1.booleanValue();
```

> ❗ 자바에서는 auto 언박싱이  됨
원시값인지 래퍼 클래스의 인스턴스인지 신경쓰지않아도 됨

# 🔆 래퍼 클래스의 유용한 메서드

```java
				//  💡 숫자 클래스 메소드들

        //  CharSequence로부터 인스턴스 반환
        //  ⭐ CharSequence : String 등이 구현하는 인터페이스
        Integer int1 = Integer.valueOf("123"); // 문자열로부터 인스턴스 반환

        //  CharSequence로부터 원시값 반환
        //  💡 다른 숫자, 불리언 래퍼 자료형들에도 존재 (parseDouble, parseBoolean...)
        int int2 = Integer.parseInt("123"); // 원시값 반환

        //  parseInt(CharSequence, 진수)
        //  정수 자료형들에만 존재
        //  ⭐ CharSequence : String 등이 구현하는 인터페이스
        int int_123_oct = Integer.parseInt("123", 8);
        int int_123_dec = Integer.parseInt("123", 10);
        int int_123_hex = Integer.parseInt("123", 16);

        //  parseInt(CharSequence, 시작위치, 끝위치, 진수)
        int int3 = Integer.parseInt("1234567", 3, 5, 10);
```

```java
				//  💡 문자 클래스 메소드들

        String strSample = "Ab가1 .";
        for (var i = 0; i < strSample.length(); i++) {
            Character c = strSample.charAt(i);
            System.out.printf(
                    "[%c] : L: %b, U: %b, L: %b, D: %b, S: %b%n",
                    c,
                    Character.isLetter(c),
                    Character.isUpperCase(c),
                    Character.isLowerCase(c),
                    Character.isDigit(c),
                    Character.isSpaceChar(c)
            );
        }
```

```java
				//  💡 인스턴스 메소드들

        //  문자열 반환 (Object에서 오버라이드)
        String intStr = int1.toString();
        String dblStr = Double.valueOf(3.14).toString();
        String blnStr = ((Boolean) false).toString();
        String chrStr = new Character('A').toString();
```


```java
				//  인스턴스끼리의 value 비교
        Integer intA = 12345;
        Integer intB = 12345;

        boolean compByOp1 = intA == intB; // ⚠️ 값은 같으나 다른 참조
        boolean compByEq1 = intA.equals(intB);

        Short srtA = 12345;

        //  ⚠️ 자료형이 다르면 false 반환 (메소드 코드 확인)
        boolean compByOp2 = intA.equals(srtA);
```

```java
				//  숫자 자료형 간 변환 - Number의 추상 메소드들

				Byte int1Byt = int1.byteValue();
        Double int1Dbl = int1.doubleValue();

        Integer int4 = 123456789;
        Byte int4Byt = int4.byteValue(); // ⚠️ 자료형보다 값이 큼

        Float flt1 = 1234.5678f;
        Integer flt1Int = flt1.intValue(); // ⚠️ 소수점 이하 버림
        Short int1DblSrt = int1Dbl.shortValue();
```
# 🔆 제너릭

- 자료형을 필요에 따라 동적으로 정할 수 있도록 해 줌
    - 자료형을 변수로 갖는다고 이해
- 메소드 또는 클래스에 사용

## 🔱 제너릭 메소드

```java
		//  제네릭 메소드
		//  T : 타입변수. 원하는 어떤 이름으로든 명명 가능
		public static <T> T pickRandom (T a, T b) {
        return Math.random() > 0.5 ? a : b;
    }
```

```java
				int randNum = pickRandom(123, 456);
        boolean randBool = pickRandom(true, false);
        String randStr = pickRandom("마루치", "아라치");

				//  import sec05.chap08.ex01.YalcoChicken;
        YalcoChicken store1 = new YalcoChicken("판교");
        YalcoChicken store2 = new YalcoChicken("역삼");
        YalcoChicken randStore = pickRandom(store1, store2);

        //  ⚠️ 타입이 일관되지 않고 묵시적 변환 불가하면 오류
        //  double randFlt = pickRandom("hello", "world");
        double randDbl = pickRandom(12, 34);
```

```java
		public static <T> void arraySwap (T[] array, int a, int b) {
        if (array.length <= Math.max(a, b)) return;
        T temp = array[a];
        array[a] = array[b];
        array[b] = temp;
    }
```

```java
				//  원시값 배열(double[])을 쓰면 오류 - 배열로는 오토박싱이 안 되므로
        var array1 = new Double[] {
                1.2, 2.3, 3.4, 4.5, 5.6, 6.7, 7.8
        };
        var array2 = new Character[] {
                'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K'
        };

        arraySwap(array1, 3, 5);
        arraySwap(array2, 2, 7);
```

```java
				// 셔플
        for (var i = 0; i < 100; i++) {
            arraySwap(
                    array2,
                    (int) Math.floor(Math.random() * array2.length),
                    (int) Math.floor(Math.random() * array2.length)
            );
        }
```

## 🔱제너릭 클래스

```java
//  원하는 자료형들로 세 개의 필드를 갖는 클래스
public class Pocket<T1, T2, T3> {
    private T1 fieldA;
    private T2 fieldB;
    private T3 fieldC;

    public Pocket(T1 fieldA, T2 fieldB, T3 fieldC) {
        this.fieldA = fieldA;
        this.fieldB = fieldB;
        this.fieldC = fieldC;
    }

    public T1 getFieldA() {
        return fieldA;
    }

    public T2 getFieldB() {
        return fieldB;
    }

    public T3 getFieldC() {
        return fieldC;
    }
}
```

```java
				//  선언시 아래와 같이 자료형에 각 타입변수의 자료형을 명시
        //  - 제내릭에는 원시값이 아닌 클래만 사용 가능
				//  - (래퍼 클래스의 또 다른 존재 이유)
        Pocket<Double, Double, Double> size3d1 =
                new Pocket<>(123.45, 234.56, 345.67);

        //  타입추론도 가능은 함
        var size3d2 =
                new Pocket<>(123.45, 234.56, 345.67);

        double width = size3d1.getFieldA();
        double height = size3d1.getFieldB();
        double depth = size3d1.getFieldC();

        Pocket<String, Integer, Boolean> person =
                new Pocket<>("홍길동", 20, false);

        //  제네릭 클래스는 배열 생성시 new로 초기화 필수
        Pocket<String, Integer, Boolean>[] people = new Pocket[] {
                new Pocket<>("홍길동", 20, false),
                new Pocket<>("전우치", 30, true),
                new Pocket<>("임꺽정", 27, true),
        };
```

원하는 자료형의 인스턴스 변수를 생성할 수 있다.

## 🔱제한된 제너릭

```java
		//  💡 T는 Number를 상속한 클래스이어야 한다는 조건
    public static <T extends Number> double add2Num(T a, T b) {
        return a.doubleValue() + b.doubleValue();
    }   
```
```java
				var sum1 = add2Num(12, 34.56);
        var sum2 = add2Num("1" + true); // ⚠️ 불가
```
>❓ 그냥 Number를 인자 자료형으로 하면 되지 않을까?

```java
		//  ⭐ 상속받는 클래스와 구현하는 인터페이스(들)을 함께 조건으로
    //  여기서는 클래스와 인터페이스 모두 extends 뒤에 &로 나열
    public static <T extends Mamal & Hunter & Swimmer>
    void descHuntingMamal (T animal)  {
        //  ⭐️ 조건에 해당하는 필드와 메소드 사용 가능
        System.out.printf("겨울잠 %s%n", animal.hibernation ? "잠" : "자지 않음");
        animal.hunt();
    }

    public static <T extends Flyer & Hunter>
    void descFlyingHunter (T animal) {
        animal.fly();
        animal.hunt();
    }
```

```java
				descHuntingMamal(new PolarBear());
        descHuntingMamal(new GlidingLizard()); // ⚠️ 불가

        descFlyingHunter(new Eagle());
        descFlyingHunter(new GlidingLizard());
        descFlyingHunter(new PolarBear()); // ⚠️ 불가
```

>❗ 제너릭에 조건을 부여하면 이렇게 구체적인 조건들을 지정할 수 있다.
여기서는 인터페이스도 extends로 포함한다.


## 🔱 와일드 카드와 다형성

```java
public class Unit {}
```

```java
public class Knight extends Unit {}
```

```java
public class MagicKnight extends Knight {}
```

```java
public class Horse<T extends Unit> {
    private T rider;

    public void setRider(T rider) {
        this.rider = rider;
    }
}
```

```java
				//  아무 유닛이나 태우는 말
        Horse<Unit> avante = new Horse<>(); // ⭐️ Horse<Unit>에서 Unit 생략
        avante.setRider(new Unit());
        avante.setRider(new Knight());
        avante.setRider(new MagicKnight());

        //  기사 계급 이상을 태우는 말
        Horse<Knight> sonata = new Horse<>(); // Knight 생략
        sonata.setRider(new Unit()); // ⚠️ 불가
        sonata.setRider(new Knight());
        sonata.setRider(new MagicKnight());

        //  마법기사만 태우는 말
        Horse<MagicKnight> grandeur = new Horse<>();
        grandeur.setRider(new Unit()); // ⚠️ 불가
        grandeur.setRider(new Knight()); // ⚠️ 불가
        grandeur.setRider(new MagicKnight());
```

```java
				//  ⚠️ 자료형과 제네릭 타입이 일치하지 않으면 대입 불가
        //  - 제네릭 타입이 상속관계에 있어도 마찬가지
        Horse<Unit> wrongHorse1 = new Horse<Knight>();
        Horse<Knight> wrongHorse2 = new Horse<Unit>();
        avante = sonata;
        sonata = grandeur;
```
#### 중요❗
>제너릭 타입이 상속관계에 있더라도 자료형과 제너릭 타입이 일치하지 않으면 대입이 불가하다.
avante(아무나 태울 수 있는 말)에 sonata를 태울 수는 없다.

>위 문제를 해결하기 위해 와일드카드가 존재한다.
ex) <? extends Knight>
ex) <? super Knight>

```java
				//  ⭐️ 와일드카드 - 제네릭 타입의 다형성을 위함

        //  💡 Knight과 그 자식 클래스만 받을 수 있음
        //  기사 계급 이상을 태우는 말 이상만 대입할 받을 수 있는 변수
        Horse<? extends Knight> knightHorse;
        knightHorse = new Horse<Unit>(); // ⚠️ 불가
        knightHorse = new Horse<Knight>();
        knightHorse = new Horse<MagicKnight>();
        knightHorse = avante; // ⚠️ 불가
        knightHorse = sonata;
        knightHorse = grandeur;
```

```java
				//  💡 Knight과 그 조상 클래스만 받을 수 있음
        //  마법기사만 태우는 말은 받지 않는 변수
        Horse <? super Knight> nonLuxuryHorse;
        nonLuxuryHorse = avante;
        nonLuxuryHorse = sonata;
        nonLuxuryHorse = grandeur; // 불가
```
?만 붙이면 어떤 타입이든 받을 수 있다.

```java
				//  💡 제한 없음 - <? extends Object>와 동일
        //  어떤 말이든 받는 변수
        Horse<?> anyHorse;
        anyHorse = avante;
        anyHorse = sonata;
        anyHorse = grandeur;
```

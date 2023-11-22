# 🔆블록과 스코프
## 🔱블록
{
}
이 단위이다.

## 🔱스코프
어떤 변수가 사용될 수 있는 범위이다.

```java
public class Ex02 {
    public static void main(String[] args) {
        System.out.println(a); // ⚠️ 클래스 메소드에서 인스턴스 필드 사용 불가
    }

    private String y = x; // ⚠️ 클래스 내 필드의 스코프 : 해당 클래스 안
    private int a = 1;
    private int b = a + 1;
    private int c = d + 1; // ⚠️ 메소드 내 변수의 스코프 : 해당 메소드 안

    public void func1 () {
        System.out.println(a + b);
        int d = 2;
    }

    public void func2 () {
        System.out.println(d); // ⚠️
    }
}
```
❗ 보통은 위처럼 클래스 내 필드의 스코프는 클래스 안이기 때문에 메소드에서 사용할 수 있다.
하지만 클래스 메소드에서는 인스턴스 변수를 사용할 수 없다.

❗ 메서드 안에 변수의 스코프는 메서드 안이기 때문에 당연히 블록 밖에서 사용할 수 없다.

#### Tip❗
>Java에서는 재정의해서 사용할 수 없다

#### ex) 
```java
				String str = "바깥쪽";
        {
            String str = "안쪽"; // ⚠️ 불가
        }

        while (true) {
            String str = "안쪽"; // ⚠️ 불가
        }
```
위처럼 블록 밖에 있는 변수를 안에서 다시 선언할 수 없는데


```java
public class Ex04 {

    public static void main(String[] args) {
        new Ex04().printKings();
    }

    String king = "사자";

    void printKings () {
        String king = "여우"; // 💡 그럼 이건 뭔가요??

        //  ⭐️ 인스턴스의 필드는 다른 영역으로 간주
        System.out.printf(
                "인스턴스의 왕은 %s, 블록의 왕은 %s%n",
                this.king, king
        );
    }
}
```
위 같은 상황에서는 다르다.

>❗ String king은 클래스의 인스턴스 변수이고 printKings 메서드 안에 String king은 메서드 안에 변수이기 때문에 다른 영역으로 간주한다.

# 🔆패키지

## 🔱 패키지 사용 이유
- 자바 프로젝트의 디렉토리(폴더) - 패키지로 불리게 됨
    - 일정 규모 이상의 프로그램을 적절히 모듈화
    - 패키지 정보: 클래스의 구성요소 중 하나
- 클래스명의 중복을 피하기 위해 사용
    - 예: `Button` 클래스 - JRE의 동명 클래스 등 확인
- 빌드의 결과도 패키지의 구조를 따름
    - 이전 예제들의 `out` 폴더 확인할 것

## 🔱접근 제어자
private -> 자식 클래스가 상속은 되지만 사용할 수 없는 변수
default(인스턴스 변수) -> 건들 수 없다.
protected -> 자식 클래스에서 사용가능

### ❗ 다른 패키지에서는 ?
import를 해줘야한다.

>import를 통해 다른 패키지의 클래스도 상속을 받을 수 있다.

#### Tip❗
>와일드 카드인 \*인 사용하여 import하면 편하다.

### ❗ 패키지에서 컴파일

각 패키지에서 빌드하면 패키지를 불러오지 못하기 때문에 src 디렉토리(import 되어 있는 모든 패키지가 해당되는 경로)에서 빌드해야한다.


# 🔆리팩토링
파일의 위치가 바뀌게 되면 그에 맞는 패키지로 변경되어야함

# 🔆프로젝트 패키지명 작명

- 인텔리제이에서 새 프로젝트 - Maven 또는 Gradle - 고급 옵션에서 예시
- 본인/또는 회사의 도메인 *(있을 경우)* 권장
    - `kr.yalco.calculator`
        - *한국에 있는 얄코란 사람/회사가 만든 계산기 프로그램*
        - 규모, 주체, 용도 등을 파악 가능
        - 다른 프로젝트들에 사용될 시…
- `java`, `javax` 가 맨 앞에 올 수 없음 *(JRE 라이브러리와 중복)*
    - `src` 폴더 안에 만들고 클래스 넣어 확인해 볼 것
- 식별자 명명 규칙 따름
    - 소문자로 시작하는 것이 컨벤션

# 🔆java.lang 패키지
>❗ import 필요 x


# 🔆내부 클래스

- 다른 클래스 안에 선언되는 클래스
- 크게 4 종류가 있음
    - 멤버 인스턴스
    - 정적 내부 클래스
    - 메소드 안에 정의된 클래스
    - 익명 클래스
    
```java
public class Outer {
    private String inst = "인스턴스";
    private static String sttc = "클래스";

    //  💡 1. 멤버 인스턴스 클래스
    class InnerInstMember {
        //  ⭐️ 외부 클래스의 필드와 클래스 접근 가능
        private String name = inst + " 필드로서의 클래스";
        private InnerSttcMember innerSttcMember = new InnerSttcMember();

        public void func () {
            System.out.println(name);
        }
    }

    //  💡 2. 정적(클래스) 내부 클래스
    //  ⭐️  내부 클래스에도 접근자 사용 가능. private으로 바꿔 볼 것
    public static class InnerSttcMember {
        //  ⭐️ 외부 클래스의 클래스 필드만 접근 가능
        private String name = sttc + " 필드로서의 클래스";

        //  ⚠️ static이 아닌 멤버 인스턴스 클래스에도 접근 불가!
        //  private InnerInstMember innerInstMember = new InnerInstMember();

        public void func () {
            // ⚠️ 인스턴스 메소드지만 클래스가 정적(클래스의)이므로 인스턴스 필드 접근 불가
            //  name += inst;
            System.out.println(name);
        }
    }

    public void memberFunc () {
        //  💡 3. 메소드 안에 정의된 클래스
        //  스코프가 메소드 내로 제한됨
        class MethodMember {
            //  외부의 필드와 클래스에 접근은 가능
            String instSttc = inst + " " + sttc;
            InnerInstMember innerInstMember = new InnerInstMember();
            InnerSttcMember innerSttcMember = new InnerSttcMember();

            public void func () {
                innerInstMember.func();
                innerSttcMember.func();
                System.out.println("메소드 안의 클래스");

                //  new Outer().memberFunc(); // ⚠️ 스택오버플로우 에러!!
            }
        }

        new MethodMember().func();
    }

    public void innerFuncs () {
        new InnerInstMember().func();
        new InnerSttcMember().func();
    }

    public InnerInstMember getInnerInstMember () {
        return new InnerInstMember();
    }
}
```
    
위처럼 static은 static만 가능하다.
❗ 스택오버플로우(무한 재귀) 조심 !

```java
				//  ⭐️ 클래스가 클래스 필드인 것 - 아래의 변수는 인스턴스
        Outer.InnerSttcMember staticMember = new Outer.InnerSttcMember();
        staticMember.func();

        System.out.println("\n- - - - -\n");

        Outer outer = new Outer();
        outer.innerFuncs();

        System.out.println("\n- - - - -\n");


        //  ⚠️  아래와 같은 사용은 불가
        //  Outer.InnerInstMember innerInstMember = new outer.InnerInstMember();

        //  💡 인스턴스 내부 클래스는 이렇게 얻을 수 있음
        Outer.InnerInstMember innerInstMember = outer.getInnerInstMember();
        innerInstMember.func();

        System.out.println("\n- - - - -\n");

        outer.memberFunc();
```
위처럼 메소드 안에서 class를 선언할 수 있다.

>❗ 인스턴스 클래스는 main함수에서 Outer 인스턴스를 생성하고 직접 접근할 수 없기 때문에 getter이용해서 inner 인스턴스를 생성할 수 있다.

>❗ 정적(Static) 클래스는 인스턴스를 생성하지 않고 바로 inner 클래스의 인스턴스를 생성할 수 있다.

## 🔱내부 클래스는 왜 쓰이는가 ?
- 보다 강력한 캡슐화
    - 외부/내부 클래스간의 관계가 긴밀할 때 사용
- 적절히 사용시 유지보수가 용이하고 가독성을 높여줌
    - 과하게 사용되면 클래스 비대화
    
#### ❗ 아래 예제로 더 쉽게 보자

```java
public class YalcoChicken {
    private String name;
    public YalcoChicken (String name) {
        this.name = name;
    }

    //  매장신설 TF팀 - 본사에서 창설
    public static class LaunchTF {
        private String title;
        private String leader;

        public LaunchTF(String title, String leader) {
            this.title = title;
            this.leader = leader;
        }

        public YalcoChicken launch () {
						//  ⚠️ 인스턴스 필드는 사용 불가
            //  System.out.println(this.name);
            return new YalcoChicken(title);
        }
    }

    //  매장마다의 기프트 - 매장에서 배부
    class Gift {
        private String message;

        public Gift(String to) {
            this.message = "From 얄코치킨 %s점 to %s님"
                    .formatted(name, to);
        }
    }

    public Gift getGift (String to) {
        return new Gift(to);
    }
}
```

```java
				YalcoChicken.LaunchTF launchTF1 = new YalcoChicken.LaunchTF("마산", "김영희");
        YalcoChicken.LaunchTF launchTF2 = new YalcoChicken.LaunchTF("영월", "이철수");

        YalcoChicken store1 = launchTF1.launch();
        YalcoChicken store2 = launchTF2.launch();

        YalcoChicken.Gift [] gifts = {
                store1.getGift("홍길동"),
                store1.getGift("전우치"),
                store2.getGift("옥동자")
        };
```
정적 내부 클래스는 본사에서 관리하는 TF팀이기 때문에 본사에서 바로 인스턴스를 생성할 수 있다.

인스턴스 내부 클래스는 매장에서 관리하기 때문에 인스턴스를 생성한 후 따로 생성할 수 있다.


# 🔆익명 클래스
- 다른 클래스나 인터페이스로부터 상속받아 만들어짐
    - 주로 오버라이드한 메소드를 사용
- 한 번만 사용되고 버려질 클래스
    - 따로 클래스명이 부여되지 않음
    - 이후 다시 인스턴스를 생성할 필요가 없으므로
- 이후 배울 람다식이 나오기 전 널리 사용

```java
				//  💡 와일드카드로 임포트 - import sec05.chap08.ex01.*;

        YalcoGroup store1 = new YalcoChicken("울산");
        YalcoGroup store2 = new YalcoCafe("창원", true);

        YalcoGroup store3 = new YalcoGroup (1, "포항") {
            
						//  원본 메소드에 public 추가
						@Override
            public void takeOrder() {
                System.out.printf(
                        "유일한 얄코과메기 %s 과메기를 주문해주세요.%n",
                        super.intro()
                );
            }

            public void dryFish () {
                System.out.println("과메기 말리기");
            }
        };

        var store3Intro = store3.intro();
        store3.takeOrder();
        //store3.dryFish // ⚠️ 불가

        System.out.println("\n- - - - -\n");

        for (var store : new YalcoGroup[] {store1, store2, store3}) {
            store.takeOrder();
        }
```
위 코드를 보면 카페 브랜드, 카페 치킨은 여러 매장으로 재사용성이 있다.
허나 과메기 매장 브랜드를 만들어서 매장을 1개만 오픈하고 싶은데 저런 브랜드화를 해버리면 좀 아깝다는 생각이 들 수 있다.

그래서 위 코드처럼 추상 클래스에 옆에 익명 클래스를 재정의하여 1회성으로 사용할 수 있다.

### 🔱 그래서 언제 사용되는지 ?

계산기를 예를 들었을 때 +나 - 같은 버튼들이 너무 일회성이라고 생각이 될 수 있다.

대표적으로 아래 안드로이드 개발을 할 때 예시를 보자.
```java
public interface OnClickListener {
    void onClick ();
}
```

```java
public class Button {
    String name;
    public Button(String name) {
        this.name = name;
    }

		//  ⭐️ 인터페이스를 상속한 클래스 자료형
    private OnClickListener onClickListener;
    public void setOnClickListener(OnClickListener onClickListener) {
        this.onClickListener = onClickListener;
    }

    public void func () {
        onClickListener.onClick();
    }
}
```

```java
        Button button1 = new Button("Enter");
        Button button2 = new Button("CapsLock");
        Button button3 = new Button("ShutDown");

				//  ⭐️ IDE에서 회색으로 표시 : 이후 배울 람다로 대체
        button1.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick() {
                System.out.println("줄바꿈");
                System.out.println("커서를 다음 줄에 위치");
            }
        });

        button2.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick() {
                System.out.println("기본입력 대소문자 전환");
            }
        });

        button3.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick() {
                System.out.println("작업 자동 저장");
                System.out.println("프로그램 종료");
            }
        });

        for (var button : new Button[] {button1, button2, button3}) {
            button.func();
        }
```

인터페이스를 상속받은 익명 클래스를 각 Button에 넣어준다.
+는 일회성이므로 익명 클래스로 세팅해서 button1에 넣어준다.

# 🔆열거형
❗ mode같은 변수는 mode가 여러개 일 때 열거형을 사용하는 것이 좋다.

인텔리제이에서 enum을 생성할 수 있다.
```java
public enum ButtonMode{
	LIGHT,DARK,GRAY
}
```
>innerclass처럼 사용할 수 있다.

## 🔱enum의 추가적인 기능
```java
public enum YalcoChickenMenu {
    FR("후라이드", 10000, 0),
    YN("양념치킨", 12000, 1),
    GJ("간장치킨", 12000, 0),
    RS("로제치킨", 14000, 0),
    PP("땡초치킨", 13000, 2),
    XX("폭렬치킨", 13000, 3);

    private String name;
    private int price;
    private int spicyLevel;

    YalcoChickenMenu(String name, int price, int spicyLevel) {
        this.name = name;
        this.price = price;
        this.spicyLevel = spicyLevel;
    }

    public String getName() { return name; }
    public int getPrice() { return price; }

		public void setPrice(int price) {
        this.price = price;
    }

    public String getDesc () {
        String peppers = "";
        if (spicyLevel > 0) {
            peppers = "🌶️".repeat(spicyLevel);
        }

        return "%s %s원 %s"
                .formatted(name, price, peppers);
    }
}
```

enum을 클래스처럼 getter,setter,생성자 등을 사용하여 지정할 수 있다.

```java
				YalcoChickenMenu menu1 = YalcoChickenMenu.YN;
        YalcoChickenMenu menu2 = YalcoChickenMenu.RS;
        YalcoChickenMenu menu3 = YalcoChickenMenu.XX;

        var menu1Name = menu1.getName();
        var menu2Price = menu2.getPrice();
        var menu1Desc = menu3.getDesc();
```
>위 같은 코드가 클래스 안에 정의된다면 편하게 모드를 지정할 수 있다.

.values로 배열에 enum의 value들을 담을 수 있다.

# 🔆 레코드

- 자바 14에서 Preview로 추가, 16에서 정식 등록
- 데이터의 묶음을 저장하기 위한, 단순한 형태의 클래스


```java
//  기존처럼 클래스로 작성해야 했다면...
public class ChildClass {
    private final String name;
    private final int birthYear;
    private final Gender gender;

    public ChildClass(String name, int birthYear, Gender gender) {
        this.name = name;
        this.birthYear = birthYear;
        this.gender = gender;
    }

    public String getName() { return name; }
    public int getBirthYear() { return birthYear; }
    public Gender getGender() { return gender; }
}
```

위 코드를 record로 작성하면

```java
// ⭐️  레코드로 작성
public record Child(
        String name,
        int birthYear,
        Gender gender
) {}
```
위처럼 작성될 수 있다.

❗기본적으로 Getter, Setter, 생성자 등이 탑재되어있다.

```java
				Child child1 = new Child("홍길동", 2020, Gender.MALE);
				//  💡 toString 메소드 구현 (이후 배울 Object에서 상속받아 오버라이드)
        String childStr = child1.toString();

        var children = new Child[] {
                new Child("김순이", 2021, Gender.FEMALE),
                new Child("이돌이", 2019, Gender.MALE),
                new Child("박철수", 2020, Gender.MALE),
                new Child("최영희", 2019, Gender.FEMALE),
        };

        for (var child : children) {
            System.out.printf(
                    "%s %d년생 %s 어린이%n",
                    child.gender().getEmoji(),
                    child.birthYear(),
                    child.name()
            );
        }
```
위처럼 사용할 수 있다.

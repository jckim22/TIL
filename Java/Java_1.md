# 🔆JAVA에 대해
## 🔱Java는 컴파일 언어

#### ❗JAVA -> JB -> JVM(호환을 위한 가상머신,요리사)
>덕분에 어떤 운영체제에서도 호환이된다.

#### ❗JRE(자바실행환경,요리 환경)
>JRE안에 JVM을 비롯해 여러 라이브러리 등등 많은 것들이 모여있다.

#### ❗JDK(자바개발키트, 레시피를 개발하는 회사) 
>자바코드 제작 키트
이 안에 JRE(레시피를 테스트할 식당)도 포함되어 있다.
자바 컴파일러가 포함되어있다.(자바 바이트코드로 번역)
디버거가 포함되어있다.

#### ❗JAR 도구(컴파일러가 번역한 결과물을 실행용 책자로 압축하는 도구)
>프로파일러(감독)이 포함되어있다.

#### ❗JAVA 버전
>자바는 ORACLE에서 관리하고 있다.
LTS는 장기 관리 버전(안정적)
오라클 JDK(유료), OpenJDk(무료)
JDK는 오라클 이 외에도 다양한 기업에서 배포하고 있다.

# 🔆함수와 메서드(Method)

함수(Function)와 메소드(Method)라는 단어를 상황에 맞게 잘 선택하기 위해 두 단어 간의 차이에 대해서 짚고 넘어가려 한다.


우선 함수는 여러 문장들이 하나의 기능을 구현하도록 구성한 것이라고 할 수 있다. 그 함수 중에서 클래스 내부에 정의한 함수를 메소드라고 부르는 것이다. 또한 메소드는 객체의 속성을 다루기 위한 행위를 정의한 것이라는 의미도 포함하고 있다.

>즉, 독립적으로 존재하는 함수이냐, 클래스 내부에 종속되어 있느냐의 구분으로 함수와 메소드를 구분할 수 있다.


#### Tip❗

>⭐ 디버그시 스텝인투와 스텝아웃 사용해보기


#### Tip❗
매개변수의 개수가 정해지지 않았을 때 ... 사용 (맨 마지막)

```java
		//  ⭐️ 다른(정해진) 인자들과 사용시 맨 마지막에 놓을 것
    static String descClass (int classNo, String teacher, String... kids) {
        return "%d반의 담임은 %s 선생님, 원생들은 %s 입니다."
                .formatted(classNo, teacher, String.join(", ", kids));
    }
```

```java
				String class3Desc = descClass(3, "목아진", "짱구", "철수", "훈이");

				String[] kids = {"짱구", "철수", "훈이"};
        String class3DescByArr = descClass(3, "목아진", kids);
```


# 🔆메소드 오버로딩
메소드는 매개변수의 개수, 자료형, 자료형 순서에 따라 오버로딩이 가능하다.
```java
		static int add(int a, int b) { return a + b; }

    //  매개변수의 개수가 다름
    static int add(int a, int b, int c) { return a + b + c; }

    //  매개변수의 자료형이 다름
    static double add(double a, double b) { return a + b; }

    //  매개변수의 자료형 순서가 다름
    static String add(String a, char b) { return a + b; }
    static String add(char a, String b) { return a + b; }

    //  ⚠️ 반환 자료형이 다른 것은 오버로딩 안 됨 - 다른 함수명 사용
    //  static double add(int a, int b) { return (double) (a + b); }
```

```java
				int res1 = add(1, 2); // 🔴 스텝인투로 들어가 볼 것
        int res2 = add(3, 4, 5);
        double res3 = add(1.2, 3.4);
        String res4 = add("로보트 태권", 'V');
        String res5 = add('X', "Men");
```




# 🔆CLASS

## 🔱super
>생성자나 메서드를 오버라이딩할 때 super를 사용해서 부모의 생성자나 메서드를 사용할 수 있다.


## 🔱상속
#### ❗ex)
>
- 자식 클래스의 인스턴스는 부모 클래스 자료형에 속함
    - *모든 셧다운버튼과 토글버튼은 버튼이다.*
- 다른 방향으로는 불가
    - *모든 버튼이 셧다운 버튼이거나 토글버튼인 것은 아니다.*
    - *셧다운 버튼은 토글 버튼이 아니다.*
    

>Button 배열에 여러 형태의 배열을 담을 수 있다.


#### Tip❗
### `instanceof` 연산자

>- 뒤에 오는 클래스의 자료형에 속하는(족보상 같거나 아래인) 인스턴스인지 확인
    - 이후 배울 인터페이스에 대해서도 사용 가능
- 상속관계가 아닌 클래스끼리는 컴파일 오류


```java
				Button button = new Button("버튼");
        ToggleButton toggleButton = new ToggleButton("토글", true);
        ShutDownButton shutDownButton = new ShutDownButton();

				//  true
        boolean typeCheck1 = button instanceof Button;
        boolean typeCheck2 = toggleButton instanceof Button;
        boolean typeCheck3 = shutDownButton instanceof Button;

        //  false
        boolean typeCheck4 = button instanceof ShutDownButton;
        boolean typeCheck5 = button instanceof ToggleButton;

        //  ⚠️ 컴파일 에러
        boolean typeCheck6 = toggleButton instanceof ShutDownButton;
        boolean typeCheck7 = shutDownButton instanceof ToggleButton;
```

# 🔆 Object 클래스
```java
				//  ⭐️ 모든 자료형을 포함할 수 있는 배열
        Object[] objs = {
                1, false, 3.45, '가', "안녕하세요", new Button("Space")
        };

        for (Object obj : objs) {
            System.out.println(obj);
        }
```


```java
				Object obj1 = new Object();

				//  ⭐️ IDE의 안내대로 다른 패키지의 클래스 임포트
				//  💡 해당 클래스의 생성자가 public 이어야 함
        Object obj2 = new YalcoChicken(3, "판교");
        Object obj3 = new ShutDownButton();
        Object obj4 = new FireSlime();

        //  원시 자료형들도 Object의 인스턴스로... - 이후 자세히 배울 것
        Object obj5 = true;
        Object obj6 = 1;
        Object obj7 = "Hello";
```

```java
				//  ⭐️ 모든 자료형을 포함할 수 있는 배열
        Object[] objs = {
                1, false, 3.45, '가', "안녕하세요", new Button("Space")
        };

        for (Object obj : objs) {
            System.out.println(obj);
        }
```

# 🔆자식 클래스의 기능을 사용하기 (명시적 형변환)
```java
				System.out.println("\n- - - - -\n");

				YalcoChicken ycStores[] = {
                new YalcoChicken(3, "판교"),
                new YalcoChicken(17, "강남"),
                new YalcoChickenDT(108, "철원"),

        };

        for (YalcoChicken store : ycStores) {
            if (store instanceof YalcoChickenDT) {
                //  ⭐️ 자식 클래스의 기능을 사용하려면 명시적 타입 변환
                ((YalcoChickenDT) store).takeDTOrder();
            } else {
                store.takeHallOrder();
            }
        }
```
# 🔆final(상수)
>inal이 붙어있으면 내용 수정은 가능하지만 새로운 객체로 바꿀 수 없고 그 메서드가 final이면 오버라이딩할 수 없다.

```java
public class YalcoChicken {
    protected static final String CREED = "우리의 튀김옷은 얄팍하다.";

    private final int no;
    public String name;

    //  ⭐️ 필수 - no가 final이고 초기화되지 않았으므로
    public YalcoChicken(int no, String name) {
        this.no = no;
        this.name = name;
    }

    public void changeFinalFields () {
        // ⚠️ 불가
        this.no++;
    }

    public final void fryChicken () {
        System.out.println("염지, 반죽입히기, 튀김");
    }
}
```

```java
public final class YalcoChickenDT extends YalcoChicken {
		public YalcoChickenDT(int no, String name) {
        super(no, name);
    }

    //  ⚠️ 불가
    public void fryChicken () {
        System.out.println("염지, 반죽입히기, 미원, 튀김");
    }

		// 생성자 추가할 것
}
```

# 🔆abstract(추상적인)

>추상 클래스는 스스로 인스턴스를 만들 수 없다.

### ❗ex)
>현대그룹은 추상적인 클래스이다..
현대 자동차야말로 인스턴스를 만들 수 있는 클래스이다.
현대 그룹은 상속을 위한 클래스❗

>❗추상 메소드는 선언만 하고 구현하지 않는다.
자식 클래스에서 무조건 구현을 해주어야 한다.
접근 제어자는 자식 클래스에서 설정한다.


>static이 붙은 클래스 메소드는 인스턴스로 생성해서 쓰는게 아니기 때문에 abstract를 사용할 수 없다.

#### Tip❗
>인텔리제이에서 생성에서 추상메소드를 오버라이딩할 수 있다.

>ex) public abstract class FormElement (화면에 보여지는 form)
public class Button extends FormElement
public class Switch extends FormElement
public class DropDown extends FormElement

>여기에 abstract 메서드를 만들고 각각의 자식 클래스에서 오버라이딩을 한다.



# 🔆인터페이스(interface)

### 🔱추상 클래스와의 차이

*🔴  : 추상 클래스 / 🔷  : 인터페이스*

- 🔴  포유류
    - 북극곰 - 🔷  사냥, 🔷  수영
    - 날다람쥐 - 🔷  비행
- 🔴  파충류
    - 거북 - 🔷  수영
    - 날도마뱀 - 🔷  사냥, 🔷  수영, 🔷  비행
- 🔴  조류
    - 독수리 - 🔷  사냥, 🔷  비행
    - 펭귄 - 🔷  사냥, 🔷  수영
    
    
>인터페이스는 변수는 상수(final)이지만, 메소드는 추상 메소드이다.

```java
public interface Hunter {
    String position = "포식자"; // ⭐️ final - 초기화하지 않을 시 오류
    void hunt ();
}
```
```java
public interface Swimmer {
    void swim();
}
```
```java
public class PolarBear extends Mamal implements Hunter, Swimmer {
    public PolarBear() {
        super(false);
    }

    @Override
    public void hunt() {
        System.out.println(position + ": 물범 사냥");
    }

    @Override
    public void swim() {
        System.out.println("앞발로 수영");
    }
}
```
인터페이스 또한 다형성의 원칙이 적용 된다.
```java
				//  ⭐ 다형성
        PolarBear polarBear = new PolarBear();
        Mamal mamal = polarBear;
        Swimmer swimmer = polarBear;

        GlidingLizard glidingLizard = new GlidingLizard();
        Eagle eagle = new Eagle();

        Hunter[] hunters = {
                polarBear, glidingLizard, eagle
        };

        //  💡 인터페이스 역시 다형성에 의해 자료형으로 작용 가능
        for (Hunter hunter : hunters) {
            hunter.hunt();
        }
```

위처럼 다형성을 통해서 Hunter인 인스턴트만 찾을 수 있다.

#### Tip❗

>자바 8부터는 하위호완성을 위해 인터페이스도 static인 클래스 메소드를 구현할 수 있다.(매장의 기능이 아님)
default를 붙이면 구상 메소드 또한 구현 가능하다.(인스턴스를 생성하지 않고도 기능을 사용할 수 있음)



# 🔆싱글턴
❓컴퓨터에서 여러 영상을 틀어도 볼륨은 setting값으로 똑같아야 한다고 했을 때 어떻게 하면 setting 값을 외부에서 건드리지 않고 하나의 싱글 클래스로 모두에게 적용할 수 있을까 ❓

```java
public class Setting {

    //  ⭐️ 이 클래스를 싱글턴으로 만들기

    // 클래스(정적) 필드
    // - 프로그램에서 메모리에 하나만 존재
    private static Setting setting;

    //  ⭐️ 생성자를 private으로!
    // - 외부에서 생성자로 생성하지 못하도록
    private Setting () {}

    //  💡 공유되는 인스턴스를 받아가는 public 클래스 메소드
    public static Setting getInstance() {
        //  ⭐️ 아직 인스턴스가 만들어지지 않았다면 생성
        //  - 프로그램에서 처음 호출시 실행됨
        if (setting == null) {
            setting = new Setting();
        }
        return setting;
    }

    private int volume = 5;

    public int getVolume() {
        return volume;
    }
    public void incVolume() { volume++; }
    public void decVolume() { volume--; }
}
```
```java
public class Tab {
    //  ⭐️ 공유되는 유일한 인스턴스를 받아옴
    private Setting setting = Setting.getInstance();

    public Setting getSetting() {
        return setting;
    }
}
```

```java
				Tab tab1 = new Tab();
        Tab tab2 = new Tab();
        Tab tab3 = new Tab();

        System.out.println(tab1.getSetting().getVolume());
        System.out.println(tab2.getSetting().getVolume());
        System.out.println(tab3.getSetting().getVolume());

        System.out.println("\n- - - - -\n");

        tab1.getSetting().incVolume();
        tab1.getSetting().incVolume();

        System.out.println(tab1.getSetting().getVolume());
        System.out.println(tab2.getSetting().getVolume());
        System.out.println(tab3.getSetting().getVolume());

        //  🎉 외부에서 각 사용처들을 신경쓸 필요 없음
```
바로 위처럼 Setting 인스턴스가 null 이면 생성하고 반환하고 아니라면 그 Setting 값을 반환하는 getInstance 메서드를 만든다.
또한 외부에서 생성자를 실행할 수 없도록 생성자는 private로 선언한다.

>❗이렇게 하면 외부에서 Setting 인스턴스를 불필요하게 생성할 수 없고 접근할 수 없다.

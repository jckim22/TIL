# 🔆Comparable & Comparator

- 둘 모두 인터페이스
- `Comparable` *(비교의 대상)* : 자신과 다른 객체를 비교
    - 숫자 클래스들, 불리언, 문자열
    - 이후 배울 `Date`, `BigDecimal`, `BigInteger` 등
- `Comparator` *(비교의 주체)* : 주어진 두 객체를 비교
    - 컬렉션에서는 정렬의 기준으로 사용
    - `Arrays`의 정렬 메소드, `TreeSet`이나 `TreeMap`등의  생성자에 활용
    
```java
				Integer int1 = Integer.valueOf(1);
        Integer int2 = Integer.valueOf(2);
        Integer int3 = Integer.valueOf(3);

        //  대상보다 작을 때: 음수, 같거나 클 때: 양수
        int _1_comp_3 = int1.compareTo(3);
        int _2_comp_1 =  int2.compareTo(1);
        int _3_comp_3 =  int2.compareTo(1);
        int _t_comp_f = Boolean.valueOf(true).compareTo(Boolean.valueOf(false));
        int _abc_comp_def = "ABC".compareTo("DEF");
        int _def_comp_abc = "efgh".compareTo("abcd");
```

```java
				Integer[] nums = {3, 8, 1, 7, 4, 9, 2, 6, 5};
        String[] strs = {
                "Fox", "Banana", "Elephant", "Car", "Apple", "Game", "Dice"
        };

				//  ⭐️ Arrays 클래스의 sort 메소드
        //  - 기본적으로 compareTo에 의거하여 정렬
        //  - 인자가 없는 생성자로 생성된 TreeSet, TreeMap도 마찬가지
        Arrays.sort(nums);
        Arrays.sort(strs);
```

❗ compareTo에 의거하여 정렬

```java
public class IntDescComp implements Comparator<Integer> {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o2 - o1;
    }
}
```
위처럼 오버라이딩하면 거꾸로 정렬한다.

### 🔱트리에 둘 모두 적용시

- `Comparator`가 없으면 `Comparable`의 기준 따름

```java
public class Person implements Comparable<Person> {
    private static int lastNo = 0;
    private int no;
    private String name;
    private int age;
    private double height;

    public Person(String name, int age, double height) {
        this.no = ++lastNo;
        this.name = name;
        this.age = age;
        this.height = height;
    }

    public int getNo() { return no; }
    public String getName() { return name; }
    public int getAge() { return age; }
    public double getHeight() { return height; }

    @Override
    public int compareTo(Person p) {
        return this.getName().compareTo(p.getName());
    }

    @Override
    public String toString() {
        return "Person{" +
                "no=" + no +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", height=" + height +
                '}';
    }
}
```
```java
public class PersonComp implements Comparator<Person> {
    public enum SortBy { NO, NAME, AGE, HEIGHT }
    public enum SortDir { ASC, DESC }

    private SortBy sortBy;
    private SortDir sortDir;

    public PersonComp(SortBy sortBy, SortDir sortDir) {
        this.sortBy = sortBy;
        this.sortDir = sortDir;
    }

    @Override
    public int compare(Person o1, Person o2) {
        int result = 0;
        switch (sortBy) {
            case NO: result = o1.getNo() > o2.getNo() ? 1 : -1; break;
            case NAME: result = o1.getName().compareTo(o2.getName()); break;
            case AGE: result = o1.getAge() > o2.getAge() ? 1 : -1; break;
            case HEIGHT: result = o1.getHeight() > o2.getHeight() ? 1 : -1; break;
        }
        return result * (sortDir == SortDir.ASC ? 1 : -1);
    }
}
```

```java
				TreeSet[] treeSets = {
                new TreeSet<>(),
                new TreeSet<>(new PersonComp(PersonComp.SortBy.NO, PersonComp.SortDir.DESC)),
                new TreeSet<>(new PersonComp(PersonComp.SortBy.AGE, PersonComp.SortDir.ASC)),
                new TreeSet<>(new PersonComp(PersonComp.SortBy.HEIGHT, PersonComp.SortDir.DESC))
        };

        for (var p : new Person[] {
                new Person("홍길동", 20, 174.5),
                new Person("전우치", 28, 170.2),
                new Person("임꺽정", 24, 183.7),
                new Person("황대장", 32, 168.8),
                new Person("붉은매", 18, 174.1),
        }) {
            for (var ts: treeSets) {
                ts.add(p);
            }
        }

        for (var ts: treeSets) {
            for (var p : ts) {
                System.out.println(p);
            }
            System.out.println("\n- - - - -\n");
        }
```
Comparable을 상속받고 compareTo를 오버라이딩한다.
자신의 객체와 다른 객체를 비교한다.

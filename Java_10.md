# 🔆자바 프로그램의 오류 *error*

- 컴파일 오류 - 컴파일 과정에서 잡히는 오류
    - 문법 오류, 자료형 오류, 잘못된 식별자 *(오타)* 등…
- 런타임 오류 - 아래의 두 종류로 구분됨
    - 에러 *error*
    - 예외 *exception*
    
## 🔱에러와 예외

*error와 exception 둘 모두 `Throwable` 의 자식 클래스*

- 💀 에러 `Error` - 해결 불가능한 문제
    - 회사에 가다가 총에 맞아 죽음
        - 회사에 죽어도 *(죽었으니까)*  못 감
    - 무한루프, 메모리 한도 초과, 스택오버플로우 등…
        - 일반적으로 시스템 레벨의 문제
        
- 🛑 예외 `Exception`  - 대비하여 해결할 수 있는 문제
    - 가려던 출근길이 공사로 막힘, 지하철 운행 중단, 스텝이 꼬여 넘어짐
        - 대책을 마련해두면 회사에 갈 수 있음
    - 읽어오려는 파일이 없음, 배열의 길이 너머로 접근…
    
    
### 🌳 상속도

`Throwable`

- `Error`
    - `VirtualMachineError`
        - `OutOfMemoryError`
        - `StackOverflowError`
        - …
    - …
- `Exception`
    - ⭐️ `RuntimeException`
        - `IndexOutOfBoundException`
        - `NullPointerException`
        - `ClassCastException`
        - …
    - `ReflectiveOperationException`
        - `ClassNotFoundException`
        - `NoSuchMethodException`
        - …
    - `IOException`
        - `FileNotFoundException` - *[java.io](http://java.io) 패키지*
    - … 
    
모든 Exception은 런타임 오류에 속한다.


## 🔱예외의 두 종류

- Unchecked Exception
    - `RuntimeException` 의 하위 클래스들
    - 개발자의 실수에 의해 발생할 수 있는 예외들
        - 걸어가다가 스텝이 꼬임
        - null 체크, 배열 등의 범위 벗어남, 0으로 나눔 등…
    - 아래에서 배울 예외처리가 필수는 아님 *(실수를 안 하면 되므로)*
- Checked Exception
    - 기타 예외들
    - 주로 외적 요인으로 발생
        - 지하철 운행중단
    - 발생 가능한 부분에는 반드시 예외처리해야 함 *(곧 배울 것)*
        - 처리하지 않을 시 컴파일 단계에서 반려
            - 지하철을 타는 코드의 경우 대안 마련해야 함
❗ 예시로 사용자가 이상한 요청을 보내거나 서버의 환경이 바뀜(파일이 없음)

>### ⭐️ 에러나 예외가 발생하면 프로그램이 종료됨
- 예외를 아래의 방법으로 예외처리 했을 경우 제외

```java
				int[] ints = {1, 2, 3};
        System.out.println(ints[3]); // ⚠️ 런타임 예외 발생
				System.out.println("예외를 방지하지 않았을 때");
```
위 코드를 보면 Out of bounds 예외가 발생한다.

```java
				try {
            // ⭐️ 예외가 일어날 여지가 있는 코드를 try 블록에 작성
            System.out.println(ints[3]);
        } catch (Exception e) {
            String errMsg = e.getMessage();
            e.printStackTrace(); // 🔴
        }
				System.out.println("예외를 try문으로 감쌌을 때 1");
```
위처럼 예외처리가 가능하다.
Exception으로 예외를 받고 그대로 printStackTrace()로 출력할 수 있다.

```java
				try {
            System.out.println(((String) null).length());
        } catch (Exception e) {
            String errMsg = e.getMessage();
            e.printStackTrace(); // 🔴
        }
				System.out.println("예외를 try문으로 감쌌을 때 2");
```
위는 nullPointerException이 생길 때 예외처리를 해준 것이다.


# 🔆try 문 더 알아보기

```java
		public static void tryThings1 (int i) {
        try {
            switch (i) {
                //  💡 예외가 발생하면 바로 실행을 멈추고 catch 문으로 감
                //  - 아래 케이스들은 각각 예외가 발생하므로 break 넣지 않았음
                case 1: System.out.println((new int[1])[1]);
                case 2: System.out.println("abc".charAt(3));
                case 3: System.out.println((Knight) new Swordman(Side.RED));
                case 4: System.out.println(((String) null).length());
            }

            //  ⭐️ 아래의 코드는 예외가 발생하면 실행되지 않음
            System.out.printf("%d: 🎉 예외 없이 정상실행됨%n", i);

        } catch (Exception e) {

            //  💡 예외 발생시에만 실행되는 블록
            System.out.printf(
                    "%d: 🛑 [ %s ] %s%n", i, e.getClass(), e.getMessage()
            );
            e.printStackTrace();
        }
    }
```
main.java
```java
				IntStream.rangeClosed(0, 4)
                        .forEach(Ex01::tryThings1);
```

main에서 IntStream을 사용해 스트림을 생성하고 .forEach의 consumer로 Ex01의 tryThings1를 넣어준다.

그럼 0부터 4까지 스트림을 흘려보내며 실행할 것이다.

- 0일 때는 case 0이 없기 때문에 예외 없이 정상적으로 실행된다.
- 1 ~ 4 일 경우 각 예외에 맞는 예외 클래스와 문이 catch문에서 프린트된다.
- 1일 때는 인덱스의 범위를 초과하여 Out of bounds
- 2일 때는 String의 인덱스 범위를 초과하여 out of range
- 3일 때는 부모 클래스를 자식 클래스로 형변환하려는 것에 대한 다형성 위반
- 4일 때는 null을 다룰려는 것에 대한 NullPorinter 예외

## 🔱 다중 catch 문
예외에 따라 여러 종류의 catch문을 실행할 수 있다.

```java
		//  💡 다중 catch 문 이후에도 사용됨
    public static void withFinally3 (int i) {
        try {
            switch (i) {
                case 1: System.out.println((new int[1])[1]);
                case 2: System.out.println("abc".charAt(3));
                case 3: System.out.println((Knight) new Swordman(Side.RED));
                case 4: System.out.println(((String) null).length());
            }
            System.out.printf("%d: 🎉 예외 없이 정상실행됨%n", i);

        } catch (ArrayIndexOutOfBoundsException | StringIndexOutOfBoundsException e) {
            System.out.printf("%d : 🤮 범위를 벗어남%n", i);
        } catch (ClassCastException e) {
            System.out.printf("%d : 🎭 해당 클래스로 변환 불가%n", i);
        } catch (Exception e) {
            System.out.printf("%d : 🛑 기타 다른 오류%n", i);
        } finally {
            System.out.println("🏁 무조건 실행");
        }
    }
```


# 🔆 `finally` 문

- 예외 발생 여부에 상관없이 반드시 실행할 코드
    - 데이터베이스 연결을 열어 작업한 뒤 닫아줄 때 등에 사용

>❗ 일을 보든 못 보든 화장실 문은 닫아줘야 한다.
    
`☕ Ex02.java`

```java
		public static void withFinally1 (boolean makeException) {
        try {
            if (makeException) System.out.println("".charAt(0));
            System.out.println("🎉 예외 없이 정상실행됨");
        } catch (Exception e) {
            System.out.println("🛑 예외 발생");
        } finally {
            System.out.println("🏁 무조건 실행");
        }

				//  ❓ 그냥 try 문 밖에 적으면 안 될까?
				System.out.println("🏁 이렇게 말이지.");
    }
```

```java
				withFinally1(false);
				System.out.println("\n- - - - -\n");
        withFinally1(true);
```
아래처럼 return이 일어나도 무조건 실행할 수 있도록 finally문을 사용해준다.
```java
		public static char withFinally2 (int index) {
        String str = "Hello";
        try {
            char result = str.charAt(index);
            System.out.println("🎉 예외 없이 정상실행됨");
            return result;
        } catch (Exception e) {
            System.out.println("🛑 예외 발생");
            return '!';
        } finally {
            //  ⭐️ 위에서 return이 발생하더라도 이건 하고 넘어감
            System.out.println("🏁 무조건 실행");
        }

        //  💡 try, catch 블록에 모두 return이 있으므로
        //  이 부분은 작성될 수 없음
        //  System.out.println("🏁 이렇게 말이지.");
    }
```

```java
				System.out.println("\n- - - - -\n");

        var char1 = withFinally2(3);
        var char2 = withFinally2(6);
```
아래와 같은 다중 catch문에서도 잘 사용된다.
```java
		//  💡 다중 catch 문 이후에도 사용됨
    public static void withFinally3 (int i) {
        try {
            switch (i) {
                case 1: System.out.println((new int[1])[1]);
                case 2: System.out.println("abc".charAt(3));
                case 3: System.out.println((Knight) new Swordman(Side.RED));
                case 4: System.out.println(((String) null).length());
            }
            System.out.printf("%d: 🎉 예외 없이 정상실행됨%n", i);

        } catch (ArrayIndexOutOfBoundsException | StringIndexOutOfBoundsException e) {
            System.out.printf("%d : 🤮 범위를 벗어남%n", i);
        } catch (ClassCastException e) {
            System.out.printf("%d : 🎭 해당 클래스로 변환 불가%n", i);
        } catch (Exception e) {
            System.out.printf("%d : 🛑 기타 다른 오류%n", i);
        } finally {
            System.out.println("🏁 무조건 실행");
        }
    }
```

```java
				System.out.println("\n- - - - -\n");

        IntStream.rangeClosed(0, 4)
                .forEach(i -> withFinally3(i));
```

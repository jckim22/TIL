# 🔆스트림
 어떤 프로그램을 만들기 위해서 지금까지 해왔던 방식으로 코드를 짜게 되면

`☕ Ex01.java`

```java
				List<Integer> int0To9 = new ArrayList<>(
                Arrays.asList(5, 2, 0, 8, 4, 1, 7, 9, 3, 6)
        );
        //  홀수만 골라낸 다음 정렬하여 "1, 3, 5..."와 같은 문자열로 만들어보기
```

```java
				//  기존의 방식
        List<Integer> odds = new ArrayList<>();
        for (var i : int0To9) { 
            if (i % 2 == 1) odds.add(i); 
        }
        odds.sort(Integer::compare);

        List<String> oddsStrs = new ArrayList<>();
        for (var i : odds) { 
            oddsStrs.add(String.valueOf(i)); 
        }
        var oddsStr = String.join(", ", oddsStrs);
```
위와 같은 방식으로 코드를 짜게 될 것이다.

허나 스트림을 사용하면 아래처럼 만들 수 있다.

```java
				//  ⭐ 스트림을 사용한 방식
        //  각 라인이 무엇을 반환하는지 확인할 것
        var oddsStrStreamed = int0To9
                .stream()
                .filter(i -> i % 2 == 1)
                .sorted(Integer::compare)
                .map(String::valueOf)
                .collect(Collectors.joining(", "));
```

- 💧 정수기, 음료제조기, 패트병 밀봉기가 나열된 파이프로 물을 흘려보냄
- 일련의 데이터를 연속적으로 가공하는데 유용
    - 내부적으로 수행 - 중간과정이 밖으로 드러나지 않음
        - 외부에 변수 등이 만들어지지 않음
    - 배열, 콜랙션, I/O 등을 동일한 프로세스로 가공
    - 함수형 프로그래밍을 위한 다양한 기능들 제공
    - 가독성 향상
    - ⭐ 원본을 수정하지 않음 - *정렬 등에 영향받지 않음*
- 멀티쓰레딩에서 병렬처리 가능




## 🔱스트림 생성

`☕ Ex02.java`

```java
				//  💡 배열로부터 생성
        Integer[] integerAry = {1, 2, 3, 4, 5};
        Stream<Integer> fromArray = Arrays.stream(integerAry);
        var fromArray_Arr = fromArray.toArray();

        //  ⚠️ 런타임 에러
        //  - 스트림은 한 번 사용하면 끝 (흘러가버린 물)
        //var ifReuse = fromArray.toArray();
```

```java
				//  원시값의 배열로부터는 스트림의 클래스가 달라짐
        int[] intAry = {1, 2, 3, 4, 5};
        IntStream fromIntAry = Arrays.stream(intAry);
        var fromIntAry_Arr = fromIntAry.toArray();

        double[] dblAry = {1.1, 2.2, 3.3};
        DoubleStream fromDblAry = Arrays.stream(dblAry);
        var fromDblAry_Arr = fromDblAry.toArray();
```

```java
				//  💡 값들로 직접 생성
        IntStream withInts = IntStream.of(1, 2, 3, 4, 5);
        Stream<Integer> withIntegers = Stream.of(1, 2, 3, 4, 5);
        Stream<Unit> withUnits = Stream.of(
                new Swordman(Side.BLUE),
                new Knight(Side.BLUE),
                new MagicKnight(Side.BLUE)
        );
        var withUnits_Arr = withUnits.toArray();
```

```java
				//  💡 컬렉션으로부터 생성
        List<Integer> intAryList = new ArrayList<>(Arrays.asList(integerAry));
        Stream fromColl1 = intAryList.stream();
        var fromColl1_Arr = fromColl1.toArray();
```

```java
				//  맵의 경우 엔트리의 스트림으로 생성
        Map<String, Character> subjectGradeHM = new HashMap<>();
        subjectGradeHM.put("English", 'B');
        subjectGradeHM.put("Math", 'C');
        subjectGradeHM.put("Programming", 'A');
        var fromHashMap_Arr = subjectGradeHM.entrySet().stream().toArray();
```

```java
				//  문자열을 각 문자에 해당하는 정수의 스트림으로
        IntStream fromString = "Hello World".chars();
        var fromString_Arr = fromString.toArray();
```

밑에 var 변수는 디버깅으로 stream이 흘려보내는 값들을 확인하기 위함

`📄 turtle.txt`

```
거북아 거북아
머리를 내어라
내놓지 않으면
구워서 먹으리
```

```java
				//  💡 파일로부터 생성
        //  - File I/O : 이후 배울 것
        Stream<String> fromFile;
        Path path = Paths.get("./src/sec09/chap04/turtle.txt");
        try {
             fromFile = Files.lines(path);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

        var fromFile_Arr = fromFile.toArray();
```
위처럼 파일의 내용으로도 스트림을 생성할 수 있다.


또한 빈 스트림도 생성하는 경우도 있다.

```java
				//  💡 빈 스트림 생성
        //  - 스트림을 받는 메소드 등에서 종종 사용
        Stream<Double> emptyDblStream = Stream.empty();
```


# 🔆스트림 연산

이제 위에서 봤던 스트림을 사용한 프로그램을 어떻게 가능케 했는지 보여주는 연산자를 살펴보자.

| 연산 | 종류 | 설명 |
| --- | --- | --- |
| peek | 중간 | 연산과정중 스트림에 영향을 끼치지는 않으면서 주어진 Consumer 작업을 실행 |
| filter | 중간 | 주어진 Predicate에 충족하는 요소만 남김 |
| distinct | 중간 | 중복되지 않는 요소들의 스트림을 반환 |
| map | 중간 | 주어진 Function에 따라 각 요소들을 변경 |
| sorted | 중간 | 요소들을 정렬 |
| limit | 중간 | 주어진 수 만큼의 요소들을 스트림으로 반환 |
| skip | 중간 | 앞에서 주어진 개수만큼의 요소를 제거 |
| takeWhile / dropWhile | 중간 | 주어진 Predicate 을 충족하는 동안 취하거나 건너뜀 |
| forEach | 최종 | 각 요소들에 주어진 Consumer 를 실행 |
| count | 최종 | 요소들의 개수를 반환 |
| min / max | 최종 | 주어진 Comparator 에 따라 최소/최대값을 반환 |
| reduce | 최종 | 주어진 초기값과 BinaryOperator 로 값들을 하나의 값으로 접어 나감 |
- 이 외 병렬 연산 및 `Optional` 관련 연산자들 


아래처럼 연산자를 활요한 프로그래밍을 할 수 있다.


```java
				IntStream
                .range(1, 100)
                .filter(i -> i % 2 == 0)
                //  💡 아래의 중간과정을 하나하나 주석해제해 볼 것
                //.skip(10)
                //.limit(10)
                //.map(i -> i * 10)
                .forEach(System.out::println);
```
아래처럼 스트림을 생성해도 된다.
```java
				Integer[] ints = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };

				//  💡 allMatch/anyMatch : (모든 요소가/어느 한 요소라도)
        //  - 주어진 Predicate에 부합하는가 여부 반환 
        boolean intsMatch = Arrays.stream(ints)
                .allMatch(i -> i > 0);
                //.allMatch(i -> i % 2 == 0);
                //.anyMatch(i -> i < 0);
                //.anyMatch(i -> i % 2 == 0);
```

```java
				//  💡 sum : IntStream, LongStream, DoubleStream 요소의 총합 반환

        int intsSum = IntStream.range(0, 100 + 1)
                //.filter(i -> i % 2 == 1)
                //.filter(i -> i % 2 == 0)
                //.filter(i -> i % 3 == 0)
                .sum();
```

Arrays.stream()으로 리스트를 스트림하고 IntStream, DoubleStream 등 range를 이용해 숫자들을 스트림으로 생성하고 스트림 연산을 사용해 가공할 수 있다.

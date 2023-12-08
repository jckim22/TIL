# 🔆이터레이터

- `java.lang.Iterable` 인터페이스 구현 클래스에서 사용
- 컬렉션을 순회하는데 사용
    - 투어가이드, 순시 감찰관 역할
    - 특정 기준의 요소들 제거에 유용
    - 순회 상태가 저장될 필요가 있을 때 유용
    
    `☕ Main.java`

```java
				Set<Integer> intHSet = new HashSet<>(
                Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9)
        );

        //  💡 이터레이터 반환 및 초기화
        //  - 기타 Collection 인터페이스를 구현한 클래스들에도 존재
        Iterator<Integer> intItor = intHSet.iterator();
```

```java
				// 💡 next : 자리를 옮기며 다음 요소 반환
        Integer int1 = intItor.next();
        Integer int2 = intItor.next();
        Integer int3 = intItor.next();

				//  💡 hasNext : 순회가 끝났는지 여부 반환
        boolean hasNext = intItor.hasNext();
```

```java
				//  ⭐️ 순회 초기화
        intItor = intHSet.iterator();
```

```java
        //  💡 remove : 현 위치의 요소 삭제
        while (intItor.hasNext()) {
            if (intItor.next() % 3 == 0) {
                intItor.remove();
            }
        }
```

```java
				//  ⚠️ foreach 문으로 시도하면 오류
        for (Integer num : intHSet) {
            if (num % 3 == 0) intHSet.remove(num);
        }
```

```java
				List<Unit> enemies = new ArrayList<>(
                Arrays.asList(
												new Swordman(Side.RED),
                        new Knight(Side.RED),
                        new Swordman(Side.RED),
                        new Swordman(Side.RED),
                        new Knight(Side.RED),
                        new Knight(Side.RED),
                        new Swordman(Side.RED),
                        new MagicKnight(Side.RED),
                        new Swordman(Side.RED),
                        new MagicKnight(Side.RED),
                        new Knight(Side.RED),
                        new MagicKnight(Side.RED))
        );

        Iterator<Unit> enemyItor = enemies.iterator();
```

```java
				var thunderBolts = 3;
        var fireBalls = 4;
        
        while (enemyItor.hasNext() && thunderBolts-- > 0) {
            var enemy = enemyItor.next();
            System.out.printf("%s 벼락 공격%n", enemy);
            enemy.hp -= 50;
        }
        while (enemyItor.hasNext() && fireBalls-- > 0) {
            var enemy = enemyItor.next();
            System.out.printf("%s 파이어볼 공격%n", enemy);
            enemy.hp -= 30;
        }
        while (enemyItor.hasNext()) {
            var enemy = enemyItor.next();
            System.out.printf("%s 화살 공격%n", enemy);
            enemy.hp -= 10;
        }
```

❗ 왜 이터레이터를 사용하는가 ?
ex) foreach 문을 돌리다 요소를 삭제하게 되면 오류가 난다.
또한 위 코드에서 공격 스킬 제한 걸려있고 순회 정보가 저장이 되어야하는데 반복문을 쓰기 애매할 때 이터레이터가 아주 좋게 쓰인다.

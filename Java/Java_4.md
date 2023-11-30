`☕ Side.java`

```java
public enum Side {
    RED("레드"), BLUE("블루");
    private String name;

    Side(String name) { this.name = name; }

    public String getName() { return name; }
}
```
enum으로 레드인지 블루인지를 구별한다.

`☕ Unit.java`

```java
public abstract class Unit {
    
		//  ⚠️ 이후 실습의 편의를 위해 일부 필드를 public으로
    //  - 실무에서는 private로 만들고 getter/setter 권장
    public Side side;
    public int hp;

    public Unit(Side side, int hp) {
        this.side = side;
        this.hp = hp;
    }

		public Side getSide() {
        return side;
    }
}
```
추상 클래스인 Unit은 enum인 side와 hp라는 인스턴트 변수를 갖고 있다.

`☕ Attacker.java`

```java
public interface Attacker {
    void defaultAttack(Unit target);
}
```
Attacker 인터페이스이다.
꼭 상속해서 구현해주어야한다. (default가 아니기 때문에)

`☕ Swordman.java`

```java
public class Swordman extends Unit implements Attacker {
    public Swordman(Side side) {
        super(side, 80);
    }

    private void swordAttack(Unit target) {
        target.hp -= 10;
    }

    @Override
    public void defaultAttack(Unit target) {
        swordAttack(target);
    }

		@Override
    public String toString() {
        return side.toString() + " 진영 검사";
    }
}
```
추상 클래스인 Unit과 Attacker 인터페이스를 상속한 Swordman은 swordAttack이라는 private메서드를 갖고 있다.

생성자는 부모의 생성자를 이용한다.

또한 인터페이스의 메서드 defaultAttack를 오버라이딩해서 swordAttack이라는 기능을 사용하게 한다.

Object객체의 toString을 오버라이딩한다.


`☕ Knight.java`

```java
public class Knight extends Swordman {
    private enum Weapon { SWORD, SPEAR }
    private Weapon weapon = Weapon.SWORD;

    public Knight(Side side) {
        super(side);
        hp += 40;
    }

    public void switchWeapon () {
        weapon = weapon == Weapon.SWORD ? Weapon.SPEAR : Weapon.SWORD;
    }

    private void spearAttack(Unit target) {
        target.hp -= 14;
    }

    @Override
    public void defaultAttack(Unit target) {
        if (weapon == Weapon.SWORD) {
            super.defaultAttack(target);
        } else {
            spearAttack(target);
        }
    }

		@Override
    public String toString() {
        return side.toString() + " 진영 기사";
    }
}
```

Knight는 3대째 자손 클래스이다.
Swordman 클래스를 상속받았고 enum으로 무기가 추가 되었다.
상속자 역시 부모 클래스의 상속자를 이용하되 그 이후 hp는 +=40 해준다.
무기가 더 생겼으므로 switchWeapon 메서드도 생겼고
무기가 창일 때를 대비하여 spearAttack 메서드도 생겼다.

defaultAttack을 또 오버라이딩하여 무기에 따라 다른 공격을 사용하게 했다.

`☕ MagicKnight.java`

```java
public class MagicKnight extends Knight {
    public int mana = 60;
    public final int MANA_USAGE = 4;
    public MagicKnight(Side side) {
        super(side);
    }

    public void lighteningAttack (Unit[] targets) {
        for (Unit target : targets) {
            if (target instanceof MagicKnight) continue;
            if (mana < MANA_USAGE) break;
						System.out.printf("⚡️ → 💀 %s%n", target);
            target.hp -= 8;
            mana -= MANA_USAGE;
        }

    }

		@Override
    public String toString() {
        return side.toString() + " 진영 마법기사";
    }
}
```
MaigicKnight는 4대째 자손 클래스이고 Knight를 상속받는다.
부모의 상속자를 사용하고 새로운 메서드가 추가 되었다.
public으로 defaultAttack과는 달리 별개적인 새로운 스킬이다.


`☕ Horse.java`

```java
public class Horse<T extends Unit> {
    private int extraHp;
    private T rider;

    public Horse(int extraHp) {
        this.extraHp = extraHp;
    }

    public void setRider(T rider) {
        this.rider = rider;
        rider.hp += extraHp;
    }

		@Override
    public String toString() {
        return "말 (추가체력: %d)".formatted(extraHp);
    }
}
```
Horse 클래스는 어떤 유닛이 탈 지 모르기 때문에 제너릭으로 extends Unit 조건을 주어서 선언되었다.

☕ Main.java
```java
				Swordman r_swordman1 = new Swordman(Side.RED);
        Knight r_knight1 = new Knight(Side.RED);
        Knight r_knight2 = new Knight(Side.RED);
        MagicKnight r_magicKnight1 = new MagicKnight(Side.RED);

        Knight b_knight1 = new Knight(Side.BLUE);
        MagicKnight b_magicKnight1 = new MagicKnight(Side.BLUE);
        MagicKnight b_magicKnight2 = new MagicKnight(Side.BLUE);

        Horse<Swordman> avante = new Horse<>(40);
        Horse<Knight> sonata = new Horse<>(50);

        avante.setRider(r_swordman1); // 🔴
        sonata.setRider(b_magicKnight1);

        r_swordman1.defaultAttack(b_knight1); // 🔴
        r_knight1.defaultAttack(b_magicKnight1);
        r_knight2.switchWeapon();
        r_knight2.defaultAttack(b_magicKnight2);

        b_magicKnight1.defaultAttack(r_swordman1);
        b_magicKnight2.lighteningAttack(new Unit[] {
                r_knight1,
                r_knight2,
                r_magicKnight1
        });
```
Horse선언 부분을 보자.
Swordman이 제너릭 타입으로 가게 되면 Swordman을 상속받은 모든 클래스가 가능해진다.
Knight 타입을 제너릭 타입으로 받게 되면 Knight를 상속받는 magicKnight까지 탈 수 있게된다.

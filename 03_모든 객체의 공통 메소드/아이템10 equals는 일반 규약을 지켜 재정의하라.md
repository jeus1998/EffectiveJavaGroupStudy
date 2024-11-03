# 아이템10 equals는 일반 규약을 지켜 재정의하라

### equals 메소드 

```java
public class Object {
    public boolean equals(Object obj) {
        return (this == obj);
    }
}
```
- equals 메소드는 재정의하기 쉬워 보이지만 함정이 많다.
- 잘못 재정의 하면 끔찍한 결과를 초래한다.
- 문제를 회피하는 가장 쉬운 길은 Object에 정의된 메소드를 그대로 사용하는 것이다.

### equals 메소드 재정의 하지 말자

지금부터 equals 메소드를 재정의 하지 않아도 되는 여러가지 케이스에 대해서 설명하겠다. 
- `각 인스턴스가 본질적으로 고유하다.`
  - 값을 표현하는 인스턴스가 아닌 동작하는 개체를 표현하는 클래스가 해당 (Thread)
- `인스턴스의 논리적 동치성을 검사할 일이 없다.`
  - Pattern 같이 equals를 재정의해서 두 인스턴스가 같은 정규 표현식을 나타내는지를 검사할 수 있지만 
    클라이언트가 이 방식을 원하지 않거나 필요 없다고 판단하면 굳이 재정의 하지 않아도 된다
- `상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.`
```java
public class SetTest {
    public static void main(String[] args) {
        Set<Integer> set1 = new HashSet<>();
        set1.add(1);
        set1.add(2);

        Set<Integer> set2 = new HashSet<>();
        set2.add(1);
        set2.add(2);

        System.out.println(set1.equals(set2)); // true
    }
}
``` 
- Set 구현체는 AbstractSet에서 재정의한 equals 메소드를 상속 받아서 사용한다.

AbstractSet equals API
```java
public boolean equals(Object o) {
    if (o == this)
        return true;

    if (!(o instanceof Set))
        return false;
    Collection<?> c = (Collection<?>) o;
    if (c.size() != size())
        return false;
    try {
        return containsAll(c);
    } catch (ClassCastException | NullPointerException unused) {
        return false;
    }
}
```
- Map 구현체: AbstractMap에서 재정의 
- List 구현체: AbstractList에서 재정의 

`메소드를 호출할 일이 없는 경우`
- 클래스가 private 이거나 package - private 이면 equals 메소드를 호출할 일은 없다.
  
### equals 메소드 언제 재정의?

```text
두 객체가 물리적으로 같은가를 검사하는 게 아닌 논리적 동치성을 확인해야 하는데 
상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때 equals 메소드를 재정의 한다. 
```
- 주로 값 클래스들이 재정의하는 케이스에 속한다. 
  - ex) Integer, String 
- 값 클래스라 하더라도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면
  equals 메소드를 재정의 하지 않아도 된다. 
  - ex) Enum 
- 이런 클래스에서는 어차피 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않는다. 
  그 말은 논리적 동치성과 객체 식별성이 사실상 같은 의미이다. 

### equals 메소드 규약 

`반사성(reflexivity)`
```text
null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true 
객체는 자기 자신과 같아야 함.
```

`대칭성(symmetry)`
```text
null이 아닌 모든 참조 x, y에 대해, x.equals(y)가 true면 y.equals(x) 또한 true 
두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻 
```
```java
public class CaseInsensitiveString {
    private final String s;
    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        if (o instanceof String)  
            return s.equalsIgnoreCase((String) o);
        return false;
    }
    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";

        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);
		
        System.out.println(cis.equals(s)); // true
        System.out.println(list.contains(s)); // false
    }
}
```
```text
현재  CaseInsensitiveString equals 메소드는 한 방향으로만 동작한다. 
CaseInsensitiveString에서 논리적 동치를 의미하는 "Polish"와 "polish"는 
CaseInsensitiveString equals에서 true로 나오지만 String의 equals에서는 false로 나온다. 
그래서 list.contains(s) = false 가 나온다.
```

String과의 연동을 포기 
```java
@Override public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString2 && ((CaseInsensitiveString2) o).s.equalsIgnoreCase(this.s);
}

// 테스트
CaseInsensitiveString2 cis = new CaseInsensitiveString2("Polish");

List<CaseInsensitiveString2> list = new ArrayList<>();
list.add(cis);

CaseInsensitiveString2 cis2 = new CaseInsensitiveString2("polish");
System.out.println(list.contains(cis2)); // true
```
```text
String 과의 연동성을 포기하고 같은 CaseInsensitiveString(2)이고 가지고 있는 문자가 
대소문자 구분 없이 같으면 true 리턴 
논리적 동치성을 잘 구현하여 List 에서도 정상 동작하게 만들었다. 
```

`추이성(transitivity)`
```text
null이 아닌 모든 참조 x,y에 대해 x.equals(y)가 true고 y.equals(z)도 true면 x.equals(z) 또한 true 
이 조건은 간단하지만 상위 클래스를 상속해서 받는 하위 클래스에 필드를 추가하는 케이스에서 많이 실수한다.
```

Point 클래스
```java
public class Point {
    private final int x;
    private final int y;
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    @Override
    public boolean equals(Object o) {
         if(!(o instanceof Point)) return false;
         Point p = (Point) o;
         if(p.x == this.x && p.y == this.y) return true;
         return false;
    }
}
```

ColorPoint 클래스 - 상위 클래스에 정의된 equals 메소드 그대로 사용 
```java
public class ColorPoint extends Point {
    private final String color;
    public ColorPoint(int x, int y, String color) {
        super(x, y);
        this.color = color;
    }
    // TODO equals 메소드 재정의
}
```

클라이언트
```java
public class App {
    public static void main(String[] args) {
        Point p1 = new Point(1, 2);
        ColorPoint p2 = new ColorPoint(1, 2, "red");
        System.out.println(p1.equals(p2)); // true
        System.out.println(p2.equals(p1)); // true
    }
}
```
```text
p1과 p2는 똑같은 x,y 값을 가지고 있어서 둘 다 true가 나온다.
이는 equals 규약을 어기는 행위는 아니지만 ColorPoint의 필드인 color를 제외하고 비교하게 되었다. 
```

추이성 위배 코드 
```java
public class ColorPoint extends Point{
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    @Override
    public boolean equals(Object o) {
        if(!(o instanceof Point)) return false;

        // o가 Point 타입이라면 색상을 무시한다.
        if(!(o instanceof ColorPoint)) return o.equals(this);

        return super.equals(o) && ((ColorPoint) o ).color == this.color;
    }
    public static void main(String[] args) {
        ColorPoint p1 = new ColorPoint(1, 2, Color.GREEN);
        Point p2 = new Point(1, 2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.RED);

        System.out.println(p1.equals(p2)); // true
        System.out.println(p2.equals(p3)); // true
        System.out.println(p1.equals(p3)); // false
    }
}
```
- 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다. 

`일관성(consistency)`
- null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

`null-아님`
- null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false 

### 상속이 아닌 컴포지션을 통한 값 추가하기 

```text
구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다. 
하지만 상속이 아닌 컴포지션을 사용하면 equals 규약을 지킬 수 있다. 
```

```java
public class ColorPoint {
    private final Point point;
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = color;
    }
    @Override
    public boolean equals(Object o) {
        if(!(o instanceof ColorPoint)) return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(this.point) && cp.color == this.color;
    }
    public static void main(String[] args) {
        ColorPoint p1 = new ColorPoint(1, 2, Color.BLUE);
        Point p2 = new Point(1, 2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.RED);

        System.out.println(p1.equals(p2)); // false
        System.out.println(p2.equals(p3)); // false
        System.out.println(p1.equals(p3)); // false
    }
}
```

### 양질의 equals 메소드 구현 방법

`1. == 연산자를 사용해 입력이 자기 자신의 참조인지를 확인한다.`
```text
이는 단순한 성능 최적화용이다. 
비교 작업이 복잡하고 리소스 사용이 많다면 값어치를 할 것이다. 
ex) 크기가 큰 컬렉션끼리 비교 
```

`2. intanceof 연산자로 입력이 올바른 타입인지 확인한다. `
```text
보통 올바른 타입인지 확인은 equals가 정의된 클래스지만 가끔 그 클래스가 구현한 인터페이스가 될 수도 있다. 
어떤 인터페이스는 자신을 구현한 서로 다른 클래스끼리도 비교할 수 있도록 equalse 규약을 수정하기도 한다. 
이런 인터페이스를 구현한 클래스라면 equals에서 클래스가 아닌 해당 인터페이스를 사용하자.
ex) List, Map, Map.Entry, Set등의 컬렉션 인터페이스
```
추상 리스트 (Abstract List) equals 메소드 
```java
public boolean equals(Object o) {
    if (o == this)
        return true;
    if (!(o instanceof List))
        return false;
    // 생략 ..    
}
```

`3. 입력을 올바른 타입으로 형변환 `
```text
이미 2번에서 instanceof로 올바른 타입인지 체크했기 때문에 ClassCastException이나 NullPointerException은 절대 발생하지 않음을 보장할 수 있다. 
```

`4. 입력 객체와 자기 자신의 대응되는 필드들이 모두 일치하는지 하나씩 검사한다 `

`5. equals를 재정의할 땐 hashCode도 반드시 재정의하자 (아이템 11)에서 설명 `

`6. 비교 방법 + 예외 `
```text
기본 타입들은 == 참조 타입은 equals 비교를 하자. 
기본 타입인 double, float는 부동 소수점이라는 개념이 있다. 
그러니 == 비교를 하지 말자. 
대신 Float.compare(float, float) & Double.comare(double, double)를 사용하자 (정적 메소드)
Float.equals와 Double.equals 메소드를 대신 사용할 수 있지만 이는 오토 방식을 수반할 수 있으니 성능상 좋지 않다. 
배열은 Arrays.equals 메소드들 중에서 하나를 사용하자.
```
`7. 성능을 위해 비용이 싼 필드를 먼저 비교하자. `

`8. Object 외 타입을 매개변수로 받는 equals 메소드는 선언하지 말자 `

`9. equals 구현이 끝났다면 세 가지만 자문하자.`
```text
대칭적인가?
추이성이 있는가?
일관적인가?
자문에서 끝나지 말고 단위 테스트를 통해서 확실하게 확인하자
```

### 정리 

```text
꼭 필요한 경우가 아니면 equals를 정의하지 말자
재정의해야 할 때는 클래스의 핵심 필드들은 전부 빠지없이 규약을 지키며 비교하자.

equals를 테스트하는 코드는 항상 뻔하고 지루하다. 
이 작업을 대신해줄 오픈소스인 AutoValue 프레임워크를 고려하자.
```


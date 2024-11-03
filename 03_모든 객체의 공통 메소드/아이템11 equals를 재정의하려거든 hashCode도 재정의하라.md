# 아이템11 equals를 재정의하려거든 hashCode도 재정의하라

### 해시(Hash)

- 이번 아이템을 이해하려면 먼저 해시에 대해서 알아야 한다. 
- [해시 정리 자료](https://20240228.tistory.com/86)

### 해시 코드(HashCode)

```text
equals를 재정의한 클래스 모두에서 hashCode 또한 재정의해야 한다. 
그렇지 않으면 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킨다. 
```

### Object 명세 해시 코드 규약 

`1. 일관성 `
```text
equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 
hashCode 메소드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
```

`2. equals와 동일 o`
```text
equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다. 
```

`3. equals와 동일 x`
```text
equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.
단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다. 

같은 값을 반환하면 해시 충돌이 일어난다. 즉, 해당하는 값을 찾는데 추가적으로 시간을 소비한다.
chaining: 링크드 리스트나 트리 순회에 걸리는 시간 
open addressing: 다른 빈 커빗을 찾는 시간 
```

### Object의 기본 해시 코드 메소드 

```java
@IntrinsicCandidate
public native int hashCode();
```

ObjectExample
```java
public class ObjectExample {
    private final int a;
    public ObjectExample(int a) {
        this.a = a;
    }
    @Override
    public boolean equals(Object o) {
        if(o instanceof ObjectExample){
            return ((ObjectExample) o).a == this.a;
        }
        return false;
    }
    public static void main(String[] args) {
        ObjectExample t1 = new ObjectExample(1);
        ObjectExample t2 = new ObjectExample(1);

        System.out.println(t1.equals(t2)); // true
        System.out.println(t2.equals(t1)); // true

        System.out.println(t1.hashCode()); // 1044036744
        System.out.println(t2.hashCode()); // 1826771953
        
        Set<ObjectExample> set = new HashSet<>();
        set.add(t1);
        System.out.println(set.contains(t2)); // false
    }
}
```
- 해당 클래스는 Object의 hashCode를 그대로 사용한다. 
- HashSet에 t2와 논리적 동치를 가지는 t1을 넣고 set의 contains 메소드를 실행하면 false가 나온다.
- 당연하게 hashCode를 재정의하지 않았기 때문에 각 hashCode가 서로 다른 값을 반환하기 때문이다. 
- 이 문제는 ObjectExample 클래스에 적절한 hashCode 메소드만 작성해주면 해결된다. 
- 올바른 hashCode 정의란 무엇일까?

### 최악의 (하지만 적법한) hashCode 구현 - 절대 사용 금지!

```java
@Override
public int hashCode() {
    return 42;
}
```
- 모든 객체에 대해서 동일한 해시 값을 반환한다.
- 하지만 모든 객체가 해시테이블의 버킷 하나에 담겨서 링크드 리스트에 담겨서 동작한다. 
- `참고`: 이는 해시 충돌을 피하기 위해서 사용하는 구현 방법인 chaining 기법이다. 
- `참고`: 자바 8 이후로는 같은 버킷 내 엔트리 수가 8개 이상이 되면 레드-블랙 트리(자가 균형 트리)에 담긴다. 
```text
이런 최적화를 했지만 보통 해시테이블의 기대 평균 수행 시간은 O(1)이지만 
자바 7 이전 O(N) 링크드 리스트만 사용 자바 8 이후 O(logN)의 수행 시간이 나온다. 
최적화가 되어있긴 하지만 여전히 객체가 많아지면 도저히 쓸 수 없게 된다. 
따라서 hashCode 메소드는 가능한 객체별로 고르게 분포된 해시 값을 반환하도록 설계해야 한다. 
```

### 규약과 성능을 고려하는 hashCode 메소드 

```text
좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다.
이상적인 해시 함수는 주어진 서로 다른 인스턴스들은 32비트 정수 범위에 균일하게 분배해야 한다. 
```

`1. int 변수 result를 선언한 후 c로 초기화한다. `
```text
c는 해당 객체의 첫 번째 핵심 필드를 2.a 방식으로 계산한 해시코드다. 
참고: 핵심 필드란 equals 비교에 사용되는 필드를 의미한다. (아이템 10 참고)  
```

`2. 해당 객체의 나머지 핵심 필드 f에 각각에 대한 다음 작업을 수행한다. `
```text
a: 해당 필드의 해시코드 c를 계산한다. 

기본 타입 필드라면 Type.hashCode(f)를 수행한다. 
여기서 Type은 해당 기본 타입의 박싱 클래스이다. 
참조 타입 필드면서 이 클래스의 equals 메소드가 이 필드의 equals 메소드를 재귀적으로 호출하면, 이 필드의 hashCode를 재귀적으로 호출한다.
필드의 값이 null이면 0을 사용한다. 
필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 
각각을 재귀적으로 적용해 2.b 방식으로 갱신한다. 
배열에 핵심 원소가 없다면 0을 사용한다. 
모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다. 

b: 2.a에서 계산한 해시코드를 result로 갱신한다. 
result = result * 31 + (2.a 계산한 해시 코드)
```

`3. result를 반환한다. `

example
```java
public class BirthDate {
    private final int year;
    private final int month;
    private final int day;
    // 범위 검사는 pass 하겠다.
    public BirthDate(int year, int month, int day) {
        this.year = year;
        this.month = month;
        this.day = day;
    }
    @Override
    public boolean equals(Object o) {
        if(!(o instanceof BirthDate)) return false;
        BirthDate bd = (BirthDate) o;
        return bd.year == year && bd.month == month && bd.day == day;
    }
    @Override
    public int hashCode() {
        // 첫 번째 필드의 해시코드를 result 저장
        int result = Integer.hashCode(year);
        result = result * 31 + Integer.hashCode(month);
        result = result * 31 + Integer.hashCode(day);
        return result;
    }
}
```

테스트 
```java
public static void main(String[] args) {
    BirthDate birthDate1 = new BirthDate(1998, 12, 12);
    BirthDate birthDate2 = new BirthDate(2024, 11, 1);

    Set<BirthDate> set = new HashSet<>();
    set.add(birthDate1);

    System.out.println(set.contains(new BirthDate(1998, 12, 12))); // true
    System.out.println(set.contains(birthDate2)); // false
}
```
- equals 비교에 사용되지 않는 필드는 반드시 제외하자
- 단계 2.b의 곱셈 31 * result는 필드를 곱하는 순서에 따라서 result 값이 달라지게 한다.
  그 결과 클래스에 비슷한 필드가 여러 개일 때 해시 효과를 크게 높여준다. 

곱할 숫자를 31로 정한 이유는? 
```text
31은 홀수이면서 소수(prime)이다. 

만약 이 숫자가 짝수이고 오버플로가 발생하면 정보를 잃게 된다. 

2를 곱하는 것은 시프트 연산과 같은 결과를 내기 때문이다. 
```

### Objects 클래스 제공 hash 메소드 

```java
// Objects hash
public static int hash(Object... values) {
    return Arrays.hashCode(values);
}

// Arrays hashCode
public static int hashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a)
        result = 31 * result + (element == null ? 0 : element.hashCode());

    return result;
}
```

```text
Objects 클래스는 임의의 개수만큼 객체를 받아 해시코드로 계산해주는 정적 메소드인 hash를 제공한다.
이 메소드를 활용하면 앞서의 요령대로 구현한 코드와 비슷한 수준의 hashCode 함수를 단 한 줄로 작성할 수 있다. 
하지만 속도는 더 느리다. 
입력 인수를 담기 위한 배열이 만들어지고, 입력 중에 기본 타입이 있다면 박싱 언방싱도 거친다. 
그러니 Objects가 제공하는 hash 메소드는 성능에 민감하지 않다면 사용하자. 
```

### 해시 코드 캐싱

```text
클래스가 불변 클래스이고 해시 코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 
캐싱하는 방식을 고려해야 한다.
또한 hashCode가 처음 불릴 때 계산하는 지연 초기화 전략 또한 사용이 가능하다.
물론 이 방식은 스레드에 안전하게 신경 써야 한다. 
```

```java
public class HashCaching {
    private final int a;
    private final int b;
    private int hashCode;
    // equals 메소드 생략
    public HashCaching(int a, int b) {
        this.a = a;
        this.b = b;
    }
    @Override
    public int hashCode() {
        int result = hashCode;
        if(result == 0){
            result = Integer.hashCode(a);
            result = result * 31 + Integer.hashCode(b);
        }
        return result;
    }
}
```
- 현재 객체에 hashCode가 지연 초기화하기 전에 여러 스레드가 접근하면 잘못된 값을 가져갈 가능성이 있다. 

```java
public class HashCaching {
    private final int a;
    private final int b;
    private int hashCode;
    // equals 메소드 생략
    public HashCaching(int a, int b) {
        this.a = a;
        this.b = b;
    }
    @Override
    public int hashCode() {
        int result = hashCode;
        if(result == 0){
            synchronized (this){
                result = hashCode;
                if(result == 0){
                    result = Integer.hashCode(a);
                    result = result * 31 + Integer.hashCode(b);    
                }
            }
        }
        return result;
    }
}
```

### 주의점 & 정리 

```text
성능을 높인다고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다. 
해시 코드를 계산하는 속도는 빨라지겠지만, 해시 품질이 나빠져  해시 테이블의 성능을 심각하게 떨어뜨릴 수도 있다. 
특히 어떤 필드는 특정 영역에 몰린 인스턴스들의 해시코드(클러스터링)을 넓은 범위로 고르게 퍼트려주는 효과가 있을 수도 있다. 

equals 메소드를 정의할 때 hashCode 또한 반드시 재정의하자.
그렇지 않으면 프로그램에 제대로 동작하지 않을 것이다. 
재정의한 hashCode는 규약을 꼭 따르자.
아이템10에서 이야기한 AutoValue 프레임워크를 고려하자.
```
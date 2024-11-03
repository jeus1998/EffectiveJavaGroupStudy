해당 인터페이스의 compareTo는 Object의 공통 메서드는 아니지만
그래도 값을 나타내는 클래스가 구현할법한 공통 인터페이스이므로 해당 메서드를 이 장에 넣은듯합니다.

Comparable을 구현했다는 것은 그 클래스들에는 자연적인 순서가 있음을 의미합니다.
그래서 Comparable을 구현한 `객체들은 손쉽게 정렬`이 가능합니다.
가장 쉽게 볼 수 있는 예는 Collections.sort나 Arrays.sort()에 사용됩니다.
즉 해당 인터페이스를 구현하면 이 객체는 객체만의 크기값이 존재하고
객체의 비교가 가능해진다고 보면 됩니다.

compareTo 메서드의 일반 규약은 equals의 규약과 비슷합니다.
> 이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를
> 같으면 0일 크면 양의 정수를 반환합니다. 비교할 수 없는 타입의 객체가 주어지면 ClassCastException이 발생합니다.
>  x.compareTo(y) == - y.compareTo(x)
>  x.compareTo(y) > 0 , y.compareTo(z) > 0 이라면 x.compareTo(z) > 0
>  x.compareTo(y) == 0 이라면 x.compareTo(z) == y.compareTo(z)

복잡하다고 겁먹지 않아도 된다. equals 규약처럼 통상적으로 이해 가능한 규약들이다.
compareTo는 타입이 다른 객체를 신경쓰지 않아도 된다.
타입이 다른 객체라면 ClassCastException을 던져주면 된다.

그리고 인수의 타입이 잘못됐다면 컴파일 자체가 되지 않는다. 또한 null을 인수로 넣어 호출하면 NPE를 던져야 한다. 
compareTo 메서드는 각 필드가 동치인지를 비교하는 게 아니라 **그 순서를 비교한다**.

일련적으로 필드를 비교할때는 compareTo를 재귀적으로 호출하는것을 사용합니다.
그리고 
자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었습니다.
이 비교자들을 Comparable 인터페이스가 원하는 compareTo 메서드를 구현하는데 멋지게 사용할 수 있습니다.

```Java
private static final Comparator<PhoneNumber> COMPARATOR =
	comparingInt ((PhoneNumber pn) -> pn.areaCode)
		.thenComparingInt(pn -> pn.prefix)
		.thenComparingInt(pn -> pn.lineNum)

public int compareTo(PhoneNumber pn) {
	return COMPARATOR.compare(this, pn)
}
```

박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 클래스를 사용하자
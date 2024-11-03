# 아이템13 clone 재정의는 주의해서 진행하라.
해당 아이템을 살펴보기 전에 Clonnable 인터페이스와 clone 메서드, Object.clone을 살펴보자
![[Pasted image 20241103143741.png]]

객체를 복제하고 싶다면 Clonnable 인터페이스를 구현하면 된다.
즉 Clonnble 인터페이스를 구현하고 있다면 해당 구현체가 복사될 수 있다는 의미입니다.

여기서보면 Clonnable 인터페이스는 Object.clone()메서드를 위임할
뿐 무언가 일을 하지는 않습니다. 따라서 `구현만 한다해서 복제가 성공하리라는 보장은 없습니다.`

그리고 위의 문단을 보면 일반적으로 `field-for-field copy`로 복사를 한다고 합니다.
즉 깊은 복사를 통해 필드들이 복사됩니다.

책으로 다시 돌아와서 보자면 
`실무에서 Clonnable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 되리라 기대한다.`
해당 기대를 만족하려면 모든 상위 클래스는 복잡하고, 강제할 수 없고, 허술하게 작성되야합니다.
`매우 위험하고 모순적인 매커니즘이 탄생하게 됩니다.` 
`생성자를 호출하지 않고도 객체를 생성할 수 있게 됩니다.`

과연 Clonnable의 구현과 객체의 안정성과 일관성중 어떤게 더 중요할까요?
저는 후자가 더 중요하다고 생각합니다.

79p 
clone 메서드로 얻은 객체는 원본의 완벽한 복제본입니다.
모든 필드가 기본 타입이거나 불변 객체를 참조한다면 이 객체는 완전히 우리가 원하는 상태라 더 손볼 것이 없습니다.
`하지만 이런 객체라면 굳이 clone 메서드를 제공하지 않는 게 좋습니다.`

```Java
@Override
public PhoneNumber clone() {
	try {
		return (PhoneNumber) super.clone()
	} catch (CloneNotSupportedException e) {
		throw new AssertionError(); // 일어날 수 없는 일
	}
}

```
Object.clone()은 Object를 반환하지만 해당 메서드는 PhoneNumber를 반환하게 되었다.
보통 이런 방식이 권장된다. (본인의 타입을 갖도록)

clone 메서드가 검사예외가 아닌 비 검사 예외를 던졌어야 한다.(아이템 71)

#### 하지만 클래스가 가변 객체를 참조하는 순간 재앙으로 돌아온다

```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16; 

	public Stack() {
		...일반적인 구현 
	}

	public void push(Object e) {
		... 일반적인 구현
	}

	public Object pop() {
		... 일반적인 구현
	}
}
```
만약 이 클래스를 복제할 수 있게 만든다면 단순히 clone 메서드가 super.clone을 반환한다면 어떻게 될까?

elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조할 것입니다.
원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다는 이야기입니다.
따라서 프로그램이 이상하게 동작하거나 NPE를 발생시킬 수 있습니다.

`clone 메서드는 사실상 생성자와 같은 효과를 냅니다. `
`즉 clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 합니다.`

가장 쉬운 방법은 element 배열의 clone을 재귀적으로 호출해주는 것입니다.
```Java
@Override public Stack clone() {
	try {
		Stack result = (Stack) super.clone();
		result.elements = elements.clone();
		return result;
	} catch (CloneNotSupportedException e) {
		throw new AssertionError();
	}
}
```
배열을 복사할때는 배열의 clone 메서드를 사용하라고 권장합니다.
한편 elements 필드가 final이었다면 해당 방법은 작동하지 않습니다.(객체의 생성자 안에서만 초기화 가능)
이게 바로 앞서 말한 클래스가 허술하게 작성된다는 단점입니다.
`그리고 Clonnable 아키텍처는 '가변 객체를 참조하는 필드로 final로 선언하라'는 일반 용법과 충돌합니다.`

해당 배열을 복사하는 방법보다 더 복잡한 케이스들이 존재하고 `상속을 할때도 문제가 발생할 수 있습니다.`
스레드 안전 클래스를 작성할 때는 clone 메서드 역시 적절하게 동기화해줘야하는 번거로움도 존재합니다.
Clonnable 인터페이스는 전형적인 부모 클래스에 의존하게되는 문제가 발생합니다.

해당 책에서는 Clonnable 인터페이스의 대안으로는 정적 팩터리 메서드를 사용하는것을 제안하고 있습니다.

논외적으로 이런 객체의 복제를 언어 스펙에서 지원하는것이
kotlin의 data class의 copy 함수이다.
copy 함수는 객체를 복제하는데 도움을 주지만 복제 방식(깊은 복사)에 따른 단점도 존재한다.


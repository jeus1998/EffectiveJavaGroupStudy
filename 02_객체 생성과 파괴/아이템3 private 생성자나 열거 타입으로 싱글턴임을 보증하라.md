## 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
* 싱글턴 : 인스턴스를 오직 하나만 생성할 수 있는 클래스
  * ex) 함수와 같은 무상태(stateless) 객체, 유일해야 하는 시스템 컴포넌트 <br>
  * 단점) 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트 하기가 어려워질 수 있음
### 싱글턴 만드는 방식 - 1. public static 멤버가 final 필드인 방식
* 생성자는 private으로 감추기 <br>
* 유일한 접근 수단으로 public static final 필드 만들어놓기
```java
public class Elvis{
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}
```
> INSTANCE이 필드가 초기화 될 때 딱 한 번만 private 생성자가 호출됨<br>
> 장점 1) final이므로 API 상에 클래스가 싱글턴임이 드러남<br>
> 장점 2) 간결함
### 싱글턴 만드는 방식 - 2. 정적 팩토리 메서드 이용
* 생성자는 private으로 감추기 <br>
* private 인스턴스를 정적 팩토리 메서드가 반환하도록 하기
```java
public class Elvis{
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}
```
> 장점 1) API를 변경하지 않아도 싱글턴이 아니게 변경할 수 있다. 그렇게 되면 스레드 별로 다른 인스턴스를 반환하게 할 수 있음 <br>
> 장점 2) 정적 팩토리를 제네릭 싱글턴 팩토리로 만들 수 있다
  Q. 제네릭 싱글턴 팩토리? <br>
> 장점 3) 정적 팩토리 메서드 참조를 공급자로 사용할 수 있다
  Q. 공급자? suplier? <br>
### 주의사항 - 직렬화 시
* 위의 두 방법 모두 싱글턴 클래스의 직렬화를 원하는 경우, 모든 인스턴스 필드를 일시적이라고 선언하고 readResolve 메서드를 제공하도록 하여야 한다.
* 그렇지 않으면 역직렬화 시 객체가 추가적으로 생길 수 있음
### 싱글턴 만드는 방식 - 3. 열거 타입 선언
* 원소가 하나뿐인 열거 타입으로 선언하기
```java
public enum Elvis{
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```
> 장점 1) 간결 <br>
> 장점 2) 바로 직렬화 가능 <br>
> 장점 3) 복잡한 직렬화 상황 or 리플렉션 공격에도 제 2의 인스턴스가 생기는 일을 차단 해줌
### ❗대부분 상황에서는 원소가 하나뿐인 열거 타입을 이용하여 싱글턴을 만드는 것이 가장 좋은 방법이다. 단, Enum외의 클래스를 상속해야하는 싱글턴의 경우 이 방법은 사용할 수 없다

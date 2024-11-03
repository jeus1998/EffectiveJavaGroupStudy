# 아이템12 toString을 항상 재정의하라

### Object의 기본 toString 메소드 

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

```text
Object의 기본 toString 메소드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 없다. 

단순하게 "클래스_이름@16진수로표현한해시코드"를 반환할 뿐이다.
```

```java
public class BirthDate {
    private final int year;
    private final int month;
    private final int day;
    public BirthDate(int year, int month, int day) {
        this.year = year;
        this.month = month;
        this.day = day;
    }
}    
public static void main(String[] args) {
    BirthDate birthDate = new BirthDate(1998, 12, 12);
    System.out.println(birthDate);
}

// com.example.effectivejavacode._03.item12.BirthDate@1dc22d
```

```text
toString의 일반 규약에 따르면 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다. 
또한 모든 하위 클래스에서 이 메소드를 재정의하라 한다. 
```

```java
@Override
public String toString() {
    return year + ":" + month + ":" + day;
}

// 1998:12:12
```
- toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁다?

### toString 재정의 필요성 

```text
toString 메소드는 꼭 우리가 직접 호출하지 않아도 어딘가에서 쓰인다. 
ex) println, printf, 문자열 연결 연산자(+), assert()

만약 우리가 작성한 객체를 참조하는 컴포넌트가 오류 메시지를 로깅할 때 자동으로 호출할 수 있다.
만약 toString을 제대로 재정의하지 않았다면 쓸모없는 메시지만 로그에 남는다. 
```

### 양질의 toString 재정의 방법

```text
보통 toString은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다.

하지만 객체가 거대하거나 객체의 상태가 문자열로 표한기에 적합하지 않다면 무리가 있다. 
ex) 서울 거주자 전화번호부 (n개) 
ex) Thread 

이런 상황이라면 요약 정보를 담아야 한다. 
이상적으로는 스스로의 상태를 완벽하게 설명하는 문자열이어야 한다. 
```

포맷 
- 전화번호처럼 포맷이 정해져 있는 경우, 주석을 달아서 문서화를 해줄 수도 있다. 
```java
/**
 * 이 전화번호의 문자열 표현을 반환한다.
 * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
 * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
 * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
 *
 * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
 * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
 * 전화번호의 마지막 네 문자는 "0123"이 된다.
 */
@Override public String toString() {
    return String.format("%03d-%03d-%04d",areaCode, prefix, lineNum);
}
```
```text
하지만 포맷이 정해지면 앞으로의 유지보수에도 이 포맷을 사용해야 한다. 
그러니 신중하게 포맷을 정할 필요가 있다. 
또한 명시한 포맷에 맞게 정적 팩터리나 생성자를 함께 제공해주면 좋다.
```

포맷을 사용하지 않는다면?
```java
/**
* 생년월일을 반환한다.
* 이 설명의 일반적인 형태나 상세 형식은 정해지지 않았다.
* ex) BirthDate{1998:12:12}
*/
@Override
public String toString() {
return "BirthDate{" + year + ":" + month + ":" + day + "}";
}
```
- 생년월일은 사실 변경 가능성이 적어서 포맷도 나쁘지 않아 보인다. 
# 아이템8 - finalizer와 cleaner 사용을 피하라

### Finalizer와 Cleaner

- 자바는 2가지 소멸자를 제공한다. (Finalizer & Cleaner)
- Finalizer는 예측 불가능하고, 위험하며, 대부분 불필요하다. 
- 오동작, 낮은 성능, 이식성 문제의 원인이 된다. 
- Cleaner 또한 Finalizer 보다 덜 위험하지만, 여전히 예측할 수 없고, Finalizer 단점과 거의 비슷하다. 

Finalizer 이식성 문제 
- 자바 9에서 Finalizer를 deprecated API로 지정하고 Cleaner를 그 대안으로 소개했다. 

자바에서의 자원 회수 
- try-with-resources
- try-finally 

### Finalizer & Cleaner 단점: 실행 예측 불가능 

```java
public class FinalizerExample {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("Clean Up");
    }
    public void hello() {
        System.out.println("hello");
    }
}
```
- FinalizerExample 클래스는 finalize() 메소드를 오버라이딩 하였다.
- Object 클래스에도 메소드만 존재하지 별다른 로직은 없다. 

```java
public class SampleRunner {
    public static void main(String[] args) throws InterruptedException {
        SampleRunner runner = new SampleRunner();
        runner.run();
        Thread.sleep(1000l);
    }
    private void run() {
        FinalizerExample example = new FinalizerExample();
        example.hello();
        // 어디에서도 참조x -> GC 대상
    }
}

// 출력 hello
```
- run() 메소드 이후 example이 가리키는 FinalizerExample 객체는 어디에서도 참조되지 않는다.
- 이는 GC의 대상이 된다. 
- 하지만 GC의 대상이 되는 거지 바로 GC가 처리하지는 않는다. 
- 즉 finalize() 메소드 호출도 마찬가지로 예측 불가능하다는 뜻이다. 
- finalizer & cleaner를 얼마나 신속히 수행할지는 전적으로 가비지 컬렉터의 알고리즘에 달렸으며, 이는 가비지 컬렉터 구현마다 천차만별이다. 

```text
책에서는 파일 닫기를 예시로 말한다. 
파일 닫기를 finalizer & cleaner에게 맡긴다면 중대한 오류를 일으킬 수 있다.
시스템이 동시에 열 수 있는 파일의 개수가 한계가 있기 때문이다. 
시스템이 finalizer / cleaner 실행을 게을리해서 파일을 계속 열어둔다면 새로운 파일을 열지 못한다. 
```

### Finalizer & Cleaner 단점: 인스턴스 반납 지연

```text
Finalizer 쓰레드는 우선 순위가 낮아서 언제 실행될지 모른다.
따라서, Finalizer 안에 어떤 작업이 있고, 그 작업을 쓰레드가 처리 못해서 대기하고 있다면, 해당 인스턴스는 GC가 되지 않고 계속 쌓이다가 결국엔 OutOfMomoryException이 발생할 수도 있다.
Cleaner는 별도의 쓰레드에서 동작하니까 이 부분에 있어서 조금은 나을 수 있다.
하지만 여전히 쓰레드는 백그라운드에서 동작하고 가비지 컬렉터의 통제하에 있으니 언제 처리될지는 알 수 없다. 
```

### Finalizer & Cleaner 단점: 수행 여부 보장 X

- 자바 언어 명세에서는 finalizer & cleaner의 수행 시점뿐 아니라 수행 여부조차 보장하지 않는다.
- 만약 데이터베이스 같은 자원의 락을 그것들로 반환하는 작업을 한다면 프로그램 자체가 중단될 수도 있다.
- 해당 메소드를 신뢰하지 말자 
  - `System.gc()`: 가비지 컬렉터 호출 
  - `System.runFinalization()`: 더이상 강한 참조가 없는 객체들이 가비지 컬렉션 후 finalizer 메소드 실행
  - finalizer & cleaner가 실행 가능성을 높여줄 수 있으나, 보장해주진 않는다.
- 그걸 보장해주겠다고 만든 System.runFinalizersOnExit와 그 쌍둥이 Runtime.runFinalizersOnExit은 둘다 망했고 수십년간 deprecated 상태다

### Finalizer & Cleaner 단점: 성능 문제 

```text
책 내용이다. 
AutoCloseable 객체로 만들고, try-with-resource로 자원 반납을 하는데 걸리는 시간은 12ns 인데 
반면에 Finalizer를 사용한 경우에 550ns가 걸린다. 약 50배 
Cleaner를 사용한 경우에는 66ns가 걸린다. 약 5배 
```

### Finalizer & Cleaner 단점: 보안 문제 

- finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다. 
- 공격 원리
  - 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 
    수행될 수 있게 한다.
  - 또한 이 finalizer는 정적 필드에 자신의 참조를 할당하여 GC 대상에서 벗어날 수 있다.
- 방어 방법
  - final 클래스는 그 누구도 하위 클래스를 만들 수 없으니 이 공격에 안전하다.
  - 또는 finalize() 메소드 자체를 final로 만들어도 방어가 가능하다. 

### Finalizer & Cleaner 대안? 

- AutoCloseable 인터페이스를 구현하고 클라이언트에서 인스턴스를 다 쓰고 나면 close 메소드를 호출하는 방법이다. 

AutoCloseable 인터페이스를 구현한 Resource 클래스 
```java
public class Resource implements AutoCloseable{
    @Override
    public void close() throws RuntimeException{
        System.out.println("close");
    }
    public void hello() {
        System.out.println("hello");
    }
}
```

try-with-resource 
```java
public class SampleRunner {
    public static void main(String[] args) throws InterruptedException {
        SampleRunner runner = new SampleRunner();
        runner.run();
    }
    private void run() {
        // try-with-resource
        try(Resource resource = new Resource()) {
            resource.hello();
        }
    }
}

// 출력 
hello
close
```
- try-finally 처럼 명시적으로 close() 메소드를 호출하지 않아도 자동으로 close() 메소드가 호출된다.

자바 10부터는 var 사용 가능 
```java
private void run() {
    // try-with-resource
    try(var resource = new Resource()) {
        resource.hello();
    }
}
```

### Finalizer 안전망으로 사용하기 

```text
자원의 소유자가 close 메소드를 호출하지 않는 것에 대비한 안전망 역할이다.
cleaner와 finalizer가 즉시 혹은 끝까지 호출되리라는 보장은 없지만 클라이언트가 하지 않은 자원 반납에 대해서 
늦게라도 해주는 것이 아예 안 하는 것 보단 낫다.
```

Finalizer use for safetynet
```java
public class Resource implements AutoCloseable{
    private boolean closed;
    @Override
    public void close() throws RuntimeException{
        if(closed){
            throw new IllegalStateException();
        }
        closed = true;
        System.out.println("close");
    }
    @Override
    protected void finalize() throws Throwable {
        if(!closed) close();
    }
    public void hello() {
        System.out.println("hello");
    }
}
```

```java
public class SampleRunner {
    public static void main(String[] args) throws InterruptedException {
        SampleRunner runner = new SampleRunner();
        runner.run();
        System.gc();
    }
    private void run() {
        // 명시적 close() 호출 x & try-catch 블록
        try {
           Resource resource = new Resource();
           resource.hello();
        }
        catch (Exception e){
            e.printStackTrace();
        }
    }
}
```
- 명시적으로 close() 호출도 하지 않고 try-with-resource도 사용하지 않았다.
- 하지만 finalizer에 close() 호출을 넣었기 때문에 close()가 System.gc() 가비지 컬렉터 호출 이후 출력되었다. 
- 그래도 이런 안정망 역할의 finalizer를 작성할 때는 그럴만한 값어치가 있는지 충분하게 고려하자.
- 자바 일부 finalizer 제공 라이브러리
  - FileInputStream, FileOutPutStream, ThreadPoolExecutor 

### Cleaner 안전망으로 사용하기 

```java
public class Resource implements AutoCloseable{
    private boolean closed;
    private static final Cleaner CLEANER = Cleaner.create();
    private final Cleaner.Cleanable cleanable;
    private final ResourceCleaner resourceCleaner;
    public Resource() {
        this.resourceCleaner = new ResourceCleaner(); // 스레드 생성
        this.cleanable = CLEANER.register(this, resourceCleaner);
    }
    // Cleaner: 별도의 스레드가 필요
    private static class ResourceCleaner implements Runnable{
        @Override
        public void run() {
            // TODO: 자원 반납
            System.out.println("Clean");
        }
    }
    @Override
    public void close() throws RuntimeException{
        if(closed){
            throw new IllegalStateException();
        }
        closed = true;
        System.out.println("close");
        cleanable.clean();
    }
    public void hello() {
        System.out.println("hello");
    }
}
```
- 클라이언트가 try-with-resource를 사용하여 close()이 호출 되거나 
- 아님 명시적으로 close()를 호출하면 내부에서 ResourceCleaner 쓰레드의 run 메소드가 동작하게 된다.
- 아니면 가비지 컬렉터가 Resource를 회수할 때 까지 클라이언트가 close()를 호출하지 않는다면 
- cleaner가 run() 메소드를 호출해줄 것이다.
- 주의점 
  - ResourceCleaner는 Resource 인스턴스를 참조해서는 안 된다. 
  - Resource 인스턴스를 참조할 경우 순환참조가 생겨 가비지 컬렉터가 Resource 인스턴스를 청소할 기회가 오지 않는다.


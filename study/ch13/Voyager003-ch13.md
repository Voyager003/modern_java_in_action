## 디폴트 메서드

보통 인터페이스는 규칙을 잡거나, 서비스의 플로우 로직을 잡는 데 사용한다.

하지만 인터페이스를 구현하는 클래스에서는 메서드를 모두 구현해야하기 때문에 인터페이스에 메서드를 추가할때 문제가 발생함. 메서드 하나를 추가하려면 해당 인터페이스를 구현하는 모든 클래스에서는 해당 메서드를 모두 구현해줘야 한다.

Java 8에서 이러한 문제를 해결하기 위해 2가지 방법을 제공한다.

1. 인터페이스 정적 메서드(static method)
2. 인터페이스의 기본 구현을 제공할 수 있도록 디폴트 메서드(default method)

인터페이스에 디폴트 메서드를 사용하여 메서드를 구현할 수 있다.

**디폴트 메서드를 이용하면 인터페이스의 기분 구현을 그대로 상속하므로 인터페이스에 자유롭게 새로운 메서드를 추가할 수 있게 된다.** 이로인해 호환성을 유지하면서 API를 바꿀 수 있다.

여기서 Java 프로그램을 바꾸는 것과 관련된 호환성 문제는 크게 바이너리 호환성, 소스 호환성, 동작 호환성 세가지로 분류할 수 있는데,

> **바이너리 호환성**은
> 뭔가를 바꾼 이후에도 에러 없이 기존 바이너리가 실행될 수 있는 상황을 바이너리 호환성이라고 한다.  
> -> 인터페이스에 메서드를 추가했을 때 추가된 메서드를 호출하지 않는 한 문제가 일어나지 않는 경우
>
> **소스 호환성**은
> 코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일할 수 있음을 의미한다.  
> -> 인터페이스에 메서드를 추가하면 추가한 메서드를 구현하도록 클래스를 고쳐야 하기 때문에 소스 호환성 문제가 발생
>
> **동작 호환성**은
> 코드를 반꾼 다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행한다는 의미이다.
-> 인터페이스에 메서드를 추가하더라도 프로그램에서 추가된 메서드를 호출할 일은 없으므로 동작 호환성은 유지된다.
## 디폴트 메서드 활용 패턴

선택형 메서드(optional method), 동작 다중 상속(multiple inheritance of behavior) 두 가지 방식이 있다.

### 선택형 메서드(optional method)

```java
public interface Iterator<E> {
	...
    
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }

    ...
}
```

Iterator 인터페이스를 보면 remove 메서드는 사용자들이 잘 사용하지 않아 Java8 이전에는 remove 기능을 무시했다.

결과적으로 Iterator를 구현하는 많은 클래스에서는 remove에 빈 구현을 제공했다. 

사용하지 않는 메서드를 디폴트 메서드로 구현해 사용하지 않는 클래스에서는 구현을 할 필요가 없어져 불필요한 코드를 줄일수 있다.
### 동작 다중 상속
Java 8에서는 인터페이스가 구현을 포함할 수 있으므로 클래스는 여러 인터페이스에서 동작(구현 코드)를 상속받을 수 있다.
#### **기능이 중복되지 않는 최소의 인터페이스**

스포츠카에 부스터 기능이 있는데, 그 기능은 기본 출력에 2배라고 가정해보자.

```java
public interface SportCar {
    void setOutput(int output);
    int getOutput();
    default void setBoosterOutput() {
        setOutput(getOutput() * 2);
    }
}
```

기존에 setOutput과 getOutput 메서드를 사용하여 setBoosterOutput이라는 기능을 제공할 수 있다.

SportCar 인터페이스를 구현하는 모든 클래스는 setOutput, getOutput 메서드를 구현해야 한다. 하지만 setBoosterOutput 메서드는 구현하지 않아도 된다.
#### **인터페이스 조합**

새로나온 청소기를 예시로 들면 진공 청소기 기능도 있을 뿐더러 침구 청소, 물걸레질까지 가능하다.

이런 모델링을 만들때 기존의 진공 청소기 기능을 그대로 사용을하고 침구 청소, 물걸레질 기능을 추가하면 이전 코드를 재사용하면서 간단히 구현할 수 있을 것이다.

```java
public interface VacuumCleaner { // 진공 청소기
    default void vacuumCleaning() {
        // 진공 청소 구현
    }
}

public interface BeddingCleaner { // 침구 청소기
    default void beddingCleaning() {
        // 침구 청소 구현
    }
}

public interface WaterMop { // 물 걸레
    default void mopping() {
        // 물 걸레 청소 구현
    }
}

public class SamsungCleaner implements VacuumCleaner, BeddingCleaner, WaterMop {
   
}
```

또한 물 걸레 청소 구현 코드를 더 효율적으로 리팩토링한다고 할때 디폴트 메서드를 수정하게 되면 해당 인터페이스를 상속받는 클래스들도 리팩토링될 수 있다.

---
## [메서드 구현 규칙](https://devbksheen.tistory.com/entry/%EB%94%94%ED%8F%B4%ED%8A%B8-%EB%A9%94%EC%84%9C%EB%93%9Cdefault-method%EB%9E%80#%EB%A9%94%EC%84%9C%EB%93%9C%20%EA%B5%AC%ED%98%84%20%EA%B7%9C%EC%B9%99-1)

실전에서 자주 일어나는 일은 아니지만 인터페이스는 다중 상속이 가능하기 때문에 같은 시그니처를 갖는 디폴트 메서드를 상속받는 상황이 생길 수 있다.

```java
public interface A {
    default void run() {
        System.out.println("A : run!");
    }
}

public interface B {
    default void run() {
        System.out.println("B : run!");
    }
}

public class C implements B, A {

    @Override
    public void run() {
        B.super.run();
    }
    
}
```

만약 상속받은 두 디폴트 메서드의 시그니처가 같다면 반드시 override하여 구현해야 한다.

아예 새로운 기능을 구현할 수도 있고 상속받은 인터페이스의 디폴트 메서드를 사용하도록 할 수 있다.

그런데 만약 같은 시그니처를 갖는 인터페이스들이 상속관계라면?

```java
public interface A {
    default void run() {
        System.out.println("A : run!");
    }
}

public interface B extends A {
    default void run() {
        System.out.println("B : run!");
    }
}

public class C implements B, A {
}

new C().run(); // B : run!
```

이 경우 서브 인터페이스(B)가 우선권을 갖는다.

그리고 클래스에서 정의한 메서드와 인터페이스 디폴트 메서드의 시그니처가 같은 상황도 있을 수 있다.

```java
public interface A {
    default void run() {
        System.out.println("A : run!");
    }
}

public interface B extends A {
    default void run() {
        System.out.println("B : run!");
    }
}

public class C implements A, B {
    public void run() {
        System.out.println("C : run!");
    }
}

new C().run(); // C : run!
```

이런 경우는 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖게된다.
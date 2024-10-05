## 디폴트 메서드
자바 8에선 기본 구현을 포함하는 인터페이스를 만드는 방법이 두가지 있다.
1. 인터페이스 내부에 정적 메서드(static method) 사용
2. 인터페이스의 기본 구현인 디폴트 메서드(default method) 사용
```Java
// 디폴트 메서드
default void sort(Comparator<? super E> c) {
    Collections.sort(this, c);
} 
```

디폴트 메서드는 주로 라이브러리를 만들 때 사용한다. 이미 구현된 메서드를 제공받기 때문에, 인터페이스를 구현할 때 추가적으로 해야할 일이 줄어든다.

### 13.1 변화하는 API
라이브러리에 인터페이스 A가 있고, 사용자가 구현한 클래스 B가 있다. A에 메서드를 추가하면서 라이브러리 내부의 A를 구현한 클래스는 수정하였지만, 사용자가 만든 B는 개발자 입장에서 어떻게 할 수 없다.

#### 13.1.1 API 버전 1
다음은 Resizable 인터페이스의 초기 버전이다.
```Java
public interface Resizeable extends drawable {
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int witdh, int height);
}
```
**사용자 구현**

사용자가 이를 구현한 Ellipse 클래스를 만들었고, 이것을 활용하는 프로그램을 작성하였다.
```Java
public class Ellipse implements Resizable{
    ...
}
```

#### 13.1.2 API 버전 2
이후에 Resizable 클래스에 `void setRelativeSize(int wFactor, int hFactor)`라는 메서드를 추가했다. 

사용자가 이 메서드를 구현하지 않더라도 호출을 하지 않는다면 프로그램은 실행된다. 이는 **바이너리 호환성**이 유지되기 때문이다. 

사용자가 setRelativeSize()를 사용할 경우엔 AbstractMethodError 예외가 발생한다. 

또한 구현되지 않았기 때문에 재빌드할 시 컴파일 에러도 발생한다.

이러한 문제를 디폴트 메서드를 통해 해결할 수 있다.

> **자바 프로그램의 호환성**
> ---
> 자바 프로그램의 변경에 관련된 호환성 문제는 바이너리 호환성, 소스 호환성, 동작 호환성 세가지로 나눌 수 있다. 
> 1. 바이너리 호환성 : 변경 이후에도 에러 없이 기존 바이너리가 실행될 수 있는 상황을 바이너리 호환성이라 한다. (바이너리 실행엔 인증, 준비, 해석등의 과정이 포함된다.)
> 2. 소스 호환성은 코드를 고치더라도 기존 프로그램을 성공적으로 재컴파일할 수 있음을 의미한다.
> 3. 동작 호환성은 코드를 바꾼 다음에도 같은 입력값이 주어지면 같은 출력값이 나온다.

### 13.2 디폴트 메서드란?
디폴트 메서드는 자신을 구현하는 클래스에서 구현하지 않아도 되는 새로운 메서드이다. 디폴트 메서드는 default라는 키워드로 시작한다.
```Java
public interface Sized {
    int size();
    default boolean isEmpty() {
        return size() == 0;
    }
}
```

자바 8에선 다양한 디폴트 메서드가 사용됐는데, Collection의 stream, List의 sort와 Predicate, Function, Comparator의 and, andThen 등이 있다.

> **추상클래스 vs 인터페이스**
> ---
> 둘 다 추상 메서드와 바디를 포함하는 메서드로 정의할 수 있다.
> 
> 추상 클래스는 하나만 상속 가능하며, 인터페이스는 여러개를 구현할 수 있다.

### 13.3 디폴트 메서드 활용 패턴
#### 13.3.1 선택형 메서드
메서드를 구현하는 클래스에서 빈 구현이 있는 경우를 해결할 수 있다. 예를 들어서 Iterator 클래스의 remove는 비워둔채 구현한 경우가 많았는데, 디폴트 메서드를 쓰면 이 문제를 해결할 수 있다.
```Java
interface Iterator<T> {
    boolean hasNext();
    T next();
    default void remove() {
        throw new UnsupportedOperationException();
    }
}
```
#### 13.3.2 동작 다중 상속
디폴트 메서드를 사용하면 동작 다중 상속 기능을 구현할 수 있다. 자바는 여러개의 인터페이스를 구현할 수 있다. 예를 들어 ArrayList는 List, RandomAccess, Cloneable, Serializable 등 여러개를 구현한다.

**기능이 중복되지 않는 최소의 인터페이스**

이를 위해 공통된 기능은 디폴트 메서드로 구현한 뒤, 다른 부분은 추상 메서드로 둬서 해결할 수 있다.
```Java
public interface Rotatable {
    void setRotationAngle(int angleDegrees);
    int getRotationAngle();
    default void rotateBy(int angleInDegrees) {
        setRotationAngle((getRotationAngle() + angleInDegrees) % 360);
    }
}
```
위 코드는 구현해야할 메서드에 따라 뼈대 알고리즘이 결정되는 템플릿 디자인 패턴과 비슷해보인다.

**인터페이스 조합**

위와 같이 여러개의 인터페이스를 만들고, 블럭 조립하듯이 필요한 기능들만 끼워서 필요한 클래스를 만들 수 있다.


> **옳지 못한 상속**
> ---
> 하나의 메서드를 위해 100개의 메서드를 가진 클래스를 상속 받는 것은 옳지 않다. 이럴 때는 멤버 변수를 통해 하나만 쓰는 것이 낫다. 그리고 가끔 final로 선언된 클래스가 있는데, 이는 상속을 금지하는 것이다. 예를 들어 String 클래스가 있다.

### 13.4 해석 규칙
자바는 여러개의 인터페이스를 구현할 수 있다. 그런데 같은 이름의 디폴트 메서드가 있을 경우, 어떻게 구분할까?

#### 13.4.1 알아야할 세가지 해결 규칙
1. **클래스** : 클래스나 슈퍼클래스에서 정의한 메서드가 제일 우선이다.
2. **서브 인터페이스** : 상속 관계가 있는 인터페이스라면, 가장 아래에 있는 인터페이스가 우선이다.
3. 여전히 우선순위가 결정되지 않았다면 여러 인터페이스가 상속 받은 클래스가 명시적으로 오버라이드하여 사용해야한다.
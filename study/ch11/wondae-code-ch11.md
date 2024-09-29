## null 대신 Optional 클래스
자바에서 가장 흔하게 발생하는 예외 중 하나인 NullPointerException.

프로그래밍에서 null이란 개념은 1965년, ALGOL이란 언어에서 처음 등장하였다.null의 목적은 `컴파일러의 확인 기능으로 모든 참조를 안전하게 사용할 수 있을 것`이다. 구현이 가장 쉬웠기 때문에 null을 만들었지만, 이로 인해 발생할 문제는 정말로 컸다.

### 11.1 값이 없는 상황 처리
```Java
public class Person {
    private Car car;
    public Car getCar() {
        return car;
    }
}

public class Car {
    private Insurance insurance;
    public Insurance getInsurance() {
        return insurance;
    }
}

public class Insurance {
    private String name;
    public String getName() {
        return name;
    }
}
```
위와 같은 클래스가 있을 때, `person.getCar().getInsurance();`를 호출할 때, car가 null이면 NullPointerException이 발생한다. 이를 줄이기 위해 여러가지 방법이 있다.
#### 11.1.1 보수적으로 NullPointerException 줄이기
하나하나씩 if문을 추가하여 줄인다.
```Java
if (person != null) {
    if(person.car != null) {
        ...
    }
}
```
이렇게 할 경우 확실히 체크할 순 있지만, 코드의 들여쓰기 수준이 증가한다. 이런 반복 패턴은 `깊은 의심`이라 한다.

혹은 깊은 들여쓰기를 줄이기 위해 아래처럼 작성할 수도 있다.
```Java
if (person == null) return;

if (person.car == null) return;
```
이렇게 작성할경우 들여쓰기는 줄지만, 메서드의 출구가 너무 많아져 유지보수가 힘들어진다.

### 11.1.2 null 때문에 발생할 수 있는 문제
* **에러의 근원** : NullPointerException은 가장 흔하게 발생하는 에러이다.
* **코드를 어지럽힘** : 중첩된 null 체크로 코드를 지저분하게 한다.
* **아무 의미가 없음** : 아무 의미도 표현하지 않기 때문에 정적 형식 언어에서 값이 없음을 표현하기에 적절치 않다.
* **자바 철학에 위배** : 자바는 개발자로부터 포인터를 숨겼으나, null 포인터는 예외이다.
* **형식 시스템에 구멍 발생** : null은 무형식이며 정보를 포함하고 있지 않기에, 모든 참조 형식에 할당 가능하다. 이렇게 계속될 경우, null이 어떤 의미로 사용되었는지 알기 어렴다.

### 11.1.3 다른 언어는 null 대신 무엇을 사용하나
* Groovy : `?.` (안전 네비게이션 연산자) 만약 접근하려는 참조가 null이면 null을 리턴한다.
* Haskell, Scala : Optional 값을 지원하는 Maybe 형식 제공 (값을 갖거나 없거나)
  
=> Java8부터 Optional\<T>를 통해 선택적 값 제공

### 11.2 Optional 클래스 소개
자바 8에선, haskell과 scala의 영향을 받아 `java.util.Optional<T>`클래스를 제공한다. Optional은 선택형 값을 캡슐화한다.

Optional 클래스는 값이 있을 경우에 값을 감싸고, 없는 경우 `Optional.Empty`를 리턴한다. 

`Optional.empty`는 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드이다. null과 `Optional.Empty()`는 비슷하지만 차이점이 많은데, null을 참조하려면 NullPointerException이 발생하지만, Optional.Empty()는 Optional 객체를 반환하기에 유연하게 사용가능하다.

```Java
public class Person {
    private Optional<Car> car;
    public Optional<Car> getCar() {
        return car;
    }
}

public class Car {
    private Optional<Insurance> insurance;
    public Optoinal<Insurance> getInsurance() {
        return insurance;
    }
}

public class Insurance {
    // 반드시 필요한 정보는 optional로 하지 않음
    private String name;
    public String getName() {
        return name;
    }
}
```

### 11.3 Optional 적용 패턴
#### 11.3.1 Optional 객체 만들기
* **빈 Optional** : Optional.empty() 사용
* **null이 아닌 값으로 Optional 만들기** : Optional.of() 사용
* **null 값으로 Optional 만들기** : Optional.ofNullable() 사용
  
#### 11.3.2 맵으로 Optional 값 추출 후 변환하기
```Java
String name = null;
if (insurance != null) {
    name = insurance.getName();
}
```
이런 패턴에 사용할 수 있도록 Optional은 map 메서드를 지원한다.

```Java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);

Optional<String> name = optInsurance.map(Insurance::getName);
```

#### 11.3.3 flatMap으로 Optional 객체 연결
```Java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = optPerson
                            .map(Person::getCar)
                            .map(Car::getInsurance)
                            .map(Insurance::getName);
```
위 코드는 컴파일되지 않는데, 이를 고치기 위해 flatMap을 사용할 수 있다.
```Java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = optPerson
                            .flatMap(Person::getCar)
                            .flatMap(Car::getInsurance)
                            .map(Insurance::getName);
```

> Optional 클래스는 Serializable 인터페이스를 지원하지 않는다. Optional의 설계자는 필드 형식으로 사용할 것을 가정하지 않았기 때문이다.

#### 11.3.5 디폴트 액션과 Optional 언랩
* `get()` : 간단히 값을 읽을 수 있지만, 값이 없을 경우 NoSuchElementException 예외를 발생시킨다.
* `orElse(T other)` : 값이 없을 경우 기본값을 제공할 수 있다.
* `orElseGet(Supplier<? extends T> other)` : orElse에 대응하는 lazy한 메서드. Optional이 없을 때만 Supplier 실행. Optional이 비어있을 때 기본값을 생성하고 싶다면 사용해야한다.
* `orElseThrow(Supplier<? extends X> exceptionSupplier)` : Optional이 비어있을 때 예외를 던질 수 있다.  `get()`과 달리 던질 예외를 선택할 수 있다.
* `ifPresent(Consumer<? super T> consumer)` : 값이 존재할 때 인수롤 넘겨준 동작을 실행할 수 있다
* `ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)` : Optional이 비었을 때 실행할 수 있는 Runnable을 인수로 받는다.

#### 11.3.7 필터로 특정값 거르기
Optional의 filter 메서드는 값이 있는 Optional에 대해서만 연산하므로 빈 Optional 객체를 쉽게 찾을 수 있다.

### 11.4 Optional을 사용한 실용 예제
#### 11.4.1 null일 수 있는 값을 Optional로 감싸기
```Java
// nullable 체크 X
Object value = map.get("key");

// null일 경우에도 대응 가능
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

#### 11.4.2 예외와 Optional 클래스
```Java
// 문자열을 정수로 변환, 만약 숫자로 이루어진 문자열이 아닐 시 NumberFormatException 예외 발생
int number = Integer.parseInt(str);

// 문자열이 숫자로 이루어진 문자열이 아닌 경우 처리를 쉽게 할 수 있음
public static Optional<Integer> stringToInt(String str) {
    try {
        return Optional.of(Integer.parseInt(str));
    } catch (NumberFormatException e) {
        return Optional.empty();
    }
}
```

#### 11.4.3 기본형 Optional
Optional에선 Int, Long, Double에 대응하는 OptionalInt, OptionalLong, OptionalDouble이 존재한다. 그러나, Optional의 최대 요소는 1개이므로 성능을 개선할 수 없으며, map, flatMap, filter 같은 메서드가 없기에 사용을 권장하지 않는다.
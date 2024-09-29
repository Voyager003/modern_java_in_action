### 억만 달러짜리 실수 null

NullPointer 라는 개념을 만든 토니 호어(Tony Hoare) 의 잘 알려진 [_“10억 달러 짜리 실수”_](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/) 라는 발언은 많은 시스템들이 null 로 인해 얼마나 많은 비용을 발생시키고 있으며 얼마나 많은 개발자들이 이를 안전하게 처리 하기 위해 노력하고 있는지 보여준다.

다음 코드에서 null로 생기는 문제가 무엇인지 알아보자.

Copy

```
public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
}
```

코드에서 Car를 소유하지 않은 사람(Person)이거나, 자동차는 소유하고 있지만 보험을 보유하고 있지 않다면? 이렇듯 여러 방면에서 `NullPointerException이 발생할 가능성`을 가지고 있다.
#### NullPointerException 줄이기
이처럼 예기치 않은 NullPointerException을 피하려면 어떻게 해야 할까? 대부분의 프로그래머는 필요한 곳에 다양한 null 확인 코드를 추가해서 null 예외 문제를 해결하여 할 것이다.

Copy

```
public String getCarInsuranceName(Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if (insurance != null) {
                return insurance.getName();
            }
        }
    }
    return "Unknown";
}
```

이와 같은 반복 패턴 코드를 '`깊은 의심`'이라고 부른다. 이를 반복하다 보면 `코드의 구조가 엉망이 되고 가독성도 떨어진다.` 때문에 다른 방법을 고려해봐야 한다.

Copy

```
public String getCarInsuranceName(Person person) {
    if (person != null) {
        return "Unknown";
    }

    Car car = person.getCar();
    if (car != null) {
        return "Unknown";
    }

    Insurance insurance = car.getInsurance();
    if (insurance != null) {
        return "Unknown";
    }

    return insurance.getName();
}
```

위 코드는 조금 다른 방법으로 `중첩 if 블록을 없앴다.` 그러나 null일 때 반환되는 기본값 'Unknown'이 세 곳에서 반복되고 있는데 같은 문자열을 반복하면서 오타 등의 실수가 생길 수 있다.
#### null 때문에 발생하는 문제
Java에서 null 레퍼런스를 사용하면서 발생할 수 있는 `이론적, 실용적 문제`를 확인해보자.

- 에러의 근원이다 : NullPointerException은 자바에서 가장 흔히 발생하는 에러다
- 코드를 어지럽힌다 : null 확인 코드를 추가해야 하므로 `과도한 체크 로직`으로 가독성이 떨어진다.
- 아무 의미가 없음 : 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다.
- 자바 철학에 위배된다
- 형식 시스템에 구멍을 만든다

### 그래서 등장한 Optional
Java 8에 추가된 Optional은 `선택형값을 캡슐화하는 클래스`로 'Optional< T >' 클래스를 사용해 NPE를 방지할 수 있도록 도와준다. 'Optional< T > '는 null이 올 수 있는 값을 감싸는 Wrapper 클래스로, 참조하더라도 NPE가 발생하지 않도록 도와준다.

메서드가 반환할 결과값이 ‘없음’을 명백하게 표현할 필요가 있고, `null`을 반환하면 에러를 유발할 가능성이 높은 상황에서 메서드의 **반환 타입**으로 `Optional`을 사용하자는 것이 `Optional`을 만든 주된 목적이다.

예를 들어 다음과 같은 레파지토리가 존재하고 회원의 이름을 얻기위해 데이터를 가져온다고 가정한다.

```java
public class MemberRepository {
  Member findByName(String name);
}
```

다음과 같은 코드에서 findByName()을 호출하는데, Repository(DB)에 **인자로 넘긴 이름(고길동)의 회원 객체가 없을 경우(NULL)**

`Member member1 = memberRepository.findByName("고길동");`

member1.getAge(); <<-- **NPE 발생(NullpointerException)**

NPE이 발생하게 된다.

이를 해결하기 위해 반환타입을 Optional로 적용하여 다음과 같이 사용한다.

```java
public class MemberRepository {

Optional <Member> findByName(String name);

}
```

이 때 주의할 점은 Optional은 데이터를 직렬화할 수 없기때문에 필드로 사용할수 없다는 것이다.

도메인 모델에 Optional을 사용했을 때 데이터를 직렬화할 수 없는 이유는 아키텍처의 문제이다.

Java 아키텍트인 브라이언 고츠는 Optional의 용도가 선택형 반환값을 지원하는 것이라고 명확하게 못박았다. Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않았으므로 Serializable 인터페이스를 구현하지 않기 때문에 도메인 모델에 Optional을 사용한다면 직렬화(serializable) 모델을 사용하는 도구나 프레임워크에서 문제가 생길 수 있다는 것이다.

```java
// Bad
public class Member {
    private Long id;
    private String name;
    private Optional<String> email = Optional.empty();
}

// Good
public class Member {
    private Long id;
    private String name;
    private String email;
}
```

만약 객체 그래프에서 일부 또는 전체 객체가 null일 수 있는 상황이라 Optional을 사용해서 도메인 모델을 구성해야하고 직렬화 모델이 필요하다면 다음 예제에서 보여주는 것처럼 Optional로 값을 반환받을 수 있는 메서드를 추가하는 방식을 권장한다.

```java
public class Person {
    private Car car;
    public Optional<Car> getCarAsOptional() {
        return Optional.ofNullable(car);
    }
}
```
#### Optional을 생성자나 메서드 인자로 사용 금지하기
`Optional`을 생성자나 메서드 인자로 사용하면, 호출할 때마다`Optional`을 생성해서 인자로 전달해줘야 한다.

하지만 호출되는 쪽, 즉 api나 라이브러리 메서드에서는 인자가 `Optional`이든 아니든 `null` 체크를 하는 것이 언제나 안전하다. 따라서 굳이 비싼 `Optional`을 인자로 사용하지 말고 호출되는 쪽에 `null` 체크 책임만 남겨두는 것이 권장된다.

```java
// Bad
public class HRManager {
    
    public void increaseSalary(Optional<Member> member) {
        member.ifPresent(member -> member.increaseSalary(10));
    }
}
hrManager.increaseSalary(Optional.ofNullable(member));

// Good
public class HRManager {
    
    public void increaseSalary(Member member) {
        if (member != null) {
            member.increaseSalary(10);
        }
    }
}
hrManager.increaseSalary(member);
```
### Kotlin의 Null 안정성
Kotlin은 기본적으로 null을 허용하지 않는다. 따라서 null은 중요한 메시지를 전달하는 데 사용되어야 하며, 다른 개발자가 보았을 때 의미 없는 경우에 null을 사용하지 말아야 한다. 클래스 생성 이후에 확실하게 값이 설정된다면 lateinit과 notnull 델리게이트를 사용할 수 있다.

만약 null을 반환할 가능성이 있다면 다음과 같이 타입 뒤에 ?(엘비스 연산자)를 붙여서 표현한다.

```java
fun length(): Int? = null
```
null인 경우를 위해 안전 호출(safe call)과 엘비스 연산자(Elvis operator)를 제공한다.

안전 호출은 값이 널이면 null을 반환하고, null이 아닐 경우에만 호출을 진행한다. 호출을 통해 반환된 값이 null인 경우, null이 아닌 기본값을 반환하기를 원하는 경우를 위해서는 elvis 연산자를 제공한다. 안전 호출 결과의 추론 타입도 nullable 타입이므로, 엘비스 연산자와 함께 사용하는 것이 권장된다.



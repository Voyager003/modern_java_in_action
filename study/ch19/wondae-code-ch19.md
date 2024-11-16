## 함수형 프로그래밍 기법
**함수형 프로그래밍** : 함수를 마치 일반적인 값처럼 전달할 수 있는 것 (일급 함수). 자바에선 메서드 참조를 활용하거나 람다식을 사용해서 직접 사용할 수 있다.
#### 19.1.1 고차원 함수
하나 이상의 함수를 인수로 받거나 리턴값으로 함수를 제공하는 것을 고차원 함수라 한다. 예를 들어 `Comparator.comparing` 같은 메서드가 있다.

스트림 연산으로 전달하는 메서드는 부작용이 없는 메서드여야 한다는 말이 있었다. 부작용이란 부정확한 결과가 나오거나, 레이스 컨디션 때문에 예상치 못한 결과가 나오는 경우를 말한다. 이 문제를 해결하기 위해선 부작용이 없게 잘 설계하거나, 부작용이 발생하는 경우를 잘 문서화 해야한다.

#### 19.1.2 커링
간단하게 말하면 함수를 리턴하는 함수로, 변수를 줄이면서 새로운 메서드를 만드는 데 사용된다.

### 19.2 영속 자료구조
함수형 프로그래밍에서 쓰이는 자료구조로서, 함수형 메서드 내에서 변경이 불가능하다. 이는 참조 투명성을 위해서이다.

#### 19.2.1 파괴적인 갱신과 함수형
```Java
class TrainJourney {
   public int price;
   public TrainJourney onward;
   public TrainJourney(int p, TrainJourney t) {
      price = p;
      onward = t;
   }
}
```
위와 같은 클래스가 있을 때, 아래와 같은 메서드를 통해 경로를 연결할 수 있다.

```Java
static TrainJourney link(TrainJourney a, TrainJourney b) {
   if (a == null) return b;
   TrainJourney t = a;
   while (t.onward != null) {
      t = t.onward;
   }
   t.onward = b;
   return a;
}
```
이렇게 진행 했을 시 처음 연결한 TrainJourney의 값이 변경되는 파괴적인 갱신이 일어난다. 이러한 문제를 해결하기 위해 다음과 같은 방법을 사용할 수 있다.

```Java
static TrainJourney append(TrainJourney a, TrainJourney b) {
   return a == null ? b : new TrainJourney(a.price, append(a.onward, b));
}
```

### 19.3 스트림과 게으른 평가
자바의 스트림은 한번 사용하면 재사용이 불가능하다. 따라서 재귀적으로 사용하기 어려운데, 이로 인해 다음과 같은 문제가 발생할 수 있다.

#### 19.3.1 자기 정의 스트림
```Java
public static Stream<Integer> primes(int n) {
   return Stream.iterate(2, i -> i + 1)
                .filter(MyMathUtils::isPrime)
                .limit(n);
}

public static boolean isPrime(int candidate) {
   int candidateRoot = (int) Math.sqrt((double) candidate);
   return IntStream.rangeClosed(2, candidateRoot)
                   .noneMatch(i -> candidate % i == 0);
}
```

다음과 같은 알고리즘을 스트림으로 변경해보자.

```Java
static IntStream numbers() {
   return IntStream.iterate(2, n -> n + 1);
}

static int head(IntStream numbers) {
   return numbers.findFirst().getAsInt();
}

static IntStream tail(IntStream numbers) {
   return numbers.skip(1);
}

static IntStream primes(IntStream numbers) {
   int head = head(numbers);
   return IntStream.concat{
      IntStream.of(head),
      primes(tail(numbers).filter(n -> n % head != 0))
   );
}
```
하지만 이렇게 구현한 경우 IllegalStateException이 발생한다. 이를 해결하기 위해선 concat의 두번째 인수 prime을 게으르게 평가하는 것으로 해결할 수 있다.

```Java
import java.util.function.Supplier;

class LazyList<T> implements MyList<T> {
   final T head;
   final Supplier<MyList<T>> tail;
   public LazyList(T head, Supplier<MyList<T>> tail) {
      this.head = head;
      this.tail = tail;
   }
   
   public T head() {
      return head;
   }
   
   public MyList<T> tail() {
      return tail.get(); // 위의 head와 달리 tail에서는 Supplier로 게으른 동작을 만들었다.
   }
   
   public boolean isEmpty() {
      return false;
   }

   public MyList<T> filter(Predicate<T> p) {
      return isEmpty() ?
           this :
           p.test(head()) ?
               new LazyList<>(head(), () -> tail().filter(p)) :tail().filter(p);
    }
}

LazyList<Integer> numbers = from(2);
int two = primes(numbers).head();
int three = primes(numbers).tail().head();
int five = primes(numbers).tail().tail().head();

System.out.println(two + " " + three + " " + five);
```

### 19.4 패턴 매칭
함수형 프로그래밍을 구분할 떄 또 사용되는 것이 패턴 매칭이다. if-else, switch 등을 사용하지 않고 사용할 수 있다.

#### 19.4.1 방문자 디자인 패턴
```Java
class BinOp extends Expr {
  ...
  public Expr accept(SimpiftExprVisitor v) {
    return v.visit(this);
  }
}

public class SimpiftExprVisitor {
   ...
   public Expr visit(BinOp e) {
        if("+".equal(e.opname) && e.right instanceof Number && ...) {
            return e.left;
        }
        return e;
    }
}
```

### 19.5 기타 정보
#### 19.5.1 캐싱 또는 기억화
* 기억화는 메서드에 캐시를 추가하는 기법으로, 래퍼가 호출되면 인수, 결과 쌍이 캐시에 존재하는지 먼저 확인. 
* 캐시에 값이 존재하면 해당 값을 즉시 반환하고, 존재하지 않으면 계산 후 결과를 반환하고 인수, 결과 쌍을 캐시에 저장.
* 순수 함수형 해결방식은 아니지만, 감싼 버전의 코드는 참조 투명성을 유지 가능
#### 19.5.2 같은 객체를 반환함
`참조 투명성`이란 `인수가 같으면 결과도 같다`라는 뜻이다. 
#### 19.5.3 콤비네이터
`콤비네이터`는 함수를 변수로 받아 함수를 리턴하는 것으로, 함수형 프로그래밍에서 많이 사용된다.
## 9. 리팩터링, 테스팅, 디버깅
> 이번 챕터부턴, 시연가능한 코드 위주로 작성한 후, 추가 내용을 더욱 딥하게 알아봅니다.

기존 코드를 리팩토링 하는 방법, 람다 표현식을 통한 다양한 디자인 패턴 간소화, 람다 표현식과 스트림 API를 디버깅하고 테스트 하는 방법에 대해 다룰 것이다.

### 9.1 가독성, 유연성 개선
#### 9.1.1 가독성 개선
코드 가독성 -> 작성한 코드를 다른 사람이 쉽게 이해하고 유지보수할 수 있게 하는 것
#### 9.1.2 익명 클래스를 람다 표현식으로 개선
```Java
// 기존 코드
Runnable r1 = new Runnable() {
    public void run() {
        System.out.println("Hello");
    }
};

// 람다를 활용한 신규 코드
Runnable r2 = () -> System.out.println("Hello");
```
익명 클래스에서 this와 super는 다른 의미를 가진다.

익명 클래스에서 this는 자기 자신, 람다 표현식에선 람다를 감싸는 클래스이다.
익명 클래스는 섀도우 변수를 통해 익명 클래스를 감싸는 클래스의 변수를 가릴수 있지만, 람다 표현식으론 불가능하다.
```Java
int a = 10;
Runnable r1 = () -> {
    int a = 3; // 컴파일 에러
    System.out.println(a);
}

Runnable r2 = new Runnable() {
    public void run() {
        int a = 3; // 잘 동작한다.
        System.out.println(a);
    }
}
```

#### 9.1.3 람다 표현식을 메서드 참조로 리팩터링 하기
메서드 참조를 통해 가독성을 더욱 높힐 수 있다.
```Java
// 람다 표현식을 사용한 코드
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = 
    menu.stream()
        .collect(
            GrouppingBy(dish -> {
                if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                else if (dish.getCalories() <= 700) return CaloricLevel.NORAML;
                else return CaloricLevel.FAT;
             })
        );
```
```Java
// 람다 표현식 내부를 추출하여 메서드 참조로 변경
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = 
    menu.stream().collect(GrouppingBy(Dish::getCaloricLevel));
```

#### 9.1.4 명령형 데이터 처리를 스트림으로 리팩터링 하기
```Java
// 전통적인 반복문
List<String> dishNames = new ArrayList<>();
for(Dish dish: menu) {
    if (Dish.getCalories() > 300) {
        dishNames.add(disg.getName());
    }
}

// 스트림 API를 이용
menu.parallelStream()
    .filter(d -> d.getCalories() > 300)
    .map(Dish::getName)
    .collect(toList());
```

#### 9.1.5 코드 유연성 개선
> 함수형 인터페이스 적용 : 조건부 연기실행과 실행 어라운드 패턴에 람다 표현식을 적용한다.

**조건부 연기 실행**
```Java
// 일반적인 조건문
if(logger.isLoggable(Log.FINER)) {
    logger.finer("Problem: " + generateDiagnostic());
}

// 적절한 logger가 없더라도 내용을 계속 확인함.
logger.log(Level.FINER, "Problem: " + generateDiagnostic());

// supplier를 통해 메세지 생성 과정을 defer함
public void log(Level level, Supplier<String> msgSupplier);

logger.log(Level.FINER, () -> "Problem: " + generateDiagnostic());
```

**실행 어라운드**
```Java
// 람다 표현식 전달로 사용
String oneLine = processFile((BufferedReader b) -> b.readLine());

String twoLines = processFile(
        (BufferedReader b) -> b.readLine() + b.readLine()
    );

// 구현체
public static String ProcessFile(BufferedReaderProcessor p) throws IOException {
    try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br);
    }
}

public interface BufferedReaderProcessor {
    String process(BufferedReader br) throws IOException;
}
```

### 9.2 람다로 객체지향 디자인 패턴 리팩터링 하기
디자인 패턴에 람다 표현식을 더하여 문제를 더욱쉽고 간단하게 해결한다.
#### 9.2.1 전략 패턴
전략 패턴은 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법이다.

전략 패턴은 세가지 구성요소로 이루어져 있다
* 전략 객체 : 알고리즘을 표현한 인터페이스
* 다양한 알고리즘을 나타내는 인터페이스 구현체
* 전략 객체를 사용하는 클라이언트

**람다 표현식을 사용한 구현 예시**
```Java
Validator numericValidator = new Validator((String s) -> s.matches("[a-z]+"));

boolean b1 = numericValidator.validate("aaaa")

Validator lowerCaseValidator = new Validator((String s) -> s.matches("\\d+"));

boolean b2 = lowerCaseValidator.validate("Bbbb");
```

#### 9.2.2 탬플릿 메서드 패턴
주어진 알고리즘은 조금 바꿔야 하는 유연성이 필요할 떄 사용한다. 공통부분은 정의하고, 바꿔야할 부분은 추상 메서드로 나눔으로서, 구현체가 알아서 구현하게 한다.

**람다 표현식을 사용한 구현 예시**
```Java
new OnlineBankingLambda().processCustomer(1337, (Customer c) -> System.out.println("Hello " + c.getName));
```

#### 9.2.3 옵저버 패턴
이벤트 발생시 한 객체가 다른 객체에게 신호 혹은 알림을 보내야하는 경우에 사용한다.

**람다 표현식을 사용한 구현 예시**
Observer 인터페이스의 notify 부분을 변경하였다.
```Java
f.registerObserver((String tweet) -> {
    if(tweet != null && tweet.contains("money")) {
        System.out.println("Breaking news in NY! " + tweet);
    }
})

f.registerObserver((String tweet) -> {
    if(tweet != null && tweet.contains("queen")) {
        System.out.println("Yet more news from London " + tweet);
    }
})
```

#### 9.2.4 의무 체인 패턴
작업 처리 객체의 체인을 만들 때 사용한다 (다음으로 작업할 객체와 작업 후 다음으로 처리할 객체 정보를 유지)

**람다 표현식을 사용한 구현 예시**
```Java
UnaryOperator<String> headerProcessing =
        (String text) -> "From Raul, Mario and Alan: " + text;

UnaryOperator<String> spellCheckerProcessing = 
        (String text) -> text.replaceAll("labda", "lambda");

Function<String, String> pipeline =
    headerProcessor.andThen(spellCheckProcessing);
```

#### 9.2.5 팩토리 패턴
인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용한다.

**람다 표현식을 사용한 구현 예시**
```Java
Supplier<Product> loanSupplier = Loan::new;
Loan loan = loanSupplier.get();

final static Map<String, Supplier<Product>> map = new HashMap();

static {
    map.put("loan", Loan::new);
    map.put("stock", Stock::new);
    map.put("bond", Bond::new);
}

public static Product createProduct(String name) {
    Supplier<Product> p = map.get(name);
    if(p != null) return p.get();
    throws new IllegalArgumentException("No such product " + name);
}
```

### 9.3 람다 테스팅
#### 9.3.1 보이는 람다 표현식의 동작 테스팅
람다는 익명 함수이기 때문에, 토스트 코드를 호출할 수 없다. 하지만 람다를 변수에 저장해놓으면 재사용 가능하다.
```Java
@Test
public void testComparingTwoPoints() throws Exception {
    Point p1 = new Point(10, 15);
    Point p2 = new Point(10, 20);
    int result = Point.comparebyXandThenY.compare(p1, p2);
    assertTrue(result < 0);
}
```

#### 9.3.2 람다를 사용하는 메서드의 동작에 집중하라
람다 표현식을 사용하는 메서드의 동작을 테스트함으로서 람다를 공개하지 않고 표현식을 검증할 수 있다
```Java
@Test
public void testMoveAllPointsRightBy() throws Exception {
    List<Point> points = Arrays.asList(
                            new Point(5, 5), new Point(10, 5));
    
    List<Point> expectedPoints = Arrays.asList(
                            new Point(15, 5), new Point(20, 5));

    List<Point> newPoints = Point.moveAllPointRightBy(points, 10);

    assertEquals(expectedPoints, newPoints);
}
```

#### 9.3.3 복잡한 람다를 개별 메서드로 분할하기
복잡한 람다식의 경우 메서드 참조를 이용해 여려개의 메서드로 분리 후 테스트를 진행하면 쉽다.

#### 9.3.4 고차원 함수 테스팅
함수를 인수로 받거나 다른 함수를 반환하는 메서드의 경우, 람다를 통해 테스트 가능하다.
```Java
@Test
public void testFilter() throws Exception{
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4);

    List<Integer> even = filter(numbers, i -> i % 2 == 0);
    List<Integer> smallerThenThree = filter(numbers, i -> i < 3);
    assertEquals(Arrays.asList(2, 4), even);
    assertEquals(Arrays.asList(1, 2), smallerThenThree);
}
```

### 9.4 디버깅
디버깅 시 가장 중요한 것은 스택 트레이스와 로깅이다. 하지만 람다 표현식은 기존 방법으론 어렵다.

#### 9.4.1 스택 트레이스
```
    at Debugging.lambda$main$0(Debugging.java:6)
        at Debugging.Lambda$55/284720968.apply(Unknown Source)
```
이렇게 람다 내부에서 문제가 발생하였을 때는 메서드 이름이 보이지 않는데, 메서드 참조는 이름이 보인다.

#### 9.4.2 로깅
스트림API에서 forEach 같은 연산을 통해 값을 출력할 경우, 스트림 전체가 소비되어 중간 연산의 결과를 확인하기 어렵다. 이러한 경우, peek 연산을 활용하면 실제로 소비하지 않았지만, 소비한 것 처럼 결과를 보여주기 떄문에, 결과 중간 동작을 표현할 수 있다.

----

### 디버깅 과정에서 구별 가능한 람다함수를 만드는 방법
1. 커스텀 메서드 래핑 : 의미 있는 이름을 가진 메소드를 정의하고 그 메소드가 람다를 반환하도록 하면, 그 메소드 이름이 스택 트레이스에 나타나 쉽게 식별할 수 있다.
2. 람다 내 예외 처리 : 람다의 컨텍스트를 포함한 커스텀 로깅을 추가 가능
    ```Java
    data.forEach(element -> {
        try {
            // 여기에 람다 로직 작성
            System.out.println(element);
        } catch (Exception e) {
            // 커스텀 컨텍스트로 예외 로깅
            System.err.println("람다에서 예외 발생: " + element);
            e.printStackTrace();
        }
    });
    ```
3. 이름이 있는 함수형 인터페이스와 함께 사용 : 스택 트레이스에 인터페이스의 이름이 나타나서 이를 더 쉽게 식별할 수 있다.
    ```Java
    @FunctionalInterface
    interface ElementProcessor {
        void process(String element);
    }

    public void processData() {
        List<String> data = Arrays.asList("a", "b", "c");
        ElementProcessor processor = this::processElement;
        data.forEach(processor::process);
    }

    private void processElement(String element) {
        // 여기에 람다 로직 작성
        System.out.println(element);
    }
    ```
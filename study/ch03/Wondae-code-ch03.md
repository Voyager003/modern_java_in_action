## 람다 표현식
익명 클래스도 좋긴 하지만, 코드가 그렇게 깔끔하진 않았다.
-> 그 이후 Java8에 도입된 람다 표현식 : 이름이 없는 함수며, 메서드를 인수로 전달 가능!

### 람다란 무엇인가?
메서드로 전달 가능한 익명함수를 단순화 한 것 
이름은 없지만, 파라미터 리스트, 바디, 반환 형식, 예외리스트는 가지고 있다.
#### 주요 특징
* 익명 : 이름이 없다.
* 함수 : 특정 클래스에 종속되지 않으므로 메서드가 아닌 함수라 부른다.
* 전달 : 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
* 간결성 : 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다.
```Java
Comparator<Apple> byWeight = new Comparator<Apple>{
    @Override
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
}
```

```Java
Comparator<Apple> byWeight = 
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

#### 람다 표현식의 구성 요소
* 파라미터 리스트 : Comparator의 compare 메서드 파라미터
* 화살표 : 파라미터 리스트와 바디를 구분
* 람다 바디 : 람다의 반환값에 해당하는 표현식
```Java
// 반환 시 표현식을 쓰려면 중괄호를 생략한다.
// 혹은 중괄호와 함께 명시적으로 return을 붙인다.
(String s) -> s.length();
(Apple a) -> {return a.getWeight() > 50;}

// 여러 줄 사용 가능
(int x, int y) -> {
    System.out.println(x);
    System.out.println(y);
}
```

### 어디에, 어떻게 람다를 사용할까?
#### 1. 함수형 인터페이스
하나의 추상 메서드를 가지는 인터페이스인 함수형 인터페이스에서 사용 가능하다. (상속 X)
자바 API에선, Comparator, Runnable이 있다.

이를 통하여 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다.
```Java
// 람다 사용
Runnable r1 = () -> System.out.println("Hello World 1");

// 익명 클래스 사용
Runnable r2 = new Runnable() {
    public void run(){
        System.our.println("Hello world 2");
    }
}
// inline 방식으로 제공
process(() -> System.out.println("Hello world 3"));
```

#### 2. 함수 디스크립터
함수 디스크립터는 람다 표현식의 시그니처를 서술하는 메서드에 사용 가능하다.
시그니처 : 인수와 반환값을 설명하는 것
```Java
// 함수 디스크립터 예씨
(Apple, Apple) -> int
```

#### 3. 람사 활용 : 실행 어라운드 패턴
실행 어라운드 패턴 : 초기화 -> 작업 -> 정리와 같이 앞 뒤는 동일한데 가운데가 바뀌는 형식의 코드
```Java
public String processFile() throws IOException {
    // Java 7에 추가된 try with resource
    try (BufferedReader br = 
            new BufferedReader(new FileReader("data.txt")){
        return br.readLine();
    }
}
```
##### 동작 파라미터화 및 람다 적용
```Java
String result = processFile((BufferedReader br) -> br.readLine());
```

##### 함수형 인터페이스를 통해 동작 전달
함수형 인터페이스에 람다 적용 가능하다.
```Java
@FuntionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
```

##### 동작 실행
```
public String processFile(BufferedReaderProcessor p) throws
        IOException{
    try (BufferedReader br = 
            new BufferedReader(new FileReader("data.txt"))){
        return p.process(br);
    }
}
```

##### 람다 전달
```
String oneLine = processFile((BufferedReader br) -> br.readLine());

String twoLines =
    processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

#### 4. 함수형 인터페이스 적용
함수형 인터페이스는 단 하나의 추상 메서드만 가진다. 그리고 그 추상메서드는 람다 표현식의 시그니처를 묘사하며, 이를 함수 디스크립터라고 한다. 
이를 위해 자바 API는 Comprable, Runnable, Callable 등 함수형 인터페이스를 포함한다.

이외에 Predicate, Consumer, Function 인터페이스를 소개한다.
##### Predicate
java.util.function.Predicate\<T>는 test라는 추상메서드를 정의하며, test는 제네릭 형식의 T의 객체를 인수로 받아 불리언을 반환한다. 이를 사용하여 boolean 값이 필요할 상황에서 바로 사용할 수 있다.
```Java
@FunctionalInterface
public interface Predicate<T>{
    boolean test(T t);
}

public <T> List<T> filter(List<T> list, Prediacate<T> p){
    List<T> results = new ArrayList<>();
    for(T t: list){
        if(p.test(t)){
            results.add(t);
        }
    }
    return results;
}

Predicate<String> nonEmptyString = (String s) -> !s.isEmpty();

List<String> nonEmpty = filter(listOfStrings, nonEmptyString);
``` 

##### Consumer
java.util.funciton.Funtion\<T> 인터페이스는 제네릭 형식 T 객체를 받아서 void를 반환하는 accept라는 추상 메서드를 정의. T 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 사용.
```
@FunctionalInterface
public interface Consumer<T>{
    void accept(T t);
}

public <T> void forEach(List<T> list, Consumer<T> c){
    for(T t : list){
        c.accept(t);
    }
}

forEach(
    Arrays.asList(1,2,3,4,5),
    (Integer i) -> System.out.println(i);
);
```

##### Function
java.util.funciton.Function<T, R> 인터페이스는 제네릭 형식 T를 인수로 받아 제네릭 형식 R 객체를 반환하는 추상 메서드 apply를 정의한다. 입력을 출력으로 매핑하는 람다를 정의할 때 사용할 수 있다.

```
@FunctionalInterface
public interface Function<T, R>{
    R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> result = new ArrayList<>();
    for(T t: list){
        result.add(f.apply(t));
    }
    return result;
}

// [7, 2, 6]
List<Integer> l = map(
    Arrays.asList("lambdas", "in", "action");
    (String s) -> s.length()
);
```

**기본형 특화**
자바의 기본형 (int, long, char ...)를 제네릭으로 사용하기 위해 박싱과 언박싱, 그리고 이 동작을 자동으로 해주는 오토박싱도 지원한다. 하지만 오토박싱은 자원 소모가 더 크기 때문에, 오토박싱 없이 기본형을 지원하는 함수형 인터페이스가 있다.
* IntPredicate
* LongPredicate
* ...

**Java 8의 함수형 인터페이스**
* Predicate\<T> : T -> boolean
* Consumer\<T> : T -> void
* Function\<T, R> : T -> R
* Supplier\<T> : () -> T
* UnaryOperator\<T> : T -> T
* BinaryOperator\<T> : (T, T) -> T
* BiPrecate<L, R> : (L, R) -> boolean
* BiConsumer<T, U> : (T, U) -> void
* BiFunction<T, U, R> : <T, U> -> R

#### 5. 형식 검사, 형식 추론, 제약
람다를 더 제대로 이해하기 위해 실제 형식을 파악해야한다.
##### 형식 검사
람다가 사용되는 컨텍스트를 사용하여 람다의 형식을 추론할수 있다. 이 때 추론된 형식을 대상 형식이라 한다.

**형식 검사 과정**
```
List<Apple> heavierThan150g = 
filter(inventory, (Apple apple) -> apple.getWeight() > 150);
```

1. filter 메서드의 선언 확인
2. filter 메서드는 두번째 파라미터로 Predicate\<Apple> 형식 기대
3. Predicate\<Apple>은 test라는 추상메서드를 정의하는 함수형 인터페이스
4. test 메서드는 Apple을 받아 boolean 반환하는 함수 디스크립터 묘사
5. filter 메서드로 전달된 인수는 위 요구사항을 만족해야 함

##### 같은 람다, 다른 함수형 인터페이스
대상 형식 때문에, 같은 람다 표현식이라도 다른 추상 메서드 들에서 사용 가능
```Java
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

##### 형식 추론
자바 컴파일러는 대상 형식을 통해 함수 디스크립터를 알 수 있다. 그렇기에 컴파일러는 람다의 시그니처도 파악 가능하다.
```Java
Comparator<Apple> c = 
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

Comparator<Apple> c = 
    (a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

##### 지역 변수 사용
람다표형식은 익명 함수처럼 자유변수 (외부에서 정의된 변수)를 사용할 수 있다.
이와 같은 동작을 람다 캡처링이라 한다.
```Java
int portNumber = 1337;
Runable r = () -> System.out.println(portNumber);
```
이렇게 자유변수를 사용하기 위해선 final로 선언되거나, final 처럼 사용되어야 한다.

#### 6. 메서드 참조
메서드 참조를 사용하여 메서드를 재활용해 람다처럼 전달할 수 있다.
```Java
inventory.sort((Apple a1, Apple a2) ->
        a1.getWeight().compareTo(a2.getWeight()));

invenroty.sort(comparing(Apple::getWeight));
```
##### 요약
메서드 참조는 명시적으로 메서드 명을 알려줌으로서, 가독성을 높일 수 있다.
메서드 참조를 만드는 방식은 다음과 같다.
1. 정적 메서드 참조 : Integer::parseInt
2. 인스턴스 메서드 참조 : String::length
3. 기존 객체의 인스턴스 메서드 참조 : expensiveTransaction::getValue

```Java
// List 속 문자열을 대소문자 구분 없이 정렬하는 프로그램
List<String> str = Arrays.asList("a", "b", "A", "B");
str.sort((s1, s2) -> s1.compareToIgnoreCase(s2));

// 메서드 참조
str.sort(String::compareToIgnoreCase);
```

##### 생성자 참조
클래스 이름과 new 키워드를 이용하여 기존 생성자의 참조를 만들 수 있다.
```Java
// 메서드 참조
Supplier<Apple> c1 = Apple::new;
// 기존 람다
Supplier<Apple> c1 = () -> new Apple();
Apple a1 = c1.get();
```

매개 변수를 포함하기 위해선, Function 혹은 BiFunction 인터페이스를 사용하면 된다.

#### 7. 람다, 메서드 참조 활용하기
##### 코드 전달
```Java
public class AppleComparator implements Comparator<Apple>{
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
}
inventory.sort(new AppleComparator());
```

##### 익명 클래스 사용
```Java
inventory.sort(new Comparator<Apple>(){
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
});
```

##### 람다 표현식 사용
```Java
inventory.sort((Apple a1, Apple a2) ->
        a1.getWeight().compareTo(a2.getWeight()));

inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));

Comparator<Apple> c = 
        Comparator.comparing((Apple a) -> a.getWeight());

inventory.sort(comparing(apple -> apple.getWeight()));
```

##### 메서드 참조 사용
```Java
inventory.sort(comparing(Apple::getWeight));
```

#### 8. 람다 표현식 조합
##### Comparator 조합
**역정렬**
```Java
inventory.sort(comparing(Apple::getWeight).reversed());
```

**comparator 추가**
```Java
inventory.sort(comparing(Apple::getWeight)
        .reversed()
        .thenComparing(Apple::getCountry));
```

##### Predicator 조합
```Java
Predicator<Apple> notRedApple = redApple.negate();

Predicator<Apple> redAndHeavyApple = redApple
                        .and(apple -> apple.getWeight() > 150);
```

##### Function 조합
```Java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.andThen(g);

int result = h(4) // 10

Function<Integer, Integer> h2 = f.compose(g);
int result2 = h2(4) // 9
```


### 추가 조사
## JVM에서 람다를 다루는 방식

람다 표현식이 자바 코드와 JVM 내부에서 어떻게 동작하는가? 람다는 확실히 어떤 자료형의 값이다. 자바는 기본형과 참조형 두가지 타입만이 있는데, 일단 람다는 기본형이 아니다. 그렇기에 람다 표현식은 참조형을 반환하는 표현식이다.

예시를 보자.
```Java
public class LambdaExample {
    private static final String HELLO = "Hello World!";

    public static void main(String[] args) throws Exception {
        Runnable r = () -> System.out.println(HELLO);
        Thread t = new Thread(r);
        t.start();
        t.join();
    }
}
```
내부 클래스에 익숙한 개발자들은 람다가 그냥 Runnable의 익명 구현을 위한 synthetic sugar라 추측할 지도 모른다. 그러나 예시 클래스를 컴파일하면 `LambdaExample.class` 하나만 나온다. 내부 클래스를 위한 추가 파일은 없다.

이 말은 람다가 내부 클래스가 아니라는 것을 의미한다. 그렇기에, 다른 메카니즘이 있을 것이다. 사실은, 바이트코드를 `javap -c -p`로 디컴파일하면 두가지가 드러난다. 첫번째 사실은 람다 바디가 메인 클래스의 private static method로 있다는 것이다.
```Java
private static void lambda$main$0();
    Code:
       0: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #9                  // String Hello World!
       5: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return
```
아마도 당신은 private body method의 시그니처가 람다의 것과 맞다고 생각했을 것이고, 실제로 그러하다. 아래와 같은 람다는 다음과 같은, 문자열을 받고, 정수를 반환하는 body method를 만들것이다.
```Java
public class StringFunction {
    public static final Function<String, Integer> fn = s -> s.length();
}
```
```Java
private static java.lang.Integer lambda$static$0(java.lang.String);
    Code:
       0: aload_0
       1: invokevirtual #2                  // Method java/lang/String.length:()I
       4: invokestatic  #3                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       7: areturn
```
 
 바이트코드에 관해 알아야 할 두번째는 메인 메서드의 형태이다.
 ```Java
public static void main(java.lang.String[]) throws java.lang.Exception;
    Code:
       0: invokedynamic #2,  0              // InvokeDynamic #0:run:()Ljava/lang/Runnable;
       5: astore_1
       6: new           #3                  // class java/lang/Thread
       9: dup
      10: aload_1
      11: invokespecial #4                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
      14: astore_2
      15: aload_2
      16: invokevirtual #5                  // Method java/lang/Thread.start:()V
      19: aload_2
      20: invokevirtual #6                  // Method java/lang/Thread.join:()V
      23: return
```

bytecode가 invokedynamic의 호출로 시작한다. 이 opcode는 자바 버전 7에 추가됐다 (그리고 이건 JVM 바이트 코드에 추가된 유일한 opcode이다). 이것에 관해선 “[Real-world bytecode Handling with ASM](https://blogs.oracle.com/javamagazine/real-world-bytecode-handling-with-asm)”과 “[Understanding Java method invocation with invokedynamic](https://www.oracle.com/a/ocom/docs/corporate/java-magazine-nov-dec-2017.pdf#page=67)”에서 다뤘으므로, 나중에 같이 읽어볼 수 있을 것이다.

이 코드에서 invokedynamic 호출을 이해하는 가장 직접적인 방법은 일반적이지 않은 형태의 팩토리 메서드의 호출이라 생각하는 것이다. 이 메서드 호출은 Runnable 구현체의 인스턴스를 반환한다. 정확한 자료형은 바이트 코드에 표시되어 있지 않으며, 근본적으로 중요하지 않다.

실제 자료형은 컴파일 타임에 없으며, 런타임에 필요에 의해 만들어질 것이다. 더 나은 설명으로는, 이것을 가능하게 하는 세가지 메카니즘인 call sites, method handles, 그리고 bootstrapping에 논해보자.

### Call Sites
call site는 바이트코드에서 함수 호출 명령어가 발생하는 장소로 알려져있다.

자바 바이트 코드는 전통적으로 메서드 호출의 다양한 경우를 처리하는 4가지 opcode인 정적 메서드, 일반 호출 (메서드 오버라이딩과 연관된 가상 호출), 인터페이스 확인, 특별 호출 (상위 클래스 호출과 private 메서드 같이 오버라이드가 필요하지 않은 경우)

동적 호출은 여기에 더해, 메서드가 실제로 호출되는 결정을 프로그래머가 Call site 단위로 할 수 있는 메커니즘을 제공한다.

여기서, invokedynamic call site는 자바 힙 영역의 CallSIte 객체로 표현된다. 이 것은 특이한 것이 아니다. 자바는 비슷한 것들을 Method, Class와 같은 자료형을 통해 Reflection API로 Java 1.1부터 지원했다. Java 런타임에는 많은 동적 동작이 있기에, java가 call site와 다른 런타임 정보를 모델링하고 있는 것은 그리 놀라운 일이 아니다.

invoke dynamic 명령어가 실행되면, JVM은 해당 call site 객체를 찾거나 생성한다 (처음일 경우). call site 객체에는 메서드 핸들이 포함되어 있는데, 이것은 실제로 호출하려는 메서드를 나타내는 객체이다.

call site 객체는 필요한 간접 레벨을 제공하며, 연관된 호출 대상 (메서드 핸들)이 시간에 따라 변경될 수 있도록 한다.

CallSite(추상 클래스)의 세 가지 하위 클래스는 ConstantCallSite, MutableCallSite, VolatileCallSite이다. 기본 클래스 CallSite는 패키지 전용 생성자만 가지고 있고, 세 하위 클래스가 public 생성자를 가지고 있다. 이는 사용자 코드에서 CallSite를 직접 하위 클래스화할 수 없지만, 하위 클래스를 하위 클래스화할 수 있다는 것을 의미한다. 예를 들어, JRuby 언어는 invokedynamic을 구현의 일부로 사용하며 MutableCallSite를 하위 클래스화한다.

참고로, 일부 invokedynamic 호출 사이트는 실제로 lazy하게 계산되며, 처음 실행된 이후에는 대상 메서드가 변경되지 않는 경우가 많다. 이는 ConstantCallSite에 매우 일반적인 사용 사례이며, 람다 표현식도 이에 포함된다.

이는 상수가 아닌 call site가 프로그램 수명 동안 많은 다른 메서드 핸들을 대상으로 가질 수 있다는 것을 의미한다.

### Method  handles
리플렉션은 런타임 기술을 수행하는 데 강력한 기법이지만, 여러 가지 설계 결함이 있으며, 현재 확실히 낡아가고 있다. 리플렉션의 주요 문제 중 하나는 성능이며, 특히 리플렉티브 호출이 just-in-time(JIT) 컴파일러에 의해 인라인되기 어렵다.

이는 좋지 않다. 왜냐하면 인라이닝은 JIT 컴파일에 매우 중요한데, 그 이유 중 하나는 일반적으로 첫 번째로 적용되는 최적화이며 다른 기술(예: 탈출 분석 및 dead code 제거)을 위한 문을 연다.

두 번째 문제는 리플렉티브 호출이 Method.invoke() 호출 지점에서 매번 링크된다는 것이다. 예를 들어, 보안 액세스 검사가 수행됩니다. 이는 매우 낭비적인데, 왜냐하면 검사가 일반적으로 첫 번째 호출에서 성공하거나 실패하며, 성공하면 프로그램 수명 동안 계속 그렇기 때문이다. 그러나 리플렉션은 이 링크를 반복적으로 수행한다. 따라서 리플렉션은 재링크와 CPU 시간 낭비로 인해 많은 불필요한 비용을 발생시킨다.

이러한 문제(및 기타 문제)를 해결하기 위해 Java 7에서는 java.lang.invoke라는 새로운 API를 도입했는데, 이는 주요 클래스의 이름 때문에 method handles로 흔히 불린다.

method handle(MH)은 Java의 타입-안전 함수 기능-포인터다. 코드에서 호출하고자 하는 메서드를 참조하는 방법으로, Java 리플렉션의 Method 객체와 유사하다. MH에는 기반 메서드를 실제로 실행하는 invoke() 메서드가 있다.

한 수준에서 MH는 메탈(?)에 더 가까운 더 효율적인 리플렉션 메커니즘에 불과합니다. Reflection API의 객체로 표현되는 것은 모두 동등한 MH로 변환할 수 있다. 예를 들어, 반사 Method 객체는 Lookup.unreflect()를 사용하여 MH로 변환할 수 있다. 생성된 MH는 일반적으로 기본 메서드에 더 효율적으로 액세스할 수 있다.

MH는 MethodHandles 클래스의 헬퍼 메서드를 통해 조합 및 부분 인수 바인딩(커링)과 같은 다양한 방식으로 적응될 수 있다.

일반적으로 메서드 링킹에는 정확한 유형 설명자 일치가 필요하다. 그러나 MH의 invoke() 메서드에는 메서드 시그니처에 관계없이 링킹을 허용하는 특별한 다형성 시그니처가 있다.

런타임에 invoke() 호출 지점의 시그니처는 직접 참조된 메서드를 호출하는 것처럼 보여야 하며, 이를 통해 반사 호출에 일반적인 유형 변환 및 자동 박싱 비용을 피할 수 있다.

Java는 정적으로 유형화된 언어이므로, 이와 같은 근본적으로 동적인 메커니즘을 사용할 때 얼마나 많은 유형 안전성을 유지할 수 있는지가 문제가 된다. MH API는 MethodType이라는 유형을 사용하여 이 문제를 해결한다. MethodType은 메서드가 취하는 인수의 불변 표현, 즉 메서드 시그니처다.

MH의 내부 구현은 Java 8 생애 동안 변경됐다. 새로운 구현은 lambda forms라고 불리며, MH가 많은 사용 사례에서 반사보다 더 나은 성능을 제공하는 극적인 성능 향상을 제공했었다.

### Bootstrapping
각 `invokedynamic` call site가 바이트코드 명령 스트림에서 처음 만날 때, JVM은 어떤 메서드를 대상으로 하는지 알지 못한다. 사실, 해당 명령어와 연관된 호출 사이트 객체가 없다.

call site는 부트스트래핑되어야 하며, JVM은 부트스트랩 메서드(BSM)를 실행하여 호출 사이트 객체를 생성하고 반환한다.

각 `invokedynamic` call site에는 클래스 파일의 별도 영역에 저장된 BSM이 연결되어 있다. 이러한 메서드를 통해 사용자 코드는 런타임에 동적으로 링크를 결정할 수 있다.

`Runnable`의 원본 예제와 같은 `invokedynamic` 호출을 역컴파일하면 다음과 같은 형태를 볼 수 있다.
```Java
0: invokedynamic #2,  0
```
그리고 클래스 파일의 상수 풀에서, 항목 #2가 `CONSTANT_InvokeDynamic` 유형의 상수임을 알 수 있다. 관련 상수 풀 항목은 다음과 같다:
```Java
#2 = InvokeDynamic      #0:#31
   ...
  #31 = NameAndType        #46:#47        // run:()Ljava/lang/Runnable;
  #46 = Utf8               run
  #47 = Utf8               ()Ljava/lang/Runnable;
```
상수 풀에 0이 있다는 것은 단서이다. 상수 풀 항목은 1부터 번호가 매겨지므로, 0은 실제 BSM이 클래스 파일의 다른 부분에 있음을 알려준다.

람다의 경우, `NameAndType` 항목은 특별한 형태를 취한다. 이름은 임의이지만, 유형 시그니처에는 유용한 정보가 포함되어 있다.

반환 유형은 `invokedynamic` 팩토리의 반환 유형에 해당하며, 이는 람다 표현식의 대상 유형이다. 또한 인수 목록은 람다에 의해 캡처되는 요소의 유형으로 구성된다. 무상태 람다의 경우 반환 유형은 항상 비어 있다. 자바 클로저만 인수가 있다.

BSM은 최소 3개의 인수를 받고 `CallSite`를 반환한다. 표준 인수는 다음 유형이다:

* `MethodHandles.Lookup`: call site가 발생하는 클래스에 대한 조회 객체 
* `String`: `NameAndType`에 언급된 이름 
* `MethodType`: `NameAndType`의 확인된 유형 설명자 이러한 인수 다음에는 BSM에 필요한 추가 정적 인수가 올 수 있다.

BSM의 일반적인 경우는 매우 유연한 메커니즘을 허용하며, 비-자바 언어 구현자들이 이를 사용한다. 그러나 자바 언어는 임의의 `invokedynamic` 호출 사이트를 생성할 수 있는 언어 수준의 구문을 제공하지 않는다.

람다 표현식의 경우, BSM은 특별한 형태를 취하며, 이 메커니즘이 어떻게 작동하는지 이해하려면 더 자세히 살펴볼 필요가 있다.

### 람다의 bootstrap 메서드 해석
javap 명령에 -v 인수를 사용하면 부트스트랩 메서드를 볼 수 있다. 이는 부트스트랩 메서드가 클래스 파일의 특별한 부분에 있고 메인 상수 풀로 다시 참조하기 때문에 필요하다. 이 간단한 Runnable 예제에서는 해당 섹션에 단일 부트스트랩 메서드가 있다:
```Java
BootstrapMethods:
  0: #28 REF_invokeStatic java/lang/invoke/LambdaMetafactory.metafactory:
        (Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;
         Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;
         Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #29 ()V
      #30 REF_invokeStatic LambdaExample.lambda$main$0:()V
      #29 ()V
```
이것은 조금 읽기 어려우므로 이를 해독해 보자.

이 호출 사이트의 부트스트랩 메서드는 상수 풀의 #28 엔트리이다. 이것은 Java 7에 표준에 추가된 MethodHandle 유형의 엔트리이다. 이제 문자열 함수 예제의 경우와 비교해 보자.
```Java
0: #27 REF_invokeStatic java/lang/invoke/LambdaMetafactory.metafactory:
        (Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;
         Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;
         Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #28 (Ljava/lang/Object;)Ljava/lang/Object;
      #29 REF_invokeStatic StringFunction.lambda$static$0:(Ljava/lang/String;)Ljava/lang/Integer;
      #30 (Ljava/lang/String;)Ljava/lang/Integer;
```
BSM으로 사용될 메서드 핸들은 동일한 정적 메서드 LambdaMetafactory.metafactory( ... )이다.

변경된 부분은 메서드 인수이다. 이는 람다 표현식의 추가 정적 인수이며 세 개이다. 이들은 람다의 시그니처와 람다 본문의 실제 최종 호출 대상인 메서드 핸들을 나타낸다. 세 번째 정적 인수는 시그니처의 지워진 형태이다.

이제 java.lang.invoke 코드로 들어가서 플랫폼이 어떻게 메타팩토리를 사용하여 람다 표현식의 대상 유형을 구현하는 클래스를 동적으로 생성하는지 살펴보자.
### 람다 메타팩토리
BSM은 이 정적 메서드를 호출하며, 이는 결국 콜 사이트 객체를 반환한다. invokedynamic 명령어가 실행될 때, 콜 사이트에 포함된 메서드 핸들은 람다의 타겟 타입을 구현하는 클래스의 인스턴스를 반환할 것이다.
Metafactory 메서드의 소스 코드는 비교적 간단하다:
```Java
public static CallSite metafactory(MethodHandles.Lookup caller,
                                       String invokedName,
                                       MethodType invokedType,
                                       MethodType samMethodType,
                                       MethodHandle implMethod,
                                       MethodType instantiatedMethodType)
            throws LambdaConversionException {
        AbstractValidatingLambdaMetafactory mf;
        mf = new InnerClassLambdaMetafactory(caller, invokedType,
                                             invokedName, samMethodType,
                                             implMethod, instantiatedMethodType,
                                             false, EMPTY_CLASS_ARRAY, EMPTY_MT_ARRAY);
        mf.validateMetafactoryArgs();
        return mf.buildCallSite();
}
```
Lookup 객체는 invokedynamic 명령어가 존재하는 컨텍스트에 해당한다. 이 경우 람다가 정의된 동일한 클래스이므로, Lookup 컨텍스트에는 람다 본문이 컴파일된 private 메서드에 액세스할 수 있는 적절한 권한이 있다.

호출된 이름과 타입은 VM에서 제공되며 구현 세부 사항이다. 마지막 세 개의 매개변수는 BSM에서 제공된 추가 정적 인수이다.

현재 구현에서, 메타팩토리는 내부적으로 차단된 ASM 바이트코드 라이브러리의 복사본을 사용하여 타겟 타입을 구현하는 내부 클래스를 생성한다.

람다가 Enclosing Scope의 매개변수를 캡처하지 않는 경우, 결과 객체는 Stateless이므로 구현은 단일 인스턴스를 미리 계산하여 최적화한다. 이는 람다의 구현 클래스를 싱글톤으로 만든다.
```
jshell> Function<String, Integer> makeFn() {
   ...>   return s -> s.length();
   ...> }
|  created method makeFn()

jshell> var f1 = makeFn();
f1 ==> $Lambda$27/0x0000000800b8f440@533ddba

jshell> var f2 = makeFn();
f2 ==> $Lambda$27/0x0000000800b8f440@533ddba

jshell> var f3 = makeFn();
f3 ==> $Lambda$27/0x0000000800b8f440@533ddba
```
이것이 문서가 Java 프로그래머들에게 람다에 대한 어떠한 형태의 의미론에 의존하지 말 것을 강력히 권장하는 이유 중 하나이다.

### 결론
이 글은 JVM이 람다 표현식에 대한 지원을 정확히 어떻게 구현하는지에 대한 세부적인 내용을 탐구했다. 이는 언어 구현자 영역에 깊이 들어가는 것이기 때문에 가장 복잡한 플랫폼 기능 중 하나이다.

이 과정에서 저는 invokedynamic과 method handles API에 대해 논의했다. 이 두 가지 기술은 현대 JVM 플랫폼의 주요 부분이다. 이 두 메커니즘 모두 생태계 전반에 걸쳐 사용이 증가하고 있다. 예를 들어, invokedynamic은 Java 9 이상에서 새로운 형태의 문자열 연결을 구현하는 데 사용되었다.

이러한 기능을 이해하면 플랫폼의 가장 내부적인 작동 원리와 Java 애플리케이션이 의존하는 현대 프레임워크에 대한 핵심 통찰력을 얻을 수 있다.
#### [출처](https://blogs.oracle.com/javamagazine/post/behind-the-scenes-how-do-lambda-expressions-really-work-in-java)







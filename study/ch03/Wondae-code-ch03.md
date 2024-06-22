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
JVM에서 람다를 다루는 방식 [출처](https://blogs.oracle.com/javamagazine/post/behind-the-scenes-how-do-lambda-expressions-really-work-in-java)

진행 중...
## 5. 스트림 활용
### 5.1 필터링
스트림의 요소를 선택하는 방법은 Predicate 필터링과 고유 요소 필터링이 있다.
#### 5.1.1 Predicate 필터링
스트림 인터페이스의 filter 메서드는 Predicate를 인수로 받아 조건과 일치하는 모든 요소를 포함하는 스트림을 리턴한다.
```Java
List<Dish> vegetarianMenu = menu.stream().filter(Dish::isVegiterian).collect(toList());
```
#### 5.1.2 고유 요소 필터링
스트림은 고유 요소로 이루어진 스트림을 리턴하는 distinct 메서드를 지원한다. (고유 여부는 hashCode와 equals로 판단)
```Java
List<Integer> numbers = Array.asList(1,2,1,3,4,5,2);
numbers.stream().distinct().forEach(System.out::println);

// 1, 2, 3, 4, 5 출력
```
> distinct 메서드는 HashSet을 이용하여 중복을 판단
### 5.2 스트림 슬라이싱
#### 5.2.1 Predicate를 이용한 슬라이싱
자바 9에선 스트림의 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 두가지 메서드를 지원한다.
##### takeWhile
이미 정렬되어 있는 스트림에 대해 predicate 조건을 만족할 시 더 검사하지 않고 조건을 만족하는 스트림을 반환한다.
```Java
List<Dish> filteredMenu = specialMenu
                                .stream()
                                .takeWhile(dish -> 
                                    dish.getCalories() < 320
                                )
                                .collect(toList());
```
##### dropWhile
takeWhile과 반대로, 스트림에 대해 predicate 조건을 만족할 시 더 검사하지 않고 조건을 만족하지 않는 스트림을 반환한다.
```Java
List<Dish> filteredMenu = specialMenu
                                .stream()
                                .dropWhile(dish -> 
                                    dish.getCalories() < 320
                                )
                                .collect(toList());
```
#### 5.2.2 스트림 축소
스트림은 주어진 값 이하 크기를 갖는 새로운 스트림을 반환하는 limit() 메서드가 있다. 스트림이 정렬되어 있을 시 처음 요소 n개를 반환할 수 있다.
```Java
List<Dish> filteredMenu = specialMenu
                                .stream()
                                .dropWhile(dish -> 
                                    dish.getCalories() > 300
                                )
                                .limit(3)
                                .collect(toList());
```
#### 5.2.3 요소 건너뛰기
스트림은 limit와 반대로, 초반 요소 n개를 건너뛰고 나머지 요소를 담은 스트림을 반환한다. 만약 스트림의 크기보다 큰 값을 넣는다면, 빈 스트림을 반환한다.
```Java
List<Dish> filteredMenu = specialMenu
                                .stream()
                                .dropWhile(dish -> 
                                    dish.getCalories() > 300
                                )
                                .skip(2)
                                .collect(toList());
```
### 5.3 매핑
#### 5.3.1 스트림 각 요소에 함수 적용
스트림은 함수를 인수로 받는 map을 통해 각 요소에 연산 적용 후 결과를 스트림으로 받을 수 있다.
```Java
List<Dish> dishNames = menu.stream().map(Dish::getName).collect(toList());
```
#### 5.3.2 스트림 평탄화
```Java
List<String> words = new ArrayList();
words.add("Hello");
words.add("World");
```
단어 리스트에서 고유 문자로 이뤄진 리스트를 출력해보자.
```Java
words.stream()
        .map(word -> word.split(""))
        .distinct()
        .collect(toList());
```
이럴 경우, map이 String[]을 리턴하기 때문에, `[{"H","e","l","o"}, {"W","o","r","l","d"}]`라는 결과가 나온다.

**map과 Arrays.stream 활용**
```Java
words.stream()
        .map(word -> word.split(""))
        .map(Arrays::stream)
        .distinct()
        .collect(toList());
```
이럴 경우 List<Stream<String>>이 만들어지면서, 의도한 결과가 나오지 않는다.

**flatMap 사용**
```Java
List<String> uniqueCharacters = words.stream()
                                .map(word -> word.split(""))
                                .flatMap(Arrays::stream)
                                .distinct()
                                .collect(toList());
```
flatMap 메서드는 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결한다.
### 5.4 검색과 매칭
#### 5.4.1 Predicate가 적어도 한 요소와 일치하는 지 검사
Predicate를 매개변수로 받는 anyMatch를 사용하면, 스트림 속 적어도 한 요소가 만족하는 지 결과를 보여줄 수 있다.

결과를 boolean으로 표현하기 때문에 anyMatch는 최종 연산이다.
#### 5.4.2 Predicate가 모든 요소와 일치하는 지 검사

1. allMatch : Predicate를 매개변수로 받는 allMatch를 사용하면, 스트림 속 모든 요소가 만족하는 지 결과를 보여줄 수 있다.
2. noneMatch: allMatch와 정 반대로, 스트림 속 모든 요소가 만족하는 요소가 없는지 확인한다.
> **쇼트서킷 평가**
>
> 전체 스트림을 확인하지 않아도 결과를 찾을 수 있다. 예를 들어서 and 연산을 통해 확인하는 경우에, 하나만 틀려도 그 뒤를 확인하지 않아도 전체 결과는 틀리듯이, 이런 연산을 **쇼트서킷**이라 부른다.

#### 5.4.3 요소 검색
findAny 메서드는 조건을 만족하는 결과를 발견할 시 바로 리턴한다.
> **Optional**
>
> Optional<T>는 값의 여부나 부재 여부를 표현하는 컨테이너 클래스다. 값이 없을 때 null을 리턴할 수 있는데, 그럼 문제가 발생할 여지가 크므로, Java 8에서 도입된 개념이다.
>
>* isPresent() : 값을 포함하고 있으면 true, 아니면 false를 리턴한다.
>* ifPresent(Consumer<T> block) : 값이 있으면 block을 실행한다.
>* T get() : 값이 있으면 반환하고, 없을 시 NoSuchElementException을 반환한다.
>* T orElse(T other) : 값이 있으면 값을 반환하고, 없으면 기본값을 반환한다.

#### 5.4.2 첫번째 요소 찾기
논리적인 순서가 정해진 스트림에서, 조건을 만족하는 첫번째 요소를 찾기 위해 findFirst() 메서드를 사용한다.
> **findAny() vs findFirst()**
> 병렬 실행 시, 첫번쨰 요소를 찾는 것은 힘들기에, firstAny() 메서드를 사용한다.

### 5.5 리듀싱
최종 결과가 Integer와 같은 결과가 나올 때 까지 모든 요소를 반복적으로 처리하는 연산을 리듀싱 연산이라 한다.

#### 5.5.1 요소의 합
reduce를 사용하여 스트림의 합을 표현하려면 다음과 같이 한다.
```Java
// reduce의 매개변수 : 초기값 0, 할 연산
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
```
**초기값 없음**
초기 값을 넣지 않도록 오버로드 된 reduce도 있다. 이런 경우 Optional을 리턴한다.

#### 5.5.2 최댓값과 최솟값
최댓값과 최소값을 찾을 때도 reduce를 활용할 수 있다.
```Java
Optional<Integer> max = numbers.stream().reduce(Integer::max);

Optional<Integer> max = numbers.stream().reduce(Integer::min);
```

> **reduce 메서드의 장점과 병렬화**
> 
> reduce를 사용하면 내부 반복이 추상화 되면서 병렬 실행할 수 있게 된다.
> 현재는 parellelStream()을 사용하는데, 자세한 내용은 7장에서 다룰 것이다.

> **스트림 연산 : 상태 없음과 상태 있음**
>
> 스트림을 사용하는 연산들은 내부 상태를 고려해야 한다.
> 
> map, filter와 같은 연산은 각 요소를 받아 결과를 출력 스트림으로 보낸다. 이러한 연산을 상태가 없는 연산이라 한다.
>
> reduce, sum, max와 같은 연산은 내부 상태를 저장하며 진행해야 하므로 상태가 있는 연산이라 한다. 스트림에서 처리하는 요소의 수와 상관없이 내부 상태는 한정되어 있다.
>
> sorted나 distinct 같은 연산은 역시 과거 이력을 알아야 하기 때문에 모든 요소가 버퍼에 추가되어 있다. 따라서 마찬가지로 상태가 있는 연산이다

### 5.7 기본형 특화 스트림
Java 8에선 boxing 비용을 피할 수 있도록 IntStream, DoubleStream, LongStream 세 가지 기본형 특화 스트림을 제공한다. 각각의 인터페이스는 숫자 스트림의 합을 구하는 sum과 같이 자주 사용하는 숫자 관련 리듀싱 연산을 제공한다.

**숫자 스트림으로 매핑**

mapToInt(), mapToLong(), maptoDouble() 메서드를 사용하면 stream을 기본형 스트림으로 바꿀 수 있다.

기본형 스트림은 boxed() 메서드를 사용하면 일반 스트림으로 돌아갈 수 있다

**기본값 : OptionalInt, OptionalDouble, OptionalLong**

Optional에서 따온 개념으로 Optional의 기본형 특화 버전이다.

#### 5.7.2 숫자 범위
특정 범위의 수를 이용해야하는 상황에서, IntStream과 LongStream은 range와 rangeClosed 두가지 메서드를 제공한다. range는 열린 범위 (양 끝 포함), rangeClosed는 닫힌 범위 (양 끝 미포함)이란 차이가 있다.

### 5.8 스트림 만들기
#### 5.8.1 값으로 스트림 만들기
Stream.of()을 통해 스트림을 만들 수 있다.
```Java
Stream<String> stream = Stream.of("Modern ", "Java ", "In ", "Action");
stream.map(String::toUpperCase).forEach(System.out::println)
```

#### 5.8.2 nullable한 객체로 스트림 만들기
Java 9에선 nullable한 객체를 스트림으로 만들 수 있다. 예를 들어서 System.getProperty는 제공된 키에 대응하는 속성이 없으면 null을 반환한다. 이런 메서드를 스트림에 활용하기 위해선, null을 명시적으로 선언해야 한다.

```Java
// JAVA 8
String homeValue = System.getProperty("home");

Stream<String> homeValueStream = home == null ? Stream.empty() : Stream.of(value);

// JAVA 9
Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));
```

#### 5.8.3 배열로 스트림 만들기
Arrays.stream을 이용하여 스트림을 만들 수 있다.
```Java
int[] numbers = {2, 3, 5, 7, 11, 13};

int sum = Arrays.stream(numbers).sum();
```

#### 5.8.4 파일로 스트림 만들기
파일 처리 등 I/O 연산에 사용하는 자바의 NIO API (비블록 I/O)에도 스트림을 활용할 수 있다.

이를 이용하면, 파일에서 고유한 단어 수를 찾는 프로그램을 만들 수 있다.
```Java
long uniqueWord = 0;
try(Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())){
    uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
                        .distinct()
                        .count();
} catch (IOException e){
    // 예외처리
}
```

#### 5.8.5 함수로 무한 스트림 만들기
스트림 API는 함수에서 스트림을 만들 수 있는 정적 메서드 Stream.iterate와 Stream.generate를 제공한다. 이를 사용하여 **무한 스트림**을 만들 수 있는데, 그렇게 고정되지 않은 크기의 스트림을 만들 수 있다. 하지만 보통 무한히 출력하지 않게 하기 위해 limit()를 사용한다.

**iterate**
```Java
Stream.iterate(0, n -> n + 2)
    .limit(10)
    .forEach(System.out::println);
```
iterate 메서드는 초기 값을 받아 새로운 값을 끊임없이 만들 수 있다. 따라서 끝이 없기 때문에 무한 스트림을 만든다. 그리고 이런 스트림을 언바운드 스트림이라 한다.

**generate**

generate 메서드는 연속된 값을 만들지 않고, supplier\<T>를 인수로 받아 새로운 값을 생산한다.
```Java
Stream.generate(Math::random)
    .limit(5)
    .forEach(System.out::println);
```

----
## Reflection
Reflection API는 런타임에 클래스, 메서드, 필드 등의 정보를 동적으로 탐색하고 조작할 수 있게 한다.

 이를 통해 프로그램의 구조와 동작을 분석하거나, 실행 시점에 동적으로 객체를 생성하고 메서드를 호출하며, 필드 값을 읽고 쓸 수 있다.
### Reflection의 주요 구성 요소
1.	Class 클래스:
* 	클래스의 메타데이터(클래스 이름, 패키지 이름, 부모 클래스, 인터페이스 등)를 얻을 수 있다.
*	예: Class<?> clazz = MyClass.class;
2.	Field 클래스:
*	클래스의 필드(변수)에 대한 정보를 얻고, 해당 필드의 값을 읽거나 쓸 수 있다.
*	예: Field field = clazz.getDeclaredField("fieldName");
3.	Method 클래스:
*	클래스의 메서드에 대한 정보를 얻고, 해당 메서드를 호출할 수 있다.
*	예: Method method = clazz.getDeclaredMethod("methodName", parameterTypes);
4.	Constructor 클래스:
*	클래스의 생성자에 대한 정보를 얻고, 새로운 객체를 생성할 수 있다.
*	예: Constructor<?> constructor = clazz.getDeclaredConstructor(parameterTypes);

### 주요 메서드와 사용 예제
1. 클래스 정보 얻기
```Java
Class<?> clazz = MyClass.class;

System.out.println("Class Name: " + clazz.getName());
```
2. 필드 정보 얻기 및 조작
```Java
Field field = clazz.getDeclaredField("fieldName");
field.setAccessible(true); // private 필드 접근 허용

Object value = field.get(instance); // 필드 값 읽기
field.set(instance, newValue); // 필드 값 쓰기
```
3. 메서드 정보 얻기 및 호출
```Java
Method method = clazz.getDeclaredMethod("methodName", parameterTypes);
method.setAccessible(true); // private 메서드 접근 허용
Object returnValue = method.invoke(instance, args); // 메서드 호출
```
4. 생성자를 사용한 객체 생성
```Java
Constructor<?> constructor = clazz.getDeclaredConstructor(parameterTypes);
constructor.setAccessible(true); // private 생성자 접근 허용
Object newInstance = constructor.newInstance(args); // 객체 생성
```
### Reflection의 장점

*	동적 처리: 런타임에 객체의 타입이나 메서드 등을 알 수 없을 때 유용함.
*	프레임워크 개발: Spring, Hibernate 등 많은 Java 프레임워크에서 Reflection을 사용하여 객체를 동적으로 관리.

### Reflection의 단점

*	성능 저하: 런타임에 메타데이터를 읽고 조작하는 것은 일반적인 메서드 호출보다 느림.
*	안전성 문제: 접근 제어자(private, protected 등)를 무시할 수 있어 잘못 사용하면 보안 문제나 예기치 않은 버그가 발생할 수 있음.
*	복잡성 증가: 코드가 복잡해지고 가독성이 떨어질 수 있음.

### 실제 사용 사례
1.	프레임워크 개발
*	의존성 주입 : 스프링(Spring) 같은 프레임워크는 의존성 주입을 위해 Reflection을 사용하여 런타임에 객체를 생성하고, 필요한 필드와 메서드를 설정한다.
*	어노테이션 처리 : Hibernate나 JUnit 같은 프레임워크는 Reflection을 사용하여 애노테이션을 읽고, 이를 기반으로 특정 동작을 수행한다.
2.	동적 메서드 호출
*	메서드 이름이나 파라미터를 컴파일 타임에 알 수 없는 경우, Reflection을 통해 런타임에 메서드를 호출할 수 있다.
*	예시 : 스크립팅 엔진이나 플러그인 시스템에서 동적으로 메서드를 호출할 때 사용.
3.	런타임 클래스 정보
*	객체의 클래스 정보를 런타임에 동적으로 얻어야 할 때 사용.
*	예시 : JSON 직렬화/역직렬화 라이브러리에서 객체의 필드와 타입 정보를 얻어 JSON으로 변환하거나, JSON을 객체로 변환할 때 사용.
4.	디버깅 및 테스트 도구
*	디버깅 도구나 테스트 프레임워크는 Reflection을 사용하여 객체의 상태를 검사하고, private 메서드나 필드에 접근하여 테스트를 수행할 수 있다.
*	예시 : JUnit에서는 private 메서드를 테스트하기 위해 Reflection을 사용할 수 있다.
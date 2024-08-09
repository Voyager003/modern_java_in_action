## 컬렉션 API 개선
이번 장에서는, 자바8, 9에서 추가된 새로운 컬렉션 API에 대하여 베울 것이다.
### 8.1 컬렉션 팩토리
자바9에선 작은 컬렉션 객체를 만드는 컬렉션 팩토리 기법이 생겼다. 그런데 이게 왜 필요할까?? 아래의 예시를 통해 알아보자.
```Java
List<String> friends = new ArrayList<>();

friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
```
문자열 3개를 저장하는데도 많은 코드가 필요하다. 다음처럼 `Arrays.asList()` 메서드를 사용하면 간단하게 줄일 수 있다.

```Java
Arrays.asList("Raphael", "Olivia", "Thibaut");
```
고정된 크기의 리스트를 만들었으니 갱신은 할 수 있지만 추가하거나 삭제할 순 없다. 만약 할 경우, `UnsupportedOperationException` 예외가 발생한다.

**UnsupportedOperationException 예외**

내부적으로 고정된 크기의 배열로 구현되었기 때문에 발생한다.
그렇다면 set으로 만들 경우에는, List를 인수로 받는 HashSet의 생성자, StreamAPI를 통해 만들 수 있다.
```Java
Set<String> friends = 
        new HashSet(Arrays.asList("Raphael", "Olivia", "Thibaut"));

Set<String> friends = Stream.of("Raphael", "Olivia", "Thibaut")
                             .collect(Collectors.toSet());
```
하지만 두 방법 모두 매끄럽지 못하며 불필요한 객체 할당을 한다.

따라서 리스트의 새로운 기능부터 시작하여 컬렉션을 만드는 새로운 방법을 소개한다.
> 컬렉션 리터럴
>
> 추가 조사 필요

#### 8.1.1 리스트 팩토리
`List.of` 팩토리 메서드를 이용하여 간단하게 리스트를 만들 수 있다.
```Java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
```
하지만 이렇게 생성한 List는 요소를 추가하거나 삭제할 수 없다. 불편하게 느껴질 수도 있지만, 몇가지 장점이 있다.
1. 리스트 크기의 불변성 보장 : 요소가 바뀌는 것까지 막을 순 없다.
2. null 요소 금지

> **오버로드 vs 가변인수**
>
> List.of은 다양한 오버로딩을 통해 구현되었는데, 가변 인수를 사용하지 않는 이유는 다음과 같다. 가변 인수는 내부적으로 추가 배열을 할당하여 리스트로 감싼다. 따라서 배열을 할당하고 나중에 GC를 해야한다. 따라서 지양하는 것이며, 10개 이상의 요소를 만들 때는 가변인수를 사용하고 그 이하는 오버로딩된 메서드를 사용한다.

> **팩토리 메서드 vs Stream API**
>
> * 팩토리 메서드는 데이터 처리 형식을 설정하거나 데이터를 변환하지 말아야할 때 사용한다.
> * 다른 경우에는 Stream API를 사용한다.

#### 8.1.2 집합 팩토리
`List.of`과 마찬가지로, 불변성 집합도 `Set.of`을 통해 만들 수 있다.
```Java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
```

#### 8.1.3 맵 패곹리
자바 9에서는, 두가지 방법으로 불변성을 갖는 맵을 초기화할 수 있다. 첫번째는 홀수번째에 key, 짝수번쨰에 value가 들어가는 방식이다.
```Java
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 26, "Thibaut", 25);
```

두번째는 Map.Entry를 사용하는 방법이다.
```Java
Map<String, Integer> ageOfFriends = Map.of(
    entry("Raphael", 30), entry("Olivia", 26), entry("Thibaut", 25)
    );
```

### 8.2 리스트와 집합 처리
자바 8에선, List, Set 인터페이스에 아래와 같은 메서드가 추가되었다.
* `removeIf` : predicate를 만족하는 요소를 제거한다.
* `replaceAll` : 리스트에서 UnaryOperator를 사용해 요소를 바꾼다.
* `sort` : 리스트를 정렬한다.

위 메소드들은 새로운 메서드를 만드는 것이 아닌, 호출한 컬렉션을 바꾼다.

#### 8.2.1 removeIf 메서드
```Java
List<String> transaction = new ArrayList<>();

transaction.add("0asdvaasydf");
transaction.add("sdfgxvbxsgf");
transaction.add("xvbxbsfgsqa");

for(String t : transaction){
    if(Character.isDigit(t.charAt(0))){
        transaction.remove(t);
    }
}
```

위와 같은 코드는 `ConcurrentModificationException`을 일으킨다. 왜냐하면 for-each는 내부적으로 Iterator를 사용하기 떄문이다. 위 반복문은 아래와 같이 구현되는데, 이렇게 되면 컬렉션의 상태가 동기화되지 않는다.
```Java
for(Iterator<String> iterator = transaction.iterator() ; iterator.hasNext();){
    String t = iterator.next();
    if(Character.isDigit(t.charAt(0))){
        transaction.remove(t);
    }
}
```
아래와 같이 해결할 순 있다. 다만 코드가 복잡해진다.

```Java
for(Iterator<String> iterator = transaction.iterator() ; iterator.hasNext();){
    String t = iterator.next();
    if(Character.isDigit(t.charAt(0))){
        iterator.remove();
    }
}
```

자바 8에선 removeIf을 통해 다음과 같이 바꿀 수 있다.
```Java
transaction.removeIf(t -> Character.isDigit(t.charAt(0)));
```
이렇게 작성하면 코드가 단순해지며 버그도 예방할 수 있다.

#### 8.2.2 replaceAll 메서드
List의 `replaceAll` 메서드를 이용해 리스트의 각 요소를 새로운 요소로 바꿀 수 있다. Stream API를 이용하면 해결할 수 있지만, 새로운 컬렉션을 만들게 된다. 기존 컬렉션을 바꾸기 위해선 , ListIterator 객체를 이용할 수 있다.
```Java
for(ListIterator<String> iterator = referenceCodes.listIterator(); iterator.hasNext()){
    String code = iterator.next();
    iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
}
```
이런 복잡한 코드를 `replaceAll`을 이용하면 쉽게 구현할 수 있다.

```Java
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```

### 8.3 맵 처리
Map 인터페이스에도 몇가지 디폴트 메서드가 추가되었다. 디폴트 메서드에 관한 자세한 내용은 13장에서 다룰 것이다.

#### 8.3.1 forEach 메서드
기존의 key와 value를 반복하며 확인하는 방법은 참 귀찮다.
```Java
for(Map.entry<String, Integer> entry : ageOfFriends.entrySet()){
    String friend = entry.getKey();
    int age = entry.getValue();
    System.out.println(friend + " is " + age + " years olds");
}
```

아래와 같이 key와 value를 받는 BiConsumer를 통해 간단하게 바꿀 수 있다.

```Java
ageOfFriends.forEach((friend, age) -> 
        System.out.println(friend + " is " + age + " years olds"));
```

#### 8.3.2 정렬 메서드
다음과 같이 두개의 새로운 메서드를 사용하면 맵의 항목을 value 또는 key를 기준으로 정렬할 수 있다.

* **Entry.comparingByValue**
* **Entry.comparingByKey**

> **HashMap 성능**
>
> 추후 정리

#### 8.3.3 getOfDefault 메서드
만약 찾으려는 키가 없는 경우 null이 반환되고, `NullPointerException`이 발생할 수 있다. 그래서 매번 null을 체크해야 하는데, `getOfDefalut`메서드를 사용하면 쉽게 이 문제를 해결할 수 있다. 이 메서드는 첫번째 인수로 key를, 두번째 인수로 null일 시 반환할 기본값을 받는다. 만약 value가 null일 시 null이 반환될 수도 있다.

#### 8.3.4 계산 패턴
맵에 key가 존재하는지 여부에 다라 어떤 동작을 실행하고 결과를 저장해야 한다면, 다음과 같은 메서드들을 사용한다.
* computeIfAbsent : 제공된 키에 대한 값이 없으면 키를 이용해 새 값을 맵에 추가한다.
* computeIfPresent : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.
* compute : 제공된 키로 새 값을 계산하고 저장한다.

value로 컬렉션을 갖는 맵에도 유용하게 사용할 수 있따.
  
#### 8.3.5 삭제 패턴
제공된 키에 해당하는 맵을 제거하는 remove 대신 자바 8 에선 키가 특정 값과 연관이 있을 때 삭제하는 오버로드 메서드를 제공한다.
```Java
// 키가 있고, 해당하는 값이 value일 때 제거한다.
map.remove(key, value);
```

#### 8.3.6 교체 패턴
맵의 항목을 바꿀 수 있는 두가지 메서드가 추가되었다.

* `replaceAll()` : BiFunction에 해당하는 모든 값을 교체한다.
* `Replace()` : 키가 존재하면 맵의 값을 바꾼다.

#### 8.3.7 합침
한 맵의 모든 요소를 다른 맵에 추가할 때 `putAll()`을 사용할 수 있다.
만약 중복된 키가 있을 경우 `merge()`를 사용하면서, 중복된 키를 어떻게 합칠지 결정하는 BiFunction을 사용할 수 있다.

`merge()`를 사용할 때, 만약 지정된 키와 연결된 값이 없거나 null인 경우 키를 null이 아닌 값과 연결한다.

### 8.4 개선된 ConcurrentHashMap
ConcurrentHashMap은 동시성 친화적이며 최신 기술을 반영한 HashMap이다.
내부 자료구조의 특정 부분만 Lock을 적용하여 동시 추가 갱신 작업을 허용한다.

#### 8.4.1 reduce와 검색
ConcurrentHashMap은 새로운 세가지 연산을 제공한다.
* forEach
* reduce
* search : null이 아닌 값을 반환할 떄 까지 각 Entry에 함수 적용

다음처럼 네 가지 연산 형태를 지원한다
* key, value로 연산 (forEach, reduce, search)
* key로 연산 (forEachKey, reduceKeys, searchKeys)
* value로 연산 (forEachValue, reduceValues, searchValues)
* Map.Entry로 연산 (forEachEntry, reduceEntries, searchEntries)

이들 연산은 상태를 잠그지 않고 연산을 수행한다. 따라서 이들 연산은 객체, 값, 순서에 의존하지 않아야 한다.

그리고 연산에 병렬 연산을 할 기준인 threshold를 지정해야한다. 만약 threshold보다 작을 경우 순차적 연산을 하며, 이상일 경우 병렬적인 연산을 한다.

#### 8.4.2 계수
ConcurrentHashMap은 맵의 매핑 개수를 제공하는 mappingCount()를 제공한다. 이는 Long을 반환하는데, 매핑의 개수가 int의 범위를 넘어갈 것 같을 때 사용하면 좋다.

#### 8.4.3 집합뷰
ConcurrentHashMap은 이를 Set으로 변환하는 keySet()을 제공한다. 이는 바뀌는 Map에 영향을 주는데, newKeySet()을 사용하면 유지되는 Set을 만들 수 있다.

-------
## 추가 조사
### 자바 10 이후 CollectionAPI 변천사
#### Java 10 : CopyOf 메서드
Java 10에는 List, Set, Map에 CopyOf 메서드가 추가됐다. 기존 컬렉션의 변경 불가능한 복사본을 생성한다. 만약 이미 변경 불가능하다면 동일한 인스턴스를 반환한다.
```Java
List<String> originalList = List.of("a", "b", "c");
List<String> copyList = List.copyOf(originalList);
```

#### Java 11 : toArray(IntFunction\<T[]\> generator) 메서드
Java 11에는 Collection 인터페이스에 새로운 메서드 toArray()가 추가됐다. 이 메서드는 컬렉션을 type-safety한 배열로 변환할 수 있다.
```Java
List<String> list = List.of("a", "b", "c");
String[] array = list.toArray(String[]::new);
```

#### Java 12 : Collector 개선
Java 12에서는 Collectors에 teeing Collector가 추가됐다.

이 컬렉터는 동일한 입력 요소에 대해 여러 가지 작업을 수행한 후 결과를 결합할 수 있게 한다.
스트림을 두 가지 방식으로 처리한 후 결과를 병합해야 하는 시나리오에 유용합니다.

```Java
import static java.util.stream.Collectors.*;

Map<Boolean, List<String>> result = Stream.of("a", "bb", "ccc")
    .collect(partitioningBy(s -> s.length() > 1, teeing(
        toList(),
        counting(),
        (list, count) -> Map.of("list", list, "count", count)
    )));
```

#### Java 16 : 새로운 스트림 연산
Java 16에서는 Stream 인터페이스에 두 가지 새로운 메서드가 도입됐다.
* Stream.toList() : 스트림에서 변경 불가능한 리스트를 생성한다.
```Java
List<String> list = Stream.of("a", "b", "c").toList();
```

* mapMulti : 메서드를 여러 개의 값으로 plat하게 만드는데, 더 읽기 좋게 한다.
```Java
List<String> list = Stream.of("a", "b", "c")
    .<String>mapMulti((s, consumer) -> {
        consumer.accept(s.toUpperCase());
        consumer.accept(s.toLowerCase());
    })
    .toList();
```

#### Java 21 : 순차 컬렉션 (Sequenced Collection)
Java 21에서는 순차적 컬렉션(SequencedCollection, SequencedSet, SequencedMap) 개념이 도입됐다. 이들은 반복과 접근 모두에서 예측 가능한 순서를 보장한다. 이는 순서가 일관되게 유지되어야 하는 컬렉션에 유용하다.

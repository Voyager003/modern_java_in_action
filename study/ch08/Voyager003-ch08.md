## 컬렉션 API 개선

### 컬렉션 팩토리

#### 리스트 팩토리
```java
// 1. 기존 방식 (변경 가능)
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");

// 2. Arrays.asList() 방식 (변경 불가)
List<String> friends
   = Arrays.asList("Raphael", "Olivia", "Thibaut");

// 3. Stream 방식 (변경 가능)
Set<String> friends
   = Stream.of("Raphael", "Olivia", "Thibaut")
           .collect(Collectors.toSet());

// 4. List Factory (변경 불가) - java9
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
```

컬렉션의 의도치 않게 변경을 사전에 방지할 수 있다. 용도에 따라 데이터 변경이 필요하면 Stream, 필요없다면 Factory를 권장한다. (변경을 제약하는 것이 꼭 나쁜것은 X)

#### 집합 팩토리

```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
```

#### 맵 팩토리
```java
// 10개 이하의 key-value
Map<String, Integer> ageOfFriends
   = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);

// 10개 초과 key-value
Map<String, Integer> ageOfFriends
       = Map.ofEntries(entry("Raphael", 30),
                       entry("Olivia", 25),
                       entry("Thibaut", 26));
```

### 리스트와 집합 처리

- removeIf : Predicate를 만족하는 요소 제거

```java
// 기존 코드 - Iterator와 Collection이 동기화 되지 않은 문제를 해결하느라 사용하기 까다롭다
for (Iterator<Transaction> iterator = transactions.iterator();
             iterator.hasNext(); ) {
           Transaction transaction = iterator.next();
           if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
               iterator.remove();
           }
}

// removeIf 
transactions.removeIf(transaction ->
             Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

- replaceAll

```java
1. 기존 코드
for (ListIterator<String> iterator = referenceCodes.listIterator();
             iterator.hasNext(); ) {
           String code = iterator.next();
           iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
}

2. replaceAll 사용
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) +
             code.substring(1));
```

### 맵 처리

#### for-each

```java
// 기존 코드 
for(Map.Entry<String, Integer> entry: ageOfFriends.entrySet()) {
           String friend = entry.getKey();
           Integer age = entry.getValue();
           System.out.println(friend + " is " + age + " years old");
}

// forEach 사용
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " +
             age + " years old"));
```

#### 정렬 메서드

```java
Map<String, String> favouriteMovies
               = Map.ofEntries(entry("Raphael", "Star Wars"),
               entry("Cristina", "Matrix"),
               entry("Olivia",
               "James Bond"));

favouriteMovies
  .entrySet()
  .stream()
  .sorted(Entry.comparingByKey())
  .forEachOrdered(System.out::println);
```

#### getOrDefault

```java
Map<String, String> favouriteMovies
               = Map.ofEntries(entry("Raphael", "Star Wars"),
              entry("Olivia", "James Bond"));

System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix"));
System.out.println(favouriteMovies.getOrDefault("Thibaut", "Matrix"));
```

#### computeIfAbsent

```java
// 기존 코드
String friend = "Raphael";
List<String> movies = friendsToMovies.get(friend);
if(movies == null) {
    movies = new ArrayList<>();
   friendsToMovies.put(friend, movies);
}
movies.add("Star Wars");

// computeIfAbsent
friendsToMovies.computeIfAbsent("Raphael", name -> new ArrayList<>())
              .add("Star Wars");
```

#### 삭제 패턴

```java
// 기존 코드
String key = "Raphael";
String value = "Jack Reacher 2";
if (favouriteMovies.containsKey(key) &&
     Objects.equals(favouriteMovies.get(key), value)) {
   favouriteMovies.remove(key);
   return true;
} else {
   return false;
}

// remove 사용
favouriteMovies.remove(key, value);
```

#### 교체 패턴

```java
Map<String, String> favouriteMovies = new HashMap<>();
favouriteMovies.put("Raphael", "Star Wars"); 
favouriteMovies.put("Olivia", "james bond");
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```

#### 합치기

```java
Map<String, Long> moviesToCount = new HashMap<>();

// 기존 코드
String movieName = "JamesBond";
long count = moviesToCount.get(movieName);
if(count == null) {
   moviesToCount.put(movieName, 1);
}
else {
   moviesToCount.put(moviename, count + 1);
}

// merge
moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);
```

## 참고자료

- 모던 자바 인 액션
- https://docs.oracle.com/javase/tutorial/collections/streams/parallelism.html


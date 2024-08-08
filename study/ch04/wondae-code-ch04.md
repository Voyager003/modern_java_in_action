## 4. 스트림 소개
자바에는 Collections가 있지만, 그룹화를 하거나 다양한 조건을 적용하는 것이 번거롭다. 또한 대용량의 Collections를 다룰 때 필요한 병렬처리도 복잡하다.
위와 같은 문제를 해결하기 위해, 자바 설계자들은 스트림을 도입하였다.

### 4.1 스트림이란 무엇인가?
스트림은 자바 8에서 추가되었으며, 데이터를 선언형으로 처리할 수 있다. 또한, 별도의 코드 구현 없이 데이터를 투명하게 병렬로 처리 가능하다. 스트림의 기능을 아래 코드를 바꿔가며 느껴보자.
```Java
List<Dish> lowCaloricDishes = new ArrayList<>();
// 누적자로 자료 필터링
for(Dish dish: menu){
	if(dish.getCalories() < 400){
		lowCaloricDishes.add(dish);
	}
}

// 익명 클래스로 요리 정렬
Collections.sort(lowCaloricDishes, new Comparator<Dish>(){
	@Override
	public int compare(Dish dish1, Dish dish2){
		return Integer.compare(dish1,getCalories(), dish2.getCalories());
	}
});

Lists<String> lowCaloricDishedName = new ArrayList<>();
for(Dish dish: lowCaloricDishes){
	// 정렬된 리스트에서 요리 이름 선택
	lowCaloricDishesName.add(dish.getName());
}
```
위 코드에선 lowCaloricDishes라는 가비지 변수를 사용하였다. 이 변수는 컨테이너 역할만 하는 중간 변수인데, 자바 8에서 이런 변수들은 라이브러리 내에서 모두 처리된다.

아래는 자바 8 코드 예시이다.
```Java
List<String> lowCaloricDishesName = 
		menu.stream()
			.filter(d -> d.getCalories() < 400)
			.sorted(comparing(Dish::getCalories))
			.map(Dish::getName)
			.collect(toList());
```
위에서 stream() 대신 parallelStream()으로 바꾸면 멀티코어 아키텍처에서 병렬로 실행 가능하다.

스트림을 사용하여 얻을 수 있는 장점음 다음과 같다.
1. 선언형으로 코드 구현 가능 : loop와 if 없이 동작의 수행만 지정할 수 있다.
2. filter, sorted, map, collect와 같이 여러 빌딩 블록 연산을 통해 데이터 처리 파이프라인을 만들 수 있고, 그럼에도 가독성과 명확성이 유지된다.

고수준 빌딩 블록인 filter, sorted, map, collect 같은 연산들은 멀티 스레딩이던, 싱글 스레딩이던 그것에 구애받지 않고 사용 하능하다.

### 4.2 스트림 시작하기
스트림은 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소로 정의할 수 있다.
* **연속된 요소** : 컬렉션처럼, 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공한다. 컬렉션은 자료구조라 구현과 관련된 저장 및 접근이 주된 요소라면, 스트림은 데이터를 처리하는 표현 계산식이 주를 이룬다.
* **소스** : 스트림은 컬렉션, 배열, I/O 자원등 데이터 제공 소스로부터 데이터를 소비한다.
* **데이터 처리 연산** : 스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 DB와 비슷한 연산을 지원한다. 스트림 연산은 순차적 또는 병렬적으로 실행할 수 있다.

그리고 중요한 두가지 특징이 있다.
* **파이프라이닝** : 대부분의 스트림 연산은 스트림 연산끼리 연결하여 거대한 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다. 이를 통해 `게으름 (laziness)`, `쇼트 서킷 (short-circuiting)`과 같은 최적화도 얻을 수 있다.
* **내부 반복** : 반복자를 이용하여 명시적으로 반복하는 컬렉션과 달리, 스트림은 내부 반복을 지원한다.

람다의 연산들이 수행하는 작업은 다음과 같다.
* **filter** : 람다를 인수로 받아 스트림에서 특정 요소를 제외시킨다.
* **map** : 람다를 인수로 받아 요소를 다른 요소로 변환하거나 정보를 추출한다.
* **limit** : 일정 개수 이상의 요소가 스트림에 저장되지 못하게 스트림 크기를 축소시킨다.
* **collect** : 스트림을 다른 형식으로 변환한다.

### 4.3 스트림과 컬렉션
스트림과 컬렉션 모두 순차적으로 값에 접근한다는 공통점이 있다.
차이점들은 다음과 같다.
#### 1. 데이터 연산 시점
컬렉션은 모든 값을 메모리에 저장하지만, 스트림은 요청할 때만 요소를 계산한다.
컬렉션은 데이터를 생산하고, 스트림은 데이터를 소비한다.
#### 2. 한번만 탐색 가능
반복자와 마찬가지로 스트림도 한 번만 탐색할 수 있다. 탐색된 요소는 소비되어 다시 사용할 수 없다. 다시 사용하기 위해선 같은 소스에서 새롭게 스트림을 만들어야 한다.
#### 3. 외부 반복과 내부 반복
컬렉션을 사용하면, 사용자가 직접 요소를 반복해야한다. 이를 외부 반복이라 한다.
스트림에선 반복을 알아서 처리하고 결과 스트림 값을 어디에 저장해주는 내부 반복을 사용한다.

외부 반복은 명시적으로 컬렉션 항목을 가져와서 처리한다. 그렇기 때문에, 사용자가 직접 관리해야 하는 부분이 많다.
(예시 : 병렬성)

### 4.4 스트림 연산
스트림의 연산은 크게 두가지로 나뉘는데, 다른 연산과 연결할 수 있는 중간 연산과 스트림을 담는 최종 연산 두가지로 나눌 수 있다.
#### 1. 중간 연산
중간 연산은 다른 스트림을 반환한다. 따라서 여러 중간 연산을 연결하여 질의를 만들 수 있다. 그리고 중간연산들은 스트림 파이프라인을 실행하기 전엔 아무 연산도 수행하지 않는다 (게으름).

#### 2. 최종 연산
최종 연산은 스트림 파이프라인에서 결과를 도출한다. 주로 List, Integer, void 등 스트림 이외의 결과를 반환한다.

#### 3. 스트림 이용하기
스트림 이용 과정은 다음과 같이 세 가지로 요약 가능하다.
* 질의를 수행할 데이터 소스
* 스트림 파이프라인을 구성할 중간 연산 연결
* 스트림 파이프라인을 실행하고 결곽를 만들 최종 연산

아래 요소는 간단하게 중간 연산과 최종 연산을 정리해 놓은 것이다.
| 연산 | 형식 | 반환 형식 | 인수 | 디스크립터 |설명|
|---|---|---|---|---|---|
|filter|중간 연산|Stream\<T>|predicate\<T>|T -> boolean
|map|중간 연산|Stream\<T>|Function\<T, R>|T -> R
|limit|중간 연산|Stream\<T>|||
|sorted|중간 연산|Stream\<T>|Comparator\<T>|(T, T) -> int
|distinct|중간 연산|Stream\<T>|||
|forEach|최종 연산|void|||스트림의 요소를 소비하며 람다 적용
|count|최종 연산|long (generic)|||스트림의 요소 개수 반환
|collect|최종 연산||||맵, 리스트, 셋 형식의 컬렉션을 만든다.

## 추가 조사
### 다른 언어에서 쓰이는 스트림과 유사한 개념들
1. Python의 Generator와 Generator Expression
    -   Python의 Generator는 데이터를 메모리에 모두 로드하지 않고 필요한 데이터만 생성하는 방식을 제공
    -   Generator Expression은 Generator를 사용하여 데이터 처리 로직을 간결하게 작성할 수 있게 해줍니다.
2. JavaScript의 Array.prototype.* 메서드
    -   JavaScript의 Array 객체는 map, filter, reduce 등 스트림과 유사한 메서드를 제공
    -   이를 통해 데이터 처리 로직을 함수형 프로그래밍 스타일로 작성 가능
3.  Scala의 Collection API
    -   Scala는 함수형 프로그래밍 언어로, 컬렉션 API를 통해 스트림과 유사한 기능을 제공
    -   map, flatMap, filter 등의 메서드를 사용하여 데이터 처리 로직 작성 가능
4.  C#의 LINQ (Language Integrated Query)
    -   C#에서 제공하는 LINQ는 데이터 처리를 위한 쿼리 표현식을 제공합니다.
    -   LINQ는 배열, 컬렉션, 데이터베이스 등 다양한 데이터 소스를 추상화하고 선언형 프로그래밍을 지원합니다.

### 스트림과 일반 반복문의 속도 차이
스트림의 다양한 장점에도 불구하고 단순한 for 반복문보다 속도가 느리다.
```Java
List<Integer> l = createIntList(100);

// 일반 반복문
long startTime = System.nanoTime();
for(int i = 0; i < l.size(); i++) {
	l.set(i, l.get(i) + 1);
}
long endTime = System.nanoTime();
System.out.println(String.format("for-loop: %dns", endTime - startTime));

// 스트림
startTime = System.nanoTime();
l = l.stream().map(n -> n--).collect(Collectors.toList()); 
endTime = System.nanoTime(); 
System.out.println(String.format("Stream: %dns", endTime - startTime));
```
size가 100일 때 출력 결과 :
for-loop: 59351ns
Stream: 3023868ns
size가 1000000일 때 출력 결과 :
for-loop: 25930715ns
Stream: 53186341ns

for문은 단순 인덱스 기반이며, 컴파일러가 최적화를 하기 때문에 더욱 빠르다.

그럼에도 불구하고 스트림은 다양한 편의성, 가독성이 있기 때문에 상황에 맞춰서 사용하면 좋을 것이다.
## 동작 파라미터화 코드 전달하기
### 동작 파라미터화
프로그램에서 호출에 따라 어떻게 실행될 지 결정되는 코드 블록 
동작 파라미터화 코드로 할 수 있는 일들
* 리스트의 모든 요소에 대해 *특정한 동작*을 수행
* 리스트 관련 작업을 끝낸 후에 *특정한 다른 동작*을 수행
* 에러가 발생했을 때 *정해진 특정한 다른 동작*을 수행
#### 변화하는 요구사항에 대응하기
##### 녹색 사과 필터링
사과 색을 정의한 enum을 가지고 녹색 사과를 필터링 하는 코드
```Java
enum Color { RED, GREEN }

public static List<Apple> filterGreenApples(List<Apple> inventory){
    List<Apple> result = new ArrayList<>();
	  for(Apple apple : inventory){
			  if(GREEN.equals(apple.getColor())){
					  result.add(apple);
			  }
	  }
	  return result;
}
```
-> 녹색 사과는 필터링 가능하지만, 나중에 빨간색 혹은 enum에 색이 추가되어 다른 색을 필터링 하고자 한다면, 공통적인 부분을 추상화할 필요가 있다.

##### 색을 파라미터화
```Java
public static List<Apple> filterGreenApples(List<Apple> inventory, Color color){
    List<Apple> result = new ArrayList<>();
	  for(Apple apple : inventory){
			  if(color.equals(apple.getColor())){
					  result.add(apple);
			  }
	  }
	  return result;
}
```
다만 비교할 요소(무게, 크기 등)가 추가된다면 코드는 더욱 복잡해진다.

##### 가능한 모든속성으로 필터링 -> very bad
```Java
public static List<Apple> filterGreenApples(
							List<Apple> inventory, Color color, int weight, boolean flag){
    List<Apple> result = new ArrayList<>();
	  for(Apple apple : inventory){
			  if((flag && apple.getColor().equals(color)) || 
				(!flag && apple.getWeight() > weight)){
					  result.add(apple);
			  }
	  }
	  return result;
}
```
=> 이런 문제를 해결하기 위하여 **동작 파라미터화** 적용

#### 동작 파라미터화
좀 더 유연하게 대응하기 위해 선택 조건을 결정하는 방법이 필요하다.
사과의 어떤 속성에 기초하여 boolean 값을 리턴하는 방법을 사용할 수 있다.

선택 조건을 결정하는 인터페이스를 정의하자.
```Java
public interface Apple Predicate {
		boolean test (Apple apple);
}
```

이렇게 다양한 선택 요소에 대해 ApplePredicate를 정의할 수 있다.
```Java
public class AppleGreenColorPredicate implements ApplePredicate{
		@Override
		public boolean test(Apple apple){
				return GREEN.equals(apple.getColor());
		}
}

public class AppleWeightPredicate implements ApplePredicate{
		@Override
		public boolean test(Apple apple){
				return apple.getWeight() > 150;
		}
}
```
이러한 패턴을 Strategy pattern이라 한다. 각 알고리즘을 캡슐화해 놓은 뒤, 런타임에 알고리즘을 선택한다.
메서드에선 패턴을 인자로 받아 처리하도록 해야한다. 이 것을 **동작 파라미터화**라 한다.

##### 추상적 조건으로 필터링
```Java
public static List<Apple> filterApples(
				List<Apple> inventory, Color color, ApplePredicate p){
		List<Apple> result = new ArrayList<>();
		for(Apple apple : inventory){
			  if(p.test(apple)){
					  result.add(apple);
			  }
		}
		return result;
}
```
**코드/동작 전달하기**
요구하는 조건에 대해 적절한 class만 만들어두면 대응할 수 있게 됐다.
이제 우리가 전달한 ApplePredicate 객체에 의해 filterApples 동작이 결정된다.
다만 메서드의 인자로는 객체만 받으므로 class로 래핑을 해야한다. 이건 나중에 lambda를 통해 더욱 간편하게 사용할 수 있다. -> **동작 파라미터화의 강점**

#### 복잡한 과정 간소화
현재는 선택 조건에 대하여 각각의 ApplePredicate를 구현하는 클래스들을 만들어야 한다.
이런 경우 보일러플레이트 코드가 너무 많이지며, 이는 **익명 클래스**로 해결 가능하다.

##### 익명 클래스
익명 클래스는 인자에서 바로 인터페이스를 구현하여 사용가능하다.

##### 익명 클래스 사용
```Java
List<Apple> redApples = filterApples(inventory, new ApplePredicate(){
		@Override
		public boolean test(Apple apple){
				return RED.equals(apple.getColor());
		}
});
```
GUI 어플리케이션에서 이벤트를 핸들링할 때 이러한 구조를 보여준다.
다만, 익명클래스 역시 너무 장황하다.

##### 람다 표현식 사용
```Java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

##### 리스트 형식으로 추상화
```Java
public interface T Predicate {
		boolean test (T t);
}

public static List<T> filter(List<T> list, Predicate p){
		List<T> result = new ArrayList<>();
		for(T t : list){
			  if(p.test(t)){
					  result.add(t);
			  }
		}
		return result;
}
```

#### 실전 예제
Compartaor, Runnavble, Callable, GUI 이벤트 처리

### 추가 조사
#### 전략 디자인 패턴 (Strategy Design Pattern)
객체들이 할 수 있는 행위들에 대해 전략 클래스들을 만들고, 유사한 행위들을 추상화하는 인터페이스를 만들어 메서드에서 동적으로 대응할 수 있게 하는 것.

##### 사용 이유
비슷한 인터페이스를 구현하여 사용하는 클래스들이 많은 경우, 같은 메서드들을 다양한 방법으로 구현하게 되고, 이런 경우 메서드들이 중복되는 경우가 많아 수정이 어려울 때가 있다. 이런 때에 전략 디자인 패턴을 사용한다.

##### Java에서 사용하는 곳
> executed by among others Collections#sort().
> 
> 다른 Collections.sort()에서 실행된다.
* java.util.Comparator#compare()

> the service() and all doXXX() methods take 
> HttpServletRequest and HttpServletResponse and 
> the implementor has to process them
> (and not to get hold of them as instance variables!).
> 
> HttpServletRequest와 HttpServletResponse를 다루는 service()와
> 모든 doXXX() 꼴의 메서드, 그리고 그것을 연산하는 구현체들

* javax.servlet.http.HttpServlet

* javax.servlet.Filter#doFilter()

출처 : [Examples of GoF Design Patterns in Java's core libraries](https://stackoverflow.com/questions/1673841/examples-of-gof-design-patterns-in-javas-core-libraries/2707195#2707195)
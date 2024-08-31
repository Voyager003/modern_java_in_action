## 람다를 이용한 도메인 전용 언어
### 10.1 도메인 전용언어
**DSL(도메인 전용 언어)** : 비즈니스 도메인 문제 해결을 위한 언어. 각 도메인마다 필요한 개념을 표현할 수 있어야 한다. 자바에선 이를 클래스와 메서드 집합으로 해결한다.

DSL은 특정 도메인에 집중하기 때문에 해당 도메인의 복잡성을 잘 다룰 수 있고, 문제 해결에 집중할 수 있다. DSL을 구현할 때는 다음과 같은 필요성을 고려해야 한다.
* 의사소통 : 개발자가 아닌 사람이 읽어도 이해가 되어야 한다.
* 가독성 : 코드를 읽기 쉽게 작성하여 동료 개발자가 유지보수하기 쉽게 해야 한다.

### 10.1.1 DSL의 장단점
#### 장점
* 간결함: 캡슐화를 통해 반복을 피하고 코드를 간결하게 한다.
* 가독성: 도메인 영역의 용어로 비전문가도 쉽게 이해할 수 있다.
* 유지보수: 잘 설계된 DSL로 쉽게 유지보수 가능하다.
* 높은 수준의 추상화 : 높은 단계의 추상화로, 도메인과 관련되지 않은 사항을 숨긴다.
* 집중: 설계 목적에 따라 프로그래머가 특정 코드에 집중할 수 있다. 
#### 단점
* 설계의 어려움: 제한된 언어에 도메인 지식을 담기란 어렵다.
* 개발 비용: 새로운 언어를 만드는 것은 많은 시간과 비용이 걸리며, 유지보수 또한 부담을 준다.
* 추가 우회 계층: 도메인 모델을 감싸며 계층이 하나 더 증가한다.
* 새롭게 배워야 하는 언어: 팀원들이 새로운 언어를 배워야 하는 비용이 든다.
* 호스팅 언어의 한계

### 10.1.2 JVM에서 이용 가능한 다른 해결책
#### 내부 DSL

호스팅 언어를 기반으로 구현한 것으로, 여기선 자바로 구현한 것이다. 자바는 장황하고 딱딱한 문법 때문에 DSL을 만들기에 어려움이 있었다. 하지만 람다 표현식이 등장하며 이 문제를 어느정도 해결할 수 있다.

**장점**
* 새로운 언어를 배우는 비용이 대폭 감소한다.
* JVM으로 다 해결되기 때문에 추가 비용이 들지 않는다.
* 외부 도구를 배울 필요가 없다.
* IDE의 기존 기능을 사용할 수 있다.
* 추가로 DSL을 개발할 때, 같은 호스팅 언어를 사용함으로 합치기 쉬워진다 -> *다중 DSL*

#### 다중 DSL
스칼라는 커링, 임의 변환등 DSL 개발에 유용한 특성을 갖췄다.
```Scala
// f를 i번 실행, stackOverflow가 없음.
def time(i: Int, f: => Unit): Unit = {
    f
    if (i > 1) times(i - 1, f)
}

// hello world를 3번 호출
times(3, println("Hello world"))

// times 함수 커링
def times(i: Int)(f: => Unit): Unit = { f
    if(i > 1 times(i-1))(f)
}

// 함수가 반복할 인수를 받는 한 함수를 가지면서 Int를 익명 클래스로 암묵적 변환
implcit def intToTimes(i: Int) = new { // 암묵적 변환
    // 다른 함수 f를 인수로 받는 times 함수 하나만 정의
    def times(f: => Unit): Unit = {
        // 가장 가까운 범주에서 정의한 두 개의 인수를 받는 함수를 이용
        def time(i: Int, f: => Unit): Unit = {
            f
            if (i > 1) times(i - 1, f)
        }
        // 내부 times 함수 호출
        times(i, f)
    }
}
```

문법적 잡음이 전혀 없으며 개발자가 아닌 사람도 코드를 쉽게 이해가능하다 (???)

#### 외부 DSL
자신만의 구문과 문법으로 새로운 언어를 설계해야한다. 이는 매우 어려운 작업이며, 설계한 목적을 벗어나는 경우가 많다. 무한한 유연성을 제공하여 필요한 특성을 완벽하게 지원 가능하다.

### 10.2 최신 자바의 작은 DSL
기존의 자바는 한개의 추상 메서드를 가진 인터페이스를 가지고 있었지만, 람다 표현식과 메서드 참조를 통해 크게 변화했다.

* Stream 인터페이스 : 컬렉션 조작 용도의 내부 DSL
* Collector 인터페이스 : 데이터 수집을 위한 내부 DSL

### 10.3 자바로 DSL을 만드는 패턴 및 기법
1. **메서드 체인**: 플루언트 API사용을 위해 도메인 객체를 만드는 빌더를 만들어야 한다.
2. **중첩 함수 이용**: 함수 안에 함수를 이용한 도메인 모델 제작
3. **람다표현식을 이용한 함수 시퀀싱**: 위 두가지 방식에 람다 표현식 적용
   ```Java
    Order order = order( o -> {
        o.forCustomer("BigBank")l
        o.buy(t -> {
            t.quantity(80);
            t.price(125.00);
            t.stock(s -> {
                s.symbol("IBM");
                s.market("NYSE");
            })
        });
    });
   ```
4. **DSL에 메서드 참조 사용하기**

### 10.4 실제로 쓰이는 Java8 DSL
1. **JQQQ**: SQL 문법을 작성하는데 사용하는 내부 DSL
2. **큐컴버**: 동작 주도 개발(BDD)를 위해 만들어진 DSL
3. **Spring Integration**: 의존성 주입에 기반한 스프링 프로그래밍 모델 확장

## 추가 정보
DSL 자체가 낯선 개념이라 익숙한 예시를 더 찾아보았습니다.
### Gradle
```gradle
plugins {
    id 'java'
}

group = 'com.example'
version = '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'junit:junit:4.13.2'
}

sourceCompatibility = '1.8'
targetCompatibility = '1.8'

jar {
    manifest {
        attributes 'Main-Class': 'com.example.Main'
    }
}
```
#### Gradle DSL
Gradle의 DSL은 Groovy나 Kotlin을 기반으로 하며, 개발자가 Maven과 같은 XML 기반 빌드 도구에 비해 더 표현력 있고 간결한 방식으로 빌드 스크립트를 작성할 수 있게 한다.
#### 목적
Gradle DSL은 프로젝트 구조, 의존성, 태스크, 빌드 프로세스를 사람이 읽기 쉽고 기계가 실행할 수 있는 방식으로 설명하도록 설계됨.
#### 주요 구성 요소
* 프로젝트 구성
* 태스크 정의
* 의존성 관리
* 플러그인 적용 및 구성

### Groovy
```groovy
// Define a class
class Person {
    String name
    int age
    
    def sayHello() {
        println "Hello, my name is $name and I'm $age years old."
    }
}

// Create an instance of the class
def person = new Person(name: "Alice", age: 30)

// String interpolation and multi-line strings
def message = """
    Welcome to Groovy!
    It's now ${new Date()}.
    Enjoy coding!
"""
println message

// Demonstrate optional typing
def dynamicVar = 10
dynamicVar = "Now I'm a string"
println dynamicVar
```
#### 특성
Groovy는 자바 플랫폼을 위한 프로그래밍 언어이자 스크립팅 언어입니다. 간결하고 친숙한 문법으로 개발자의 생산성을 향상시키도록 설계되었다
#### 동적 타이핑
정적 타이핑을 지원하지만, Groovy는 기본적으로 동적 타입을 사용하여 더 유연한 코드 구조를 가능하게 한다
#### 클로저
Groovy는 클로저를 지원하며, 이는 객체로 전달될 수 있는 코드 블록.
#### 메타프로그래밍
강력한 메타프로그래밍 기능을 제공하여 클래스와 메서드를 런타임에 수정할 수 있다.

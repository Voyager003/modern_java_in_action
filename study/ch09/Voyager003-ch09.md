## 코드의 가독성을 개선한다는 것은?
명확하게 정의하긴 어렵지만 '가독성'이 좋다는 것은 다른 사람이 보더라도 쉽게 이해할 수 있는 코드라면 가독성이 좋다고 말할 수 있다.

Java에서는 앞서 살펴봤던 기능을 통해 간결하고 의도가 명확한 코드를 나타낼 수 있다.

![스크린샷 2024-08-24 오전 12 10 03](https://github.com/user-attachments/assets/2ada31d5-8c9a-42b7-bcef-b94917b0af43)

익명 클래스을 람다 표현식으로 개선하여 가독성을 높일 수 있다.

하지만 모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다.

이유 중 하나는 익명 클래스의 this와 super는 람다 표현식에서는 다른 의미를 갖기 때문.

익명 클래스의 this는 익명 클래스 자신을 가리키지만, 람다 표현식의 this는 람다를 감싸는 클래스를 가리킨다.

또한 익명 클래스는 감싸고 있는 클래스의 변수를 가리킬 수 있지만, 람다 표현식으로는 변수를 가리킬 수 없으며, 익명 클래스는 인스턴스화시 명시적으로 형식이 정해지지만, 람다는 컨텍스트에 따라 달라진다.

---
### 패턴
#### 전략 패턴
전략 패턴은 알고리즘을 보유한 상태에서 런타임에 알고리즘을 선택하는 기법이다.

구성 요소는 크게 세 가지로

1. **Context (문맥) 혹은 클라이언트**: 전략을 사용하는 역할을 수행하는 클래스로 특정 전략에 의존하지 않고, 전략을 변경할 수 있도록 한다.

2. **Strategy (전략 인터페이스)**: 특정 기능(알고리즘)을 위한 인터페이스를 정의하며, 각 전략은 이 인터페이스를 구현하여 특정 기능을 수행한다.

3. **ConcreteStrategy (구체적인 전략 클래스)**: Strategy 인터페이스를 구현한 클래스로, 각각의 클래스는 전략 인터페이스에 정의된 특정 알고리즘을 구현한다.

![SCR-20240824-bdsw](https://github.com/user-attachments/assets/666f57f2-838b-454b-ab69-5248b12cd4fa)

#### 템플릿 메서드
템플릿 메서드 패턴은 상위 클래스에서 알고리즘의 구조를 정의하고, 하위 클래스에서 알고리즘의 세부 단계를 구현하는 디자인 패턴이다. 이 패턴을 사용하면 알고리즘의 골격을 유지하면서, 알고리즘의 특정 단계의 구현을 하위 클래스에서 다르게 할 수 있다.

구성 요소는 다음과 같다.

1. **Abstract Class (추상 클래스)**: 알고리즘의 구조를 정의하고, 일부 메서드를 추상 메서드로 선언하며, 이 추상 메서드는 하위 클래스에서 구현된다.

2. **Concrete Class (구체 클래스)**: 추상 클래스에서 정의된 추상 메서드를 구현하여 알고리즘의 특정 단계를 정의한다.

![스크린샷 2024-08-24 오전 12 11 50](https://github.com/user-attachments/assets/7135763e-01c1-4b73-82d0-b83f0a7ab0e1)

어찌보면 전략 패턴과 결이 비슷하지만 차이점을 구분해보자면 다음과 같다.

1. **상속 vs. 구성**

• 템플릿 메서드 패턴은 **상속**을 사용하여 알고리즘의 기본 구조를 정의하며 상위 클래스(추상 클래스)에서 알고리즘의 기본 흐름을 정의하고, 하위 클래스에서 일부 단계들을 구체적으로 구현한다. 즉, 알고리즘의 공통 부분은 상위 클래스에 위치하며, 변하는 부분은 하위 클래스에 위치하는 것이다.

• 반면 **전략 패턴은 **구성(composition)**을 사용하여 알고리즘을 캡슐화한다. 클라이언트는 인터페이스(전략 인터페이스)를 통해 다양한 전략(구체적인 알고리즘)을 사용할 수 있으며, 다양한 행동(전략)을 독립된 클래스로 분리하고, 클라이언트 객체가 런타임에 이 행동을 선택하거나 변경할 수 있도록 한다.

2. **알고리즘의 고정된 틀 vs. 교체 가능한 알고리즘**

• **템플릿 메서드 패턴**은 알고리즘의 전체 구조(고정된 틀)는 변경되지 않으며, 알고리즘의 일부 단계만 하위 클래스에서 오버라이드할 수 있다. 즉, 알고리즘의 주요 흐름은 고정되어 있고, 알고리즘의 일부 단계만 변경 가능하기 때문에 유연성이 떨어질 수 있다.

• **전략 패턴**은 전체 알고리즘(행동)을 쉽게 교체할 수 있다. 알고리즘의 각 버전이 독립된 클래스로 정의되기 때문에, 클라이언트는 쉽게 하나의 전략에서 다른 전략으로 전환할 수 있다. 이를 통해 알고리즘 자체를 완전히 변경할 수 있는 유연성을 제공한다.

3. **패턴 적용 시점**

• **템플릿 메서드 패턴**은 컴파일 시점에 상속을 통해 알고리즘의 틀이 결정되므로, 런타임에 변경할 수 없다. 알고리즘의 특정 단계에 대해서만 변경 가능하며, 이러한 변경은 컴파일 시점에 결정된다.

• **전략 패턴**: 런타임에 전략을 변경할 수 있다. 클라이언트는 인터페이스를 통해 전략을 설정하고, 필요에 따라 다른 전략으로 교체할 수 있다. 이로 인해 런타임에 행동을 유연하게 변경할 수 있다.

4. **의존성 방향**

• **템플릿 메서드 패턴**: 상위 클래스가 하위 클래스에서 구현된 메서드에 의존한다. 즉, 추상 클래스에서 정의된 템플릿 메서드가 호출될 때, 하위 클래스에서 구현된 메서드가 실행된다.

• **전략 패턴**: 클라이언트가 전략 객체에 의존한다. 전략 객체는 독립적으로 구현되며, 클라이언트는 전략 인터페이스를 통해 원하는 전략을 호출한다. 전략은 클라이언트가 사용될 때까지 독립적으로 존재하게 된다.

#### 옵저버
옵저버 패턴(Observer Pattern)은 객체의 상태 변화를 관찰하고, 그 상태 변화가 발생할 때마다 자동으로 다른 객체들에게 통보하는 디자인 패턴이다. 주로 이벤트 기반의 시스템에서 사용되며, 데이터 변경이 발생했을 때 여러 개의 객체에 통보해야 하는 상황에서 유용한 방식이다.

옵저버 패턴은 **발행-구독(Publish-Subscribe) 모델**의 기본적인 형태를 제공하며, 이 패턴에서 객체는 두 가지 역할을 가진다.

• **Subject(주제)**: 상태를 갖고 있으며, 그 상태가 변경될 때 옵저버들에게 알리며,
• **Observer(옵저버)**: Subject의 상태 변경을 감지하고 반응한다.

![스크린샷 2024-08-24 오전 12 12 28](https://github.com/user-attachments/assets/2b0dbe84-eadf-42cf-b0b8-63ddd417b941)

#### 의무 체인
의무 체인은 요청을 처리할 수 있는 기회가 여러 객체에 분산되어 있을 때 유용한 행동 디자인 패턴이다.

이 패턴에서는 요청을 처리할 수 있는 여러 객체가 체인으로 연결되어 있으며, 각 객체는 요청을 처리하거나 다음 객체로 요청을 전달할 수 있다. 이를 통해 요청을 보내는 객체와 요청을 처리하는 객체 간의 결합을 피할 수 있다.

**의무 체인 패턴의 구성 요소**는 크게 다음과 같이 구성된다.

1. **Handler (처리자)**: 요청을 처리하거나 다음 처리자로 전달하는 인터페이스 또는 추상 클래스로, 다음 처리자를 참조할 수 있는 링크를 포함한다.

2. **ConcreteHandler (구체적인 처리자)**: 요청을 처리하거나 다음 처리자로 전달하는 구체적인 구현 클래스이다.

1. **Client (클라이언트)**: 요청을 시작하고, 체인의 첫 번째 처리자에게 요청을 전달한다.

동작 과정을 단순하게 표현하면 다음과 같다.

1. **클라이언트**가 요청을 처리해야 할 때, 체인의 첫 번째 **Handler**에 요청을 보내고

2. 각 **Handler**는 요청을 처리할 수 있는지 확인

3. 요청을 처리할 수 있으면 **Handler**는 요청을 처리하고 종료

4. 요청을 처리할 수 없다면, **Handler**는 요청을 다음 **Handler**로 전달

5. 이 과정은 체인의 끝에 도달하거나 요청이 처리될 때까지 반복


![스크린샷 2024-08-24 오전 12 12 53](https://github.com/user-attachments/assets/953b73aa-366a-4519-ba62-e84a9407b56d)

ManagerHandler, DirectorHandler, CEOHandler 클래스는 각각 요청을 처리할 수 있는 구체적인 조건을 정의하며, 요청이 자신이 처리할 수 있는 수준이 아니면, 다음 처리자에게 전달하게 된다.

#### 팩토리
팩토리 디자인 패턴(Factory Design Pattern)은 객체 생성 로직을 캡슐화하여 클라이언트가 구체적인 클래스 이름을 직접 사용하지 않고도 객체를 생성할 수 있게 하는 생성 디자인 패턴이다. 객체 생성의 책임을 팩토리 클래스에 위임함으로써 객체 생성을 단순화하고, 코드의 확장성을 높이는 데 중점을 둔다.

![스크린샷 2024-08-24 오전 12 13 13](https://github.com/user-attachments/assets/6471d511-175f-409e-a421-7138def6599b)

객체를 생성하기 위해 인터페이스를 정의하지만, 어떤 클래스의 인스턴스를 생성할지에 대한 결정은 서브클래스가 내리도록 한다.

이로써 객체 생성 로직이 캡슐화되어 클라이언트가 직접 객체를 생성하는 데 필요한 세부 사항을 알 필요가 없으며, 객체 생성을 전담하는 팩토리 클래스가 있어서 객체 생성에 관한 책임을 한 곳에 집중시키는 단일 책임의 원칙을 지킬 수 있다.
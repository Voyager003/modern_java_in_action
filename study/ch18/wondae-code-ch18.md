## 함수형 관점으로 생각하기
이 장에서는 함수형 프로그래밍이란 무엇인지 설명한 다음에 관련 용어를 설명할 것이다. 그리고 부작용, 불변성, 선언형 프로그래밍, 참조 투명성, 자바 8에서 제공하는 기능 등 함수형 프로그래밍의 개념을 살펴본다

### 18.1 시스템 구현과 유지보수
**시스템 업그레이드 관리 시 고려사항**
* synchronized 키워드 존재 여부 확인
* 자바 8 스트림 사용 시 고려사항
  * 상태 없는 동작이어야 함
  * 변수값 변경이 없어야 함

**프로그램 구조 평가 도구**
* 결합성(coupling): 시스템 부분 간 상호 의존성
* 응집성(cohesion): 시스템 부분 간 관계

#### 18.1.1 공유된 가변 데이터 문제점
* 여러 메서드에서 데이터 구조 공유로 인한 문제:
  * 데이터 소유권 불명확
  * 갱신 사실 공유의 어려움
  * 안전한 사용 방법 고민
  * 프로그램 이해도 저하
  * 디버깅 어려움 증가

#### 18.1.2 선언형 프로그래밍

**프로그래밍 구현 방식**
1. 명령형 프로그래밍
   * 작업 수행 방법에 집중
   * 일련의 명령 실행
   * 고전적 객체지향 프로그래밍 방식

```java
Transaction mostExpensive = transactions.get(0);

if(mostExpensive == null)
    throw new IllegalArgumentException("Empty list of transactions");

for(Transaction t: transactions.subList(1, transactions.size())) {
    if(t.getValue() > mostExpensive.getValue()) {
        mostExpensive = t;
    }
}
```

2. 선언형 프로그래밍 ('무엇을')
   * 목표와 달성 규칙에 집중
   * 문제가 코드로 명확하게 드러남

#### 18.1.3 왜 함수형 프로그래밍인가?
* 특징
  * 선언형 프로그래밍 방식
  * 부작용 없는 계산 지향
  * 시스템 구현과 유지보수 용이
  * 람다 표현식을 통한 작업 조합
  * 스트림을 통한 복잡한 질의 표현

* 활용
  * 부작용 없는 복잡한 기능 구현 가능
  * 자바 8의 새로운 기능과 연계

### 18.2 함수형 프로그래밍이란 무엇인가?

#### 기본 개념
* 수학적 함수와 동일한 개념
* 특징
  * 0개 이상의 인수
  * 1개 이상의 결과 반환
  * 부작용 없음
  * 동일 입력에 대해 항상 동일 출력

#### 18.2.1 함수형 자바

**제약사항**
* 순수 함수형 프로그래밍 구현의 어려움
* 함수/메서드의 조건
  * 지역 변수만 변경 가능
  * 참조 객체는 불변 객체여야 함
  * 모든 필드는 final
  * 예외 발생 지양

**예외 처리 방안**
* Optional<T> 사용
* 예시:
  ```java
  Optional<Double> sqrt(double x)
  ```

### 18.2.2 참조 투명성

**특징**
* 동일 인수로 호출 시 항상 같은 결과 반환
* 예시:
  * 투명: String.replace
  *  비투명: Random.nextInt, Scanner.nextLine

**장점**
* 프로그램 이해도 향상
* 기억화(memorization)와 캐싱 최적화 가능

### 18.2.3 객체지향 프로그래밍과 함수형 프로그래밍
**비교**
* 익스트림 객체지향: 모든 것을 객체로 취급
* 함수형: 참조 투명성 중시, 변화 제한
* 실제 자바: 두 방식의 혼합 사용

### 18.2.4 함수형 실전 연습

#### 부분집합 생성 예제
```java
static List<List<Integer>> subsets(List<Integer> list) {
    if (list.isEmpty()) {
        List<List<Integer>> ans = new ArrayList<>();
        ans.add(Collections.emptyList());
        return ans;
    }
    Integer first = list.get(0);
    List<Integer> rest = list.subList(1,list.size());

    List<List<Integer>> subans = subsets(rest);
    List<List<Integer>> subans2 = insertAll(first, subans);
    return concat(subans, subans2);
}
```

#### 보조 메서드 구현
```java
static List<List<Integer>> insertAll(Integer first, List<List<Integer>> lists) {
    List<List<Integer>> result = new ArrayList<>();
    for (List<Integer> list : lists) {
        List<Integer> copyList = new ArrayList<>();
        copyList.add(first);
        copyList.addAll(list);
        result.add(copyList);
    }
    return result;
}

static List<List<Integer>> concat(List<List<Integer>> a, List<List<Integer>> b) {
    List<List<Integer>> r = new ArrayList<>(a);
    r.addAll(b);
    return r;
}
```
#### 함수형 프로그래밍 접근법
* 무엇(what)을 해야 하는가에 집중
* 인수에 의한 출력 결정
* 입력값 불변성 유지
* 순수 함수 지향
### 18.3 재귀와 반복
#### 순수 함수형 프로그래밍의 특징
* while, for 같은 반복문 미포함
* 이유: 변경이 코드에 자연스럽게 스며들 수 있음

### 반복문의 문제점
#### 부작용이 있는 검색 예제
```java
public void searchForGold(List<String> l, Stats stats) {
    for(String s: l) {
        if("gold".equals(s)) {
            stats.incrementFor("gold");
        }
    }
}
```
* 루프 바디에서 함수형과 충돌 -> 부작용 발생
### 재귀와 반복 비교

#### 반복 방식의 팩토리얼
```java
static int factorialIterative(int n) {
    int r = 1;
    for (int i = 1; i <= n; i++) {
        r *= i;
    }
    return r;
}
```

#### 재귀 방식의 팩토리얼
```java
static long factorialRecursive(long n) {
    return n == 1 ? 1 : n * factorialRecursive(n-1);
}
```

#### 스트림 방식의 팩토리얼
```java
static long factorialStreams(long n) {
    return LongStream.rangeClosed(1, n)
        .reduce(1, (long a, long b) -> a * b);
}
```

### 꼬리 재귀 최적화

#### 꼬리 재귀 팩토리얼 구현
```java
static long factorialTailRecursive(long n) {
    return factorialHelper(1, n);
}

static long factorialHelper(long acc, long n) {
    return n == 1 ? acc : factorialHelper(acc * n, n-1);
}
```

#### 특징
* 일반 재귀: 각 호출마다 새로운 스택 프레임 생성
* 꼬리 재귀: 단일 스택 프레임 재사용 가능
* 자바의 한계: 꼬리 재귀 최적화 미지원
* 다른 JVM 언어(스칼라, 그루비): 재귀를 반복으로 변환하는 최적화 제공
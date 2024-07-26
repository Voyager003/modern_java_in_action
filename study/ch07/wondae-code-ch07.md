## 7. 병렬 데이터 처리와 성능
### 7.1 병렬 스트림
컬렉션에 parallelStream을 호출하면 병렬스트림을 생성할 수 있다. 병렬 스트림이란, 각 스레드에서 처리할 수 있도록 스트림 요소를 여러 단위로 분할한 스트림이다.

예시로 1부터 n까지 합을 반환하는 메서드를 병렬 스트림으로 바꿔보자.
```Java
public long sequentialSun(long n){
    return Stream.iterate(1L, i -> i+1)
                .limit(n)
                .reduce(0L, Long::sum);
}
```

#### 7.1.1 순차 스트림을 병렬 스트림으로 변환하기
순차 스트림에 pararell()을 호출하면 기존의 reduce 연산이 병렬로 처리된다.
```Java
public long sequentialSun(long n){
    return Stream.iterate(1L, i -> i+1)
                .limit(n)
                .parallel()
                .reduce(0L, Long::sum);
}
```
이렇게 하면, 분할된 스트림에서 나온 결과를 다시 리듀싱으로 합쳐서 결과를 도출할 수 있다.

사실 순차 스트림에 parallel을 호출하더라도 스트림 자체에는 아무 변화가 일어나지 않고, 단순히 병렬 연산인지 체크하는 플래그가 true로 설정된다. 만약 sequential()을 호출하면 플래그가 false 되어 순차적으로 연산한다.

> **병렬 스트림에서 사용하는 스레드 풀 설정**
> 
> 병렬 스트림은 내부적으로 `ForkJoinPool`을 사용한다. 기본적으로 `ForkJoinPool`은 프로세서 수에 알맞는 스레드를 갖는다.
> 
> ```Java
> int processorCount = Runtime.getRuntime().availableProcessors();
> // m1 processor에선 8
> System.out.println(processorCount);
> // 전역적으로 스레드 수 조절 가능
> System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", 12);
> ```

#### 7.1.2 스트림 성능 측정
반복형, 순차 스트림, 병렬 스트림을 JHM(Java Microbenchmark Harness)란 라이브러리를 사용하여 측정할 수 있다. 가비지 컬렉터, 바이트 코드 최적화 준비등 부차적인 시간을 최소화 하여 측정한 결과는 다음과 같다.
* 순차 스트림 : 61.057
* 반복문 : 3.404
* 병렬 스트림 : 229.767
* 최적화 된 병렬 스트림 : 0.665

#### 7.1.3 병렬 스트림의 올바른 사용법
병렬 스트림을 사용할 떄, 공유된 상태를 바꾸는 알고리즘을 사용해선 안된다. 만약 사용할 경우, 요소에 접근할 때 레이스 컨디션 문제가 생겨 동기화가 꼭 필요하게 되고, 그렇다면 병렬화를 하는 이유가 사라질 것이다.

#### 7.1.4 병렬 스트림 효과적으로 사용하기
1. **측정해서 확인하자** : 순차 스트림을 병렬스트림으로 쉽게 변경할 수 있는 만큼, 직접 벤치마크하여 측정해보고 결정하는 것이 좋다.
2. **박싱을 주의하라** : 자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있다. 따라서 기본형일 경우 특화 스트림인 IntStream, LongStream, DoubleStream을 사용하자.
3. **순차 스트림보다 병렬 스트림이 느린 연산도 있다** : Limit()나 findFirst()와 같은 메서드는 비용이 많이 든다. findFirst()의 경우, findAny()가 훨신 낫다.
4. **스트림에서 수행하는 연산 비용을 고려하라** : 요소 수가 N이고, 하나를 처리하는 비용이 Q라면, 전체 처리 비용은 N * Q라 할 수 있다. Q가 커질 수록 병렬 스트림을 통한 성능 개선 가능성이 높아진다.
5. **데이터의 크기 또한 중요하다** : 소량의 데이터에선 병렬 스트림이 이득을 보기 어렵다. 병렬화 과정에서 생기는 부가 비용을 상쇄할 만큼 이득을 얻지 못하기 때문이다.
6. **스트림을 구성하는 자료구조가 적절한지 확인하라** : ArrayList는 LinkedList보다 효율적으로 나눌 수 있다. 또한 range로 만든 기본형 스트림도 쉽게 분해할 수 있다. 또한 커스텀 Spliterator를 사용해 분해 과정을 완벽하게 제어할 수 있다.
7. **스트림의 특성과 중간 연산을 통한 스트림의 특성을 파악하라** : 크기가 정해진 스트림은 같은 크기의 두개의 스트림으로 분할 가능할 수 있다. 하지만 filter를 사용한다면 길이를 예츨할 수 없으므로 효과적으로 스트림을 병렬처리할 수 있을지 알 수 없다.
8. **최종 연산의 병합 과정 비용을 살펴보라** : 병합 과정의 비용이 비싸다면 병렬 스트림으로 얻은 이득이 서브 스트림의 결과를 합치는 과정에서 상쇄될 수 있다.

### 7.2 포크/조인 프레임워크
포크/조인 프레임워크에선 서브태스크를 스레드 풀의 작업자 스레드에 분산 할당하는 ExecutorService 인터페이스를 구현한다.
#### 7.2.1 RecursiveTask 활용
스레드 풀을 사용하기 위해 RecursiveTask<R>의 서브 클래스를 만들어야 한다. R은 병렬화된 태스크가 생성하는 결과 형식 또는 RecursiveAction 형식이다. 이를 구현하기 위해선 `protected abstract R compute()`를 구현해야 한다. `compute()`는 태스크를 서브태스크로 분할하는 로직과 분할 할 수 없을 때 개별 서브태스크의 결과를 생산할 알고리즘이다. 대부분의 구현은 아래 형태를 띈다.
```
if(더 이상 분해할 수 없을 때){
    순차적 계산
} else {
    태스크 분할
    재귀 호출
    모든 연산이 끝날 때까지 기다림
    결과를 합침
}
```
이는 Devide & Conquer 알고리즘의 병렬화 버전이다.

**예시**
```Java
public class ForkJoinSumCalculator extends RecursiveTask<Long> {
    private final long[] numbers;
    private final int start;
    private final int end;
    public static final long THRESHOLD = 10_000;
    
    public ForkJoinSumCalculator(long[] numbers){
        this(numbers, 0, numbers.length);
    }
    
    private ForkJoinSumCalculator(long[] numbers, int start, int end){
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }
    
    @Override
    protected Long compute() {
        int length = end - start;
        if(length < THRESHOLD){
            return computeSequentially();
        }
        
        ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length / 2);
        leftTask.fork();
        
        ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length / 2, end);
        Long rightResult = rightTask.compute();
        Long leftResult = leftTask.join();
        return leftResult + rightResult;
    }
    
    private long computeSequentially(){
        long sum = 0L;

        for (int i = start; i < end; i++) {
            sum += numbers[i];
        }
        return sum;
    }
}
```

위 메서드를 다음과 같이 사용할 수 있다.

```Java
public static long forkJoinSum(long n){
    long[] numbers = LongStream.rangeClosed(1, n).toArray();
    ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);

    return new ForkJoinPool().invoke(task);
}
```
invoke 메서드의 결과는 ForkJoinSumCalculator에서 정의한 태스크의 결과가 된다.

일반적으로 애플리케이션에선 둘 이상의 ForkJoinPool을 사용하지 않는다. 따라서 어플리케이션 어디서든 사용할 수 있도록 싱글턴으로 한번 인스턴스화 해서 사용한다.

이 결과 41msec가 나온다.
(로컬에서 돌리려 했으나 램이 모자라 안된다고 뜸)

#### 7.2.2 포크/조인 프레임워크를 제대로 사용하는 방법
1. join 메서드를 태스크에 호출하면 태크스가 생산하는 결과가 준비될때까지 호출자를 블락시킨다. 따라서 두 서브태스크가 모두 시작된 다음에 join을 호출해야한다.
2. RecursiveTask 내에선 ForkJoinPool의 invoke메서드를 사용하지 말아야 한다. 대신 compute나 fork 메서드를 직접 호출할 수 있다. 병렬 계산을 시작할 때만 invoke를 호출한다.
3. 서브태스크에 fork 메서드를 호출하여 ForkJoinPool의 일정을 조절할 수 있다. 양쪽에 fork를 호출하기 보단 한쪽에 compute를 호출하는게 효율적이다.
4. 포크/조인 프레임워크를 이용하는 병렬 계산을 디버깅이 어렵다.
5. 병렬 스트림과 마찬가지로, 멀티코어에 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 무조건 빠르진 않다. 애매할 때는 측정 후 사용하자.

#### 7.2.3 작업 훔치기
적절한 크기로 분할된 많은 태스크를 포킹하는 것이 좋다. 이상적으론 코어 개수만큼 병렬화된 태스크로 분할하면 같은 시간에 종료되고 좋을 것 같지만, 복잡한 시나리오에선 쉽지 않은 일이다.

포크/조인 프레임워크에선 작업 훔치기라는 기법으로 이 문제를 해결한다. 작업 훔치기 기법에선 ForkJoinPool의 모든 스레드를 거의 공정하게 분할한다. 각 스레드들은 자신에게 할당된 태스크를 포함하는 이중 연결 리스트를 참조하며 작업이 끝날 때마다 큐에서 다른 태스크를 가져와서 작업을 처리한다. 만약 스레드가 할 일이 없어지면 다른 스레드의 꼬리에서 일을 가져와서 처리한다.

### 7.3 Spliterator 인터페이스
자바 8부터 제공하는 Spliterator 인터페이스는 병렬 작업에서 소스의 요소 탐색 기능을 제공한다. Spliterator 인터페이스는 여러 메서드를 정의한다.
```Java
public interface Spliterator<T>{
    boolean tryAdvance(Consumer<? super T> action);
    Spliterator<T> trySplit();
    long estimateSize();
    int characteristics();
}
```
T는 Spliterator에서 탐색하는 요소의 형식이다. tryAdvance 메서드는 Spliterator의 요소를 하나씩 순차적으로 소비하면서 탐색해야할 요소가 남아있으면 참을 반환한다. trySplit 메서드는 Spliterator의 일부 요소를 분할 해서 다른 Spliterator를 만드는 메서드이다. estimateSize 메서드를 통해 탐색해야 할 요소 수 정보를 알 수 있다.

#### 7.3.1 분할과정
스트림을 여러 스트림으로 분할하는 과정을 재귀적으로 일어난다.
1개 -> 2개 -> 4개로 trySplit()의 결과가 null이 될때까지 반복한다.

**Spliterator 특성**
Spliterator의 characteristic() 메서드는 자체 특성 집합을 포함하는 정수를 반환한다. 이를 통해 프로그램은 Spliterator를 더 잘 제어하고 최적화 할 수 있다.
* ORDERED : 리스트처럼 요소에 정해진 요소가 있다.
* DISTINCT : x, y 두 요소를 방문했을 때 x.equals(y)는 언제나 false다. 즉, 중복 값이 없다.
* SORTED : 미리 정렬된 순서가 있다.
* SIZED : 크기가 정해진 소스이다.
* NON-NULL : 모든 요소가 null이 아니다.
* IMMUTABLE : 요소를 추가하거나 삭제하거나 수정할 수 없다.
* CONCURRENT : 소스를 병렬적으로 수정할 수 있다.
* SUBSIZED : Spliterator와 분할되는 Spliterator 모두 SIZED 속성을 갖는다.

#### 7.3.2 커스텀 Spliterator 구현
Spliterator를 구현하는 예제를 만들어보자. 다음은 반복형으로 만든 예제이다
```java
public int countWordsIteratively(String s){
    int counter = 0;
    boolean lastSpace = true;
    for(char c : s.toCharArray()){
        if(Character.isWhitespace(c)){
            lastSpace = true;
        }else{
            if(lastSpace) counter++;
            lastSpace = flase;
        }
    }

    return counter;
}
```

함수형을 사용하면 직접 스레드를 동기화 하지 않고도 작업을 병렬화할 수 있다.

아래는 문자열을 스트림으로 변환하는 코드이다.
```Java
Stream<Character> stream = IntStream.range(0, SENTENCE.length()).mapToObj(SENTENCE::charAt);
```

단어 수와 마지막 문자가 공백인지 체크하기 위해 래핑 클래스를 만들어야한다.
```Java
class WordCounter {
    private final int counter;
    private final boolean lastSpace;
    public WordCounter(int counter, boolean lastSpace){
        this.counter = counter;
        this.lastSpace = lastSpace;
    }

    public WordCounter accumulate(Character c){
        if(Character.isWhiteSpace(c)){
            return lastSpace ?
                this :
                new WordCounter(counter, true);
            
        } else {
            return lastSpace ?
                new WordCounter(counter+1, false) :
                this;
        }
    }

    public WordCounter combine(WordCounter wordCounter){
        return new WordCounter(counter + wordCounter.counter, wordCounter.lastSpace);
    }

    public int getCounter(){
        return counter;
    }
}
```
accumulate 메서드는 불변인 WordCounter 클래스 대신 새로운 WordCounter를 어떻게 만들어야 할 지 체크한다.

여기에 리듀싱 연산을 직관적으로 구현할 수 있다.
```Java
private int countWords(Stream<Character> Stream) {
    WordCounter wordCounter = stream.reduce(
                        new WordCounter(0, true),
                        WordCounter::accumulate,
                        WordCounter::combine);

    return wordCounter.getCounter();
}
```

이를 병렬로 수행하면 다음과 같이 처리할 수 있다. 여기엔 문자열을 단어가 끝날 떄 분리하는 custom Spliterator가 필요하다.

```java
class WordCounterSpliterator implements Spliterator<Character> {
    private final String string;
    private int currentChar = 0;

    public WordCounterSpliterator(String string){
        this.string = string;
    }

    @Override
    public boolean tryAdvance(Consumer<? super Character> action) {
        action.accept(String.charAt(currentChar++));
        return currentChar < string.length();
    }

    @Override
    public Spliterator<Character> trySplit(){
        int currentSize = string.length() - currentChar;
        if(currentSize < 10){
            return null;
        }

        for(int splitPos = currentSize / 2 + currentChar ; splitPos < string.length(); splitPos++){
            if(Character.isWhitespace(string.charAt(splitPos))){
                Spliterator<Character> spliterator =
                    new WordCounterSpliterator(string.substring(currentChar, splitPos));
                currentChar = splitPos;
                return spliterator;
            }
        }

        return null;
    }

    @Override
    public long estimateSize() {
        return string.length() - currentChar;
    }

    @Override
    public int characteristics(){
        return ORDERED + SIZED + SUBSIZED + NON-NULL + IMMUTABLE;
    }
}
```

이제 WordCounterSpliterator를 통해 병렬 스트림을 사용할 수 있다.
```Java
Spliterator<Character> spliterator = new WordCounterSpliterator(SENTENCE);

Stream<Character> stream = StreamSupport.stream(spliterator, true);
```
StreamSupport.stream의 두번쨰 매개변수는 병렬 스트림로 만들지 판단하는 플래그이다.

## 추가 학습

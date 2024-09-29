## 새로운 날짜와 시간 API
자바 8에선 기존의 시간 및 날짜 API를 개선한 새로운 API를 제공한다.

자바 1.0에선 `java.util.date`를 통해 날짜와 시간 기능을 제공했는데, 1900년부터 시작하는 연도, 0부터 시작하는 달 등 모호한 설계가 많았다.
```Java
// 2017년 9월 21일을 표기하기 위한 코드
Date date = new Date(117, 8, 21);
```

이러한 문제를 해결하기 위해 `java.util.Calendar`를 추가하였지만, 이는 개발자들에게 혼란스러움을 추가할 뿐이었다.

### 12.1 LocalDate, LocalTIme, Instant, Duration, Peroid 클래스
LocalDate는 시간을 제외한 날짜를 표시하는 불변 객체이다. 시간대 정보를 포함하지 않는다.

정적 팩토리 메서드 of로 인스턴스를 만들 수 있다.
```Java
// 2017년 9월 21일을 표기하기 위한 코드
LocalDate date = LocalDate.of(2017, 9, 21);
```
팩토리 메서드 now를 통해서 현재 날짜 정보를 얻을 수 있다. 또한 get 메서드에 TemporalField를 전달해서 정보를 얻는 방법도 있다. TemporalField를 구현한 ChronoField을 통해 정보를 얻을 수 있다.
```Java
int year = date.get(ChronoField.YEAR);
int month = date.get(ChronoField.MONTH_OF_YEAR);
int day = date.get(ChronoField.DAY_OF_YEAR);
```

LocalTime 역시 LocalDate처럼 사용 가능하다.
```Java
// 13시 45분 20초
LocalTime time = LocalTime.of(13, 45, 20);
```

날짜와 시간 문자열로 LocalDate와 LocalTime의 인스턴스를 만들 수도 있따.
```Java
LocalDate date = LocalDate.parse("2017-07-21");
```

#### 12.1.1 날짜와 시간 조합
LocalDateTime은 LocalDate와 LocalTime을 조합한 클래스이다.

LocalDate.atTime() 혹은 LocalTime.atDate()을 통해 만들 수 있으며, 직접 값을 넣음으로서 만들 수도 있다.
```Java
// 2017년 9월 21일 13시 45분 20초
LocalDateTime dt1 = LocalDateTime.of(2017, 9, 21, 13, 45, 20);

LocalDateTime dt2 = date.atTime(time);
LocalDateTime dt3 = time.atDate(date);
```

#### 12.1.2 Instant 클래스 : 기계의 날짜와 시간
LocalDate와 LocalTime이 인간의 관점에서 시간을 표현하였다면, Instant는 기계의 관점에서 시간을 표시한다. 유닉스 에포크 시간인 1970년 1월 1일 0시 0분 0초를 기준으로 특정 시점까지의 시간을 초로 표시한다.

ofEpochSecond 메서드에 값을 넘겨줘서 인스턴스를 만들 수 있다.
```Java
// 아래 4개 메서드는 같은 인스턴스를 반환한다.
Instant.ofEpochSecond(3);
Instant.ofEpochSecond(3, 0);
Instant.ofEpochSecond(2, 1_000_000_000);
Instant.ofEpochSecond(4, -1_000_000_000);
```

#### 12.1.3 Duration과 Period 정의
앞의 모든 클래스들은 모두 Temporal 인터페이스를 구현한 것이다. 두 시간 객체 사이의 시간은 Duration 클래스의 between 메서드로 표현할 수 있다.
```Java
Duration d1 = Duration.between(time1, time2);
```

LocalDateTime와 Instant를 같이 사용할 수 없다. LocalDateTime의 간격을 비교할 때는 Period 클래스를 사용한다.
```Java
Period p1 = Period.between(time1, time2);
```

### 12.2 날짜 조정, 파싱, 포매팅
withAttribute 메서드로 LocalDateTime을 바꾼 버전을 보여줄 수 있다.
```Java
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017년 9월 21일
LocalDate date2 = date1.withYear(2012); // 2012년 9월 21일
```

선언형으로도 LocalDate를 사용하는 방법이 있다. 
```Java
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017년 9월 21일
LocalDate date2 = LocalDate.plusYears(6); // 2023년 9월 21일
```

#### 12.2.1 TemporalAdjusters 사용하기
다음주 일요일, 어떤 달의 마지막 날등 복잡한 날짜 조정이 필요할 때는 TemporalAdjusters를 사용한다.
```Java
LocalDate date1 = LocalDate.of(2014, 3, 18);
// 2014년 03년 23일
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); 
```

추가적으로 필요한 연산이 있을 경웅 TemporalAdjusters 인터페이스를 구현하여 사용할 수 있다. 메서드가 하나이기 때문에, 함수형 인터페이스이다.
```Java
@FunctionalInterface
public interface TemporalAdjuster {
    Temporal adjustInto(Temporal temporal);
}
```

#### 12.2.2 날짜와 시간 객체 출력과 파싱
DateTimeFormatter를 통해 쉽게 포매터를 만들 수 있다. BASIC_ISO_DATE나 ISO_LOCAL_DATE를 통해 출력할 수 있다.
```Java
LocalDate date = LocalDate.of(2017, 7, 21);
// 20170721
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE);
// 2017-07-21
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE);
```
DateTimeFormatter는 특정 패턴으로 포매터를 만들 수 있는 정적 팩토리 메서드도 제공한다.
```Java
LocalDateFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");

LocalDate date = LocalDate.of(2014, 3, 21);

// 21/03/2014
String s = date.format(formatter);
```
DateTimeFormatterBuilder 클래스로 복합적인 포매터를 정의하여 좀 더 세부적으로 제어할 수 있다.
```Java
DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
                .appendText(ChronoField.DAY_OF_MONTH)
                .appendLiteral(". ")
                .appendText(Chorofield.MONTH_OF_YEAR)
                .appendLiteral(". ")
                .appendText(ChronoField.YEAR)
                .parseCaseInsentive()
                .toFormatter(Locale.ITALIAN);
```

### 12.3 다양한 시간대와 캘린더 활용법
시간대를 다루기 위해선 `java.time.zoneId` 클래스를 사용하면 된다.
#### 12.3.1 시간대 사용하기
표준 시간이 같은 지역을 묶어서 시간대 규칙 집합을 정의한다. ZoneRules 클래스에선 약 40개의 시간대가 있는데, 지역 ID로 zoneID를 구분한다.
```Java
ZondId romeZone = ZoneId.of("Europe/Rome");
```
toZoneId 메서드를 이용하면 TimeZone 객체를 ZoneId로 변환할 수 있다.
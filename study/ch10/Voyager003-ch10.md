## 람다를 이용한 도메인 전용 언어
### DSL(Domain Specific Language)
`DSL`은 Domain Specific Language의 약자로 '특정 도메인'에 국한하여 사용하는 언어이다. 반대 개념으로는 General Purpose Language가 있으며 우리가 일반적으로 사용하는 C, C++, Kotlin, Swift 등의 프로그래밍 언어들이 이에 해당한다.

개발자로서 프레임워크나 라이브러리들을 사용하면서 제공되는 DSL을 사용하게 되는데, 이러한 DSL들은 General Purpose Language가 아니기 때문에 사용법을 찾아보기 위해서는 특정 언어의 명세가 아니라 해당 DSL을 정의한 프레임워크나 라이브러리의 가이드에서 찾아야 한다.

예를 들어 Kotlin으로 안드로이드를 개발한다면 이러한 DSL들을 접할 수 있다. 빌드 스크립트에서 안드로이드 관련 옵션을 명시하는 것이다.

```
android {
    compileOptions {
        sourceCompatibility(JavaVersion.VERSION_11)
        targetCompatibility(JavaVersion.VERSION_11)
    }
    kotlinOptions {
        jvmTarget = "11"
    }
    buildFeatures {
        viewBinding = true
    }
    hilt {
        enableExperimentalClasspathAggregation = true
    }
}
```

살펴보면 빌더나 팩토리를 이용하여 필요한 옵션을 명시하고 객체를 생성하는 구현 패턴과 비교하여 더 간략하고 읽기 쉽게 작성되어 있다는 것을 알 수 있다.

이처럼 라이브러리 개발 시 DSL을 적절히 활용하여 제공하면 라이브러리 사용자는 보다 쉽고 간결하게 호출 코드들 작성할 수 있고, 잘 정의된 DSL은 상대적으로 자연어에 가까워 가독성도 높일 수 있게된다.
####  DSL in Java
##### Stream API
JDK 8에 추가된 Stream API는 네이티브 자바 API에 작은 내부 DSL을 적용한 좋은 예이다. Java 컬렉션의 항목을 필터, 정렬, 변환, 그룹화하는 기능을 지원한다.

```java
List<String> errors = Files.lines(Paths.get(fileName))
			.filter(line -> line.startWith("ERROR"))
			.limit(40)
			.collect(toList());
```
로그 파일의 에러 행을 읽어들이는 함수형으로 작성된 코드이다. 스트림의 API 특성인 메서드 체인을 통해 가독성을 높이면서 간결한 스타일이다. (fluent style)

##### Collectors
Collector 인터페이스는 데이터 수집을 수행하는 DSL로 간주할 수 있다.

```java
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> c arGroupingCollector = 
	groupingBy(Car::getBrand, groupingBy(Car::getColor));
```

Collectros를 중첩하여 작성한 다중 수준 Collector이다.

#### Jooq 다뤄보기

![SCR-20240830-ufnw](https://github.com/user-attachments/assets/ec910e13-efc0-492c-834a-0231a0e2bb8f)


먼저 의존성에 jooq를 추가한다.

```java
create table AUTHOR
(
    ID         integer PRIMARY KEY,
    FIRST_NAME varchar(255),
    LAST_NAME  varchar(255),
    AGE        integer
);

create table ARTICLE
(
    ID          integer PRIMARY KEY,
    TITLE       varchar(255) not null,
    DESCRIPTION varchar(255),
    AUTHOR_ID   integer
        CONSTRAINT fk_author_id REFERENCES AUTHOR
);
```
테스트를 위한 Author와 Article 테이블을 생성한다.

```java
String userName = "user";
String password = "pass";
String url = "jdbc:postgresql://db_host:5432/baeldung";
Connection conn = DriverManager.getConnection(url, userName, password);
```
DB 연결을 위한 속성을 작성한다. 드라이버매니저와 getConnection 메서드로 연결 개체를 만들어야 한다.

```java
DSLContext context = DSL.using(conn, SQLDialect.POSTGRES);
```
이 후, DSLContext 인스턴스를 생성한다. 이는 jooq의 인터페이스 진입점이 된다.

그리고 DB 테이블에 대한 Java 클래스를 생성하려면 다음 _jooq-config.xml_ 파일이 필요하다.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<configuration xmlns="http://www.jooq.org/xsd/jooq-codegen-3.13.0.xsd">
    
    <jdbc>
        <driver>org.postgresql.Driver</driver>
        <url>jdbc:postgresql://db_url:5432/baeldung_database</url>
        <user>username</user>
        <password>password</password>
    </jdbc>

    <generator>
        <name>org.jooq.codegen.JavaGenerator</name>

        <database>
            <name>org.jooq.meta.postgres.PostgresDatabase</name>
            <inputSchema>public</inputSchema>
            <includes>.*</includes>
            <excludes></excludes>
        </database>

        <target>
            <packageName>com.baeldung.jooq.model</packageName>
            <directory>C:/projects/baeldung/tutorials/jooq-examples/src/main/java</directory>
        </target>
    </generator>
</configuration>
```

사용자 지정 구성을 사용하려면 DB 자격 증명을 배치 하는 jdbc 섹션 과 생성할 클래스의 패키지 이름 및 위치 디렉토리를 구성하는 target섹션을 변경해야 한다.

이 후, jOOQ 코드 생성 도구를 실행하려면 다음 코드를 실행해야 한다.

```java
GenerationTool.generate( 
	Files.readString( Path.of("jooq-config.xml") 
	) 
);
```
생성이 완료되면 설정에 추가한 Article, Author 두 클래스를 얻게된다.

##### 간단한 생성 작업
Article 클래스의 레코드를 생성하도록 해보자.

```java
ArticleRecord article = context.newRecord(Article.ARTICLE);
```
Article.ARTICLE의 변수에 대한 참조 인스턴스 _문서의_ 데이터베이스 테이블. 코드 생성 중에 jOOQ에 의해 자동으로 생성되며, 필요한 모든 속성에 대한 값을 설정할 수 있다.

```java
article.setId(2);
article.setTitle("jOOQ examples");
article.setDescription("A few examples of jOOQ CRUD operations");
article.setAuthorId(1);
```

그리고 DB 저장을 위한 store 메서드를 호출하면 저장!

```java
article.store();
```

이제 DB 값을 읽어보자. 모든 Author를 선택하는 코드이다.

```java
Result<Record> authors = context.select()
	  .from(Author.AUTHOR)
	  .fetch();
```

읽고 싶은 테이블을 나타내기 위해 _from_절 과 결합된 select 메소드를 사용하고 있는데, fetch 메서드를 호출하면 SQL 쿼리가 실행되고 생성된 결과가 반환된다.



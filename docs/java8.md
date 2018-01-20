# java 8 의 새로운 기능

출처:
- http://www.moreagile.net/2014/04/AllAboutJava8.html
-

람다 표현식 - 람다는 메서드에 매개 변수와 메소드로 전달 매개 변수로 기능 (기능을 할 수 있습니다.

메서드 참조가 - 참조 방법은 매우 유용한 구문을 제공, 직접 기존의 Java 클래스 또는 개체 (예) 메서드 또는 생성자를 참조 할 수 있습니다.람다와 조합에있어서 기준 구성이 중복 코드를 줄이기 위해 더 소형이고 간결한 언어가 될 수있다.

기본 방법 - 기본 방법이 방법에서 인터페이스로 구현된다.

새로운 도구 - 새로운 컴파일러 툴과 같은 : Nashorn 엔진 jjs, 클래스에 의존 파서 jdeps.

스트림 API는 - 새로운 스트림 API (java.util.stream) 자바에 도입 된 사실 함수형 프로그래밍 스타일을 추가했습니다.

날짜 시간 API는 - 처리의 날짜와 시간을 강화한다.

옵션 클래스 - 옵션 클래스는 널 포인터 예외를 해결하는 데 사용되는 자바 8 클래스 라이브러리의 일부가되었다.

Nashorn, 자바 스크립트 엔진 - 자바 (8)은 우리가 JVM 특정 자바 스크립트 응용 프로그램에서 실행할 수있는 새로운 Nashorn 자바 스크립트 엔진을 제공합니다.



자바8의 전체 기능에 대한 상세는 Java.net가 제공하는 기능일람을 참고하기 바란다.

## 인터페이스의 개선 

인터페이스에 static 메소드를 정의하는것이 가능해졌다. 

java.util.Comparator에 추가된 static naturalOrder메소드를 살펴보자.

```java
    public static <T extends Comparable<? super T>> Comparator<T> naturalOrder() {
        return (Comparator<T>) Comparators.NaturalOrderComparator.INSTANCE;
    }
```


default 지시자를 이용해 기본 메소드의 정의가 가능하게되어 인터페이스를 구현하는 기존 코드의 변경없이 새 메소드의 추가가 가능해졌다. 예를 들어 `java.lang.Iterable`에는 `forEach`메소드가 `default`로 정의되어 있다.


```java
    public default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```

처리해야할 데이터인 Customer와 처리 내용인 action이 메소드의 인자값으로 전달 가능하게 되어 Iterable Collection에 대한 반복처리가 매우 간단히 구현 가능하게 된다.
여기서 한가지 주의해야 할 점은 예외적으로 Object클래스의 메소드에 대해서 default구현은 정의 할 수 없다는 점이다.

## 함수형 인터페이스

함수 인터페이스(functional interface) 는 단 하나의 추상 메소드가 정의 가능한 인터페이스이다.

인터페이스가 함수형 인터페이스임을 나타내는 수단으로서 `@FunctionalInterface` 어노테이션이 도입되었다.

예를 들어, java.lang.Runnable은 다음과 같은 함수 인터페이스를 지닌다.

```java
    @FunctionalInterface
    public interface Runnable {
        public abstract void run();
    }
```

하지만, 어노테이션을 통해 명시적으로 지정하지 않더라도 함수 인터페이스의 정의를 만족하는 인터페이스라면 자바 컴파일러가 주석의 유무에 상관없이 함수 인터페이스로서 취급한다.

## 람다식

함수형 인터페이스의 중요한 특성으로 람다식lambda expression을 사용한 인스턴스 생성이 있다.

람다식을 이용하면 동작과 데이터를 모두 동적으로 설정하는것이 가능해진다. 

아래의 예제들은 모두 왼쪽이 입력값이 되고, 오른쪽이 동작에 대한 정의이다. 입력값의 데이터 타입이  유추 가능하므로 생략되고 있다는 점에 주목하자.

```java
    (int x, int y) -> { return x + y; }
    (x, y) -> x + y
    x -> x * x
    () -> x
    x -> { System.out.println(x); }
```

예를들어 Runnable 함수 인터페이스를 인스턴스화 하는것은 다음과 같다.

```java
    Runnable r = () -> { System.out.println("Running!"); }
```

## 메소드 참조 [#4](#4)

메소드참조`method reference` 는 이미 이름이 있는 메서드를 대상으로 한 람다식의 간략형이며, 메소드 참조를 나타내는 예약어로서 (::)를 사용한다. 메소드 참조의 예와 그에 대응하는 람다식은 다음과 같다. 오른쪽이 메소드 참조, 왼쪽이 람다식이다.
```java
    String::valueOf     x -> String.valueOf(x)
    Object::toString    x -> x.toString()
    x::toString         () -> x.toString()
    ArrayList::new      () -> new ArrayList<>()
```

### 캡처 vs 비캡처 람다식Capturing versus non-capturing lambdas

람다식의 외부에 정의된 static이 아닌 변수나 객체에 억세스하는것을 람다가 객체를 "캡쳐"한다고 부른다. 예를 들면 다음은 람다 변수 x에 억세스하는것이다.
```java
    int x = 5;
    return y -> x + y;
```

람다식으로부터 억세스 가능한것은 로컬변수와 블록구의의 파라매터중에 final이거나 사실상 final판정(effectively final)을 받은 것에 한정된다.

`java.util.function`패키지에는 많은 새로운 함수형 인터페이스가 추가되었다. 몇가지를 예로 들자면 다음과 같다.


`Function <T, R>` - T를 입력으로 R을 출력으로 반환
`Predicate <T>` - T를 입력으로 boolean을 출력으로 반환
`Consumer <T>` - T를 입력으로 아무것도 반환하지 않는다
`Supplier <T>` - 입력을 취하지 않고 T를 반환
`BinaryOperator <T>` - 2 개의 T를 입력으로 하나의 T를 출력으로 반환

## java.util.stream
자바 8의 중요한 패러다임의 하나로 새로운 java.util.stream패키지는 스트림에 대한 함수형 조작을 제공한다. 좀더 알기쉽게 설명하자면 배열이나 리스트, 맵으로 대표되는 컬랙션을 스트림으로 다룰 수 있게 되었다는 것 이다. 다음은 컬랙션에 대한 스트림화의 예이다.

```java
   Stream <T> stream = collection.stream ();
```

이것이 함수형 프로그래밍과 결합하면 다음과 같은 형태가 된다.
```java
    int sumOfWeights = blocks.stream () filter (b -> b.getColor () == RED)
                                      . mapToInt (b -> b.getWeight ())
                                      . sum ();
```


위의 샘플코드는 stream패키지의 Javadoc에 실린 예로서, stream의 소스로서 blocks라는 Collection을 사용하고 있다. 그 스트림에 대해 filter-map-reduce를 실행하여 붉은색(RED)블록에 대한 무게(weight)의 합(sum)을 구하는 일련의 과정이 한줄의 코드에 집약되어 표현되고 있다.


## 제네릭 타입 인터페이스의 개선

이 개선은 자바 컴파일러가 형에대한 추론능력을 갖추는 것으로 제네릭 형식 메소드 호출시 인수에 대한 형 정의를 생략 가능하게 해 준다.
예를 들어 자바7의 코드가 다음과 같은 것 이었다면

```java
    foo(Utility.<Type>bar());
    Utility.<Type>foo().bar();
```    

자바8에서는 인수와 호출에 대한 추론이 자동적으로 이루어져 다음과 같이 형태가 된다.

```java
    foo(Utility.bar());
    Utility.foo().bar();
```

## java.time
새로운 날짜/시간 관련 API가 java.time 패키지에 추가되고 있다. 클래스는 immutable이며 스레드에 대해 안전하다. 날짜 및 시간 형식으로  Instant, LocalDate, LocalDateTime, ZonedDateTime이 추가되었으며 날짜와 시간 이외의 것으로서 Duration과 Period가 추가되었다. 새로 추가된 값 형식은 Month, DayOfWeek, Year, Month YearMonth, MonthDay, OffsetTime, OffsetDateTime등이 있다. 이런한 새로운 날짜/시간 클래스는 대부분이 JDBC에서 지원됨으로서 RDB연동의 효율적인 구현이 가능하다.

## Collections API의 확장
인터페이스가 default 메소드를 가질 수 있게 됨으로써 자바8의 Collection API에는 다수의 메소드가 새롭게 추가되었다. 인터페이스는 모두 default 메소드가 구현되었으며 새로이 추가된 메소드의 일람은 다음과 같다.
```java
Iterable.forEach(Consumer)
Iterator.forEachRemaining(Consumer)
Collection.removeIf(Predicate)
Collection.spliterator()
Collection.stream()
Collection.parallelStream()
List.sort(Comparator)
List.replaceAll(UnaryOperator)
Map.forEach(BiConsumer)
Map.replaceAll(BiFunction)
Map.putIfAbsent(K, V)
Map.remove(Object, Object)
Map.replace(K, V, V)
Map.replace(K, V)
Map.computeIfAbsent(K, Function)
Map.computeIfPresent(K, BiFunction)
Map.compute(K, BiFunction)
Map.merge(K, V, BiFunction)
Map.getOrDefault(Object, V)
```

## Concurrency API의 확장
Concurrency API의 기능이 추가되었다. 몇가지를 소개해 보자면, ForkJoinPool.commonPool()은 모든 병렬 스트림 작업을 처리하는 구조이다. ForkJoinTak는 명시적으로 특정 풀을 가지지 않고, 일반적인 풀을 사용하게 되었다. 말도 많고 탈도 많았던 ConcurrentHashMap은 완전히 재 작성되었다. 또한 새로운 Locking처리의 구현으로서 추가된 StampedLock은 ReentrantReadWriteLock의 대안으로 사용할 수 있다.
 Future인터페이스의 구현인 CompletableFuture에서는 비동기 작업의 실행과 체이닝을 위한 방법이 제공된다.

## IO/NIO API의 확장
IO/NIO에 메소드가 추가되어 파일이나 입력 스트림에서 java.util.stream.Stream을 직접 생성할 수 있게 되었다.

```java
BufferedReader.lines ()
Files.list (Path)
Files.walk (Path, int FileVisitOption ...)
Files.walk (Path, FileVisitOption ...)
Files.find (Path, int BiPredicate, FileVisitOption ...)
Files.lines (Path, Charset)
DirectoryStream.stream ()
```

새로운 클래스의 `UncheckedIOException`은 `RuntimeException`을 확장한 `IOException`이다.
클로징 가능한 `CloseableStream`이 추가된것 또한 눈여겨 볼만 하다.

## 리플렉션과 어노테이션의 변경
어노테이션이 더 많은곳에서 사용될 수 있게 되었다. 예를들면, `List<@Nullable String>`과 같이 제네릭 형식 매게변수에 작성할 수도 있다. 따라서 정적 분석 도구에서 감지 가능한 오류의 범위가 확대되어 Java의 `built-in type` 시스템 또한 강화되고 정교해졌다.

## Nashorn JavaScript엔진
Nashorn은 새로 JDK에 통합된 경량 고성능 JavaScript구현 엔진이다. Rhino의 후속이며, 성능과 메모리 관리가 개선되었다. javax.script API를 지원하고 있지만, DOM/CSS와 브라우저 플러그인API는 포함되어 있지 않다.

## java.lang, java.util, 그리고 그 밖의것들
지금까지 언급한 것들 이외의 패키지에도 많은 추가 기능들이 있다. 몇가지 주목할만한 것들을 들어보자면, ThreadLocal.withInital(Supplier)는 보다 컴팩트한 thread 로컬 변수의 정의를 허용한다. 오랜 숙원이었던 StringJoiner과 String.join(...)가 자바8에서 구현되었다. Comparator는 체인 또는 필드기반의 비교를 가능하게 하는 새로운 방식을 제안하고 있다. String 풀의 기본값이 25~50K까지 확장되었다.

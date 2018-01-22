출처 : 
- http://www.popit.kr/%EB%82%98%EB%A7%8C-%EB%AA%A8%EB%A5%B4%EA%B3%A0-%EC%9E%88%EB%8D%98-java9-%EB%B9%A0%EB%A5%B4%EA%B2%8C-%EB%B3%B4%EA%B8%B0/

- https://infoscis.github.io/2017/03/24/First-steps-with-java9-and-jigsaw-part-1/

- http://greatkim91.tistory.com/197

#  I. Java 플랫폼 모듈 시스템(Java Platform Module System)
---

## Why module?
&nbsp; **Jigsaw(직소) 프로젝트** 기반하에 개발된 `Java9 Module System`은 안정적인 구성과 강력하고 유연한 캡슐화를 제공한다는 구체적인 목표를 가지고 있다. 이를 통해 응용 프로그램 개발자, 라이브러리 개발자는 또는 Java SE Platform 개발자는 확장 가능한 플랫폼을 만들고 플랫폼 무결성을 높이며 성능을 향상 시킬 수 있다.

하지만 기존의 시스템이 잘 돌고 있는데 우리는 왜 굳이 새로운 모듈 시스템을 필요로 하는가? 기존 Java 패키징 매커니즘의 문제는 **캡슐화 지원**이 부족한 것을 대표적으로 들 수 있다. 클래스 수준의 한정자가 있지만, 패키지를 넘어서는 힘은 가지고 있지 않다. JAR로 자바 패키지가 제공되고 문서화가 잘 되더라도 클래스와 메소드가 public 이라면, 여러가지 이유로 설계자가 의도하지 않은 호출이 발생할 수 있다. 이러한 호출가능성 때문에 API 설계자가 기능 개선을 위해 클래스를 변경하기 어려운 경우도 존재한다.

따라서 이러한 문제를 '모듈'을 만들고 여기에 명시적으로 외부에서 호출할 수 있는 API를 선언함으로써 해결한다. 이러한 방식은 **유연한 런타임 이미지**를 만드는 것 또한 가능하게 한다. 기존에는 JRE 일부분만 배포할 수가 없었다. `XML, SQL, Swing`같은 패키지는 항상 같이 배포되었다. 이제 언어 레벨에서 모듈간의 의존성을 알 수 있으니 사용되는 모듈만 담아서 런타임 환경을 만들 수 있다.


## Module이란?

모듈이란, 아래의 세 가지 질문에 대한 답을 선언하는 소프트웨어 단위이다.
모듈은 순환 종속성을 금지하는 비순환 그래프이다.

1. 이름이 무엇인가?(name)
2. 어떤 것을 제공하는가(export)
3. 무엇을 필요로 하는가(require)


 각 모듈에는 이름이 있고, 충돌을 피하기 위해 패키지 명명 규칙과 유사해야 한다. 두번째로, 모듈은 외부 모듈에서 사용할 수 있도록 공개 API로 간주되는 모든 패키지 목록을 제공한다. 또한 만약 어떤 클래스가 `public`이라 할지라도 `export`된 패키지에 없으면 이 클래스에 접근할 수 없다. 세번째로, 우리의 모듈과 의존 관계가 있는 모든 모듈 목록으로 얘기할 수 있다.

이것은 매우 큰 변화이다. Java 8까지 classpath에 있는 모든 public type은 다른 어떤 type에서도 접근이 가능 하였다.
Jigsaw를 사용하면 Java의 type들에 대한 기존의 접근 방법이

- `public`
- `private`
- `default`
- `protected`

로 부터

- 외부에 대하여 모두 `Public` (`public` to everyone who reads this module (exports))
- 특정 모듈에만 `Public` (`public` to some modules that read this module (exports to))
- 모듈 내부만 `Public` (`public` to every other class within the module itself)
- `private`
- `default`
- `protected`

으로 변화한다는 것을 의미한다.

## Basic syntax

Use case로 살펴보자. 우편번호 유효성 검사를 수행하는 `de.codecentric.zipvalidator`라는 모듈이 있다고 하자.

이 모듈은 `de.codecentric.addresschecker` 모듈이 사용한다.

이 zipvalidator는 다음 `module-info.java`에 다음과 같이 선언된다.

```Java
module de.codecentric.zipvalidator{
    exports de.codecentric.zipvalidator.api;        
}
```

이 모듈은 de.codecentric.zipvalidator.api 패키지를 export하고 다른 모듈은 사용하지 않는다. (java.base 제외). 그리고 이 모듈은 addresschecker에 의해 사용된다.

```Java
module de.codecentric.addresschecker{
    exports de.codecentric.addresschecker.api;
    requires de.codecentric.zipvalidator;
}
```

전체 파일시스템 구조는 다음과 같다

```java

two-modules-ok/
├── de.codecentric.addresschecker
│   ├── de
│   │   └── codecentric
│   │       └── addresschecker
│   │           ├── api
│   │           │   ├── AddressChecker.java
│   │           │   └── Run.java
│   │           └── internal
│   │               └── AddressCheckerImpl.java
│   └── module-info.java
├── de.codecentric.zipvalidator
│   ├── de
│   │   └── codecentric
│   │       └── zipvalidator
│   │           ├── api
│   │           │   ├── ZipCodeValidator.java
│   │           │   └── ZipCodeValidatorFactory.java
│   │           ├── internal
│   │           │   └── ZipCodeValidatorImpl.java
│   │           └── model
│   └── module-info.java
```

## 한계 

- 하위 호환성 부분이 가장 문제이다. 내부 API를 자신도 모르게 쓴 부분을 검사해야 할 것이다
- com.sun.* 과 같이 비공식적 API를 사용할 수 밖에 없는 경우가 많은데, 이러한 경우에 대한 지원이 없다.
- lib/rt.jar나 lib/tool.jar 같은 내부 JAR파일들의 접근이 불가하다. 바로 Java 9으로 이전 하는 것은 위험요소가 있다고 볼 수 있음


출처: http://greatkim91.tistory.com/197 [행복한 아빠]

좀더 자세한 내용은 [링크](http://greatkim91.tistory.com/197)를 확인하라.




# II. 도구(Tools)
---

## 1. Jshell – The Java Shell

-  Java9는 테스트 프로젝트나 main 메소드없이 code snippets을 신속하게 테스트 할 수 있는 대화식 REP(Read-Eval-Print-Loop) 도구를 제공한다. 
- 따라서 우리는 Java 기능을 쉽게 배우거나 평가해 볼 수 있다. 
- 이제 우리는 Code테스트를 위해  java프로젝트를 만들거나 `public static void main(String[] args)`를 정의할 필요가 없다. 오직 코드를 작성하고 즉시 실행하면 된다.


## 2. 통일된 JVM 로깅(Unified JVM Logging)

Java9는 JVM컴포넌트에 대한 공통 로깅 시스템을 제공한다. 통합된 JVM로깅은 모든 로깅 설정에 대한 새로운 명령줄 옵션 `-Xlog`를 사용하여 복잡한 수준의 JVM 구성요소에 대한 근본 원인 분석을 수행할 수 있는 구성하기 쉬운 정밀 도구를 제공한다.

Log메세지는 tag(os, gc, modules..)를 사용하여 분류된다.  하나의 메세지는  다수의 tag(tag 세트)를 가질 수 있다.
- 로깅 레벨 : error, warning, info, debug, trace, develop.
- 3가지 유형의 Output제공 : stdout, stderr, 또는 text file.
- 메세지는 time, uptime, pid, tid, level, tags 등으로 꾸밀 수 있다.


## 3. HTML5 Javadoc

Javadoc은 HTML형식의 API문서를 생성 할 수 있는도구이다. 이전 버전의 JDK에서는 HTML 4.01 형식을 사용했지만 JDK9 Javadoc은 HTML5 markup을 
생성하고 검색 기능 및 Doclint를 향상 시켰다.

HTML5를 지원하는 JDK9에서는  -html5 parameter를 추가 하기만 하면 된다.

```bash
> javadoc-sourcepath"E:\Eclipse Java 9\Java9HTML5Javadoc\src"com.javasampleapproach.html5doc-dE:\home\html  -html5
```

- Javadoc문서에서 프로그램 요소, tag가 지정된 단어나 문구를 검색하는데 사용할 수 있는 검색상자를 사이트상에서 사용할 수 있다. 검색 기능은 로컬로 구현되며 모든 서버측 연산 resources에 의존하지 않는다.
- Xdoclint는 Javadoc 주석의 문제점 ( 예: 잘못된 참조,  접근성 부족, 주석 누락, 구문오류 및 HTML태그 누락) 을 권장 검사한다. 
- 기본적으로 `-Xdoclint`는 활성화 되어 있으며, `Xdoclint:none`을 통해서 비활성화 할 수 있다.



# III. Language Updates
---

## 1. try-with-resources 향상
Java7 은 try-with-resources 구문으로 선언된 resource를 닫을 수(종료) 있는 새로운 접근 방식을 도입 했다. 그 후에 Java9의 try-with-resources는 코드를 작성하는 향상된 방법을 만들었고, 이제 간단하게 코드를 깔끔하고 명확하게 유지 할 수 있다.

```java

       // BufferedReader is declared outside try() block
        BufferedReaderbr=newBufferedReader(newFileReader("C://readfile/input.txt"));
 
        // Java 9 make it simple
        try(br){
            Stringline;
            while(null!=(line=br.readLine())){
                // processing each line of file
                System.out.println(line);
            }
        } catch(IOExceptione){
            e.printStackTrace();
        }
       // BufferedReader is declared outside try() block
        BufferedReaderbr=newBufferedReader(newFileReader("C://readfile/input.txt"));
 
        // Java 9 make it simple
        try(br){
            Stringline;
            while(null!=(line=br.readLine())){
                // processing each line of file
                System.out.println(line);
            }
        } catch(IOExceptione){
            e.printStackTrace();
        }
```


## 2. private 인터페이스 메소드 (Private Interface Method)

- Java8은 인터페이스에 default method 및 static method 라는 두 가지 새로운 기능을 제공한다.
- 하지만 이것은 여전히 우리를 불편하게 만드는데 왜냐하면 단지 특정 기능을 처리하는 내부method 일뿐인데도 외부에 공개되는 public method로 만들었어야 했다.
- interface를 구현하는 다른 interface 또는 class가 해당 method에 액세스하거나 상속 할수 있는것을 원하지 않는다.
- Java9 Private Interface Method는 interface에 `private method` 및 `private static method`라는 새로운 기능을 제공하여 문제를 해결한다. 이제 중복 코드를 피하고 interface에 대한 캡슐화를 유지 할 수 있다.

```Java

public interface IMyInterface{
 
    private void method1(String arg){
        // do something
    }
 
    private static void method2(Integer arg){
        // do something
    }
}
public interface IMyInterface{
 
    private void method1(String arg){
        // do something
    }
 
    private static void method2(Integer arg){
        // do something
    }
}
```

## 3. 다이아몬드 연산자(Diamond Operator)

Java7 에는 코드를 보다 읽기 쉽게 만드는데 도움되는 Diamond Operator라는 새로운 기능이 있었지만 익명 클래스(Anonymous Inner Class)에는 제한적이었다. ‘<>’(다이몬드 연산자)는 다음과 같은 코드를 작성하는 경우 익명 클래스와 함께 사용할 수 없다. ( `Compile Error`가 발생 )

```Java

MyHandler<Integer> intHandler = new MyHandler<>(10){// Anonymous Class };
MyHandler<?> handler = new MyHandler<>("Onehundred"){// Anonymous Class };
1
2
MyHandler<Integer> intHandler = new MyHandler<>(10){// Anonymous Class };
MyHandler<?> handler = new MyHandler<>("Onehundred"){// Anonymous Class };
Java 9는 익명 클래스에 대한 Diamond Operator를 허용한다.
Java

MyHandler<Integer> intHandler = new MyHandler<>(1){
    @Override
    public void handle(){
        // handling code...
    }
};
 
MyHandler<? extends Integer> intHandler1 = new MyHandler<>(10){
    @Override
    void handle(){
        // handling code...
    }
};
 
MyHandler<?> handler = new MyHandler<>("One hundred"){
    @Override
    void handle(){
        // handling code...
    }
};

MyHandler<Integer> intHandler = new MyHandler<>(1){
    @Override
    public void handle(){
        // handling code...
    }
};
 
MyHandler<? extends Integer> intHandler1 = new MyHandler<>(10){
    @Override
    void handle(){
        // handling code...
    }
};
 
MyHandler<?> handler = new MyHandler<>("One hundred"){
    @Override
    void handle(){
        // handling code...
    }
};
```

# IV. 새로운 Core Libraries

---

## 1. 프로세스 API (Process API)

프로세스 검색을 위한 새로운 방법 :  Java9 `Process API`로 모든 프로세스, 현재 프로세스, 하위 프로세스 및 종료된 프로세스 정보를 확인할 수 있다.

```Java

// current Process
ProcessHandle processHandle = ProcessHandle.current();
 
processHandle.getPid();
processHandle.isAlive();
processHandle.children().count();
processHandle.supportsNormalTermination();
 
ProcessHandle.InfoprocessInfo = processHandle.info();
 
processInfo.arguments();
processInfo.command();
processInfo.totalCpuDuration();
processInfo.user();
 
// all Processes
Stream<ProcessHandle> processStream = ProcessHandle.allProcesses();
 
// destroy Process
processHandle.destroy();

// current Process
ProcessHandle processHandle = ProcessHandle.current();
 
processHandle.getPid();
processHandle.isAlive();
processHandle.children().count();
processHandle.supportsNormalTermination();
 
ProcessHandle.InfoprocessInfo = processHandle.info();
 
processInfo.arguments();
processInfo.command();
processInfo.totalCpuDuration();
processInfo.user();
 
// all Processes
Stream<ProcessHandle> processStream = ProcessHandle.allProcesses();
 
// destroy Process
processHandle.destroy();
```


## 2. 플랫폼 로깅 API 및 서비스 (Platform Logging API and Service)

Java9는 플랫폼 클래스가 메세지를 기록하는 데 사용할 수 있는 최소한의 로깅 API와 해당 메세지의 사용자를 위한 서비스 인터페이스를 제공한다. 

`LoggerFinder`의 구현은 시스템 클래스 로더를 사용하는 `java.util.ServiceLoader` API의 도움으로 로드 된다. 이 구현을 기반으로 애플리케이션 / 프레임워크는 `java.util.logging` 또는 백엔드를 구성하지 않고 자체 외부 로깅 백엔드를 연결 할 수 있다. 

`LoggerFinder`가 return할 Logger를 알 수 있도록 LoggerFinder에 클래스 이름 또는 특정 Logger와 관련된 모듈을 전달할 수 있다.

만약 구체적인 구현이 발견되지 않으면 JDK 기본 LoggerFinder구현 ( `java.util.logging – java.logging` 모듈안의 )이 사용되며 따라서 로그 메세지는 `java.util.logging.Logger`로 라우팅된다.

```Java

package java.lang;
...
public class System {
    System.Logger getLogger(String name) { ... }
    System.Logger getLogger(String name, ResourceBundle bundle) { ... }
}


package java.lang;
...
public class System {
    System.Logger getLogger(String name) { ... }
    System.Logger getLogger(String name, ResourceBundle bundle) { ... }
}
```


## 3. CompletableFuture API 강화( Enhancements )

- Java Future를 개선하기 위해 Java 8은 CompletableFuture를 제공한다. 
- `CompletableFuture`는 준비 될 때 마다 일부 코드를 실행할 수 있다. 
- 이제 Java 9는 지연 및 시간 초과를 지원하는 개선된 `CompletableFuture API`를 제공한다.

#### delay()
```Java

future.completeAsync(supplier, CompletableFuture.delayedExecutor(3, TimeUnit.SECONDS))
		.thenAccept(result -> System.out.println("accept: " + result));
 
// other statements
future.completeAsync(supplier, CompletableFuture.delayedExecutor(3, TimeUnit.SECONDS))
    .thenAccept(result -> System.out.println("accept: " + result));
 
// other statements
```

####orTimeout()
```Java

		// TIMEOUT = 3;
		// doWork() takes 5 seconds to finish
 
		CompletableFuture<String> future = 
				doWork("JavaSampleApproach")
				.orTimeout(TIMEOUT, TimeUnit.SECONDS)
				.whenComplete((result, error) -> {
					if (error == null) {
						System.out.println("The result is: " + result);
					} else {
						System.out.println("Sorry, timeout in " + TIMEOUT + " seconds");
					}
				});
    // TIMEOUT = 3;
    // doWork() takes 5 seconds to finish
 
    CompletableFuture<String> future = 
        doWork("JavaSampleApproach")
        .orTimeout(TIMEOUT, TimeUnit.SECONDS)
        .whenComplete((result, error) -> {
          if (error == null) {
            System.out.println("The result is: " + result);
          } else {
            System.out.println("Sorry, timeout in " + TIMEOUT + " seconds");
          }
        });
```
#### completeOnTimeout()

```Java

		// TIMEOUT = 3;
		// doWork() takes 5 seconds to finish

		CompletableFuture<String> future = 
				doWork("JavaSampleApproach")
				.completeOnTimeout("JavaTechnology", TIMEOUT, TimeUnit.SECONDS)
				.whenComplete((result, error) -> {
					if (error == null) {
						System.out.println("The result is: " + result);
					} else {
						// this statement will never run.
						System.out.println("Sorry, timeout in " + TIMEOUT + " seconds.");
					}
				});
    // TIMEOUT = 3;
    // doWork() takes 5 seconds to finish
 
    CompletableFuture<String> future = 
        doWork("JavaSampleApproach")
        .completeOnTimeout("JavaTechnology", TIMEOUT, TimeUnit.SECONDS)
        .whenComplete((result, error) -> {
          if (error == null) {
            System.out.println("The result is: " + result);
          } else {
            // this statement will never run.
            System.out.println("Sorry, timeout in " + TIMEOUT + " seconds.");
          }
        });
```
##4. Reactive Streams – Flow API

- Java9는 상호 운용 가능한 publish-subcribe프레임워크를 지원하는 java.util.concurrent.Flow에서 Reactive Stream을 소개한다.
```Java

    @FunctionalInterface
    public static interface Publisher<T> {
        public void subscribe(Subscriber<?superT> subscriber);
    }
 
    public static interface Subscriber<T> {
        public void onSubscribe(Subscription subscription);
        public void onNext(Titem);
        public void onError(Throwable throwable);
        public void onComplete();
    }
 
    public static interface Subscription {
        public void request(long n);
        public void cancel();
    }
 
    public static interface Processor<T,R> extends Subscriber<T>, Publisher<R> {
    }

    @FunctionalInterface
    public static interface Publisher<T> {
        public void subscribe(Subscriber<?superT> subscriber);
    }
 
    public static interface Subscriber<T> {
        public void onSubscribe(Subscription subscription);
        public void onNext(Titem);
        public void onError(Throwable throwable);
        public void onComplete();
    }
 
    public static interface Subscription {
        public void request(long n);
        public void cancel();
    }
 
    public static interface Processor<T,R> extends Subscriber<T>, Publisher<R> {
    }
```
아래 다이어그램은 동작과 새로운 Flow API를 사용하여 Reactive Stream을 구현한 방법을 보여준다.
reactive-stream-flow-interface-behavior

## 5. Collections(List, Set, Map)를 위한 팩토리 메소드 (Factory Method for Collections: List, Set, Map)

Java9는 적은수의 요소로 편리하게 콜렉션과 맵의 인스턴스를 생성 할 수 있는 새로운 정적 팩토리 메소드를 제공한다.

```Java

List<String> immutableList = List.of("one","two","three");
Set<String> immutableSet = Set.of("one","two","three");
Map<Integer,String> immutableMap = Map.of(1,"one",2,"two",3,"three")


List<String> immutableList = List.of("one","two","three");
Set<String> immutableSet = Set.of("one","two","three");
Map<Integer,String> immutableMap = Map.of(1,"one",2,"two",3,"three")
다만 정적 팩토리 메소드로 작성된 콜렉션은 불변이므로, 요소 또는 null을 add/put 하려고 하면 java.lang.UnsupportedOperationException 또는 java.lang.NullPointerException 을 발생 시킨다.
```


## 6. 강화된Deprecation (Enhanced Deprecation)

Java9 @Deprecated 애노테이션의 새롭게 추가된 메소드는 forRemoval() 및 since() 이다.

새로운 @Deprecated를 사용하는 예
```java
@Deprecated(since =”1.5″, forRemoval = true)
```
## 7. Stack-Walking API

- Java 9는 StackWalker 로 스택 추척을 필터링하여 지연 액세스를 위한 스택 워킹의 효율적인 방법을 제공한다. 
- StackWalker 객체를 사용하면 스택을 가로질러 액세스 할 수 있으며 여기에는 몇가지 유용하고 강력한 메소드를 포함하고 있다.

```java

    public <T> T walk(Function<? super Stream<StackFrame>, ? extends T> function);
    public void forEach(Consumer<? super StackFrame> action);
    public Class<?> getCallerClass();
    public <T> T walk(Function<? super Stream<StackFrame>, ? extends T> function);
    public void forEach(Consumer<? super StackFrame> action);
    public Class<?> getCallerClass();
```

- 가장 중요한 메소드는walk() 이다. 이 메소드는 현재 스레드에 대한 StackFrame 스트림을 open한다.
- StackFrame 스트림과 함께 메소드를 적용하여 사용하도록 하라.

## 8. 다른 개선점들 (Other Improvements)
### 8.1 Stream 개선(Stream Improvements)

Java 9 Stream 은 `iterate(), takeWhile()/dropWhile(), ofNullable()`  과 같은 새로운 추가 메소드를 사용하여 비동기 프로그래밍에 대한 몇 가지 유용한 개선 사항을 제공한다.
##### iterate()
```Java

IntStream
	.iterate(1, i -> i < 20, i -> i * 2)
	.forEach(System.out::println);

IntStream
  .iterate(1, i -> i < 20, i -> i * 2)
  .forEach(System.out::println);
takeWhile()Java

//for ordered Stream
Stream.of(1, 2, 3, 4, 5, 6).takeWhile(i -> i <= 3).forEach(System.out::println);

// The result is:
// 1
// 2
// 3

// for unordered Stream
Stream.of(1, 6, 5, 2, 3, 4).takeWhile(i -> i <= 3).forEach(System.out::println);

// The result is:
// 1

//for ordered Stream
Stream.of(1, 2, 3, 4, 5, 6).takeWhile(i -> i <= 3).forEach(System.out::println);
 
// The result is:
// 1
// 2
// 3
 
// for unordered Stream
Stream.of(1, 6, 5, 2, 3, 4).takeWhile(i -> i <= 3).forEach(System.out::println);
 
// The result is:
// 1
dropWhile()

//for ordered Stream
Stream.of(1, 2, 3, 4, 5, 6).dropWhile(i -> i <= 3).forEach(System.out::println);
 
// It drops (1,2,3), the result is:
// 4
// 5
// 6
 
//for unordered Stream
Stream.of(1, 6, 5, 2, 3, 4).dropWhile(i -> i <= 3).forEach(System.out::println);
 
// It drops (1), the result is:
// 6
// 5
// 2
// 3
// 4

//for ordered Stream
Stream.of(1, 2, 3, 4, 5, 6).dropWhile(i -> i <= 3).forEach(System.out::println);
 
// It drops (1,2,3), the result is:
// 4
// 5
// 6
 
//for unordered Stream
Stream.of(1, 6, 5, 2, 3, 4).dropWhile(i -> i <= 3).forEach(System.out::println);
 
// It drops (1), the result is:
// 6
// 5
// 2
// 3
// 4
ofNullable()Java

		// numbers [1,2,3,null]
		// mapNumber [1 - one, 2 - two, 3 - three, null - null]
		
		List<String> newstringNumbers = numbers.stream()
				.flatMap(s -> Stream.ofNullable(mapNumber.get(s)))
				.collect(Collectors.toList());
 
		// The result is:
		// [one, two, three]

    // numbers [1,2,3,null]
    // mapNumber [1 - one, 2 - two, 3 - three, null - null]
    
    List<String> newstringNumbers = numbers.stream()
        .flatMap(s -> Stream.ofNullable(mapNumber.get(s)))
        .collect(Collectors.toList());
 
    // The result is:
    // [one, two, three]
```

## 8.2 Optional 개선 (Optional Improvements)
Java 9 는 새로운 `Optional::stream` 으로 Optional 객체 지연작업을 제공하며 zero 또는 하나 이상의 요소 스트림을 반한한다. 또한 빈요소를 자동으로 확인하고 제거한다.

```Java

// streamOptional(): [(Optional.empty(), Optional.of("one"), Optional.of("two"), Optional.of("three")]
 
List<String> newStrings = streamOptional()
				.flatMap(Optional::stream)
				.collect(Collectors.toList());
				
// Result: newStrings[one, two, three]
// streamOptional(): [(Optional.empty(), Optional.of("one"), Optional.of("two"), Optional.of("three")]
 
List<String> newStrings = streamOptional()
        .flatMap(Optional::stream)
        .collect(Collectors.toList());
        
// Result: newStrings[one, two, three]
```

- isPresent() 및orElse() 를 사용하여 코드를 보다 명확하게 만들고 “else” 케이스를 처리하는 대신 Java9의 ifPresentOrElse() 메소드를 사용할 수 있다.
```Java

		Optional<Integer> result3 = getOptionalEmpty();
		result3.ifPresentOrElse(
				x -> System.out.println("Result = " + x),
				() -> System.out.println("return " + result2.orElse(-1) + ": Result not found."));
				
		// return -1: Result not found.

    Optional<Integer> result3 = getOptionalEmpty();
    result3.ifPresentOrElse(
        x -> System.out.println("Result = " + x),
        () -> System.out.println("return " + result2.orElse(-1) + ": Result not found."));
        
    // return -1: Result not found.
```
- or() 메소드는 값이 존재하는지 검사하고 값이 있는 경우 해당  Optional 을 리턴하고 그렇지 않은 경우 supplying function에서 생성한 다른 Optional 값을 리턴한다.
```Java

		Optional<Integer> result = getOptionalEmpty() // Empty Optional object
				.or(() -> getAnotherOptionalEmpty()) // Empty Optional object
				.or(() -> getOptionalNormal())  // this return an Optional with real value 42
				.or(() -> getAnotherOptionalNormal());  // this return an Optional with real value 99
				
		// Result: Optional[42]

    Optional<Integer> result = getOptionalEmpty() // Empty Optional object
        .or(() -> getAnotherOptionalEmpty()) // Empty Optional object
        .or(() -> getOptionalNormal())  // this return an Optional with real value 42
        .or(() -> getAnotherOptionalNormal());  // this return an Optional with real value 99
        
    // Result: Optional[42]
```

# V. 클라이언트 기술 (Client Technologies)

---

## 1. 다중 해상도 이미지 (Multi-Resolution Images)

 `java.awt.image` 패키지에 정의된 새로운 API를 사용하면 다음과 같은 이점이 있음.
- 해상도가 다른 다수의 이미지를 변형된 이미지로 캡슐화 한다.
- 이미지에서 모든 변형을 얻음.
- 해상도 관련 이미지 변경 가져오기 –> 특정 DPI 메트릭을 기반으로 지정된 크기로 논리 이미지를 표현하는 최상의 변형을 가져온다.


## 2.TIFF 이미지 I/O 플러그인 (TIFF Image I/O Plugins)

이전 버전의 Java에서 이미지 I/O 프레임워크 `javax.imageio` 는 PNG 및 JPEG와 같은 일부 형식의 이미지 코덱을 플러그인하는 표준 방법을 제공한다. 하지만 TIFF는 여전히 세트에서 빠져 있다. 

이전에는  `com.sun.media.imageio.plugins.tiffbefore.` 에 패키지 되어 있었고 Java 9 TIFF 이미지 I/O 플러그인(TIFF Image I/O plugins) 에는com.sun.media.imageio.plugins.tiff. 에서 이름이 변경된 javax.imageio.plugins.tiff 라는 새로운 패키지가 있다.
이 패키지에는 내장 TIFF 판독기(reader),  기록기(writer) 플러그인을 지원하는 몇몇 클래스가 들어 있으며 다음 클래스를 포함하고 있다.
기본 TIFF사향, Exif IFD, TIFF-F(RFC 2306) 파일, GeoTIFF IFD에서 볼 수 있는 공통 추가태크 및 태그 집한을 나타내는 일부 클래스.
TIFFImageReadParam: 읽을 수 있는 메타 데이터 태그와 일부 대상 속성을 설정할 수 있는  ImageReadParam 의 확장.


# VI. 국제화 (Internationalization)

---

## 1. 유니코드 8.0 (Unicode 8.0)
- Java 8 은 유니코드 6.2를 지원함.
- Java 9 는 10,555자,  29개의 스크립트 및 42개 블록의 유니코드 8.0 표준을 지원한다.
## 2. UTF-8 프로퍼티 파일 (UTF-8 Properties Files)

- 이전 릴리즈에서는 프로퍼티 리소스 번들을 로드 할때 ISO-8859-1 인코딩이 사용되었음. (PropertyResourceBundle –InputStream 에서 해당 인스턴스를 생성하려면 입력을 스트림을 ISO-8859-1로 인코딩해야함 ) 
- 그러나 ISO-8859-1을 사용하는것은 비 라틴 문자를 표현하는 편리한 방법이 아니다.
Java9에서는 프로퍼티 파일을 UTF-8인코딩으로  로드한다.  문제가 있는 경우 다음 옵션을 고려해보자.
    - 프로퍼티파일을 UTF-8 인코딩으로 변환.
    - runtime 시스템 특성을 지정.
```java
java.util.PropertyResourceBundle.encoding=ISO-8859-1
1
java.util.PropertyResourceBundle.encoding=ISO-8859-1
```
## 3. 기본 로케일 데이터 변경(Default Locale Data Change)
JDK8 및 이전 릴리즈에서는 JRE가 기본 로케일 데이터다.  

JDK9은 기본적으로 CLDR(Unicode Common Locale Data Repository 프로젝트에서 제공하는 로케일 데이터)을 가장 높은 우선 순위로 설정한다. 

이것은  java.locale.providers 시스템 프로퍼티를 사용하여 기본 순서로 로케일 데이터 소스를 선택하는 방법이다. 프로 바이더가 로케일 데이터를 요구하지 않았을 경우 다음의 프로바이더가 처리된다.
```java
java.locale.providers=COMPAT,CLDR,HOST,SPI

java.locale.providers=COMPAT,CLDR,HOST,SPI
```
이 속성을 설정하지 않으면 기본 동작은 다음과 같다.
```java
java.locale.providers=CLDR,COMPAT,SPI
java.locale.providers=CLDR,COMPAT,SPI

```
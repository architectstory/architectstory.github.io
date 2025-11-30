---
sidebar:
  nav: "tech"
---
# Java 21

오라클은 2023년 9월 19일(현지시간) 예정된 일정에 따라
미국 라스베이거스에서 개최한 연례 컨퍼런스('Oracle Cloud World 2023')에서
최신 장기지원(LTS) 버전인 'Java 21'을 출시를 발표했다.
이와 함께 기존 자바11 LTS의 기술지원기간을 2032년으로 8년 더 연장했다.

[https://jdk.java.net/21/](https://jdk.java.net/21/){:target="_blank"}

Java 21은 플랫폼 15가지 새로운 JEP(Java Enhancement Proposal)를 비롯해
각종 성능 향상과 안정성 개선을 포함한다.

15가지 추가 변경 내용은 아래와 같다.

   - JEP 430:	String Templates (Preview)
   - JEP 431:	Sequenced Collections
   - JEP 439:	Generational ZGC
   - JEP 440:	Record Patterns
   - JEP 441:	Pattern Matching for switch
   - JEP 442:	Foreign Function & Memory API (Third Preview)
   - JEP 443:	Unnamed Patterns and Variables (Preview)
   - JEP 444:	Virtual Threads
   - JEP 445:	Unnamed Classes and Instance Main Methods (Preview)
   - JEP 446:	Scoped Values (Preview)
   - JEP 448:	Vector API (Sixth Incubator)
   - JEP 449:	Deprecate the Windows 32-bit x86 Port for Removal
   - JEP 451:	Prepare to Disallow the Dynamic Loading of Agents
   - JEP 452:	Key Encapsulation Mechanism API
   - JEP 453:	Structured Concurrency (Preview)

이중 눈에 띄는 신기능은 JEP 444: Virtual Threads 이다.

Virtual Threads 를 알아보기 이전에 Spring framework 에서의 Thread 처리에 대해서 잠시 알아보자

## Thread: Spring framework
Java 적용 프로젝트에서는 Spring Framework를 적용한다.
Java 기반 서버 애플리케이션의 고민 중의 하나가 동시성 처리 문제이다.
그럼 동시성 처리를 위한 Java Spring Framework 에서의 Thread 처리 방식에 잠시 대해 알아보자.
Spring 에서는 두 가지 Stack(Spring MVC, Spring Webflux)을 제공한다.

![spring boot](/assets/images/springboot3/springboots.png)

- Spring Boot Web (MVC) Framework

  1개의 Thread가 1개의 클라이언트 요청에 대응하는 thread-per-request 방식으로 동작한다.
  클라이언트로부터 동시 요청이 많으면 Thread의 수 역시 그에 비례해서 증가해야만 대응할 수 있다.
  이런 모델의 한계는 Thread 자체가 많은 메모리를 소비하고, 불필요한 컨텍스트 스위칭이 발생한다.
  만약, Tomcat 서버의 가용한 전체 Thread가 3개이고, 네트워크 지연 등의 사유로 블로킹이 되는 상황이라 가정하면
  새로운 클라이언트의 요청을 처리할 수 없고, 사용 가능한 Thread가 반환될때 까지 대기하거나
  대기 시간이 지나치게 길어지면 에러처리된다.

  ![spring boot](/assets/images/springboot3/threads.png)

- Spring Boot Webflux

  1개 Thread를 사용함으로 한계를 회피하려는 노력했다.

  ![spring boot](/assets/images/springboot3/spring_webflux.png)

  한 Thread가 I/O 작업이 끝나기를 기다리는 대신, Thread를 반납하여 다른 요청을 처리할 수 있도록 하는 것이다.
  I/O 작업은 제외하고 연산을 수행하는 동안에만 Thread를 보유하기 때문에 적은 수의 Thread로도 많은 요청을 처리할 수 있다.
  하지만 하나의 Thread가 온전히 처리하는 것이 아니기 때문에 Stacktrace 보면 파편화된 정보로 트러블 슈팅이 어렵고 코드 해석도 편하진 않다.

- Thread on JVM

  JVM 에서 얘기하는 Thread는 OS 에서 생성한 Thread를 1:1로 랩핑한 Thread인 Platform Thread 이다.

  ![spring boot](/assets/images/springboot3/platform_thread.png)

  그래서 Thread 하나를 생성하는 것은 OS Thread 하나를 생성하는 것과 동일하고,
  Thread간 전환을 위해서는 컨텍스트 스위칭이 발생한다.
  이러한 성능상의 이유로 Thread 풀을 만들어 관리하지만 너무 많은 Thread가 생성되면 문제가 된다.

## Virtual Thread
  Virtual Thread 는 OS Thread와 1:1 맵핑 되지 않는다.

  ![spring boot](/assets/images/springboot3/virtual_thread.png)

  기존 JVM 에서의 Platform Thread 는 'Carrier Thread'로 대체 되었고 JVM 내 'ForkJoinPool'로 관리되고
  'Virtual Thread'를 생성하여 사용한다.
  Queue 안에 있는 Virtual Thread 쓰레드들은 FIFO 방식으로 스케쥴링 되고,
  ForkJoinPool 안의 Carrier Thread 는 기본적으로 CPU 코어 갯수(정확히는 available processor)만큼 생성된다.
  Virtual Threads 의 경우 OS의 Thread도 아니고 JVM 에서 생성하기 때문에 생성 비용이 낮다.

  [Performance of virtual thread](https://www.baeldung.com/spring-6-virtual-threads){:target="_blank"}

  Virtual Thread 활용시 Reactive Programming  하지 않고도 비슷한 Performance 낼수 있을거 같다.


## Virtual Threads vs. Platform Threads

 - Spring Boot 3 에서 Java 21 을 사용할 수 있다.

   ![spring boot](/assets/images/springboot3/springboot-java21.png)

 - java 17 (Thread activeCount 가 103개)
   ``` java
   public static void main(String[] args) throws Exception {
           ScheduledThreadPoolExecutor scheduler = new ScheduledThreadPoolExecutor(2);
           scheduler.scheduleWithFixedDelay(() -> {
               System.out.println("Thread.activeCount " + Thread.activeCount());
           }, 300L, 300L, TimeUnit.MILLISECONDS);

           for (int i = 0; i < 100; i++) {
               new Thread(() -> {
                   try {
                       Thread.sleep(1000L); // blocking 작업 처리중...
                   } catch (InterruptedException ignore) {
                   }
               }, "client-task-" + i).start();
           }

           System.in.read();
   }
   ```

 - java 21 (Thread activeCount 가 3개)
   ``` java
   public static void main(String[] args) throws Exception {
           ScheduledThreadPoolExecutor scheduler = new ScheduledThreadPoolExecutor(2);
           scheduler.scheduleWithFixedDelay(() -> {
               System.out.println("Thread.activeCount " + Thread.activeCount());
           }, 300L, 300L, TimeUnit.MILLISECONDS);

           for (int i = 0; i < 100; i++) {
               Thread.startVirtualThread(() -> {
                   System.out.println("Hello World! Virtual Thread");

                   try {
                       Thread.sleep(1000L); // blocking 작업 처리중...
                   } catch (InterruptedException ignore) {
                   }
               });
           }

           System.in.read();
   }
   ```


## Java 21 Language Specification 변경사항
 - record 클래스
   - java 17 Vo 클래스 생성과 data 변수 리턴
     ``` java
     public class Vo {
         private final int data;
         public Vo(int data) {
             this.data = data;
         }
         public int getData() {
             return data;
         }
     }
     ```
   - java 17 lombok 을 이용한 Vo 클래스 생성과 변수 반환
     ``` java
     @Getter
     @AllArgsConstructor
     public class Vo {
         private final int data;
     }
     ```

   - Java 21 (lombok 이 불필요한것 같다.)
     ``` java
     public record Record(int data) {
     }
     ```

 - Collectors.toList() --> toList() 로 간략히

   - java 17
     ``` java
            List<Integer> list = List.of(1, 2, 3, 4, 5);
            List<Integer> result = list.stream()
                                       .filter(i -> i > 3)
                                       .collect(Collectors.toList());
     ```
   - java 21

     ``` java
            List<Integer> list = List.of(1, 2, 3, 4, 5);
            List<Integer> result = list.stream()
                                       .filter(i -> i > 3)
                                       .toList();
     ```

 - switch 의 단순화
   - java 17
     ``` java
            String goodOrBad = "good";
            int result = 1;
            switch (goodOrBad) {
                case "good":
                    result = 1;
                    break;
                case "bad":
                    result = 2;
                    break;
     ```
   - java 21
     ``` java
            String goodOrBad = "good";
            int result = switch (goodOrBad) {
                case "good" -> 1;
                case "bad" -> 2;
                default -> 0;
            };
     ```

 - instanceof
    - instanceof pattern matching
        ``` java
                Object obj = "java 17";
                if (obj instanceof String){
                    String str = (String) obj;
                    System.out.println(str);
                }
        ```
        ``` java
                Object obj = "java 21";
                if (obj instanceof String str)
                    System.out.println(str);
        ```

 - number format
     - number format expression

        ``` java
        int number = 1_000_000; //표현
        NumberFormat
            .getCompactNumberInstance(Locale.KOREA, NumberFormat.Style.LONG)
            .format(number);
        ```


## visual studio
https://gethlemn.tistory.com/28



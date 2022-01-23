# 8장. 경계 p143
시스템에 들어가는 모든 소프트웨어를 직접 개발하는 경우는 드물다. 오픈 소스를 이용하거나, 사내 다른 팀이 제공하는 컴포넌트를 사용하거나, 어떤 식으로든 이 외부 코드를 우리 코드에 깔끔하게 통합해야한다. 이 장에서는 소프트웨어 경계를 깔끔하게 처리하는 기법과 기교를 살펴본다.

## 목차
1. [외부 코드 사용하기](#외부-코드-사용하기-p144)
1. [경계 살피고 익히기](#경계-살피고-익이기-p146)
1. [log4j 익히기](#log4j-익히기-p147)
1. [학습 테스트는 공짜 이상이다](#학습-테스트는-공짜-이상이다-p149)
1. [아직 존재하지 않는 코드를 사용하기](#아직-존재하지-않는-코드를-사용하기-p150)
1. [깨끗한 경계](#깨끗한-경계-p151)

## 외부 코드 사용하기 p144
- 인터페이스 제공자와 사용자 사이에는 특유의 긴장이 존재한다.
```
- 패키지 제공자나 프레임워크 제공자는 적용성을 최대한 넓히려 애쓴다. (더 많은 환경에서 돌아가야 더 많은 고객이 구매하므로)
- 반면, 사용자는 자신의 요구에 집중하는 인터페이스를 바란다.
```
- 이런 차이로 시스템 경계에 문제가 생길 소지가 많다.

<br>

- 한 예로, java.util.map을 살펴보자.
- Map이 제공하는 기능과 유연성은 확실히 유용하지만 그만큼 위험도 크다.
- Map의 clear() 메서드는 Map의 사용자라면 누구나 Map 내용을 지울 권한이 있다. 혹은 Map에 특정 객체 유형만 저장하기로 결정했다고 가정하자. 그렇지만 Map은 객체 유형을 제한하지 않으므로 마음만 먹으면 사용자는 어떤 객체 유형도 추가할 수 있다.

<br>

- `Sensor`라는 객체를 담는 Map을 만드려면 다음과 같이 Map을 생성한다.

```java
Map sensors = new HashMap();
```
- `Sensor` 객체가 필요한 코드는 다음과 같이 `Sensor` 객체를 가져온다.
```java
Sensor s = (Sensor)sensors.get(sensorId);
```
- 위와 같은 코드가 한 번이 아니라 여러 차례 나온다.
- Map이 반환하는 Object를 올바른 유형으로 변환할 책임은 Map을 사용하는 클라이언트에 있다.
- 하지만, **깨끗한 코드**라 보기 어렵고, 위와 같은 코드는 의도도 분명히 드러나지 않는다.

<br>

- 제네릭을 사용하면 코드 가독성이 크게 높아진다.

```java
Map<String, Sensor> sensors = new HashMap<Sensor>();
...
Sensor s = sensor.get(sensorId);
```
- 하지만 "Map<String, Sensor>이 사용자에게 필요하지 않은 기능까지 제공한다."는 문제는 해결하지 못한다.
- 아래와 같이 제네릭스의 사용 여부를 `Sensors` 안에서 결정하면 코드를 좀 더 깔끔하게 할 수 있다.

```java
public class Sensors {
    private Map sensors = new HashMap();

    public Sensor getById(String id) {
        return (Sensor) sensors.get(id);
    }

    // 이하 생략
}
```
- 경계 인터페이스인 Map을 `Sensor`클래스 안으로 숨긴다.
- 따라서 Map 인터페이스가 변하더라도 나머지 프로그램에는 영향을 미치지 않는다.
- 제네릭스를 사용하든 하지 않든 `Sensors` 클래스 안에서 객체 유형을 관리하고 변환하기에 더 이상 문제가 안된다.

- Map 클래스를 사용할 때마다 위와 같이 캡슐화하라는 소리가 아니다. Map을 여기저기 넘기지 말란 말이다.

```
- Map과 같은 경계 인터페이스를 사용할 때는 이를 이용하는 클래스나 클래스 계열 밖으로 노출되지 않도록 주의한다. 
- Map 인스턴스를 공개 API 인수로 넘기거나 반환값으로 사용하지 않는다.
```

## 경계 살피고 익히기 p146
- 외부 코드를 사용하면 적은 시간에 더 많은 기능을 출시하기 쉬워진다.
- 외부 패키지 테스트는 우리 책임이 아니지만 우리 자신을 위해 우리가 사용할 코드는 테스트하는 편이 바람직하다.

<br>

- 외부 코드를 익히기는 어렵다. 외부 코드를 통합하기도 어렵다. 두 가지를 동시에 하기는 두 배나 어렵다.
- 우리쪽 코드를 작성해 외부 코드를 호출하는 대신 먼저 간단한 테스트 케이스르 작성해 외부 코드를 익히면 어떨까? 짐 뉴커크는 이를 `학습 테스트`라 부른다.
- 학습 테스트는 프로그램에서 사용하려는 방식대로 외부 API를 호출한다.
- 통제된 환경에서 API를 제대로 이해하는지를 확인하는 셈이다. 학습 테스트는 API를 사용하려는 목적에 초점을 맞춘다.

## log4j 익히기 p147
- 로깅 기능을 직접 구현하는 대신 아파치의 log4j 패키지를 사용하려 한다고 가정한다.
- 문서를 자세히 읽기 전에 먼저 첫 번째 테스트 케이스를 작성한다.

```java
@Test
public void testLogCreate() {
    Logger logger = Logger.getLogger("MyLogger");
    logger.info("hello");
}
```
테스트를 돌렸더니 `Appender`라는 뭔가가 필요하다는 오류가 발생해서 `ConsoleAppender`라는 클래스가 필요하다는 것을 알았다.

```java
@Test
public void testLogAddAppender() {
    Logger logger = Logger.getLogger("MyLogger");
    ConsoleAppender appender = new ConsoleAppender();
    logger.addAppender(appender);
    logger.info("hello");
}
```
이번에는 `Appender`에 출력 스트림이 없다는 사실을 발견한다. 구글에서 검색한 후 다음과 같이 시도한다.

```java
@Test
public void testLogAddAppender() {
    Logger logger = Logger.getLogger("MyLogger");
    logger.removeAppAppenders();
    logger.addAppender(new ConsoleAppender(
        new PatternLayout("%p %t %m%n"), ConsoleAppender.SYSTEM_OUT));
    logger.info("hello");
}
```
- 이제서야 제대로 돌아간다. 그런데 `ConsoleAppender`에게 콘솔에 쓰라고 알려야 한다니 뭔가 수상하다.
- 흥미롭게도 `ConsoleAppender.SYSTEM_OUT` 인수를 제거했더니 문제가 없다. 하지만 `PatternLayout`을 제거했더니 출력 스트림이 없다는 오류가 뜬다.
- 좀 더 구글을 뒤지고, 문서를 읽어보고, 테스트를 돌린 끝에 log4j가 돌아가는 방식을 상당히 많이 이해했으며 여기서 얻은 지식을 간단한 단위 테스트 케이스 몇 개로 표현했다.

```java
public class LogTest {
    private Logger logger;

    @Before
    public void initialize() {
        logger = Logger.getLogger("logger");
        logger.removeAllAppenders();
        Logger.getRootLogger().removeAllAppenders();
    }

    @Test
    public void basicLogger() {
        BasicConfigurator.configure();
        logger.info("basicLogger");
    }

    @Test
    public void addAppenderWithStream() {
        logger.addAppender(new ConsoleAppender(
            new PatternLayout("%p %t %m%n"),
            ConsoleAppender.SYSTEM_OUT));
        logger.info("addAppenderWithStream");
    }

    @Test
    public void addAppenderWithoutStream() {
        logger.addAppender(new ConsoleAppender(
            new PatternLayout("%p %t %m%n")));
        logger.info("addAppenderWithoutStream");
    }
}
```
- 지금까지 간단한 콘솔 로거를 초기화하는 방법을 익혔으니, 이제 모든 지식을 독자적인 로거 클래스로 캡슐화한다. 그러면 나머지 프로그램은 log4j 경계 인터페이스를 몰라도 된다.

## 학습 테스트는 공짜 이상이다 p149
- 학습 테스트에 드는 비용은 없다. (어쨌든 API를 배워야하므로)
- 오히려 **필요한 지식만 확보하는 손쉬운 방법**이다. 학습 테스트는 이해도를 높여주는 정확한 실험이다.
- 학습 테스트는 공짜 이상이다. 투자하는 노력보다 얻는 성과가 더 크다.

<br>

- 학습 테스트는 패키지가 예상대로 도는지 검증한다.
- 패키지 새 버전이 나올 때마다 새로운 위험이 생긴다.
- 새 버전이 우리 코드와 호환되지 않으면 학습 테스트가 이 사실을 곧바로 밝혀낸다.

<br>

- 학습 테스트를 이용한 학습이 필요하든 그렇지 않든, 실제 코드와 동일한 방식으로 인터페이스를 사용하는 테스트 케이스가 필요하다.
- 이런 경계 테스트가 있다면 패키지의 새 버전으로 이전하기 쉬워진다.
- 그렇지 않다면 낡은 버전을 필요 이상으로 오래 사용하려는 유혹에 빠지기 쉽다.

## 아직 존재하지 않는 코드를 사용하기 p150
- 경계와 관련해 또 다른 유형은 아는 코드와 모르는 코드를 분리하는 경계다.

<br>

> 예시) 무선 통신 시스템에 들어가는 소프트웨어 개발

- 소프트웨어의 하위 시스템 중 송신기에 대한 지식이 거의 없었다.
- 송신기 API가 설계되어 있지 않았다.
- 만들어지기 바라는 송신기 인터페이스를 자체적으로 정의하였다.
  <br>
  _지정한 주파수를 이용해 이 스트림에서 들어오는 자료를 아날로그 신호로 전송하라._
- 이렇게 인터페이스를 정의하여 코드는 깔끔하고 깨끗했다.
- 송신기 API가 정의된 후 `TransmitterAdapter`를 구현해 간극을 메꿨다.
- `ADAPTER패턴`으로 API 사용을 캡슐화 해 API가 바뀔 때 수정할 코드를 한곳으로 모았다.

<br>

- 이와 같은 설계는 테스트도 쉽다.
- 적절한 `FakeTransmitter` 클래스를 사용하면 `CommunicationsController` 클래스를 테스트할 수 있다. `Transmitter` API 인터페이스가 나온 다음 경계 테스트 케이스를 생성해 우리가 API를 올바로 사용하는지 테스트할 수도 있다.

## 깨끗한 경계 p151
- 경계에서는 흥미로운 일이 많이 벌어지는데, 변경이 대표적인 예다.
- 소프트웨어 설계가 우수하다면 변경하는데 많은 투자와 재작업이 필요하지 않다. 엄청난 시간과 노력과 재작업을 요구하지 않는다.

<br>

- 경계에 위치하는 코드는 깔끔히 분리한다.
- 또한 기대치를 정의하는 테스트 케이스도 작성한다.
- 이쪽 코드에서 외부 패키지를 세세하게 알아야 할 필요가 없다.
- 통제 불가능한 외부 패키지에 의존하는 대신 통제가 가능한 우리 코드에 의존하는 편이 훨씬 좋다.

<br>

- 외부 패키지를 호출하는 코드를 가능한 줄여 경계를 관리하자.
- Map에서 봤듯이 새로운 클래스로 경계를 감싸거나 아니면 ADAPTER 패턴을 사용해 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환하자.
- 어느 방법이든 가독성이 높아지며, 경계 인터페이스를 사용하는 일관성도 높아지며, 외부 패키지가 변했을 때 변경할 코드도 줄어든다.
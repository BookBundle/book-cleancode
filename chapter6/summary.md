# 6장. 객체와 자료 구조 p117
- 변수를 비공개private로 정의하는 이유가 있다. 남들이 변수에 의존하지 않게 만들고 싶어서다.

## 목차
1. [자료 추상화](#자료-추상화-p118)
1. [자료/객체 비대칭](#자료/객체-비대칭-p119)
1. [디미터 법칙](#디미터-법칙-p123)
    1. [기차 충돌](#기차-충돌)
    1. [잡종 구조](#잡종-구조)
    1. [구조체 감추기](#구조체-감추기)
1. [자료 전달 객체](#자료-전달-객체-p126)
    1. [활성 레코드](#활성-레코드)
1. [결론](#결론-p127)

<br>

## 자료 추상화 p118
```java
// 목록 6-1 구체적인 Point 클래스
public class Point { 
  public double x; 
  public double y;
}
```
```java
// 목록 6-2 추상적인 Point 클래스
public interface Point {
  double getX();
  double getY();
  void setCartesian(double x, double y); 
  double getR();
  double getTheta();
  void setPolar(double r, double theta); 
}
```
- 목록 6-2는 구현을 숨기고, 자료 구조 이상을 표현한다.
- 목록 6-1은 내부 구조를 노출하고, 개별적으로 좌표값을 읽고 설정하게 강제한다.
- 변수를 private 으로 선언하더라도 각 값마다 get과 set 함수를 제공한다면 구현을 외부로 노출하는 구조가 된다.
- 변수 사이에 함수라는 계층을 계층을 넣는다고 구현이 저절로 감춰지지는 않는다.
- 구현을 감추려면 **추상화**가 필요하다!
- 추상 인터페이스를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스다.

## 자료/객체 비대칭 p119
```
- 객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다.
- 자료 구조는 자료를 그대로 공개하며 별다른 함수는 제공하지 않는다.
```
- 두 정의는 본질적으로 상반된다. 두 개념은 사실상 정반대다.
- 사소한 차이로 보일지 모르지만 그 차이가 미치는 영향은 굉장하다.

```java
// 절차적인 도형
public class Square { 
  public Point topLeft; 
  public double side;
}

public class Rectangle { 
  public Point topLeft; 
  public double height; 
  public double width;
}

public class Circle { 
  public Point center; 
  public double radius;
}

public class Geometry {
  public final double PI = 3.141592653589793;

  public double area(Object shape) throws NoSuchShapeException {
    if (shape instanceof Square) { 
      Square s = (Square)shape; 
      return s.side * s.side;
    } else if (shape instanceof Rectangle) { 
      Rectangle r = (Rectangle)shape; 
      return r.height * r.width;
    } else if (shape instanceof Circle) {
      Circle c = (Circle)shape;
      return PI * c.radius * c.radius; 
    }
    throw new NoSuchShapeException(); 
  }
}
```

- `Geometry` 클래스에 둘레 길이를 구하는 `perimeter()` 함수를 추가하고 싶다면? 도형 클래스는 아무 영향도 받지 않는다.
- 도형 클래스에 의존하는 다른 클래스도 마찬가지다.
- 반대로 새 도형을 추가하고 싶다면? `Geometry` 클래스에 속한 함수를 모두 고쳐야 한다.

<br>

- 객체 지향적인 도형 클래스인 목록 6-6에서는 새 도형을 추가해도 기존 함수에 아무런 영향을 미치지 않는다.
- 반면 새 함수를 추가하고 싶다면 도형 클래스 전부를 고쳐야 한다.
```java
// 다형적인 도형
public class Square implements Shape { 
  private Point topLeft;
  private double side;

  public double area() { 
    return side * side;
  } 
}

public class Rectangle implements Shape { 
  private Point topLeft;
  private double height;
  private double width;

  public double area() { 
    return height * width;
  } 
}

public class Circle implements Shape { 
  private Point center;
  private double radius;
  public final double PI = 3.141592653589793;

  public double area() {
    return PI * radius * radius;
  } 
}
```
- 객체와 자료 구조는 근본적으로 양분된다.

> (자료 구조를 사용하는) 절차적인 코드는 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다. 반면, 객체 지향 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다.

- 반대쪽도 참이다.

> 절차적인 코드는 새로운 자료 구조를 추가하기 어렵다. 그러려면 모든 함수를 고쳐야 한다. 객체 지향 코드는 새로운 함수를 추가하기 어렵다. 그러려면 모든 클래스를 고쳐야 한다.

- 즉, **객체 지향 코드에서 어려운 변경은 절차적인 코드에서 쉬우며, 절차적인 코드에서 어려운 변경은 객체 지향 코드에서 쉽다.**

- 복잡한 시스템을 짜다 보면 새로운 함수가 아니라 새로운 자료 타입이 필요한 경우가 생긴다. 이때는 클래스와 객체 지향 기법이 가장 적합하다.
- 반면, 새로운 자료 타입이 아니라 새로운 함수가 필요한 경우도 생긴다. 이때는 절차적인 코드와 자료 구조가 좀 더 적합하다.

## 디미터 법칙 p123
- 디미터 법칙은 잘 알려진 휴리스틱(해결법이 정확히 해결되는지에 대한 문제를 무시하고 일반적으로 좋은 해결법이나 보다 간단한 해결법으로 풀고자 하는 문제 해결법)으로, 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙이다.
- 좀 더 정확히 표현하자면, "클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야한다"고 주장한다.

```
- 클래스 C
- f가 생성한 객체
- f인수로 넘어온 객체
- C 인스턴스 변수에 저장된 객체
```
- 하지만 위 객체에서 허용된 메서드가 반환하는 객체의 메서드는 호출하면 안된다.
- **낯선 사람은 경계하고 친구랑만 놀라는 의미다.**

### 기차 충돌
```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

- 위와 같은 코드를 *기차 충돌 train wreck* 이라 부른다.
- 여러 객차가 한 줄로 이어진 기차처럼 보이기 때문에 아래처럼 나누는 것이 좋다.

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```
- 위 코드가 디미터 법칙을 위반하는지 여부는 위 변수(opts, scratchDir, outputDir)들이 객체인지 아니면 자료 구조인지에 달렸다.
- 객체라면 내부 구조를 숨겨야 하므로 확실히 디미터 법칙을 위반한다.
- 반면, 자료 구조라면 당연히 내부 구조를 노출하므로 디미터 법칙이 적용되지 않는다.

### 잡종 구조
- 이런 혼란으로 말미암아 때때로 절반은 객체, 절반은 자료 구조인 잡종 구조가 나온다.
- 잡종 구조는 중요한 기능을 수행하는 함수도 잇고, 공개 변수나 공개 조회/설정 함수도 있다.
- 이런 구조는 새로운 함수는 물론이고 새로운 자료 구조도 추가하기 어렵다.
- 양쪽 세상에서 단점만 모아놓은 구조로, 잡종 구조는 되도록 피하는 편이 좋다.

### 구조체 감추기
- 위의 `ctxt`, `outputDir` 예제도 좋은 방식이 아니다. 이 경로가 왜 필요할지 같은 모듈에서 (한참 아래로 내려가서) 다음과 같은 코드가 있다.

```java
String outFile = outputDir + "/" + className.replace('.', '/') + ".class"; 
FileOutputStream fout = new FileOutputStream(outFile); 
BufferedOutputStream bos = new BufferedOutputStream(fout);
```
- 추상화 수준을 뒤썩어 놓아 다소 불편하다.
- 위 코드를 보면, 임시 디렉터리의 절대 경로를 얻으려는 이유가 임시 파일을 생성하기 위한 목적이라는 사실을 알 수 있다.
- 그렇다면 `ctxt` 객체에 임시 파일을 생성하라고 시키면 어떨까?

```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```
- 객체에게 맡기기 적당한 임무로 보인다!
- `ctxt`는 내부 구조를 드러내지 않으며, 모듈에서 해당 함수는 자신이 몰라야 하는 여러 객체를 탐색할 필요가 없다.
- 따라서 디미터 법칙을 위반하지 않는다.

## 자료 전달 객체 p126
- 자료 구조체의 전형적인 형태는 공개 변수만 이쏙 함수가 없는 크래스다.
- 이런 자료 구조체를 때로는 자료 전달 객체(Data Transfer Object, DTO)라 한다.

```java
public class Address { 
  public String street; 
  public String streetExtra; 
  public String city; 
  public String state; 
  public String zip;

  /* ... */
}
```
- 흔히 DTO는 데이터베이스에 저장된 가공되지 않은 정보를 애플리케이션 코드에서 사용할 객체로 변환하는 일련의 단계에서 가장 처음으로 사용하는 구조체다.

### 활성 레코드
- DTO의 특수한 형태다.
- 공개 변수가 있거나 비공개 변수에 조회설정 함수가 있는 자료 구조지만, 대개 sava나 find와 같은 탐색 함수도 제공한다.
- 활성 레코드는 데이터베이스 테이블 이나 다른 소스에서 자료를 직접 변환한 결과다.

<br>

- 불행히도 활성 레코드에 비즈니스 규칙 메서드를 추가해 이런 자료 구조를 객체로 취급하는 개발자가 흔하다.
- 이는 바람직 하지 않으며, 이렇게 하게 되면 잡종 구조가 나온다.

<br>

- 해결책은 당연히 **활성 레코드는 자료 구조로 취급**하는 것이다.
- 비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성한다. (여기서 내부 자료는 활성 레코드의 인스턴스일 가능성이 높다.)

## 결론 p127
- 객체는 동작을 공개하고 자료를 숨긴다. 그래서 기존 동작을 변경하면서 새 객체 타입을 추가하기는 쉬운 반면, 기존 객체에 새 동작을 추가하기는 어렵다.
- 자료 구조는 별다른 동작 없이 자료를 노출한다. 그래서 기존 자료 구조에 새 동작을 추가하기는 쉬우나, 기존 함수에 새 자료 구조를 추가하기는 어렵다.
- (어떤) 시스템을 구현할 때, 새로운 자료 타입을 추가하는 유연성이 필요하면 객체가 더 적합하다. 다른 경우로 새로운 동작을 추가하는 유연성이 필요하면 자료 구조와 절차적인 코드가 더 적합하다. 우수한 소프트웨어 개발자는 편견 없이 이 사실을 이해해 직면한 문제에 최적인 해결책을 선택한다.

# 12장. 창발성 p.216

1. [창발적 설계로 깔끔한 코드를 구현하자](#1)
1. [단순한 설계 규칙 1: 모든 테스트를 실행하라](#2)
1. [단순한 설계 규칙 2: 리팩터링](#3)
1. [중복을 없애라](#4)
   1. [중복 코드 추출 예제](#4-1)
   1. [Template method 예제](#4-2)
1. [표현하라](#5)
1. [클래스와 메소드 수를 최소로 줄여라 p.222](#6)
1. [결론](#7)

## <a name = '1'> 창발적 설계로 깔끔한 코드를 구현하자 p.216 </a>

- 착실하게 따르기만 하면 우수한 설계가 나오는 간단한 규칙 네 가지가 있다면?
- 켄트 벡이 제시한 단순한 설계 규칙 네 가지가 소프트웨어 설계 품질을 크게 높여준다고 믿는다.

1. 모든 테스트를 실행한다.
1. 중복을 없앤다.
1. 프로그래머 의도를 표현한다.
1. 클래스와 메소드 수를 최소로 줄인다.

- 위 목록은 중요도 순이다.

## <a name = '2'> 단순한 설계 규칙 1: 모든 테스트를 실행하라 p.216 </a>

- 설계는 의도한 대로 돌아가는 시스템을 내놓아야 한다.
- 테스트가 불가능한 시스템은 검증도 불가능하기에 그런 시스템은 출시하면 안 된다.
- 테스트가 가능한 시스템을 만들려고 애쓰면 설계 품질이 더불어 높아진다.
- 결합도가 높으면 테스트 케이스를 작성하기 어렵기에 개발자는 DIP를 적용한다.
- 또한 DI, 인터페이스, 추상화 등과 같은 도구를 사용해 결합도를 낮추면 설계 품질이 더욱 높아진다.
- '테스트 케이스를 만들고 계속 돌려라'는 단순한 규칙이지만 따를수록 낮은 결합도와 높은 응집력을 가지는 목표를 달성하게 된다.

## <a name = '3'> 단순한 설계 규칙 2: 리팩터링 p.217 </a>

- 테스트 케이스를 모두 작성했다면 이제 코드와 클래스를 정리해도 좋다.
- 새로 추가하는 코드가 설계 품질을 낮추는가? 그렇다면 정리한 후 테스트 케이스를 돌려 기존 기능을 깨뜨리지 않았다는 사실을 확인하자
- **코드를 정리하면 시스템이 깨질까 걱정할 필요가 없다. 테스트 케이스가 있으니까!**
- 리팩터링 단계에서는 소프트웨어 설계 품질을 높이는 기법이라면 무엇이든 사용해도 좋다.
- 응집도를 높이고, 결합도를 낮추고, 관심사를 분리하고, 시스템 관심사 모듈로 나누고, 함수와 클래스 크기를 줄이고, 더 나은 이름 선택 모든 방법을 동원한다.

## <a name = '4'> 중복을 없애라 p.217 </a>

- 우수한 설계에서 중복은 커다란 적이다.

```java
int size() {
boolean isEmpty() {}
}
```

- 집합 클래스에 다음 메소드가 있다는 가정한다.

```java
boolean isEmpty() {
    return 0 == size();
}
```

### <a name = '4-1'> 중복 코드 추출 예제 </a>

- isEmpty를 구현할 때 다음과 같이 size 메소드를 이용하게 되면 코드를 중복해 구현할 필요가 없어진다.

```java
public void scaleToOneDimension(float desiredDimension, float imageDimension) {
  if (Math.abs(desiredDimension - imageDimension) < errorThreshold)
    return;
  float scalingFactor = desiredDimension / imageDimension;
  scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);
  
  RenderedOpnewImage = ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor);
  image.dispose();
  System.gc();
  image = newImage;
}

public synchronized void rotate(int degrees) {
  RenderedOpnewImage = ImageUtilities.getRotatedImage(image, degrees);
  image.dispose();
  System.gc();
  image = newImage;
}
```

- 위 두 함수 내부에 일부 동일한 로직이 존재한다.

```java
public void scaleToOneDimension(float desiredDimension, float imageDimension) {
  if (Math.abs(desiredDimension - imageDimension) < errorThreshold)
    return;
  float scalingFactor = desiredDimension / imageDimension;
  scalingFactor = (float) Math.floor(scalingFactor * 10) * 0.01f);
  replaceImage(ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor));
}

public synchronized void rotate(int degrees) {
  replaceImage(ImageUtilities.getRotatedImage(image, degrees));
}

private void replaceImage(RenderedOp newImage) {
  image.dispose();
  System.gc();
  image = newImage;
}
```

- 공통적인 로직을 메소드로 따로 빼서 중복을 제거할 수 있다.
- 클래스가 SRP를 위반하게 되기에 새 메소드는 다른 클래스로 옮겨줘도 된다.

### <a name = '4-2'> Template method 예제 </a>

```java
public class VacationPolicy {
  public void accrueUSDDivisionVacation() {
    // 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
    // ...
    // 휴가 일수가 미국 최소 법정 일수를 만족하는지 확인하는 코드
    // ...
    // 휴가 일수를 급여 대장에 적용하는 코드
    // ...
  }
  
  public void accrueEUDivisionVacation() {
    // 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
    // ...
    // 휴가 일수가 유럽연합 최소 법정 일수를 만족하는지 확인하는 코드
    // ...
    // 휴가 일수를 급여 대장에 적용하는 코드
    // ...
  }
}
```

- 최소 법정 일수를 계산하는 코드만 제외하면 두 메소드는 거의 동일하다.

```java
abstract public class VacationPolicy {
  public void accrueVacation() {
    caculateBseVacationHours();
    alterForLegalMinimums();
    applyToPayroll();
  }
  
  private void calculateBaseVacationHours() { /* ... */ };
  abstract protected void alterForLegalMinimums();
  private void applyToPayroll() { /* ... */ };
}

public class USVacationPolicy extends VacationPolicy {
  @Override protected void alterForLegalMinimums() {
    // 미국 최소 법정 일수를 사용한다.
  }
}

public class EUVacationPolicy extends VacationPolicy {
  @Override protected void alterForLegalMinimums() {
    // 유럽연합 최소 법정 일수를 사용한다.
  }
}
```

- 최소 법정 일수를 계산하는 알고리즘을 추상 메소드로 두고 유형에 맞게 서브 클래스에서 구현한다.

## <a name = '5'> 표현하라 p.221 </a>

- 자신이 이해하는 코드를 짜기는 쉽지만, 나중에 타인이 코드를 유지 보수할 때 문제를 깊이 이해할 가능성은 희박하다.
- 그러므로 코드는 개발자의 의도를 분명히 표현해야 한다.

1. 좋은 이름을 선택한다. 이름과 기능이 완전히 딴판인 클래스나 함수는 안된다.
1. 함수와 클래스 크기를 가능한 줄인다. 작은 클래스와 작은 함수는 이름도 짓기 쉽고, 구현하기도 쉽고, 이해하기 쉽다.
1. 표준 명칭을 사용한다. 디자인 패턴은 의사소통과 표현력 강화가 주요 목적이다. 클래스 이름에 패턴 이름을 넣어주면 설계 의도로 파악하기 쉽다.
1. 단위 테스트 케이스를 꼼꼼히 작성한다. 잘 만든 테스트 케이스를 읽어보면 클래스 기능이 한눈에 들어온다.

## <a name = '6'> 클래스와 메소드 수를 최소로 줄여라 p.222 </a>

- 중복을 제거하고, 의도를 표현하고, SRP를 준수한다는 기본적인 개념도 극단으로 치달으면 득보다 실이 많아진다.
- 함수와 클래스 수를 **가능한** 줄이라고 제안하는 것이다.

## <a name = '7'> 결론 p.222 </a>

- 경험을 대신할 단순한 개발 기법은 존재하지 않는다.
- 단순한 설계 규칙을 따른다면 우수한 기법과 원칙을 단번에 활용할 수 있다.
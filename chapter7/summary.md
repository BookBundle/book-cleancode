# 7장. 오류 처리 p129
오류 처리는 중요하다. 하지만 오류 처리 코드로 인해 프로그램 논리를 이해하기 어려워진다면 깨끗한 코드라 부르기 어렵다.

## 목차

1. [오류 코드보다 예외를 사용하라](오류-코드보다-예외를-사용하라-p130)
1. [Try-Catch-Finally 문부터 작성하라](Try-Catch-Finally문부터-작성하라-p132)
1. [미확인 예외를 사용하라](미확인-예외를-사용하라-p133)
1. [예외에 의미를 제공하라](예외에-의미를-제공하라-p135)
1. [호출자를 고려해 예외 클래스를 정의하라](호출자를-고려해-예외-클래스를-정의하라-p135)
1. [정상 흐름을 정의하라](정상-흐름을-정의하라-p137)
1. [null을 반환하지 마라](null을-반환하지-마라-p138)
1. [null을 전달하지 마라](null을-전달하지-마라-p140)
1. [결론](결론-p142)

## 오류 코드보다 예외를 사용하라 p130
- 얼마 전까지만 해도 예외를 지원하지 않는 프로그래밍 언어가 많았다.
- 오류 플래그를 설정하거나 호출자에게 오류 코드를 반환하는 방법이 전부였다.

```java
// bad
public class DeviceController {
...
public void sendShutDown() {
  DeviceHandle handle = getHandle(DEV1);
  // Check the state of the device
  if (handle != DeviceHandle.INVALID) {
    // Save the device status to the record field
    retrieveDeviceRecord(handle);
    // If not suspended, shut down
    if (record.getStatus() != DEVICE_SUSPENDED) {
      pauseDevice(handle);
      clearDeviceWorkQueue(handle);
      closeDevice(handle);
    } else {
      logger.log("Device suspended. Unable to shut down");
    }
  } else {
    logger.log("Invalid handle for: " + DEV1.toString());
  }
}
...
}
```

- 위와 같은 방법으로 사용하면 함수를 호출한 즉시 오류를 확인해야 하므로 호출자 코드가 복잡해진다.

```java
// good
public class DeviceController {
...
public void sendShutDown() {
  try {
    tryToShutDown();
  } catch (DeviceShutDownError e) {
    logger.log(e);
  }
}

private void tryToShutDown() throws DeviceShutDownError {
  DeviceHandle handle = getHandle(DEV1);
  DeviceRecord record = retrieveDeviceRecord(handle);
  pauseDevice(handle); 
  clearDeviceWorkQueue(handle); 
  closeDevice(handle);
}

private DeviceHandle getHandle(DeviceID id) {
  ...
  throw new DeviceShutDownError("Invalid handle for: " + id.toString());
  ...
}
...
}
```
- 실제 로직과 오류를 처리하는 알고리즘을 분리했기에 각 개념을 독립적으로 살펴보고 이해할 수 있다.

## Try-Catch-Finally문부터 작성하라 p132
- try블록에서 무슨 일이 생기든지 catch블록은 프로그램 상태를 일관성 있게 유지해야 한다.
- 예외가 발생할 코드를 짤 때는 try-catch-finally 문으로 시작하는 편이 낫다.

<br>

> TDD 방식으로 메소드 구현
1. 단위 테스트를 만든다.
```java
 @Test(expected = StorageException.class)
 public void retrieveSectionShouldThrowOnInvalidFileName() {
     sectionStore.retrieveSection("invalid - file");
 }
 ```
2. 단위 테스트에 맞춰 코드를 구현한다.
 ```java
  public List<RecordedGrip> retrieveSection(String sectionName) {
     // 실제로 구현할 때까지 비어 있는 더미를 반환한다.
     return new ArrayList<RecordedGrip>();
 }
 ```
예외가 발생하지 않기 때문에 단위 테스트에서 실패한다.

3. 파일 접근을 시도하도록 구현한다.
 ```java
 public List<RecordedGrip> retrieveSection(String sectionName) {
     try {
         FileInputStream stream = new FileInputStream(sectionName);
     } catch (Exception e) {
         throw new StorageException("retrieval error", e);
     }
     return new ArrayList<RecordedGrip>();
 }
 ```
테스트가 성공할 것이다.

4. 리펙터링이 가능해졌다.
 ```java
  public List<RecordedGrip> retrieveSection(String sectionName) {
     try {
         FileInputStream stream = new FileInputStream(sectionName);
         stream.close();
     } catch (FileNotFoundException e) {
         throw new StorageException("retrieval error", e);
     }
     return new ArrayList<RecordedGrip>();
 }
 ```
- 나머지 논리는 `FileInputStream`을 생성하는 코드와 close 호출문 사이에 넣으며, 오류나 예외가 전혀 발생하지 않는다고 가정한다.
- 먼저 강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방법을 권장한다.

## 미확인 예외를 사용하라 p133
- 예전에는 메서드를 선언할 때 메서드가 반환할 예외를 모두 열거했다.
- 그러나 비용이 상응하는 이익을 제공하지 못했고 확인된 예외는 OCP(Open Closed Principle)를 위반한다.
- 메스드에서 확인된 예외를 던졌는데 catch블록이 세 단계 위에 있다면 그 사이 메서드의 선언부에 해당 예외를 정의해야 한다.
- 만약 이런 변경이 최하단의 메서드에서 일어났다면 해당 메서드를 호출하는 함수 모두가
1. catch블록에서 새로운 예외를 처리하거나
1. 선언부에 throw절을 추가해야한다.

- 아주 중요한 라이브러리를 작성한다면 모든 예외를 잡아야하는데 이럴때는 확인된 예외가 유용하다.
- 하지만 일반적인 애플리케이션은 의전성이라는 비용이 이익보다 크다.

## 예외에 의미를 제공하라 p135
- 예외를 던질 때는 전후 상황을 충분히 덧붙인다. 오류가 발생한 원인과 위치를 찾기 쉬워진다.
- 오류 메시지에 정보를 담아 예외와 함께 던진다. 실패한 연산 이름과 실패 유형도 언급한다.

## 호출자를 고려해 예외 클래스를 정의하라 p135
- 오류를 분류하는 방법은 수없이 많다. 오류가 발생한 위치로 분류가 가능하다.
- 하지만 오류를 정의할 때 가장 중요한 관심사는 **오류를 잡아내는 방법**이 되어야 한다.
- 다음은 오류를 형편없이 분류한 사례다. 외부 라이브러리를 호출하는 try-catch-finally 문을 포함한 코드로, 라이브러리가 던질 예외를 모두 잡아 낸다.

```java
ACMEPort port = new ACMEPort(12);

try {
    port.open();
} catch (DeviceResponseException e) {
    reportPortError(e);
    logger.log("Device response exception", e)
} catch (ATM1212UnlockedException e) {
    reportPortError(e);
    logger.log("Unlock excepion", e)
} catch (GMXError e) {
    reportPortError(e);
    logger.log("Device response exception")
} finally {
    ...
}
```
- 대다수 상황에서 오류를 처리하는 방식은 1. 오류를 기록하고, 2. 프로그램을 계속 수행해도 좋은지 확인한다.
- 위 코드는 예외에 대응하는 방식이 예외 유형과 무관하게 거의 동일하다.
- 호출하는 라이브러리 API를 감싸면서 예외 유형 하나를 반환하면 된다.

```java
LocalPort port = new LocalPort(12);

try {
    port.open();
} catch (PortDeviceFailure e) {
    reportError(e);
    logger.log(e.getMessage(), e);
}
```
- 여기서 `LocalPort` 클래스는 단순히 `ACMEPort` 클래스가 던지는 예외를 잡아 변환하는 `감싸기(wrapper)` 클래스일 뿐이다.
- `Wrapper 클래스`를 사용하면 의존성이 크게 줄어들고 나중에 다른 라이브러리로 갈아타도 비용이 적다.

```java
LocalPort port = new LocalPort(12);
try {
    port.open();
}
catch (PortDeviceFailure e) {
    reportError(e);
    logger.log(e.getMessage(), e);
}
finally {
    ...
}

public class LocalPort {
    private ACMEPort innerPort;
    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }

    public void open() {
        try {
            innerPort.open();
        }
        catch (DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        }
        catch (ATM1212UnlockedException e) {
            throw new PortDeviceFailure(e);
        }
        catch (GMXError e) {
            throw new PortDeviceFailure(e);
        }
    }
}
```

## 정상 흐름을 정의하라 p137
- 비즈니스 논리와 오류 처리가 잘 분리도니 코드가 나온다면 오류 감지가 프로그램 언저리로 밀려난다.
- 외부 API를 감싸 독자적인 예외를 던지고, 코드 위에 처리기를 정의해 중단된 계산을 처리하는 방법이 멋진 처리 방식이지만 때로는 중단이 적합하지 않은 때도 있다.
- 다음은 비용 청구 애플리케이션에서 총계를 계산하는 허술한 코드다.

```java
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch (MealExpensesNotFound e) {
    m_total += getMealPerDiem();
}
```
- 식비를 비용으로 청구했다면 직원이 청구한 식비를 총계에 더한다.
- 식비를 비용으로 청구하지 않았다면 일일 시본 식비를 총계에 더한다.
- 그런데 예외가 논리를 따라가기 어렵게 만든다.
- 특수 상황을 처리할 필요가 없다면 코드가 훨씬 더 간결해진다.

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```
- `ExpenseReportDAO`를 고쳐 언제나 `MealExpense`객체를 반환한다. 청구한 식비가 없다면 일일 기본 식비를 반환하는 `MealExpense`객체를 반환한다.

```java
public class PerDiemMealExpenses implements MealExpense {
    public int getTotal() {
        // 기본값으로 일일 기본 식비를 반환한다.
    }
}
```
- 이를 **특수 사례 패턴 SPECIAL CASE PATTERN** 이라 부른다.
- 클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식이다.
- 클래스나 객체가 예외적인 상황을 캡슐화해서 처리하므로 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다.

## null을 반환하지 마라 p138
- 우리가 흔히 저지르는 바람에 오류를 유발하는 행위 중 하나가 바로 **null을 반환하는 습관** 이다.
- 다음은 한 예다.

```java 
// 한 줄 건너 하나씩 null을 확인하는 코드
public void registerItem(Item item) {
    if (item != null) {
        ItemRegistry registry = peristentStore.getItemRegistry();
        if (registry != null) {
            Item existing = registry.getItem(item.getID());
            if (existing.getBillingPeriod().hasRetailOwner()) {
                existing.register(item);
            }
        }
    }
}
```
- 메서드에서 null을 반환하고 싶은 생각이 들면 위의 _특수 사례 패턴_ 을 반환하거나 예외를 던져라.
- 사용하려는 외부 API가 null을 반환한다면 감싸기 메서드를 구현해 예외를 던지거나 특수 사례 객체를 반환하는 방식을 고려한다.

```java
// good
List<Employee> employees = getEmployees();
for(Employee e : employees) {
  totalPay += e.getPay();
}

public List<Employee> getEmployees() {
  if( .. 직원이 없다면 .. )
    return Collections.emptyList();
  }
}
```
- 이렇게 코드를 변경하면 코드도 깔끔해질뿐더러 NullPointerException이 발생할 가능성도 줄어든다.

## null을 전달하지 마라 p140
- 메서드에서 null을 반환하는 방식도 나쁘지만 메서드로 null을 전달하는 방식은 더 나쁘다.
- 정상적인 인수로 null을 기대하는 api가 아니라면 메서드로 null을 전달하는 코드는 최대한 피한다.

```java
// bad
public class MetricsCalculator {
  public double xProjection(Point p1, Point p2) {
    return (p2.x – p1.x) * 1.5;
  }
  ...
}
```
- 누군가 인수로 `calculator.xProjection(null, new Point(12, 13));` 와 같은 코드로 null을 전달하면 `NullPointerException`이 발생한다.

```java
// bad
public class MetricsCalculator {
  public double xProjection(Point p1, Point p2) {
    if(p1 == null || p2 == null){
      throw InvalidArgumentException("Invalid argument for MetricsCalculator.xProjection");
    }
    return (p2.x – p1.x) * 1.5;
  }
}
```
- `NullPointerException`은 발생하지 않지만 `InvalidArgumentException`을 잡아내는 처리기가 필요하다.

```java
// bad
public class MetricsCalculator {
  public double xProjection(Point p1, Point p2) {
    assert p1 != null : "p1 should not be null";
    assert p2 != null : "p2 should not be null";
    
    return (p2.x – p1.x) * 1.5;
  }
}
```
- 문서화가 잘 되어 있어 코드 읽기는 편하지만 `NullPointerException` 문제를 해결하지는 못한다.

<br>

- 대다수 프로그래밍 언어는 호출자가 실수로 null을 넘기는 것을 처리하는 방법이 없다.
- 그러니 애초에 null을 넘기지 못하도록 금지하는 정책이 합리적이다.

## 결론 p142
- 깨끗한 코드는 읽기도 좋아야 하지만 안정성도 노팡야 한다.
- 오류 처리를 프로그램 논리와 분리해 독자적인 사안으로 고려하면 튼튼하고 깨끗한 코드를 작성할 수 있다.
- 오류 처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아진다.
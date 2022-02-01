# 16장. SerialDate 리팩터링 p.343

1. [첫째, 돌려보자](#1)
1. [둘째, 고쳐보자](#2)
   1. [변경 이력 제거](#2-1)
   1. [import문 정리](#2-2)
   1. [HTML 주석 지양](#2-3)
   1. [적합한 클래스 이름 짓기](#2-4)
   1. [DayDate(SerialDate)가 MonthConstant를 사용하는 이유가 무엇일까?](#2-5)
   1. [serialVersionUID 변수 제거](#2-6)
   1. [불필요한 주석 제거](#2-6)
   1. [적합한 변수 이름 짓기](#2-7)
   1. [추상 클래스 내에 구현 정보 제거](#2-8)


- JCommon 라이브러리를 내에 org.jfree.date라는 패키지 속에 SerialDate라는 클래스를 탐험한다.
- SerialDate는 java.util.Date, java.util.Calendar와 같이 날짜를 표현하는 클래스이다.
- SerialDate는 Date와 같이 정밀한 시간 기반 날짜 클래스보다 순수 날짜 클래스를 만들고자 구현한 것이다.

## <a name = '1'> 첫째, 돌려보자 p.345 </a>

- SerialDateTests라는 클래스는 단위 테스트 케이스 몇 개를 포함한다. (p.473)
- 실패하는 테스트 케이스는 없지만 훑어보면 모든 경우를 점검하지 않는 사실이 드러난다.
- 클로버(정적 분석 도구)를 이용해 실행하는 코드와 실행하지 않는 코드를 분석하였고 185개 중 단위 테스트를 실행하는 문장은 91개로 대략 50%였다.
- 그래서 **독자적으로 단위 테스트 케이스를 구현**하였다. (p.484)
- 대부분의 코드를 주석으로 처리하였고 이들은 실패한 테스트 케이스였다. 그 결과 185개 중 170개를 실행하여 코드 커버리지가 대략 92%에 이르렀다.
- p.484 23행 ~ p.485 63행 주석이 마음에 걸린다.
- stringToWeekDayCode를 왜 만들었는지 모르겠다.
- 저자는 대소문자 구분 없이 모두 통과해야 한다고 봤다. (p.454 259행, 263행)
- tues, thurs 약어를 지원해야 할지 불분명하였다. (p.484 32행, p485 45행)

```java
if  ((result < 1) || result > 12) {
    result = -1;
    for (int i = 0; i < maonthNames.length; i++) {
        // if (s.equals(shortMonthNames[i])) {
        if (s.equalsIgnoreCase(shortMonthNames[i])) {
            result = i + 1;
            break;
        }
        // if (s.equals(monthNames[i])) {
        if (s.equalsIgnoreCase(monthNames[i])) {
            result = i + 1;
            break;
        }
    }
}
```

- 저자의 의도대로 고친 stringToWeekCode이다.
- 아래 코드는 getFollowingDayOfWeek 메소드(p.463 672행)에 있는 버그(p.491 318행)나 나타난다.
- 2004년 12월 25일은 토요일이고 다음 토요일은 2005년 1월 1일이지만 테스트를 돌리면 12월 25일을 다음 토요일로 반환한다.

```
assertEquals(d(1, JANUARY, 2005), getFollowingDayOfWeek(SATURDAY, d(25, DECEMBER, 2004)));
```

- 명백한 버그이고 이것은 전형적인 경계 조건 오류이다.

```java
// if (baseDOW > targetWeekday) {
if (baseDow >= targetWeekday) { 
```

- 올바른 코드는 위와 같다.
- 놀라운 사실은 이 부분은 이전에 한 번 손봤다는 사실이다.

```java
 * 12-Nov-2001 : IBD requires setDescription() method, now that NotableDate 
 *               class is gone (DG);  Changed getPreviousDayOfWeek(), 
 *               getFollowingDayOfWeek() and getNearestDayOfWeek() to correct 
 *               bugs (DG);
```

- getTestNearestDayOfWeek 메소드(p.464 705행)를 단위 테스트하는 메소드(p.491 329행)는 처음부터 이렇게 길지 않았고 처음 구현한 테스트 케이스가 계속 실패하기에 추가된 것이다.
- 주석으로 처리한 코드를 살펴보면 실패하는 패턴이 보인다.
- 알고리즘은 가장 가까운 날짜가 미래면 실패한다. 역시 앞선 내용과 같은 경계 조건 오류이다.
- adjust >= 4 if문은 항상 거짓이고 실행되지 않는다. adjust는 항상 음수이기 때문이다.
- 아래는 알고리즘을 올바르게 고친 코드이다.

```java
// final int baseDOW = base.getDayOfWeek();
// int adjust = -Math.abs(targetDOW - baseDOW);
int delta = targetDOW - base.getDayOfWeek();
int positiveDelta = delta + 7;
int adjust = positiveDelta % 7;
// if (adjust >= 4) {
//     adjust = 7 - adjust;
// }
// if (adjust <= -4) {
//     adjust = 7 + adjust;
// }
if (adjust > 3)
    adjust -= 7;
return SerialDate.addDays(adjust, base);
```

## <a name = '2'> 둘째, 고쳐보자 p.347 </a>

### <a name = '2-1'> 변경 이력 제거 </a>

- p.448 1행
- 라이선스 정보, 저작권, 작성자, 변경 이력이 나온다. 법적인 정보와 라이선스 정보, 저작권은 보존한다.
- 반면 **변경 이력은 1960년대에 나온 방식이므로 삭제해도 무방**하다.

### <a name = '2-2'> import문 정리 </a>

- p.449 61행
- 여러 개의 import 문은 java.text.*와 java.util.*로 줄여도 된다.

### <a name = '2-3'> HTML 주석 지양 </a>

- p.449 67행
- javadoc 주석은 HTML 태그를 사용한다. 한 소스에 여러 개의 언어가 사용된다. 자바, 영어, HTML, javadoc
- HTML을 주석에서 사용하는 것은 지양한다.
- 차라리 모든 주석은 <pre>로 묶으면 소스 코드에 보이는 형식도 javadoc에 유지가 되고 좋다.

### <a name = '2-4'> 적합한 클래스 이름 짓기 </a>

- p.450 86행
- 클래스 이름이 SerialDate인 이유가 무엇일까?
- 이유는 일련번호를 사용해 클래스를 구현했기 때문이다. 여기서 1899년 12월 30일 기준으로 지나간 날짜 수를 이용한다.
- 두 가지가 꺼림칙하다.
- 첫 번째, 일련번호라는 용어는 정확하지 못하다. 일련번호는 날짜보다는 제품 식별 번호로 더 적합하다. 그렇기에 SerialDate는 서술적인 이름이 아니라 생각한다.
- 두 번째, SerialDate 이름은 구현을 암시하는데 실상 추상 클래스이다. 이름이 추상화 수준에 어울리지 않고 그냥 Date가 좋다.
- 하지만 Date는 자바 라이브러리에 이미 많이 사용되고 Day 또한 그렇기에 DayDate로 사용하기로 하였다.

### <a name = '2-5'> DayDate(SerialDate)가 MonthConstant를 사용하는 이유가 무엇일까? </a>

- Comparable, Serializable을 상속하는 이유는 알겠다.
- MonthContants(p.482)를 보게 되면 달을 정의하는 static final 상수의 모음이다.
- 그래서 상수 클래스를 상속받으면 MonthConstants.January와 같은 표현을 할 필요 없기 때문이다.
- 하지만 이런 방식은 옛날 방식이고 바람직하다고 보기 어렵기에 **MonthConstants는 enum으로 정의해야 마땅하다.**

```java
public abstract class DayDate implements Comparable, Serializable {
    public static enum Month {
        JANUARY(1),
        FEBRUARY(2),
        MARCH(3),
        APRIL(4),
        MAY(5),
        JUNE(6),
        JULY(7),
        AUGUST(8),
        SEPTEMBER(9),
        OCTOBER(10),
        NOVEMBER(11),
        DECEMBER(12);

        Month(int index) {
            this.index = index;
        }

        public static Month make(int monthIndex) {
            for (Month m : Month.values()) {
                if (m.index == monthIndex)
                    return m;
            }
            throw new IllegalArgumentException("Invalid month index " + monthIndex);
        }
        public final int index;
    }
}
```

- enum으로 변경하면서 달을 int로 받던 메소드는 이제 Month로 받는다.
- isValidMonthCode, monthCodeToQuarter와 같은 오류 검사 코드들은 더 이상 필요로 하지 않는다.

### <a name = '2-6'> serialVersionUID 변수 제거 </a>

- p.450 91행
- serialVersionUID 변수는 직렬화를 제어한다.
- 이 변수 값을 변경하면 이전 소프트웨어 버전에서 직렬화한 DayDate를 더 이상 인식하지 못한다.
- 이 값을 선언하지 않으면 컴파일러가 자동으로 생성한다. 즉 모듈을 변경할 때마다 변수 값이 달라진다.
- 문서에서는 직접 선언하라고 권하지만, 나로서는 자동 제억 훨씬 더 안전하게 여겨지기에 변수를 삭제한다.

### <a name = '2-7'> 불필요한 주석 제거 </a>

- p.450 93행
- 불필요해 보이는 주석은 삭제한다.

### <a name = '2-8'> 적합한 변수 이름 짓기 </a>

- p.450 97, 100행
- 주석은 앞서 설명한 일련번호를 언급한다.
- DayDate 클래스가 표현할 수 있는 최초 날짜와 최후 날짜를 의미하기에 좀 더 깔끔하게 표현한다.

```java
// public static final int SERIAL_LOWER_BOUND = 2;
public static final int EARLIEST_DATE_ORDINAL = 2;      // 1/1/1900
// public static final int SERIAL_UPPER_BOUND = 2958465;
public static final int LATEST_DATE_ORDINAL = 2958465;  // 12/31/9999
```

- EARLIEST_DATE_ORDINAL이 0이 아니라 2인 이유는 SpreadsheetDate 클래스(p.496 71행)를 살펴보면 이해한다.
- 위에 언급한 두 변수는 SpreadsheetDate 클래스만 두 변수를 사용하고 그 외에 JCommon 클래스는 두 변수를 사용하지 않는다.
- 그렇기에 두 변수를 SpreadsheetDate 클래스로 옮겼다.

### <a name = '2-9'> 추상 클래스 내에 구현 정보 제거 </a>

- p.450 104, 107행
- MINIMUM_YEAR_SUPPORTED와 MAXIMUM_YEAR_SUPPORTED(p.450 104, 107행) 두 변수는 DayDate는 어디까지나 추상 클래스이기 때문에 구체적인 구현 정보를 포함할 필요가 없다.
- SpreadsheetDate 클래스로 옮기려고 했으나 RelativeDayOfWeekRule이라는 클래스에서도 두 변수를 사용한다. (p.511 177, 178행)
- getDate로 넘어온 인수 year이 올바른지 확인할 목적으로 사용한다. 즉, 추상 클래스 사용자가 구현 정보를 알아야 한다는 딜레마가 생긴다.
- DayDate를 훼손하지 않으면서 구현 정보를 전달할 방법이 필요하다. 일반적으로 파생 클래스의 인스턴스로부터 구현 정보를 가져온다.
- getDate 메소드로 넘어오는 인자는 DayDate 인스턴스가 아니다. 하지만 getDate 메소드는 DayDate 인스턴스를 반환한다.
- -> p.511 187~205 RelativeDayOfWeekRule.java DayDate getDate() {} 
- -> p.462~464 DayDate.java DayDate get...Week() {}
- -> p.461 DayDate.java DayDate addDays() {}
- -> p.466 DayDate.java 808 DayDate createInstance() { return new SpreadsheetDate() }
- 기반 클래스는 파생 클래스를 몰라야 바람직하기에 추상 팩토리 패턴을 적용하였다.

```java
public abstract class DayDateFactory {
    private static DayDateFactory factory = new SpreadsheetDateFactory();
    public static viud setInstance(DayDateFactory factory) {
        DayDateFactory.factory = factory;
    }

    protected abstract DayDate _makeDate(int ordinal);
    protected abstract DayDate _makeDate(int day, DayDate.Month month, int year);
    protected abstract DayDate _makeDate(int day, int month, int year);
    protected abstract DayDate _makeDate(java.util.Date date);
    protected abstract int _getMinimumYear();
    protected abstract int _getMaximumYear();

    public static DayDate makeDate(int ordinal) {
        return factory._makeDate(ordinal);
    }

    public static DayDate makeDate(int day, DayDate.Month month, int year) {
        return factory._makeDate(day, month, year);
    }

    public static DayDate makeDate(int day, int month, int year) {
        return factory._makeDate(day, month, year);
    }

    public static DayDate makeDate(java.util.Date date) {
        return factory._makeDate(date);
    }

    public static int getMinimumYear() {
        return factory._getMinimumYear();
    }

    public static int getMaximumYear() {
        return factory._getMaximumYear();
    }
}
```

- MINIMUM_YEAR_SUPPORTED와 MAXIMUM_YEAR_SUPPORTED 두 변수를 적절한 위치에 이동시켰다.

```java
public class SpreadsheetDateFactory extends DayDateFactory {
    public DayDate _makeDate(int ordinal) {
        return new SpreadsheetDate(ordinal);
    }

    public DayDate _makeDate(int day, DayDate.Month month, int year) {
        return new SpreadsheetDate(day, month, year);
    }

    public DayDate _makeDate(int day, int month, int year) {
        rturn new SpreadsheetDate(day, month, year);
    }

    public DayDate _makeDate(Date date) {
        final GregorianCalendar calendar = new GregorianCalendar();
        calendar.setTime(date);
        return new SpreadsheetDate(
            calendar.get(Calendar.DATE),
            DayDate.Month.make(calendar.get(Calendar.MONTH) + 1),
            calendar.get(Calendar.YEAR));
    }

    protected int _getMinimumYear() {
        return SpreadsheetDate.MINIMUM_YEAR_SUPPORTED;
    }

    protected int _getMaximumYear() {
        return SpreadsheetDate.MAXIMUM_YEAR_SUPPORTED;
    }
}
```
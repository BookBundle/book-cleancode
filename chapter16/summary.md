# 16장. SerialDate 리팩터링 p.343

1. [첫째, 돌려보자](#1)
1. [둘째, 고쳐보자](#2)
   1. [변경 이력 제거](#2-1)
   1. [import문 정리](#2-2)
   1. [HTML 주석 지양](#2-3)
   1. [적합한 클래스 이름 짓기](#2-4)
   1. [DayDate(SerialDate)가 MonthConstant를 사용하는 이유가 무엇일까?](#2-5)
   1. [serialVersionUID 변수 제거](#2-6)
   1. [불필요한 주석 제거](#2-7)
   1. [적합한 변수 이름 짓기](#2-8)
   1. [추상 클래스 내에 구현 정보 제거](#2-9)
   1. [요일 상수](#2-10)
   1. [변수 정리](#2-11)
   1. [enum 변환](#2-12)
   1. [적합한 변수 이름](#2-13)
   1. [enum 변환](#2-14)
   1. [불필요한 내용 제거](#2-15)
   1. [if문 병합](#2-16)
   1. [메소드 이동](#2-17)
   1. [메소드 병합](#2-18)
   1. [메소드 삭제](#2-19)
   1. [메소드 추가](#2-20)
   1. [메소드 인자로 플래그](#2-21)
   1. [메소드 이동 및 단순화](#2-22)
   1. [서술적인 표현](#2-23)
   1. [메소드 이동 및 단순화](#2-24)
   1. [static X](#2-25)
   1. [메소드 단순화](#2-26)
   1. [메소드 변경](#2-27)
   1. [메소드 삭제](#2-28)
   1. [메소드 이름 변경](#2-29)
   1. [반환 타입 변경](#2-30)
   1. [의존 관계](#2-31)
   1. [isInRange 리팩토링](#2-32)
1. [결론](#3)


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
- 차라리 모든 주석은 \<pre\>로 묶으면 소스 코드에 보이는 형식도 javadoc에 유지가 되고 좋다.

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
- 문서에서는 직접 선언하라고 권하지만, 나로서는 자동 제어가 훨씬 더 안전하게 여겨지기에 변수를 삭제한다.

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

### <a name = '2-10'> 요일 상수

- p.450 109행
- 앞에서 본 예제와 동일하게 enum으로 해야 한다.

### <a name = '2-11'> 변수 정리

- p.451 140행
- 각 배열을 나타내는 주석은 필요하지 않고 변수 이름만으로도 의미가 분명하기에 주석을 제거한다.
- LAST_DAY_OF_MONTH 변수는 lastDayOfMonth 정적 메소드에서만 사용하므로 public -> private으로 변경한다.
- AGGREGATE_DAYS_OF_MONTH는 Jcommon 프레임워크 어디서도 사용되지 않음으로 삭제한다.
- AGGREGATE_DAYS_~_MONTH는 SpreadsheetDate에서만 사용한다. 옮겨야 하지만 이 변수는 특정 구현에 의존하지 않음으로 최대한 가까운 곳으로 옮겼다.

### <a name = '2-12'> enum 변환

- p.451 162행 ~ p.452 205행
- 다음에 나오는 상수들은 enum으로 변환하고 첫 세 상수는 달에서 주를 선택한다.
- 그렇기에 WeekInMonth라는 enum으로 변환한다.

```java
public enum WeekInMonth {
    FIRST(1), SECOND(2), THIRD(3), FOURTH(4), LAST(0);
    public final int index;

    WeekInMonth(int index) {
        this.index = index;
    }
}
```

### <a name = '2-13'> 적합한 변수 이름

- p.452 177행 ~ 187행
- INCLUDE_NONE, INCLUDE_FIRST, INCLUDE_SECOND, INCLUDE_BOTH 상수는 범위 끝 날짜를 범위에 포함할지 여부를 지정한다.
- 수학적으로 개구간, 반개구간, 폐구간이라는 개념이고 좀 더 명확하게 표현하게 수학적 개념으로 변경하였다.
- CLOSED, CLOSED_LEFT, CLOSED_RIGHT, OPEN으로 변경하였다.

### <a name = '2-14'> enum 변환

- p.452 190행 ~ 205행
- 상수는 주어진 날짜를 기준으로 특정 요일을 계산할 때 사용한다.
- 직전 요일, 다음 요일, 가장 가까운 요일을 의미한다.
- LAST, NEXT, NEAREST로 정의하고 WeekdayRange라는 이름을 붙여 enum으로 만들었다.

### <a name = '2-15'> 불필요한 내용 제거

- description 변수를 더 이상 사용하지 않는 듯하여 삭제하였다. (p.452 208행)
- 기본 생성자도 컴파일러가 기본으로 생성하기에 삭제하였다. (p.452 213행)
- isValidWeekdayCode 메소드는 건너뛴다. 이미 앞에서 Day enum 생성 시 삭제하였다. (p.453 216행 ~ 238행)
- 부실한 주석은 코드를 복잡하게 한다. 반환 값 -1은 의미가 있지만 앞서 IllegalArgument Exception을 던지는 것으로 변경하였으니 주석을 제거한다. (p.453 242행 ~ p.454 270행)
- 또한, 실질적인 가치가 없는 코드를 복잡하게 만든다고 판단하였기에 final 키워드는 모두 삭제한다.

### <a name = '2-16'> if문 병합

- p.454 259행 ~ 263행
- for loop 안에 if문이 두 개가 나오는데 || 연산자를 통하여 if문 하나로 만들었다.

### <a name = '2-17'> 메소드 이동

- p.454 259행 ~ 263행
- stringToWeekdayCode 메소드는 DayDate 클래스에 속하지 않고 사실 Day의 구문분석 메소드이다.
- Day로 메소드를 옮기니 덩치가 커져서 DayDate 클래스에서 빼서 독자적인 클래스로 만들었다.
- 이동 후에 weekdayCodeToString 메소드도 Day로 옮기고 toString으로 변경하였다. (p.454 272행 ~ 286행)

```java
public enum Day {
    MONDAY(Calendar.MONDAY),
    TUESDAY(Calendar.TUESDAY),
    WEDNESDAY(Calendar.WEDNESDAY),
    THURSDAY(Calendar.THURSDAY),
    FRIDAY(Calendar.FRIDAY),
    SATURDAY(Calendar.SATURDAY),
    SUNDAY(Calendar.SUNDAY);

    public final int index;
    private static DateFormatSymbols dateSymbols = new DateFormatSymbols();

    Day(int day) {
        index = day;
    }

    public static Day make(int index) throws IllegalArgumentException {
        for (Day d : Day.values())
            if (d.index == index)
                return d;
        throw new IllegalArgumentException(
            String.format("Illegal day index: %d.", index));
    }

    public static Day parse(String s) throws IllegalArgumentException {
        String[] shortWeekdayNames = 
            dateSymbols.getShortWeekdays();
        String[] weekDayNames =
            dateSymbols.getWeekdays();
        
        s = s.trim();
        for (Day day : Day.values()) {
            if (s.equalsIgnoreCase(shortWeekdayNames[day.index])) ||
                s.equalsIgnoreCase(weekDayNames[day.index])) {
                return day;
            }
        }
        throw new IllegalArgumentException(
            String.format("%s is not a valid weekday string", s));
    }

    public String toString() {
        return dateSymbols.getWeekdays()[index];
    }
}
```

### <a name = '2-18'> 메소드 병합

- p.454 288행 ~ p.455 316행
- getMonths라는 메소드 두 개가 나온다. 첫 번째 메소드가 두 번째 메소드를 호출하는 구조이다.
- 그래서 두 메소드를 합치고 서술적인 이름으로 변경하였다.

```java
public static String[] getMonthNames() {
    return dateFormatSymbols.getMonth();
}
```

### <a name = '2-19'> 메소드 삭제

- p.455 326행
- isValidMonthCode 메소드는 Month enum을 만들면서 필요가 없어졌기에 삭제하였다.

### <a name = '2-20'> 메소드 추가

- p.456 356행
- monthCodeToQuarter 메소드는 기능 욕심으로 보이기에 대신 Month에 quarter라는 메소드를 추가하였다.

```java
public int quarter() {
    return 1 + (index-1)/3;
}
```

### <a name = '2-21'> 메소드 인자로 플래그

- p.456 377행 ~ p.457 426행
- monthCodeToString이라는 메소드 두 개가 나온다.
- 앞서 마찬가지로, 한 메소드가 다른 메소드를 호출하여 플래그를 넘기는데 이런 형식을 바람직하지 않다.
- 그렇기에 두 메소드 이름을 변경하고, 단순화하여, Month enum으로 옮겼다.

```java
public String toString() {
    return dateFormatSymbols.getMonths()[index - 1]; // return monthCodeTodString(month, false)
}

public String toShortString() {
    return dateFormatSymbols.getShortMonths()[index - 1];
}
```

### <a name = '2-22'> 메소드 이동 및 단순화

- p.457 428행 ~ p.459 472행
- stringToMonthCode 메소드 이름을 변경하고, Month enum으로 옮기고, 단순화하였다.

```java
public static Month parse(String s) {
    s = s.trim();
    for (Month m : Month.values())
        if (m.matches(s))
            return m;
    
    try {
        return make(Integer.parseInt(s));
    }
    catch (NumberFormatException e) {}
    throw new IllegalArgumentException("Invalid month " + s);
}

private boolean matches(String s) {
    return s.equalsIgnoreCase(toString()) ||
        s.equalsIgnoreCase(toShortString());
}
```

### <a name = '2-23'> 서술적인 표현

- p.459 495행 ~ 517행
- 윤년인지 판단하는 isLeapYear 메소드는 서술적인 표현으로 가독성을 높였다.

```java
public static boolean isLeapYear(int year) {
    boolean fourth = year % 4 == 0;
    boolean hundredth = year % 100 == 0;
    boolean fourHundredth = year % 400 == 0;
    return fourth && (!hundreth || fourHundredth);
}
```

### <a name = '2-24'> 메소드 이동 및 단순화

- p.460 538행 ~ 560행
- lastDayOfMonth 메소드는 LAST_DAY_OF_MONTH 배열을 사용한다.
- 이 배열은 Month enum에 속하므로 이동하고 메소드를 단순화하고 좀 더 서술적인 표현으로 고쳤다.

```java
public static int lastDayOfMonth(Month month, int year) {
    if (month == Month.FEBRUARY && isLeapYear(year))
        return month.lastDay() + 1;
    else
        return month.lastDay();
}
```

### <a name = '2-25'> static X

- p.461 562행 ~ 576행
- 이 메소드는 온갖 DayDate 변수를 사용하므로 static이어서는 안되기에 인스턴스 메소드로 변경하였다.
- toSerial 메소드를 호출하는데 toOrdinal로 변경하고 메소드를 단순화하였다.

```java
public DayDate addDays(int days) {
    return DayDateFactory.makeDate(toOrdinal() + days);
}
```

- p.461 578행 ~ p.462 602행
- addMonths 또한 마찬가지이므로 인스턴스 메소드로 변경하였다.
- 알고리즘이 다소 복잡하여 임시 변수 설명을 사용하여 좀 더 읽기 쉽게 변경하였고 getYYY라는 이름 또한 getYear로 변경하였다.

```java
public DayDate addMonths(int months) {
    int thisMonthAsOrdinal = 12 * getYear() + getMonth().index - 1;
    int resultMonthAsOrdinal = thisMonthAsOrdinal + months;
    int resultYear = resultMonthAsOrdinal / 12;
    Month resultMonth = Month.make(resultMonthAsOrdinal % 12 + 1);

    int lastDayOfResultMonth = lastDayOfMonth(resultMonth, resultYear);
    int resultDay = Math.min(getDayOfMonth(), lastDayOfResultMonth);
    return DayDateFactory.makeDate(reusltDay, resultMonth, resultYear);
}
```

- p.462 604행 ~ 626행
- addYears도 마찬가지이다.

```java
public DayDate plusYears(int years) {
    int resultYear = getYear() + years;
    int lastDayOfMonthInResultYear = lastDayOfMonth(getMonth(), resultYear);
    int resultDay = Math.min(getDayOfMonth(), lastDayOfMonthInResultYear);
    return DayDateFactory.makeDate(resultDay, getMonth(), resultYear);
}
```

- date.addDays(5)라는 표현이 date 객체를 변경하지 않고 새 DayDate 인스턴스를 반환한다는 사실이 드러날까?

```java
DayDate date = DateFactory.makeDate(5, Month.DECEMBER, 1952);
date.addDays(7); // 날짜를 일주일만큼 뒤로 미룬다.
```

- 코드를 읽는 사람은 분명히 addDays가 date 객체를 변경한다고 생각하기에 이런 모호함을 해결할 이름이 필요하다.
- 그래서 plushDays, plusMonths라는 이름을 선택했다.

```java
DayDate date = oldDate.plusDays(5);
```

### <a name = '2-26'> 메소드 단순화

- p.462 628행 ~ 660행
- getPreviousDayOfWeek 메소드는 올바르게 동작하지만 다소 복잡하다.
- 단순화하고 임시 변수 설명을 사용해 좀 더 읽기 쉽게 고쳤으며 정적 메소드를 인스턴스 메소드로 고쳤다.
- 또한, 중복된 인스턴스 메소드도 제거하였다. (p.471 997행 ~ 1008행)

```java
public DayDate getPreviousDayOfWeek(Day targetDayOfWeek) {
    int offsetToTarget = targetDayOfWeek.index - getDayOfWeek().index;
    if (offsetToTarget >= 0)
        offsetToTarget -= 7;
    return plusDays(offsetToTarget);
}
```

- p.463 662행 ~ p.464 693행
- getFollowingDayOfWeek 메소드도 같은 원리로 고쳤다.

```java
public DayDate getFollowingDayOfWeek(Day targetDayOfWeek) {
    int offsetToTarget = targetDayOfweek.index - getDayOfWeek().index;
    if (offsetToTarget <= 0)
        offsetToTarget += 7;
    return plusDays(offsetToTarget);
}
```

- p.464 695행 ~ 726행
- getNearestDayOfWeek 메소드도 같은 원리로 고쳤다.

```java
public DayDate getNearestDayOfWeek(final Day targetDay) {
    int offsetToThisWeeksTarget = targetDay.index - getDayOfWeek().index;
    int offsetToFutureTarget = (offsetToThisWeeksTarget + 7) % 7;
    int offsetToPreviousTarget = offsetToFutureTarget - 7;
    if (offsetToFutureTarget > 3)
        return plusDays(offsetToPreviousTarget);
    else
        return plusDays(offsetToFutureTarget);
}
```

### <a name = '2-27'> 메소드 변경

- p.464 728행 ~ p.465 740핸
- getEndOfCurrentMonth 메소드는 DayDate 인스턴스를 인자로 받아 DayDate 메소드를 호출하는 인스턴스 메소드이다.
- 진짜 인스턴스 메소드로 바꾸고 몇몇 이름을 변경했다.

```java
public DayDate getEndOfMonth() {
    Month month = getMonth();
    int year = getYear();
    int lastDay = lastDayOfMonth(month, year);
    return DayDateFactory.makeDate(lastDay, month, year);
}
```

### <a name = '2-28'> 메소드 삭제

- p.465 742행 ~ 761행
- weekInMonthToString은 WeekInMonth enum으로 옮긴 후 toString이라는 이름으로 변경하였고 정적 메소드를 인스턴스 메소드로 고쳤다.
- p.465 765행 ~ p.466 781행
- relativeToString 메소드 역시 테스트 케이스 외에는 아무도 호출하지 않았기에 전부 삭제하였다.

### <a name = '2-29'> 메소드 이름 변경

- p.467 829행 ~ 836행
- 이전 toOrdinal로 변경하였지만 getOrdinalDay이 더 적합하다 느껴져 변경하였다.

### <a name = '2-30'> 반환 타입 변경

- p.467 844행 ~ 846행
- toDate 메소드는 반환 타입을 DayDate를 java.util.Date로 변경한다.

### <a name = '2-31'> 의존 관계

- p.467 844행 ~ 846행
- SpreadsheetDate에서 구현한 toDate를 살펴보면 메소드는 SpreadsheetDate에 의존하지 않기에 DayDate로 끌어올린다. (p.499 198행 ~ 207행)
- getDayOfWeek 메소드도 SpreadsheetDate 구현에 의존하지 않으므로 DayDate로 올려도 될 것 같다. 그럴까? (p.500 246행)
- 구현 알고리즘을 보게 되면 서수 날짜 시작일의 요일에 암시적으로 의존한다. 즉 0번째 날짜의 요일에 의존하기에 해당 메소드는 DayDate로 옮길 수 없다. 논리적 의존성이 존재하기 때문이다.
- 그렇기에 DayDate에 getDayOfWeekForOrdinalZero라는 추상 메소드를 구현하고 SpreadsheetDate에서 Day.SATURDAY를 반환하도록 구현하였다.
- 그런 다음 getDayOfWeek 메소드를 DayDate로 옮긴 후 getOrdinalDay와 getDayOfWeekForOrdinalZero를 호출하도록 변경하였다.

```java
public int getDayOfWeek() {
    return (this.serial + 6) % 7 + 1;
}

public Day getDayOfWeek() {
    Day startingDay = getDayOfWeekForOrdinalZero(); // 물리적 의존성
    int startingOffset = startingDay.index - Day.SUNDAY.index;
    return Day.make((getOrdinalDay() + startingOffset) % 7 + 1);
}
```

- compare 메소드 또한 추상 메소드일 필요가 없다 (p.469 902행 ~ 913행)
- SpreadsheetDate에 있는 compare 메소드를 DayDate로 끌어올렸다.
- 그리고 이름도 불분명하다. 오늘 날짜와 차이를 일수로 반환하기에 이름을 daysSince로 변경하였고 테스트 케이스도 없어 추가하였다.
- p.469 915행 ~ p.470 980행까지 총 여섯 메소드는 모두 DayDate에서 구현해야 마땅한 추상 메소드이기에 SpreadsheetDate에서 DayDate로 끌어올렸다.

### <a name = '2-32'> isInRange 리팩토링

- p.470 982행 ~ p.471 995행, p.504 386행 ~ 419행
- if문 연쇄가 다소 번거롭게 보여 DateInterval enum으로 옮겨 if문 연쇄를 완전히 없앴다.
 
```java
public enum DateInterval {
    OPEN {
        public boolean isIn(int d, int left, int right) {
            return d > left && d < right;
        }
    },
    CLOSED_LEFT {
        public boolean isIn(int d, int left, int right) {
            return d >= left && d < right;
        }
    },
    CLOSED_RIGHT {
        public boolean isIn(int d, int left, int right) {
            return d > left && d <= right;
        }
    },
    CLOSED {
        public boolean isIn(int d, int left, int right) {
            return d >= left && d <= right;
        }
    };

    public abstract boolean isIn(int d, int left, int right);
}

public boolean isInRange(DayDate di, DayDate d2, DateInterval interval) {
    int left = Math.min(d1.getOrdinalDay(), d2.getOrdinalDay());
    int right = Math.max(d1.getOrdinalDay(), d2.getOrdinalDay());
    return interval.isIn(getOrdinalDay(), left, right);
}
```

## <a name = '3'> 결론 p.365

- 지금까지 한 작업을 정리한다면

1. 처음 나오는 주석은 너무 오래되어 간단하게 고치고 개선하였다.
1. enum을 모두 독자적인 소스 파일로 옮겼다.
1. 정적 변수(dateFormatSymbols)와 정적 메소드(getMonthNames, isLeapYear, lastDayOfMonth)를 DateUtil이라는 새 클래스로 옮겼다.
1. 일부 추상 메소드를 DayDate 클래스로 끌어올렸다.
1. Month.make를 Month.fromInt로 변경하였고 다른 enum도 똑같이 변경하였다. 또한 모든 enum에 toInt() 접근자를 생성하고 index필드를 private으로 정의하였다.
1. plusYear, plusMonths에 중복이 있었고 correctLastDayOfMonth라는 새 메소드를 생성해 중복을 없앴다.
1. 여기저기 사용하던 숫자 1을 없앴고 모두 Month.JANUARY.toInt() 혹은 Day.SUNDAY.toInt()로 적절히 변경했다. SpreadsheetDate 코드의 알고리즘을 변경하였다.

- 우리는 보이스카우트 규칙을 따랐고 테스트 커버리지를 증가시키고, 버그 몇 개를 고쳤으며, 코드 크기를 줄이고, 코드가 명확해졌다.
# 17장. 냄새와 휴리스틱 p.368

1. [주석](#C)
   1. [부적절한 정보](#C-1)
   1. [쓸모 없는 주석](#C-2)
   1. [중복된 주석](#C-3)
   1. [성의 없는 주석](#C-4)
   1. [주석 처리된 코드](#C-5)
1. [환경](#E)
   1. [여러 단계로 빌드해야 한다](#E-1)
   1. [여러 단계로 테스트해야 한다](#E-2)
1. [함수](#F)
   1. [너무 많은 인수](#F-1)
   1. [출력 인수](#F-2)
   1. [플래그 인수](#F-3)
   1. [죽은 함수](#F-4)
1. [일반](#G)
   1. [한 소스 파일에 여러 언어를 사용한다](#G-1)
   1. [당연한 동작을 구현하지 않는다](#G-2)
   1. [경계를 올바로 처리하지 않는다](#G-3)
   1. [안전 절차 무시](#G-4)
   1. [중복](#G-5)
   1. [추상화 수준이 올바르지 못하다](#G-6)
   1. [기초 클래스가 파생 클래스에 의존한다](#G-7)
   1. [과도한 정보](#G-8)
   1. [죽은 코드](#G-9)
   1. [수직 분리](#G-10)
   1. [일관성 부족](#G-11)
   1. [잡동사니](#G-12)
   1. [인위적 결합](#G-13)
   1. [기능 욕심](#G-14)
   1. [선택자 인수](#G-15)
   1. [모호한 의도](#G-16)
   1. [잘못 지운 책임](#G-17)
   1. [부적절한 static 함수](#G-18)
   1. [서술적 변수](#G-19)
   1. [이름과 기능이 일치하는 함수](#G-20)
   1. [알고리즘을 이해하라](#G-21)
   1. [논리적 의존성은 물리적으로 드러내라](#G-22)

## <a name = 'C'> 주석

### <a name = 'C-1'> C1: 부적절한 정보

- 다른 시스템(소스 코드 관리 시스템, 버그 추적 시스템, 이슈 추적 시스템, 외에 기록 관리 시스템)에 저장할 정보는 주석으로 적절하지 못하다.
- 주석은 코드와 설계에 기술적인 설명을 부연하는 수단이다.

### <a name = 'C-2'> C2: 쓸모 없는 주석

- 오래된 주석, 엉뚱한 주석, 잘못된 주석은 더 이상 쓸모가 없다.

### <a name = 'C-3'> C3: 중복된 주석

- 코드만으로 충분한데 구구절절 설명하는 주석은 중복된 주석이다.

```java
i++; // i 증가

/**
 * @param sellRequest
 * @return
 * @throws ManagedComponentException
 */
public SellResponse beginSellItem(SellRequest sellRequest)
throws ManagedComponentException
```

### <a name = 'C-4'> C4: 성의 없는 주석

- 단어를 신중하게 선택하고 문법과 구두점을 올바로 사용한다.
- 주절대지 않고 당연한 소리를 반복하지 않고 간결하고 명료하게 작성한다.

### <a name = 'C-5'> C5: 주석 처리된 코드

- 주석으로 처리된 코드가 줄줄이 나오면 신경이 거슬린다.
- 더 이상 존재하지 않는 함수나 이름은 모듈을 오혐시키고 읽는 사람을 헷갈리게 만든다.
- 주석으로 처리된 코드는 발견하면 즉각 지워버리자!

## <a name = 'E'> 환경

### <a name = 'E-1'> E1: 여러 단계로 빌드해야 한다

- 빌드는 간단히 한 단계로 끝나야 한다.
- 불가해한 명령이나 스크립트를 잇달아 실행해 각 요소를 따로 빌드하거나 소스 코드 관리 시스템에서 이것저것 따로 체크아웃할 필요가 없어야 한다.

```bash
svn get mySystem
cd mySystem
ant all
```

### <a name = 'E-2'> E2: 여러 단계로 테스트해야 한다

- 모든 단위 테스트는 한 명령으로 돌려야 한다. 모든 테스트를 한 번에 실행하는 능력은 아주 근본적이고 아주 중요하다.
- IDE에서 버튼 하나로 실행시키거나, 아무리 열악한 환경이라도 셸에서 명령 하나로 가능해야 한다.

## <a name = 'F'> 함수

### <a name = 'F-1'> F1: 너무 많은 인수

- 함수에서 인자 개수는 작을수록 좋다.
- 인자가 넷 이상은 의심스러우니 최대한 지양한다.

### <a name = 'F-2'> F2: 출력 인수

- 일반적으로 독자는 인자를 입력으로 간주한다.
- 함수에서 뭔가의 상태를 변경해야 한다면 함수가 속한 객체의 상태를 변경한다.
- 객체 지향 언어에서는 출력 인수를 사용할 필요가 거의 없다. 출력 인수로 사용하라고 만든 것이 this이기 때문이다. (p.56)
- 함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택해야 한다.

```java
// 무언가에 s를 바닥글로 붙이는 것일까? s에 바닥글을 붙이는 것일까?
appendFooter(s);

// 인자 대신 함수가 속한 객체 상태를 변경하는 방식이면 의미가 정확함
report.appendFooter();
```

### <a name = 'F-3'> F3: 플래그 인수

- boolean 인자는 함수가 여러 기능을 수행한다는 명백한 증거이기에 혼란을 초래할 수 있음으로 피하는 것이 좋다.

### <a name = 'F-4'> F4: 죽은 함수

- 아무도 호출하지 않는 함수는 삭제한다.
- 소스 코드 관리 시스템이 모두 기억하므로 걱정할 필요는 없다.

## <a name = 'G'> 일반

### <a name = 'G-1'> G1: 한 소스 파일에 여러 언어를 사용한다

- 오늘날 프로그래밍 환경은 한 소스 내에 다양한 언어를 지원한다.
- 다양하게 쓰는 것은 혼란을 야기시키기에 소스 파일 하나에 언어 하나만 사용하는 방식이 이상적이다.
- 소스 파일에서 언어 수와 범위를 최대한 줄이도록 노력해야 한다.

### <a name = 'G-2'> G2: 당연한 동작을 구현하지 않는다

- 최소 놀람의 원칙에 따라 함수나 클래스는 다른 프로그래머가 당연하게 여길 만한 동작과 기능을 제공해야 한다.
- 당연한 동작을 구현하지 않으면 코드를 읽거나 사용하는 사람이 더 이상 함수 이름만으로 기능을 직관적으로 예상하기 어렵다.

```java
// 'Monday' -> Day.MONDAY로 변환하기를 기대한다.
Day day = DayDate.StringToDay(String dayName);
```

> **최소 놀람의 원칙**
>
> 코드가 읽는 이를 놀라게 해서는 안 된다. 즉, 읽었을 때 당연한 기능을 해야 한다.

### <a name = 'G-3'> G3: 경계를 올바로 처리하지 않는다

- 흔히 개발자들은 머릿속에서 코드를 돌려보고 끝낸다. 즉, 자신의 직관에 의존할 뿐 모든 경계와 구석진 곳에서 코드를 증명하려고 애쓰지 않는다.
- 자신의 직관에 의존하지 말고 모든 경계 조건을 찾아내고, 모든 경계 조건을 테스트하는 테스트 케이스를 작성해라.

### <a name = 'G-4'> G4: 안전 절차 무시

- 안전 절차를 무시하면 위험하다
- 컴파일러를 꺼버리면 빌드는 쉬워지겠지만 자칫하면 끝없는 디버깅에 시달리게 된다.
- 실패하는 테스트 케이스를 미루는 태도는 신용카드가 공짜 돈이라는 생각만큼 위험하다.

### <a name = 'G-5'> G5: 중복

- DRY(Don't Repeat Yourself) 원칙, OAOO(Once and only once) 라고 부른다.
- 코드에서 중복을 발견할 때마다 추상화할 기회로 간주 해야한다. 중복된 코드를 하위 루틴이나 다른 클래스로 분리해라
- 추상화 수준을 높이게 되면 구현이 빨라지고 오류가 적어진다.
- 똑같은 코드가 중복해서 나오면 간단한 함수로 교체한다.
- switch/case나 if/else 문으로 똑같은 조건을 거듭 확인하는 것은 다형성으로 대체한다.
- 알고리즘은 비슷하나 코드가 서로 다른 중복은 Template method 패턴이나 strategy 패턴으로 중복을 제거한다.

### <a name = 'G-6'> G6: 추상화 수준이 올바르지 못하다

- 추상화는 저차원 상세 개념에서 고차원 일반 개념을 분리한다.
- 우리는 추상 클래스(고차원 개념을 표현하는)와 파생 클래스(저차원 개념을 표현하는)를 생성해 추상화를 수행한다.
- 고차원 개념과 저차원 개념을 섞어서는 안 된다.

```java
public interface Stack {
  Object pop() throws EmptyException;
  void push(Object o) throws FullException;
  double percentFull(); // 추상화 수준이 올바르지 못하기 때문에 파생 인터페이스에 넣어줘야한다.
  class EmptyException extends Exception {}
  class FullException extends Exception {}
}
```

- percentFull 함수는 추상화 수준이 올바르지 않다.
- 어떤 구현은 '꽉 찬 정도'라는 개념이 타당하고 다른 건 알아낼 방법이 전혀 없다.
- 그러므로 이 함수는 BoundedStack과 같은 파생 인터페이스에 넣어야 마땅하다.
- 크기가 무한한 스택은 0을 반환하면 되지 않나? 하지만 진정으로 무한한 스택은 존재하지 않기에 0을 반환하면 거짓말을 하는 셈이다.

### <a name = 'G-7'> G7: 기초 클래스가 파생 클래스에 의존한다

- 기초 클래스가 파생 클래스를 사용한다면 뭔가 문제가 있다는 것이다. 즉, 기초 클래스는 파생 클래스를 아예 몰라야 마땅하다.
- 하지만 파생 클래스 개수가 확실히 고정되었다면 기초 클래스에 파생 클래스를 선택하는 코드가 들어가는 예외가 존재한다. FSM구현에서 많이 본 사례이다.
- FSM은 기초 클래스와 파생 클래스를 같은 JAR 파일로 배포한다. 하지만 일반적으로 기초, 파생 클래스는 개별로 배포하는 편이 좋다.
- 독립적인 개별 컴포넌트 단위로 시스템을 배치할 수 있으므로 컴포넌트 변경 시에 기초 컴포넌트까지 재배치할 필요가 없기에 유지보수가 한결 편해진다.

> **FSM(Finite State Machine)**
>
> 유한한 개수의 행동(상태)으로 정의하고, 이 상태 사이의 전이를 제어함으로써 반응하는 기계

### <a name = 'G-8'> G8: 과도한 정보

- 잘 정의된 모듈은 인터페이스가 아주 작고 많은 동작이 가능하다.
- 잘 정의된 인터페이스는 많은 함수를 제공하지 않아 결합도가 낮고 부실하게 정의된 인터페이스는 반드시 호출해야 하는 온갖 함수를 제공해 결합도가 높다.
- 클래스가 제공하는 메소드 수는 작을수록 좋다.
- 인터페이스를 매우 작게 그리고 매우 깐깐하게 만들어 정보를 제한하여 결합도를 낮춰라.

### <a name = 'G-9'> G9: 죽은 코드

- 실행되지 않는 코드는 없는 것이 좋다.
- 불가능한 조건을 확인하는 if, throw 문이 없는 try, catch 블록 등이 좋은 예이다.
- 이러한 코드는 시간이 지날수록 악취가 풍기므로 발견하는 즉시 시스템에서 제거해라.

### <a name = 'G-10'> G10: 수직 분리

- 변수와 함수는 사용되는 위치에 가깝게 정의한다.
- 지역 변수는 처음으로 사용하기 직전에 선언하며 수직으로 가까운 곳에 위치해야 한다.
- 비공개 함수는 처음 호출한 직후에 정의하는 것이 눈에 띄고 좋다.

### <a name = 'G-11'> G11: 일관성 부족

- 어떤 개념을 특정 방식으로 구현했다면 유사한 개념도 같은 방식으로 구현한다.
- 앞에서 언급한 최소 놀람의 원칙에도 부합하며 표기법은 신중하게 선택하며, 선택한 표기법은 신중하게 따른다.
- 한 메소드를 processVerificationRequest이라고 작성했다면 processDeletionRequest 처럼 유사한 이름을 사용한다.

### <a name = 'G-12'> G12: 잡동사니

- 비어 있는 기본 생성자가 왜 필요한가? 쓸데없이 코드만 복잡하게 만든다.
- 아무도 사용하지 않는 변수, 호출하지 않는 함수, 정보를 제공하지 못하는 주석을 깔끔하게 정리하자.

### <a name = 'G-13'> G13: 인위적 결합

- 서로 무관한 개념을 인위적으로 결합하지 않는다.
- 일반적인 enum은 특정 클래스에 속할 이유가 없고 범용 static 함수 또한 특정 클래스에 속할 이유가 없다.
- 인위적인 결합은 뚜렷한 목적 없이 변수, 상수, 함수를 당장 편한 위치에 넣어버린 결과이고 게으르고 부주의한 행동이다.
- 변수, 상수, 함수를 선언할 때는 시간을 들여 올바른 위치를 생각하자.

### <a name = 'G-14'> G14: 기능 욕심

- 클래스 메소드는 자기 클래스의 변수와 함수에 관심을 가져야지 다른 클래스의 변수와 함수에 관심을 가져서는 안 된다.
- 메소드가 다른 객체의 참조자와 변경자를 사용해 그 객체 내용을 수정한다면 메소드가 그 객체 클래스의 범위를 욕심내는 탓이다.

```java
public class HourlyPayCalculator {
    public Money calculateWeeklyPay(HourlyEmployee e) { // 기능 욕심
        int tenthRate = e.getTenthRate().getPennies();
        int tenthsWorked = e.getTenthsWorked();
        int straightTime = Math.min(400, tenthWorked);
        int overTime = Math.max(0, tenthsWorked - straightTime);
        int straightPay = straightTime * tenthRate;
        int overtimePay = (int)Math.round(overTime * tenthRate * 1.5);
        return new Money(straightPay + overtimePay);
    }
}
```

- calculateWeeklyPay 메소드는 HourlyEmployee 클래스의 범위를 욕심내는 예제이다.

```java
public class HourlyEmployeeReport {
    private HourlyEmployee employee;

    public HourlyEmployeeReport(HourlyEmployee e) {
        this.employee = e;
    }

    String reportHours() { 
        "Name : %s\tHours : %d.%1d\n",
        employee.getName(),
        employee.getTenthsWorked() / 10,
        employee.getTenthsWorked() % 10);
    }
}
```

- 하지만 위 예제처럼 어쩔 수 없는 경우도 생긴다.
- reportHours 메소드가 HourlyEmployee 클래스를 욕심내지만 HourlyEmployee 클래스가 보고서 형식을 알 필요는 없다.
- 만약 결합하게 된다면 객체 지향 설계 여러 원칙을 위반하게 된다.
- HourlyEmployee가 보고서 형식을 알 필요도 없을뿐더러 형식이 바뀌게 되면 클래스도 변경이 일어나야 한다.

### <a name = 'G-15'> G15: 선택자 인수

- 함수 인자로 선택자가 들어가는 것은 좋지 않다.
- 선택자 인자는 목적을 기억하기 어려울 뿐 아니라 각 선택자 인자가 여러 함수를 하나로 조합한다.
- 큰 함수를 작은 함수 여럿으로 쪼개지 않으려는 게으름이다.

```java
public int calculateWeeklyPay(boolean overtime) {
    int tenthRate = getTenthRate();
    int tenthsWorked = getTenthsWorked();
    int straightTime = Math.min(400, tenthsWorked);
    int overTime = Math.max(0, tenthsWorked - straightTime);
    int straightPay = straightTime * tenthRate;
    double overtimeRate = overtime ? 1.5 : 1.0 * tenthRate;
    int overtimePay = (int)Math.round(overTime * overtimeRate);
    return straightPay + overtimePay;
}
```

- 초과근무 수당을 1.5배로 지급하면 true 아니면 false 이다.
- 독자는 calculateWeeklyPay(false)라는 코드를 발견할 때마다 의미를 떠올리느라 골치를 앓는다.

```java
public int straightPay() {
    return getTenthsWorked() * getTenthRate();
}

public int overTimePay() {
    int overTimeTenths = Math.max(0, getTenthsWorked() - 400);
    int overTimePay = overTimeBonus(overTimeTenths);
    return straightPay() + overTimePay;
}

private int overTimeBonus(int overTimeTenths) { 
    double bonus = 0.5 * getTenthRate() * overTimeTenths;
    return (int) Math.round(bonus);
}
```

- 독자는 위와 같이 코드를 구현할 기회를 놓쳤다.
- boolean 인자가 문제라기보다는 enum, int 등 함수 동작을 제어하려는 인수는 하나 같이 바람직하지 않다.
- 일반적으로 인자를 넘겨 동작을 선택하는 대신 새로운 함수를 만드는 편이 좋다.

### <a name = 'G-16'> G16: 모호한 의도

- 코드를 짤 때는 의도를 최대한 분명히 밝힌다.

```java
public int m_otCalc() {
    return iThsWkd * iThsRte + 
    (int) Math.round(0.5 * iThsRte *
        Math.max(0, iThsWkd - 400)
    );
}
```

- 위의 예제는 짧고 빽빽할 뿐만 아니라 거의 불가해하다.
- 행을 바꾸지 않고 표현한 수식, 헝가리식 표기법, 매직 번호 등은 모두 저자의 의도를 흐린다.

### <a name = 'G-17'> G17: 잘못 지운 책임

- 코드를 배치하는 위치를 정하는 것은 중요하다.
- 여기서도 최소 놀람의 원칙이 적용되고 독자가 자연스럽게 기대할 위치에 배치해야 한다.
- 때로는 독자에게 직관적인 위치가 아니라 개발자에게 편한 함수에 배치한다.
- 예로 보고서를 출력하는 함수에서 총계를 계산하는 방법, 근무 시간을 입력받는 코드에서 총계를 보관하는 방법이 각각 있다.
- 보고서 모듈에 getTotalHours, 근무 시간을 입력받는 모듈 saveTimeCard 두 함수 중 어느 쪽이 총계를 계산해야 옳을까?
- 성능을 높이고자 근무 시간을 입력받는 모듈에서 총계를 계산하는 편이 좋다고 판단할 수도 있다. 그렇다면 이 사실을 반영해 이름을 computeRunningTotalOfHours와 같이 제대로 지어야 한다.

### <a name = 'G-18'> G18: 부적절한 static 함수

- 메소드를 재정의할 가능성이 없는 것을 static으로 정의한다.
- Math.max(double a, double b)는 좋은 static 메소드의 예이다. 만약 new Math().max(a, b)나 a.max(b)라 하면 오히려 우습다.

```java
HourlyPayCalculator.calculatePay(employee, overtimeRate);
```

- 위 예제는 특정 객체와 관련이 없으면서 모든 정보를 인수에서 모든 정보를 인자에서 가져오므로 static 함수가 적절하다고 생각할 수 있다.
- 하지만, 수당을 계산하는 알고리즘이 하나가 아니고 여러 개일지도 모르기에 메소드를 재정의할 가능성이 존재하기 때문에 static으로 적절치 않다.

### <a name = 'G-19'> G19: 서술적 변수

- 프로그램 가독성을 높이는 가장 효과적인 방법 중 하나가 계산을 여러 단계로 나누고 중간값으로 서술적인 변수 이름을 사용하는 방법이다.

```java
Matcher match = headerPattern.matcher(line);
if(match.find())
{
  String key = match.group(1);
  String value = match.group(2);
  headers.put(key.toLowerCase(), value);
}
```

- 서술적인 변수 이름은 많이 써도 괜찮다. 일반적으로 많을수록 더 좋다.
- 계산을 몇 단계로 나누고 중간값에 좋은 변수 이름만 붙여도 해독하기 어렵던 모듈이 순식간에 읽기 쉬운 모듈로 탈바꿈한다.

### <a name = 'G-20'> G20: 이름과 기능이 일치하는 함수

```java
Date newDate = date.add(5);
```

- 위 예제는 5일을 더하는지 5주, 5시간을 더하는지 date 인스턴스를 변경하는 함수인지 그대로 두고 새로운 Date를 반환하는 함수인지 알 수 없다.
- day 인스턴스에 5일을 더해 date 인스턴스를 변경하는 함수라면 addDaysTo 혹은 increaseByDays 라는 이름이 좋다.
- 반면, date 인스턴스가 변하지 않고 새 날짜를 반환한다면 daysLater, daysSince라는 이름이 좋다.
- 이름만으로도 분명하게 하는 역할이 드러날 수 있도록 기능을 정리하거나 이름을 새로 붙이는 게 좋다.

### <a name = 'G-21'> G21: 알고리즘을 이해하라

- 프로그램이 '돌아간다는' 사실은 모든 테스트 케이스를 통과한다고 생각할 수 있다.
- 하지만 '돌아간다고' 말하기에는 뭔가 부족하다.
- 구현이 끝났다고 선언하기 전에 함수가 돌아가는 방식을 확실히 이해해야 한다.
- 테스트 케이스가 모두 통과한다는 사실만으로는 부족하고 작성자가 알고리즘이 올바르다는 사실을 알아야 한다.

### <a name = 'G-22'> G22: 논리적 의존성은 물리적으로 드러내라

- 한 모듈이 다른 모듈에 의존한다면 물리적인 의존성이 존재한다. 논리적 의존성만으로는 부족하다.
- 의존하는 모듈이 상대 모듈에 대해 뭔가를 가정하면 안 되고 의존하는 모든 정보를 명시적으로 요청하는 편이 좋다.

```java
public class HourlyReporter {
  private HourlyReportFormatter formatter;
  private List<LineItem> page;
  private final int PAGE_SIZE = 55;

  public HourlyReporter(HourlyReportFormatter formatter) {
    this.formatter = formatter;
    page = new ArrayList<LineItem>();
  }

  public void generateReporter(List<HourlyEmployee> employees) {
    for (HourlyEmployee e : employees) {
      addLineItemToPage(e);
      if (page.size() == PAGE_SIZE) {
        printAndClearItemList();
      }
    }
    if (page.size() == 0)
      printAndClearItemList();
  }

  private void printAndClearItemList() {
    formatter.format(page);
    page.clear();
  }

  private void addLineItemToPage(HourlyEmployee e) {
    LineItem item = new LineItem();
    item.name = e.getName();
    item.hours = e.getTenthsWorked() / 10;
    item.tenths = e.getTenthsWorked() % 10;
    page.add(item);
  }

  private class LineItem {
    public String name;
    public int hours;
    public int tenths;
  }
}
```

- 위 코드는 논리적인 의존성이 존재한다. PAGE_SIZE라는 상수이다.
- 어째서 HourlyRepoter 클래스가 페이지 크기를 알아야 할까? 그것은 HourlyReportFormatter가 책임질 정보다.
- PAGE_SIZE를 HourlyReporter 클래스에 선언한 실수는 잘못 지운 책임[G17]에 해당한다.
- HourlyReporter 클래스가 HourlyReportFormatter가 페이지 크기를 알 거라고 가정하는 것이 논리적 의존성이다.
- HoulyReportFormatter에 getMaxPageSize() 메소드를 추가하여 논리적인 의존성이 물리적인 의존성으로 변한다.
- HourlyReporter 클래스는 PAGE_SIZE 상수를 사용하는 대신 getMaxPageSize() 함수를 호출하면 된다.
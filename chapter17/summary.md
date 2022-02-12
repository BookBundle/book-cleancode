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
   1. [G23: If/Else 혹은 Switch/Case 문보다 다형성을 사용하라](#G-23)
   1. [G24: 표준 표기법을 따르라](#G-24)
   1. [G25: 매직 숫자는 명명된 상수로 교체하라](#G-25)
   1. [G26: 정확하라](#G-26)
   1. [G27: 관례보다 구조를 사용하라](#G-27)
   1. [G28: 조건을 캡슐화하라](#G-28)
   1. [G29: 부정 조건은 피하라](#G-29)
   1. [G30: 함수는 한 가지만 해야 한다](#G-30)
   1. [G31: 숨겨진 시간적인 결합](#G-31)
   1. [G32: 일관성을 유지하라](#G-32)
   1. [G33: 경계 조건을 캡슐화하라](#G-33)
   1. [G34: 함수는 추상화 수준을 한 단계만 내려가야 한다](#G-34)
   1. [G35: 설정 정보는 최상위 단계에 둬라](#G-35)
   1. [G36: 추이적 탐색을 피하라](#G-36)
1. [자바](#J)
   1. [J1: 긴 import 목록을 피하고 와일드카드를 사용하라](#J-1)
   1. [J2: 상수는 상속하지 않는다](#J-2)
   1. [J3: 상수 대 Enum](#J-3)
1. [이름](#N)
   1. [N1: 서술적인 이름을 사용하라](#N-1)
   1. [N2: 적절한 추상화 수준에서 이름을 선택하라](#N-2)
   1. [N3: 가능하다면 표준 명명법을 사용하라](#N-3)
   1. [N4: 명확한 이름](#N-4)
   1. [N5: 긴 범위는 긴 이름을 사용하라](#N-5)
   1. [N6: 인코딩을 피하라](#N-6)
   1. [N7: 이름으로 부수 효과를 설명하라](#N-7)
1. [테스트](#T)
   1. [T1: 불충분한 테스트](#T-1)
   1. [T2: 커버리지 도구를 사용하라!](#T-2)
   1. [T3: 사소한 테스트를 건너뛰지 마라](#T-3)
   1. [T4: 무시한 테스트는 모호함을 뜻한다](#T-4)
   1. [T5: 경계 조건을 테스트하라](#T-5)
   1. [T6: 버그 주변은 철저히 테스트하라](#T-6)
   1. [T7: 실패 패턴을 살펴라](#T-7)
   1. [T8: 테스트 커버리지 패턴을 살펴라](#T-8)
   1. [T9: 테스트는 빨라야 한다](#T-9)
1. [결론](#conclusion)

## <a name = 'C'> 주석 p.368 </a>

### <a name = 'C-1'> C1: 부적절한 정보 </a>

- 다른 시스템(소스 코드 관리 시스템, 버그 추적 시스템, 이슈 추적 시스템, 외에 기록 관리 시스템)에 저장할 정보는 주석으로 적절하지 못하다.
- 주석은 코드와 설계에 기술적인 설명을 부연하는 수단이다.

### <a name = 'C-2'> C2: 쓸모 없는 주석 </a>

- 오래된 주석, 엉뚱한 주석, 잘못된 주석은 더 이상 쓸모가 없다.

### <a name = 'C-3'> C3: 중복된 주석 </a>

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

### <a name = 'C-4'> C4: 성의 없는 주석 </a>

- 단어를 신중하게 선택하고 문법과 구두점을 올바로 사용한다.
- 주절대지 않고 당연한 소리를 반복하지 않고 간결하고 명료하게 작성한다.

### <a name = 'C-5'> C5: 주석 처리된 코드 </a>

- 주석으로 처리된 코드가 줄줄이 나오면 신경이 거슬린다.
- 더 이상 존재하지 않는 함수나 이름은 모듈을 오혐시키고 읽는 사람을 헷갈리게 만든다.
- 주석으로 처리된 코드는 발견하면 즉각 지워버리자!

## <a name = 'E'> 환경 p.370 </a>

### <a name = 'E-1'> E1: 여러 단계로 빌드해야 한다 </a>

- 빌드는 간단히 한 단계로 끝나야 한다.
- 불가해한 명령이나 스크립트를 잇달아 실행해 각 요소를 따로 빌드하거나 소스 코드 관리 시스템에서 이것저것 따로 체크아웃할 필요가 없어야 한다.

```bash
svn get mySystem
cd mySystem
ant all
```

### <a name = 'E-2'> E2: 여러 단계로 테스트해야 한다 </a>

- 모든 단위 테스트는 한 명령으로 돌려야 한다. 모든 테스트를 한 번에 실행하는 능력은 아주 근본적이고 아주 중요하다.
- IDE에서 버튼 하나로 실행시키거나, 아무리 열악한 환경이라도 셸에서 명령 하나로 가능해야 한다.

## <a name = 'F'> 함수 p.370 </a>

### <a name = 'F-1'> F1: 너무 많은 인수 </a>

- 함수에서 인자 개수는 작을수록 좋다.
- 인자가 넷 이상은 의심스러우니 최대한 지양한다.

### <a name = 'F-2'> F2: 출력 인수 </a>

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

### <a name = 'F-3'> F3: 플래그 인수 </a>

- boolean 인자는 함수가 여러 기능을 수행한다는 명백한 증거이기에 혼란을 초래할 수 있음으로 피하는 것이 좋다.

### <a name = 'F-4'> F4: 죽은 함수 </a>

- 아무도 호출하지 않는 함수는 삭제한다.
- 소스 코드 관리 시스템이 모두 기억하므로 걱정할 필요는 없다.

## <a name = 'G'> 일반 p.371 </a>

### <a name = 'G-1'> G1: 한 소스 파일에 여러 언어를 사용한다 </a>

- 오늘날 프로그래밍 환경은 한 소스 내에 다양한 언어를 지원한다.
- 다양하게 쓰는 것은 혼란을 야기시키기에 소스 파일 하나에 언어 하나만 사용하는 방식이 이상적이다.
- 소스 파일에서 언어 수와 범위를 최대한 줄이도록 노력해야 한다.

### <a name = 'G-2'> G2: 당연한 동작을 구현하지 않는다 </a>

- 최소 놀람의 원칙에 따라 함수나 클래스는 다른 프로그래머가 당연하게 여길 만한 동작과 기능을 제공해야 한다.
- 당연한 동작을 구현하지 않으면 코드를 읽거나 사용하는 사람이 더 이상 함수 이름만으로 기능을 직관적으로 예상하기 어렵다.

```java
// 'Monday' -> Day.MONDAY로 변환하기를 기대한다.
Day day = DayDate.StringToDay(String dayName);
```

> **최소 놀람의 원칙**
>
> 코드가 읽는 이를 놀라게 해서는 안 된다. 즉, 읽었을 때 당연한 기능을 해야 한다.

### <a name = 'G-3'> G3: 경계를 올바로 처리하지 않는다 </a>

- 흔히 개발자들은 머릿속에서 코드를 돌려보고 끝낸다. 즉, 자신의 직관에 의존할 뿐 모든 경계와 구석진 곳에서 코드를 증명하려고 애쓰지 않는다.
- 자신의 직관에 의존하지 말고 모든 경계 조건을 찾아내고, 모든 경계 조건을 테스트하는 테스트 케이스를 작성해라.

### <a name = 'G-4'> G4: 안전 절차 무시 </a>

- 안전 절차를 무시하면 위험하다
- 컴파일러를 꺼버리면 빌드는 쉬워지겠지만 자칫하면 끝없는 디버깅에 시달리게 된다.
- 실패하는 테스트 케이스를 미루는 태도는 신용카드가 공짜 돈이라는 생각만큼 위험하다.

### <a name = 'G-5'> G5: 중복 </a>

- DRY(Don't Repeat Yourself) 원칙, OAOO(Once and only once) 라고 부른다.
- 코드에서 중복을 발견할 때마다 추상화할 기회로 간주 해야한다. 중복된 코드를 하위 루틴이나 다른 클래스로 분리해라
- 추상화 수준을 높이게 되면 구현이 빨라지고 오류가 적어진다.
- 똑같은 코드가 중복해서 나오면 간단한 함수로 교체한다.
- switch/case나 if/else 문으로 똑같은 조건을 거듭 확인하는 것은 다형성으로 대체한다.
- 알고리즘은 비슷하나 코드가 서로 다른 중복은 Template method 패턴이나 strategy 패턴으로 중복을 제거한다.

### <a name = 'G-6'> G6: 추상화 수준이 올바르지 못하다 </a>

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

### <a name = 'G-7'> G7: 기초 클래스가 파생 클래스에 의존한다 </a>

- 기초 클래스가 파생 클래스를 사용한다면 뭔가 문제가 있다는 것이다. 즉, 기초 클래스는 파생 클래스를 아예 몰라야 마땅하다.
- 하지만 파생 클래스 개수가 확실히 고정되었다면 기초 클래스에 파생 클래스를 선택하는 코드가 들어가는 예외가 존재한다. FSM구현에서 많이 본 사례이다.
- FSM은 기초 클래스와 파생 클래스를 같은 JAR 파일로 배포한다. 하지만 일반적으로 기초, 파생 클래스는 개별로 배포하는 편이 좋다.
- 독립적인 개별 컴포넌트 단위로 시스템을 배치할 수 있으므로 컴포넌트 변경 시에 기초 컴포넌트까지 재배치할 필요가 없기에 유지보수가 한결 편해진다.

> **FSM(Finite State Machine)**
>
> 유한한 개수의 행동(상태)으로 정의하고, 이 상태 사이의 전이를 제어함으로써 반응하는 기계

### <a name = 'G-8'> G8: 과도한 정보 </a>

- 잘 정의된 모듈은 인터페이스가 아주 작고 많은 동작이 가능하다.
- 잘 정의된 인터페이스는 많은 함수를 제공하지 않아 결합도가 낮고 부실하게 정의된 인터페이스는 반드시 호출해야 하는 온갖 함수를 제공해 결합도가 높다.
- 클래스가 제공하는 메소드 수는 작을수록 좋다.
- 인터페이스를 매우 작게 그리고 매우 깐깐하게 만들어 정보를 제한하여 결합도를 낮춰라.

### <a name = 'G-9'> G9: 죽은 코드 </a>

- 실행되지 않는 코드는 없는 것이 좋다.
- 불가능한 조건을 확인하는 if, throw 문이 없는 try, catch 블록 등이 좋은 예이다.
- 이러한 코드는 시간이 지날수록 악취가 풍기므로 발견하는 즉시 시스템에서 제거해라.

### <a name = 'G-10'> G10: 수직 분리 </a>

- 변수와 함수는 사용되는 위치에 가깝게 정의한다.
- 지역 변수는 처음으로 사용하기 직전에 선언하며 수직으로 가까운 곳에 위치해야 한다.
- 비공개 함수는 처음 호출한 직후에 정의하는 것이 눈에 띄고 좋다.

### <a name = 'G-11'> G11: 일관성 부족 </a>

- 어떤 개념을 특정 방식으로 구현했다면 유사한 개념도 같은 방식으로 구현한다.
- 앞에서 언급한 최소 놀람의 원칙에도 부합하며 표기법은 신중하게 선택하며, 선택한 표기법은 신중하게 따른다.
- 한 메소드를 processVerificationRequest이라고 작성했다면 processDeletionRequest 처럼 유사한 이름을 사용한다.

### <a name = 'G-12'> G12: 잡동사니 </a>

- 비어 있는 기본 생성자가 왜 필요한가? 쓸데없이 코드만 복잡하게 만든다.
- 아무도 사용하지 않는 변수, 호출하지 않는 함수, 정보를 제공하지 못하는 주석을 깔끔하게 정리하자.

### <a name = 'G-13'> G13: 인위적 결합 </a>

- 서로 무관한 개념을 인위적으로 결합하지 않는다.
- 일반적인 enum은 특정 클래스에 속할 이유가 없고 범용 static 함수 또한 특정 클래스에 속할 이유가 없다.
- 인위적인 결합은 뚜렷한 목적 없이 변수, 상수, 함수를 당장 편한 위치에 넣어버린 결과이고 게으르고 부주의한 행동이다.
- 변수, 상수, 함수를 선언할 때는 시간을 들여 올바른 위치를 생각하자.

### <a name = 'G-14'> G14: 기능 욕심 </a>

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

### <a name = 'G-15'> G15: 선택자 인수 </a>

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

### <a name = 'G-16'> G16: 모호한 의도 </a>

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

### <a name = 'G-17'> G17: 잘못 지운 책임 </a>

- 코드를 배치하는 위치를 정하는 것은 중요하다.
- 여기서도 최소 놀람의 원칙이 적용되고 독자가 자연스럽게 기대할 위치에 배치해야 한다.
- 때로는 독자에게 직관적인 위치가 아니라 개발자에게 편한 함수에 배치한다.
- 예로 보고서를 출력하는 함수에서 총계를 계산하는 방법, 근무 시간을 입력받는 코드에서 총계를 보관하는 방법이 각각 있다.
- 보고서 모듈에 getTotalHours, 근무 시간을 입력받는 모듈 saveTimeCard 두 함수 중 어느 쪽이 총계를 계산해야 옳을까?
- 성능을 높이고자 근무 시간을 입력받는 모듈에서 총계를 계산하는 편이 좋다고 판단할 수도 있다. 그렇다면 이 사실을 반영해 이름을 computeRunningTotalOfHours와 같이 제대로 지어야 한다.

### <a name = 'G-18'> G18: 부적절한 static 함수 </a>

- 메소드를 재정의할 가능성이 없는 것을 static으로 정의한다.
- Math.max(double a, double b)는 좋은 static 메소드의 예이다. 만약 new Math().max(a, b)나 a.max(b)라 하면 오히려 우습다.

```java
HourlyPayCalculator.calculatePay(employee, overtimeRate);
```

- 위 예제는 특정 객체와 관련이 없으면서 모든 정보를 인수에서 모든 정보를 인자에서 가져오므로 static 함수가 적절하다고 생각할 수 있다.
- 하지만, 수당을 계산하는 알고리즘이 하나가 아니고 여러 개일지도 모르기에 메소드를 재정의할 가능성이 존재하기 때문에 static으로 적절치 않다.

### <a name = 'G-19'> G19: 서술적 변수 </a>

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

### <a name = 'G-20'> G20: 이름과 기능이 일치하는 함수 </a>

```java
Date newDate = date.add(5);
```

- 위 예제는 5일을 더하는지 5주, 5시간을 더하는지 date 인스턴스를 변경하는 함수인지 그대로 두고 새로운 Date를 반환하는 함수인지 알 수 없다.
- day 인스턴스에 5일을 더해 date 인스턴스를 변경하는 함수라면 addDaysTo 혹은 increaseByDays 라는 이름이 좋다.
- 반면, date 인스턴스가 변하지 않고 새 날짜를 반환한다면 daysLater, daysSince라는 이름이 좋다.
- 이름만으로도 분명하게 하는 역할이 드러날 수 있도록 기능을 정리하거나 이름을 새로 붙이는 게 좋다.

### <a name = 'G-21'> G21: 알고리즘을 이해하라 </a>

- 프로그램이 '돌아간다는' 사실은 모든 테스트 케이스를 통과한다고 생각할 수 있다.
- 하지만 '돌아간다고' 말하기에는 뭔가 부족하다.
- 구현이 끝났다고 선언하기 전에 함수가 돌아가는 방식을 확실히 이해해야 한다.
- 테스트 케이스가 모두 통과한다는 사실만으로는 부족하고 작성자가 알고리즘이 올바르다는 사실을 알아야 한다.

### <a name = 'G-22'> G22: 논리적 의존성은 물리적으로 드러내라 </a>

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

### <a name = 'G-23'> G23: If/Else 혹은 Switch/Case 문보다 다형성을 사용하라 </a>

- 3장에서 새 유형을 추가할 확률보다 새 함수를 추가할 확률이 높은 코드에서는 switch 문이 더 적합하다 주장했다.  

- 첫째, 대다수 개발자가 switch 문을 사용하는 이유는 그 상황에서 당장 손쉬운 선택이기 때문이다. 그러므로 switch를 선택하기 전에 다형성을 먼저 고려하라는 의미다.  
- 둘째, 유형보다 함수가 더 쉽게 변하는 경우는 극히 드물다. 그러므로 모든 switch문을 의심해야 한다.

- 나는 'switch문 하나' 규칙을 따른다. 같은 선택을 수행하는 다른 코드에서는 다형성 객체를 생성해 switch문을 대신한다.

### <a name = 'G-24'> G24: 표준 표기법을 따르라 </a>

- 팀은 업계 표준에 기반한 구현 표준을 따라야 한다. 표준을 설명하는 문서는 코드 자체로 충분해야 하며 별도 문서를 만들 필요는 없어야 한다.  
- 팀이 정한 표준은 팀원들 모두가 따라야 한다.  
- 내가 따르는 표기법이 궁금하다면 512쪽 목록 B-7에서 목록 B-14까지 제시한 코드를 살펴본다.

### <a name = 'G-25'> G25: 매직 숫자는 명명된 상수로 교체하라 </a>

- 일반적으로 코드에서 숫자를 사용하지 말라는 규칙이다. 숫자는 명명된 상수 뒤로 숨기라는 의미다.  
- 어떤 상수는 이해하기 쉬우므로, 코드 자체가 자명하다면, 상수 뒤로 숨길 필요가 없다.  
- '매직 숫자'라는 용어는 단지 숫자만 의미하지 않는다. 의미가 분명하지 않은 토큰을 모두 가리킨다.

### <a name = 'G-26'> G26: 정확하라 </a>

- 코드에서 뭔가를 결정할 때는 정확히 결정한다. 결정을 내리는 이유와 예외를 처리할 방법을 분명히 알아야 하며, 대충 결정해서는 안 된다.  
- 호출하는 함수가 null을 반환할지도 모른다면 null을 반드시 점검한다. 병행(concurrent)특성으로 인해 동시에 갱신할 가능성이 있다면 적절한 잠금 매커니즘을 구현한다.  
- 코드에서 모호성과 부정확은 의견차나 게으름의 결과다. 어느 쪽이든 제거해야 마땅하다.

### <a name = 'G-27'> G27: 관례보다 구조를 사용하라 </a>

- 설계 결정을 강제할 때는 규칙보다 관례를 사용한다. 명명 관례도 좋지만 구조 자체로 강제하면 더 좋다.  
- switch/case 문을 매번 똑같이 구현하게 강제하기는 어렵지만, 파생 클래스는 추상 메서드를 모두 구현하지 않으면 안되기에 enum 변수가 있는 switch/case문보다 추상 메서드가 있는 기초 클래스가 더 좋다.

### <a name = 'G-28'> G28: 조건을 캡슐화하라 </a>

- 부울 논리는 (if나 while문에다 넣어 생각하지 않아도) 이해하기 어렵다. 조건의 의도를 분명히 밝히는 함수로 표현하라.  

```java
if (shouldBeDeleted(timer)) // good
if (timer.hasExpired() && !timer.isRecurrent()) // bad
```

### <a name = 'G-29'> G29: 부정 조건은 피하라 </a>

- 부정 조건은 긍정 조건보다 이해하기 어렵다. 가능하면 긍정 조건으로 표현한다.  

```java
if (buffer.shouldCompact()) // good
if (!buffer.shouldNotCompact()) //bad
```

### <a name = 'G-30'> G30: 함수는 한 가지만 해야 한다 </a>

- 함수를 짜다보면 한 함수 안에 여러 단락을 이어, 일련의 작업을 수행하고픈 유혹에 빠진다. 이런 함수는 한 가지만 수행하는 함수가 아니다.  
**한 가지만 수행하는 좀 더 작은 함수 여럿으로 나눠야 마땅하다.**  

```java
// bad
public void pay() {
    for (Employee e : employees) {
        if (e.isPayday()) {
            Money pay = e.calculatePay();
            e.deliverPay(pay);
        }
    }
}
```
```java
// good
public void pay() {
    for (Employee e : employees) {
        payIfNecessary(e);
    }
}

private void payIfNecessary(Employee e) {
    if (e.isPayday())
    calculateAndDeliverPay(e);
}

private void calculateAndDeliverPay(Employee e) {
    Money pay = e.calculatePay();
    e.deliverPay(pay);
}
```

### <a name = 'G-31'> G31: 숨겨진 시간적인 결합 </a>

- 때로는 시간적인 결합이 필요하지만, 시간적인 결합을 숨겨서는 안 된다.  
- 함수를 짤 때는 함수 인수를 적절히 배치해 함수가 호출되는 순서를 명백히 드러낸다. 

```java
// bad
public class MoogDiver {
    Gradient gradient;
    List<Spline> splines;

    public void dive(String reason) {
        saturateGradient();
        reticulateSplines();
        diveForMoog(reason);
    }
    ...
}
```

- 위 코드에서 세 함수가 실행되는 순서가 중요하다.  
- 먼저 `gradient`를 처리하기 위해 `saturateGradient()`를 호출하고 나서, 다시 `splines`를 처리하기 위해 `reticulateSplines()`를 호출하고, 마지막으로 `diveForMoog()`를 수행해야 한다.  
- 불행히도 위 코드는 이런 시간적인 결합을 강제하지 않는다. 프로그래머가 `reticulateSplines()`를 호출하고 `saturateGradient()`를 호출해 `UnsaturatedGradientException` 오류가 발생해도 막을 도리가 없다.

```java
// good
public class moogdiver {
    Gradient gradiient;
    List<Spline> splines;

    public void dive(String reason) {
        Gradient gradient = saturateGradient();
        List<Spline> splines = reticulateSplines(gradient);
        diveForMoog(splines, reason);
    }
    ...
}
```

- 위 코드는 연결 소자를 생성해 시간적인 결합을 노출한다. 각 함수가 내놓는 결과가 다음 함수에 필요하므로, 순서를 바꿔 호출할 수가 없다.  
의도적으로 추가한 구문적인 복잡성이 원래 있던 시간적인 복잡성을 드러낸 셈이다.  
- 제자리를 찾은 변수들이 시간적인 결합을 좀 더 명백히 드러낸다.

### <a name = 'G-32'> G32: 일관성을 유지하라 </a>

- 코드 구조를 잡을 때는 이유를 고민하고, 그 이유를 코드 구조로 명백히 표현하라.  
- 구조에 일관성이 없어 보인다면 남들이 맘대로 바꿔도 괜찮다고 생각한다. 시스템 전반에 걸쳐 구조가 일관성이 있다면 남들도 일관성을 따르고 보존한다.

### <a name = 'G-33'> G33: 경계 조건을 캡슐화하라 </a>

- 경계 조건은 빼먹거나 놓치기 쉬우므로, 한 곳에서 별도로 처리한다. 코드 여기저기에서 처리하지 않는다.  
- 코드 여기저기에 `+1`이나 `-1`을 흩어놓지 않는다.  
- 다음은 FIT에서 가져온 간단한 예제다.

```java
if(level + 1 < tags.length) {
    parts = new Parse(body, tags, level + 1, offset + endTag);
}
```
- `level + 1`이 두 번 나온다. 이런 경계 조건은 캡슐화하는 편이 좋다.
```java
int nextLevel =  level + 1;
if(nextLevel < tags.length) {
    parts = new Parse(body, tags, nextLevel, offset + endTag);
    body = null;
}
```

### <a name = 'G-34'> G34: 함수는 추상화 수준을 한 단계만 내려가야 한다 </a>

- 함수 내 모든 문장은 추상화 수준이 동일해야 한다. 그리고 그 추상화 수준은 함수 이름이 의미하는 작업보다 한 단게만 낮아야 한다.  
- 다음은 FitNess에서 가져온 코드다.

```java
public String render() throws Exception {
    StringBuffer html = new StringBuffer("<hr");
    if(size > 0)
        html.append(" size=\"").append(size + 1).append("\n");
    html.append(">");

    return html.toString();
}
```
- 위 함수는 &lt;hr&gt; 태그를 생성한다. &lt;hr&gt;높이는 size 변수로 지정한다.

- 위 함수에는 추상화 수준이 최소한 두 개가 섞여 있다.  

```
- 첫째는 수평선에 크기가 있다는 개념이다.  
- 둘째는 HR 태그 자체의 문법이다.  
```

- 나는 위 코드를 다음과 같이 수정했다.  
- size 변수 이름은 목적을 반여하게 적절히 변경하고, 추가된 대시 개수를 저장하게 했다.

```java
public String render() throws Exception {
    HtmlTag hr = new HtmlTag("hr");
    if (extraDashes > 0)
        hr.addAttribute("size", hrSize(extraDashes));
    return hr.html();
}

private String hrSize(int height) {
    int hrSize = height + 1;
    return String.format("%d", hrSize);
}
```

- 위 코드는 뒤섞인 추상화 수준을 멋지게 분리한다. 게다가 코드를 변경하면서 미묘한 오류도 잡아냈다.  
- 추상화 수준 분리는 리팩터링을 수행하는 가장 중요한 이유 중 하나다. 제대로 하기에 가장 어려운 작업 중 하나이기도 하다.  
- 함수에서 추상화 수준을 분리하면 앞서 드러나지 않았던 새로운 추상화 수준이 드러나는 경우가 빈번하다.

### <a name = 'G-35'> G35: 설정 정보는 최상위 단계에 둬라 </a>

- 추상화 최상위 단게에 둬야 할 기본값 상수나 설정 관련 상수를 저차원 함수에 숨겨서는 안 된다. 대신 고차원 함수에서 저차원 함수를 호출할 때 인수로 넘긴다. 

```java
// FitNess에서 가져온 코드
public static void main(String[] args) throws Exception {
    Argumnets arguments = parseCommandLine(args);
    ...
}

public class Arguments {
    public static final String DEFAULT_PATH = ".";
    public static final String DEFAULT_ROOT = "FitNesseRoot";
    public static final int DEFAULT_PORT = 90;
    public static final int DEFAULT_VERSION_DAYS = 14;
    ...
}
```

- FitNess 첫 행은 명령행 인수의 구문을 분석하는데, 각 인수 기본값은 Argument 클래스 맨 처음에 나온다. 

- 설정 관련 상수는 최상위 단계에 둔다. 그래야 변경하기도 쉽다.  
- 설정 관련된 나머지 코드는 인수로 넘긴다. 저차원 함수에 상수 값을 정의하면 안된다.

### <a name = 'G-36'> G36: 추이적 탐색을 피하라 </a>

- 일반적으로 한 모듈은 주변 모듈을 모를수록 좋다.  
- 이를 **디미터의 법칙**이라 부른다.  
- 자신이 직접 사용하는 모듈만 알아야 한다는 뜻이다. 내가 아는 모듈이 연이어 자신이 아는 모듈을 따라가며 시스템 전체를 휘저을 필요가 없다는 의미다.  

- 여러 모듈에서 `a.getB().getC()` 라는 형태를 사용한다면 설계와 아키텍처를 바꿔 `B`와 `C` 사이에 `Q`를 넣기가 쉽지 않다. `a.getB().getC()`를 모두 찾아 `a.getB().getQ().getC()`로 바꿔야 하니까. 너무 많은 모듈이 아키텍처를 너무 많이 알게되고, 그래서 아키텍처가 굳어진다.

- 내가 사용하는 모듈이 내게 필요한 서비스를 모두 제공해야 한다.  
원하는 메서드를 찾느라 객체 그래프를 따라 시스템을 탐색할 필요가 없어야 한다. 


## <a name='J'> 자바 p.396 </a>

### <a name = 'J-1'> J1: 긴 import 목록을 피하고 와일드카드를 사용하라 </a>

- 패키지에서 클래스를 둘 이상 사용한다면 와일드카드(*)를 사용해 패키지 전체를 가져오라.  

- 명시적인 import문은 강한 의존성을 생성하지만 와일드카드는 그렇지 않다.  
- 명시적으로 클래스를 import하면 그 클래스가 반드시 존재해야 하지만, 와일드카드로 패키지를 지정하면 특정 클래스가 존재할 필요는 없다. 그러므로 모듈 간에 결합성이 낮아진다.

- 와일드카드 import문은 때로 이름 충돌이나 모호성을 초래한다. 이름이 같으나 패키지가 다른 클래스는 명시적인 import문을 사용하거나 코드에서 클래스를 사용할 때 전체 경로를 명시한다. 


### <a name = 'J-2'> J2: 상수는 상속하지 않는다 </a>

- 이런 상황은 여러 차례 접했는데 매번 인상이 구겨진다.

```java
public class HourlyEmployee extends Employee {
    private int tenthsWorked;
    private double hourlyRate;

    public Money calculatePay() {
        int straightTime = Math.min(tenthsWorked, TENTHS_PER_WEEK);
        int overTime = tenthsWorked - straightTime;
        return new Money(hourlyRate * (tenthsWorked + OVERTIME_RATE * overTime));
    }
    ...
}
```

- `TENTHS_PER_WEEK`과 `OVERTIME_RATE`라는 상수는 어디서 왔을까?

```java
public interface PayrollConstants {
    public static final int TENTHS_PER_WEEK = 400;
    public static final double OVERTIME_RATE = 1.5;
}
```

- 참으로 끔직한 관행이다! 상수를 상속 계층 맨 위에 숨겨놨다.  
- 상속을 이렇게 사용하면 안 된다. 언어의 범위 규칙을 속이는 행위다.  
대신 `static import`를 사용하라.

```java
import static PayrollConstants.*;

public class HourlyEmployee extends Employee {
    private int tenthsWorked;
    private double hourlyRate;

    public Money calculatePay() {
        int straightTime = Math.min(tenthsWorked, TENTHS_PER_WEEK);
        int overTime = tenthsWorked - straightTime;
        return new Money(hourlyRate * (tenthsWorked + OVERTIME_RATE * overTime));
    }
    ...
}
```

### <a name = 'J-3'> J3: 상수 대 Enum </a>

- 자바 5는 enum을 제공한다. 마음껏 활용하라!  
- `public static final int` 라는 옛날 기교를 더 이상 사용할 필요가 없다. int는 코드에서 의미를 잃어버리기도 하지만, enum은 그렇지 않다. enum은 이름이 부여된 열거체(`enumeration`)에 속하기 때문이다.

- enum 문법을 자세히 살펴보면 메서드와 필드도 사용할 수 있다. int보다 훨씬 더 유연하고 서술적인 강력한 도구다. 다음은 좋은 예다.

```java
public class HourlyEmployee extends Employee {
    private int tenthsWorked;
    HourlyPayGrade grade;

    public Money calculatePay() {
        int straightTime = Math.min(tenthsWorked, TENTHS_PER_WEEK);
        int overTime = tenthsWorked - straightTime;
        return new Money(grade.rate() * (tenthsWorked + OVERTIME_RATE * overTime));
    }
    ...
}

public enum HourlyPayGrade {
    APPERNTICE {
        public double rate() {
            return 1.0;
        }
    },
    LIEUTENANT_JOURNEYMAN {
        public double rate() {
            return 1.2;
        }
    },
    JOURNEYMAN {
        public double rate() {
            return 1.5;
        }
    }, 
    MASTER {
        public double rate() {
            return 2.0;
        }
    };

    public abstract double rate();
}
```

## <a name = 'N'> 이름 p.399 </a>

### <a name = 'N-1'> N1: 서술적인 이름을 사용하라 </a>

- 이름을 성급학게 정하지 않는다. 서술적인 이름을 신중하게 고른다.  
소프트웨어 가독성의 90%는 이름이 결정한다. 그러므로 시간을 들여 현명한 이름을 선택하고 유효한 상태로 유지한다.  

```java
// bad
public int x() {
    int q = 0;
    int z = 0;
    for (int kk = 0; kk < 10; kk++) {
        if (l[z] == 10) {
            q += 10 + (l[z + 1] + l[z + 2]);
            z += 1;
        } else if (l[z] + l[z + 1] == 10) {
            q += 10 + l[z + 2];
            z += 2;
        } else  {
            q += l[z] + l[z + 1];
            z += 2;
        }
    }
    return q;
}
```
```java
// good
public int score() {
    int score = 0;
    int frame = 0;
    for (int frameNumber = 0; frameNumber < 10; frameNumber++) {
        if (isStrike(frame)) {
            score += 10 + nextTwoBallsForStrike(frame);
            frame += 1;
        } else if (isSpare(frame)) {
            score += 10 + nextBallForSpare(frame);
            frame += 2;
        } else {
            score += twoBallsInFrame(frame);
            frame += 2;
        }
    }
    return score;
}
```

- 신중하게 선택한 이름은 추가 설명을 포함한 코드보다 강력하다. 신중하게 선택한 이름을 보고 독자는 모듈 내 다른 함수가 하는 일을 짐작한다. `isStrike()` 함수를 찾아보면 '거의 예상한 대로' 돌아간다.

```java
private boolean isStrike(int frame) {
    return rolls[frame] == 10;
}
```

### <a name = 'N-2'> N2: 적절한 추상화 수준에서 이름을 선택하라 </a>

- 구현을 드러내는 이름은 피하고, 작업 대상 클래스나 함수가 위치하는 추상화 수준을 반영하는 이름을 선택하라.  
- 코드를 살펴볼 때마다 추상화 수준이 너무 낮은 변수 이름을 발견하리라. 발견할 때마다 기회를 잡아 바꿔놓아야 한다. 안정적인 코드를 만들려면 지속적인 개선과 노력이 필요하다.

```java
public interface Modem {
    boolean dial(String phoneNumber);
    boolean disconnect();
    boolean send(char c);
    char recv();
    String getConnectedPhoneNumber();
}
```
- 얼핏 봐서는 문제가 없어보이고, 함수는 모두 적절해 보인다.  
- 사실상 대다수 애플리케이션에서는 문제가 없다. 하지만 전화선에 연결되지 않고 전용선을 사용하는 모뎀을 고려해보라. 일부는 USB로 연결된 스위치에 포트 번호를 보낼지도 모른다. 그렇다면 전화번호라는 개념은 확실히 추상화 수준이 틀렸다.

```java
public interface Modem {
    boolean connect(String connectionLocator);
    boolean disconnect();
    boolean send(char c);
    char recv();
    String getConnectedPhoneNumber();
}
```

- 위 코드는 연결 대상의 이름을 더 이상 전화번호로 제한하지 않는다. 전화번호는 물론이고 다른 연결 방식에도 사용 가능하다.

### <a name = 'N-3'> N3: 가능하다면 표준 명명법을 사용하라 </a>

- 기존 명명법을 사용하는 이름은 이해가기 더 쉽다.  
예를 들어, `DECORATOR 패턴`을 활용한다면 장식하는 클래스 이름에 `Decorator`라는 단어를 사용해야 한다.  

- 패턴은 한 가지 표준에 불과하다.  
- 예를 들어, 자바에서 객체를 문자열로 변환하는 함수는 `toString` 이라는 이름을 많이 쓴다. 이런 이름은 관례를 따르는 편이 좋다.

- 프로젝트에 유효한 의미가 담긴 이름을 많이 사용할수록 독자가 코드를 이해하기 쉬워진다.

### <a name = 'N-4'> N4: 명확한 이름 </a>

- 함수나 변수의 목적을 명확히 밝히는 이름을 선택한다. 

```java
// FitNess에서 가져온 코드
private String doRename() throws Exception {
    if(refactorReferences)
        renameReferences();
    renamePage();

    pathToRename.removeNameFromEnd();
    pathToRename.addNameToEnd(newName);
    return PathParser.render(pathToRename);
}
```

- 이름만 봐서는 함수가 하는 일이 분명하지 않다. 아주 광범위하며 모호하며, 이름만으로도 `doRename`함수와 `renamePage`라는 함수 사이의 차이점이 전혀 드러나지 않는다.  
- `renamePageAndOptionallyAllReferences`라는 이름이 더 좋다. 아주 길지만 모듈에서 한 번만 호출되고, 길다는 단점을 서술성이 충분히 메꾼다.

### <a name = 'N-5'> N5: 긴 범위는 긴 이름을 사용하라 </a>

- 이름 길이는 범위 길이에 비례해야 한다.  
- 범위가 작으면 아주 짧은 이름을 사용해도 괜찮지만, 범위가 길어지면 긴 이름을 사용한다.  

```java
// "볼링 게임"에서 가져온 코드
private void rollMany(int n, int pins) {
    for (int i=0; i<n; i++) 
        g.roll(pins);
}
```

- 변수 `i`를 `rollCount`라고 썼으면 헷갈릴 터이다.  
- 이름이 짧은 변수나 함수는 범위가 길어지면 의미를 잃는다. 그러므로 이름 범위가 길수록 이름을 정확하고 길게 짓는다.

### <a name = 'N-6'> N6: 인코딩을 피하라 </a>

- 이름에 유형 정보나 범위 정보를 넣어서는 안 된다.  
- 중복된 정보이며 독자만 혼란하게 만든다.  
- 오늘날 환경은 이름을 조작하지 않고도 모든 정보를 제공한다. 헝가리안 표기법의 오염에서 이름을 보호하라.

### <a name = 'N-7'> N7: 이름으로 부수 효과를 설명하라 </a>

- 함수, 변수, 클래스가 하는 일을 모두 기술하는 이름을 사용한다.  
이름에 부수 효과를 숨기지 않는다. 

## <a name = 'T'> 테스트 p.404 </a>

### <a name = 'T-1'> T1: 불충분한 테스트 </a>

- 테스트 케이스는 잠재적으로 깨질 만한 부분을 **모두** 테스트해야 한다. 테스트 케이스가 확인하지 않는 조건이나 검증하지 않는 계산이 있다면 그 테스트는 불완전하다. 

### <a name = 'T-2'> T2: 커버리지 도구를 사용하라! </a>

- 커버리지 도구는 테스트가 빠뜨리는 공백을 알려준다.  
- 커버리지 도구를 사용하면 테스트가 불충분한 모듈, 클래스, 함수를 찾기가 쉬워진다.  
- 대다수 IDE는 테스트 커버리지를 시각적으로 표현하므로, 전혀 실행되지 않는 if 혹은 case 문 블록이 금방 드러난다.

### <a name = 'T-3'> T3: 사소한 테스트를 건너뛰지 마라 </a>

- 사소한 테스트는 짜기 쉽다. 사소한 테스트가 제공하는 문서적 가치는 구현에 드는 비용을 넘어선다.

### <a name = 'T-4'> T4: 무시한 테스트는 모호함을 뜻한다 </a>

- 때로는 요구사항이 불분명하기에 프로그램이 돌아가는 방식을 확신하기 어렵다.  
- 불분명한 요구사항은 테스트 케이스를 주석으로 처리하거나 테스트 케이스에 `@Ignore`를 붙여 표현한다. 선택 기준은 모호함이 존재하는 테스트 케이스가 컴파일이 가능한지 불가능한지에 달려있다.

### <a name = 'T-5'> T5: 경계 조건을 테스트하라 </a>

- 경계 조건을 각별히 신경 써서 테스트한다. 알고리즘의 중앙 조건은 올바로 짜놓고 경계 조건에서 실수하는 경우가 흔하다.

### <a name = 'T-6'> T6: 버그 주변은 철저히 테스트하라 </a>

- 버그는 서로 모이는 경향이 있다. 한 함수에서 버그를 발견했다면 그 함수를 철저히 테스트하는 편이 좋다. 십중팔구 다른 버그도 발견하리라.

### <a name = 'T-7'> T7: 실패 패턴을 살펴라 </a>

- 때로는 테스트 케이스가 실패하는 패턴으로 문제를 진단할 수 있다. 합리적인 순서로 정렬된 꼼꼼한 테스트 케이스는 실패 패턴을 드러낸다.  
- 간단한 예로, 입력이 5자를 넘기는 테스트 케이스가 모두 실패한다면? 함수 둘째 인수로 음수를 넘기는 테스트 케이스가 실패한다면?  
- 때로는 테스트 보고서에서 빨간색/녹색 패턴만 보고도 "아!"라는 깨달음을 얻는다.

### <a name = 'T-8'> T8: 테스트 커버리지 패턴을 살펴라 </a>

- 통과하는 테스트가 실행하거나 실행하지 않는 코드를 살펴보면 실패하는 테스트 케이스의 실패 원인이 드러난다.

### <a name = 'T-9'> T9: 테스트는 빨라야 한다 </a>

- 느린 테스트 케이스는 실행하지 않게 된다. 일정이 촉박하면 느린 테스트 케이스를 제일 먼저 건너뛴다. 그러므로 테스트 케이스가 빨리 돌아가게 최대한 노력한다. 

## <a name = 'conclusion'> 결론 p.406 </a>

- 이 장에서 소개한 휴리스틱과 냄새 목록이 완전하다 말하기는 어렵다. 여기서 소개한 목록은 가치 체계를 피력할 뿐이다.

- 사실상 가치 체계는 이 책의 주제이자 목표다. 일군의 규칙만 따른다고 깨끗한 코드가 얻어지지 않는다. 휴리스틱 목록을 익힌다고 소프트웨어 장인이 되지는 못한다.  
- 전문가 정신과 장인 정신은 가치에서 나온다. 그 가치에 기반한 규율과 절제가 필요하다.

# 4장. 주석 p.68

잘 달린 주석은 그 어떤 정보보다 유용하다. 하지만 반대로 근거 없는 주석은 코드를 이해하기 어렵게 만든다. 주석은 오래될수록 코드에서 멀어진다.
이유는 코드는 계속해서 변화하고 진화하기 때문이다. 이렇게 부정확한 **주석은 없는 주석보다 못하기 때문에 프로그래머는 주석을 엄격하게 관리**해야 한다.

# 목차

1. [주석은 나쁜 코드를 보완하지 못한다](#주석은-나쁜-코드를-보완하지-못한다-p.69)
1. [코드로 의도를 표한하라!](#코드로-의도를-표현하라!-p.70)
1. [좋은 주석](#좋은-주석-p.70)
   1. [법적인 주석](#법적인-주석)
   1. [정보를 제공하는 주석](#정보를-제공하는-주석)
   1. [의도를 설명하는 주석](#의도를-설명하는-주석)
   1. [의미를 명료하게 밝히는 주석](#의미를-명료하게-밝히는-주석)
   1. [결과를 경고하는 주석](#결과를-경고하는-주석)
   1. [TODO 주석](#TODO-주석)
   1. [중요성을 강조하는 주석](#중요성을-강조하는-주석)
1. [나쁜 주석](#나쁜-주석-p.75)
   1. [같은 이야기를 중복하는 주석](#같은-이야기를-중복하는-주석)
   1. [오해할 여지가 있는 주석](#오해할-여지가-있는-주석)
   1. [의무적으로 다는 주석](#의무적으로-다는-주석)
   1. [이력을 기록하는 주석](#이력을-기록하는-주석)
   1. [있으나 마나 한 주석](#있으나-마나-한-주석)
   1. [무서운 잡음](#무서운-잡음)
   1. [함수나 변수로 표현할 수 있다면 주석을 달지 마라](#함수나-변수로-표현할-수-있다면-주석을-달지-마라)
   1. [닫는 괄호에 다는 주석](#닫는-괄호에-다는-주석)
   1. [공로를 돌리거나 저자를 표시하는 주석](#공로를-돌리거나-저자를-표시하는-주석)
   1. [주석으로 처리한 코드](#주석으로-처리한-코드)
   1. [HTML 주석](#HTML-주석)
   1. [전역 정보](#전역-정보)
   1. [너무 많은 정보](#너무-많은-정보)
   1. [모호한 관계](#모호한-관계)
   1. [함수 헤더](#함수-헤더)
   1. [리팩토링 예제](#리팩토링-예제-p.90)

<br>

## 주석은 나쁜 코드를 보완하지 못한다 p.69

**코드에 주석을 추가하는 일반적인 이유는 코드 품질이 나쁘기 때문이다.** 표현력이 풍부하고 깔끔한 코드에 주석이 거의 없는 코드가 복잡하고 어수선한 코드에 주석이 많이 달린 코드보다 좋다.
그렇기에 **난장판을 주석으로 설명하기보다는 그 난장판 코드를 치우는 데 시간을 보내야 한다.**

## 코드로 의도를 표현하라! p.70

```java
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다.
if ((employee.flags) & HOURLY_FLAG && (employee.age > 65))
```

위 코드를 보게 되면 여러 조건이 걸려있는 것은 알겠지만 주석이 없다면 무엇을 뜻하는지 알 수 없다.

``` java
if(employee.isEligibleForFullBenefits())
```

위와 같이 주석으로 달려는 설명을 함수로 만들어서 표현해도 충분하다.

## 좋은 주석 p.70

어떤 주석은 필요하거나 유익하다.

### 법적인 주석

회사가 정립한 구현 표준에 맞춰 법적인 이유로 특정 주석을 명시하는 경우

```java
// Copyright (C) 2003, 2004, 2005 by Object Mentor, Inc. All rights reserved.
// GNU General Public License 버전 2 이상을 따르는 조건으로 배포한다.
```

### 정보를 제공하는 주석

기본적인 정보를 주석으로 제공하면 편리하다.

```java
// 테스트 중인 Responder 인스턴스를 반환한다.
protected abstract Responder responderInstacne();
```

하지만, 이런 주석이 유용할지라도 결국 함수 이름에 정보를 담는 편이 좋다.

```java
protected abstract Responder responderBeingTested();
```

또 다른 예제로

```java
// kk:mm:ss EEE, MMM dd, yyyy 형식이다.
Pattern timeMatcher = Pattern.compile(
        "\\d*:\\d*:\\d* \\w*, \\w* \\d*, \\d*");
```

해당 주석은 코드에서 사용된 정규표현식이 시각과 날짜를 뜻한다는 것을 설명한다.

### 의도를 설명하는 주석

주석은 구현을 이해하게 도와주는 선을 넘어 결정에 깔린 의도까지 설명한다.

```java
public int compareTo(object o) {
        // 생략
        return 1; // 오른쪽 유형이므로 정렬 순위가 더 높다.
}

// 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다.
for (int i = 0; i < 25000; i++) {
        WidgetBuilderThread widgetBuilderThread = new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
        Thread thread = new Thread(widgetBuilderThread);
        thread.start();
}
```

해당 소스의 문제 해결 방식에는 동의하지 않더라도 저자의 의도는 분명히 드러난다.

### 의미를 명료하게 밝히는 주석

모호한 인수나 반환 값은 그 의미를 읽기 좋게 표현하면 이해하기 쉬워진다.

```java
assertTrue(a.compareTo(a) == 0); // a == a
assertTrue(a.compareTo(b) != 0); // a != b
assertTrue(a.compareTo(b) == -1); // a < b
assertTrue(a.compareTo(b) == 1); // a > b
```

### 결과를 경고하는 주석

프로그래머에게 결과를 경고할 목적으로 주석을 사용한다.

```java
// 여유 시간이 충분하지 않다면 실행하지 마십시오.
public void _testWithReallyBigFile() {}
```

위 예제는 특정 테스트케이스를 끄는 예제이다. Junit4가 나온 후에는 @Ignore("실행이 너무 오래 걸린다.")으로 쓰고 전에는 메소드 앞에 _를 붙여 적는 것이 관례였다.

```java
public static SimpleDateFormat makeStandardHttpDateFormat() {
        // SimpleDateFormat은 스레드에 안전하지 못하다.
        // 따라서 각 인스턴스는 독립적으로 생성해야 한다.
        SimpleDateFormat df = new SimpleDateFormat("EEE, dd, MMM yyyy HH:mm:ss z");
        df.setTimeZone(TimeZone.getTimeZone("GMT"));
        return df;
}
```

다음과 같이 경고 주석을 적게 되면 프로그램 효율을 높이기 위해 정적 초기화 함수를 사용하려던 열성적인 프로그래머가 주석 때문에 실수를 면할 수 있다.

### TODO 주석

"앞으로의 할 일"을 TODO 주석으로 남겨두면 편하다. 더 이상 필요 없는 기능을 삭제하라는 알림, 누군가에게 문제를 봐달라는 요청, 더 좋은 이름을 떠올려달라는 부탁, 앞으로 발생할 이벤트에 맞춰 코드를 고치라는 주의 등에 유용하다.

하지만 어떻게든 나쁜 코드를 남겨 놓는 핑계가 되어서는 안 된다. 그렇기에 주기적으로 TODO 주석을 점검해 없애도 괜찮은 주석은 없애는 것을 권고한다.

요새는 IDE에서 TODO를 모두 찾아낼 수 있다. Intellij의 경우 ⌘ + 6을 클릭하게 되면 아래 토글 버튼에 TODO라고 있는 것을 확인할 수 있고 누르면 모든 TODO를 검색해서 확인할 수 있다.

```java
// TODO 해당 메소드는 필요 없다.
public void test(){}
```

### 중요성을 강조하는 주석

자칫 대수롭지 않다고 여겨질 뭔가의 중요성을 강조하기 위해서 주석을 사용한다.

```java
// 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
// 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다.
String listItemContent = match.group(3).trim();
```

## 나쁜 주석 p.75

### 같은 이야기를 중복하는 주석

간단한 함수임에도 불구하고, 헤더에 달린 주석이 코드 내용과 중복되는 것을 말한다. 마치 엔진 후드를 열어볼 필요가 없다며 고객에게 아양 떠는 중고차 판매원과 비슷하다.

```java
public abstract class ContainerBase implements Container, Lifecycle, Pipeline, MBeanRegistration, Serializable {
        /**
        * 이 컴포넌트의 프로세서 지연값
        */
        protected int backgroundProcessorDelay = -1;
        /**
        * 이 컴포넌트를 지원하기 위한 생명주기 이벤트
        */
        protected LifecycleSupport lifecycle = new LifecycleSupport(this);
        /**
        * 이 컴포넌트를 위한 컨테이너 이벤트 Listener
        */
        protected ArrayList listeners = new ArrayList();
}
```

톰캣에서 가져온 코드이다. 이러한 중복된 주석은 코드를 지저분하게만 하고 정신없게 만들어 기록이라는 목적에는 전혀 기여하지 못한다.

### 오해할 여지가 있는 주석

```java
// this.closed가 true일 때 반환되는 유틸리티 메서드이다.
// 타임아웃에 도달하면 예외를 던진다.
public synchronized void waitForClose(final long timeoutMillis) throws Exception {
        if (!closed) {
                wait(timeoutMillis);
                if (!closed) {
                        throw new Exception("MockResponseSender cloud not be closed");
                }
        }
}
```

this.closed가 true로 변하는 순간에 메서드는 반환되지 않고 this.closed가 true**여야** 메서드는 반환된다. 아니면 타임아웃을 기다렸다 this.closed가 그래도 true가 **아니면** 예외를 던진다.

(코드보다 읽기도 어려운) 주석에 담긴 '살짝 잘못된 정보'로 인해 this.closed가 true로 변하는 순간에 함수가 반환되리라는 생각으로 어느 프로그래머가 경솔하게 함수를 호출할지도 모른다. 그래서 프로그래머는 그 문제 원인을 찾기 위해 골머리를 앓을 것이다.

### 의무적으로 다는 주석

모든 함수에 javadocs를 달거나 모든 변수에 주석을 달아야 한다는 규칙은 어리석다. 이런 주석은 코드를 복잡하게 만들거나 거짓말을 퍼뜨리고, 혼동과 무질서를 초래한다.

```java
/**
 *
 * @param title CD 제목
 * @param author CD 저자
 * @param tracks CD 트랙 숫자
 * @param durationInMinutes CD 길이(단위: 분)
 */
 public void addCD(String title, String author, int tracks, int durationInMinutes) {
         CD cd = new CD();
         cd.title = title;
         cd.author = author;
         cd.tracks = tracks;
         cd.duration = durationInMinutes;
         cdList.add(cd);
 }
```

### 이력을 기록하는 주석

모듈의 첫머리 주석에는 지금까지 모듈에 변경을 모두 기록하는 일종의 로그가 되었다. 예전에는 첫머리에 변경 이력을 기록하고 관리하는 관례가 바람직했지만, 당시에는 소스 코드 관리 시스템이 없었으니까 그렇다. 이제는 혼란만 가중시킬 수 있으니 제거하는 편이 좋다.

```java
/*
* 변경 이력 (11-Oct-2001부터)
* ------------------------
* 11-Oct-2001 : ~~
*
* 05-Nov-2001 : ~~
*/
```

### 있으나 마나 한 주석

당연한 사실을 언급하며 새로운 정보를 제공하지 못하는 주석이다.

```java
/**
 * 기본 생성자
 */
protected AnnualDateRule() {
}
/**
 * 월 중 일자
 */
 private int dayOfMonth;
 /**
 * 월 중 일자를 반환한다.
 * @return 월 중 일자
 */
 public int getDayOfMonth() {
         return dayOfMonth;
 }
```

위와 같이 지나치게 참견하게 되면 개발자가 주석을 무시하는 습관에 빠질 수 있다.

### 무서운 잡음

때로는 javadocs도 잡음이다. 다음을 잘 알려진 오픈 소스 라이브러리에서 가져온 코드이다. 아래 나오는 javadocs는 목적이 존재하지 않고 단지 문서를 제공해야 한다는 잘못된 욕심으로 탄생한 잡음일 뿐이다.

```java
/* The name. */
private String name;
/* The version. */
private String version;
/* The licenceName. */
private String licenㅊeName;
/* The version. */
private String info;
```

위 예제처럼 문서를 제공해야 한다는 욕심으로 의미 없는 주석을 달게 되면 실수로 오타가 생길 수도 있고 복사해 붙여넣기로 인한 잡음이 생길 수 있다.

### 함수나 변수로 표현할 수 있다면 주석을 달지 마라

```java
// Before
// 전역 목록 <smodule>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
if (smodule.getDependSubsystems().contains(subSysMod.getSubSystem())) {}

// After
ArrayList moduleDependees = smodule.getDependSubsystems();
String ourSubSystem = subSysMod.getSubSystem();
if (moduleDependees.contains(ourSubSystem)) {}
```

after와 같이 주석이 필요하지 않도록 코드를 개선하는 편이 더 좋다.

### 닫는 괄호에 다는 주석

중첩이 심하고 장황한 함수라면 의미 있을지는 모르지만 작고 캡슐화된 함수에는 잡음일 뿐이다. 그러므로 닫는 괄호에 주석을 달아야겠다는 생각이 든다면 주석 대신에 함수를 줄이려고 시도하자.

```java
public class Test {
        public static void main(String[] args) {
                try {
                        while() {

                        } // while
                } // try
                catch {

                } // catch
        } // main
}
```

### 공로를 돌리거나 저자를 표시하는 주석

소스 코드 관리 시스템은 누가 언제 무엇을 추가했는지 귀신처럼 기억한다. 저자 이름으로 코드를 오염시킬 수 있다. 앞서 말했던 이력을 기록하는 주석과 같이 소스 코드 관리 시스템에 저장하는 것이 좋다.

### 주석으로 처리한 코드

주석으로 처리된 코드는 사람들이 지우기를 주저한다. 이유가 있어 남겨놓았으리라고, 중요하니까 지우면 안 된다고 생각한다. 그렇게 질 나쁜 와인병 바닥에 앙금이 쌓이듯 쓸모없는 코드가 점차 쌓이게 된다.

지금은 1960년대가 아니다. 우수한 소스 코드 관리 시스템이 있기 때문에 우리 대신에 시스템이 코드를 기억해준다. 그렇기에 과감하게 코드를 삭제해라!

### HTML 주석

주석에 HTML 태그를 삽입해야 하는 책임은 프로그래머가 아니라 도구가 져야 한다.

### 전역 정보

주석을 달아야 한다면 근처에 있는 코드만 기술하라. 코드 일부에 주석을 달면서 시스템의 전반적인 정보를 기술하지 마라.

```java
/**
 * 적합성 테스트가 동작하는 포트: 기본값 <b>8082</b>.
 *
 *
 * @param fitnessePort
 */
public void setFitnessePort(int fitnessPort) {
        this.fitnessePort = fitnessePort;
}
```

해당 코드는 중복이 일어났다는 사실 외에도 주석은 기본 포트 정보를 기술한다. 하지만 함수 자체는 포트 기본값을 전혀 통제하지 못하고 아래 함수가 아니라 시스템 어딘가에 다른 함수를 설명한다는 말이다.
이 말은 시스템 기본값을 설정하는 코드를 변경하더라도 현재 위에 있는 주석이 변할 거라는 것을 보장하지 않는다.

### 너무 많은 정보

주석에다 흥미로운 역사나 관련 없는 정보를 장황하게 늘어놓지 마라. 독자에게는 불가사의한 정보일 뿐이다.

### 모호한 관계

주석과 주석이 설명하는 코드는 둘 사이 관계가 명백해야 한다.

```java
/*
 * 모든 픽셀을 담은 만큼 충분한 배열로 시작한다(여기에 필터 바이트를 더한다)
 * 그리고 헤더 정보를 위해 200바이트를 더한다.
 */
this.pngBytes = new byte[((this.width + 1) * this.height * 3) + 200];
```

여기서 필터 바이트는 무엇을 나타내는 것일까?? +1? *3? +200? 아니면 세 개 다를 가르키는지 설명이 부족해서 알 수가 없다. 주석을 다시 설명해야 되는 지경에 이른 이런 주석은 좋지 않다.

### 함수 헤더

짧은 함수는 긴 설명이 필요 없다. 짧고 한 가지만 수행하며 이름을 잘 붙인 함수가 주석으로 헤더를 추가한 함수보다 훨씬 좋다.

### 리팩토링 예제 p.90
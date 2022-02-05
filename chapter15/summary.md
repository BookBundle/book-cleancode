# 15장. JUnit 들여다보기 p.324

1. [JUnit 프레임워크](#JUnit-프레임워크-p.324)
   1. [ComparisonCompactor 모듈에 대한 테스트 케이스](#ComparisonCompactor-모듈에-대한-테스트-케이스)
   1. [ComparisonCompactor 모듈](#ComparisonCompactor-모듈)
   1. [개선하는 방법](#개선하는-방법)
      1. [인코딩을 피해라](#인코딩을-피해라)
      1. [조건문 캡슐화](#조건문-캡슐화)
      1. [이름을 명확하게 붙여라](#이름을-명확하게-붙여라)
      1. [부정문보다는 긍정문으로 표현해라](#부정문보다는-긍정문으로-표현해라)
      1. [이름으로 부수 효과를 표현해라](#이름으로-부수-효과를-표현해라)
      1. [함수는 한 가지만 실행해야 한다](#함수는-한-가지만-실행해야-한다)
      1. [일관성을 맞춰라](#일관성을-맞춰라)
      1. [서술적인 이름을 사용해라](#서술적인-이름을-사용해라)
      1. [숨겨진 시간적인 결합](#숨겨진-시간적인-결합)
      1. [경계 조건을 캡슐화하라](#경계-조건을-캡슐화하라)
1. [최종 코드](#최종-코드-p.338)
1. [결론](#결론-p.341)

## JUnit 프레임워크 p.324

- 살펴볼 예제는 ComparisonCompactor라는 모듈이다.
- ABCDE와 ABXDE를 받으면 <...B[X]D..>를 반환한다.

### ComparisonCompactor 모듈에 대한 테스트 케이스

- 이 모듈에 대한 테스트 케이스는 코드 커버리지 분석이 100%가 나온다.
- 테스트 케이스가 모든 행, 모든 if문, 모든 for 문을 실행한다는 의미이다.

```java
package junit.tests.framework;

import junit.framework.ComparisonCompactor;
import junit.framework.TestCase;

public class ComparisonCompactorTest extends TestCase {

    public void testMessage() {
        String failure = new ComparisonCompactor(0, "b", "c").compact("a");
        assertTrue("a expected:<[b]> but was:<[c]>".equals(failure))ㅋ;
    }

    public void testStartSame() {
        String failure = new ComparisonCompactor(1, "ba", "bc").compact(null);
        assertEquals("expected:<b[a]> but was:<b[c]>", failure);
    }

    public void testEndSame() {
        String failure = new ComparisonCompactor(1, "ab", "cb").compact(null);
        assertEquals("expected:<[a]b> but was:<[c]b>", failure);
    }

    public void testSame() {
        String failure = new ComparisonCompactor(1, "ab", "ab").compact(null);
        assertEquals("expected:<ab> but was:<ab>", failure);
    }

    public void testNoContextStartAndEndSame() {
        String failure = new ComparisonCompactor(0, "abc", "adc").compact(null);
        assertEquals("expected:<...[b]...> but was:<...[d]...>", failure);
    }

    public void testStartAndEndContext() {
        String failure = new ComparisonCompactor(1, "abc", "adc").compact(null);
        assertEquals("expected:<a[b]c> but was:<a[d]c>", failure);
    }

    public void testStartAndEndContextWithEllipses() {
        String failure = new ComparisonCompactor(1, "abcde", "abfde").compact(null);
        assertEquals("expected:<...b[c]d...> but was:<...b[f]d...>", failure);
    }

    public void testComparisonErrorStartSameComplete() {
        String failure = new ComparisonCompactor(2, "ab", "abc").compact(null);
        assertEquals("expected:<ab[]> but was:<ab[c]>", failure);
    }

    public void testComparisonErrorEndSameComplete() {
        String failure = new ComparisonCompactor(0, "bc", "abc").compact(null);
        assertEquals("expected:<[]...> but was:<[a]...>", failure);
    }

    public void testComparisonErrorEndSameCompleteContext() {
        String failure = new ComparisonCompactor(2, "bc", "abc").compact(null);
        assertEquals("expected:<[]bc> but was:<[a]bc>", failure);
    }

    public void testComparisonErrorOverlappingMatches() {
        String failure = new ComparisonCompactor(0, "abc", "abbc").compact(null);
        assertEquals("expected:<...[]...> but was:<...[b]...>", failure);
    }

    public void testComparisonErrorOverlappingMatchesContext() {
        String failure = new ComparisonCompactor(2, "abc", "abbc").compact(null);
        assertEquals("expected:<ab[]c> but was:<ab[b]c>", failure);
    }

    public void testComparisonErrorOverlappingMatches2() {
        String failure = new ComparisonCompactor(0, "abcdde", "abcde").compact(null);
        assertEquals("expected:<...[d]...> but was:<...[]...>", failure);
    }

    public void testComparisonErrorOverlappingMatches2Context() {
        String failure = new ComparisonCompactor(2, "abcdde", "abcde").compact(null);
        assertEquals("expected:<...cd[d]e> but was:<...cd[]e>", failure);
    }

    public void testComparisonErrorWithActualNull() {
        String failure = new ComparisonCompactor(0, "a", null).compact(null);
        assertEquals("expected:<a> but was:<null>", failure);
    }

    public void testComparisonErrorWithActualNullContext() {
        String failure = new ComparisonCompactor(2, "a", null).compact(null);
        assertEquals("expected:<a> but was:<null>", failure);
    }

    public void testComparisonErrorWithExpectedNull() {
        String failure = new ComparisonCompactor(0, null, "a").compact(null);
        assertEquals("expected:<null> but was:<a>", failure);
    }

    public void testComparisonErrorWithExpectedNullContext() {
        String failure = new ComparisonCompactor(2, null, "a").compact(null);
        assertEquals("expected:<null> but was:<a>", failure);
    }

    public void testBug609972() {
        String failure = new ComparisonCompactor(10, "S&P500", "0").compact(null);
        assertEquals("expected:<[S&P50]0> but was:<[]0>", failure);
    }
}
```

### ComparisonCompactor 모듈

- 코드가 잘 분리되었고, 표현력이 적절하며, 구조가 단순하다.

```java
public class ComparisonCompactor {

    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String DELTA_START = "[";

    private int fContextLength;
    private String fExpected;
    private String fActual;
    private int fPrefix;
    private int fSuffix;

    public ComparisonCompactor(int contextLength, String expected, String actual) {
        fContextLength = contextLength;
        fExpected = expected;
        fActual = actual;
    }

    public String compact(String message) {
        if (fExpected == null || fActual == null || areStringsEqual()) {
            return Assert.format(message, fExpected, fActual);
        }

        findCommonPrefix();
        findCommonSuffix();
        String expected = compactString(fExpected);
        String actual = compactString(fActual);
        return Assert.format(message, expected, actual);
    }

    private String compactString(String source) {
        String result = DELTA_START + source.substring(fPrefix, source.length() - fSuffix + 1) + DELTA_END;
        if (fPrefix > 0) {
            result = computeCommonPrefix() + result;
        }
        if (fSuffix > 0) {
            result = result + computeCommonSuffix();
        }
        return result;
    }

    private void findCommonPrefix() {
        fPrefix = 0;
        int end = Math.min(fExpected.length(), fActual.length());
        for (; fPrefix < end; fPrefix++) {
            if (fExpected.charAt(fPrefix) != fActual.charAt(fPrefix)) {
                break;
            }
        }
    }

    private void findCommonSuffix() {
        int expectedSuffix = fExpected.length() - 1;
        int actualSuffix = fActual.length() - 1;
        for (; actualSuffix >= fPrefix && expectedSuffix >= fPrefix; actualSuffix--, expectedSuffix--) {
            if (fExpected.charAt(expectedSuffix) != fActual.charAt(actualSuffix)) {
                break;
            }
        }
        fSuffix = fExpected.length() - expectedSuffix;
    }

    private String computeCommonPrefix() {
        return (fPrefix > fContextLength ? ELLIPSIS : "") + fExpected.substring(Math.max(0, fPrefix - fContextLength), fPrefix);
    }

    private String computeCommonSuffix() {
        int end = Math.min(fExpected.length() - fSuffix + 1 + fContextLength, fExpected.length()); // 경계조건
        return fExpected.substring(fExpected.length() - fSuffix + 1, end) + (fExpected.length() - fSuffix + 1 < fExpected.length() - fContextLength ? ELLIPSIS : ""); // 경계조건
    }

    private boolean areStringsEqual() {
        return fExpected.equals(fActual);
    }
}
```

- 해당 모듈을 아주 좋은 상태로 남겨두었지만 **보이스카우트 규칙에 따라 코드를 개선해야 한다.**

> **보이스카우트 규칙**
>
> 처음 왔을 때보다 더 깨끗하게 해놓고 떠나야 한다.

### 개선하는 방법

#### 인코딩을 피해라

- 가장 먼저 거슬리는 부분은 멤버 변수 앞에 붙은 접두사 f다.
- 오늘날 사용하는 개발 환경에서는 이처럼 변수 이름에 범위를 명시할 필요가 없기에 모두 제거한다.

```java
private int contextLength;
private String expected;
private String actual;
private int prefix;
private int suffix;​
```

#### 조건문 캡슐화

- compact 함수 시작부에 캡슐화되지 않은 조건문이 보인다.

```java
public String compact(String message) {
    if (expected == null || actual == null || areStringsEqual()) {
        return Assert.format(message, expected, actual);
    }

    findCommonPrefix();
    findCommonSuffix();
    String expected = compactString(this.expected);
    String actual = compactString(this.actual);
    return Assert.format(message, expected, actual);
}
```

- 의도를 명확히 표현하려면 조건문을 캡슐화해야 한다.
- 즉, 조건문을 메소드로 뽑아내 적절한 이름을 붙인다.

```java
public String compact(String message) {
    if (shouldNotCompact()) {
        return Assert.format(message, expected, actual);
    }

    findCommonPrefix();
    findCommonSuffix();
    String expected = compactString(this.expected);
    String actual = compactString(this.actual);
    return Assert.format(message, expected, actual);
}

private boolean shouldNotCompact() {
    return expected == null || actual == null || areStringsEqual();
}
```

#### 이름을 명확하게 붙여라

- compact 함수에 이미 expected 변수 이름이 있기에 this.expected와 this.actual도 눈에 거슬린다.
- f 접두사를 빼버리는 바람에 생긴 결과이고 다음과 같이 의미를 명확하게 바꿔주는 것이 좋다.

```java
String compactExpected = compactString(expected);
String compactActual = compactString(actual);
```

#### 부정문보다는 긍정문으로 표현해라

- 부정문은 긍정문보다 이해하기 약간 더 어렵다.

```java
public String compact(String message) {
    if (canBeCompacted()) { // if (shouldNotCompact()) {
        findCommonPrefix();
        findCommonSuffix();
        String compactExpected = compactString(expected);
        String compactActual = compactString(actual);
        return Assert.format(message, compactExpected, compactActual);
    } else {
        return Assert.format(message, expected, actual);
    }       
}

// private boolean shouldNotCompact() {
//     return expected == null || actual == null || areStringsEqual();
// }

private boolean canBeCompacted() {
    return expected != null && actual != null && !areStringsEqual();
}
```

#### 이름으로 부수 효과를 표현해라

- 문자열을 압축하는 함수라지만 실제 canBeCompacted가 false이면 압축하지 않는다.
- 오류 점검이라는 부가 단계가 숨겨지고 해당 함수는 압축된 문자열이 아닌 형식이 갖춰진 문자열을 반환한다.

```java
// public String compact(String message) { ... }

public String formatCompactedComparison(String message) { ... }
```

#### 함수는 한 가지만 실행해야 한다

- formatCompactedComparison은 형식을 맞추는 작업을 전적으로 맡게 한다.
- 압축하는 작업은 compactExpectedAndActual이라는 메소드로 분리하여 구현한다.

```java
...

private String compactExpected;
private String compactActual;

...

public String formatCompactedComparison(String message) {
    if (canBeCompacted()) {
        compactExpectedAndActual();
        return Assert.format(message, compactExpected, compactActual);
    } else {
        return Assert.format(message, expected, actual);
    }       
}

private compactExpectedAndActual() {
    findCommonPrefix();
    findCommonSuffix();
    compactExpected = compactString(expected);
    compactActual = compactString(actual);
}
```

#### 일관성을 맞춰라

- 위에서 compactExpected와 compactActual이 멤버 변수로 승격했다는 사실을 주의해야 한다.
- 새로 만들어진 함수 끝에서 두 줄은 변수를 반환하지만, 앞에서 두 줄은 반환하지 않는다.
- 함수 사용 방식이 일관적이지 못하기에 해당 함수들을 변경하여 접두사와 접미사 값을 반환하게 한다.

```java
private compactExpectedAndActual() {
    // findCommonPrefix();
    // findCommonSuffix();
    prefixIndex = findCommonPrefix();
    suffixIndex = findCommonSuffix();
    String compactExpected = compactString(expected);
    String compactActual = compactString(actual);
}

private int findCommonPrefix() {
    int prefixIndex = 0;
    int end = Math.min(expected.length(), actual.length());
    for (; prefixIndex < end; prefixIndex++) {
        if (expected.charAt(prefixIndex) != actual.charAt(prefixIndex)) {
            break;
        }
    }
    return prefixIndex;
}

private int findCommonSuffix() {
    int expectedSuffix = expected.length() - 1;
    int actualSuffix = actual.length() - 1;
    for (; actualSuffix >= prefixIndex && expectedSuffix >= prefix; actualSuffix--, expectedSuffix--) {
        if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix)) {
            break;
        }
    }
    return expected.length() - expectedSuffix;
}
```

#### 서술적인 이름을 사용해라

- prefix, suffix는 색인 위치를 나타내기 때문에 뒤에 index를 붙여주었다.

```java
...

// prefix
// suffix
prefixIndex
suffixIndex

...
```

#### 숨겨진 시간적인 결합

- findCommonSuffix는 숨겨진 시간적인 결합이 존재한다.
- findCommonSuffix는 findCommonPrefix가 prefix를 계산한다는 사실에 의존한다.
- 만약 이 둘을 잘못된 순서로 호출하게 되면 밤샘 디버깅이라는 고생문이 열리게 될 것이다.
- 이를 시간 결합을 외부로 노출해주기 위해서 findCommonSuffix에 인자로 prefixIndex를 받았다.

```java
private compactExpectedAndActual() {
    prefixIndex = findCommonPrefix();
    suffixIndex = findCommonSuffix(prefixIndex);
    String compactExpected = compactString(expected);
    String compactActual = compactString(actual);
}

private int findCommonSuffix(int prefixIndex) {
    int expectedSuffix = expected.length() - 1;
    int actualSuffix = actual.length() - 1;
    for (; actualSuffix >= prefixIndex && expectedSuffix >= prefix; actualSuffix--, expectedSuffix--) {
        if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix)) {
            break;
        }
    }
    return expected.length() - expectedSuffix;
}
```

- 위 방법은 썩 내키지 않는 방식이다.
- prefixIndex를 인자로 전달하는 방식은 다소 자의적이다.
- 함수 호출 순서는 확실히 정해지지만 prefixIndex가 필요한 이유는 설명하지 못한다.
- 아래 코드로 변경하게 되면 호출하는 순서가 앞서 고친 코드보다 훨씬 더 분명해진다.

```java
private compactExpectedAndActual() {
    findCommonPrefixAndSuffix();
    String compactExpected = compactString(expected);
    String compactActual = compactString(actual);
}

private void findCommonPrefixAndSuffix() {
    findCommonPrefix();
    int expectedSuffix = expected.length() - 1;
    int actualSuffix = actual.length() - 1;
    for (; actualSuffix >= prefixIndex && expectedSuffix >= prefix; actualSuffix--, expectedSuffix--) {
        if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix)) {
            break;
        }
    }
    suffixIndex = expected.length() - expectedSuffix;
}

private void findCommonPrefix() {
    int prefixIndex = 0;
    int end = Math.min(expected.length(), actual.length());
    for (; prefixIndex < end; prefixIndex++) {
        if (expected.charAt(prefixIndex) != actual.charAt(prefixIndex)) {
            break;
        }
    }
}
```

- 함수가 지저분 해보이니 함수를 정리해보자

```java
private void findCommonPrefixAndSuffix() {
    findCommonPrefix();
    int suffixLength = 1;
    for (; suffixOverlapsPrefix(suffixLength); suffixLength++) {
        if (charFromEnd(expected, suffixLength) != charFromEnd(actual, suffixLength)) {
            break;
        }
    }
    suffixIndex = suffixLength;
}

private char charFromEnd(String s, int i) {
    return s.charAt(s.length() - i);
}

private boolean suffixOverlapsPrefix(int suffixLength) {
    return actual.length() = suffixLength < prefixLength || expected.length() - suffixLength < prefixLength;
}
```

#### 경계 조건을 캡슐화하라

- 코드를 고치고 나니 suffixIndex가 실제로는 접미사 길이라는 사실이 드러난다. 이 말은 이름이 적절치 않다는 것이다.
- length와 index는 동의어이다. 하지만 length가 더 타당하다.
- 이유는 suffixIndex는 실제 0에서 시작하지 않고 1에서 시작하기 때문이다.
- computeCommonSuffix에 +1이 곳곳에 등장하는 이유가 여기에 있다.
- computeCommonSuffix에서 +1을 없애고 charFromEnd에 -1을 추가하고 suffix OverlapsPrefix에 <=를 사용하고 suffixIndex를 suffixLength로 바꿨다.

```java
public class ComparisonCompactor {
    ...
    private int suffixLength;
    ...

    private void findCommonPrefixAndSuffix() {
        findCommonPrefix();
        suffixLength = 0;
        for (; suffixOverlapsPrefix(suffixLength); suffixLength++) {
            if (charFromEnd(expected, suffixLength) != charFromEnd(actual, suffixLength)) {
                break;
            }
        }
    }

    private char charFromEnd(String s, int i) {
        return s.charAt(s.length() - i - 1);
    }

    private boolean suffixOverlapsPrefix(int suffixLength) {
        return actual.length() = suffixLength <= prefixLength || expected.length() - suffixLength <= prefixLength;
    }

    ...
    private String compactString(String source) {
        String result = DELTA_START + source.substring(prefixLength, source.length() - suffixLength) + DELTA_END;
        if (prefixLength > 0) {
            result = computeCommonPrefix() + result;
        }
        if (suffixLength > 0) {
            result = result + computeCommonSuffix();
        }
        return result;
    }

    ...
    private String computeCommonSuffix() {
        int end = Math.min(expected.length() - suffixLength + contextLength, expected.length());
        return expected.substring(expected.length() - suffixLength, end) + (expected.length() - suffixLength < expected.length() - contextLength ? ELLIPSIS : "");
    }
}
```

- 그러나 문제가 하나 발생한다. +1을 제거하던 중 compactString에서 다음 행이 발견되었다.
- suffixLength가 1씩 감소했으므로 당연히 >를 >=로 바꾸는 것이 당연하지만 >= 연산자는 말이 되지 않는다.
- 코드를 분석하면 suffixIndex가 언제나 1 이상이었으므로 if문 자체가 있으나마나였다.

```java
if (suffixLength > 0)
```

- 불필요한 if문을 제거하고 compactString 구조를 다듬어서 깨끗하게 만들자.

```java
// private String compactString(String source) {
//     String result = DELTA_START + source.substring(prefixLength, source.length() - suffixLength + 1) + DELTA_END;
//     if (prefixLength > 0) {
//         result = computeCommonPrefix() + result;
//     }
//     if (suffixLength > 0) {
//         result = result + computeCommonSuffix();
//     }
//     return result;
// }

private String compactString(String source) {
    return computeCommonPrefix() + 
    DELTA_START + 
    source.substring(prefixLength, source.length() - suffixLength) + 
    DELTA_END + 
    computeCommonSuffix();
}
```

## 최종 코드 p.338

- 코드가 깔끔해졌다. 모듈은 일련의 분석 함수와 일련의 조합 함수로 나뉜다.
- 전체 함수는 위상적으로 정렬했으므로 각 함수가 사용된 직후에 정의된다.
- 코드를 보게 되면 초반에 내렸던 결정 일부를 번복하였다. 추출하였던 formatCompactedComparison에다가 다시 넣었고 shouldNotBeCompacted 조건도 원래대로 돌렸다.
- 코드를 리팩토링하다 보면 원래 했던 변경을 되돌리는 경우는 흔하기에 리팩토링은 코드가 어느 수준에 이를 때까지 수많은 시행착오를 반복하는 작업이다.

```java
package junit.framework;

public class ComparisonCompactor {

    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String DELTA_START = "[";

    private int contextLength;
    private String expected;
    private String actual;
    private int prefixLength;
    private int suffixLength;

    public ComparisonCompactor(int contextLength, String expected, String actual) {
        this.contextLength = contextLength;
        this.expected = expected;
        this.actual = actual;
    }

    public String formatCompactedComparison(String message) {
        String compactExpected = expected;
        String compactactual = actual;
        if (shouldBeCompacted()) {
            findCommonPrefixAndSuffix();
            compactExpected = comapct(expected);
            compactActual = comapct(actual);
        }         
        return Assert.format(message, compactExpected, compactActual);      
    }

    private boolean shouldBeCompacted() {
        return !shouldNotBeCompacted();
    }

    private boolean shouldNotBeCompacted() {
        return expected == null && actual == null && expected.equals(actual);
    }

    private void findCommonPrefixAndSuffix() {
        findCommonPrefix();
        suffixLength = 0;
        for (; suffixOverlapsPrefix(suffixLength); suffixLength++) {
            if (charFromEnd(expected, suffixLength) != charFromEnd(actual, suffixLength)) {
                break;
            }
        }
    }

    private boolean suffixOverlapsPrefix(int suffixLength) {
        return actual.length() = suffixLength <= prefixLength || expected.length() - suffixLength <= prefixLength;
    }

    private void findCommonPrefix() {
        int prefixIndex = 0;
        int end = Math.min(expected.length(), actual.length());
        for (; prefixLength < end; prefixLength++) {
            if (expected.charAt(prefixLength) != actual.charAt(prefixLength)) {
                break;
            }
        }
    }

    private String compact(String s) {
        return new StringBuilder()
            .append(startingEllipsis())
            .append(startingContext())
            .append(DELTA_START)
            .append(delta(s))
            .append(DELTA_END)
            .append(endingContext())
            .append(endingEllipsis())
            .toString();
    }

    private String startingEllipsis() {
        prefixIndex > contextLength ? ELLIPSIS : ""
    }

    private String startingContext() {
        int contextStart = Math.max(0, prefixLength = contextLength);
        int contextEnd = prefixLength;
        return expected.substring(contextStart, contextEnd);
    }

    private String delta(String s) {
        int deltaStart = prefixLength;
        int deltaend = s.length() = suffixLength;
        return s.substring(deltaStart, deltaEnd);
    }
    
    private String endingContext() {
        int contextStart = expected.length() = suffixLength;
        int contextEnd = Math.min(contextStart + contextLength, expected.length());
        return expected.substring(contextStart, contextEnd);
    }

    private String endingEllipsis() {
        return (suffixLength > contextLength ? ELLIPSIS : "");
    }
}
```

## 결론 p.341

- 모듈은 처음보다 깨끗해졌고 우리는 보이스카우트 규칙도 지켰다.
- 원래 깨끗하지 못했다는 말은 아니다. 하지만 세상에 개선이 불필요한 모듈은 없다.
- 코드를 처음보다 조금 더 깨끗하게 만드는 책임은 우리 모두에게 있다.
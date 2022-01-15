# 5장. 형식 맞추기 p.96

# 목차

1. [형식을 맞추는 목적](#형식을-맞추는-목적-p.96)
1. [적절한 행 길이를 유지하라](#적절한-행-길이를-유지하라-p.96)
   1. [신문 기사처럼 작성하라](#신문-기사처럼-작성하라)
   1. [개념은 빈 행으로 분리하라](#개념은-빈-행으로-분리하라)
   1. [세로 밀집도](#세로-밀집도)
   1. [수직 거리](#수직-거리)
      1. [변수 선언](#변수-선언)
      1. [인스턴스 변수](#인스턴스-변수)
      1. [종속 함수](#종속-함수)
      1. [개념적 유사성](#개념적-유사성)
   1. [세로 순서](#세로-순서)
1. [가로 형식 맞추기](#가로-형식-맞추기-p.107)
   1. [가로 공백과 밀집도](#가로-공백과-밀집도)
   1. [가로 정렬](#가로-정렬)
   1. [들여쓰기](#들여쓰기)
1. [팀 규칙](팀-규칙-p.113)
1. [밥 아저씨의 형식 규칙](밥-아저씨의-형식-규칙-p.114)

## 형식을 맞추는 목적 p.96

코드 형식은 **중요하다!** 오늘 구현한 기능이 다음 버전에서 바뀔 확률은 아주 높다. 오늘 구현한 코드의 가독성은 앞으로 바뀔 코드의 품질에 지대한 영향을 미친다. 오랜 시간이 지나 원래 코드의 흔적을 더 이상 찾아보기 힘들지라도 맨 처음 잡아놓은 구현 스타일과 가독성 수준은 유지보수 용이성과 확장성에 계속 영향을 미친다. 즉, 원래 코드는 사라질지라도 개발자의 스타일과 규율은 사라지지 않는다.

## 적절한 행 길이를 유지하라 p.96

p.97에 있는 이미지는 junit, ant, tomcat 등과 같은 프로젝트의 파일 길이를 조사한 표이다. 이 표를 보게 되면 평균적으로 500줄을 넘어가는 파일은 없으며 대다수 200줄 미만이다. 이 말은 즉 500줄이 넘지 않고 대부분 200줄 정도인 파일의 구성으로도 커다란 시스템을 구축할 수 있다는 사실이다. 해당 사실은 반드시 지킬 엄격한 규칙은 아니지만 바람직한 규칙으로 삼으면 좋다. 일반적으로 큰 파일보다는 작은 파일이 이해하기 쉽기 때문이다.

### 신문 기사처럼 작성하라

신문 기사는 최상단에 기사를 몇 마디로 요약하는 표제를 작성한다. 그 이후로 첫 문단은 전체 기사 내용을 요약하고 쭉 읽으면서 세세한 사실이 드러난다. 소스 파일도 신문 기사와 비슷하게 작성한다. 이름은 간단하면서도 설명이 가능하게 짓고, 아래로 내려갈수록 의도를 세세하게 묘사한다.

### 개념은 빈 행으로 분리하라

일련의 행 묶음은 완결된 생각 하나를 표현한다. 생각 사이에는 빈 행을 넣어 분리해야 마땅하다.

```java
package fitnesse.wikitext.widgets;

import java.util.regex.*;

public classBoldWidget extends ParanetWidget {
    public static final String REGEXP = "'''.+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?)'''", Pattern.MULTILINE + Pattern.DOTALL);

    public BoldWidget(PatrentWidget parent, String text) throws Exception {
        super(parent);
        Matcher match = pattern.matcher(text);
        match.find();
        addChildWidgets(match.group(1));
    }
}
```

```java
package fitnesse.wikitext.widgets;
import java.util.regex.*;
public classBoldWidget extends ParanetWidget {
    public static final String REGEXP = "'''.+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?)'''", Pattern.MULTILINE + Pattern.DOTALL);
    public BoldWidget(PatrentWidget parent, String text) throws Exception {
        super(parent);
        Matcher match = pattern.matcher(text);
        match.find();
        addChildWidgets(match.group(1));
    }
}
```

위에는 두 개의 소스가 있고 하나는 빈 행이 있고 하나는 빈 행이 없는 것이다. 아마 아래보다는 빈 행이 있는 위 소스가 더 가독성이 좋은 것을 알 수 있을 것이다.

### 세로 밀집도

세로 밀집도는 연관성을 의미한다. 서로 밀집한 코드 행은 세로로 가까이 놓여야 한다는 뜻이다.

```java
public class ReporterConfig {
    /**
    * 리포터 리스너의 클래스 이름
    */
    private String m_className;

    /**
    * 리포터 리스너의 속성
    */
    private List<property> m_properties = new ArrayList<Property>();
    public void addProperty(Property property) {
        m_properties.add(property);
    }
}
```

```java
public class ReporterConfig {
    private String m_className;
    private List<property> m_properties = new ArrayList<Property>();
    public void addProperty(Property property) {
        m_properties.add(property);
    }
}
```

아래와 같이 작성하면 불필요하게 머리나 눈을 움직일 필요가 거의 없기 때문에 읽기가 더 편하다.

### 수직 거리

함수 연관 관계와 동작 방식을 이해하려고 이 함수에서 저 함수로 오가며 소스 파일을 위아래로 뒤지는 등 뺑뺑이를 돌았으나 결국은 미로 같은 코드 때문에 혼란만 가중된 경험이 있을 것이다. 시스템이 무엇을 하는지 이해하고 싶은데, 이 조각 저 조각이 어디에 있는지 찾고 기억하느라 시간과 노력을 소모한다.
결론은 연관성이 깊은 두 개념이 멀리 떨어져 있으면 코드를 읽는 사람이 소스 파일과 클래스를 여기저기 뒤지게 되게 때문에 가까운 곳에 두어야 한다.

#### 변수 선언

변수는 사용하는 위치에 최대한 가까이 선언한다.

```java
private static void readPreferences() {
    InputStream is = null;
    try {
        is = new FileInputStream(getPreferencesFile());
        ...
    } catch (IOException e) {
        if (is != null) {
            is.close();
        }
    }
}

public int  countTestCases() {
    int count = 0;
    for (Test each: tests) { ... }
}

for(XmlTest test: m_suite.getTests()) {
    TestRunner tr = m_runnerFactory.newTestRunner(this, test);
    tr.addListener(m_textReporter);
    m_testRunners.add(tr);
    ...
}
```

#### 인스턴스 변수

인스턴스는 클래스 맨 처음에 선언하고 변수 간에 세로로 거리를 두지 않는다.

#### 종속 함수

한 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다. 또한 가능하다면 호출하는 함수를 호출되는 함수보다 먼저 배치한다.

```java
public class WikiPageResponder implements SecureResponder {
    public Response makeResponse(FitNesseContext context, Request request) throws Exception {
        String pageName = getPageNameOrDefault(request, "FrontPage");
        ...
    }
    private String getPageNameOrDefault(Request request, String defaultPageName) { ... }
}
```

#### 개념적 유사성

개념적인 친화도가 높을수록 코드를 가까이 배치한다. 친화도가 높은 이요인은 한 함수가 다른 함수를 호출해 생기는 종속성, 변수와 그 변수를 사용하는 함수  외에도 명명법과 기본 기능이 유사한 개념적으로 친화도가 높은 경우이다.

```java
public class Assert {
    static public void assertTrue(String message, boolean condition) {
        if (!condition)
            fail(message);
    }
    static public void assertTrue(boolean condition) {
        assertTrue(null, condition);
    }
    static public void assertFalse(String message, boolean condition) {
        assertTrue(message, !condition);
    }
    static public void assertFalse(boolean condition) {
        assertFalse(null, condition);
    }
}
```

예제를 보면 서로가 서로를 호출하는 관계이기에 종속성이 생긴다고 하지만 이것은 부차적인 요인이고 종속적인 관계가 없더라도 가까이 배치해야 할 함수이다.

### 세로 순서

일반적으로 함수 호출 종속성은 아래 방향으로 유지한다. 쉽게 말해 호출되는 함수를 호출하는 함수보다 나중에 배치한다.

## 가로 형식 맞추기 p.107

p.107에 있는 이미지를 보게 되면 20~60자 사이는 행의 총수는 40%에 달하고 10자 미만은 30% 정도이다. 80자 이후부터는 행 수가 급격하게 감소하며 프로그래머는 명백하게 짧은 행을 선호한다.

### 가로 공백과 밀집도

가로는 공백을 사용해 밀접한 개념과 느슨한 개념을 표현한다.

```java
private void measureLine(String line) {
    lineCount++;
    int lineSize = line.length();
    totalChars += lineSize;
    lineWidthHistogram.addLine(lineSize, lineCount);
    recordWidestLine(lineSize);
}
```

할당 연션자 앞 뒤로 공백을 주어 두 가지 주요 요소가 확실 나뉜다는 것이 분명해졌다. 반면, 함수 이름과 이어지는 괄호 사이에는 공백을 넣지 않았다. 서로 밀접하기 때문이고 여기에 공백을 넣게 되면 서로 별개로 보일 것이다. 함수 내에 인자는 쉼표 뒤에 공백으로 구분했고 두 개가 별개라는 것을 강조된다.

### 가로 정렬

특정 구조를 강조하기 위해서 아래와 같이 정렬해서 선언하게 된다.

```java
public class FitNesseExpediter implements ResponseSender {
    private   Socket       socket;
    private   InputStream  input;
    protected long         requestParsingTimeLimit;

    public FitNesseExpediter(Socket FitNesseContext context) throws Exception {
        this.conext =             conext;
        requestParsingTimeLimit = 10000;
    }
}
```

구조를 강조하려다보니 진짜 의도는 가려지게 된다. 위 예제에는 선언부는 변수 유형보다 변수 이름부터 읽게 되고 할당문을 보게 되면 할당 연산자가 아닌 피연산자를 먼저 눈이 가게 된다. 그렇기에 위와 가티 정렬하는 것이 좋지 않다.

만약 정렬이 필요할 정도로 목록이 길다면 문제는 목록 길이지 정렬 부족이 아니므로 선언부가 길어진다면 클래스를 쪼갤 수 있는지부터 확인한다.

### 들여쓰기

소스 파일은 윤곽도와 계층이 비슷하다. 계층에서 각 수준은 이름을 선언하는 범위이자 선언문과 실행문을 해석하는 범위이다.

프로그래머는 들여쓰기 체계에 크게 의존한다. 왼쪽으로 코드를 맞춰 코드가 속하는 범위를 시각적으로 표현한다.

```java
public static void main(String[] args) { 
    for (int i = 0; i < 5; i++) {
        for (int j = 0; i < 10; j++) {
            for (int k = 0; k < 15; k++) {

            }
        }
        for (int j = 0; j < 10; j++) {

        }
        for (int l = 0; l < 15; l++) {

        }
    }
}
```

```java
public static void main(String[] args) { 
for (int i = 0; i < 5; i++) {
for (int j = 0; i < 10; j++) {
for (int k = 0; k < 15; k++) {

}
}
for (int j = 0; j < 10; j++) {

}
for (int l = 0; l < 15; l++) {

}
}
}
```

위 소스보다 아래 소스가 가독성이 좋다는 것을 알 수 있을 것이다.

## 팀 규칙 p.113

프로그래머라면 각자 선호하는 규칙이 있다. 하지만 팀에 속한다면 자신이 선호해야 할 규칙은 바로 팀 규칙이다.

팀은 한 가지 규칙에 합의해야 한다. 그리고 모든 팀원은 그 규칙을 따라야 한다.

## 밥 아저씨의 형식 규칙 p.114
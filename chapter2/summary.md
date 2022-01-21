# 2장. 의미 있는 이름 p.21

## 목차
1. [의도를 분명히 밝혀라](#의도를-분명히-밝혀라-p22)
2. [그릇된 정보를 피하라](#그릇된-정보를-피하라-p24)
3. [의미 있게 구분하라](#의미-있게-구분하라-p25)
4. [발음하기 쉬운 이름을 사용하라](#발음하기-쉬운-이름을-사용하라-p27)
5. [검색하기 쉬운 이름을 사용하라](#검색하기-쉬운-이름을-사용하라-p28)
6. [인코딩을 피하라](#인코딩을-피하라-p29)
   1. [헝가리식 표기법](#헝가리식-표기법)
   2. [멤버 변수 접두어](#멤버-변수-접두어)
   3. [인터페이스 클래스와 구현 클래스](#인터페이스-클래스와-구현-클래스)
10. [자신의 기억력을 자랑하지 마라](#자신의-기억력을-자랑하지-마라-p31)
11. [클래스 이름](#클래스-이름-p32)
12. [메서드 이름](#메서드-이름-p32)
13. [기발한 이름은 피하라](#기발한-이름은-피하라-p32)
14. [한 개념에 한 단어를 사용하라](#한-개념에-한-단어를-사용하라-p33)
15. [말장난을 하지 마라](#말장난을-하지-마라-p34)
16. [해법 영역 용어를 사용하라](#해법-영역-용어를-사용하라-p34)
17. [문제 영역 용어를 사용하라](#문제-영역-용어를-사용하라-p34)
18. [의미 있는 맥락을 추가하라](#의미-있는-맥락을-추가하라-p35)
19. [불필요한 맥락을 없애라](#불필요한-맥락을-없애라-p37)


## 의도를 분명히 밝혀라 p22
- 변수의 **존재 이유, 기능, 사용 방법**이 드러나도록 한다.
- 의미를 함축하거나 코드를 읽는 사람이 사전 지식을 가지고 있다고 가정하지 않는다.
- 명료하게 설정하여 목적을 쉽게 이해할 수 있도록 한다.
```java
// bad
int d;
ArrayList<int[]> list = new ArrayList<int[]>();
public List<int[]> getThem() { ... }
```

```java
// good
int fileAgeInDays;
ArrayList<int[]> flaggedCells = new ArrayList<int[]>();
public List<int[]> getFlaggedCells() { ... }
```


## 그릇된 정보를 피하라 p24
- 중의적으로 해석될 여지가 있는 이름 지양한다.
- 프로그래머에게 특수한 의미를 지니는 단어는 변수명에 붙이지 않는다.
- 흡사한 이름을 사용하지 않도록 주의한다.

## 의미 있게 구분하라 p25
- `a1, a2, ...` 와 같이 숫자로 구분하는 것은 적절하지 않다.
- 불용어(noise word)를 추가하는 경우도 적절하지 않다.
- 예시
```java
// 두 변수의 차이가 분명하지 않다.
message vs theMessage;
name vs nameString

// 각 메서드의 역할을 정확히 알기 어렵다.
getActiveAccoutn() vs getActiveAccounts() vs getActiveAccountInfo()

// 각 클래스의 역할 차이를 알기 어렵다.
class Customer {}
class CustomerObject {}
```

## 발음하기 쉬운 이름을 사용하라 p27
- 발음 하기 어려운 이름은 토론하기도 어렵다.
```java
// bad
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
    /* ... */
}

// good
class Customer {
    private Date generationTimestamp;
    private Date modificationTimestamp;
    private final String recordId = "102";
    /* ... */
}
```

## 검색하기 쉬운 이름을 사용하라 p28
- 변수 이름의 길이는 범위 크기에 비례해서 길어진다.

## 인코딩을 피하라 p29
- 인코딩한 이름은 거의가 발음하기 어려우며 오타가 생기기도 쉽다.

### 헝가리식 표기법
- 변수명에 변수의 타입(int, long, String 등)을 적지 않는다.
### 멤버 변수 접두어
- private와 같은 접두어를 변수 앞에 붙이지 않는다.

### 인터페이스 클래스와 구현 클래스
- 때로는 인코딩이 필요하다.
- 인터페이스 클래스 이름과 구현 클래스 이름 중 하나를 인코딩해야 한다면 구현 클래스 이름을 인코딩한다.

## 자신의 기억력을 자랑하지 마라 p31
- 코드를 읽는 사람이 한 번 더 생각해 변환해야 한다면 그 변수 이름은 적절하지 않다.
- 똑똑한 프로그래머와 전문가 프로그래머를 나누는 차이점은 **명료함**이다.

## 클래스 이름 p32
- 명사 혹은 명사구를 사용한다.
- `Customer, Account, Address`
- `Manager, Processor, Data, Info` 와 같은 이름은 지양한다.
- 동사는 사용하지 않는다.

## 메서드 이름 p32
- 동사 혹은 동사구를 사용한다.
- `deletePage, saveAccount, postPayment` 등
- 접근자, 변경자, 조건자는 javabean 표준에 따라 get, set, is를 붙인다.
- 생성자 사용을 제한하려면 해당 생성자를 private로 선언한다.
```java
// bad
Complex fulcrumPoint = new Complex(23.0);  

// good
Complex fulcrumPoint = Complex.FromRealNumber(23.0);
```

## 기발한 이름은 피하라 p32
- 특정 문화에서만 사용되는 이름 혹은 재미 있는 이름보다 **명료한 이름**을 사용한다.

## 한 개념에 한 단어를 사용하라 p33
- 똑같은 메서드를 클래스마다 다르게 사용하지 않는다.
- `fetch, retrieve, get` (X)

## 말장난을 하지 마라 p34
- 한 단어를 두 가지 이상의 목적으로 사용하지 않는다.
```java
// add보다 append 혹은 insert로 바꾸는게 옳다.
public String add(String firstName, String lastName)
public int add(int currentValue, int nextValue)
```

## 해법 영역 용어를 사용하라 p34
- 코드를 읽는 사람도 프로그래머다.
- `JobQueue, AccountVisitor(Visitor pattern)`등을 사용하지 않을 이유는 없다. 
- 전산용어, 알고리즘 이름, 패턴 이름, 수학 용어 등은 사용해도 괜찮다.

## 문제 영역 용어를 사용하라 p34
- 적절한 `프로그래머 용어`가 없다면 문제 영역에서 이름을 가져온다.
- 문제 영역과 관련이 깊은 용어의 경우 문제 영역 용어를 사용한다. 

## 의미 있는 맥락을 추가하라 p35
- 클래스, 함수, name space 등으로 감싸서 맥락을 부여한다.
- 그래도 불분명하다면 접두어를 붙인다.
```java
// bad
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluralModifier;
    if (count == 0) {  
        number = "no";  
        verb = "are";  
        pluralModifier = "s";  
    }  else if (count == 1) {
        number = "1";  
        verb = "is";  
        pluralModifier = "";  
    }  else {
        number = Integer.toString(count);  
        verb = "are";  
        pluralModifier = "s";  
    }
    String guessMessage = String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );

    print(guessMessage);
}
```
```java
// good
public class GuessStatisticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;

    public String make(char candidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );
    }

    private void createPluralDependentMessageParts(int count) {
        if (count == 0) {
            thereAreNoLetters();
        } else if (count == 1) {
            thereIsOneLetter();
        } else {
            thereAreManyLetters(count);
        }
    }

    private void thereAreManyLetters(int count) {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }

    private void thereIsOneLetter() {
        number = "1";
        verb = "is";
        pluralModifier = "";
    }

    private void thereAreNoLetters() {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }
}
```

## 불필요한 맥락을 없애라 p37
- `Gas Station Delux` 이라는 어플리케이션을 작성한다고 해서 클래스 이름의 앞에 `GSD`를 붙이는 것은 좋지 않다.
- `G`를 입력하고 자동완성을 누를 경우 모든 클래스가 나타나는 등 효율적이지 않다.
- `GSDAccountAddress` 대신 `Address`라고만 해도 충분하다.
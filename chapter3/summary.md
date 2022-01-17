# 3장. 함수 p.39

## 목차
1. [작게 만들어라!](#작게-만들어라!-p42)
    1. [블록과 들여쓰기](#블록과-들여쓰기)
1. [한 가지만 해라!](#한-가지만-해라!-p44)
    1. [함수 내 섹션](#함수-내-섹션)
1. [함수 당 추상화 수준은 하나로!](#함수-당-추상화-수준은-하나로!-p45)
    1. [위에서 아래로 코드 읽기 : 내려가기 규칙](#위에서-아래로-코드-읽기:내려가기-규칙)
1. [Switch문](#Switch문-p47)
1. [서술적인 이름을 사용하라!](#서술적인-이름을-사용하라!-p49)
1. [함수 인수](#함수-인수-p50)
    1. [많이 쓰는 단항 형식](#많이-쓰는-단항-형식)
    1. [플래그 인수](#플래그-인수)
    1. [이항 함수](#이항-함수)
    1. [삼항 함수](#삼항-함수)
    1. [인수 객체](#인수-객체)
    1. [인수 목록](#인수-목록)
    1. [동사와 키워드](#동사와-키워드)
1. [부수 효과를 일으키지 마라!](#부수-효과를-일으키지-마라!-p54)
    1. [출력 인수](#출력-인수)
1. [명령과 조회를 분리하라!](#명령과-조회를-분리하라!-p56)
1. [오류코드보다 예외를 사용하라!](#오류코드보다-예외를-사용하라!-p57)
    1. [Try/Catch 블록 뽑아내기](#Try/Catch-블록-뽑아내기)
    1. [오류 처리도 한 가지 작업이다](#오류-처리도-한-가지-작업이다)
    1. [Error.java의 의존성 자석](#Error.java의-의존성-자석)
1. [반복하지마라!](#반복하지마라!-p60)
1. [구조적 프로그래밍](#구조적-프로그래밍-p61)
1. [함수를 어떻게 짜죠?](#함수를-어떻게-짜죠?-p61)

<br>

## 작게 만들어라! p42
- 함수를 만들 때 최대한 **작게** 만들어라.
```java
public static String renderPageWithSetupsAndTeardowns(PageData pageData, boolean isSuite) throws Exception {
        boolean isTestPage = pageData.hasAttribute("Test");
        if (isTestPage) {
        WikiPage testPage = pageData.getWikiPage();
        StringBuffer newPageContent = new StringBuffer();
        includeSetupPages(testPage, newPageContent, isSuite);
        newPageContent.append(pageData.getContent());
        includeTeardownPages(testPage, newPageContent, isSuite);
        pageData.setContent(newPageContent.toString());
        }
        return pageData.getHtml();
}
```
- 위 코드도 길다.
- 한 함수당 3~5줄이 적당하다.
```java
public static String renderPageWithSetupsAndTeardowns(PageData pageData, boolean isSuite) throws Exception { 
    if (isTestPage(pageData)) 
        includeSetupAndTeardownPages(pageData, isSuite); 
    return pageData.getHtml();
}
```

### 블록과 들여쓰기
- if/else, while 등에 들어가는 블록은 한 줄이 좋다.
- 각 함수 별 들여쓰기 수준이 2단을 넘지 않도록 한다.

## 한 가지만 해라! p44
> **함수는 한 가지만을 해야 하고, 그 한 가지를 잘 해야 한다.**

### 함수 내 섹션
- 한 가지 작업만 하는 함수는 자연스럽게 섹션으로 나누기 어렵다.

## 함수 당 추상화 수준은 하나로! p45
- 지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한 가지 작업만 하는 것이다.

### 위에서 아래로 코드 읽기 : 내려가기 규칙
- 코드는 위에서 아래로 읽혀야 좋다.
- 함수 추상화 부분이 한 번에 한단계씩 낮아지는 것이 가장 이상적이다.(내려가기 규칙)

## Switch문 p47
- switch문은 작게 만들기 어렵다.
- 다형성을 이용하여 각 switch문을 저차원 클래스에 숨기고 반복하지 않는 방법이 있다.
- 하지만 불가피하게 switch문을 사용해야할 때도 있다.
```java
// bad
public Money calculatePay(Employee e) throws InvalidEmployeeType {
    switch (e.type) { 
        case COMMISSIONED:
            return calculateCommissionedPay(e); 
        case HOURLY:
            return calculateHourlyPay(e); 
        case SALARIED:
            return calculateSalariedPay(e); 
        default:
            throw new InvalidEmployeeType(e.type); 
    }
}
```
```java
// good
public abstract class Employee {
    public abstract boolean isPayday();
    public abstract Money calculatePay();
    public abstract void deliverPay(Money pay);
}
-----------------
public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType; 
}
-----------------
public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
        switch (r.type) {
            case COMMISSIONED:
                return new CommissionedEmployee(r) ;
            case HOURLY:
                return new HourlyEmployee(r);
            case SALARIED:
                return new SalariedEmploye(r);
            default:
                throw new InvalidEmployeeType(r.type);
        } 
    }
}
```

## 서술적인 이름을 사용하라! p49
- 작은 함수는 그 기능이 명확하므로 이름 붙이기가 더 쉽다.
- 일관적이고 서술적인 이름을 사용하면 코드를 순차적으로 이해하고 개선하기 쉬워진다.
- `includeSetupAndTeardownPages, includeSetupPages, includeSuiteSetupPage` 등이 좋은 예이다.

## 함수 인수 p50
- 가장 이상적인 인수 개수는 0개(무항)
- 인수를 써야한다면 1~2개 까지가 적당하다.
- 함수 인수가 늘어날수록 코드 이해가 어려워진다.
- 인수가 2~3개 필요하다면 일부를 독자적인 클래스 변수로 선언하는 것이 좋다.

### 많이 쓰는 단항 형식
- 인수에 질문들 던지는 경우 : `boolean fileExists(“MyFile”);`
- 인수를 뭔가로 변환해 결과를 반환하는 경우 : `InputStream fileOpen(“MyFile”);`
- 이벤트 함수일 경우 : 이벤트라는 사실이 코드에 명확히 드러나야 한다.

### 플래그 인수
- 플래그 인수는 추하다.
- `boolean` 값을 넘기는 것 자체가 함수가 한 번에 여러 가지를 처리한다는 뜻이다.

### 이항 함수
- 단항 함수보다 이해하기 어렵다.
- Point class의 경우에는 이항 함수가 적절하다.
- 무조건 나쁘다는 것은 아니지만 가능하면 단항 함수로 바꾸도록 한다.

### 삼항 함수
- 이항 함수보다 이해하기 훨씬 어렵다.
- 삼항 함수를 만들 때는 신중히 고려한다.

### 인수 객체
- 인수가 2~3개 필요하다면 일부를 독자적인 클래스 변수로 선언할 가능성을 짚어 본다.

### 인수 목록
- 때로는 인수 개수가 가변적인 함수도 필요하다. (`String.format()`)
- `String.format` 선언부를 보면 List형 인수이기 때문에 이항 함수라고 할 수 있다.

### 동사와 키워드
- 함수의 의도나 인수의 순서와 의도를 제대로 표현하려면 좋은 함수 이름이 필요하다.
- 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다. (`writeField(name)`)
- 함수 이름에 **키워드**를 추가하면 인수 순서를 기억할 필요가 없다.

## 부수 효과를 일으키지 마라! p54
- 부수 효과는 거짓말이다.
- 아래 코드에서 `Session.initialize();`는 함수 이름만 보고 알 수 없다.
```java
public class UserValidatot {
    private Cryptographer cryptographer;

    public boolean checkPassword(String userName, String password) {
        User user = UserGateway.findByName(userName);
        if(user != User.NULL) {
            String codedPhrase = user.getPhraseEncodedByPassword();
            String phrase = cryptographer.decrypt(codedPhrase, password);
            if("Valid Password".equals(phrase)) {
                Session.initialize();
                return true;
            }
        }
        return false;
    }
}
```

### 출력 인수
- 일반적으로 출력 인수는 피해야 한다.
- 함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택한다.

## 명령과 조회를 분리하라! p56
- 함수는 수행하거나 답하거나 둘 중 하나만 해야한다.
- 둘 다 수행해서는 안된다.

## 오류코드보다 예외를 사용하라! p57


### Try/Catch 블록 뽑아내기
- 오류 코드 대신 try/catch를 사용하면 코드가 깔끔해진다. 

```java
// 정상 작동과 오류 처리 동작을 뒤섞는 추한 구조이다.
// if/else와 마찬가지로 블록을 별도 함수로 뽑아내는 편이 좋다.
if (deletePage(page) == E_OK) {
    if (registry.deleteReference(page.name) == E_OK) {
        if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
            logger.log("page deleted");
        } else {
            logger.log("configKey not deleted");
        }
    } else {
        logger.log("deleteReference from registry failed"); 
    } 
} else {
    logger.log("delete failed"); return E_ERROR;
}
```
```java
public void delete(Page page) {
    try {
        deletePageAndAllReferences(page);
      } catch (Exception e) {
          logError(e);
      }
}

private void deletePageAndAllReferences(Page page) throws Exception { 
    deletePage(page);
    registry.deleteReference(page.name); 
    configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) { 
    logger.log(e.getMessage());
}
```

### 오류 처리도 한 가지 작업이다
- 오류 처리도 '한 가지' 작업에 속하므로 오류를 처리하는 함수는 오류만 처리해야 마땅하다.

### Error.java의 의존성 자석
- 오류 코드를 반환한다는 말은 어디선가 오류 코드를 정의한다는 뜻이다.
- 새 오류 코드를 추가하거나 변경할 때 비용이 많이 든다.
- 그래서 새 오류 코드를 추가하는 대신 예외를 사용하는 것이 좋다.
```java
public enum Error { 
    OK,
    INVALID,
    NO_SUCH,
    LOCKED,
    OUT_OF_RESOURCES,     
    WAITING_FOR_EVENT;
}
```
## 반복하지마라! p60
- 중복은 소프트웨어에서 모든 악의 근원이다.
- 중복을 없애도록 지속적으로 노력한다.

## 구조적 프로그래밍 p61
- 함수는 `return` 문이 하나여야 한다.
- 루프 안에서 `break, continue`를 사용해서는 안된다.
- `goto`는 **절대** 사용하지 않는다. `goto`는 큰 함수에서만 의미 있다.
- 함수를 작게 만든다면 간혹 `return, break, continue`를 여러번 사용해도 괜찮다.

## 함수를 어떻게 짜죠? p61
- 처음에는 길고 복잡하다.
- 들여쓰기 단계도 많고 중복된 루프도 많으며 인수 목록도 아주 길다.
- 이 코드들을 빠짐없이 테스트하는 단위 테스트 케이드도 만들고, 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거한다.
- 처음부터 탁 짜지지는 않는다.

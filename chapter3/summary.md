# 3장. 함수

## 작게 만든다.
- 100줄을 넘어서는 안 되고, 20줄도 긴 편이다.
- if/else, while 등에 들어가는 블록은 한 줄이 좋다.
- 각 함수 별 들여쓰기 수준이 2단을 넘지 않도록 한다.

## 함수는 한 가지 일을 한다.
 - **함수는 한 가지만을 해야 하고, 그 한 가지를 잘 해야 한다.**
 - 함수가 한 가지 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 한다.
 - 부수적인 일을 하지 않도록 한다.
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

## 위에서 아래로 읽기
 - 코드는 위에서 아래로 이야기처럼 읽혀야 좋다.
 - 함수 추상화 수준이 한 번에 한 단계씩 낮아지는 것이 이상적이다.

## 서술적인 이름을 사용한다.
 - 함수가 작고 단순할수록 이름을 고르기도 쉽다.
 - 일관성 있는 서술형 이름을 사용하면 코드를 이해하기도 쉬워진다.
 - `includeSetupAndTeardownPages, includeSetupPages, includeSuiteSetupPage` 등이 좋은 예이다.

## 인수는 최대한 적게 사용한다.
 - 가장 이상적인 인수 개수는 0개(무항)
 - 인수를 써야한다면 1~2개 까지가 적당하다.
 - 함수 인수가 늘어날수록 코드 이해가 어려워진다.
 - 인수가 2~3개 필요하다면 일부를 독자적인 클래스 변수로 선언하는 것이 좋다.

## 명령 / 조회 분리
 - 함수는 수행하거나 답하거나 둘 중 하나만 해야한다.
 - 둘 다 수행해서는 안된다.

## 오류코드보다 예외를 사용
 - 오류 처리도 한가지 작업이므로 예외를 사용하는 것이 더 좋다.
 - 오류 코드 대신 예외를 사용하면 코드가 깔끔해진다.
 
## 반복하지 마라.
 - 중복은 소프트웨어에서 모든 악의 근원

## 구조적 프로그래밍
 - 함수는 `return` 문이 하나여야 한다.
 - 루프 안에서 `break, continue`를 사용해서는 안된다.
 - `goto`는 **절대** 사용하지 않는다. `goto`는 큰 함수에서만 의미 있다.
 - 함수를 작게 만든다면 간혹 `return, break, continue`를 여러번 사용해도 괜찮다.

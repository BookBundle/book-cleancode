# 11장. 시스템 p.194

1. [클래스 체계](#도시를-세운다면?-p.194)
1. [시스템 제작과 시스템 사용을 분리하라](#시스템-제작과-시스템-사용을-분리하라-p.194)
   1. [Main 분리](#Main-분리)
   1. [팩토리](#팩토리)
   1. [의존성 주입](#의존성-주입)
   1. [진정한 의존성 주입](#진정한-의존성-주입)
1. [확장](#확장)
   1. [횡단(cross-cutting) 관심사](#횡단(cross-cutting)-관심사)
1. [자바 프록시](#자바-프록시)
1. [순수 자바 AOP 프레임워크](#순수-자바-AOP-프레임워크)
1. [AspectJ 관점](#AspectJ-관점)
1. [테스트 주도 시스템 아키텍처 구축](#테스트-주도-시스템-아키텍처-구축)
1. [의사 결정을 최적화하라](#의사-결정을-최적화하라)
1. [명백한 가치가 있을 때 표준을 현명하게 사용하라](#명백한-가치가-있을-때-표준을-현명하게-사용하라)
1. [시스템은 도메인 특화 언어가 필요하다](#시스템은-도메인-특화-언어가-필요하다)
1. [결론](#결론)

## 도시를 세운다면? p.194

- 도시를 구상한다면?
   - 한 사람의 힘으로는 불가능하다.
   - 수도 관리팀, 전력 관리팀, 교통 관리팀, 치안 관리팀, 건축물 관리팀 등 각 분야를 관리하는 팀이 존재
   - 도시에는 큰 그림을 그리는 사람과 작은 사항에 집중하는 사람 다양하게 있다.
- 도시가 돌아가는 이유는 적절한 **추상화와 모듈화**
   - 큰 그림을 이해하지 못할지라도 개인과 개인이 관리하는 '구성요소'는 효율적으로 돌아간다.
- 소프트웨어 팀도 도시처럼 구성한다.
   - 깨끗한 코드를 구현하면 낮은 추상화 수준에서 관심사를 분리하기 쉬워진다.
   - 이 장에서는 높은 추상화 수준, **시스템** 수준에서도 깨끗함을 유지하는 방법을 살펴본다.

## 시스템 제작과 시스템 사용을 분리하라 p.194

- 소프트웨어 시스템은 (애플리케이션 객체를 제작하고 의존성을 서로 '연결'하는) 준비 과정과 (준비 과정 이후에 이어지는) 런타임 로직을 분리해야 한다.
- 시작 단계는 모든 애플리케이션이 풀어야 할 **관심사**다.

```java
public Service getService() {
    if (service == null)
        service = new MyServiceImpl(...);
    return service;
}
```

- 실제 필요할 때까지 객체를 생성하지 않음으로 불필요한 부하가 걸리지 않고 애플리케이션 시작 시각이 그만큼 빨라진다.
- 하지만, getService 메소드가 MyServiceImpl과 생성자 인자에 명시적으로 의존한다.
- 그리하여 런타임 로직에서 MyServiceImpl 객체를 전혀 사용하지 않더라도 의존성을 해결하지 않으면 컴파일이 안 된다.
- 테스트 일 경우에도 service가 null인 경로와 null이 아닌 경로 모든 실행 경로도 테스트해야 한다.
- 이 말은 책임이 둘이라는 말이고 메소드가 작업을 두 가지 이상 수행한다는 의미이기에 SRP을 깬다.
- **체계적이고 탄탄한 시스템을 만들고 싶다면 설정 논리는 일반 실행 논리와 분리해야 모듈성이 높아진다.**

### Main 분리

- 시스템 생성과 시스템 사용을 분리하는 한 가지 방법
- 생성과 관련된 코드는 모두 main이나 main이 호출하는 모듈로 옮기고, 나머지 시스템은 모든 객체가 생성되었고 모든 의존성이 연결되었다고 가정한다.

<img width="519" alt="main" src="https://user-images.githubusercontent.com/50051656/151554403-4c4d0764-82a4-4a9c-ba39-696d3f38f921.png">

- main 함수에서 시스템에 필요한 객체를 생성한 후 애플리케이션에 넘긴다.
- 애플리케이션은 단지 객체가 생성되는 과정을 알지 못하고 모든 객체가 적절히 생성되었다고 가정한다.

### 팩토리

- 때로는 객체가 생성되는 시점을 애플리케이션이 결정할 필요도 생긴다.

![factory](https://user-images.githubusercontent.com/50051656/151554389-c456358c-4952-4af2-8d16-167a2a142251.png)

- 모든 의존성이 main에서 OrderProcessing 애플리케이션으로 향한다.
- OrderProcessing 애플리케이션은 LineItem이 생성되는 구체적인 방법을 모른다.

### 의존성 주입

- 사용과 제작을 분리하는 강력한 메커니즘 하나가 **의존성 주입**이다.
- 의존성 주입은 제어 역전 기법(IoC)을 의존성 관리에 적용한 메커니즘이다.
- 새로운 객체는 넘겨받은 책임만 맡음으로 SRP를 만족시킨다.

```java
MyService myService = (MyService)(jndiContext.lookup("NameOfMyService"));
```

- 호출하는 객체는 (반환되는 객체가 적절한 인터페이스를 구현하는 한) 실제로 반환되는 객체의 유형을 제어하지 않는다.
- 대신 호출하는 객체는 의존성을 능동적으로 해결한다.

### 진정한 의존성 주입

- 클래스가 의존성을 해결하려고 시도하지 않고 클래스는 완전히 수동적이다.
- 의존성을 주입하는 방법으로 설정자(setter) 메소드나 생성자 인자를 제공한다.
- DI 컨테이너는 (대개 요청이 들어올 때마다) 필요한 객체의 인스턴스를 만든 후 생성자 인자나 설정자 메소드를 사용해 의존성을 설정한다.

## 확장

- 군락은 마을로, 마을은 도시로 성장한다. 처음에는 좁거나 없던 길이 포장되며 점차 넓어지게 된다.
- 위와 같은 방식이 아닌 다른 방식은 어렵다. 성장할지 모르는 마을에 누가 6차선을 뚫는 데 들어가는 비용을 정당화할 수 있는가?
- 우리는 오늘 주어진 사용자 스토리에 맞춰 시스템을 조정하고 구현하고 내일은 새로 스토리에 맞게 시스템을 조정하고 확장한다.
- 이것이 반복적이고 점진적인 애자일 방식의 핵심이다.
- **TDD, 리팩토링으로 얻어지는 깨끗한 코드는 코드 수준에서 시스템을 조정하고 확장하기 쉽게 만든다.**
- 소프트웨어 시스템은 물리적인 시스템과 다르다. 관심사를 적절히 분리해 관리한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있다.

```java
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public interface BankLocal extends java.ejb.EJBLocalObject {
    String getStreetAddr1() throws EJBException;
    String getStreetAddr2() throws EJBException;
    String getCity() throws EJBException;
    String getState() throws EJBException;
    String getZipCode() throws EJBException;
    void setStreetAddr1(String street1) throws EJBException;
    void setStreetAddr2(String street2) throws EJBException;
    void setCity(String city) throws EJBException;
    void setState(String state) throws EJBException;
    void setZipCode(String zip) throws EJBException;
    Collection getAccounts() throws EJBException;
    void setAccounts(Collection accounts) throws EJBException;
    void addAccount(AccountDTO accountDTO) throws EJBException;
}
```

- Bank EJB용 EJB2 지역 인터페이스
- 열거하는 속성은 Bank 주소, 은행이 소유하는 계좌이다.

```java
public abstract class Bank implements javax.ejb.EntityBean {
    // 비즈니스 논리...
    public abstract String getStreetAddr1();
    public abstract String getStreetAddr2();
    public abstract String getCity();
    public abstract String getState();
    public abstract String getZipCode();
    public abstract void setStreetAddr1(String street1);
    public abstract void setStreetAddr2(String street2);
    public abstract void setCity(String city);
    public abstract void setState(String state);
    public abstract void setZipCode(String zip);
    public abstract Collection getAccounts();
    public abstract void setAccounts(Collection accounts);
    
    public void addAccount(AccountDTO accountDTO) {
        InitialContext context = new InitialContext();
        AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
        AccountLocal account = accountHome.create(accountDTO);
        Collection accounts = getAccounts();
        accounts.add(account);
    }
    
    // EJB 컨테이너 논리
    public abstract void setId(Integer id);
    public abstract Integer getId();
    public Integer ejbCreate(Integer id) { ... }
    public void ejbPostCreate(Integer id) { ... }
    
    // 나머지도 구현해야 하지만 일반적으로 비어있다.
    public void setEntityContext(EntityContext ctx) {}
    public void unsetEntityContext() {}
    public void ejbActivate() {}
    public void ejbPassivate() {}
    public void ejbLoad() {}
    public void ejbStore() {}
    public void ejbRemove() {}
}
```

- EJB2 지역 인터페이스를 구현한 빈이다.
- 비즈니스 로직이 EJB2 컨테이너 애플리케이션 '컨테이너'와 강하게 결합된다.
- 클래스를 생성할 때는 컨테이너에서 파생해야 하며 컨테이너가 요구하는 다양한 생명주기 메서드도 제공해야 한다.
- 비즈니스 논리가 덩치 큰 컨테이너와 밀접하게 결합한 탓에 독자적인 단위 테스트가 어렵다.
- 결국 객체 지향 프로그래밍이라는 개념이 흔들리며 빈은 다른 빈을 상속받지 못한다.
- 일반적으로 EJB2 빈은 DTO를 정의한다. DTO에는 메소드가 없으며 사실상 구조체이다.
- 즉, 동일한 정보를 저장하는 자료 유형이 두 개이고 한 객체에서 다른 객체로 자료를 복사하는 반복적인 규격 코드가 필요하다.

### 횡단(cross-cutting) 관심사

- EJB2 아키텍처는 일부 영역에서 관심사를 거의 완벽하게 분리하다.
- 영속성 같은 관심사는 애플리케이션의 자연스러운 객체 경계를 넘나드는 경향이 있기에 모든 객체가 전반적으로 동일한 방식으로 이용하게 만들어야 한다.
- EJB 아키텍처가 영속성, 보안, 트랜잭션을 처리하는 방식은 관점 지향 프로그래밍(AOP)을 예견했다고 보인다.
- AOP는 횡단 관심사에 대처해 모듈성을 확보하는 일반적인 방법론이다.

## 자바 프록시

- 단순한 상황에 적합하며 개별 객체나 클래스에서 메소드 호출을 감싸는 경우가 좋은 예다.
- JDK에서 제공하는 동적 프록시는 인터페이스만 지원하고 클래스 프록시를 사용하려면 CGLIB, ASM, javassist 등과 같은 바이트 코드 처리 라이브러리가 필요하다.

```java
// Bank.java (suppressing package names...)
import java.utils.*;

// The abstraction of a bank.
public interface Bank {
    Collection<Account> getAccounts();
    void setAccounts(Collection<Account> accounts);
}

// BankImpl.java
import java.utils.*;

// The “Plain Old Java Object” (POJO) implementing the abstraction.
public class BankImpl implements Bank {
    private List<Account> accounts;

    public Collection<Account> getAccounts() {
        return accounts;
    }
    
    public void setAccounts(Collection<Account> accounts) {
        this.accounts = new ArrayList<Account>();
        for (Account account: accounts) {
            this.accounts.add(account);
        }
    }
}

// “InvocationHandler” required by the proxy API.
public class BankProxyHandler implements InvocationHandler {
    private Bank bank;
    
    public BankHandler (Bank bank) {
        this.bank = bank;
    }
    
    // Method defined in InvocationHandler
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        if (methodName.equals("getAccounts")) {
            bank.setAccounts(getAccountsFromDatabase());
            
            return bank.getAccounts();
        } else if (methodName.equals("setAccounts")) {
            bank.setAccounts((Collection<Account>) args[0]);
            setAccountsToDatabase(bank.getAccounts());
            
            return null;
        } else {
            ...
        }
    }
    
    // Lots of details here:
    protected Collection<Account> getAccountsFromDatabase() { ... }
    protected void setAccountsToDatabase(Collection<Account> accounts) { ... }
}

// Somewhere else...
Bank bank = (Bank) Proxy.newProxyInstance(
    Bank.class.getClassLoader(),
    new Class[] { Bank.class },
    new BankProxyHandler(new BankImpl())
);
```

- Bank 애플리케이션에서 JDK 프록시를 사용해 영속성을 지원하는 예제
- 프록시로 감쌀 인터페이스 Bank와 비즈니스 논리를 구현하는 POJO BankImpl를 정의
- InvocationHandler를 구현하는 BankProxyHandler를 작성
- 프록시를 사용하면 깨끗한 코드를 작성하기 어렵고, 시스템 단위로 실행 '지점'을 명시하는 메커니즘도 제공하지 않는다.

## 순수 자바 AOP 프레임워크

- 대부분의 프록시 코드는 판박이라 도구로 자동화할 수 있다.
- 스프링 AOP, JBoss AOP 등과 같은 여러 자바 프레임워크는 내부적으로 프록시를 사용한다.
- 스프링은 비즈니스 논리를 POJO로 구현
   - POJO는 도메인에 초점을 맞춘다.
   - 엔터프라이즈 프레임워크에 의존하지 않는다.
   - 테스트가 개념적이고 더 쉽고 간단하여 사용자 스토리를 올바르게 구현하기 쉬우며 보수하고 개선하기 편하다.
- 프로그래머는 설정 파일이나 API를 사용해 필수적인 애플리케이션 기반 구조를 구현한다.

```yml
<beans>
    ...
    <bean id="appDataSource"
        class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="me"/>
    
    <bean id="bankDataAccessObject"
        class="com.example.banking.persistence.BankDataAccessObject"
        p:dataSource-ref="appDataSource"/>
    
    <bean id="bank"
        class="com.example.banking.model.Bank"
        p:dataAccessObject-ref="bankDataAccessObject"/>
    ...
</beans>
```

- 각 빈은 중첩된 러시아 인형의 일부분과 같다.
- 클라이언트는 Bank 객체에서 getAccounts()를 호출한다고 믿지만 실제로는 Bank POJO의 기본 동작을 확장한 중첩 DECORATOR 객체 집합의 가장 외곽과 통신한다.

<img width="615" alt="bank" src="https://user-images.githubusercontent.com/50051656/151554408-d717986b-00da-49a7-a730-c64a5fbbb2ae.png">

```java
XmlBeanFactory bf = new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```

- 애플리케이션에서 DI 컨테이너에 (XML 파일에 명시된) 시스템 내 최상위 객체를 요청하려면 위 코드가 필요하다.
- 스프링 관련 자바 코드가 거의 필요 없음으로 애플리케이션은 **사실상 스프링과 독립적이다.**
- EJB2 시스템이 지녔던 강한 결합(tight-coupling)이라는 문제가 사라진다.
- XML은 장황하고 읽기 어렵다는 문제가 있음에도 불구하고, 프록시보다는 단순하다.
- 이런 아키텍처가 매력적이라 EJB 버전 3를 완전히 뜯어고치는 계기가 되었다.
- XML 설정 파일과 자바 5 어노테이션 기능을 사용해 횡단 관심사를 선언적으로 지원하는 스프링 모델을 따른다.

```java
package com.example.banking.model;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.Collection;

@Entity
@Table(name = "BANKS")
public class Bank implements java.io.Serializable {
    @Id @GeneratedValue(strategy=GenerationType.AUTO)
    private int id;
    
    @Embeddable // An object “inlined” in Bank’s DB row
    public class Address {
        protected String streetAddr1;
        protected String streetAddr2;
        protected String city;
        protected String state;
        protected String zipCode;
    }
    
    @Embedded
    private Address address;
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, mappedBy="bank")
    private Collection<Account> accounts = new ArrayList<Account>();
    public int getId() {
        return id;
    }
    
    public void setId(int id) {
        this.id = id;
    }
    
    public void addAccount(Account account) {
        account.setBank(this);
        accounts.add(account);
    }
    
    public Collection<Account> getAccounts() {
        return accounts;
    }
    
    public void setAccounts(Collection<Account> accounts) {
        this.accounts = accounts;
    }
}
```

- EJB2 코드보다 위 코드가 훨씬 더 깨끗하다.
- 일부 상세한 엔티티 정보는 어노테이션에 포함되어 그대로 남아있지만, 모든 정보가 어노테이션 속에 있어 코드 자체는 깔끔해졌다.
- 어노테이션에 들어있는 영속성 정보는 일부나 전부를 필요하다면 XML 배치 기술자로 옮겨도 되며 그러면 순수 POJO만 남는다.

## AspectJ 관점

- 관심사를 관점으로 분리하는 가장 강력한 도구는 AspectJ 언어이다.
- 스프링 AOP와 JBoss AOP가 제공하는 순수 자바 방식은 관점이 필요한 상황 중 80~90%에 충분하다.
- AspectJ는 관점을 분리하는 강력하고 풍부한 도구 집합을 제공하지만, 새 도구를 사용하고 새 언어 문법과 사용법을 익혀야 하는 단점이 있다.
- 최근에 나온 AspectJ 어노테이션 폼은 새로운 도구와 새로운 언어라는 부담을 어느 정도 완화한다.

## 테스트 주도 시스템 아키텍처 구축

- 코드 수준에서 아키텍처 관심사를 분리할 수 있다면, 진정한 테스트 주도 아키텍처 구축이 가능해진다.
- 그때그때 새로운 기술을 채택해 단순한 아키텍처를 복잡한 아키텍처로 키워나갈 수 있다. 즉 BDUF(Big Design Up Front)를 추구할 필요가 없다.
- 건축가는 BDUF 방식을 취한다. 물리적 구조는 일단 짓기 시작하면 변경하기 쉽지 않기 때문이다.
- 소프트웨어도 나름대로 형체가 있지만, 소프트웨어 구조가 관점을 효과적으로 분리하면, 극적인 변화가 경제적으로 가능하다.
- 최선의 시스템 구조는 각기 POJO 객체로 구현되는 모듈화된 관심사 영역으로 구성된다.

## 의사 결정을 최적화하라

- 모듈을 나누고 관심사를 분리하면 지엽적인 관리와 결정이 가능해진다.
- 의사 결정은 가장 적합한 사람에게 책임을 맡기면 가장 좋다.
- 우리는 때때로 **가능한 마지막 순간까지 결정을 미루는 방법이 최선이라는 사실**을 까먹고 한다.
- 이는 우리가 게으르거나 무책임해서가 아니고 최대한 정보를 모아 최선의 결정을 내리기 위해서이다.

## 명백한 가치가 있을 때 표준을 현명하게 사용하라

- EJB2는 단지 표준이라는 이유만으로 많은 팀이 사용했다
- 가볍고 간단한 설계로 충분했을 프로젝트에서도 EJB2를 채택하였다.
- 표준은 사용하면 아이디어와 컴포넌트를 재사용하기 쉽고, 적절한 경험을 구하기 쉬우며, 좋은 아이디어를 캡슐화하기 쉽고, 컴포넌트를 엮기 쉽다.
- 하지만 때로는 표준을 만드는 시간이 너무 오래 걸려 업계가 기다리지 못한다.

## 시스템은 도메인 특화 언어가 필요하다

- 좋은 DSL은 도메인 개념과 그 개념을 구현한 코드 사이에 존재하는 '의사소통 간극'을 줄여준다. 
- 효과적으로 사용하면 DSL은 추상화 수준을 코드 관용구나 디자인 패턴 이상으로 끌어올려 준다.

## 결론

- 시스템 역시 깨끗해야 한다.
- 깨끗하지 못한 아키텍처는 도메인 논리를 흐리며 기민성을 떨어뜨리고 도메인 논리가 흐려지면 제품 품질이 떨어진다.
- 모든 추상화 단계에서 의도는 명확히 표현해야 한다.
   - POJO 작성
   - 관점 혹은 관점과 유사한 메커니즘을 사용해 각 구현 관심사를 분리해야 한다.
- 시스템을 설계하든 개별 모듈을 설계하든, **실제로 돌아가는 가장 단순한 수단을 사용**해야 한다.
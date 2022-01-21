# 10장. 클래스 p.172

1. [클래스 체계](#클래스-체계-p.172)
    1. [캡슐화](#캡슐화)
1. [클래스는 작아야 한다!](#클래스는-작아야-한다!-p.172)
    1. [단일 책임 원칙](#단일-책임-원칙(Single-Responsibility-Principle,-SRP))
    1. [응집도](#응집도(Cohesion))
    1. [응집도를 유지하면 작은 클래스 여럿이 나온다](#응집도를-유지하면-작은-클래스-여럿이-나온다)
1. [변경하기 쉬운 클래스](#변경하기-쉬운-클래스-p.185)
    1. [Before](#Before)
    1. [After](#After)
    1. [변경으로부터 격리](#변경으로부터-격리)
    
## 클래스 체계 p.172

- 클래스를 정의하는 표준 자바 관례

1. static public
1.static private
1. private 인스턴스 변수
1. ~~public 변수~~
1. 공개 함수
1. 비공개 함수(자신을 호출하는 공개 함수 직후)

- 즉, 추상화 단계가 순차적으로 내려간다. 그래서 프로그램은 신문기사처럼 읽힌다.

### 캡슐화

- 변수와 유틸리티 함수는 가능한 한 숨겨야 한다. 때로는 테스트를 위해 protected로 선언해 허용하기도 한다. 
- 하지만, 그 전에 비공개 상태로 유지할 방법을 강구하고 캡슐화를 풀어주는 결정은 최후의 수단이다.

## 클래스는 작아야 한다! p.172

- 클래스를 만들 때의 규칙은 크기이며 작아야 한다. 두 번째 규칙 또한 크기가 더 작아야 한다.
- 함수와 마찬가지로 '작게'가 기본 규칙
- 함수는 물리적인 행 수로 크기를 측정했다면, 클래스는 맡은 책임으로 센다.
- Processor, Manager, Super 동과 같이 클래스 이름이 모호하다면 

### 단일 책임 원칙(Single Responsibility Principle, SRP)

- 클래스나 모듈을 변경할 이유는 단 하나뿐이어야 한다는 원칙

```java
public class SuperDashboard extends JFrame implements MetaDataUser {
   public Component getLastFocusedComponent()
   public void setLastFocused(Component lastFocused)
   public int getMajorVersionNumber()
   public int getMinorVersionNumber()
   public int getBuildNumber()
}
```

- 위 예제는 변경해야 할 이유가 두 가지이다.
- 첫째, SuperDashboard는 소프트웨어 버전 정보를 추적한다.
- 둘째, SuperDashboard는 자바 스윙 컴포넌트를 관리한다.
- 버전을 관리하는 독자적인 클래스를 만든다.

```java
public class Version {
    public int getMajorVersionNumber() 
    public int getMinorVersionNumber() 
    public int getBuildNumber()
}
```

- SRP는 클래스 설계자가 가장 무시하는 규칙 중 하나
- 우리들 대다수는 '깨끗하고 체계적인 소프트웨어' 보다는 '돌아가는 소프트웨어'에 초점을 맞추기 때문
- 하지만 작은 클래스가 많은 시스템이든 큰 클래스가 몇 개뿐인 시스템이든 돌아가는 부품은 그 수가 비슷하다.
- **큰 클래스 몇 개가 아니라 작은 클래스 여럿으로 이뤄진 시스템이 더 바람직**하다.

### 응집도(Cohesion)

- 클래스는 인스턴스 변수 수가 적어야 한다.
- 각 클래스 메소드는 클래스 인스턴스 변수를 하나 이상 사용해야 한다.
- 일반적으로 메소드가 변수를 더 많이 사용할수록 메소드와 클래스는 응집도가 높다.

```java
public class Stack {
    private int topOfStack = 0;
    List<Integer> elements = new LinkedList<Integer>();
 
    public int size() { 
        return topOfStack;
    }
 
    public void push(int element) { 
        topOfStack++; 
        elements.add(element);
    }
    
    public int pop() throws PoppedWhenEmpty { 
        if (topOfStack == 0)
            throw new PoppedWhenEmpty();
        int element = elements.get(--topOfStack); 
        elements.remove(topOfStack);
        return element;
    }
}
```

- 때때로 몇몇 메서드만이 사용하는 인스턴스 변수가 많아진다면 새로운 클래스로 쪼개야 한다는 신호이다.

### 응집도를 유지하면 작은 클래스 여럿이 나온다

- 큰 함수를 작은 함수 여럿으로 나누기만 해도 클래스 수가 많아진다.
- 예로 변수가 아주 많은 큰 함수가 하나 있다 -> 큰 함수 일부를 작은 함수로 뺀다 -> 빼내려는 코드가 큰 함수에 정의된 변수 넷을 사용한다.
- -> 새 함수에 인자로?
- -> 인스턴스 변수로 승격?
- 만약 인스턴스 변수로 승격시킨다면 편하기는 하지만 응집력 잃고 새 함수로 인자를 넘기면 잃지 않는다.
- p.179 예제 소스 코드

## 변경하기 쉬운 클래스 p.185

- 대다수 시스템은 지속적인 변경이 가해짐
- 깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 수반하는 위험을 낮춤

### Before

```java
public class Sql {
    public Sql(String table, Column[] columns)
    public String create()
    public String insert(Object[] fields)
    public String selectAll()
    public String findByKey(String keyColumn, String keyValue)
    public String select(Column column, String pattern)
    public String select(Criteria criteria)
    public String preparedInsert()
    private String columnList(Column[] columns)
    private String valuesList(Object[] fields, final Column[] columns) private String selectWithCriteria(String criteria)
    private String placeholderList(Column[] columns)
}
```

- 새로운 SQL 문을 지원하거나 기존 SQL문을 수정할 때도 Sql 클래스를 손대야 한다.

### After

```java
abstract public class Sql {
    public Sql(String table, Column[] columns) 
    abstract public String generate();
}
public class CreateSql extends Sql {
    public CreateSql(String table, Column[] columns) 
    @Override public String generate()
}

public class SelectSql extends Sql {
    public SelectSql(String table, Column[] columns) 
    @Override public String generate()
}

public class InsertSql extends Sql {
    public InsertSql(String table, Column[] columns, Object[] fields) 
    @Override public String generate()
    private String valuesList(Object[] fields, final Column[] columns)
}

public class SelectWithCriteriaSql extends Sql { 
    public SelectWithCriteriaSql(
    String table, Column[] columns, Criteria criteria) 
    @Override public String generate()
}

public class SelectWithMatchSql extends Sql { 
    public SelectWithMatchSql(String table, Column[] columns, Column column, String pattern) 
    @Override public String generate()
}

public class FindByKeySql extends Sql public FindByKeySql(
    String table, Column[] columns, String keyColumn, String keyValue) 
    @Override public String generate()
}

public class PreparedInsertSql extends Sql {
    public PreparedInsertSql(String table, Column[] columns) 
    @Override public String generate() {
    private String placeholderList(Column[] columns)
}

public class Where {
    public Where(String criteria) public String generate()
}

public class ColumnList {
    public ColumnList(Column[] columns) public String generate()
}
```

- 공개 인터페이스는 모두 각각 Sql 클래스에서 파생하는 클래스로 만듦
- 비공개 메소드는 해당하는 파생 클래스로 이동
- 이로써 update 문을 추가하더라도 기존 클래스(Sql)를 변경할 필요가 없다.
- SRP, OCP를 지원

### 변경으로부터 격리

- 요구사항은 언제든 변하기 마련이고 따라서 코드 또한 변화한다.
- 추상 클래스는 개념만 포함하고, 구체적인 클래스는 상세 코드를 포함한다.
- 상세 구현에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험에 빠지므로 인터페이스, 추상 클래스로 구현에 미치는 영향을 격리한다.

```java
public interfae StocckExchange {
    Money currentPrice(String symbol);
}
 
public Portfolio {
    private StockExchange exchange;
    public Portfolio(StockExchange exchange) {
        this.exchange = exchange; 
    }
    // ... 
}

public class PortfolioTest {
    private FixedStockExchangeStub exchange;
    private Portfolio portfolio;
    
    @Before
    protected void setUp() throws Exception {
        exchange = new FixedStockExchangeStub(); 
        exchange.fix("MSFT", 100);
        portfolio = new Portfolio(exchange);
    }
 
    @Test
    public void GivenFiveMSFTTotalShouldBe500() throws Exception {
        portfolio.add(5, "MSFT");
        Assert.assertEquals(500, portfolio.value()); 
    }
}
```

- Portfolio는 구체적인 클래스가 아니라 StockExchange 인터페이스에 의존한다.
- StockExchange 인터페이슨 주식 기호를 받아 현재 주식 가격으로 반환한다는 추상적인 개념을 표현
- 이와 같은 추상화로 실제로 주가를 얻어오는 출처나 얻어오는 방식 등과 같은 구체적인 사실을 숨김
- 결합도를 최소로 줄이면서 자연스럽게 DIP를 따르는 클래스가 나오게 된다.
# 2장. 의미 있는 이름

## 의도를 분명히 한다.

- 변수의 **존재 이유, 기능, 사용 방법**이 드러나도록 한다.
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


## 그릇된 정보 / 의미 없는 정보
<ul>
    <li>중의적으로 해석될 여지가 있는 이름 지양한다.</li>
    <li>서로 흡사한 이름 지양한다.</li>
    <ul>
        <li>int o = l; </li>
        <li>int a = 1; </li>
    </ul>
    <li> 의미 있게 구분한다.</li>
    <ul>
        <li> int a1, a2, a2; </li>
        <li> int cnt, sum; </li>
    </ul>
    <li> 불용어를 중복하지 않는다. </li>
    <ul>
        <li> String message;</li>
        <li> String theMessage; </li>
    </ul>
    <ul>
        <li> class Customer {} </li>
        <li> class CustomerObject {} </li>
    </ul>
</ul>


## 발음 / 검색이 쉬운 이름
- 발음하기 어려운 이름은 토론하기도 어렵다.
- 변수 이름의 길이는 범위 크기에 비례해야 한다. 
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

## 적합한 이름

### 클래스 이름
 - 명사 혹은 명사구를 사용한다.
 - `Customer, Account, Address`
 - `Manager, Processor, Data, Info` 와 같은 이름은 지양한다.
 - 동사는 사용하지 않는다.

### 메서드 이름
 - 동사 혹은 동사구를 사용한다.
 - `deletePage, saveAccount`
 - 접근자, 변경자, 조건자는 javabean 표준에 따라 get, set, is를 붙인다.

### 기발한 이름, 인코딩은 피한다.
- 특정 문화에서만 사용되는 이름 혹은 재미 있는 이름보다 **명료한 이름**을 사용한다.
- 헝가리안 표기법은 적절하지 않다.

### 한 개념에 한 단어
<ul>
    <li>똑같은 메서드를 클래스마다 다르게 사용하지 않는다.</li>
    <ul>
        <li>fetch, retrieve, get</li>
    </ul>
</ul>
<ul>
    <li>한 단어를 두 가지 이상의 목적으로 사용하지 않는다.</li>
    <ul>
        <li>public String add(String firstName, String lastName)</li>
        <li>public int add(int currentValue, int nextValue)</li>
    </ul>
</ul>

### 의미 있는 맥락 추가
 - 주소를 표한하고자 할 때는 아래와 같이 표현하는 것이 더 분명해진다.
 - `city → addrCity`
 - `street → addrStreet`
 - `state → addrState`
 - 불필요한 맥락은 없앤다.

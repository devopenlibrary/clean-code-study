# 10장 클래스

## 클래스 체계

클래스를 만들 때는 **변수와 함수의 위치와 순서**를 체계적으로 정리한다.

```java
class {
    // 변수
    public static final ... // 정적 공개 상수
    private static ...      // 정적 비공개 변수
    private ...             // 비공개 인스턴스 변수

    // 함수
    public ...              // 공개 함수 (필요 시 비공개 함수 호출)
    private ...             //   ↪ 유틸리티 등 비공개 함수
}
```

변수는 역할에 따라 가시성을 신중히 조정해야 하며, 공개 함수는 필요한 동작만 외부에 노출해야 한다.

### 캡슐화

**캡슐화를 풀어주는 결정은 언제나 최후의 수단이다.**

- 내부 구현을 외부에 숨기고 필요한 부분만 공개한다.
    - 내부 구현을 외부에서 함부로 건드릴 수 없다.
    - 변경이 필요한 경우에도 **내부만 수정**하면 된다.
- 테스트나 동일 패키지 내에서 접근이 꼭 필요하다면 `protected`나 패키지 접근 수준(default)을 고려할 수 있지만, **항상 감추는 것이 기본**이다.



## 클래스는 작아야 한다

**클래스도 함수와 마찬가지로 작고 명확해야 한다.** 

- 한 클래스가 너무 많은 책임을 지고 있다면, 코드가 복잡해지고 수정이 어려워진다.
- 클래스의 크기는 클래스가 맡은 책임으로 센다.
- 클래스 이름은 해당 클래스 책임을 기술해야 한다.
    - 간결한 이름이 떠오르지 않는다면 클래스 크기가 너무 커서 그렇다.
    - 클래스 이름이 모호하다면 클래스 책임이 너무 많진 않은지 생각해봐야 한다.

👎👎👎

```java
// 클래스 책임이 수억개
public class SuperDashboard extends JFrame implements MetaDataUser {
  public String getCustomizerLanguagePath()
  public void setSystemConfigPath(String systemConfigPath) 
  public String getSystemConfigDocument()
  public void setSystemConfigDocument(String systemConfigDocument) 
  public boolean getGuruState()
  public boolean getNoviceState()
  public boolean getOpenSourceState()
  public void showObject(MetaObject object) 
  public void showProgress(String s)
  public boolean isMetadataDirty()
  public void setIsMetadataDirty(boolean isMetadataDirty)
  public Component getLastFocusedComponent()
  public void setLastFocused(Component lastFocused)
  public void setMouseSelectState(boolean isMouseSelected) 
  public boolean isMouseSelected()
  public LanguageManager getLanguageManager()
  public Project getProject()
  public Project getFirstProject()
  public Project getLastProject()
  public String getNewProjectName()
  public void setComponentSizes(Dimension dim)
  public String getCurrentDir()
  public void setCurrentDir(String newDir)
  public void updateStatus(int dotPos, int markPos)
  public Class[] getDataBaseClasses()
  // ... 그 외 수많은 공개/비공개 메서드 ...
  // ~~ 중략 ~~

  
}
```

👎👎👎

```java
// 메소드는 5개지만 책임도 여러개
public class SuperDashboard extends JFrame implements MetaDataUser {
  public Component getLastFocusedComponent()
  public void setLastFocused(Component lastFocused)
  public int getMajorVersionNumber()
  public int getMinorVersionNumber()
  public int getBuildNumber() 
}
```

### 단일 책임 원칙(SRP, Single Responsibility Principle)

**클래스는 하나의 책임만 가지고 있어야 한다.**

클래스나 모듈을 변경할 이유는 하나뿐이어야 한다.

👍👍👍

```java
// 버전 정보를 다루는 메서드 3개를 따로 빼서 
// Version 이라는 독자적인 클래스를 만들어 다른 곳에서 재사용하기 쉬워졌다.
public class Version {
  public int getMajorVersionNumber() 
  public int getMinorVersionNumber() 
  public int getBuildNumber()
}
```

이렇게 책임을 나누면, 하나의 기능을 변경할 때 다른 기능에 영향을 주지 않게 된다.



### 응집도(Cohesion)

**클래스 내 메서드들이 동일한 데이터에 관심이 없을 경우, 분리할 필요가 있다는 신호이다.**

- **응집도**: 클래스 내부의 메서드와 변수들이 **서로 밀접하게 연관**되어 있음을 말한다.
    - 응집도가 높으면 클래스가 **자기 역할에 집중**하고, 다른 역할을 침범하지 않는다.
- 메서드가 인스턴스 변수를 많이 쓸수록 응집도가 높다.
- 응집도가 낮으면, 클래스를 나누는 것을 고려해야 한다.

👍👍👍

```java
// 높은 응집도
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



### 응집도를 유지하면 작은 클래스 여럿이 나온다

- 하나의 큰 클래스를 만들기보다는 **작은 책임을 가진 클래스들로 쪼개는 것이 바람직**하다.
- 클래스를 쪼개면 응집도는 유지되고, 클래스는 자연스럽게 작아진다.

👎👎👎

```java
public class PrintPrimes {
  public static void main(String[] args) {
    final int M = 1000;
    final int RR = 50;
    final int CC = 4;
    final int WW = 10;
    final int ORDMAX = 30;
    int P[] = new int[M + 1];
    int PAGENUMBER;
    int PAGEOFFSET;
    int ROWOFFSET;
    int C;
    int J;
    int K;
    boolean JPRIME;
    int ORD;
    int SQUARE;
    int N;
    int MULT[] = new int[ORDMAX + 1];

    J = 1;
    K = 1;
    P[1] = 2;
    ORD = 2;
    SQUARE = 9;

    while (K < M) {
      do {
        J = J + 2;
        if (J == SQUARE) {
          ORD = ORD + 1;
          SQUARE = P[ORD] * P[ORD];
          MULT[ORD - 1] = J;
        }
        N = 2;
        JPRIME = true;
        while (N < ORD && JPRIME) {
          while (MULT[N] < J)
            MULT[N] = MULT[N] + P[N] + P[N];
          if (MULT[N] == J)
            JPRIME = false;
          N = N + 1;
        }
      } while (!JPRIME);
      K = K + 1;
      P[K] = J;
    }
    {
      PAGENUMBER = 1;
      PAGEOFFSET = 1;
      while (PAGEOFFSET <= M) {
        System.out.println("The First " + M + " Prime Numbers --- Page " + PAGENUMBER);
        System.out.println("");
        for (ROWOFFSET = PAGEOFFSET; ROWOFFSET < PAGEOFFSET + RR; ROWOFFSET++) {
          for (C = 0; C < CC; C++)
            if (ROWOFFSET + C * RR <= M)
              System.out.format("%10d", P[ROWOFFSET + C * RR]);
          System.out.println("");
        }
        System.out.println("\f");
        PAGENUMBER = PAGENUMBER + 1;
        PAGEOFFSET = PAGEOFFSET + RR * CC;
      }
    }
  }
}

1. 명확하지 않은 변수명
2. 단일 main 메서드에 모든 로직 집중 (책임 분리x)
3. 하드코딩된 상수 및 매직 넘버
4. 복잡한 제어 흐름
```

👍👍👍 각 책임을 클래스로 나누면?

```java
// 실행 담당
public class PrimePrinter {
  public static void main(String[] args) {
    final int NUMBER_OF_PRIMES = 1000;
    int[] primes = PrimeGenerator.generate(NUMBER_OF_PRIMES);

    final int ROWS_PER_PAGE = 50;
    final int COLUMNS_PER_PAGE = 4;
    RowColumnPagePrinter tablePrinter =
        new RowColumnPagePrinter(ROWS_PER_PAGE,
            COLUMNS_PER_PAGE,
            "The First " + NUMBER_OF_PRIMES + " Prime Numbers");
    tablePrinter.print(primes);
  }
}

// 프린트 포맷 담당
public class RowColumnPagePrinter {
  private int rowsPerPage;
  private int columnsPerPage;
  private int numbersPerPage;
  private String pageHeader;
  private PrintStream printStream;

  public RowColumnPagePrinter(int rowsPerPage, int columnsPerPage, String pageHeader) {
    this.rowsPerPage = rowsPerPage;
    this.columnsPerPage = columnsPerPage;
    this.pageHeader = pageHeader;
    numbersPerPage = rowsPerPage * columnsPerPage;
    printStream = System.out;
  }

  public void print(int data[]) {
    int pageNumber = 1;
    for (int firstIndexOnPage = 0;
        firstIndexOnPage < data.length;
        firstIndexOnPage += numbersPerPage) {
      int lastIndexOnPage = Math.min(firstIndexOnPage + numbersPerPage - 1, data.length - 1);
      printPageHeader(pageHeader, pageNumber);
      printPage(firstIndexOnPage, lastIndexOnPage, data);
      printStream.println("\f");
      pageNumber++;
    }
  }
  
  private void printPage(int firstIndexOnPage, int lastIndexOnPage, int[] data) {
    int firstIndexOfLastRowOnPage =
        firstIndexOnPage + rowsPerPage - 1;
    for (int firstIndexInRow = firstIndexOnPage;
        firstIndexInRow <= firstIndexOfLastRowOnPage;
        firstIndexInRow++) {
      printRow(firstIndexInRow, lastIndexOnPage, data);
      printStream.println("");
    }
  }
  
  ...
  
}

// 소수 판별 담당
public class PrimeGenerator {
  private static int[] primes;
  private static ArrayList<Integer> multiplesOfPrimeFactors;

  protected static int[] generate(int n) {
    primes = new int[n];
    multiplesOfPrimeFactors = new ArrayList<Integer>();
    set2AsFirstPrime();
    checkOddNumbersForSubsequentPrimes();
    return primes;
  }

  private static void set2AsFirstPrime() {
    primes[0] = 2;
    multiplesOfPrimeFactors.add(2);
  }

  private static void checkOddNumbersForSubsequentPrimes() {
    int primeIndex = 1;
    for (int candidate = 3; primeIndex < primes.length; candidate += 2) {
      if (isPrime(candidate))
        primes[primeIndex++] = candidate;
    }
  }

  private static boolean isPrime(int candidate) {
    if (isLeastRelevantMultipleOfNextLargerPrimeFactor(candidate)) {
      multiplesOfPrimeFactors.add(candidate);
      return false;
    }
    return isNotMultipleOfAnyPreviousPrimeFactor(candidate);
  }
  
	...

}

1. 좀 더 길고 서술적인 변수명 사용
2. 함수 선언과 클래스 선언으로 코드를 설명
3. 가독성을 위해 공백을 추가 및 형식 맞춤
```

클래스는 작아지고 각자의 책임에 집중할 수 있다.

1. 가장 먼저 원래 프로그램의 정확한 동작을 검증하는 테스트 슈트를 작성하라.
2. 그 다음 한번에 하나씩 여러번에 걸쳐 코드를 변경하고,
3. 코드를 변경 할 때 마다 테스트를 수행해 원래 프로그램과 동일하게 동작하는지 확인하라.



## 변경하기 쉬운 클래스

우리가 작성하는 대부분의 소프트웨어는 시간이 지나면서 **변경**이 필요해진다(위험 ⏫). 클래스가 체계적으로 정리되어 있으면 변경도 훨씬 쉬워진다.

- 공개할 필요 없는 메서드는 반드시 **감춘다.**
    - 외부 클래스와의 **불필요한 결합**을 피하기 위함

👎👎👎

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
  private String valuesList(Object[] fields, final Column[] columns)
  private String selectWithCriteria(String criteria)
  private String placeholderList(Column[] columns)
}
```

👍👍👍

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

...

// update 문을 추가 할때 기존 클래스를 변경할 필요가 없다.
// 테스트 관점에서도 모든 논리를 검증하고 증명하기 쉬워졌다.
```


### 변경으로부터 격리

**상세 구현보다 추상화에 의존하라**

- 구현에 직접 의존하는 코드는 변경에 취약하다. 인터페이스나 추상 클래스를 통해 구현 세부사항을 감추고, 변경으로부터 코드를 격리해야 한다.
- 추상화에 의존하면 결합도가 낮아지고, 테스트도 쉬워지며 재사용성과 유연성도 높아진다.

예시)

Portfolio라는 클래스가 외부 TokyoStockExchange API를 사용한다고 할 때, 

```java
// 5분마다 값이 달라지는 주식 시세를 가져오는 API
public interface StockExchange {
  Money currentPrice(String symbol);
}
```

Portfolio는 직접 API를 호출하지 않고 StockExchange 인터페이스에 의존한다.

TokyoStockExchange는 이 인터페이스를 구현한다.

```java
public class Portfolio {
  private StockExchange exchange;

  public Portfolio(StockExchange exchange) {
    this.exchange = exchange;
  }

  // ...
}

```

Portfolio 생성자에서 StockExchange를 인수로 받는다.

```java
public class PortfolioTest {
  private FixedStockExchangeStub exchange;
  private Portfolio portfolio;

  @Before
  public void setUp() {
    exchange = new FixedStockExchangeStub();
    exchange.fix("MSFT", 100);
    portfolio = new Portfolio(exchange);
  }

  @Test
  public void testTotalValue() {
    portfolio.add(5, "MSFT");
    Assert.assertEquals(500, portfolio.value());
  }
}

```

결과적으로 실제 주가를 어디서 어떻게 가져오는지는 숨기고, Portfolio는 StockExchange라는 인터페이스에만 의존하므로 훨씬 유연하고 관리하기 쉬워진다.

- 결합도를 낮추면  **유연성과 재사용성**이 높아진다.
    - 각 요소가 서로 독립적이고, 변경에 덜 민감해진다.
    - 시스템 요소가 서로 잘 격리되어 있으면 각 요소를 이해하기 더 쉽다.
- 결합도를 최소로 줄이면 자연스럽게 **DIP(Dependency Inversion Principle)**를 따르는 클래스가 나온다.
    - DIP : 클래스가 상세한 구현이 아니라 **추상화에 의존해야 한다**는 원칙



## 요약

- 클래스는 변수와 함수를 체계적으로 배치한다.
- 캡슐화로 내부를 숨기고, 꼭 필요한 부분만 공개한다.
- 클래스는 작고 단일 책임 원칙을 지켜야 한다.
- 응집도가 높아야 클래스가 자기 역할에 집중할 수 있다.
- 변경에 대비해 추상화와 낮은 결합도를 설계한다.
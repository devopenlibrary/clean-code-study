---
nav_order: 17
---

# 17장 냄새와 휴리스틱

아래 목록들은 다양한 프로그램을 리팩터링하는 과정에서, 수정할 때마다 "왜?"라고 자문하고 그 답을 기록한 결과이다. 

_즉 아래 목록들은 코드를 수정해야 하는 이유인 냄새(나쁜 냄새)를 풍기는 요소들이며, **지양해야 하는** 요소들이다._

## 주석
### _C1: 부적절한 정보_
- 다른 시스템(e.g. 소스 코드 관리 시스템, 이슈 추적 시스템 등)에 저장할 정보는 주석으로 적절치 못하다. 
- **주석은 코드와 설계에 기술적인 설명을 부연하는 수단**이다.

### _C2: 쓸모없는 주석_
- 오래된, 엉뚱한, 잘못된 주석은 쓸모없다. 
- 쓸모없는 주석은 아예 달지 않거나, 발견하는 즉시 삭제하는 편이 좋다.

### _C3: 중복된 주석_
- 중복된 주석은 코드만으로 충분한데 구구절절 설명하는 주석이다. 
- 다음은 중복된 주석의 예이다.

    - ```java
        i++; // i 증가
        ```
    혹은, 함수 서명만 달랑 기술하는 `Javadoc`이다.
    - ```java
         /**
        * @param sellRequest
        * @return
        * @throws ManagedComponentException
        */
        public SellResponse beginSellItem(SellRequest sellRequest)
        throws ManagedComponentException
      ```

- 다시 한 번, **주석은 코드만으로 다하지 못하는 설명을 부연하는 역할이다.**

### _C4: 성의 없는 주석_
- 주석을 작성하려면, 공을 들여서 작성해야 한다. 
- 단어는 신중하게 선택하고, 당연한 소리를 반복하지 않는다. 
- 간결하고 명료하게 작성한다.

### _C5: 주석 처리된 코드_
- 주석 처리된 코드는 흉물 그 자체다. 그러니 발견 즉시 지워버려라. 
- 소스 코드 관리 시스템이 기억한다. 
- 주석 처리된 코드는 얼마나 오래되었는지, 사용하는 코드인지 아무도 알 수가 없다.

## 환경
### _E1: 여러 단계로 빌드해야 한다_
- 빌드는 간단히 한 단계로 끝나야 한다. 
- 소스 코드 관리 시스템에서 이것저것 체크아웃할 필요가 없어야 한다. 
- 또한 불가해한 명령이나 스크립트를 잇달아 실행해 각 요소를 따로 빌드할 필요가 없어야 한다.

### _E2: 여러 단계로 테스트해야 한다_
- 모든 단위 테스트는 한 명령으로 돌려야 한다.
- IDE에서 버튼 하나로 모든 테스트를 돌린다면 가장 이상적이다.

## 함수
### _F1: 너무 많은 인수_
- **함수에서 인수 개수는 작을수록 좋다.**
- 인수가 넷 이상이라면 의심하고 최대한 피한다.

### _F2: 출력 인수_
- 일반적으로 인수는 입력으로 간주된다. 따라서 출력 인수는 직관적으로 이해하기 매우 어렵다.
- 함수에서 뭔가의 상태를 변경해야 한다면, 함수가 속한 객체의 상태를 변경한다.

### _F3: 플래그 인수_
- 플래그 인수는 그 함수가 둘 이상의 일을 수행한다는 명백한 증거이므로 피해야 한다.

### _F4: 죽은 함수_
- 아무도 호출하지 않는 함수는 과감히 삭제하라.

## 일반
### _G1: 한 소스 파일에 여러 언어를 사용한다_
- 오늘날 프로그래밍 환경은 한 소스 파일 내에서 다양한 언어를 지원한다.
- 가능한 소스 파일 하나에 언어 하나만 사용하는 방식이 가장 좋다.
- 현실적으로 여러 언어가 불가피하지만 각별한 노력을 기울여 최대한 줄이도록 노력한다.

### _G2: 당연한 동작을 구현하지 않는다_
- 최소 놀람의 원칙에 의거해 함수나 클래스는 다른 프로그래머가 당연하게 여길 만한 동작과 기능을 제공해야 한다. 
    - > 소프트웨어 설계에서 시스템이 사용자의 예상에 최대한 부합하도록 설계해야 한다는 원칙
- 예를 들어, 요일 문자열에서 요일을 나타내는 `enum`으로 변환하는 함수를 살펴보자.
    - ```java
        Day day = DayDate.StringToDay(String dayName);
       ```
    - 우리는 함수가 'Monday'를 Day.MONDAY로 변환하리라 기대한다. 또한 일반적으로 쓰는 요일 약어도 올바로 변환하리라 기대한다. 
- 당연한 동작을 구현하지 않으면 독자는 더 이상 함수 이름만으로 기능을 직관적으로 예상하기 어려워진다.

### _G3: 경계를 올바로 처리하지 않는다_
- 자신의 직관에 의존하지 말고, 부지런히 모든 경계 조건, 모든 예외를 살펴보자.
- **모든 경계 조건을 찾아내고, 그것들을 테스트하는 테스트 케이스를 작성하라.**

### _G4: 안전 절차 무시_
- 예를 들어 컴파일러 경고 일부를 꺼버리면 빌드가 쉬워질지 모르지만, 자칫하면 끝없는 디버깅에 시달릴 수 있다.
- 실패하는 테스트 케이스를 일단 제쳐두고 나중에 미루는 태도는 아주 위험하다.    

### _G5: 중복_
- **이 책에 나오는 가장 중요한 규칙 중 하나이므로 심각하게 숙고해야 한다.**
- 코드에서 중복을 발견할 때마다 추상화할 기회로 간주하라. 중복된 코드를 하위 루틴이나 다른 클래스로 분리하라.
- **어디서든 중복을 발견하면 없애라.**
---
- _유형 1:_ 똑같은 코드가 여러 차례 나온다. 이런 중복은 간단한 함수로 교체한다.
- _유형 2:_ 여러 모듈에서 분기문으로 똑같은 조건을 거듭 확인하는 중복이다. 이런 중복은 다형성으로 대체해야 한다.
- _유형 3:_ 알고리즘이 유사하나 코드가 서로 다른 중복이다. 이는 TEMPLATE METHOD 패턴이나, STRATEGY 패턴으로 중복을 제거한다. 
---

### _G6: 추상화 수준이 올바르지 못하다_
- 추상화로 개념을 분리할 때는 철저해야 한다. 모든 저차원 개념은 파생 클래스에 넣고, 모든 고차원 개념은 기초 클래스에 넣는다.
- **기초 클래스는 구현 정보에 무지해야 한다.** 예를 들어, 세부 구현과 관련한 상수, 변수, 유틸리티 함수 등은 기초 클래스에 넣으면 안 된다.

```java
public interface Stack {
    Object pop() throws EmptyException;
    /* ... */
    double percentFull();
    /* ... */
}
```
- `percentFull()`은 추상화 수준이 올바르지 못하다. 이유는 Stack을 구현하는 방법은 다양해, percent full(백분율)이라는 개념은 구현하는 방식에 따라 적합할 수도, 그렇지 않을 수도 있다. **따라서 해당 개념은 특정한 파생 인터페이스에 넣어야 마땅하다.**

### _G7: 기초 클래스가 파생 클래스에 의존한다_
- 일반적으로 기초 클래스는 파생 클래스를 아예 몰라야 마땅하다. 고차원 개념을 저차원 개념으로부터 분리하는 것이 기초/파생 클래스를 나누는 이유이기 때문이다.
- 하지만 예외는 있다. 예를 들어 파생 클래스 개수가 확실히 고정되었다면 기초 클래스에 **파생 클래스를 선택하는 코드**가 들어간다.
    - 이러한 경우 기초 클래스와 파생 클래스를 각기 다른 파일로 배포하는 편이 좋다.
    - 각기 다른 파일로 배포될 경우, 시스템에서 각기 개별 컴포넌트 단위로 배치할 수 있기 때문이다.

### _G8: 과도한 정보_
- **잘 정의된 모듈은 인터페이스가 아주 작다.** **부실한 모듈은 구질구질하다.**
- 잘 정의된 인터페이스는 많은 함수를 제공하지 않아 결합도가 낮다. 부실한 인터페이스는 **반드시 호출해야 하는 함수들을 제공한다.**
- **정보를 제한해 인터페이스를 매우 작게 만들어라. 결합도를 낮추어라.**
    - 자료와 유틸리티 함수를 숨겨라. 상수와 임시 변수를 숨겨라.

### _G9: 죽은 코드_
- 실행되지 않는 코드를 가리킨다.
- 예를 들어,
    - 불가능한 조건을 확인하는 if, switch/case 문
    - throw 문이 없는 try 문에서 catch 블록
- 죽은 코드는 설계가 변하거나, 새로운 규칙이나 표기법이 생겨도 제대로 수정되지 않는다.
- 발견 시 제거하라.

### _G10: 수직 분리_
- 변수와 함수는 사용되는 위치에 가깝게 정의한다. 선언한 위치로부터 몇백 줄 아래에서 사용하면 안 된다.
- 비공개 함수는 처음으로 호출한 직후에 정의한다.

### _G11: 일관성 부족_
- 어떤 개념을 특정 방식으로 구현했다면 유사한 개념도 같은 방식으로 구현한다.

### _G12: 잡동사니_
- 아무도 사용하지 않는 변수, 아무도 호출하지 않는 함수, 정보를 제공하지 못하는 주석. 모두 잡동사니이다. 제거해라.

### _G13: 인위적 결합_
- **서로 무관한 개념을 인위적으로 결합하지 않는다.**
- **뚜렷한 목적 없이 변수, 상수, 함수를 당장 편한 위치에 넣어버린 결과이다.** 올바른 위치를 고민하자.

### _G14: 기능 욕심_
- 클래스 메서드는 자기 클래스의 변수와 함수에 관심을 가져야지 다른 클래스의 변수와 함수에 관심을 가져서는 안 된다.
- 메서드가 다른 객체의 내용을 조작하는 것은, 그 객체 클래스의 범위를 욕심내는 것이다.
- 기능 욕심은 한 클래스의 속사정을 다른 클래스에 노출하므로, 별다른 문제가 없다면 제거하는 편이 좋다. 

- 다음은 기능 욕심에 관한 예제이다.

```java
public class HourlyPayCalculator {
    public Money calculateWeeklyPay(HourlyEmployee e) {
        int tenthRate = e.getTenthRate().getPennies();
        int tenthsWorked = e.getTenthsWorked();

        int straightTime = Math.min(400, tenthsWorked);
        int overTime = Math.max(0, tenthsWorked - straightTime);
        int straightPay = straightTime * tenthRate;
        int overtimePay = (int) Math.round(overTime * tenthRate * 1.5);
        return new Money(straightPay + overtimePay);
    }
}
```

`calculateWeeklyPay` 메서드는 `HourlyEmployee` 객체에서 온갖 정보를 가져온다. 즉, `calculateWeeklyPay` 메서드는 `HourlyEmployee` 클래스의 범위를 욕심 낸다. 

### _G15: 선택자 인수_

- 선택자 인수는 큰 함수를 작은 함수 여럿으로 쪼개지 않으려는 게으름의 소산이다. 다음 코드를 살펴보자.

```java
public int calculateWeeklyPay(boolean overtime) {
    int tenthRate = getTenthRate();
    int tenthsWorked = getTenthsWorked();
    int straightTime = Math.min(400, tenthsWorked);
    int overTime = Math.max(0, tenthsWorked - straightTime);
    int straightPay = straightTime * tenthRate;
    double overtimeRate = overtime ? 1.5 : 1.0 * tenthRate;
    int overtimePay = (int) Math.round(overTime * overtimeRate);
    return straightPay + overtimePay;
}
```

- 독자는 `calculateWeeklyPay(false)`라는 코드를 발견할 때마다 인수의 의미를 떠올리느라 골치를 앓는다. 

- enum, int 등 함수 동작을 제어하려는 인수 또한 바람직 하지 않다.
- **인수를 넘겨 동작을 선택하는 대신, 새로운 함수를 만드는 편이 좋다.**

### _G16: 모호한 의도_
- 코드를 짤 때는 의도르르 최대한 분명히 밝힌다.
- 행을 바꾸지 않는 수식, 헝가리식 표기법, 매직 번호 등은 모두 저자의 의도를 흐린다.

### _G17: 잘못 지운 책임_
- 코드를 배치하는 기준은 **독자가 자연스럽게 기대할 만한 곳이다.** 이는 역시 최소 놀람의 원칙을 적용한다.

### _G18: 부적절한 static 함수_
- `Math.max()` 는 좋은 `static` 메서드다. **메서드를 소유하는 객체에서 가져오는 정보가 아니며, 해당 메서드를 재정의할 가능성이 없기 때문이다.**
- 하지만 다음 예를 살펴보자.
    ```java
    HourlyPayCalculator.calculatePay(employee, overtimeRate);
    ```
   언뜻보면 적절해보이지만, 함수를 재정의할 가능성이 존재한다. 혹은 수당을 계산하는 알고리즘이 여러 개일 수도 있다.
- 일반적으로 `static` 함수보다 인스턴스 함수가 더 좋다. `static` 함수를 정의해야 할 때는 재정의할 가능성이 없는지 살펴보자.

---
_다음 목록부터는 휴리스틱을 중점적으로 다룬다._

> 휴리스틱은 복잡하거나 불확실한 문제 상황에서, 제한된 시간이나 정보로 인해 합리적이고 체계적인 판단이 어렵거나 굳이 필요하지 않을 때 사용하는 간편하고 실용적인 문제 해결 방법 또는 추론 전략 이다.

### _G19: 서술적 변수_
- 프로그램 가독성을 높이는 가장 효과적인 방법 중 하나가 계산을 여러 단계로 나누고 중간 값으로 서술적인 변수 이름을 사용하는 방법이다.
```java
Matcher match = headerPattern.matcher(line);
if(match.find())
{
    String key = match.group(1);
    String value = match.group(2);
    headers.put(key.toLowerCase(), value);
}
```

- 서술적인 이름을 사용한 탓에 `key` 와 `value` 로 원하는 정보가 명확하게 드러난다.

### _G20: 이름과 기능이 일치하는 함수_
- 이름만으로 분명하지 않은 함수는 구현을 살펴보거나 문서를 뒤적여야 한다. 
- 기능을 정확하게 표현하는 이름을 짓는다.

### _G21: 알고리즘을 이해하라_
- **구현이 끝났다고 선언하기 전에 함수가 돌아가는 방식을 확실히 이해하는 지 확인하라.** 이것은 테스트 케이스를 모두 통과하는 것과는 별개의 이야기이다.
### _G22: 논리적 의존성은 물리적으로 드러내라_
- 의존하는 모듈이 상대 모듈에 대해 뭔가를 가정하면 안된다. 의존하는 모든 정보를 명시적으로 요청하는 편이 좋다.

- 예를 들어, 근무시간 보고서를 가공되지 않은 상태로 출력하는 함수를 구현한다고 가정하자. `HourlyReporter`라는 클래스는 모든 정보를 모아 `HourlyReportFormatter`에 적당한 형태를 넘긴다. `HourlyReportFormatter`는 넘어온 정보를 출력한다.

```java
public class HourlyReporter {
    private HourlyReportFormatter formatter;
    private List<LineItem> page;
    private final int PAGE_SIZE = 55;

    public HourlyReporter(HourlyReportFormatter formatter) {
        this.formatter = formatter;
        page = new ArrayList<LineItem>();
    }

    public void generateReport(List<HourlyEmployee> employees) {
        for (HourlyEmployee e : employees) {
            addLineItemToPage(e);
            if (page.size() == PAGE_SIZE)
                printAndClearItemList();
        }
        if (page.size() > 0)
            printAndClearItemList();
    }

    private void printAndClearItemList() {
        formatter.format(page);
        page.clear();
    }

    private void addLineItemToPage(HourlyEmployee e) {
        LineItem item = new LineItem();
        item.name = e.getName();
        item.hours = e.getTenthsWorked() / 10;
        item.tenths = e.getTenthsWorked() % 10;
        page.add(item);
    }

    public class LineItem {
        public String name;
        public int hours;
        public int tenths;
    }
}

```

위에서 논리적인 의존성은 `PAGE_SIZE` 이다. `HourlyReporter`가 페이지의 크기를 알 필요는 없다. 페이지 크기는 `HourlyReportFormatter`가 책임질 정보다. 이는 잘못 지운 책임에 해당된다.

`HourlyReportFormatter` 에 `getMaxPageSize()` 라는 메서드를 추가하면 논리적인 의존성이 물리적인 의존성으로 변한다.

### _G23: If/Else 혹은 Switch/Case 문보다 다형성을 사용하라_
- 대부분 `switch/case` 문을 선택하는 이유는 **당장 가장 손쉬운 선택이기 때문** 이다. 그러므로 먼저 다형성을 고려해보자.
- 선택 유형 하나에는 `switch` 문을 한번만 사용한다.
- 같은 선택을 수행하는 다른 코드에서는 다형성 객체를 생성해 `switch` 문을 대신한다.

### _G24: 표준 표기법을 따르라_
- 팀은 업계 표준에 기반한 구현 표준을 따라야 한다.
- 팀이 정한 표준은 팀원들 모두가 따라야 한다.

### _G25: 매직 숫자는 명명된 상수로 교체하라_
- 이미 오래된 규칙이다. 코드에서 숫자를 사용하지 말라는 규칙이다.
- 매직 숫자는 단순히 숫자만 의미하지 않는다. 의미가 분명하지 않은 토큰을 모두 가리킨다.
    ```java
    assertEquals(7777, Employee.find("John Doe").employeeNumber());
    ```
   위 코드에서 매직 숫자는 "7777", "John Doe" 이다. 둘 다 의미가 분명하지 않기 때문이다. 다음과 같이 수정해보자.
   ```java
   assertEqulas(HOURLY_EMPLOYEE_ID, Employee.find(HOURLY_EMPLOYEE_NAME).employeeNumber());
   ```

### _G26: 정확하라_
- **코드에서 뭔가를 결정할 때는 정확히 결정한다. 결정을 내리는 이유와 예외를 처리할 방법을 분명히 알아야 한다.**
- 예를 들어,
    - 호출하는 함수가 null 을 반환할지 모른다면 null 을 반드시 점검한다.
    - 조회 결과가 하나뿐이라고 짐작한다면 하나인지 확실히 확인한다.
    - 동시에 갱신할 가능성이 있다면 적절한 잠금 매커니즘을 구현한다.

### _G27: 관례보다 구조를 사용하라_
- 설계 결정을 강제할 때는 규칙보다 관례를 사용한다.
- 예를 들어, enum 변수가 멋진 switch/case 문보다 추상 메서드가 있는 기초 클래스 가 더 좋다.
- switch/case 문을 매번 똑같이 구현하게 강제하기는 어렵지만, 파생 클래스는 추상 메서드를 모두 구현하지 않으면 안 되기 때문이다.

### _G28: 조건을 캡슐화하라
- 부울 논리는 이해하기 어렵다. 조건의 의도를 분명히 밝히는 함수로 표현하라.
- 예를 들어, 
    -  ```java
        if (shouldBeDeleted(timer))
       ``` 
       라는 코드는 다음 코드보다 좋다.
    - ```java
        if (timer.hasExpired() && !timer.isRecurrent())
      ```

### _G29: 부정 조건은 피하라_
- 부정 조건은 긍정 조건보다 이해하기 어렵다. 가능하면 긍정 조건으로 표현한다.
- 예를 들어,
    - ```java
        if (buffer.shouldCompact())
      ```
      라는 코드가 아래 코드보다 좋다.
    - ```java
        if (!buffer.shouldNotCompact())
      ```

### _G30: 함수는 한 가지만 해야 한다_
- 함수를 짜다 보면, 한 함수안에서 일련의 작업을 수행하고픈 유혹에 빠진다. 하지만 이는 각각의 작업을 담당하는 여럿의 함수로 나누어야한다.
### _G31: 숨겨진 시간적인 결합_
- 때로는 시간적인 결합이 필요하다. 함수를 짤 때는 함수 인수를 적절히 배치해 함수가 호출되는 순서를 명백히 드러낸다. 다음 코드를 살펴보자.

```java
public class MoogDiver {
    Gradient gradient;
    List<Spline> splines;

    public void dive(String reason) {
        saturatedGradient();
        reticulateSplines();
        diverForMoog(reason);
    }
    ...
}
```

위 코드에서는 세 함수가 실행되는 순서가 중요하다.

먼저 `gradient`를 처리하기 위해 `saturatedGradient()`를 호출하고 나서, `reticulateSplines()`를, 마지막으로 `diverForMoog()`를 수행한다.

하지만 위의 코드는 시간적인 결합을 강제하지 않는다. 따라서, 도중에 오류가 발생해도 막을 도리가 없다. 다음 코드가 더 좋다.

```java
public class MoogDiver {
    Gradient gradient;
    List<Spline> splines;

    public void dive(String reason) {
        Gradient gradient = saturateGradient();
        List<Spline> splines. = reticulateSplines(gradient);
        diveForMoog(Splines, reason);
    }
}
```

위 코드는 함수의 인자를 일종의 연결 소자로써 사용하여 시간적인 결합을 노출한다. 

### _G32: 일관성을 유지하라_
- **코드 구조를 잡을 때는 이유를 고민하라. 그리고 그 이유를 코드 구조로 명백히 표현하라.**
- 시스템 전반에 걸친 구조가 일관성이 있다면, 남도 일관성을 따르고 보존한다.

### _G33: 경계 조건을 캡슐화하라_
- 경계 조건은 별도로 한곳에서 처리한다.
```java
if(level + 1 < tags.length>)
{
    parts = new Parse(body, tags, level + 1, offset + endTag);
    body = null;
}
```

위의 코드에서 `level + 1` 은 여기 저기에 나온다. 이러한 경계 조건은 캡슐화하는 편이 좋다.

```java
int nextLevel = level + 1;
if(nextLevel < tags.length)
{
    parts = new Parse(body, tags, nextLevel, offset + endTag);
    body = null;
}
```
### _G34: 함수는 추상화 수준을 한 단계만 내려가야 한다_
- 함수 내 모든 문장은 추상화 수준이 동일해야 한다. 그리고 그 수준은 함수 이름이 의미하는 작업보다 한 단계 더 낮아야 한다.(작업이 더 구체적이어야 한다.)
- 함수에서 추상화 수준을 분리하면 드러나지 않았던 새로운 추상화 수준이 드러나는 경우가 빈번하다.

```java
public String render() throws Exception {
    StringBuffer html = new StringBuffer("<hr");
    if (size > 0) {
        html.append(" size=\"").append(size + 1).append("\"");
    }
    html.append(">");

    return html.toString();
}
```
위 함수에는 추상화 수준이 최소한 두 개가 섞여 있다. 첫째는 수평선에 크기가 있다는 개념이다. 둘째는 HR 태그 자체의 문법이다. 

위 코드를 다음과 같이 수정해보자. `size` 변수 이름은 목적을 반영하게 적절히 변경했다. `size` 변수는 추가된 대시 개수를 저장한다. `render`함수는 `HR` 태그만 생성하며, 구체적인 문법은 전혀 상관하지 않는다. 추상화 수준을 분리되었다.
```java
public String render() throws Exception {
    HtmlTag hr = new HtmlTag("hr");
    if (extraDashes > 0) {
        hr.addAttribute("size", hrSize(extraDashes));
    }
    return hr.html();
}

private String hrSize(int height) {
    int hrSize = height + 1;
    return String.format("%d", hrSize);
}
```

### _G35: 설정 정보는 최상위 단계에 둬라_
- 추상화 최상위 단계에 둬야 할 기본값 상수나 설정 관련 상수를 저차원 함수에 숨겨서는 안 된다.
    - **대신 고차원 함수에서 저차원 함수를 호출할 때 인수로 넘긴다.**
- 설정 정보를 최상위 단계에 둬야 찾기도, 변경하기도 쉽다.
```java
public static void main(String [] args) throws Exception
{
    Arguments arguments = parseCommandLine(args);
    ...
}

public class Arguments 
{
    // 설정 정보
    public static final String DEFAULT_APTH = ".";
    public static final String default_root = "FitNesseRoot";
    ...
}
```

### _G36: 추이적 탐색을 피하라_
- 일반적으로 한 모듈은 주변 모듈을 모를수록 좋다.
- A가 B를 사용하고, B가 C를 사용한다 해서, A가 C를 알아야 할 필요는 없다는 뜻이다.
    - 이를 디미터 법칙이라고 부른다.
- 여러 모듈에서 `a.getB().getC()` 라는 형태를 사용한다면 설계와 아키텍처를 바꿔 B와 C사이에 Q를 넣기 쉽지않다.
- 원하는 기능을 찾아 객체를 따라 시스템 전체를 탐색할 필요가 없어야 한다.

## 자바

### _J1: 긴 import 목록을 피하고 와일드 카드를 사용하라_
- 패키지에서 클래스를 둘 이상 사용한다면 와일드카드를 사용해 패키지 전체를 가져오라.
    - 명시적으로 사용하는 import 문은 강한 의존성을 생성하지만, 와일드카드는 그렇지 않다.
```java
import package.*;
```
- 와일드카드가 이름 충돌이나 모호성을 초래할 경우에는 명시적으로 import 문을 사용한다. 아니면 코드에서 클래스를 사용할 때 전체 경로를 명시한다.

### _J2: 상수는 상속하지 않는다_
- 상수를 상속하는 것은 언어의 범위 규칙을 속이는 행위다.
- 대신 `static import` 를 사용하라.

### _J3: 상수 대 Enum_
- `enum`은 이름이 부여된 열거체에 속한다. 
- 메서드와 필드도 사용할 수 있다. int 보다 훨씬 더 유연하고 **서술적인** 강력한 도구다. 맘껏 사용하라.

## 이름
### N1: 서술적인 이름을 사용하라
- 소프트웨어 가독성의 90%는 이름이 결정한다. 그러니 이름을 정할 때는 시간을 들여 현명하게 선택한다.
- 신중하게 선택한 이름은 추가 설명이 필요하지 않다.
```java
private boolean isStrike(int frame) {
    return rolls[frame] == 10;
}
```
### N2: 적절한 추상화 수준에서 이름을 선택하라
- **구현을 드러내는 이름은 피하라.**
- 작업 대상 클래스나 함수가 위치하는 추상화 수준을 반영하는 이름을 선택하라.

```java
public interface Modem {
    boolean dial(String phoneNumber);
    boolean disconnect();
    boolean send(char c);
    char recv();
    String getConnectedPhoneNumber();
}
```
`Modem` 관련 인터페이스이다. 얼핏 문제 없어 보이지만 전화선에 일부 연결되지 않는 모뎀을 생각해보자. 그렇다면 전화번호라는 개념은 추상화 수준이 틀렸다. 다음과 같이 수정하면, 연결 대상은 전화로 제한되지 않는다.

```java
public interface Modem {
    boolean connect(String connectionLocator);
    boolean disconnect();
    boolean send(char c);
    char recv();
    String getConnectedPhoneNumber();
}
```

### _N3: 가능하다면 표준 명명법을 사용하라_
- 예를 들어, `DECORATOR` 패턴을 활용하는 클래스 이름에는 `Decorator`라는 단어를 사용해야 한다.
- 패턴은 한 가지 표준에 불과하다. 이러한 경우 이름은 관례에 따르는 편이 좋다.
- **프로젝트에 유효한 의미가 담긴 이름을 많이 사용할수록 독자가 코드를 이해하기 쉬워진다.**

### _N4: 명확한 이름_
- 함수나 변수의 목적을 명확히 밝히는 이름을 선택한다.
- 길어도 좋으며, 길다는 단점은 서술성이 충분히 메꿀수 있다.
### _N5: 긴 범위는 긴 이름을 사용하라_
- **이름 길이는 범위 길이에 비례해야 한다.** 범위가 작으면 아주 짧은 이름을 사용해도 괜찮다. 하지만 범위가 길어지면 긴 이름을 사용한다.
- 이름범위가 길어질 수록 이름을 정확하고 길게 짓는다.
- 범위가 5줄 안팎이라면 `i`나 `j`와 같은 변수 이름도 괜찮다. 

```java
private void rollMany(int n, int pins)
{
    for (int i=0; i<n; i++)
        g.roll(pins);
}
```

만일 `i` 대신 `rollCount`라고 썼다면 더 헷갈렸을 것이다.

### _N6: 인코딩을 피하라_
- 이름에 유형 정보나 범위 정보를 넣어서는 안된다.
- 예를 들어 오늘날 개발 환경에서, `m_` 이나 `f` 와 같은 접두어는 불필요하다.
### _N7: 이름으로 부수 효과를 설명하라_
- 이름에 부수 효과를 숨기지 않는다.

```java 
public ObjectOutprStream getOos() throws IOException {
    if (m_oos == null) {
        m_oos = new ObjectOutprStream(m_socket.getOutputStream());
    } 
    return m_oos;
}
```

위 함수는 단순히 "oos"만 가져오지 않고, 없으면 생성하기도 한다. 그러므로 `createOrReturnOos` 라는 이름이 더 적합하다.

## 테스트
### _T1: 불충분한 테스트_
- 테스트 케이스는 잠재적으로 깨질 만한 부분을 모두 테스트 해야한다.
- 테스트 케이스가 확인하지 않는 조건이나 검증하지 않는 계산이 있다면 그 테스트는 불완전하다.

### _T2: 커버리지 도구를 사용하라!_
- 커버리지 도구는 테스트가 빠뜨리는 공백을 알려준다. 
- 커버리지 도구를 사용하면 테스트가 불충분한 모듈, 클래스, 함수를 찾기가 쉬워진다.

### _T3: 사소한 테스트를 건너 뛰지 마라_
- 사소한 테스트는 짜기 쉽다. 
- 사소한 테스트가 제공하는 문서적 가치는 구현에 드는 비용을 넘어선다.

### _T4: 무시한 테스트는 모호함을 뜻한다_
- 때로는 요구사항이 불분명해 프로그램이 돌아가는 방식을 확신하기 어렵다.
    - 이러한 경우 테스트 케이스를 주석 처리하거나, `@Ignore`를 붙여 표현한다.

### _T5: 경계 조건을 테스트하라_
- 경계 조건은 각별히 신경 써서 테스트 한다.

### _T6: 버그 주변은 철저히 테스트하라_
- 버그는 서로 모이는 경향이 있다.
- 한 함수 에서 버그를 발견했다면 그 함수를 철저히 테스트하는 편이 좋다.

### _T7: 실제 패턴을 살펴라_
- 때로는 테스트 케이스가 실패하는 패턴으로 문제를 진단할 수 있다.
- 합리적인 순서로 정렬된 꼼꼼한 테스트 케이스들은 실패 패턴을 드러낸다.

### _T8: 테스트 커버리지 패턴을 살펴라_
- 통과하는 테스트나 실행하거나 실행하지 않는 코드를 살펴보면 실패하는 테스트 케이스의 실패 원인이 드러난다.

### _T9: 테스트는 빨라야 한다_
- 느린 테스트 케이스는 실행하지 않게 된다. 그러므로 테스트 케이스는 최대한 빨리 돌아가게 노력한다.

## 결론
이 장에서 소개한 휴리스틱과 냄새 목록이 완전하다 말하기는 어렵다. 하지만 완전한 목록이 목표가 아닌, 가치 체계를 피력한다.

사실상 가치 체계는 이 책의 주제이자 목표다. 일군의 규칙만 따른다고 깨끗한 코드가 얻어지지 않는다. 휴리스틱 목록을 익힌다고 소프트웨어 장인이 되지는 못한다. 장인 정신은 가치에서 나온다. 그 가치에 기반한 규율과 절제가 필요하다.
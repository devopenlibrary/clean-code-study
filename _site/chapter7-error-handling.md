# 7장 오류 처리

깨끗한 코드를 다루는 책에 오류 처리를 논하는 장이 있어 이상하게 여길지도 모르겠다. 하지만 깨끗한 코드와 오류 처리는 확실히 연관성이 있다. 

만약 오류 처리 코드로 인해 프로그램 논리를 이해하기 어려워진다면 깨끗한 코드라 부르기 어렵다.

따라서 이 장에서는 **깨끗하고 튼튼한 코드에 한걸음 더 다가가는 단계로 우아하고 고상하게 오류를 처리하는 기법과 고려 사항 몇 가지를 소개**한다.

## 오류 코드보다 예외를 사용하라
얼마 전까지만 해도 예외를 지원하지 않는 프로그래밍 언어가 많았다. 예외를 지원하지 않는 언어는 오류를 처리하고 보고하는 방법이 제한적이었다. 오류 플래그를 설정하거나 호출자에게 오류 코드를 반환하는 방법이 전부였다. 

목록 7-1은 오류를 호출자 코드에서 처리한다.

```java
// 목록 7-1 DeviceController.java
public class DeviceController {
    /*...*/
    public void sendShutDown() {
        DeviceHandle handle = getHandle(DEV1);
        // 디바이스 상태를 점검한다.
        if (handle != DeviceHandle.INVALID) {
            // 레코드 필드에 디바이스 상태를 저장한다.
            retrieveDeviceRecord(handle);
            // 디바이스가 일시정지 상태가 아니라면 종료한다.
            if (record.getStatus() != DEVICE_SUSPENDED) {
                pauseDevice(handle);
                clearDeviceWorkQueue(handle);
                closeDevice(handle);
            } else {
                // 오류 처리
                logger.log("Device suspended. Unable to shut down");
            }
        } else {
            // 오류 처리
            logger.log("Invalid handle for: " + DEV1.toString());
        }
    }
}
```

위와 같은 방법을 사용하면 호출자 코드가 복잡해진다. 함수를 호출한 즉시 오류를 확인해야 하기 때문이다. 그래서 오류가 발생하면 예외를 던지는 편이 낫다. 그러면 논리와 오류 처리 코드가 분리되어, 호출자 코드가 더 깔끔해진다.

목록 7-2는 오류를 발견하면 예외를 던지는 코드다.

```java
// 목록 7-2 DeviceController.java (예외 사용)
public class DeviceController {
    /*...*/
    public void sendShutDown() {
        try {
            // 논리 처리
            tryToShutDown();
        } catch (DeviceShutDownError e) {
            // 오류 처리
            logger.log(e);
        }
    }

    private void tryToShutDown() throws DeviceShutDownError {
        DeviceHandle handle = getHandle(DEV1);
        DeviceRecord record = retrieveDeviceRecord(handle);
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
    }

    private DeviceHandle getHandle(DeviceID id) {
        /*...*/
        throw new DeviceShutDownError("Invalid handle for: " + id.toString());
        /*...*/
    }
}
```

코드가 확실히 깨끗해졌고, 품질도 나아졌다. 앞서 뒤섞였던 즉 디바이스를 종료하는 알고리즘과 오류를 처리하는 알고리즘을 분리했다. 이제는 각 개념을 독립적으로 살펴보고 이해할 수 있다.

## Try-Catch-Finally 문부터 작성하라
`try-catch-finally` 문에서 `try` 블록에 들어가는 코드를 실행하면 어느 시점에서든 실행이 중단된 후 `catch` 블록으로 넘어갈 수 있다.

어떤 면에서 `try` 블록은 트랜잭션과 비슷하다. `try` 블록에서 무슨 일이 생기든 지 **`catch` 블록은 프로그램 상태를 일관성 있게 유지해야 한다.** 그러므로 예외가 발생할 코드를 짤 때는 `try-catch-finally` 문으로 시작하는 편이 낫다. 그러면 `try` 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다.

예제를 살펴보자. 최종적으로파일을 열어 직렬화된 객체 몇 개를 읽어 들이는 코드가 필요하다.


다음은 파일이 없으면 예외를 던지는지 알아보는 단위 테스트다.

```java
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
    sectionStore.retrieveSection("invalid - file");
}
```

위의 단위 테스트에 맞춰 다음 코드를 구현했다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
    // 실제로 구현할 때까지 비어 있는 더미를 반환한다.
    return new ArrayList<RecordedGrip>();
}
```

그런데 코드는 예외를 던지지 않으므로 단위 테스트 또한 실패한다. 예외를 던지도록 `try` 블록과 `catch` 블록을 추가해보자.

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
    } catch (Exception e) {
        throw new StorageException("retrieval error", e);
    }
    return new ArrayList<RecordedGrip>();
}
```

이제 코드는 예외를 던지므로 단위 테스트 또한 성공한다. 이 시점에서 리팩터링이 가능하다. `catch` 블록에서 예외 유형을 `Exception` 에서 `FileNotFoundException`으로 좁혀 `FileInputStream` 생성자가 던지는 `FileNotFoundException`을 잡아낸다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
        stream.close();
    } catch (FileNotFoundException e) {
        throw new StorageException("retrieval error", e);
    }
    return new ArrayList<RecordedGrip>();
}
```

이처럼 먼저 강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방법을 권장한다. 그러면 자연스럽게 `try` 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워진다.

## 미확인(unchecked)예외를 사용하라
자바 첫 버전이 확인된 예외를 선보였던 당시 확인된 예외는 멋진 아이디어로 여겨졌다.

메서드를 선언할 때는 메서드가 반환할 예외를 모두 열거했다. 게다가 메서드가 반환하는 예외는 메서드 유형의 일부였다. 코드가 메서드를 사용하는 방식이 메서드 선언과 일치하지 않으면 아예 컴파일도 못했다.

확인된 예외는 여러 장점을 제공하지만, 지금은 안정적인 소프트웨어를 제작하는 요소로 확인된 예외가 반드시 필요하지 않다는 사실이 분명해졌다. 그러므로 우리는 확인된 오류가 치르는 비용 대비 그에 상응하는 이익을 제공하는지 따져봐야한다.

**확인된 예외는 OCP("소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다"는 원칙)를 위반**한다. 어떤 메서드에서 확인된 예외를 던졌는데 `catch` 블록이 세 단계 위에 있다면 그 사이 모든 메서드 선언부에 해당 예외를 정의해야한다. **즉, 하위 단계에서 코드 변경이 일어나면 상위 단계를 모두 고쳐야한다는 말이다.**

이러한 상황이 대규모 시스템에서 어떻게 작용하는지 상상해보자. 최상위 함수부터 최하위 함수까지 단계를 내려갈수록 호출하는 함수 수는 늘어날 것이다. 

만일 최하위 함수에서 확인된 예외를 던진다고 가정해보자. 그렇다면 해당 함수의 선언부에는 `throw` 절이 추가될 것이다. 그렇다면 이제 **해당 최하위 함수를 호출하는 함수와 그 함수를 호출하는 함수 ... 최상위 함수까지 함수 모두가 1) catch 블록에서 새로운 예외를 처리하거나 2) 선언부에 throw 절을 추가해야한다.** 최하위 단계의 확인된 예외를 처리하기 위해 이 모든 연쇄 수정이 일어나는 것이다. 당연히 모든 함수가 최하위 함수에서 던지는 예외를 알아야 하므로 **캡슐화 또한 깨지게 된다**.

## 예외에 의미를 제공하라
예외를 던질 때는 전후 상황을 충분히 덧붙인다. 그러면 오류가 발생한 원인과 위치를 찾기가 쉬워진다. 자바는 모든 예외에 호출 스택을 제공하긴 하지만, 이것으로 실제 의도를 파악하기엔 부족하다.

오류 메시지에 정보를 담아 예외와 함께 던진다. 실패한 연산 이름과 실패 유형도 언급한다. 애플리케이션이 로깅 기능을 사용한다면 `catch` 블록에서 오류를 기록한다.

## 호출자를 고려해 예외 클래스를 정의하라
오류를 분류하는 방법은 수없이 많을 수 있다. 애플리케이션에서 오류를 정의할 때 가장 중요한 관심사는 **오류를 잡아내는 방법**이 되어야한다.

다음은 오류를 형편없이 분류한 사례다. 외부 라이브러리를 호출하는데, 라이브러리가 던질 예외를 모두 잡아낸다.

```java
ACMEPort port = new ACMEPort(12);
try {
    port.open();
} catch (DeviceResponseException e) {
    reportPortError(e);
    logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
    reportPortError(e);
    logger.log("Unlock exception", e);
} catch (GMXError e) {
    reportPortError(e);
    logger.log("device response exception");
} finally {
    /*...*/
}
```

대다수의 상황에서 오류를 처리하는 방식은 다음과 같이 비교적 일정하다.

1) 오류를 기록한다.
2) 프로그램을 계속 수행해도 좋은지 확인한다.

위 경우는 예외 유형과 무관하게 거의 동일하게 예외에 대응하는 방식이다. 그래서 코드를 간결하게 고치기 쉽다. 예외를 처리하는 역할을 하는 클래스로 외부 라이브러리를 감싸, 예외 유형 하나만 반환하게 한다.

```java
LocalPort port = new LocalPort(12);
try {
    port.open();
} catch (PortDeviceFailure e) {
    reportError(e);
    logger.log(e.getMessage(), e);
} finally {
    /*...*/
}
```

여기서 `LocalPort` 클래스는 단순히 `ACMEPort` 클래스가 던지는 예외를 잡아변환하는 wrapper 클래스일 뿐이다.

```java
public class LocalPort {
    private ACMEPort innerPort;
    
    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }
    
    public void open() {
        try {
            innerPort.open();
            // 예외 처리
        } catch (DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        } catch (ATM1212UnlockedException e) {
            throw new PortDeviceFailure(e);
        } catch (GMXError e) {
            throw new PortDeviceFailure(e);
        }
    }
}
```

**이러한 wrapper 클래스는 매우 유용하다.** 실제로 외부 라이브러리를 사용할 때는 감싸기 기법이 최선이다. 외부 라이브러리와 프로그램의 의존성이 크게 줄어들기 때문이다. 또한 테스트하기 쉬워지는데, 외부 라이브러리르 직접 호출하는 대신 해당 wrapper 클래스에 테스트 코드를 넣어주면 되기 때문이다.

흔히 예외 클래스가 하나만 있어도 충분한 코드가 많다. 한 예외는 잡아내고 다른 예외는 무시해도 괜찮은 경우라면 여러 예외 클래스를 사용한다.

## 정상 흐름을 정의하라
앞선 지침을 따른다면 비즈니스 논리와 오류 처리가 잘 분리된 깨끗한 코드가 나온다. 

하지만 그러다 보면 오류 감지가 프로그램 언저리로 밀려난다. 외부 라이브러리를 감싸 독자적인 예외를 던지고, 코드 위에 처리기를 정의해 중단된 계산을 처리한다. 대개는 멋진 방법이지만, 어떨 때는 중단이 적합하지 않은 때도 있다.

다음 예제는 비용 청구 애플케이션에서 총계를 계산하는 허술한 코드이다. 해당 논리는 식비를 비용으로 청구 했다면 직원이 청구한 식비를 총계에 더하고, 청구하지 않았다면 일일 기본 식비를 총계에 더한다.

```java
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    // 비용 청구 시
    m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
    // 청구하지 않았을 시, 일일 기본 식비
    m_total += getMealPerDiem();
}
```
하지만 예외 처리가 논리를 따라가기 더 어렵게 만든다. 예외가 발생하지 않게, 즉 특수한 상황을 처리할 필요가 없게 만든다면 더 좋지 않을 까?

다음 예제를 살펴보자. `expenseReportDAO`를 고쳐 언제나 `MealExpenses` 객체를 반환토록 한다. 청구한 식비가 없다면 일일 기본 식비를 반환하는 `MealExpenses` 객체를 반환하면 된다.

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```

```java
// 청구한 식비가 없을 때 반환될 객체 (특수 사례 객체)
public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
        // 기본값으로 일일 기본 식비를 반환한다.
    }
}
```

위와 같은 코드를 특수 사례 패턴이라 부른다. 

> 특수 사례 패턴(Special Case Pattern)은 예외적인 상황을 객체로 캡슐화하여 처리하는 디자인 패턴이다. 이 패턴은 null을 반환하거나 예외를 던지는 대신 특수한 상황을 나타내는 객체를 반환함으로써 클라이언트 코드를 단순화한다.

그러면 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다. 클래스나 객체가 예외적인 상황을 캡슐화해서 처리하기 때문이다.

## null을 반환하지 마라
오류 처리를 논하는 장이라면 우리가 흔히 저지르는 바람에 오류를 유발하는 행위도 언급해야 한다고 생각한다.

그 중 첫째가 `null`을 반환하는 습관이다. 다음은 한 예제이다.

```java
public void registerItem(Item item) {
    if (item != null) {
        ItemRegistry registry = persistentStore.getItemRegistry();
        if (registry != null) {
            Item existing = registry.getItem(item.getID());
            if (existing.getBillingPeriod().hasRetailOwner()) {
                existing.register(item);
            }
        }
    }
}
```

위 코드는 한줄 건너 하나씩 `null`을 체크하고있다. 이는 나쁜 코드이다. 누구 하나라도 `null` 확인을 빼먹는다면 애플리케이션이 통제 불능에 빠질지도 모른다. 

위 코드에서 문제가 발생했다면 `null` 확인이 누락된 문제라 말하기 쉽다. 하지만 실상은 null 확인이 **너무 많아** 문제다. **메서드에서 null 을 반환하고 싶은 유혹이 든다면 그 대신 예외를 던지거나 특수 사례 객체를 반환한다.** 많은 경우에 특수 사례 객체가 손쉬운 해결책이다. 

다음 예제를 살펴보자. `getEmployees()` 는 `null`을 반환한다.

```java
List<Employee> employees = getEmployees();
if (employees != null) {
    for (Employee e : employees) {
        totalPay += e.getPay();
    }
}
```

하지만 `getEmployees()`가 반드시 `null`을 반환할 필요가 있을까? 만약 데이터가 없을 때 빈 리스트를 반환하도록 변경한다면 코드는 깔끔해진다.

```java
List<Employee> employees = getEmployees();
for (Employee e : employees) {
    totalPay += e.getPay();
}
```

## null을 전달하지 마라
메서드로 `null`을 전달하는 방식은 반환하는 것보다 훨씬 나쁘다. 예제를 살펴보면 이유가 드러난다. 다음은 두 지점 사이의 거리를 계산하는 간단한 메서드다.

```java
public class MetricsCalculator {
    public double Projection(Point p1, Point p2) {
        return (p2.x - p1.x) * 1.5;
    }
}
```

누군가 인수로 `null`을 전달하면 어떤 일이 벌어질까? 당연히 `NullPointerException`이 발생한다. 

하나의 방법으로 다음과 같이 새로운 예외 유형을 만들어 던질수 있다.

```java
public class MetricsCalculator {
    public double Projection(Point p1, Point p2) {
        if (p1 == null || p2 == null) {
            // 새로운 예외 유형
            throw new InvalidArgumentException(
                "Invalid argument for MetricsCalculator.xProjection");
        }
        return (p2.x - p1.x) * 1.5;
    }
}
```
위 코드는 `NullPointerException`을 발생시키는 기존 코드보다 나을지 몰라도, `InvalidArgumentException`을 잡아내는 처리기가 필요하다.

다음은 또다른 방법이다. `assert`문을 사용한다.

> `assert`문은 프로그램의 논리적 오류를 감지하고 디버깅하기 위한 Java의 도구이다.

```java
public class MetricsCalculator {
    public double Projection(Point p1, Point p2) {
        assert p1 != null : "p1 should not be null";
        assert p2 != null : "p2 should not be null";
        return (p2.x - p1.x) * 1.5;
    }
}
```

이는 문서화가 잘 되어 코드를 읽기는 편하지만 본질적인 문제를 해결하지는 못한다. 여전히 누군가 `null`을 인수로 넘긴다면 실행 오류가 발생한다.

대다수 프로그래밍 언어는 호출자가 실수로 넘기는 null을 적절히 처리하는 방법이 없다. **즉, 인수로 null이 넘어오면 코드에 문제가 있다는 말이다.**

## 결론
깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야한다. 오류 처리를 프로그램 논리와 분리해 독자적인 사안으로 고려하면 튼튼하고 깨끗한 코드를 작성할 수 있다. **오류 처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아진다.**
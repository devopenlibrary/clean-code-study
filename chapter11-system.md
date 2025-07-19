---
nav_order: 11
---

# 11장 시스템
> _"복잡성은 죽음이다. 개발자에게서 생기를 앗아가며, 제품을 걔획하고 제작하고 테스트하기 어렵게 만든다"_ - 레이 오지, 마이크로소프트 CTO

## 도시를 세운다면?
만약 나 혼자서 도시를 세운다고 생각해보자. 도시에서 돌아가는 모든 세세한 일을 다 관리할 수는 없을 것이다. 

흔히 소프트웨어 팀도 도시처럼 구성한다. 도시(소프트웨어 팀)는 수도 관리팀, 전력관리팀, 교통 관리 팀 등 각 분야를 관리하는 팀으로 돌아간다.

또한, 도시가 돌아가는 또다른 이유는 적절한 추상화와 모듈화 때문이다. 그래서 큰 그림을 이해하지 못할지라도 개인개인이 관리하는 '구성요소'는 효율적으로 돌아간다.

깨끗한 코드를 구현하면 낮은 추상화 수준에서 관심사를 분리하기 쉬워진다. 이 장에서는 **높은 추상화 수준, 즉 시스템 수준에서도 깨끗함을 유지하는 방법**을 살펴본다.


## 시스템 제작과 시스템 사용을 분리하라
우선 제작(construction)과 사용(use)은 아주 다르다는 사실을 명심한다. 

```text
예를 들어, 호텔을 새로 짓는다고 가정하자. 콘크리트 상자에 기중기와 승강기가 부착되어있고, 안전모에 작업복을 착용한 사람들이 바쁘게 움직인다. 1년 정도면 호텔은 완공된다. 

호텔이 완공되면 기중기와 승강기는 사라진다. 건물은 유리벽과 예쁜 색상의 벽으로 말끔히 꾸며지고, 호텔에 근무하고 채류하는 사람들은 호텔을 지을때 근무하던 사람들과 아주 다른 차림새를 하고 있을 것이다.
```

**소프트웨이 시스템은 (애플리케이션 객체를 제작하고 의존성을 서로 '연결'하는) 준비 과정과 (준비 과정 이후에 이이지는) 런타임 로직을 분리해야 한다.**

시작 단계는 모든 애플리케이션이 풀어야 할 **관심사**다. 이것이 이 장에서 우리가 맨 처음 살펴볼 **관심사**다. **관심사 분리는 우리 분야에서 가장 오래되고 가장 중요한 설계 기법 중 하나**다.

다음은 시작 단계 코드를 주먹구구식으로 구현할 뿐만 아니라 린타임 로직과 마구 뒤섞어 놓은 전형적인 예다.
```java
public Service getService() {
    if (service = null)
        service = new MyServiceImpl(... ) ; // 모든 상황에 적합한 기본값일까?
    return service;
}
```

이것은 초기화 지연(Lazy initializaton) 혹은 계산 지연(Lazy Evaluaton)이라는 기법이다.


> 초기화 지연은 객체나 데이터, 또는 계산 비용이 큰 리소스의 초기화를 그것이 실제로 필요할 때까지 미루는 프로그래밍 기법이다. 즉, 객체나 데이터를 미리 생성하지 않고, 실제로 접근하거나 사용할 때까지 초기화를 연기한다

우선, 실제로 필요할 때까지 객체를 생성하지 않으므로 불필요한 부하가 걸리지 않아, 애플리케이션을 시작하는 시간이 그만큼 빨라진다. 둘째, 어떤 경우에도 Null 포인터를 반환하지 않는다. 이러한 장점이 있다.

하지만 `getService` 메서드가 `MyServicelmpl`과 (위에서는 생략한) 생성자 인수에 명시적으로 의존한다. 따라서, **런타임 로직에서 `MyServicelmpl`객체를 전혀 사용 하지 않더라도 의존성을 해결하지 않으면 컴파일이 안 된다.**

테스트도 문제인데, `MyServicelmpl`이 무거운 객체라면 단위 테스트에서 `getService` 메서드를 호출하기 전에 **적절한 테스트 전용 객체**를 `service` 필드에 할당해야 한다. 

또한 `Service`가 null인 경로/null이 아닌 경로 등 모든 실행 경로도 테스트해야 한다. 즉, 메서드가 두 가지 이상 일을 수행하며, **작게나마 단일 책임 원칙(SRP)을 깬다는 말이다.**

하지만 무엇보다 우려인점은 **`MyServicelmpl`이 모든 상황에 적합한 객체인지 모든다는 사실이다.** 현실적으로 한 객체 유형이 모든 문맥에 적합할 가능성은 희박하다.

초기화 지연 기법을 한 번 정도 사용한다면 별로 심각한 문제가 아니다. 하지만 이러한 설정 방식이 애플리케이션 전반에 흩어져있다면, 모듈성은 저조하며 대개 중복이 심각해진다.

체계적이고 탄탄한 시스템을 만들고 싶다면 모듈성을 깨서는 절대로 안 된다. 객체를 생성하거나 의존성을 연결할 때도 마찬가지다. **설정 논리는 일반 실행 논리와 분리해야 모듈성이 높아진다.**

또한 **주요 의존성을 해소하기 위한 방식, 즉 전반적이며 일관적인 방식도 필요하다.**

### _Main 분리_

시스템 생성과 시스템 사용을 분리하는 한 가지 방법으로, **생성과 관련한 코드는 모두 `main`이나 `main`이 호출하는 모듈로 옮기고**, 나머지 시스템은 모든 객체가 생성되었고 모든 의존성이 연결되었다고 가정한다.

![그림 11.1 main()에서 생성 분리](/images/chapter11/main-example1.png)

흐름은 다음과 같다. 

1. `main`에서 시스템에 필요한 객체들을 모두 생성한 후 이를 애플리케이션에 넘긴다. 
2. 애플리케이션은 객체를 사용한다.

main과 애플리케이션 사이에 표시된 의존성 화살표를 보면, 모든 화살표는 main에서 애플리케이션 방향으로 향한다. 

**즉, 애플리케이션은 main이나 객체가 생성되는 과정을 모른다. 하지만, 모든 객체가 적절히 생성되었다고 가정하며 객체들을 사용할 수 있다.**

### _팩토리_

물론 때로는 **객체가 생성**되는 **시점**을 애플리케이션이 결정할 필요도 생긴다.

예를 들어, 주문처리 시스템에서 애플리케이션은 `LineItem` 인스턴스를 생성해 `Order`에 추가한다. 

이런 경우에는 추상 팩토리 패턴을 사용한다. 그러면 `LineItem`을 생성하는 시점은 애플리케이션이 결정하지만, `LineItem`을 생성하는 코드는 애플리케이션이 모른다. (그림 11-2 참조)

![그림 11.1 팩토리로 생성 분리](/images/chapter11/main-example2.png)

여기서도 마찬가지로 모든 의존성이 `main`에서 `OrderProcessing` (애플리케이션)으로 향한다. 즉, `OrderProcessing`은 `LineItem`이 생성되는 구체적인 방법을 모른다. 

그 방법은 `main` 의 `LineItemFactoryImplementation`이 안다. 
그럼에도 `OrderProcessing` 애플리케이션은 `LineItem` 인스턴스가 생성되는 시점을 완벽하게 통제할 수 있다.

### _의존성 주입_

사용과 제작을 분리하는 강력한 메커니즘 하나가 **의존성 주입(Dependency Injection, DI)** 이다. 의존성 주입은 **제어 역전(Inversion of Control, IoC) 기법을 의존성 관리에 적용한 메커니즘**이다. 

> 제어 역전은 프로그램의 제어 흐름을 개발자가 아닌 외부 환경(e.g. 프레임워크, 컨테이너)에 위임하는 소프트웨어 설계 원칙이다.

제어 역전에서는 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 떠넘긴다. 새로운 객체는 넘겨받은 책임만 맡으므로 단일 책임 원칙을 따른다. 

의존성 관리 맥락에서, 객체는 스스로 인스턴스로 만드는 책임은 지지않고, 이러한 책임을 다른 **전담 메커니즘에 넘겨야만** 한다. 그렇게 함으로써 제어를 역전한다. **초기 설정은 시스템 전체에서 필요하므로 대개 '책임질' 메커니즘으로 `main` 루틴이나 특수 컨테이너를 사용한다.**

다음 예제 코드를 살펴보자. JNDI 검색은 의존성 주입을 '부분적으로' 구현한 기능이다. 객체를 직접 생성하지 않고 외부 저장소(JNDI 디렉터리)에서 가져온다. 

```java
MyService myService = (MyService)(jndiContext.lookup("NameOfMyService"));
```

`myService`를 호출하는 클래스는 실제로 반환되는 객체의 유형을 제어하지 않는다. 대신 이 클래스는 의존성을 능동적으로 해결한다. (e.g. 수동적은 프레임워크가 해결)

하지만, 의존성 주입이 진정하게 이루어지려면 **클래스는 의존성을 해결하려 시도하지 않는다.** 대신에 의존성을 주입하는 방법으로 설정자(Setter) 메서드나 생성자 인수를 (혹은 둘 다를) 제공한다. 

예를 들어, DI 컨테이너가 있다. DI 컨테이너는 요청이 들어오면 필요한 객체의 인스턴스를 만든 후 생성자 인수나 설정자 메서드를 사용해 의존성을 설정한다. 실제로 생성되는 객체 유형은 설정 파일에서 지정하거나 특수 생성 모듈에서 코드로 명시한다. 

> _스프링 프레임워크_ 는 가장 널리 알려진 자바 DI 컨테이너를 제공한다. 객체 사이 의존성은 XML 파일에 정의한다. 그리고 자바 코드에서는 이름으로 특정한 객체를 요청한다. 

## 확장 (스케일링)

'처음부터 올바르게' 시스템을 만들 수 있다는 믿음은 미신이다. 

대신에 우리는 오늘 주어진 사용자 스토리에 맞춰 시스템을 구현해야 한다. 내일은 새로운 스토리에 맞춰 시스템을 조정하고 확장하면 된다. TDD, 리팩토터링, 깨끗한 코드는 코드 수준에서 시스템을 조정하고 확장하기 쉽게 만든다.

**하지만 시스템 수준에서는 어떨까?** 현실적으로 단순한 아키텍처를 복잡한 아키텍처로 조금씩 키울 수 없다.

_소프트웨어 시스템은 물리적인 시스템과 다르다. 관심사를 적절히 분리해 관리한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있다._

다음은 관심사를 적절히 분리하지 못해 유기적인 성장이 어려웠던 아키텍쳐인 `EJB1`, `EJB2` 이다.

목룍 11-1은 클라이언트가 사용할 프로세스 내 지역 인터페이스이다. 목록 11-1 에서 열거하는 속성은 Bank 주소, 은행이 소유하는 계좌다. 

```java
// 목록 11-1 Bank EJB용 EJB2 지역 인터페이스
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

목록 11-2는 목록 11-1 인터페이스에 상응하는 `EJB2` 빈(엔티티 빈)이다. 

> 엔티티 빈은 관계형 자료, 즉 테이블 행을 표현하는 객체로, 메모리에 상주한다.

```java
// 목록 11-2 상응하는 EJB2 엔티티 빈 구현
import java.util.Collections;
import javax.ejb.*;

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

**비즈니스 논리**는 `EJB2` 애플리케이션 **'컨테이너'에 강하게 결합된다**. 클래스를 생성하려면, **컨테이너에서 파생해야 하며, 컨테이너가 요구하는 다양한 생명주기 메서드도 제공**해야 한다.

이렇듯 **비즈니스 논리가 덩치 큰 컨테이너와 밀접하게 결합**되면, **독자적인 단위 테스트가 어렵다.** 컨테이너를 목킹하거나, EJB와 테스트를 실제 서버에 배치해야 한다.

객체 지향 프로그래밍의 개념 또한 흔들린다. **빈은 다른 빈을 상속 받지 못한다.** 일반적으로 EJB2 빈은 DTO를 정의한다. DTO는 사실상 자료구조이다. 즉, 동일한 정보를 저장하는 자료 유형이 두 개라는 의미다. 그래서 한 객체에서 다른 객체로 자료를 복사하는 반복적인 규격 코드가 필요하다.

### _횡단(cross-cutting) 관심사_
반면, 어떤면에서 `EJB2` 아키텍처는 관심사를 거의 완벽하게 분리한다.   

예를 들어, 원하는 트랜잭션, 보안, 일부 영속적인 동작은 소스 코드가 아닌 배치 기술자(deployment descriptors)에서 정의한다.

영속성과 같은 **관심사**는 애플리케이션의 전반에 걸쳐 객체의 경계를 넘나드는 경향이 있다. 따라서, 모든 객체가 전반적으로 동일한 방식을 이용하게 만들어야 한다.

원론적으로는 모듈화되고 캡슐화된 방식으로 영속성 방식을 구상할 수 있다. 하지만 현실적으로는 힘들다. 여기서 **횡단 관심사**라는 용어가 나온다.

사실상 **EJB 아키텍처가 영속성, 보안, 트랜잭션과 같은 관심사를 처리하는 방식은 관점 지향 프로그래밍(Aspect-Oriented Programming, AOP)를 예견**했다고도 보인다.

**AOP는 횡단 관심사에 대처해 모듈성을 확보하는 일반적인 방법론이다**. AOP에서 **관점**(aspect)은 횡단 관심사를 모듈화 한것이라고 생각하면 된다. AOP에서 모듈 구성 개념은 **"특정 관심사를 지원하려면 시스템에서 특정 지점들이 동작하는 방식을 일관성 있게 바꿔야 한다"**라고 명시한다. 

이는 영속성을 예로 들면, 프로그래머는 영속적으로 저장할 객체와 속성을 선언만하고, 그 이후 영속성 책임은 영속성 프레임워크에 위임한다. 그러면 AOP 프레임워크는 대상 코드에 영향을 미치지 않고 동작 방식을 변경한다.

자바에서 사용하는 이와 비슷한 관점 혹은 유사한 메커니즘 세 개를 살펴보자.

## 자바 프록시

자바 프록시는 개별 객체나 클래스에서 메서드 호출을 감싸는 경우에 적합하다. 하지만 JDK 에서 사용하는 동적 프록시는 인터페이스만 지원한다. 클래스 프록시를 사용하려면 바이트 코드 처리 라이브러리(CGLIBM ASM, Javassist ...)가 필요하다.

목록 11-3은 `Bank` 애플리케이션에서 JDK 프록시를 사용해 영속성을 지원하는 예제다. 프록시로 감쌀 인터페이스 `Bank` 와 비즈니스 논리를 구현하는 POJO(Plain Old Java Object)인 `BankImpl`을 정의한다.

```java
// Bank.java (패키지 이름을 감춘다)
import java.util.*;

// 은행 추상화
public interface Bank {
    Collection<Account> getAccounts();
    void setAccounts(Collection<Account> accounts);
}

// BankImpl.java
import java.util.*;

// 추상화를 위한 POJO(Plain Old Java Object) 구현
public class BankImpl implements Bank {
    private List<Account> accounts;
    
    public Collection<Account> getAccounts() {
        return accounts;
    }
    
    public void setAccounts(Collection<Account> accounts) {
        this.accounts = new ArrayList<Account>();
        for (Account account : accounts) {
            this.accounts.add(account);
        }
    }
}

// BankProxyHandler.java
import java.lang.reflect.*;
import java.util.*;

// 프록시 API가 필요한 "InvocationHandler"
public class BankProxyHandler implements InvocationHandler {
    private Bank bank;
    
    public BankProxyHandler(Bank bank) {
        this.bank = bank;
    }
    
    // InvocationHandler에 정의된 메서드
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        String methodName = method.getName();
        if (methodName.equals("getAccounts")) {
            bank.setAccounts(getAccountsFromDatabase());
            return bank.getAccounts();
        } else if (methodName.equals("setAccounts")) {
            bank.setAccounts((Collection<Account>) args[0]);
            setAccountsToDatabase(bank.getAccounts());
            return null;
        } else {
            // ...
        }
    }
    
    // 세부사항은 여기에 이어진다.
    protected Collection<Account> getAccountsFromDatabase() { ... }
    protected void setAccountsToDatabase(Collection<Account> accounts) { ... }
}

// 다른 곳에 위치하는 코드
Bank bank = (Bank) Proxy.newProxyInstance(
    Bank.class.getClassLoader(),
    new Class[] { Bank.class },
    new BankProxyHandler(new BankImpl()));
```

이는 단순한 예제지만 코드가 상당히 많으며 제법 복잡하다. 바이트 조작 라이브러리를 사용하더라도 어렵다.

**다시 말해, 프록시를 직접 구현하여 사용하면 깨끗한 코드를 작성하기는 어렵다.**

## 순수 자바 AOP 프레임워크

다행히 대부분의 프록시 코드는 도구로 자동화할 수 있으며, 순수 자바 관점을 구현하는 스프링 AOP, JBoss AOP 등과 같은 **여러 자바 프레임워크는 내부적으로 프록시를 사용한다.**

스프링은 비즈니스 논리를 POJO로 구현한다. **POJO는 순수하게 도메인에 초점을 맞춘다.**

> POJO는 Plain Old Java Object 의 줄임말로, 말 그대로 "오래된 방식의 간단한 자바 객체"를 의미한다. 특정 기술이나 프레임워크에 종속되지 않은 순수한 자바 언어로만 작성된 객체를 뜻한다.

프로그래머는 **설정 파일(XML 등)이나 API를 사용하여 필수적인 애플리케이션 기반 구조를 구현**한다. 여기에는 횡단 관심사(영속성, 트랜잭션, 보안, 캐시, 장애 조치 등)도 포함된다.

프레임워크는 사용자가 모르게 프록시나 바이트코드 라이브러리를 사용해 이를 구현한다. **이런 선언들이 요청에 따라 주요 객체를 생성하고 서로 연결하는 등 DI 컨테이너의 구체적인 동작을 제어한다.**

목록 11-4는 스프링 V2.5 **설정 파일** app.xml 일부로, 아주 전형적인 모습이다.

```xml
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
</beans>
...
```

위의 파일에서 각 빈(bean)은 마치 중첩된 러시아 인형과도 같다. 다음 그림 11.3을 살펴보자.

_그림 11.3 DECORATOR의 '러시아 인형'_
![DECORATOR의 '러시아 인형'](/images/chapter11/decorator.png)

`Bank` 도메인 객체는 DAO(자료 접근자 객체)로 프록시되었으며, DAO는 JDBC 드라이버 자료 소스로 프록시되었다.

> DAO는 비즈니스 로직과 데이터베이스 사이의 중간 계층 역할을 하며, 데이터 접근을 추상화하는 디자인 패턴이다. 쉽게 말해 애플리케이션이 데이터베이스와 소통할 때 사용하는 전용 통역사 역할을 한다.

따라서 클라이언트에서 `Bank` 객체의 `getAccounts()`를 호출할 때, 이것이 직접적으로 이루어질것이라고 믿지만 실제로는 `Bank` POJO(도메인에 초점이 맞추어진)의 기본 동작을 확장한 중첩 DECORATOR 객체 집합의 가장 외곽과 통신하는 것이다.

> DECORATOR 객체는 기존 객체에 새로운 기능을 동적으로 추가하거나 확장할 수 있게 해주는 래퍼(Wrapper) 객체를 뜻한다.

예를 들어 애플리케이션에서 DI 컨테이너에게 시스템 내 최상위 객체를 요청하려면 다음 코드가 필요하다. 해당 객체는 앞서 설명한 프로젝트 기반구조를 구현하는 설정파일(XML)에 포함되어 있다.

```java
XmlBeanFactory bf = new XmlBeanFactory(new ClassPathResource("app.dxml", getClass()));

Bank bank = (Bank) bf.getBean("bank")
```

위의 코드에서 스프링 관련 자바 코드는 거의 필요없다. **따라서 사실상 스프링과 독립적이게 된다.** 즉, `EJB2` 시스템이 지녔던 강한 결합(tight-coupling)이라는 문제가 모두 사라진다.

이렇듯 XML과 같은 설정파일에 명시된 '정책'은 겉으로 보지이지 않지만 자동으로 생성되는 프록시나 관점 논리보다는 단순하다.

목록 11-5는 `EJB3`로 Bank 객체를 다시 작성한 코드다. `EJB3`은 XML 설정 파일과 자바 5 애너테이션 기능을 사용해 횡단 관심사를 선언적으로 지원하는 스프링 모델을 따른다.

```java
// 목록 11-5 EBJ3 Bank EJB
package com.example.banking.model;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.Collection;

@Entity
@Table(name = "BANKS")
public class Bank implements java.io.Serializable {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private int id;
    
    @Embeddable // Bank의 데이터베이스 행에 '인라인으로 포함된' 객체
    public class Address {
        protected String streetAddr1;
        protected String streetAddr2;
        protected String city;
        protected String state;
        protected String zipCode;
    }
    
    @Embedded
    private Address address;
    
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER,
               mappedBy="bank")
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

원래 EJB2 코드보다 위 코드가 훨씬 더 깨끗하다. 영속성과 같은 횡단 관심사가 애너테이션과 XML 파일에 포함되어있기 때문이다. **이러한 관심사를 비즈니스 로직과 분리하였기에 EJB2에 비해 해로운 문제가 발생할 가능성이 훨씬 적다.**

## AspectJ 관점
마지막으로, 관심사를 관점으로 분리하는 가장 강력한 도구는 AspectJ 언어다.

AspectJ는 **언어 차원에서 관점을 모듈화 구성으로 지원**하는 **자바 언어 확장**이다. 

AspectJ에 대한 상세한 설명은 이 책 범위를 벗어나므로, 자세한 내용은 [AspectJ](https://eclipse.dev/aspectj/), [Colyer](https://spring.io/authors/acolyer), [Spring](https://spring.io/) 을 참조한다.

## 테스트 주도 시스템 아키텍처 구축
관점으로 관심사를 분리하는 방식은 그 위력이 막강하다.

애플리케이션에서 도메인 논리(비즈니스 논리)를 POJO로 작성할 수 있다면, 즉 **코드 수준에서 아키텍처 관심사를 분리할 수 있다면, 진정한 테스트 주도 아키텍처 구축이 가능**해진다.

BDUF(Big Design Up Front, 구현을 시작하기 전에 앞으로 벌어질 모든 상황을 설계하는 기법)를 추구할 필요가 없다.

다시말해, 우선 단순하게 분리된 아키텍처로 결과물을 빠르게 출시 후, 기반 구조를 추가하며 조금씩 화장해 나가도 괜찮다는 말이다.

그렇게 하려면 프로젝트를 시작할때 일반적인 범위, 목표, 일정, 시스템의 일반적인 구조를 생각하되 변하는 환경에 대처해 진로를 변경할 능력도 구사할 수 있어야한다.

```
최선의 시스템 구조는 각기 POJO (또는 다른) 객체로 구현되는 모듈화된 관심사 영역(도메인)으로 구성된다. 이렇게 서로 다른 영역은 해당 영역 코드에 최소한의 영향을 미치는 관점이나 유사한 도구를 사용해 통합한다. 이런 구조 역시 코드와 마찬가지로 테스트 주도 기법을 적용할 수 있다.
```

## 의사 결정을 최적화하라

모듈을 나누고 관심사를 분리하면 지엽적인 관리와 결정이 가능해진다. 아주 큰 시스템에서는 한 사람이 모든 결정을 내리기 어렵다.

따라서 가장 적합한 사람에게 책임을 맡기면 가장 좋다. 우리는 **때때로 가능한 마지막 순간까지 결정을 미루는** 방법이 최선이라는 사실을 까먹곤한다.

최대한 정보를 모아 최적의 사람이 최선의 결정을 내리기 위해, 좀 더 고민하는 것은 좋다.

```
관심사를 모듈로 분리한 POJO 시스템은 기민함을 제공한다. 이런 기민함 덕택에 최신 정보에 기반해 최선의 시점에 최적의 결정을 내리기가 쉬워진다. 또한 결정의 복잡성도 줄어든다.
```

## 명백한 가치가 있을 때 표준을 현명하게 사용하라
표준을 사용할 때 명백한 가치와 근거가 있는지 고민하고 사용해야한다.

EJB2는 단지 표준이라는 이유만으로 많은 팀이 사용했다. 가볍고 간단한 설계로 충분했을 프로젝트에서도 말이다.

이렇듯 아주 과장되게 포장된 표준에 집착하는 바람에 고객 가치가 뒷전으로 밀려난 사례들이 많이 있다.

```
표준을 사용하면 아이디어와 컴포넌트를 재사용하기 쉽고, 적절한 경험을 가진 사람을 구하기 쉬우며, 좋은 아이디어를 캡슐화하기 쉽고, 컴포넌트를 엮기 쉽다. 하지만 때로는 표준을 만드는 시간이 너무 오래 걸려 업계가 기다리지 못한다. 어떤 표준은 원래 표준을 제정한 목적을 잊어버리기도 한다.
```

## 시스템은 도메인 특화 언어가 필요하다.
도메인 특화 언어(DSL)은 간단한 스크립트 언어나 표준 언어로 구현한 API를 가리킨다. 

좋은 DSL은 **도메인 개념과 코드 사이의 의사소통 간극을 줄여준다.** 프로젝트 이해관계자와 개발자 사이의 의사소통 간극을 줄여주듯 말이다.

**따라서 DSL은 효과적으로 사용한다면 추상화 수준을 코드 관용구나 디자인 패턴 이상으로 끌어올린다.** 그래서 개발자가 적절한 추상화 수준에서 코드 의도를 표현할 수 있다.

```
도메인 특화 언어를 사용하면 고차원 정책에서 저차원 세부사항에 이르기까지 모든 추상화 수준과 모든 도메인을 POJO로 표현할 수 있다.
```

## 결론
**시스템 역시 깨끗해야한다.**

깨끗하지 못한 아키텍처는 도메인 논리를 흐린다. 도메인 논리가 흐려지면 버그가 끼어들기 쉬워지고, 이는 기민성과 생산성을 낮춘다. 이는 결국 TDD가 제공하는 장점을 무효화한다.

**모든 추상화 단계에서 의도는 명확히 표현**해야 한다. 그러려면 POJO를 작성하고, 관점 혹은 관점과 유사한 메커니즘을 사용해 **각 구현 관심사를 분리**해야한다.

시스템을 설계하든 개별 모듈을 설계하든, 실제로 **돌아가는 가장 단순한 수단**을 사용해야 한다는 사실을 명심하자.
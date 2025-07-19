---
nav_order: 9
---

# 9장 단위 테스트

TDD라는 개념이 생소하던 시절, 우리들 대다수에게 단위 테스트란 자기 프로그램이 돌아간다는 사실만 확인하는 일회성 코드에 불과했다.

하지만 10년 동안 우리 분야는 눈부신 성장을 이뤘다. 애자일과 TDD 덕택에 단위 테스트를 자동화하는 프로그래머들이 이미 많아졌으며 점점 더 늘어나는 추세다. 

하지만 우리 분야에 테스트를 추가하려고 급하게 서두르는 와중에 많은 프로그래머들이 제대로 된 테스트 케이스를 작성해야 한다는 좀 더 미묘한(그리고 더욱 중요한) 사실을 놓쳐버렸다.

## TDD 법칙 세 가지
- **첫째 법칙** : 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
- **둘째 법칙** : 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
- **셋째 법칙** : 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

위 세 가지 규칙을 따르면 개발과 테스트가 대략 30초 주기로 묶이며, 매일 수십 개, 매달 수백 개, 매년 수천 개에 달하는 테스트 케이스가 나온다.

이러한 방대한 테스트 코드는 심각한 관리 문제를 유발하기도 한다.

## 깨끗한 테스트 코드 유지하기

몇 년전, 테스트 코드의 품질에 신경 쓰지않고, "_지저분해도 빨리_"라는 주 제어하에 테스트를 진행하는 팀의 코치를 맡은 경험이 있다.

실제 코드에 변경이 생기면, 테스트 코드도 변해야 한다. 그런데 만일 **테스트 코드가 지저분하다면, 변경하기 어려워진다.** 

테스트 코드가 복잡할수록 실제 코드를 짜는 시간보다 테스트 케이스를 추가하는 시간이 더 걸리기 십상이다. 실제 코드를 변경해 기존 테스트 케이스가 실패하기 시작하면, 지저분한 테스트코드로 인해, 실패하는 테스트 케이스를 점점 더 통과시키기 어려워진다. **그래서 테스트 코드는 계속해서 늘어나는 부담이 되버린다.**

새 버전을 출시할 때마다 팀이 테스트 케이스를 유지하고 보수하는 비용도 점차 늘어난다. 이제 점차 테스트 코드는 개발자 사이에서 불만으로 자리잡는다. 따라서 테스트 슈트를 폐기하지 않으면 안 되는 상황에 처한다.

> 테스트 슈트는 테스트 케이스들의 집합으로, 특정 테스트 목표나 기능 단위로 관련된 테스트 케이스들을 그룹화한 것을 뜻한다.

하지만 테스트 슈트가 없으면 개발자는 자신이 수정한 코드가 제대로 동작하는지 확인할 방법이 없다. 그래서 결함율이 높아지고, 변경에 대해 주저하게 된다. 

결국 테스트 슈트도 없고, 코드는 뒤섞이고, 테스트에 쏟아 부은 노력이 허사였다는 실망감만 남는다.

이 이야기가 전하는 교훈은 "**테스트 코드는 실제 코드 못지 않게 중요하다**" 는 점이다.

### _테스트는 유연성, 유지보수성, 재사용성을 제공한다_

코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목이 바로 단위 테스트다. **테스트 케이스가 있으면 변경이 두렵지 않고, 변경이 쉬워진다.**

아무리 아키텍쳐가 유연하고, 설계를 잘 나눴더라도 테스트 케이스가 없으면 잠정적인 버그를 두려워해 변경을 주저하게 된다.

하지만 반대로 테스트 케이스가 있다면 그러한 주저함은 사라진다. 오히려 아키텍처가 부실하더라도 별다른 우려없이 코드를 변경할 수 있다.

그러므로 실제 코드를 점검하는 자동화된 단위 테스트 슈트는 설계와 아키텍처를 최대한 깨끗하게 보존하는 열쇠다. 

## 깨끗한 테스트 코드

깨끗한 테스트 코드를 만들기 위해 가장 중요한 것은 **가독성**이다.

그렇다면 가독성을 높이기 위해 필요한 것은 명료성, 단순성, 풍부한 표현력이다. 테스트 코드는 최소한의 표현으로 많은 것을 나타내야 한다.

목록 9-1은 FitNess에서 가져온 코드다. 세 개의 테스트 케이스는 이해하기 어렵기에 개선할 여지가 있다. 첫째, `addPage`와 `assertSubString`을 부르느라 중복된 코드가 매우 많다. 좀 더 중요하게는 자질구레한 사항이 너무 많아 테스트 코드의 표현력이 떨어진다.

```java
// 목록 9-1 SerializedPageResponderTest.java
public void testGetPageHierarchyAsXml() throws Exception {
    crawler.addPage(root, PathParser.parse("PageOne"));
    crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
    crawler.addPage(root, PathParser.parse("PageTwo"));
    
    request.setResource("root");
    request.addInput("type", "pages");
    
    Responder responder = new SerializedPageResponder();
    SimpleResponse response = 
        (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
    String xml = response.getContent();
    
    assertEquals("text/xml", response.getContentType());
    assertSubString("<name>PageOne</name>", xml);
    assertSubString("<name>PageTwo</name>", xml);
    assertSubString("<name>ChildOne</name>", xml);
}

public void testGetPageHierarchyAsXmlDoesntContainSymbolicLinks() throws Exception {
    WikiPage pageOne = crawler.addPage(root, PathParser.parse("PageOne"));
    crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
    crawler.addPage(root, PathParser.parse("PageTwo"));
    
    PageData data = pageOne.getData();
    WikiPageProperties properties = data.getProperties();
    WikiPageProperty symLinks = properties.set(SymbolicPage.PROPERTY_NAME);
    symLinks.set("SymPage", "PageTwo");
    pageOne.commit(data);
    
    request.setResource("root");
    request.addInput("type", "pages");
    
    Responder responder = new SerializedPageResponder();
    SimpleResponse response = 
        (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
    String xml = response.getContent();
    
    assertEquals("text/xml", response.getContentType());
    assertSubString("<name>PageOne</name>", xml);
    assertSubString("<name>PageTwo</name>", xml);
    assertSubString("<name>ChildOne</name>", xml);
    assertNotSubString("SymPage", xml);
}

public void testGetDataAsHtml() throws Exception {
    crawler.addPage(root, PathParser.parse("TestPageOne"), "test page");
    
    request.setResource("TestPageOne");
    request.addInput("type", "data");
    
    Responder responder = new SerializedPageResponder();
    SimpleResponse response = 
        (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
    String xml = response.getContent();
    
    assertEquals("text/xml", response.getContentType());
    assertSubString("test page", xml);
    assertSubString("<Test", xml);
}

```

예를 들어, `PathParser` 호출을 살펴보자. `PathParser는` 문자열을 `pagePath` 인스턴스로 변환한다. 이는 웹 크롤러가 사용하는 객체로 테스트에 무관하며 테스트 코드의 의도를 흐린다.

```java
crawler.addPage(root, PathParser.parse("PageOne"));
crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
crawler.addPage(root, PathParser.parse("PageTwo"));
```

`responsder`객체를 생성하는 코드와 `response`를 수집해 변환하는 코드 역시 잡음에 불과하다.
```java
Responder responder = new SerializedPageResponder();
SimpleResponse response = 
        (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
String xml = response.getContent();
```

이 코드는 읽는 사람을 고려하지 않는다. 테스트와 무관한 코드가 도처에 남발되어있어, 정작 중요한 테스트 케이스를 이해하는데 방해된다.

이제 목록 9-2를 살펴보자. 목록 9-1을 개선한 코드로, 목록 9-1과 정확히 동일한 테스트를 수행한다. 하지만 좀 더 깨끗하고 이해하기 쉽다.

```java
// 목록 9-2 SerializedPageResponderTest.java (리팩터링한 코드)
public void testGetPageHierarchyAsXml() throws Exception {
    // BUILD
    makePages("PageOne", "PageOne.ChildOne", "PageTwo");

    // OPERATE
    submitRequest("root", "type:pages");

    // CHECK
    assertResponseIsXML();
    assertResponseContains(
        "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
    );
}

public void testSymbolicLinksAreNotInXmlPageHierarchy() throws Exception {
    // BUILD
    WikiPage page = makePage("PageOne");
    makePages("PageOne.ChildOne", "PageTwo");

    // OPERATE
    addLinkTo(page, "PageTwo", "SymPage");


    submitRequest("root", "type:pages");


    // CHECK
    assertResponseIsXML();
    assertResponseContains(
        "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
    );
    assertResponseDoesNotContain("SymPage");
}

public void testGetDataAsXml() throws Exception {
    // BUILD
    makePageWithContent("TestPageOne", "test page");


    // OPERATE
    submitRequest("TestPageOne", "type:data");


    // CHECK
    assertResponseIsXML();
    assertResponseContains("test page", "<Test");
}
```

BUILD-OPERATE-CHECK 패턴이 위와 같은 테스트 구조에 적합하다.

BUILD-OPERATE-CHECK 패턴은 테스트 코드 작성 시 가독성을 높이기 위한 구조화된 접근 이다. 이 패턴은 테스트를 세 가지 명확한 단계로 나누어 진행한다.

- **BUILD 단계** : 테스트에 필요한 자료와 환경을 구성하는 단계. Given 단계와 유사한 개념으로, 입력 데이터를 준비한다.

- **OPERATE 단계** : 준비된 테스트 자료를 실제로 조작하는 단계. When 단계와 유사하게, 실제 코드를 실행하는 과정.

- **CHECK 단계** : OPERATE 단계의 결과를 검증하는 단계. 이 단계에서 assert 문을 사용하여 테스트 결과를 판단한다. Then 단계와 유사한 개념이다.

또한 잡다하고 세세한 코드를 거의 다 없앴다는 사실에 주목해야 한다. 그러므로 읽는 사람은 필요없는 코드에 헷갈릴 필요가 없고, 코드가 수행하는 기능에 집중할 수 있게 된다.

### _도메인에 특화된 테스트 언어_
목록 9-2는 도메인에 특화된 언어(DSL)로 테스트 코드를 구현하는 기법을 보여준다.

흔히 쓰는 시스템 조작 API를 사용하는 대신, API위에 함수와 유틸리티를 구현한다. 즉, 이러한 함수와 유틸리티는 당사자와 나중에 테스트를 읽을 사람을 도와주는 테스트 **언어**가 된다.

이런 테스트 API는 처음부터 설계된 API가 아니다. 계속 리팩터링한 결과이며, 숙련된 개발자라면 자기 코드를 좀 더 간결하고 표현력이 풍부한 코드로 리팩터링해야 한다.

### _이중 표준_
**테스트 API코드에 적용하는 표준과 실제 코드에 적용하는 표준은 다르다.**

단순하고, 간결하고, 표현력이 풍부해야 하지만, 실제 코드만큼 효율적인 필요는 없다. 실제 환경이 아닌 테스트 환경에서 돌아가는 코드이기 때문이다.

목록 9-3을 살펴보자. 환경 제어 시스템에 속한 테스트 코드다. 세세하게 설명하지 않더라도 온도가 "급격하게 떨어지면" 경보, 온풍기, 송풍기가 모두 가동되는지 확인하는 코드라는 사실이 드러난다.

```java
// 목록 9-3 EnvironmentControllerTest.java
@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
    hw.setTemp(WAY_TOO_COLD);
    controller.tic();
    assertTrue(hw.heaterState());
    assertTrue(hw.blowerState());
    assertFalse(hw.coolerState());
    assertFalse(hw.hiTempAlarm());
    assertTrue(hw.loTempAlarm());
}
```

목록 9-3을 읽다보면 눈길이 이리저리 흩어져, 읽기 어렵다. 그래서 목록 9-4로 변홚해 코드 가독성을 크게 높였다.

```java
// 목록 9-4 EnvironmentControllerTest.java (리팩터링)
@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
    wayTooCold();
    assertEquals("HBchL", hw.getState()) ;
}
```

목록 9-3의 `tic` 함수는 `wayTooColod`라는 함수 내부에 숨겼다. 그런데 `assertEquals`에 들어있는 이상한 문자열("HBchL")은 무엇일까? 

이는 대문자는 "켜짐"이고 소문자는 "꺼짐"임을 뜻하며, 문자의 순서는 {heater, blower, cooler, hi-temp-alarm, lo-temp-alarm} 이다.

즉 "HBchL" 이라는 문자열은 heater, blower, lo-temp-alarm 이 "켜짐" 상태이고, cooler, hi-temp-alarm 은 "꺼짐" 상태임을 나타낸다.

위의 방식이 _그릇된 정보를 피하라_(코드에 코드의 의미를 흐리는 단어를 남기는)는 규칙의 위반에 가깝지만, 여기서는 적절해보인다. 앞서 말했듯, 테스트 코드와 실제 코드의 환경은 다르다. 일단 의미만 안다면, 코드를 이하는 것은 훨씬 쉬워질 것이다. 이는 목록 9-5를 보면 알수 있다.

```java
// 목록 9-5 EnvironmentControllerTest.java (더 복잡한 선택)
@Test
public void turnOnCoolerAndBlowerIfTooHot() throws Exception {
    tooHot();
    assertEquals("hBChl", hw.getState());
}

@Test
public void turnOnHeaterAndBlowerIfTooCold() throws Exception {
    tooCold();
    assertEquals("HBchl", hw.getState());
}

@Test
public void turnOnHiTempAlarmAtThreshold() throws Exception {
    wayTooHot();
    assertEquals("hBCHL", hw.getState());
}

@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
    wayTooCold();
    assertEquals("HBchL", hw.getState());
}
```

목록 9-6은 `getState`함수를 보여준다. 코드가 그리 효율적이지 못하며, 효율을 높이려면 `StringBuffer`가 더 적합하다. 하지만, 실제로 `StringBuffuer`를 사용하지 않아 치르는 대가는 미미하다.



이는 테스트 환경과 실제 애플리케이션 환경은 다름을 시사한다. **이것이 이중 표준의 본질이다. 실제 환경에서는 절대로 안 되지만 테스트 환경에서는 전혀 문제없는 방식이 있다.** 대게 메모리나 CPU 효율과 관련 있는 경우다. **코드의 깨끗함과는 철저히 무관하다.**

## 테스트 당 assert 하나

assert 문이 단 하나인 함수는 결론이 하나라서 코드를 이해하기 쉽고 빠르다.

하지만 목록 9-2는 어떨까? "출력이 XML이다"라는 assert 문과 "특정 문자열을 포함한다"는 assert 문을 하나로 병합하는 방식이 불합리해 보인다. 하지만 목록 9-7에서 보듯이 테스트를 두 개로 쪼개 각자가 assert를 수행하게 된다.

```java
// 목록 9-7 SerializedPageResponderTest.java (단일 assert)
public void testGetPageHierarchyAsXml() throws Exception {
    givenPages("PageOne", "PageOne.ChildOne", "PageTwo");


    whenRequestIsIssued("root", "type:pages");


    thenResponseShouldBeXML();
}

public void testGetPageHierarchyHasRightTags() throws Exception {
    givenPages("PageOne", "PageOne.ChildOne", "PageTwo");


    whenRequestIsIssued("root", "type:pages");


    thenResponseShouldContain(
        "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
    );
}
```

위에서 함수 이름을 바꿔 given-when-then이라는 관례를 사용했다는 사실에 주목한다. 그러면 테스트 코드를 읽기가 쉬워진다. 하지만 불행하게도, 위에서 보듯이, 테스트를 분리하면 중복되는 코드가 많아진다.

TEMPLATE METHOD 패턴을 사용하면 중복을 제거할 수 있다. given/when 부분을 부모 클래스에 두고 then 부분을 자식 클래스에 두면된다.

> 템플릿 메소드(Template Method) 패턴은 알고리즘의 구조를 정의하고 일부 단계를 서브클래스에서 구현할 수 있도록 하는 행동 디자인 패턴이다.

혹은, 완전히 독자적인 테스트 클래스를 만들어 `@Before` 함수에 given.when 부분을 넣고 `@Test` 함수에 then 부분을 넣어도 된다.

하지만 모두가 배보다 배꼽이 더 크다. 결국 목록 9-2 처럼 assert 문을 여럿 사용하는 편이 좋다고 생각한다.

'단일 assert 문'이라는 규칙이 훌륭한 지침이라고 생각한다. 하지만 때로는 주저 없이 함수 하나에 여러 assert 문을 넣기도 한다. 단지 assert 문 개수를 최대한 줄여야 좋다는 생각이다.

### _테스트 당 개념 하나_
어쩌면 "_테스트 함수마다 한 개념만 테스트하라_"는 규칙이 더 낫겠다.

목록 9-8은 바람직하지 못한 테스트 함수다. 독자적인 개념 세 개를 테스트 하므로, 독자적인 테스트 함수 세 개로 쪼개야 마땅하다.

```java
/**
 * 목록 9-8
 *  addMonths() 메서드를 테스트하는 장황한 코드 
 */
public void testAddMonths() {
    SerialDate d1 = SerialDate.createInstance(31, 5, 2004);
    SerialDate d2 = SerialDate.addMonths(1, d1);
    assertEquals(30, d2.getDayOfMonth());
    assertEquals(6, d2.getMonth());
    assertEquals(2004, d2.getYY());
    
    SerialDate d3 = SerialDate.addMonths(2, d1);
    assertEquals(31, d3.getDayOfMonth());
    assertEquals(7, d3.getMonth());
    assertEquals(2004, d3.getYY());
    
    SerialDate d4 = SerialDate.addMonths(1, SerialDate.addMonths(1, d1));
    assertEquals(30, d4.getDayOfMonth());
    assertEquals(7, d4.getMonth());
    assertEquals(2004, d4.getYY());
}
```

셋으로 분리한 테스트 함수는 각각 다음 기능을 수행한다.

- (5월처럼) 31일로 끝나는 달의 마지막 날짜가 주어지는 경우

    1. (6월처럼) 30일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되어서는 안된다.
    2. 두 달을 더하면 그리고 두 번째 달이 31일로 끝나면 날짜는 31일이 되어야 한다.
- (6월처럼) 30일로 끝나는 달의 마지막 날짜가 주어지는 경우

    1. 31일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되면 안된다.

이렇게 표현하면 일반적인 규칙이 보인다. 즉, 목록 9-8의 문제는 assert 문이 여럿이라는 점이 아니라는 것이다. 한 테스트 함수에서 여러 개념을 테스트한다는 사실이 문제다.

그러므로 가장 좋은 규칙은 **"개념 당 assert 문 수를 최소로 줄여라"**와 **"테스트 함수 하나는 개념 하나만 테스트하라"**라 할 수 있겠다.

## F.I.R.S.T

깨끗한 테스트는 다음 다섯 가지 규칙을 따른다.

1. 빠르게(Fast) : **테스트는 빠른 주기로 돌려야 한다.** 자주 돌리지 않으면 문제를 찾아내거나, 코드를 정리하기 어렵다.
2. 독립적으로(Independent) : **각 테스트는 서로 의존하면 안 된다.** 한 테스트가 다음 테스트가 실행될 환경을 준비해서는 안된다. 테스트들이 서로 의존하면 결과가 서로 영향을 미쳐 원인을 파악하기 어려워 진다.
3. 반복가능하게(Repeatable) : **테스트는 어떤 환경에서도 반복 가능해야한다.** 테스트가 돌아가지 않는 환경이 하나라도 있다면 테스트가 실패한 이유를 둘러댈 변명이 생긴다.
4. 자가검증하는(Self-Validating) : **테스트는 부울 값으로 결과를 내야 한다.** 예를 들어 통과 여부를 알려고 로그 파일을 읽게 해서는 안된다. 테스트 스스로 성공과 실패를 가늠하지 못한다면, 판단은 주관적이 된다.
5. 적시에(Timely) : **테스트는 적시에 작성해야 한다.** 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다.

## 결론
테스트 코드는 실제 코드만큼이나 프로젝트 건강에 중요하다. 테스트 코드는 실제 코드의 유연성, 유지보수성, 재사용성을 보존하고 강화하기 때문이다. 그러므로 테스트 코드는 지속적으로 깨끗하게 관리하자. 

표현력을 높이고 간결하게 정리하자. 테스트 API를 따로 구현해 도메인 특화 언어로 만들자. 

테스트 코드가 방치되어 망가지면 실제 코드도 망가진다. 테스트 코드를 깨끗하게 유지하자.
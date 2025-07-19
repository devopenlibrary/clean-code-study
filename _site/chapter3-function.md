# 3장 함수

프로그래밍 초창기에는 시스템을 루틴과 하위 루틴으로 나눴다. 포트란과 PL/1 시절에는 시스템을 프로그램, 하위 프로그램, 함수로 나눴다. 지금은 함수만 살아남아, 어떤 프로그램이든 가장 기본적인 단위가 함수이게 되었다. 이 장은 **함수를 만드는 법**을 소개한다.


다음의 코드는 FitNesse 라고하는 오픈 소스 테스트 도구를 구성하는 함수이다. 다음 함수를 2번의 리팩터링을 거친 함수들을 보며, 함수를 깨끗하고 잘 만드는 법에 대해 배워보겠다. 

FitNesse 에 익숙하지 않아 코드를 100% 이해하기 어려운 것을 감안하더라도, 다음 함수는 무슨 역할을 하는 지 알기 어렵다. 이유는 다음과 같다.

1. 추상화 수준이 다양하다.
2. 코드가 길다. 
3. if 문이 중첩되어 가독성이 좋지않다.
4. 각 변수(플래그, 문자열, 함수)의 의미가 불명확하다.
5. 하나의 함수에서 하는 일이 많다.

## 더러운 함수 (목록 3-1 함수)
```java
// 목록 3-1 HtmlUtil.java
public static String testableHtml(PageData pageData, boolean includeSuiteSetup) throws Exception {
    WikiPage wikiPage = pageData.getWikiPage();
    StringBuffer buffer = new StringBuffer();
    if (pageData.hasAttribute("Test")) {
        if (includeSuiteSetup) {
            WikiPage suiteSetup = PageCrawlerImpl.getInheritedPage(
                SuiteResponder.SUITE_SETUP_NAME, wikiPage
            );
            if (suiteSetup != null) {
                WikiPagePath pagePath = suiteSetup.getPageCrawler().getFullPath(suiteSetup);
                String pagePathName = PathParser.render(pagePath);
                buffer.append("!include -setup.")
                      .append(pagePathName)
                      .append("\n");
            }
        }
        WikiPage setup = PageCrawlerImpl.getInheritedPage("SetUp", wikiPage);
        if (setup != null) {
            WikiPagePath setupPath = wikiPage.getPageCrawler().getFullPath(setup);
            String setupPathName = PathParser.render(setupPath);
            buffer.append("!include -setup.")
                  .append(setupPathName)
                  .append("\n");
        }
    }
    buffer.append(pageData.getContent());
    if (pageData.hasAttribute("Test")) {
        WikiPage teardown = PageCrawlerImpl.getInheritedPage("TearDown", wikiPage);
        if (teardown != null) {
            WikiPagePath tearDownPath = wikiPage.getPageCrawler().getFullPath(teardown);
            String tearDownPathName = PathParser.render(tearDownPath);
            buffer.append("\n")
                  .append("!include -teardown.")
                  .append(tearDownPathName)
                  .append("\n");
        }
        if (includeSuiteSetup) {
            WikiPage suiteTeardown = PageCrawlerImpl.getInheritedPage(
                SuiteResponder.SUITE_TEARDOWN_NAME, wikiPage
            );
            if (suiteTeardown != null) {
                WikiPagePath pagePath = suiteTeardown.getPageCrawler().getFullPath(suiteTeardown);
                String pagePathName = PathParser.render(pagePath);
                buffer.append("!include -teardown.")
                      .append(pagePathName)
                      .append("\n");
            }
        }
    }
    pageData.setContent(buffer.toString());
    return pageData.getHtml();
}

```

## refactoring 버전 (목록 3-2 함수)

```java
// 목록 3-2 HtmlUtil.java (리팩터링한 버전)
public static String renderPageWithSetupsAndTeardowns(
    PageData pageData, boolean isSuite
) throws Exception {
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

## re-refactoring 버전 (목록 3-3 함수)

```java
// 목록 3-3 HtmlUtil.java (리-리팩터링한 버전)
public static String renderPageWithSetupsAndTeardowns(
    PageData pageData, boolean isSuite) throws Exception {
    if (isTestPage(pageData))
        includeSetupAndTeardownPages (pageData, isSuite); 
    return pageData.getHtml();
}
```

# 작게 만들어라!
함수를 만드는 규칙은 다음과 같다.

1. _**'작게!'**_ 만든다.
2. _**'더 작게!'**_ 만든다.

그렇다면 얼마나 짧아야 좋을까? 목록 3-1의 코드는 총 70줄이었다. 목록 3-2로 리팩터링하며 많이 줄였지만 일반적으로 이보다도 짧아야한다. 목록 3-3 정도 줄여야 마땅하다.

## 블록과 들여쓰기
목록 3-1은 if 문/else 문이 3단으로 중첩되어있다. 코드를 이해하기 어렵게하는 요소이다.

```java
if (pageData.hasAttribute("Test")) {
        if (includeSuiteSetup) {
            WikiPage suiteSetup = PageCrawlerImpl.getInheritedPage(
                SuiteResponder.SUITE_SETUP_NAME, wikiPage
            );
            if (suiteSetup != null) {
                /* ... */
```

중첩 구조가 생길만큼 함수가 커져서는 안된다. 함수에서 들여쓰기 수준은 1단이나 2단을 넘어서면 안된다.

다시 말해, if 문/else 문/while 문 등에 들어가는 블록은 한 줄이어야 한다는 의미다. 대게 거기서 함수를 호출한다. 그러면 바깥을 감싸는 함수(enclosing function)가 작아질 뿐 아니라, 블록 안에서 호출하는 함수 이름을 적절히 짓는다면, 코드를 이해하기도 쉬워진다.

# 한 가지만 해라!
목록 3-1은 여러 가지를 처리한다. 이는 함수가 정확히 어떤 역할을 하는지 의미를 불명확하게 한다.

1. 버퍼를 생성
2. 페이지 가져오기
3. 상속된 페이지를 검색
4. 경로를 렌더링
5. HTML 생성

반면 목록 3-3은 한 가지만 처리한다. 설정 페이지와 해제 페이지를 테스트 페이지에 넣는 일을 한다.

> **함수는 한가지를 해야 한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.**

## 한 가지의 기준 
위의 충고에서 문제라면 그 '한 가지'가 무엇인지 알기 어렵다는 점이다. 목록 3-3이 진정 한가지만 하는가? 다음과 같이 세가지를 한다고 볼 수도 있다.

1. 페이지가 테스트 페이지인지 판단한다.
2. 만약 그렇다면, 설정 페이지와 해제 페이지를 넣는다.
3. 페이지를 HTML로 렌더링 한다.

이는 TO 문단으로 기술하면 다음과 같다.

> TO 는 LOGO 라는 언어에서 사용되는 함수를 설계하는 방식이다.

```text
TO RenderPageWithSetupsAndTeardowns, 페이지가 테스트 페이지인지 확인한 후 테스트 페이지면 설정 페이지와 해제 페이지를 넣는다. 테스트 페이지든 아니든 페이지를 HTML로 렌더링한다.
```

위의 TO 문단으로 기술한 작업 내용은 추상화 수준이 하나인 단계만 수행한다. 위의 의미를 유지하면서 목록 3-3을 더이상 줄이기는 힘들다. 즉 단순히 다른 표현이아니라, 완전히 다른 의미있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러작업을 하는 것이다.


### 함수 당 추상화 수준은 하나로!
함수가 확실히 '한 가지' 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야한다. 

목록 3-1은 다양한 추상화 수준에서 여러 단계를 처리한다. 

예를 들어,
1. `getHtml()`은 추상화 수준이 아주 높다. 
2. `String pagePathName = PathParser.render(pagepath);`는 추상화 수준이 중간이다.
3. `.appedn("\n")`와 같은 코드는 추상화 수준이 아주 낮다.

이처럼 한 함수 내에 추상화 수준이 섞여있으면, 코드를 읽는 사람이 헷갈린다. 특정 표현이 근본 개념인지, 아니면 세부사항인지 구분하기 어려운 탓이다.

### 위에서 아래로 코드 읽기: _내려가기_ 규칙
코드는 위에서 아래로 이야기처럼 읽혀야 좋다. 한 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 온다. 즉 이러한 프로그램은 위에서 아래로 갈수록 추상화 수준이 낮아진다. 이것을 **내려가기** 규칙이라 부른다.

다르게 표현하면, 일련의 TO 문단을 읽듯 프로그램이 읽혀야한다는 의미이다. 다음 예제를 살펴보자. 이는 앞서 보았던 함수의 추상화 수준을 설명한다.

```
T0 설정 페이지와 해제 페이지를 포함하려면, 설정 페이지를 포함하고, 테스트 페이지 내용을 포함하고, 해제 페이지를 포함한다.
    TO 설정 페이지를 포함하려면, 슈트이면 슈트 설정 페이지를 포함한 후 일반 설정 페이지를 포함한다.
        TO 슈트 설정 페이지를 포함하려면, 부모 계층에서 "SuiteSetUp" 페이지를 찾 아 include 문과 페이지 경로를 추가한다.
            TO 부모 계층을 검색하려면,...
```

위처럼 위에서 아래로 TO 문단을 읽어내려 가듯이 코드를 구현하면 추상화 수준을 일관되게 유지하기가 쉬워진다.

# Switch 문
switch 문은 작게 만들기 어렵다. 불행하게도 switch 문을 완전히 피할 방법은 없다. 하지만 각 switch 문을 저차원 클래스에 숨기고 절대로 반복하지 않는 방법은 있다. 물론 다형성을 이용한다.

목록 3-4를 살펴보자. 직원 유형에 따라 다른 값을 계산해 반환하는 함수다.

```java
// 목록 3-4 Payroll.java
public Money calculatePay(Employee e)
throws InvalidEmployeeType {
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

위 함수에는 문제가 몇가지 있다.

1. 함수가 길다. 새 직원 유형을 추가할때마다 길어진다.
2. '한 가지'작업만 수행하지 않는다.
3. SRP("한 클래스는 하나의 책임만 가져야 한다"는 원칙)를 위반한다. 코드를 변경할 이유가 여럿이기 때문이다.
4. OCP("소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다"는 원칙)를 위반한다. 새 직원 유형을 추가할 때마다 코드를 변경하기 때문이다.

하지만 무엇보다 심각한 문제라면, 위 함수와 구조가 동일한 함수가 무한정 존재할 수 있다는 사실이다. 예를 들면 다음과 같은 함수가 가능하다.

`isPayday(Employee e, Date date);`, `deliverPay(Employee e, Money pay);` 등 등

```java
// deliverPay() 예시
public Money deliverPay(Employee e, Money pay)
throws InvalidEmployeeType {
    switch (e.type) {
        case COMMISSIONED:
            return deliverCommissionedPay(e, pay);
        case HOURLY:
            return deliverHourlyPay(e, pay);
        case SALARIED:
            return deliverSalariedPay(e, pay);
        default:
            throw new InvalidEmployeeType(e.type);
    }
}
```

등 가능성은 무한하다. 이를 목록 3-5 코드가 해결한다. 팩토리는 switch 문을 사용해 적절한 `Employee` 파생 클래스(`CommissionedEmployee`, `HourlyEmployee` ...)의 인스턴스를 생성한다. `calculatePay`, `isPayday`, `deliverPay` 등과 같은 함수는 `Employee` 인터페이스를 거쳐 호출된다. 그러면 다형성으로 인해 실제 파생 클래스 의 함수가 실행된다.

```java
// 목록 3-5 Employee and Factory
public abstract class Employee {
    public abstract boolean isPayday();
    public abstract Money calculatePay();
    public abstract void deliverPay(Money pay);
}
/* ------------------------------------------- */
public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}
/* ------------------------------------------- */
public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
        switch (r.type) {
            case COMMISSIONED:
                return new CommissionedEmployee(r);
            case HOURLY:
                return new HourlyEmployee(r);
            case SALARIED:
                return new SalariedEmployee(r);
            default:
                throw new InvalidEmployeeType(r.type);
        }
    }
}

```

# 서술적인 이름을 사용하라!
좋은 이름이 주는 가치는 아무리 강조해도 지나치지 않는다. 한 가지만 하는 작은 함수에 좋은 이름을 붙인다면 아래와 같은 원칙을 달성함에 있어 이미 절반은 성공했다.

> _코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라 불러도 되겠다. - 위드_

길고 서술적인 이름이 짧고 어려운 이름보다 좋다. 함수 이름을 정할 때는 여러 단어가 쉽게 읽히는 명명법을 사용한다. 그런 다음, 여러 단어를 사용해 함수 기능을 잘 표현하는 이름을 선택한다. 목록 3-7에서 예제 함수 이름 `tetableHtml`을 `SetupTeardownIncluder.render` 로 변경했다. 함수가 하는 일을 좀 더 잘 표현하므로 훨씬 좋은 이름이다.

시간이 오래걸려도 좋으니, IDE 를 활용하는 방법도 좋다. 

이름을 붙일때는 일관성이 있어야한다. 모듈 내에서 함수 이름은 같은 문구, 명사, 동사를 사용한다. 목록 3-1,2,3 을 예시로 들면 모두 `include` 라는 단어를 사용했다.

# 함수 인수
함수에서 이상적인 인수 개수는 0게(무항)다. 다음은 1개(단항), 2개(이항) 순이다. 3개(삼항)은 되도록 피하는 편이 좋다. 4개 이상(다항)은 특별한 이유가 필요하다. 하지만 특별한 이유가 있어도 사용하면 안된다.

그 이유는, 인수는 개념을 이해하기 어렵게 만드는 요소이다. 함수에서 전달한 인수를 발견할때마다 그 의미를 해석해야한다. 혹은, 함수 이름과 인수 사이의 추상화 수준이 다를 수 있다. 

테스트 관점에서 보면 인수는 더 어렵다. 인수가 많을 수록 갖가지 인수 조합으로 함수를 검증하는 테스트 케이스는 늘어나야한다.

출력 인수는 입력 인수보다 이해하기 어렵다. 흔히 우리는 함수에다 인수로 입력을 넘기고 반환값으로 출력을 받는다는 개념에 익숙하지, 인수로 결과를 받으리라 기대하지 않기 때문이다.

> 출력 인수는 함수의 결과를 돌려받기 위한 인수이다. 객체지향 언어에서는 'this'가 출력인수 역할을 하기 때문에, 출력인수를 사용할 필요가 거의 없다.

## 많이 쓰는 단항 형식

함수에 인수 1개를 넘기는 이유로 가장 흔한 경우는 두 가지다. 

하나는 인수에 질문을 던지는 경우다. `boolean fileExists("MyFile")`이 좋은 예다. 

다른 하나는 인수를 뭔가로 변환해 결과를 반환하는 경우다. `InputStream fileOpen("MyFile")`은 `String` 형의 파일 이름을 `InputStream으로` 변환한다. 함수 이름을 지을 때는 두 경우를 분명히 구분한다. 또한 언제나 일관적인 방식으로 두 형식을 사용한다.

다소 드물게 사용하지만 그래도 아주 유용한 단항 함수 형식이 `이벤트`다. 이벤트 함수는 입력 인수만 있다. 출력 인수는 없다. 프로그램은 함수 호출을 이벤트로 해석해 입력 인수로 시스템 상태를 바꾼다. 이벤트 함수는 조심해서 사용한다. 이벤트라는 사실이 코드에 명확히 드러나야 한다. 그러므로 이름과 문맥을 주의해서 선택한다.


지금까지 설명한 경우가 아니라면 단항 함수는 가급적 피한다. 

입력 인수를 변환하는 함수라면 변환 결과는 반환값으로 돌려준다. `StringBuffer transform(StringBuffer in)`이 `void transform(StringBuffer out)`보다 낫다. `StringBuffer transform(StringBuffer in)`이 입력 인수를 그대로 돌려주는 함수라 할지라도 변환 함수 형식을 따르는 편이 좋다. 변환 함수에서 출력 인수를 사용하면 혼란을 일으킨다. 적어도 변환 형태는 유지하기 때문이다.

## 플래그 인수
함수로 부울 값을 넘기는 플래그 인수는 정말로 끔찍하다. 이는 함수가 한꺼번에 여러 가지를 처리한다는 의미이기 때문이다. (참이면 A 실행, 거짓이면 B 실행). 

다시 한번, 함수는 '한 가지'만 수행해야한다.

## 이항 함수
인수가 2개인 함수는 1개인 함수보다 더 이해하기 어렵다. 예를 들어, 첫 인수가 불필요할 수 있는 경우 이를 무시해야한다는 사실을 깨닫는 시간이 필요할 수 있다. 혹은, 2개의 인수에서 실수로 값을 넣는 경우도 발생한다. 이렇게 주춤하게 만드는 상황들은 인지적으로 거슬린다는 의미이며, 이는 문제를 일으킨다.

물론 이항 함수가 적절한 경우도 있다. `Point p = new Point(0, 0)`가 좋은 예다. 직교 좌표계 점은 일반적으로 인수 2개를 취한다.(x 좌표, y 좌표) 이는 사실상 하나의 정보를 표현하는 두 요소라고 볼 수 있다. 또한 이 두 인수 사이에는 자연적인 순서도 있다.

이항 함수가 무조건 나쁘다는 소리는 아니지만, 그만큼 위험이 따르니 가능한 단항 함수로 바꾸려고하는 노력이 필요하다. 인수로 들어가는 값을 클래스 구성원으로 만드는 것 등이 방법이 될 수 있다.

## 삼항 함수
인수가 3개인 함수는 훨씬 이해하기 어렵다. 그래서 삼항 함수를 만들때는 신중히 고려해야한다.

예를 들어 assertEquals(message, expected, actual)이라는 함수를 살펴보자. 첫 인수가 예상과 다르거나, 두번째, 세번째 인수가 예상과 달라 멈칫하고 주춤할 가능성이 커진다. 다시한번, 이러한 상황들이 문제를 일으킨다.

## 인수 객체
인수가 2-3개 필요하다면 일부를 독자적인 클래스 변수로 선언할 가능성을 짚어본다.
```java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```
객체를 생성해 인수를 줄이는 방법이 눈속임이라 여겨질지 모르지만 그렇지 않다. 위 예제에서 `x`와 `y`를 묶었듯이 변수를 묶어 넘기려면 이름을 붙여야 하므로 결국은 개념을 표현하게 된다.

## 인수 목록
때로는 인수 개수가 가변적인 함수도 필요하다. `String.format` 메서드가 좋은 예다.

`String.format("%s worked %.2f hours.", name, hours);`

위 예제처럼 가변 인수 전부를 동등하게 취급하면 `List` 형 인수 하나로 취급할 수 있다. 이런 논리로 따져보면 `String.format`은 사실상 이항 함수다. 실제로 `String.format` 선언부를 살펴보면 이항 함수라는 사실이 분명히 드러난다.

`public String format(String format, Object... args)`

가변 인수를 취하는 모든 함수에 같은 원리가 적용된다. 가변 인수를 취하는 함수는 단항, 이항, 삼항 함수로 취급할 수 있다. 

## 동사와 키워드
함수의 의도나 인수의 순서와 의도를 제대로 표현하려면 좋은 함수 이름이 필수다. 

단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다. 예를 들어, `write(name)`은 누구나 곧바로 이해한다. '이름(name)'이 무엇이든 '쓴다(write)'는 뜻이다. 좀 더 나은 이름은 `writeField(name)`이다. 그러면 '이름(name)'이 필드라는 사실이 분명히 드러난다.

함수 이름에 키워드를 추가하는 형식이다. 즉, 함수 이름에 인수 이름을 넣는다. 예를 들어, `assertEquals`보다 `assertExpectedEqualsActual(expected, actual`)`이 더 좋다. 그러면 인수 순서를 기억할 필요가 없어진다.

# 부수 효과를 일으키지 마라!
부수 효과는 함수에서 '한 가지'를 하는 것 처럼 보이지만, 실제로 남몰래 다른 일도 일으킨다. 예를 들어 예상치 못하게 클래스 변수를 수정하거나, 함수로 넘어온 인수나 시스템 전역 변수를 수정한다.

다음 함수는 부수 효과를 일으킨다. 살펴보자.

```java
// 목록 3-6 UserValidator.java
public boolean checkPassword(String userName, String password) {
    User user = UserGateway.findByName(userName);
    if (user != User.NULL) {
        String codedPhrase = user.getPhraseEncodedByPassword();
        String phrase = cryptographer.decrypt(codedPhrase, password);
        if ("Valid Password".equals(phrase)) {
            Session.initialize();
            return true;
        }
        return false;
    }
}

```

여기서, 함수가 일으키는 부수 효과는 `Session.initialize()`호출이다. `checkPassword`함수는 이름만 봐서는 세션을 초기화한다는 사실이 드러나지 않는다. 이런 부수 효과가 시간적인 결합을 초래한다. 

> 시간적 결합(Temporal Coupling)은 소프트웨어 설계에서 특정 요소들이 시간이나 순서에 따라 의존하는 관계를 의미한다. 예를 들어, "메서드 A는 항상 메서드 B보다 먼저 호출되어야 한다"와 같이 실행 순서에 제약이 있는 경우를 말한다.

즉, `checkPassword`함수는 특정 상황에서만 호출이 가능하다. 다시 말해, 세션을 초기화해도 괜찮은 경우에만 호출이 가능하다. 자칫 잘못 호출하면 의도하지 않게 세션 정보가 날아간다.

이렇듯 시간적인 결합은 혼란을 일으킨다. 특히 부수 효과로 숨겨진 경우에는 더더욱 혼란이 커진다. 만약 시간적인 결합이 필요하다면 함수 이름에 분명히 명시한다. 따라서 목록 3-6은 `checkPasswordAndInitializeSession`이라는 이름이 훨씬 좋다. 물론 이는 함수가 '한 가지'만 한다는 규칙을 위반한다.

## 출력 인수
일반적으로 우리는 인수를 함수 **입력**으로 해석한다. 예를 들어 다음 함수를 보자.

```java
appendFooter(s);
```

이 함수는 무언가에 s '를 바닥글로 첨부' 할까? 아니면 s '에 바닥글을 첨부' 할까? 즉, 인수 s 는 입력일까 출력일까? 이는 함수 선언부를 찾아보면 분명해진다.

```java
public void appendFooter(StringBuffer report)
```

인수 s가 출력 인수라는 사실을 알았지만, 함수 선언부를 보고나서야 알게 되었다. 이는 주춤하게 하며 인지적으로 거슬리는 행위이다. 피해야하는 경우이다.

객체 지향 프로그램이이 나오기 전에는 출력 인수가 불가피한 경우도 있었다. 하지만 객체 지향 언어에서는 출력 인수를 사용할 필요가 거의 없다.

## 명령과 조회를 분리하라!
함수는 뭔가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야 한다. 둘 다 하면 안된다. 객체 상태를 변경하거나 아니면 객체 정보를 반환하거나 둘 중 하나이다. 다음 함수를 살펴보자.

```java
public boolean set(String attribute, String value);
```

이 함수는 이름이 `attribute`인 속성을 조회해 값을 `value`로 설정한 후 성공하면 `true`를 반환하고 실패하면 `false`를 반환한다. 이 함수를 호출할 때 독자는 혼란이 온다.

```java
if (set("username", "unclebob")) ...
```

`username`이 `unclebob`으로 설정되어 있는지 확인하는 코드인가? 아니면 `username`을 `unclebob`으로 설정하는 코드인가? `set`이라는 단어가 동사인지 형용사인지 분간하기 어려운 탓이다.

함수를 구현한 개발자는 `set`을 **동사**로 의도했다. 하지만 if 문에 넣고 보면 **형용사**로 느껴진다. 

그래서 if 문은 "`username` 속성이 `unclebob`으로 설정되어 있다면..."으로 읽힌다. 

"`username`을 `unclebob`으로 설정하는데 성공하면..."으로 읽히지 않는다. 

`set`이라는 함수 이름을 `setAndCheckifExists`라고 바꾸는 방법도 있지만 if 문에 넣고 보면 여전히 어색하다. 진짜 해결책은 명령과 조회를 분리해 혼란을 애초에 뿌리뽑는 방법이다.

```java
if (attributeExists("username")) {
    setAttribute("username", "unclebob");
}
```
# 오류 코드보다 예외를 사용하라!
명령 함수에서 오류 코드를 반환하는 방식은 명령/조회 분리 규칙을 미묘하게 위반한다. 자칫하면 if 문에서 **명령을 표현식으로 사용**하기 쉬운 탓이다.

```java
if (deletePage(page) == E_OK)
```

위 코드는 동사/형용사 혼란을 일으키지 않는 대신 여러 단계로 중첩되는 코드를 야기할 수 있다. 해당 함수는 오류 코드를 반환하기에, 호출자는 오류 코드를 곧바로 처리해야 하기 때문이다. 다음과 같이 3단으로 중첩된 코드를 야기한다.

```java
if (deletePage(page) == E_OK) {
    if (registry.deleteReference(page.name) == E_OK) {
        if (configkeys.deleteKey(page.name.makeKey()) == E_OK) {
            logger.log("page deleted");
        } else {
            logger.log("configKey not deleted");
        }
    } else {
        logger.log("deleteReference from registry failed");
    }
} else {
    logger.log("delete failed");
    return E_ERROR;
}
```

이를 예외를 사용하면, 오류 처리 코드가 원래 코드에서 분리되므로 코드가 깔끔해진다.

```java
try {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
} catch (Exception e) {
    logger.log(e.getMessage());
}
```

# Try/Catch 블록 뽑아내기
try/catch 블록은 코드 구조에 혼란을 일으키며, 정상 동작과 오류 처리 동작을 뒤섞는다. 그러므로 try/catch 블록을 별도 함수로 뽑아내는 편이 좋다. 아래는 기존의 `deletePage` 에서 오류를 처리하는 `delete` 함수와 실제 페이지를 제거하는 함수인 `deletePageAndAllReferences`로 분리한다.

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

위에서 `delete` 함수는 모든 오류를 처리한다. 그래서 코드를 이해하기 쉽다. 한 번 훑어보고 넘어가면 충분하다. 실제로 페이지를 제거하는 함수는 `deletePageAndAllReferences`다. `deletePageAndAllReferences`함수는 예외를 처리하지 않는다. 이렇게 정상 동작과 오류 처리 동작을 분리하면 코드를 이해하고 수정하기 쉬워진다.

## 오류 처리도 한 가지 작업이다.
함수는 '한 가지' 작업만 해야 한다. 오류 처리도 '한 가지' 작업에 속한다. 그러므로 오류를 처리하는 함수는 오류만 처리해야 마땅하다. (위 예제에서 보았듯이) 함수에 키워드 try가 있다면 함수는 try 문으로 시작해 catch/finally 문으로 끝나야 한다는 말이다.

## Error.java 의존성 자석
오류 코드를 반환한다는 이야기는, 클래스든 열거형 변수든, 어디선가 오류 코드를 정의한다는 뜻이다.

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

위와 같은 클래스는 의존성 자석(magnet)이다. `Error enum`이 변한다면 `Error enum`을 사용하는 클래스 전부를 다시 컴파일하고 다시 배치해야 한다. 그래서 `Error` 클래스 변경이 어려워진다. 그래서 새 오류 코드를 추가하는 대신 기존 오류 코드를 재사용한다.

이는 예외를 사용하여 해결할 수 있다. 오류 코드 대신 예외를 사용하면 새 예외는 `Exception` 클래스에서 **파생**된다. 따라서 재컴파일/재배치 없이도 새 예외 클래스를 추가할 수 있다.

# 반복하지 마라!
목록 3-1을 주의해서 읽어보면 네 번이나 반복되는 알고리즘이 보인다. 알고리즘 하나가 `SetUp`, `SuiteSetUp`, `TearDown`, `SuiteTearDown`에서 반복된다. 다른 코드와 섞이면서 모양이 조금씩 달라져 중복이 금방 드러나지는 않지만 중복은 문제이다. 목록 3-7 에서는 `include`함수로 중복을 없앤다.

중복을 없애면 모듈 가독성이 크게 높아진다.

어쩌면 중복은 소프트웨어에서 모든 악의 근원이다. 많은 원칙과 기법이 중복을 없애거나 제어할 목적으로 나왔다. 

- 예를 들어, E. F. 커드(E.F. Codd)는 자료에서 중복을 제거할 목적으로 관계형 데이터베이스에 정규 형식을 만들었다.
- 객체 지향 프로그래밍은 코드를 부모 클래스로 몰아 중복을 없앤다. 
- 구조적 프로그래밍, AOP(Aspect Oriented Programming), COP(Component Oriented Programming) 모두 어떤 편에서 중복 제거 전략이다. 

이처럼 하위 루틴을 발병한 이래로 소프트웨어 개발에서 지금까지 일어난 혁신은 소스 코드에서 중복을 제거하려는 지속적인 노력으로 보인다.

# 구조적 프로그래밍
에츠허르 데이크스트라(Edsger Dijkstra)는 구조적 프로그래밍적은 측면에서 모든 함수와 함수 내 모든 블록에 입구(entry)와 출구가 하나만 존재해야 한다고 말했다. 즉, 함수는 `return` 문이 하나여야 한다는 말이다. 루프 안에서 `break`나 `continue`를 사용해선 안 되며 `goto`는 절대로, 절대로 안 된다.

하지만 이러한 함수가 작다면 위 규칙은 별 이익을 제공하지 못한다. 함수가 아주 클 때만 상당한 이익을 제공한다.

그러므로 함수를 작게 만든다면 간혹 `return`, `break`, `continue`를 **여러 차례 사용해도 괜찮다**. 오히려 때로는 단일 입/출구 규칙(single entry-exit rule)보다 의도를 표현하기 쉬워진다. 반면, `goto` 문은 큰 함수에서만 의미가 있으므로, 작은 함수에서는 피해야만 한다.

# 함수를 어떻게 짜죠?
소프트웨어를 짜는 행위는 여느 글짓기와 비슷하다. 논문이나 기사를 작성할 때는 먼저 생각을 기록한 후 읽기 좋게 다듬는다. 초안은 대개 서투르고 어수선하므로 원하는 대로 읽힐 때까지 말을 다듬고 문장을 고치고 문단을 정리한다.

함수를 짤 때도 마찬가지다. 처음에는 길고 복잡하다. 들여쓰기 단계도 많고 중복된 루프도 많다. 인수 목록도 아주 길다. 이름은 즉흥적이고 코드는 중복된다. 하지만 나는 그 서투른 코드를 빠짐없이 테스트하는 단위 테스트 케이스도 만든다.

그런 다음 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거한다. 메서드를 줄이고 순서를 바꾼다. 때로는 전체 클래스를 쪼개기도 한다. **이 와중에도 코드는 항상 단위 테스트를 통과한다.**

최종적으로는 이 장에서 설명한 규칙을 따르는 함수가 얻어진다. 처음부터 탁 짜내지 않는다. 그게 가능한 사람은 없으리라.

# 최종 re-re-refactoring 버전 (목록 3-7 함수)
```java
package fitnesse.html;
import fitnesse.responders.run.SuiteResponder;
import fitnesse.wiki.*;

public class SetupTeardownIncluder {
    private PageData pageData;
    private boolean isSuite;
    private WikiPage testPage;
    private StringBuffer newPageContent;
    private PageCrawler pageCrawler;

    public static String render(PageData pageData) throws Exception {
        return render(pageData, false);
    }

    public static String render(PageData pageData, boolean isSuite) throws Exception {
        return new SetupTeardownIncluder(pageData).render(isSuite);
    }

    private SetupTeardownIncluder(PageData pageData) {
        this.pageData = pageData;
        testPage = pageData.getWikiPage();
        pageCrawler = testPage.getPageCrawler();
        newPageContent = new StringBuffer();
    }

    private String render(boolean isSuite) throws Exception {
        this.isSuite = isSuite;
        if (isTestPage())
            includeSetupAndTeardownPages();
        return pageData.getHtml();
    }

    private boolean isTestPage() throws Exception {
        return pageData.hasAttribute("Test");
    }

    private void includeSetupAndTeardownPages() throws Exception {
        includeSetupPages();
        includePageContent();
        includeTeardownPages();
        updatePageContent();
    }

    private void includeSetupPages() throws Exception {
        if (isSuite)
            includeSuiteSetupPage();
        includeSetupPage();
    }

    private void includeSuiteSetupPage() throws Exception {
        include(SuiteResponder.SUITE_SETUP_NAME, "-setup");
    }

    private void includeSetupPage() throws Exception {
        include("SetUp", "-setup");
    }

    private void includePageContent() throws Exception {
        newPageContent.append(pageData.getContent());
    }

    private void includeTeardownPages() throws Exception {
        includeTeardownPage();
        if (isSuite)
            includeSuiteTeardownPage();
    }

    private void includeTeardownPage() throws Exception {
        include("TearDown", "-teardown");
    }

    private void includeSuiteTeardownPage() throws Exception {
        include(SuiteResponder.SUITE_TEARDOWN_NAME, "-teardown");
    }

    private void updatePageContent() throws Exception {
        pageData.setContent(newPageContent.toString());
    }

    private void include(String pageName, String arg) throws Exception {
        WikiPage inheritedPage = findInheritedPage(pageName);
        if (inheritedPage != null) {
            String pagePathName = getPathNameForPage(inheritedPage);
            buildIncludeDirective(pagePathName, arg);
        }
    }

    private WikiPage findInheritedPage(String pageName) throws Exception {
        return PageCrawlerImpl.getInheritedPage(pageName, testPage);
    }

    private String getPathNameForPage(WikiPage page) throws Exception {
        WikiPagePath pagePath = pageCrawler.getFullPath(page);
        return PathParser.render(pagePath);
    }

    private void buildIncludeDirective(String pagePathName, String arg) {
        newPageContent
            .append("\n!include ")
            .append(arg)
            .append(" .")
            .append(pagePathName)
            .append("\n");
    }
}
```

# 결론
모든 시스템은 특정 응용 분야 시스템을 기술할 목적으로 프로그래머가 설계한 도메인 특화 언어(Domain Specific Language, DSL)로 만들어진다. **함수는 그 언어에서 동사며, 클래스는 명사다.** 요구사항 문서에 나오는 명사와 동사를 클래스와 함수 후보로 고려한다는 끔찍한 옛 규칙으로 역행하자는 이야기가 아니다. 아니, 이것은 오히려 훨씬 더 오래된 진실이다. **프로그래밍의 기술은 언제나 언어 설계의 기술**이다. 예전에도 그랬고 지금도 마찬가지다.

**대가(master) 프로그래머는 시스템을 "구현할" 프로그램이 아니라 "풀어갈" 이야기로 여긴다.** 프로그래밍 언어라는 수단을 사용해 좀 더 풍부하고 좀 더 표현력이 강한 언어를 만들어 이야기를 풀어간다. 시스템에서 발생하는 모든 동작을 설명하는 함수 계층이 바로 그 언어에 속한다. 

이 장은 함수를 잘 만드는 기교를 소개했다. 여기서 설명한 규칙을 따른다면 길이가 짧고, 이름이 좋고, 체계가 잡힌 함수가 나올 것이다. 

하지만 진짜 목표는 시스템이라는 이야기를 풀어가는 데 있다는 사실을 명심해야한다. **작성하는 함수가 분명하고 정확한 언어로 깔끔하게 같이 맞아떨어져야 이야기를 풀어가기가 쉬워**진다는 사실을 기억해야 한다.
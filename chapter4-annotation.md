---
nav_order: 4
---

# 4장 주석

- 주요 내용
    - 어떤 내용을 주석으로 표시해야하는지
    - 주석을 주의해서 달아야하는 이유
    - 좋은 주석과 나쁜 주석의 종류와 예시
    - 나쁜 주석을 개선하는 예시

---

## 주석은 나쁜 코드를 보완하지 못한다

```
요약: 코드를 잘 작성한다면, 주석은 필요하지 않다. 그렇기에 주석을 다는 것보다 코드를 잘 작성하는 것이 좋다.
```

- 코드로만 작성하는 것이 가장 좋다는 것을 이야기한다.
  > 프로그래밍 언어 자체가 표현력이 풍부하다면, 아니 우리에게 프로그래밍 언어를 치밀하게 사용해 의도를 표현할 능력이 있다면, 주석은 거의 필요하지 않으리라. 아니, 전혀 필요하지 않으리라.

- 보통 주석을 사용하는 이유는, 코드를 깨끗하게 작성하는 것을 실패해 주석을 사용한다.
  > 우리는 코드로 의도를 표현하지 못해, 그러니까 실패를 만회하기 위해 주석을 사용한다.

- 주석을 되도록이면 사용하지 않아야하는 큰 이유: `프로그래머들이 주석을 유지하고 보수하기란 현실적으로 불가능하다.`
  > 내가 이렇듯 주석을 무시하는 이유가 무엇이냐고? 거짓말을 하니까. 오래될수록 완전히 그릇될 가능성도 커진다.
  > 이유는 단순하다. 프로그래머들이 주석을 유지하고 보수하기란 현실적으로 불가능하다.

- 주석을 지양해야하는 이유로 부정확한 주석은 매우 좋지 않고, 코드만이 진실이기 때문임을 얘기한다.   
  > 부정확한 주석은 아예 없는 주석보다 훨씬 더 나쁘다.
  > 부정화한 주석은 독자를 현혹하고 오도한다.
  > 진실은 한곳에만 존재한다. 바로 코드다.

- 주석은 나쁜 코드를 보완하지 못한다.
  > 표현력이 풍부하고 깔끔하며 주석이 거의 없는 코드가, 복잡하고 어우선하며 주석이 많이 달린 코드보다 훨씬 좋다.
  > 자신이 저지른 난장판을 주석으로 설명하려 얘스는 대신에 그 난장판을 깨끗이 치우는 데 시간을 보내라!

## 코드로 의도를 표현하라!
```
요약: 대부분의 주석은 코드로 작성할 수 있고, 이러한 방향이 더 좋다.
```

- 코드로 의도를 작성하기 어려운 경우가 있지만, 이게 코드가 나쁜 수단이라는 의미가 아니다.
  > 확실히 코드만으로 의도를 설명하기 어려운 경우가 존재한다.
  > 불행히도 많은 개발자가 이를 코드는 훌륭한 수단이 아니라는 의미로 해석한다.
  > 이는 분명히 잘못된 생각이다. 

- 주석을 코드로 변경하는 예시
  - 이전 
    ```java
    // 직원에게 복지 혜택을 받을 자격이 있는지 검사한다.
    if((emoloyee.flags & HOURLY_FLAG) && (emoloyee.age > 65))
    ```
  - 변경
    ```java
    if(emoloyee.isEligibleForFullBenefits())
    ```
  

## 좋은 주석

### 법적인 주석

- 저작권 정보, 소유권 정보와 같은 법적인 주석은 필요하다
  > 표준에 맞춰 법적인 이유로 특정 주석을 넣으라고 명시한다. 
  > 각 소스 첫머리에 주석으로 들어가는 저작권 정보와 소유권 정보는 필요하고도 타당하다.

- 예시
  ```java
  // Copyright (c) 2003,2004,2005 by object Mentor, rnc. All rights reserved. 
  // GNU Generat Pubtic License 버전 2 이상 따르는 조건으로 배포한다.
  ```

### 정보를 제공하는 주석

- 기본적인 정보를 제공하면 편리하지만, 이왕이면 변환 클래스나 메서드 이름으로 이를 잘 표현하는 등의 코드로 표현하는 것이 더 좋다.
  > 때로는 기본적인 정보를 주석으로 제공하면 것은 편리하다.
  > 위와 같은 주석이 유용하다 할지라도, 가능하다면, 함수 이름에 정보를 담는 편이 더 좋다.
  > 이왕이면 시각과 날짜를 변환하는 클래스를 만들어 코드를 옮겨주면 더 좋고 더 깔끔하겠다.
  > 그러면 주석이 필요 없어진다.


- 예시
  - 테스트 코드
    ```java
    // 테스트 중인 Responder 인스턴스를 반환한다.
    protected abstract Responder responderInstance();
    ```
  - 정규식
    ```java
    // kk:mm:ss EEE, MMM dd, yyyy 형식이다.
    Pattern timeMatcher = Pattern.compile("\\d*:\\d*:\\d* \\w*, \\w* \\d* \\d*");
    ```

### 의도를 설명하는 주석

- 예시
    - 의도를 설명하지 않는 코드: 오른쪽 유형이 무엇인지를 설명하지 않아 어떤 정렬을 하려고 한건지 의도를 모름
      ```java
      public int compareTo(0bject o){
        if (o instanceof WikiPagePath){
          WikiPagePath p = (WlkiPagePath) o;
          String compressedr,lame = St ringUtil.join( names, "" ) ;
          String compressedArgumentName = St ringUtit.join( p. names, "") ;
          return comp ressedName. compareTo( compressedArgumentName ) ;
        }
          return 1; // 오른쪽 유형이므로 정렬 순위가 더 높다.
      }
      ```
    - 의도를 설명하는 코드: 어떤 의도인지를 적어놓음으로써, 코드를 작성한 의도를 알게됨
      ```java
      pubtic void testConcurrentAddWidgets() throws Exception {
        WidgetBuilder widgetBuilder = new WidgetBuilder( new Class [] {BotdWidget.class} ) ;
        String text = "'''"bold text'''";
        ParentWidget parent = new BotdWidget(new MockWidgetRoot(), "'''"bold text'''");
        AtomicBootean faitFlag = new AtomicBooteanO ;
        failFlag.set(false) ;
      
        // 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다.
        for (int i = 0; i < 25000; i++) {
          WldgetBuilderThread widgetBuitderThread = new WidgetBuitderThread(widgetBui.lder, text, parent, failFlag) ;
          Thread thread = new Thread (widgetBuitderThread ) ;
          thread.start();
        }
        assertEquats(false, failFlag. get( ) ) ;
      }
      ```

### 의미를 명료하게 밝히는 주석

- 코드를 변경할 수 없는 경우 의미를 명료하게 밝히는 주석, 하지만 위험하기에 주의해야한다.
  > 표준 라이브러리나 변경하지 못하는 코드에 속한다면 의미를 명료하게 밝히는 주석이 유용하다.
  > 그러므로 위와 같은 주석을 달 때는 더 나은 방법이 없는지 고민해보고 정확히 달도록 각별히 주의한다. 

- 예시
  ```java
  assertTure(a.compareTo(a) ==0 ); // a == a
  ```

### 결과를 경고하는 주석

- 다른 프로그래머에게 결과를 경고할 목적으로 주석을 사용한다.
    > 때로 다른 프로그래머에게 결가를 경고할 목적으로 주석을 사용한다.

- 예시 
    ```java
    // SimpleDateFormat은 스레드에 안전하지 못하다.
    // 따라서 각 인스턴스를 독립적으로 생성해야 한다.
    SimpleDateFormat df = new SimpteDateFormat("EEE, dd MMM yyyy HH:nm:ss z");
    ```

### TODO 주석

- 앞으로 할 일을 TODO 주석으로 남기면 좋지만, 나쁜 코드를 남겨놓는 핑계로 사용해는 안된다.
  > 앞으로 할 일을 TODO 주석으로 남겨두면 편하다.
  > 하지만 어떤 용도로 사용하든 시스템에 나쁜 코드를 남겨 놓는 핑계가 되어서는 안 된다.

### 중요성을 강조하는 주석

- 놓치기 쉬운 중요한 부분을 강조하는데 사용한다.
  > 자칫 대수롭지 않다고 여겨질 뭔가의 중요성을 강조하기 위해서도 주석을 사용한다.

- 예시
  ```java
  String listItemContent = match(3).trim();
  // 여기서의 trim()은 정말 중요하다. trim 함수 문자열에서 시작 공백을 제거한다.
  // 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 떄문이다.
  ```

### 공개 API에서 javadocs

- 공개 API를 구현한다면 javadocs를 작성해야하지만, 주의해야한다.
  > 공개 API를 구현한다면 반드시 훌륭한 javadocs를 작성한다.
  > 하지만 이 장에서 제시하는 나머지 충고도 명심하길 바란다.

## 나쁜 주석

### 주절거리는 주석

- 이해가 안되서 다른 모듈까지 확인해야하는 주석은 독자와 제대로 소통하지 못한 주석은 시간 낭비다.
  > 주석을 달기로 결정했다면 충분한 시간을 들여 최고의 주석을 달도록 노력한다.
  > 확실히 저자에게야 의미가 있겠지만 그 의미가 다른 사람들에게는 전해지지 않는다.
  > 답을 알아내려면 다른 코드를 뒤져보는 수밖에 없다. 이해가 안 되어 다른 모듈까지 뒤져야 하는 주석은 독자와 제대로 소통하지 못하는 주석이다.

### 같은 이야기를 중복하는 주석

- 코드와 같은 내용을 전달하는 주석으로, 코드 내용이 간단하다면 오히려 주석을 읽는데 시간이 더 걸린다.
  > 헤더에 달린 주석이 같은 코드 내용을 그대로 중복한다. 자칫하면 코드보다 주석을 읽는 시간이 더 오래 걸린다.
  > 코드보다 읽기 쉽지도 않다.
  
- 예시
  - 내용이 메서드 내용과 같음
    ```java
    // this.closed가 true일 때 반환되는 유틸리티 메서드다.
    // 타임아웃에 도달하면 예외를 던진다.
    public synchronized void waitForClose(final long timeoutMillis) throws Exception{
        if(!closed){
          wait(timeoutMillis);
          if(!closed){
            throw new Exception("MockResponseSender could not be closed");
          }
        }
    }
    ```

  - 각 변수 이름을 해석한 것과 같음
    ```java
    public abstract class ContainerBase implements Contrainer, Lifecyle, Pipeline, MBeanRegistraion, Serializable{
    
        /**
        * 이 컴포넌트의 프로세서 지연 값
        */
        protected int backgroundProcessorDelay = -1;
        /**
        * 이 컴포넌트를 위한 컨테이너 이벤트 Listener
        */
        protected ArrayList listeners = new ArrayList();
    
        ...
    }
    ```

### 오해할 여지가 있는 주석

- 오해할 여지가 있는 주석으로, 오류 발생시 오류의 원인을 다른 곳에서 찾게되어 시간이 더 소요될 수 있다.
    > 떄떄로 의도는 좋았으나 프로그래머가 딱 맞을 정도로 엄밀하게 주석을 달지 못하기도 한다.
    > (코드보다 읽기도 어려운) 주석에 담긴 "살짝 잘못된 정보"로 인해 어느 프로그래머가 경솔하게 함수를 호출할지도 모른다.


- 예시
    ```java
    // this.closed가 true일 때 반환되는 유틸리티 메서드다.
    // 타임아웃에 도달하면 예외를 던진다.
    public synchronized void waitForClose(final long timeoutMillis) throws Exception{
        if(!closed){
          wait(timeoutMillis);
          if(!closed){
            throw new Exception("MockResponseSender could not be closed");
          }
        }
    }
    ```


### 의무적으로 다는 주석

- 모든 함수에 javadocs를 넣는 규칙은 안좋다.
    > 모든 함수에 javadocs를 달거나 모든 변수에 주석을 달아야 한다는 규칙은 어리석기 그지없다.
    > 이런 주석은 코드를 복합하게 만들며, 거짓말을 퍼뜨리고, 혼동과 무질서를 초래한다.

### 이력을 기록하는 주석

- 과거엔 모든 모듈 첫머리에 변경 이력을 관리하는게 바람직했지만 지금은 소스 관리 시스템을 이용하자.
    > 예전에는 모든 모듈 첫 머리에 변경 이력을 관리하는 관례가 바람직 했다.
    > 당시에는 소스 코드 관리 시스템이 없었으니까.
    > 하지만 이제는 혼란만 가중할 뿐이다. 완전히 제거하는 편이 좋다.

- 예시
```java
/**
 * 변경이력
 * -----------------------------
 * 11-Oct-2001 : 클래스를 다시 정리하고 새로운 패키지인 com.jrefinery.data로 옮겼다 (DG);
 ...
```
  
### 있으나 마나 한 주석

- 너무 당연한 내용을 알려주는 주석
    > 너무 당연한 사실을 언급하며 새로운 정보를 제공하지 못하는 주석
    > 위와 같은 주석은 지나친 참견이라 개발자가 주석을 무시하는 습관에 빠진다.
    > 있으나 마나 한 주석을 달려는 유혹에 벗어나 코드를 정리하라. 더 낫고, 행복한 프로그래머가 되는 지름길이다. 

- 예시
    ```java
    /**
     * 기본 생성자
     */
    protected AnnualDateRule(){
    }
    ```

### 무서운 잡음

- javadocs를 꼭 제공해야한다는 생각으로 copy,paste를 이용해 작성하면 잡음이 생길 수 있다. 오히려 작성하지 않는 것이 좋다.

- 예시
    ```java
    /** the name */
    private String name;
  
    /** the version */ -> 중복
    private String version;
  
    /** the licenceName */
    private String licenceName;
  
    /** the version */ -> 중복
    private String info;
  
    ```

### 함수나 변수로 표현할 수 있다면 주석을 달지 마라

- 주석을 코드로 작성할 수 있으면 코드로 작성하자.
    > 하지만 위와 같이 주석이 필요하지 않도록 코드를 개선하는 편이 더 좋았다.

- 예시
  - 이전 
    ```java
    // 전역 목록 <smodule>에 속하는 모듈은 우리가 속한 하위 시스템에 의존하는가?
    if(smodule.getDependSubsystems().contains(subSysMod.getSubSystem()))
    ```
  - 이후
    ```java
    ArrayList moduleDependenees = smodule.getDependSubsystems();
    String ourSubsystem = subSysMod.getSubsystem();
    if (moduleDependees.contains(ourSubSystem))
    ```


### 위치를 표시하는 주석

- 위치를 표시하기 위해 배너를 사용하는데, 반드시 필요할 때만 드물게 사용하자
    > 너무 자주 사용하지 않는다면 배너는 눈에 띄며 주의를 환기한다.
    > 그러므로 반드시 필요할 때만, 아주 드물게 사용하는 편이 좋다.
    > 배너를 남용하면 독자가 흔한 잡음으로 여겨 무시한다.
- 예시
    ```java
    // Actions /////////////////////////////////////////////////////////
    ```
### 닫는 괄호에 다는 주석

- 중첩이 심한 경우 닫는 괄호에 주석을 달아놓는데, 주석보다 중첩을 제거하자.
    > 그러므로 닫는 괄호에 주석을 달아야겠다는 생각이 든다면 대신에 함수를 줄이려 시도하자.

### 공로를 돌리거나 저자를 표시하는 주석

- 저자를 표시하려는 주석은 소스 코드관리 시스템을 이용하자.
    > 소스 코드 관리 시스템은 누가 언제 무엇을 추가했는지 귀신처럼 기억한다. 저자의 이름으로 코드를 오염시킬 필요가 없다.
    > 위와 같은 정보는 소스 코드 관리 시스템에 저장하는 편이 좋다.

### 주석으로 처리한 코드

- 주석으로 처리된 코드는 삭제하고, 소스 코드 관리 시스템을 이용하자.
    > 하지만 우리는 오래전부터 우수한 소스 코드 관리 시스템을 사용해왔다. 소스 코드 관리 시스템이 우리를 대신해 코드를 기억해준다.

### HTML 주석

- 주석을 뽑아 웹페이지에 올릴 작정이라면 HTML태그는 도구가 삽입해야한다.
    > (javadocs와 같은) 도구로 주석을 뽑아 웹 페이지에 올릴 작정이라면 주석에 HTML 태그를 삽입해야 하는 책임은 프로그래머가 아니라 도구가 져야한다.

### 전역 정보

- 주석은 전역정보가 아닌 근처에 있는 코드만 기술하라.
    > 주석을 달아야 한다면 근처에 있는 코드만 기술하라.
    > 코드 일부에 주석을 달면서 시스템의 전반적인 정보를 기술하지 마라.

- 예시: 함수는 포트 기본값을 통제하지 못하는데, 작성되어 있음

```java
/**
 * 적합성 테스트가 동작하는 포트: 기본값은<b>8082</b>
 * @param fitnessePort
 */
public void setFintnessePort(int fitessePort){
    this.fitnessePort = finessePort;
}
```

### 너무 많은 정보

- 관련 없는 정보(역사 등)은 작성하지 마라.
    > 주석에다 흥미로운 역사나 관련 없는 정보를 장황하게 늘어놓지 마라.

### 모호한 관계

- 주석과 코드의 관게는 명확해야하고, 코드의 부족한 점이나 외의 관련 있는 부가정보를 제공해 독자가 이해해야한다.
    > 주석과 주석이 설명하는 코드는 둘 사이 관계가 명백해야 한다.
    > 주석을 다는 목적은 코드만으로 설명이 부족해서다. 주석 자체가 다시 설명을 요구하니 안타깝기 그지없다.

- 예시: 이 주석과 코드를 보고 이해할 수 없음
    
    ```java
    /**
     * 모든 픽셀을 담을 만큼 충분한 배열로 시작한다(여기에 필터 바이트를 더한다).
     * 그리고 헤더 정보를 위해 200바이트를 더한다.
     */
    this.pngBytes - new byte[((this.width +1) * this.height *3 ) + 200];
    ```

### 함수 헤더

- 짧은 함수은 주석이 필요 없고, 이름을 잘 지은 함수가 주석이 있는 함수보다 좋다.
  > 짧은 함수는 긴 설명이 필요없다. 짧고 한 가지만 수행하며 이름을 잘 붙인 함수가 주석으로 헤더를 추가한 함수보다 훨씬 좋다.

### 비공개 코드에서 javadocs

- 비공개되는 코드라면 javadocs는 필요없다.
  > 공개 API는 javadocs가 유용하지만 공개하지 않을 코드라면 javadocs는 쓸모가 없다.

### 예제


- 이전
    ```java
    /**
     * 이 클래스는 사용자가 지정한 최대 값까지 소수를 생성한다. 사용된 알고리즘은 에라스토테네스의 체다.
     * <p> -> 삭제
     * 에라스토테네스: 기원전 276년에 리비아 키레네에서 출생, 기원전 194년에 사망, 지구 둘레를 최초로 계산한 사람이자 달력에 윤년을 도입한 사람. -> 삭제
     * <p> -> 삭제
     * 알고리즘은 상당히 단순하다. 2에서 시작하는 정수 배열을 대상으로 2의 배수를 모두 제거한다. 다음으로 남은 정수를 찾아 이 정수의 배수를 모두 지운다. 
     * 최대 값의 제곱근이 될 때까지 이를 반복한다.
     *
     * @author Apphonse-> 삭제
     * @version 13 Fed 2002 atp-> 삭제
     */
     import java.util.*;
     
     public class GeneratePrimes{
        public static int[] generatePrimes(int maxValue){
            if(maxValue >= 2){ // 유일하게 유효한 경우 -> maxValue <2 인 경우를 먼저 Return 
                // 선언-> 전역으로 변경
                int s = maxValue + 1; // 배열 크기
                boolean[] f = new boolean[s];
                int i;
                
                // 배열을 참으로 초기화 -> uncrossintegersUpTo(int maxValue)로 변경 
                for(i=0;i<s;i++){
                    f[i] = true;
                }
                
                // 소수가 아닌 알려진 숫자를 제거
                f[0] = f[1] = false;
                
                // 체
                int j;
                for (i =2; i<Math.sqrt(s)+1; i++){
                    if(f[i]){  // i가 남아있는 숫자라면 이 숫자의 배수를 구한다.
                        for(j = 2 * i; j<s; j+=i)
                            f[j] = false; // 배수는 소수가 아니다.
                    }
                }
                
                // 소수의 개수는?
                int count = 0;
                for(i =0; i<s; i++){
                    if(f[i])
                        count++; // 카운트 증가
                }
                
                int[] primes = new int[count];
                
                // 소수를 결과 배열로 이동한다.
                for(i = 0, j=0; i<s; i++){
                    if(f[i]) // 소수일 경우에
                        primes[j++] = i;
                }
                
                return primes; // 소수를 반환한다.
            }
            else // maxValue < 2
                return new int[0]; // 입력이 잘못되면 비어있는 배열을 반환한다.
            
        }
     }
  
- 이후

```java
/** 
 * 이 클래스는 사용자가 지정한 최대 값까지 소수를 구한다.
 * 알고리즘은 에라스토테네스의 체다.
 * 2에서 시작하는 정수 배열을 대상으로 작업한다.
 * 처음으로 남아 있는 정수를 찾아 배수를 모두 제거한다.
 * 배열에 더 이상 배수가 없을 때까지 반복하낟.
 */
 
 public class PrimeGenerator{
    private static boolean[] crossedOut;
    private static int[] result;
    
    public static int[] generatePrimes(int maxValue){
        if(maxValue < 2)
            reutrn new int[0];
        else{
            uncoressIntegersUpTo(maxValue);
            crossOutMultiples();
            putUncressedIntegerIntoResult();
            return result;
        }
    }
    
    private static void uncrossIntegersUpTo(int maxValue){
        crossedOut = new boolean[maxValue + 1];
        for(int i =2; i< croessedOut.length ; i++)
            crossedOut[i] = false;
    }
    
    private static void coressOutMultiples(){
        int limit = determineIterationLimit();
        for(int i=2; i <=limit; i++)
            if(notCrossed(i))
                crossOutMultiplesOf(i);
    }
    
    private static int determineIterationLimit(){
        // 배열에 있는 모든 배수는 배열의 크기의 제곱근보다 작은 소수의 인수다.
        // 따라서 이 제곱근보다 더 큰 숫자의 배수는 제거할 필요가 없다.
        double iterationLimit = Math.sqrt(crossedOut.length);
        return (int) iterationLimit;
    }
    
    private static void crossOutMultiplesOf(int i){
        for(int multiple = 2*1;multiple < crossedOut.length;mulitple +=i;)
            crossedOut[multiple] = true;
    }
    
    private static boolean notCrossed(int i){
        return crossedOut[i] == false;
    }
    
    private static void putUncrossedIntegersIntoResult(){
        for(int j=0, i=2; i<crossedOut.length; i++)
            if(notCrossed(i))
                result[j++] = i;
    }
    
    private static int numberOfUncrossedIntegers(){
        int count =0;
        for(int i =2; i < crossedOut.lenghth;i++)
            if (notCrossed(i))
                count++;
            
        return count;
    }
 }
```

--- 

# 요약

- 주석은 프로그래머가 계속해서 유지하고 보수하기가 힘들기 때문에 잘못된 정보를 전달할 가능성이 높고, 외에도 가독성을 해치는 경우가 있을 수 있다.
- 코드를 잘 작성한다면 대부분의 주석은 코드로 작성할 수 있기 때문에, 주석을 작성하는 것보다 코드를 수정하는 방향이 좋다.
- 달아야하는 주석: 항상 유의해서 주석을 추가해야한다.
    - 법적인 주석
    - 의미나 의도를 명료하게 알려주는 주석
    - 놓칠 수 있는 부분을 강조하는 주석
    - TODO 주석
    - 공개 API의 javadocs
- 삭제해야하는 주석
    - 소스 코드 관리 시스템으로 대체 가능한 주석 -> 삭제 후 소스 코드 관리 시스템 이용
        - e.g. 저자 기록, 코드 수정 기록, 사용하지 않는 코드 등
    - 주석으로써 필요한 정보가 아닌 주석 -> 삭제
        - e.g. 코드와 같은 내용, 의무적인 javadocs, 비공개 코드의 javadocs, 전역 정보, 위치 표시, 너무 많은 정보, 닫는 괄호 주석, 너무 당연한 내용 등
    - 코드로 작성할 주석 -> 코드를 변경함으로써 주석을 사용하지 않음
        - e.g. 함수 헤더 등
- 주의점
    - 되도록이면 작성하지 말기
    - 항상 주석을 작성할 때에는 의미를 빠르게 알 수 있도록, 필요한 내용만 명확하게 잘 작성하기
    - 작성된 주석은 코드와 같이 항상 유지보수하기

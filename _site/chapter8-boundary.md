# 경계

개발할 때 모든 부분을 직접 개발하는 경우는 드물고, 외부 코드(e.g. 오픈 소스, 외부 컴포넌트나 패키지 등)을 이용하는 경우가 많다.
이러한 외부 코드와 우리 코드에 깔끔하게 통합하는 기법과 기교를 살펴본다.


- 주요 내용
  - 외부 코드 사용시, 문제가 생기는 이유
  - 외부 코드 사용시, 학습 테스트를 이용하는 방법과 이점
  - 아직 존재하지 않는 코드 사용시, 경계를 구분하는 방법과 변경에 대한 대응 방법

---

## 외부 코드 사용하기

- 외부 코드 사용시 문제가 생기는 이유: 제공자와 사용자는 서로 다른 목표를 추구 -> 시스템 경계에서 문제가 생길 수 있음
  - 외부 코드 제공자: 적용성을 늘리려고 노력함
  - 외부 코드 사용자: 자신의 요구에 집중하는 인터페이스를 바람
  
  > 패키지 제공자나 프레임워크 제공자는 적용성을 최대한 넓히려 애쓴다. 더 많은 환경에서 돌아가야 더 많은 고객이 구매하니까.
  > 반면 사용자는 자신의 요구에 집중하는 인터페이스를 바란다. 이런 긴장으로 인해 시스템 경계에서 문제가 생길 소지가 많다.

- 예시(`java.util.Map`): 제공자는 수많은 기능을 제공해 유연성을 높임, 그만큼 위험도 큼
  - case 1: 주고 받는 데이터로 map을 사용한다면, map을 이용하는 코드 어디서든 내용을 지울 수 있어(`clear()`) 위험
  - case 2: 특정 객체만 map에 저장하기로 설계, 그렇지만 map을 이용하는 코드 어디서든 다른 객체 유형도 추가 가능해 위험
  - 대안: map을 감싸는 wrapper class를 정의해 사용 -> map과 유사한 인터페이스를 주고 받지 말라는 의미
    ```java
    public class Sensors{
      private Map sensors = new HashMap();
      public Sensor getById(String id){
        return (Sensor) sensors.get(id);
      }
    
      ...
    }
    ```
  > Map 클래스를 사용할 때마다 위와 같이 캡슐화하라는 소리가 아니다. Map(혹은 유사한 경계 인터페이스를) 여기저기 넘기지 말라는 말이다.
  > Map과 같은 경계 인터페이스를 이용할 때는 이를 이용하는 클래스나 클래스 계열 밖으로 노출되지 않도록 주의한다.
  > Map 인스턴스를 공개 API의 인수로 넘기거나 반환값으로 사용하지 않는다.

## 경계 살피고 익히기

- 외부 코드의 테스트를 함으로써 외부 코드의 사용법 익히고 외부 코드를 통합을 진행하자.
  - 짐 뉴커크의 학습 테스트
    - 바로 우리쪽 코드를 작성해 외부 코드를 호출하는 대신 먼저 간단한 테스트 케이스를 작성해 외부 코드를 익히는 방식
    - 사용하려는 방식대로 외부 API를 호출해, 통제도니 환경에서 API를 제대로 이해하는지 확인
  > 외부 코드를 익히기는 어렵다. 외부 코드를 통합하기도 어렵다. 두 가지를 동시에 하기는 두 배나 어렵다.
  > 다르게 접근하면 어떨까? 곧바로 우리쪽 코드를 작성해 외부 코드를 호출하는 대신 먼저 간단한 테스트 케이스를 작성해 외부 코드를 익히면 어떨까? 
  > 짐 뉴커크는 이를 학습 테스트라 부른다.

## log4j 익히기

학습 테스트의 예시: log4j 익히기

1. 사용하려는 방식으로 테스트 코드 작성
    ```java
    @Test
    public void testLogCreate(){
        Logger logger = Logger.getLogger("MyLogger");
        logger.info("hello");
    } 
    ```
2. 발생한 오류에 대해 처리하는 코드 작성
    ```java
    @Test
    public void testLogCreate(){
        Logger logger = Logger.getLogger("MyLogger");
        ConsoleAppender appender = new ConsoleAppender();
        logger.addAppender(appender);
        logger.info("hello");
    } 
    ```
    ```java
    @Test
    public void testLogCreate(){
        Logger logger = Logger.getLogger("MyLogger");
        logger.removeAllAppenders();
        logger.addAppender(new ConsoleAppender(
            new PatternLayout("%p %t %m%n"),
            ConsoleAppender.SYSTEM_OUT
        ));
        ConsoleAppender appender = new ConsoleAppender();
        logger.addAppender(appender);
        logger.info("hello");
    } 
    ```
3. 의심이 가는 코드에 대해 문서를 읽고, 수정
    ```java
    public class LogTest{
        private Logger logger;
   
        @Before
        public void initialize(){
            logger = Logger.getLogger("logger");
            logger.removeAllAppenders();
            Logger.getRootLogger().removeAllAppenders();
        }
   
        @Test
        public void basicLogger(){
            BasicConfigurator.configure();
            logger.info("basicLogger");
        }
   
        @Test
        public void addAppenderWithStream(){
            logger.addAppender(new ConsoleAppender(
                new PatternLayout("%p %t %m%n"),
                ConsoleAppender.SYSTEM_OUT
            ));
            logger.info("addAppenderWithStream");
        }
   
        @Test
        public void addAppenderWithoutStream(){
            logger.addAppender(new ConsoleAppender(
                new PatternLayout("%p %t %m%n")
            ));
            logger.info("addAppenderWithoutStream");
        }
    } 
    ```
4. 알게된 모든 지식을 클래스로 캡슐화한 뒤 이를 사용

## 학습 테스트는 공짜 이상이다

- 학습 테스트가 주는 이점
    - 비용없이 필요한 지식만 확보하는 방법
    - 패키지가 예상대로 동작하는지 확인
    - 새 버전이 출시했을 때, 우리 코드와 호환되는지 미리 확인 가능
    
    > 학습테스트는 공짜 이상이다.
    > 학습테스트는 이해도를 높여주는 정확한 실험이다.
    > 학습 테스트는 패키지가 예상대로 도는지 검증한다.
    > 패키지 새 버전이 나올 때마다 새로운 위험이 생긴다. 새 버전이 우리 코드와 호환되지 않으면 학습 테스트가 이 사실을 곧바로 밝혀낸다.

## 아직 존재하지 않는 코드를 사용하기

존재하지 않는(혹은 모르는) 코드를 사용해야한다면, 이 부분에 대한 내용을 분리해 임의로 작성(작업에 필요한 내용을 작성)한 뒤 작업을 진행한다.
그리고 실제 연동시에 Adapter패턴으로 API 사용을 캡슐화해 바뀔 때 수정할 코드를 한 곳으로 모은다.

> 경계와 관련해 또 다른 유형은 아는 코드와 모르는 코드를 분리하는 경계다.
> 저쪽 팀이 아직 API를 설계하지 않았으므로 구체적인 방법은 몰랐다.
> 이쪽 코드를 진행하고자 우리는 자체적으로 인터페이스를 정의했다.
> 우리가 바라는 인터페이스를 구현하면 우리가 인터페이스를 전적으로 통제한다는 장점이 생긴다.
> 또한 코드 가독성도 높아지고 코드 의도도 분명해진다.
> 우리는 (우리가 통제하지 못하며 정의되지도 않은) 송신기 API에서 `CommunicationsController`를 분리했다.
> ADAPTER 패턴으로 API사용을 캡슐화해 API가 바뀔 때 수정할 코드를 한 곳으로 모았다.

## 깨끗한 경계

- 통제하지 못하는 코드를 사용할 때는 너무 많은 투자를 하거나 향후 변경 비용이 지나치게 커지지 않도록 각별히 주의해야 한다.
- 경계에 위치하는 코드는 깔끔히 분리한다.
- 외부 패키지를 호출하는 코드를 가능한 줄여 경계를 관리하자.
- 어느 방법이든 코드 가독성이 높아지며, 경계 인터페이스를 사용하는 일관성도 높아지며, 외부 패키지가 변했을 때 변경할 코드도 줄어든다.


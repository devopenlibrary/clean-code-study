# 동시성

동시성과 깔끔한 코드는 양립하기 어렵다. 스레드 하나만 사용하는 코드는 작성하기 쉽지만, 동시성을 구현하려면 많은 고려가 필요하다.

동시성 문제는 시스템이 부하를 받기 전까지는 잘 드러나지 않으며, 한 번 발생하면 재현하기도 어렵다.

## 동시성이 필요한 이유?

동시성은 **결합을 없애는 전략**이다. 즉, **무엇**과 **언제**를 분리하는 전략이다.

**무엇(What)**
    
    프로그램이 해야 하는 작업 자체
    → "파일 다운로드", "DB에 저장", "로그 출력"

**언제(When)**

    그 작업을 실행하는 시점
        → "지금 바로 실행할지", "나중에 실행할지", "특정 조건이 충족될 때 실행할지"

- 스레드가 하나인 프로그램에서는 **무엇**과 **언제**가 밀접하다.
- **무엇**과 **언제**를 분리하면 애플리케이션 구조와 효율이 크게 향상된다.
    - 예: 서블릿
        - 서블릿은 EJB 컨테이너가 동시성을 일부 관리해 준다.
        - 프로그래머는 모든 웹 요청을 직접 관리하지 않아도 된다.
        - 각 서블릿 스레드는 독립적으로 동작한다.
        

**동시성이 필요한 상황**

- **구조적 개선**을 위해 도입하는 경우도 있지만
- 주로 **응답 시간, 처리량** 등의 요구사항으로 동시성이 불가피한 경우도 있다.
    - 정보 수집기
    - 사용자 수가 많은 시스템
    - 대량의 데이터를 처리하는 시스템
    

### 미신과 오해

- 동시성을 항상 성능을 높여준다.
    
    → X: 대기 시간이 충분히 길거나 독립적 계산이 많을 때만 그렇다.
    
- 동시성을 구현해도 설계는 변하지 않는다
    
    → X: 무엇과 언제를 분리하면 시스템 설계 자체가 달라진다.
    
- 웹 또는 EJB 컨테이너를 사용하면 동시성을 이해할 필요가 없다.
    
    → X: 컨테이너 동작 방식, 동시 수정/데드락 회피 방법을 이해해야 한다.
    

**동시성의 어려운 이유**

- 성능 부하: 락, 컨텍스트 스위칭 등으로 오버헤드 발생.
- 복잡성 증가: 단순한 문제도 동시성 때문에 어렵다.
- 버그 재현 어려움: 잠복 버그는 재현이 매우 힘들어 무시되기 쉽다.
- 설계 복잡성: 근본적인 설계 전략부터 재검토해야 한다.

## 난관

동시성을 구현하기 어려운 이유는 

```java
public class X {
    private int lastIdUsed;

    public int getNextId() {
        return ++lastIdUsed;
    }
}
```

lastIdUsed = 42인 상태에서 두 스레드가 동시에 `getNextId()` 호출 시:

1. A 스레드: 43 / B 스레드: 44 → lastIdUsed = 44
2. A 스레드: 44 / B 스레드: 43 → lastIdUsed = 44
3. A 스레드: 43 / B 스레드: 43 → lastIdUsed = 43 (⚠️)

스레드 간 동시 실행 경로는 수천~수만 개.

일부 경로에서 잘못된 결과가 발생한다.

## 동시성 방어 원칙

### 단일 책임 원칙Single Responsibility Principle, SRP

👉 **동시성 관련 코드는 다른 코드와  분리해야 한다.**

- 동시성 자체만으로도 충분히 복잡하다.
- 다른 코드에 섞이면 유지보수 난이도가 급격히 올라간다.
- 동시성 코드(락, 스레드 관리, 동기화)는 별도의 모듈로 분리하라.

### 따름 정리corollary: 자료 범위를 제한하라

**👉 자료를 캡슐화 하고, 공유 자료를 최대한 줄여라.**

공유 자원은 가능하면 피하고, 꼭 필요하다면:

- `synchronized` 등으로 임계영역을 보호하라.
- 임계영역의 범위를 최소화하라.

공유 자원을 수정하는 코드가 많을수록:

- 임계영역 보호를 빼먹기 쉽다.
- 모든 임계영역을 올바르게 보호했는지 확인하는 데 많은 노력이 든다.
- 찾기 어려운 동시성 버그는 더욱 찾기 어려워진다.

### 따름 정리: 자료 사본을 사용하라

가능한 경우, **공유하지 말고 복사본을 만들어 사용**하라.

- 읽기 전용 객체는 복사해 사용.
- 각 스레드가 자신만의 사본을 만들어 독립적으로 작업.
- 결과가 필요하면 한 스레드가 사본에서 최종 결과를 모은다.

예시
- 불변 객체(Immutable Object) 사용.
- 복사 생성자, 팩토리 메서드를 통한 복사.
- Thread-Local Storage.

### 따름 정리: 스레드는 가능한 독립적으로 구현하라

- 스레드 간 자료 공유를 피하라.
- 각 스레드는 자신만의 데이터를 처리하도록 하라.
- 로컬 변수만 사용하도록 설계하라.

예시: 서블릿의 `doGet()`/`doPost()` 메서드

- `HttpServlet`의 `doGet()`과 `doPost()`는 요청별로 새로운 스레드에서 실행되며, 로컬 변수만 사용한다면 **동기화 문제가 발생할 여지가 없다.**
- 각 요청 스레드가 **독립적인 클라이언트 요청**을 처리하고, 필요한 데이터는 **비공유 출처**에서 가져와 **로컬 변수**에 저장.
- 결과적으로 스레드는 독립적이고 안전하게 동작한다.

 

## 라이브러리를 이해하라

👉 **언어가 제공하는 클래스를 검토하라.** 

1. 스레드 환경에 안전한 컬렉션을 사용한다. 
    - Java 5 이상의 `java.util.concurren` 패키지
        - 이 컬렉션들은 다중 스레드 환경에서도 안전하게 동작하며, 성능까지 고려해 설계됨.
2. 서로 무관한 작업을 수행할 때는 executor  프레임워크를 사용한다.
    - `ExecutorService`, `ScheduledExecutorService`
    - 장점
        - 스레드 생성, 관리, 종료를 추상화.
        - 작업 큐 관리, 스레드 재사용, 에러 처리 등을 간단하게 처리.
3. 가능하다면 스레드가 차단되지 않는 방법을 사용한다.
    - 락 대신 비차단 알고리즘 (lock-free, CAS 기반 연산 등) 사용.
        - CAS (Compare-And-Swap) : CAS는 원자적(atomic) 연산 제공
            
            ```java
            AtomicInteger counter = new AtomicInteger(0);
            counter.compareAndSet(0, 1); 
            // counter가 0이면 1로 변경
            ```
            
        - 락을 사용하지 않고도 안전하게 변수 변경 가능. 실패하면 다시 시도
    - 동시성 유틸리티 (`Atomic*` 클래스, `LongAdder` 등) 활용.
    - 불필요한 대기를 줄이면 성능과 응답성이 향상됨.
4. 일부 클래스 라이브러리는 스레드에 안전하지 못하다.
    - `ArrayList`, `HashMap`, `SimpleDateFormat` 등.
    - 이런 클래스들을 사용할 때는:
        - 외부에서 동기화를 직접 구현하거나,
        - 가능하면 스레드 안전한 대안으로 교체 (`ConcurrentHashMap`, `DateTimeFormatter` ).

### 스레드 환경에 안전한 컬렉션

👉 **언어가 제공하는 클래스를 검토하라.** 

- Java는 **동시성 라이브러리**를 제공한다.
- 직접 구현하기 전에:
    - **표준 라이브러리**에 유사 기능이 있는지 확인.
    - 성능, 안전성, 유지보수 측면에서 **검증된 라이브러리**를 우선 사용.

## 실행 모델을 이해하라

**동시성 유형**

| 문제 | 설명 |
| --- | --- |
| 한정된 자원(Bound Resource) | 다중 스레드 환경에서 사용되는 크기나 수량이 제한된 자원은 기다림, 블로킹, 경쟁 조건 등을 유발 가능 (데이터베이스 연결, 읽기/쓰기 버퍼) |
| 상호 배제 (Mutual Exclusion) | 한 번에 한 스레드만 공유 자원/자료에 접근 가능. |
| 기아 (Starvation) | 특정 스레드가 자원을 계속 얻지 못해 영원히 기다림.예: 짧은 작업에 우선순위를 주면 긴 작업은 계속 기다리게 됨. |
| 데드락 (Deadlock) | 여러 스레드가 서로의 자원을 기다리며 진행 불가.모든 스레드가 멈춘 상태. |
| 라이브락 (Livelock) | 각 스레드가 진행하려 하지만 서로 양보/회피하다가 계속 교착 상태에 빠짐.진행은 되지만 아무것도 끝나지 않음. |

### 생산자-소비자Producer-Consumer

- 생산자 스레드: 데이터를 생성해 대기열(버퍼)에 넣음.
- 소비자 스레드: 대기열에서 데이터를 꺼내 처리.
- 문제
    - 생산자는 **버퍼가 가득 차면** 대기해야 함.
    - 소비자는 **버퍼가 비면** 대기해야 함.
    - 생산자와 소비자가 서로 시그널을 잘못 처리하면 **둘 다 기다리는 상태**에 빠질 수 있음.
    

### **읽기-쓰기**

다수의 **읽기 스레드**는 공유 자원 읽기 가능하지만 하나의 **쓰기 스레드**는 공유 자원을 갱신한다고 할 때

- 문제
    - 처리율을 강조하면 기아 현상이나 오래된 정보가 쌓이고, 갱신을 허용하면 처리율에 영향을 미친다.
    - 처리율과 일관성의 균형 필요.
- **해결 방법**
    1. **읽기 우선**
        - 쓰기 스레드는 읽기 스레드가 모두 끝나야 실행 가능.
        - 쓰기 스레드 기아(starvation) 위험.
    2. **쓰기 우선**
        - 읽기 스레드는 쓰기 스레드가 끝나야 실행 가능.
        - 쓰기 스레드가 연속 실행되면 전체 처리율 저하.

### **식사하는 철학자들**

N명의 철학자가 식탁에 앉아 있다. 각 철학자는 양쪽 포크를 모두 잡아야 식사를 할 수 있다. 인접 철학자가 포크를 들고 있다면 기다려야 한다.

여러 프로세스가 자원을 얻으려 경쟁하기 때문에 주의해서 설계하지 않으면 문제가 발생한다.

**발생 가능한 문제**

- 데드락: 서로 포크를 기다리며 멈춤.
- 라이브락: 포크를 놓았다 잡았다 반복하며 진행 못함.
- 처리율 저하, 기아 등.

--- 
👉 **위에서 설명한 기본 알고리즘과 각 해법을 이해하라.**

동시성 문제를 단순히 "락 걸기"로만 해결하려 하지 말고:

- **자원 관리**의 특성과 제한 이해.
- **데드락, 기아, 라이브락** 가능성 고려.
- 표준 알고리즘/패턴 적용:
    - 생산자-소비자 → BlockingQueue, Condition 사용.
    - 읽기-쓰기 → ReadWriteLock.
    - 식사하는 철학자 → 리소스 순서 지정, 타임아웃 도입 등

을 고려해본다.

## 동기화하는 메서드 사이에 존재하는 의존성을 이해하라

👉 **원칙: 공유 객체 하나에는 동기화 메서드 하나만 사용하라.**

공유 클래스의 여러 동기화 메서드가 서로 의존하면, 예상치 못한 문제가 발생할 수 있다. 멀티스레드 환경의 버그는 재현이 어렵고, 디버깅이 매우 힘들기 때문에 공유 객체는 하나의 메서드만 외부에서 접근 가능하도록 제한한다.

**공유 객체 하나에 여러 메서드가 필요한 상황이라면?**

| 방법 | 설명 |
| --- | --- |
| 클라이언트에서 잠금 | 클라이언트가 첫 번째 메서드 호출 전에 락을 획득하고, 마지막 메서드 호출 후 락 해제. |
| 서버에서 잠금 | 서버(공유 객체)에서 여러 메서드를 묶어서 락을 걸고 하나의 통합 메서드 제공. |
| 연결 서버 | 잠금을 담당하는 중간 계층(Wrapper, Facade)을 만들어서 관리. 서버 코드 수정 최소화. |

## 동기화하는 부분을 작게 만들어라

같은 락으로 감싼 모든 코드 영역은 한 번에 한 스레드만 실행이 가능하다.

`synchronized`로 감싼 코드가 클수록:

- 락 대기 시간이 증가
- 스레드 간 병목 현상 발생

해결 방법

- 임계영역(Critical Section)은 최소화
    
    ```java
    // 불필요한 전체 메서드 락
    public synchronized void update() {
        somethingLong(); // 오래 걸리는 작업
        sharedList.add("data"); // 실제 임계영역
    }
    
    // 동기화 구간 최소화
    public void updateData() {
        somethingLong(); // 동기화 필요 없음
        synchronized (sharedList) {
            sharedList.add("data");
        }
    }
    ```
    

## 올바른 종료 코드는 구현하기 어렵다

👉 **종료 코드는 초기에 설계한다.**

종료 처리는 나중에 붙이는 게 아니라 처음부터 고려하고, 동작하는 형태로 작성해야 한다.


스레드 종료 처리는 단순히 종료 신호를 보내는 것으로 끝나지 않는다.

**잘못된 종료 코드의 문제**

- 데드락 (Deadlock)
- 무한 대기 (스레드가 신호를 못 받아 영원히 블로킹)
- 자원 해제 누락 (락, 파일, DB 연결 등)

 **부모-자식 스레드 종료 문제**

부모 스레드가 여러 자식 스레드를 생성 → 자식 스레드 종료를 기다림 → 자원 해제 후 종료

- 문제
    - 자식 스레드 중 하나가 데드락 상태 → 부모는 영원히 대기 → 프로그램 종료 불가
    

**생산자-소비자 종료 문제**

- 소비자 스레드가 생산자의 시그널(데이터) 을 기다림
- 부모 스레드가 종료 신호를 보내도, 소비자는 생산자의 데이터가 없으므로 차단 상태
- 결국 소비자는 종료 신호를 못 받고, 생산자도 소비자가 데이터를 가져갈 때까지 차단
    
    👉 데드락 발생
    


## 스레드 코드 테스트하기

- 말이 안 되는 실패는 잠정적인 스레드 문제로 취급하라
- 다중 스레드를 고려하지 않은 순차 코드부터 제대로 돌게 만들자
- 다중 스레드를 쓰는 코드 부분을 다양한 환경에 쉽게 끼워 넣을 수 있도록 스레드 코드를 구현하라
- 다중 스레드를 쓰는 코드 부분을 상황에 맞춰 조정할 수 있게 작성하라
- 프로세서 수보다 많은 스레드를 돌려보라
- 다른 플랫폼에서 돌려보라
- 코드에 보조 코드를 넣어 돌려라. 강제로 실패를 일으켜라.

### 말이 안 되는 실패는 잠정적인 스레드 문제로 취급하라

- 다중 스레드 코드는 때때로 말이 안 되는 오류를 일으킨다.
- 스레드 버그는 수백만 번에 한 번씩 드러나기도 한다.
- 실패를 재현하기는 아주 어렵다.
- 일회성 문제를 무시하지 말자

### 다중 스레드를 고려하지 않은 순차 코드부터 제대로 돌게 만들자

- 스레드 환경 밖에서 생기는 버그와 스레드 환경에서 생기는 버그를 동시에 디버깅하지 않고 스레드 환경 밖에서 코드가 제대로 도는지 반드시 확인한다.
- 스레드가 호출하는 **POJO**를 만든다. POJO는 스레드를 모르기 때문에 스레드 환경 밖에서 테스트 가능하다.

### 다중 스레드를 쓰는 코드 부분을 다양한 환경에 쉽게 끼워 넣을 수 있게 스레드 코드를 구현하라

- 한 스레드로 실행하거나, 여러 스레드로 실행하거나, 실행 중 스레드 수를 바꿔본다.
- 스레드 코드를 실제 환경이나 테스트 환경에서 돌려본다.
- 테스트 코드를 빨리, 천천히, 다양한 속도로 돌려본다.
- 반복 테스트가 가능하도록 테스트 케이스를 작성한다.
- 다양한 설정에서 실행한 목적은 다른 환경에 쉽게 끼워 넣을 수 있게 만드는 것.

### 다중 스레드를 쓰는 코드 부분을 상황에 맞게 조율할 수 있게 작성하라

- 적절한 스레드 개수를 파악하려면 시행착오가 필요하다.
- 다양한 설정으로 프로그램의 성능을 측정하는 방법을 강구하라.
- 스레드 개수를 조율하기 쉽게 코드를 구현하라.
- 프로그램이 돌아가는 도중에 스레드 개수를 변경하는 방법도 고려하라.
- 프로그램 처리율과 효율에 따라 스스로 스레드 개수를 조율하는 코드도 고민하라.

### 프로세서 수보다 많은 스레드를 돌려보라

- 시스템이 스레드를 스와핑할 때 문제가 발생할 수 있다.
- 스와핑을 일으키려면 프로세서 수보다 많은 스레드를 돌려라.
- 스와핑이 잦을수록 임계영역을 빼먹은 코드나 데드락을 일으키는 코드를 찾기 쉬워진다.

### 다른 플랫폼에서 돌려보라

- 다중 스레드 코드는 플랫폼에 따라 다르게 돌아간다.
- 코드가 돌아갈 가능성이 있는 플랫폼 전부에서 테스트를 수행해야 한다.

### 코드에 보조 코드를 넣어 돌려라. 강제로 실패를 일으키게 해보라

- 스레드 코드는 간단한 테스트로는 버그가 드러나지 않는다.
- 스레드 버그는 실패하는 경로가 실행될 확률이 극도로 저조하다.
- 버그를 발견하고 찾아내기가 아주 어렵다.
- 보조 코드를 추가해 코드가 실행되는 순서를 바꿔 코드의 다양한 순서를 실행하라.
    - wait(), sleep(), yield(), priority()…
    - 각 메서드는 스레드가 실행되는 순서에 영향을 미치고 버그가 드러날 가능성도 높아진다. 잘못된 코드라면 가능한 초반에 가능한 자주 실패하는 편이 좋다.
    

**코드에 보조 코드를 추가하는 방법**

- **직접 구현**
    - 코드에다 직접 wait(), sleep(), yield(), priority() 메서드를 추가한다.
        
        ```java
        public synchronized String nextUrlOrNull() {
            if (hasNext()) {
                String url = urlGenerator.next();
                Thread.yield(); // 테스트를 위해 추가
                updateHasNext();
        
                return url;
            }
            
            return null;
        }
        ```
        
    - 문제
        - 보조 코드를 삽입할 적정 위치를 직적 찾아야 한다.
        - 어던 함수를 어디서 호출해야 적당한지 어떻게 알지?
        - 배포 환경에 보조 코드를 그대로 남겨두면 프로그램 성능이 떨어진다.
        - 무작위적이다. 오류가 드러날지도 모르고 드러나지 않을지도 모른다.
        - 실행할 때마다 설정을 바꿔줄 방법도 필요하다. 그래야 전체적으로 오류가 드러날 확률이 높아진다.
- **자동화**
    - AOF, CGLIB, ASM 등 도구를 사용해 자동화할 수 있다.
        
        ```java
        public class ThreadJigglePoint {
            public static void jiggle() {
        			// sleep, yield, nop 무작위 수행
            }
        }
        
        public synchronized String nextUrlOrNull() {
            if (hasNext()) {
                ThreadJigglePoint.jiggle();
                String url = urlGenerator.next();
                ThreadJigglePoint.jiggle();
                updateHasNext();
                ThreadJigglePoint.jiggle();
        
                return url;
            }
        
            return null;
        }
        ```
        
        - `ThreadJigglePoint.jiggle()` 호출은 무작위로 `sleep()`, `yield()`, `nop`(아무 동작 안 함) 중 하나를 수행한다.
        - `jiggle()` 메서드를 비워두고 배포 환경에서 사용한다.
        - 무작위로 `nop`, `sleep`, `yield` 등을 테스트 환경에서 수행한다.

### 결론

- 다중 스레드 코드는 올바르게 구현하기 어렵다.
- 다중 스레드 코드를 작성한다면 각별히 깨끗하게 코드를 짜야 한다.
- SRP(단일 책임 원칙)를 준수하여 스레드를 아는 코드와 스레드를 모르는 코드로 분리한다.
- 스레드 코드를 테스트할 때는 전적으로 스레드만 테스트한다.
- 스레드 코드는 최대한 집약되고 작아야 한다.
- 동시성 오류를 일으키는 잠정적인 원인을 철저히 이해해야 한다.
- 사용하는 라이브러리의 기본 알고리즘을 이해해야 한다.
- 보호할 코드 영역을 찾아내는 방법과 코드 영역을 잠그는 방법을 이해해야 한다.
- 초반에 드러나지 않는 문제를 일회성으로 치부해 무시하면 안 된다.
- 스레드 코드는 많은 플랫폼에서 많은 설정으로 반복해서 계속 테스트해야 한다.
- 보조 코드를 추가해 오류가 드러날 가능성을 크게 높인다.


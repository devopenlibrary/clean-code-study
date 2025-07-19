---
nav_order: 18
---

# 동시성

## 클라이언트 / 서버 예제

클라이언트 - 서버 애플리케이션을 예시로 동시성에 대해 알아보자.

### 서버

아래와 같이 단일 스레드 방식의 클라이언트 - 서버 구조인 애플리케이션이 있다고 가정하자.

- 서버: 소켓을 열어놓고 클라이언트가 연결하기를 기다림

    ```java
    ServerSocket serverSocket = new ServerSocket(8009);
    
    while (keepProcessing) {
        try {
            Socket socket = serverSocket.accept();
            process(socket);
        } catch (Exception e) {
            handle(e);
        }
    }
    ```

- 클라이언트: 소켓에 연결해 요청을 보냄

    ```java
    private void connectSendReceive(int i) {
        try {
            Socket socket = new Socket("localhost", PORT);
            MessageUtils.sendMessage(socket, Integer.toString(i));
            MessageUtils.getMessage(socket);
            socket.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    } 
    ```

- 테스트 코드
: 이때 이 코드가 잘 동작하는지 테스트 하기 위해, 아래 테스트 코드를 작성합니다.

    ```java
    @Test(timeout = 10000)
    public void shouldRunInUnder10Seconds() throws Exception {
        Thread[] threads = createThreads();
        startAllThreads(threads);
        waitForAllThreadsToFinish(threads);
    }
    ```
    - 테스트 케이스는 동시성 작업 처리 능력을 검증하는 전형적인 예시로, 10,000밀리초 내에 끝나기를 검사
    - 단일 스레드 서버는 한 번에 하나의 요청만 순차적으로 처리, 클라이언트 증가시 실패할 가능성이 매우 높음


### 스레드 추가하기

해결하기 위해 가장 먼저 떠올릴 수 있는 방법은, 각 클라이언트 요청을 별도의 스레드에서 처리하는 것

```java
ServerSocket serverSocket = new ServerSocket(8009);

while (keepProcessing) {
    try {
        Socket socket = serverSocket.accept();
        // 요청이 들어올 때마다 새로운 스레드를 생성해 처리
        new Thread(() -> process(socket)).start();
    } catch (Exception e) {
        handle(e);
    }
}
```

: 서버는 클라이언트 요청을 받자마자 새 스레드에 작업을 위임 -> 바로 다음 연결을 대기함 
=> 여러 클라이언트의 요청을 동시에 처리 가능함

> - I/O 바운드: 소켓 사용, DB연결, 가상 메모리 스와핑 등 `대기`하는 작업이 많음 -> 동시성이 성능을 높일 수 있음
> - CPU 바운드: 수치 계산, 정규식 처리, 가비지 컬렉션 등 `연산`하는 작업이 많음 -> 하드웨어를 추가해 성능을 높일 수 있음

### 서버 살펴보기

- SRP 위반: 서버는 여러개의 책임을 지고 있음
  - 소켓 연결 관리
  - 클라이언트 처리
  - 스레드 정책
  - 서버 종료 정책

### 결론

단순히 스레드를 추가하는 것만으로는 견고한 동시성 시스템을 만들 수 없다. 이는 동시성이 야기하는 근본적인 문제들을 해결하지 못하기 때문이다. 깨끗한 동시성 코드를 작성하려면 실행 경로, 자원 공유, 데드락 등 더 깊은 이해가 필요하다.

## 가능한 실행 경로

### 경로 수

```java
public class IdGenerator{

  int lastIdUsed;
  
  public int incrementValue() {
    return ++lastIdUsed;
  }
}
```

스레드가 1개라면 동일한 결과가 보장되지만, n개라면  다양한 결과가 나올 수 있다.

그 이유는 가능한 실행 경로 수와 JVM의 동작 방식을 알아야 한다.

  ```java 
  return ++lastIdUsed;
  ```
  
  이 한줄은 바이트 코드 8개가 해당되어, 스레드가 여러 개라면 명령 8개를 뒤섞어 실행할 가능성이 충분하다.


### 가능한 순열 수 계산하기

- 위의 경우의 수

  $$ \text{경로 수} = \frac{(T \times N)!}{(N!)^T} $$

  - 단계가 N, 스레드가 T라면 총 단게는 N * T개
  - 정해진 순서를 위반하는 경우를 제거

- 코드를 변경한다면?

```java
public synchronized void incrementValue(){
  ++lastIdUsed;
}
```

가능한 경로의 수는 스레드 개수만큼으로 줄어든다. (단계가 사라짐)
-> 1개가 되지 않는다.

만약 


### 심층 분석

- 원자적 연산이란?
  - 중단이 불가능한 연산을 원자적 연산이라고 정의함
  - e.g. `lastId = 0;`
- 자료형을 변경한다면? int -> long
  - 만약 자료형을 변경시, `lastId = 0;` 또한 원자적 연산이 될 수 없음
- 전처리 증가 연산자(++)는?
  - 중단이 가능하기 때문에, 원자적 연산이 아님


- 바이트 코드를 보기전 알아야할 개념
  - 프레임: 호출 스택을 정의할 때 사용하는 표준 기법으로, 모든 메서드 호출에 는 프레임이 필요함
  - 지역 변수: 메서드 내에 정의되는 모든 변수, this는 현재 스레드에서 가장 퇴근에 메시지를 받아 메서들르 호출한 객체를 가리킴
  - 피연산자 스택: 매개변수를 저장하는 장소, LIFO 구조

- `synchronized`를 이용하면, `incrementValue()`는 해결됨
  - 동기화 되기 때문에 가능한 수(경로)는 여러개지만, 결과는 단 한개가 됨

## 라이브러리를 이해하라
### Executor 프레임워크
스레드와 관련된 프레임 워크로 `Executor`클래스와 `Future`를 이용해본다.   

### 스레드를 차단하지 않는 방법

먼저 적용한 `synchronized`를 이용한 방법은 락을 이용한 방법으로 스레드를 차단한다.
동시성 문제가 크지 않을 때라면 괜찮지만, 많다면 성능에 문제가 생길 수 있다.


> CAS (Compare-And-Swap) 연산
> 
> DB분야에서 낙관적 잠금이라는 개념과 유사, (동기화는 비관적 잠금 개념과 유사)
> 
> 3가지 값을 사용
> - 메모리의 현재 값 (V: Value): 지금 현재 공유 변수에 저장된 값입니다.
> - 예상되는 현재 값 (A: Expected Old Value): 스레드가 변수를 읽어와서 변경하려고 할 때 '내가 읽었을 때 이 값이 있었을 것이다'라고 예상하는 값입니다.
> - 새로운 값 (B: New Value): 스레드가 변수를 성공적으로 업데이트하고 싶은 새로운 값입니다.
> 

### 다중 스레드 환경에서 안전하지 않은 클래스

- 안전하지 않은 스레드
  - `SimpleDateFormat`
  - DB 연결
  - java.util 컨테이너 클래스
  - 서블릿

- 만약 아래 코드가 멀티스레드 환경에서 동작한다면

  ```java
  if(!hashTable.containKey(someKey)) {
    hashTable.put(somekey, new SomeValeu());
  }
  ```
  
  1. 클라이언트 기반 잠금: 사용하는 클라이언트가 잠금(`synchronized`)를 이용
  2. 서버 기반 잠금: 서버가 잠금(`synchronized`)를 이용
  3. 스레드에 안전한 집합 클래스를 사용


## 메서드 사이에 존재하는 의존성을 조심하라

- 의존성을 만드는 예제
  
  ```java
  public class IntegerIterator implements Iterator<Integer> {
      private Integer nextValue = 0;
  
      public synchronized boolean hasNext() {
          return nextValue < 100000;
      }
  
      public synchronized Integer next() {
          if (nextValue == 100000)
              throw new IteratorPastEndException();
          return nextValue++;
      }
  
      public synchronized Integer getNextValue() {
          return nextValue;
      }
  }
  ```
  
- 위 예제를 사용하는 코드

  ```java
  IntegerIterator iterator = new IntegerIterator();
  while(iterator.hasNext()) {
      int nextValue = iterator.next();
      // nextValue를 뭔가 한다.
  }
  ```
  

- 문제상황
  1. `nextValue`가 `99999`일 때, `Thread 1`이 `hasNext()`를 호출하면 `true`를 반환
  2. 그 사이에 `Thread 2`가 `next()`를 호출하여 `nextValue`를 `100000`으로 만들고, `IteratorPastEndException을` 발생
  3. `Thread 1`이 이어서 `next()`를 호출하면, 이미 `nextValue`가 `100000`이므로 `IteratorPastEndException을` 발생

### 실패를 용인한다

떄로는 실패를 해도 괜찮도록 할 수 있다.
e.g. 클라이언트가 예외를 받아 처리, 메모리 누수를 시간마다 재부팅하는 방식 

### 클라이언트 - 기반 잠금

  ```java
  IntegerIterator iterator = new IntegerIterator();
  
  while(true){
      int nextValue;
      synchronized (iterator){
        if(!iterator.hasNext())
          break;
          
        nextValue = iterator.next();
      }
      doSomethingWith(nextValue);
  }
  ```

  각 클라이언트가 `synchronized`를 이용해 `IntegerIterator`객체에 락을 건다.
  -> `IntegerIterator`를 이용하는 모든 프로그래머가 이를 기억해야함 (위험, 해결하기 어려움)

### 서버 - 기반 잠금


클라이언트에서 중복해서 락을 걸지 않아도 된다.

  ```java
  public class IntegerIteratorServerLocked {
      private Integer nextValue = 0;
  
      public synchronized Integer nextOrNull() {
          if(nextValue < 100000)
            return nextValue++;
          else
            return null;
      }
  }
  ```

- 더 바람직한 이유
  - 코드 중복이 줄어듬
  - 성능이 좋아짐: 단일 스레드 환경으로 시스템을 배치할 경우, 서버만 교체하면 오버헤드가 줄어든다.
  - 락 관리가 1곳이라, 위험이 줄어듬
  - 스레드 정책이 1개
  - 공유 변수 범위가 줄어듬

- 만약 서버 코드를 수정할 수 없다면? Adapter 패턴을 이용해 API를 변경한 후 잠금을 추가


## 작업 처리량 높이기

- `synchronized`의 영역은 언제나 작을수록 좋다.


가정
- 페이지를 읽어오는 평균 I/O 시간 : 1초
- 페이지를 분석하는 평균 처리 시간 : 0.5초
- 처리는 CPU 100% 사용, I/O는 CPU 0% 사용


### 작업 처리량 계산 - 단일 스레드 환경

만약 스레드 1개인 단일 스레드 환경에서, N개의 페이지를 처리한다면 => 실행시간은 총1.5초 * N 초


### 작업 처리량 계산 - 다중 스레드 환경

만약 순서와 무관하게 페이지를 독립적으로 읽어와 처리해도 된다면, 다중 스레드가 속도를 높이는데 도움이 될 수 있다.

3개의 스레드를 사용한다고 하면, I/O 작업이 발생하는 1초동안 2페이지의 내용을 처리할 수 있다.
즉 1초마다 2개의 페이지를 처리할 수있으므로, 단일 스레드 환경과 비교한다면 처리율을 3배다.

## 데드락

데드락을 발생 시키는 조건
- 상호 배제(Mutual exclusion)
- 잠금 & 대기(Lock & Wait)
- 선점 불가(No preemption)
- 순환 대기(Circular Wait)

### 상호배제

여러 스레드가 한 자원을 공유하면서,
- 동시에 사용하지 못하며
- 개수가 제한적이라면
조건을 만족한다.

e.g. DB 연결, 쓰기용 파일 열기, 레코드 락, 세마포어 등

### 잠금 & 대기

자원을 점유하면, **필요한 나머지 자원까지** 모두 점유한뒤 작업을 마칠 때까지 이미 점유한 자원을 내놓지 않음

### 선점 불가

한 스레드가 다른 스레드로부터 자원을 빼앗지 못함 -> 스스로 내놓지 않으면 다른 스레드는 그 자원을 점유하지 못함

### 순환 대기

> 죽음의 포옴
> 2개의 스레드 (T1, T2)가 2개의 공유 자원(R1,R2)가 있을 때,
> T1 - R1, T2 - R2 이렇게 점유 된 상황에서
> T1 -> R2, T2 -> R1f 이렇게 원하는 상황이라면 데드락이 발생한다.
> (서로 필요한 자원이 생기지 않아 자신의 자원을 놓지 않고 있으므로)

위 4개의 조건 중 하나라도 깨버린다면, 데드락은 발생하지 않는다.

### 상호 배제 조건 깨기

- 동시에 사용해도 괜찮은 자원을 사용
- `스레드 수 < 리소스 수` 가 되도록 리소스 증가
- 자원 점유 전, 필요한 모든 자원이 있는지 확인 후 자원 점유

-> 사용하기 어려울 수 있음

### 잠금 & 대기 조건 깨기

대기하지 않으면 데드락이 발생하지 않음, 각 자원 점유전에 확인해 어느 하나라도 점유하지 못하면 모든 리소스 반환

잠재적인 문제
- 기아: 한 스레드가 계속해서 자원을 점유하지 못함
- 라이브락: 여러 스레드가 한번에 잠금 단계로 진입, 모두가 자원을 점유했다 포기했다를 반복 -> CPU를 의미 없이 많이 사용하기도 함

### 선점 불가 조건 깨기

자원을 뺏어오자!

필요한 자원이 잠겨있다면, 풀어달라고 요청 
-> 요청을 받은 스레드: 다른 스레드의  자원을 대기하는 중이였다면, 잠금을 해재하고 처음부터 다시 기다림


-> 모든 요청을 관리하는 것이 어려움

### 순환 대기 조건 깨기

가장 흔한 전략

리소스 할당은 순서를 지정해, 순서에 맞게 할당 하도록 함

잠재적인 문제
- 자원을 할당하는 순서와 자원을 사용하는 순서가 다를지도 모른다.
  : T1이 일찍 받았지만, 바로 사용하지 않고 이후에 사용할 수 있음 -> 동시에 같은 리소스가 필요했던 T2는 더 오래 기다리게 됨
- 때로는 순서에 따라 자원을 할당하기 어려움

> 항상 모든 것은 트레이드 오프가 있고, 완벽한 해결책은 없다 
> 스레드 관련 코드를 분리하면, 조율과 실험이 가능해 최적의 전략을 찾기 쉬워진다.

## 다중 스레드 코드 테스트

다중 스레드를 사용하는 부분에서 동시성 문제가 있는지 테스트하는 것은 매우 어렵다.
발생할 조건을 만들어도 매우 드물게 발생하기 때문에 테스트를 많이 진행해야 한두번 나올까 말까

- 몬테 카를로 테스트 : 조율이 가능하게 유연한 테스트를 만들고, 임의로 값을 조절하며 테스트한다. -> 테스트가 실패한 조건은 신중하게 기록한다. 
- 시스템을 배치할 모든 곳에서 테스트를 실행한다.
- 부하가 변하는 장비에서 테스트를 실행한다.

## 스레드 코드 테스트를 도와주는 도구

conTest: IBM이 만든 도구로, 스레드에 안전하지 않은 코드에 보조 코드를 더해 실패할 가능성을 높여주는 도구

사용하는 방법
- 실제 코드와 테스트 코드를 작성
- conTest로 실제 코드와 테스트 코드에 보조 코드를 추가
- 테스트 실행

이전에는 천만~백만번에 한두번 나오던 실패가, 서른번에 한번정도로 실패

## 결론

다중 시스템을 구현하려면 알아야 할 내용이 더 많다.
**더그 리**의 `Concurrent Programming in Java: Design Principles and Patterns`를 추천한다.
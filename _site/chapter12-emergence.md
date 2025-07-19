# 창발성


- 주요 내용
  - 단순한 설계 규칙에 대해 설명
  - 중복을 삭제하는 방식
---

## 창발적 설계로 깔끔한 코드를 구현하자

**켄트 벡**이 말하는 단순하게 설계하는 규칙
1. 모든 테스트를 실행한다.
2. 중복을 없앤다.
3. 프로그래머 의도를 표현한다.
4. 클래스와 메서드 수를 최소로 줄인다.

우리는 위 규칙을 지키면, 소프트웨어 설계 품질을 크게 높여준다고 믿는다.

## 단순한 설계 규칙 1: 모든 테스트를 실행하라

테스트는 중요하다: 테스트의 중요성
- 가장 중요한 것: 설계는 의도한 대로 돌아가는 시스템, 검증이 불가능하다면 인정받기 어려움
- 검증 방법: 시스템의 테스트
- 테스트가 불가능한 시스템 = 검증할 수 없는 시스템

테스트 코드 작성의 이점: 테스트가 가능한 시스템을 만들기 위해 노력하다보면, 설계 품질이 좋아짐
- 설계 품질이 낮으면 테스트가 어렵기 떄문에, 테스트를 쉽게 작성하기 위해 설계 품질이 높아지는 방향으로 개발
    - DIP, 인터페이스, 추상화 등의 도구를 이용해 결합도를 낮추고 응집력을 높임 = 객체 지향 방법론이 지향하는 목표 
- 테스트 케이스가 많으면 테스트 코드를 쉽게 작성하게 되서, 더 다양한 테스트 코드를 작성하게 됨

## 단순한 설계 규칙 2~4: 리팩터링

테스트 케이스를 모두 작성했으면, 점진적으로 리팩토링을 진행

리팩토링을 하며 시스템이 망가질까 걱정할 이유가 없음
- 이미 작성되어있는 테스트 코드가 이를 확인할 수 있도록 하기 때문에

## 중복을 없애라

중복의 위험성
- 중복은 추가작업, 추가 위험, 불필요한 복잡도를 뜻함

중복은 여러 형태로 표출됨: 구현 중복도 중복

단 몇 줄이라도 중복을 제거하겠다는 의지가 필요함

중복을 제거한 예시
- 구현 중복은 제거
  ```java
  int size(){
    ...
  }
  
  boolean isEmpty(){
      return 0 == size();
  }
  ```
- 다른 코드 중복 제거 예시
  - 기존
    ```java
    public void scaleToOneDimension(float desiredDimension, float imageDeimension){
      if(Math.abs(desiredDimension - imageDeimension) < errorThreshold)
        return;
      float scalingFactor = desiredDimension / imageDeimension;
      scalingFactor = (float) (Math.floor(scalingFactor * 100) * 0.01f);
    
      RenderedOp newImage = ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor);
      image.dispose();// 동일한 부분
      System.gc();
      image = newImage;
    }
    
    public synchronized void rotate(int degrees){
      RenderedOp newImage = ImageUtilities.getRotatedImage(image, degrees); 
      image.dispose();// 동일한 부분
      System.gc();
      image = newImage;
    }
    ```
  - 변경
    ```java
    public void scaleToOneDimension(float desiredDimension, float imageDeimension){
      if(Math.abs(desiredDimension - imageDeimension) < errorThreshold)
        return;
      float scalingFactor = desiredDimension / imageDeimension;
      scalingFactor = (float) (Math.floor(scalingFactor * 100) * 0.01f);
    
      replaceImage( ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor) );
    }
    
    public synchronized void rotate(int degrees){
      replaceImage( ImageUtilities.getRotatedImage(image, degrees) ); 
    }
    
    private void replaceImage(RenderedOp newImage){ // 공통 로직 메서드 추출
      RenderedOp newImage = ImageUtilities.getRotatedImage(image, degrees); 
      image.dispose();
      System.gc();
      image = newImage;
    }
    ```
  - 특징
    - 위와 같은 소규모 재사용은 시스템 복잡도를 극적으로 줄여줌
    - 소규모 재사용을 제대로 익혀야 대규모 재사용이 가능함

Template method 패턴
- Template method 패턴은 고차원 중복을 제거할 목적으로 자주 사용됨
- 기존
  ```java
  public class VacationPolicy{
    public void accrueUSDivisionVacation(){ // 미국 버전
      // 지금까지 근무한 시간을 바탕으로 휴일 일수를 계산하는 코드
      // 휴가 일수가 미국 최소 법정 일수를 만족하는지 확인하는 코드
      // 휴가 일수를 급여 대장에 적용하는 코드
    }
  
    public void accrueEUDivisionVacation(){ // 유럽 버전
      // 지금까지 근무한 시간을 바탕으로 휴일 일수를 계산하는 코드
      // 휴가 일수가 유럽연합 최소 법정 일수를 만족하는지 확인하는 코드
      // 휴가 일수를 급여 대장에 적용하는 코드
    }
  }
  ```
- 변경
  ```java
  abstract public class VacationPolicy{
    public void accrueVacation(){ // 공통 버전
      a(); // 지금까지 근무한 시간을 바탕으로 휴일 일수를 계산하는 함수 정의
      b(); // 휴가 일수가 최소 법정 일수를 만족하는지 확인하는 코드 -> 변경 부분이 있는 곳만 abstract으로 처리 
      c(); // 휴가 일수를 급여 대장에 적용하는 코드
    }
  }
  
  public USVacationPolicy extends VacationPolicy{ // 미국 버전으로
    // 휴가 일수가 최소 법정 일수를 만족하는지 확인하는 코드 
    @Override
    b(){ // abstract 부분만 구현
      ...
    }
  }
  
  public EUVacationPolicy extends VacationPolicy{ // 미국 버전으로
    // 휴가 일수가 최소 법정 일수를 만족하는지 확인하는 코드 
    @Override
    b(){ // abstract 부분만 구현
      ...
    }
  }
  ```
  
## 표현하라

코드는 개발자의 의도를 분명이 표현해야함
- 소프트웨어 프로젝트 비용 중 대다수는 장기적인 유지보수에 투자되는데, 유지보수 개발자가 제대로 이해하기 위해서

코드 표현력을 높이는 방법: 가장 좋은 방법은 노력하는 것
- 좋은 이름을 선택
- 함수와 클래스 크기는 가능한 줄이기
- 표준 명칭 사용(e.g. 표준 패턴 사용시, 클래스 이름에 넣음)
- 단위 테스트 케이스를 꼼꼼히 작성

## 클래스와 메서드 수를 최소로 줄여라

가능한 독단적인 견해는 멀리하고 실용적인 방법을 택한다.
- 중복 제거, 의도 표현, SRP 등의 기본적인 개념도 극단으로 치달으면 득보다 실이 커짐
- 하지만 위의 방법보다는 우선순위가 낮다.
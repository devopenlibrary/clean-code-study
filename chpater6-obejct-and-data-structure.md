---
nav_order: 6
---

# 6장 객체와 자료구조

변수를 비공개(private)로 정의하는 이유는 남들이 변수에 의존하지 않게 만들고 싶어서이다. 하지만, 어째서인지 **수많은 프로그래머들은 조회(get)함수 와 설정(set)함수를 당연하게 공개해 비공개 변수를 외부에 노출**한다.

## 자료 추상화
다음 목록 6-1, 6-2 클래스를 살펴보자. 두 클래스 모두 2차원 점을 표현하지만 한 클래스는 구현을 외부로 노출하고, 다른 클래스는 구현을 완전히 숨긴다.

```java
// 목록 6-1 구체적인 Point 클래스
public class Point {
    public double x;
    public double y;
}
```

```java
// 목록 6-2 추상적인 Point 클래스
public class Point {
    double getX();
    double getY();
    void setCartesian(double x, double y);
    double getR();
    double getTheta();
    void setPolar(double r, double theta);
}
```
6-2는 점이 직표좌표계를 사용하는지, 극좌표계를 사용하는지 알 길이 없다. 하지만, 점을 표현하는 자료 구조라는 점은 명확하게 안다. 
> 직교좌표계와 극좌표계는 2차원 평면상의 점을 표현하는 두 가지 다른 방식. 직교좌표계는 점을 두 개의 직선 거리(x, y)로 표현하고, 극좌표계는 거리(r)와 각도(θ)로 표현한다. 

**즉, 인터페이스를 통해 자료구조는 알지만 어떤 방식으로 구현했는 지 내부 동작에 관련해서는 알수 없도록 숨긴 것이다.**

사실 6-2 는 자료 구조 그 이상을 표현한다. **클래스 메서드가 접근 정책을 강제**한다. 예를 들면, 좌표를 읽을 때는 각 값을 개별적으로 읽어야하지만 좌표를 설정할 때는 두 값을 한거번에 설정해야 한다.

반면 6-1은 확실히 직교좌표계를 사용한다는 것을 알 수 있다. 구현을 노출하는 것이다. 또한 개별적으로 좌표값을 읽고 설정하게 강제한다. **만약 변수를 private하게 바꾸더라도, 각 값마다 조회/설정함수를 제공한다면 구현을 외부로 노출하는 셈이다.**

이처럼 변수 사이에 함수라는 계층을 넣는다고 구현이 저절로 감춰지지 않는다. 구현을 감추기 위해 필요한 것은 **추상화**이다. **추상 인터페이스를 통해 사용자가 구현을 모른 채 자료 구조의 핵심을 조작할 수 있어야 진정한 의미의 클래스다.**

마지막으로 목록 6-3과 목록 6-4를 살펴보자. 

목록 6-3은 자동차 연료 상태를 **구체적인 숫자 값**으로 알려준다. 해당 값이 변수값을 읽어 반환한 값이라는 사실이 거의 확실하다. 

반면, 목록 6-4는 자동차 연료 상태를 백분율이라는 **추상적인 개념**으로 알려준다. 이 정보는 어디서 오는지 전혀 드러나지 않는다.

```java
// 목록 6-3 구체적인 Vehicle 클래스
public interface Vehicle {
    double getFuelTankCapacityInCallons();
    double getGallonsOfGasoline();
}
```

```java
// 목록 6-4 추상적인 Vehicle 클래스
public interface Vehicle {
    double getPercentFuelRemaining();
}
```

목록 6-1과 목록 6-2에서는 목록 6-2가, 목록 6-3과 목록 6-4에서는 목록 6-4가 더 좋다. 자료를 세세하게 공개하지않고 추상적인 개념으로 표현하기 때문이다.

인터페이스나 조회/설정 함수만으로는 추상화가 이뤄지지 않는다. **개발자는 객체가 포함하는 자료를 표현할 가장 좋은 방법을 심각하게 고민해야한다.** 아무 생각 없이 조회/설정 함수를 추가하는 방법이 가장 나쁘다.

## 자료/객체 비대칭
앞선 두 가지 예제(목록 6-1/목록 6-2, 목록 6-3/목록 6-4)는 **객체와 자료 구조 사이에 벌어진 차이**를 보여준다. 

- **객체**는 추상화 뒤로 **자료를 숨긴 채** 자료를 다루는 **함수만 공개**한다. 
- **자료구조**는 **자료를 그대로 공개**하며 별다른 **함수는 제공하지 않는다**.

**두 정의는 본질적으로 상반되며, 차이를 만든다.** 절차적인 코드인 6-5와 객체지향 코드인 목록 6-5의 예제를 통해 살펴보자. 

목록 6-5는 절차적인 도형클래스이다. `Geometry 클래스`는 세 가지 도형 클래스를 다룬다. 각 도형 클래스는 간단한 자료구조다. 즉, 아무 메서드도 제공하지 않는다. 도형이 동작하는 방식은 `Geometry 클래스`에서 구현한다.

```java
// 목록 6-5 절차적인 도형 클래스
public class Square {
    public Point topLeft;
    public double side;
}

public class Rectangle {
    public Point topLeft;
    public double height;
    public double width;
}

public class Circle {
    public Point center;
    public double radius;
}

public class Geometry {
    public final double PI = 3.141592653589793;
    
    public double area(Object shape) throws NoSuchShapeException {
        if (shape instanceof Square) {
            Square s = (Square) shape;
            return s.side * s.side;
        }
        else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle) shape;
            return r.height * r.width;
        }
        else if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return PI * c.radius * c.radius;
        }
        throw new NoSuchShapeException();
    }
}
```

객체 지향 프로그래머가 위 코드를 본다면 클래스가 절차적이라고 비판할수도 있다. 하지만 그것이 100% 옳다고 말하긴 어렵다.

만약 `Geometry 클래스`에 둘레 길이를 구하는 `perimeter()` 라는 함수를 추가한다고 치자. 이때 세개의 **도형 클래스들은 아무런 영향을 받지 않는다.** 도형 클래스에 의존하는 다른 클래스도 마찬가지이다.

```java
/*...*/

public class Geometry {
    public double area(Object shape) throws NoSuchShapeException {
        /* ... */
    }
    
    public double perimeter(Object shape) throws NoSuchShapeException {
        /*...*/
    }
}
```

하지만 반면 **새 도형 클래스를 추가하고 싶다면, `Geometry 클래스`에 속한 함수를 모두 고쳐야 한다.**

```java
/*...*/

// 새 도형 클래스 추가
public class Pentagon {
    /*...*/
}

public class Geometry {
    public double area(Object shape) throws NoSuchShapeException {
        /*...*/

        // 함수 수정
        else if (shape instance of Pentagon){
            /*...*/
        }
    }

    public double perimeter(Object shape) throws NoSuchShapeException {
        /*...*/
        else if (shape instance of Pentagon){
            /*...*/
        }
    }
}
```

이번에는 객체 지향적인 코드인 목록 6-6을 살펴보자. `area()`는 다형 메서드다. 

```java
// 목록 6-6 다형적인 도형
public class Square implements Shape {
    private Point topLeft;
    private double side;
    
    public double area() {
        return side * side;
    }
}

public class Rectangle implements Shape {
    private Point topLeft;
    private double height;
    private double width;
    
    public double area() {
        return height * width;
    }
}

public class Circle implements Shape {
    private Point center;
    private double radius;
    public final double PI = 3.141592653589793;
    
    public double area() {
        return PI * radius * radius;
    }
}
```

`area()`는 다형 메서드이므로 `Geometry 클래스`는 필요없다. 따라서 **새 도형 클래스를 추가해도 기존 함수에 아무런 영향을 미치지 않는다.**

```java
public class Square implements Shape {
    /*...*/
}

/**
.
.
.
*/

public class Pentagon implements Shape {
    public double area() {
        /*...*/
    }
}
```

하지만 반면 **새 함수를 추가하고 싶다면 도형 클래스 전부를 고쳐야한다.**

```java
public class Square implements Shape {
    /*...*/

    public double perimeter(){
        /*...*/
    }
}

public class Rectangle implements Shape {
    /*...*/

    public double perimeter(){
        /*...*/
    }
}

public class Circle implements Shape {
    /*...*/

    public double perimeter(){
        /*...*/
    }
}
```

앞서도 말했듯이, 목록 6-5와 목록 6-6은 객체와 자료구조 사이의 양분되는 특질을 보여준다.

- (자료구조를 사용하는) 절차적인 코드는 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다. 반면, 객체 지향 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다.

- 절차적인 코드는 새로운 자료 구조를 추가하기 어렵다. 그러려면 모든 함수를 고쳐야 하기 한다. 객체 지향 코드는 새로운 함수를 추가하기 어렵다. 그러려면 모든 클래스를 고쳐야한다.

다시 말해, 객체 지향 코드(객체 중심의)에서 어려운 변경은 절차적인 코드(자료 구조 중심의)에서 쉬우며, 절차적인 코드에서 어려운 변경은 객체 지향 코드에서 쉽다.

분별 있는 프로그래머라면 모든 것이 객체라는 생각이 미신임을 잘 안다. 때로는 단순한 자료 구조와 절차적인 코드가, 때로는 객체 지향 코드가 적합한 경우들이 존재한다.

## 디미터 법칙
디미터 법칙은 잘 알려진 휴리스틱으로, 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙이다. 

앞 절에서 봤듯이, 객체는 자료를 숨기고 함수를 공개한다. 즉, 객체는 조회 함수로 내부 구조를 공개하면 안된다는 의미다. 디미터 법칙은 좀 더 정확히 표현하자면, _"클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다"_ 고 주장한다.

- 클래스 C
- f가 생성한 객체
- f 인수로 넘어온 객체
- C 인스턴스 변수에 저장된 객체

하지만 위 객체에서 허용된 메서드가 반환하는 객체의 메서드는 호출하면 안된다.

다음 코드는 디미터 법칙을 어기는 듯 보인다. 객체 `ctxt`의 함수인 `getOptions()` 함수가 반환하는 객체의 `getScratchDir()` 함수를 호출한 후, 또 그 함수가 반환하는 객체의 `getAbsolutePath()` 함수를 호출하기 때문이다.

```java
// 임시 디렉터리의 절대 경로를 얻는 코드
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

### 기차 충돌
위와 같은 코드를 기차 충돌이라 부른다. 여러 객차가 한 줄로 이어진 기차처럼 보이기 때문이다. 일반적으로 조잡하다 여겨지는 방식이므로 피하는 편이 좋다.

따라서 위 코드는 다음과 같이 나누는 편이 좋다.

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

이로써 기차 충돌 문제는 해결했지만, 디미터 법칙을 여전히 위반하는지 알 수없다.

그 여부는 `ctxt`, `options`, `scratchDir`이 객체인지 아니면 자료 구조인지에 달렸다. 객체라면 내부 구조를 숨겨야 하기에 디미터 법칙을 위반하게 되고, 자료 구조라면 내부 구조를 노출하므로 디미터 법칙이 적용되지 않는다. 

그런데 위 예제는 조회 함수(get)를 사용하는 바람에 혼란을 일으킨다. 위 예제를 다음과 같이 구현했다면 디미터 법칙을 거론할 필요가 없어진다.

```java
final String ouputDir = ctxt.options.scratchDir.absolutePath;
```

자료 구조는 무조건 함수 없이 공개 변수만 포함하고 객체는 비공개 변수와 공개 함수를 포함하면 훨씬 간단해진다.

### 잡종 구조
위와 같은 혼란은 때때로 절반은 객체, 절반은 자료 구조인 잡종 구조를 야기한다. 잡종 구조에는 중요한 기능을 수행하는 함수도 있고, 공개 변수나 공개 조회/설정 함수도 있다.

이런 잡종 구조는 새로운 함수는 물론, 새로운 자료 구조도 추가하기 어렵다. 그러므로 잡종 구조는 되도록 피하는 편이 좋다. 잡종 구조는 프로그래머가 함수나 타입을 보호할지 공개할지 확신하지 못해(더 나쁘게는 무지해) 어중간하게 내놓은 설계에 불과하다.

### 구조체 _감추기_
만약 `ctxt`, `options`, `scratchDir`이 진짜 객체라면 내부 구조를 감춰야 하기 때문에, 앞선 코드 예제처럼 줄줄이 사탕으로 엮어서는 안된다. 그렇다면 어떤 방식으로 임시 디렉터리의 절대 경로를 얻는 이 코드를 변경할 수 있을까?

다음 두 코드를 살펴보자.

```java
ctxt.getAbsolutePathOfScratchDirectoryOption();


ctxt.getScratchDirectoryOption().getAbsolutePath();
```

첫 번째 방법은 `ctxt` 객체에 공개해야하는 메서드가 너무 많아진다. 

두 번째 방법은 `getScratchDirectoryOption()`이 객체가 아닌, 자료 구조를 반환한다고 가정한다. 

두 변경 방법 모두 내부 구조를 드러내는 방향이기에 디미터 법칙을 위반하게된다. 객체의 메서드에게는 **뭔가를 하라고** 말해야지 속을 드러내라고 말하면 안된다.

다음은 같은 모듈에서 가져온 코드이다. 이는 절대 경로를 가져오는 이유가 임시 파일을 생성하기 위한 목적이라는 사실이 드러난다.

```java
String outFile = outputDir + "/" + className.replace('•', '/') + "-class";
FileOutputStream fout = new FileOutputStream(outFile);
BufferedOutputStream bos = new BufferedOutputStream(fout);
```

그렇다면 ctxt 객체에 임시 파일을 생성하라고 시키면 어떨까?

```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

이로써 ctxt는 내부 구조를 드러내지 않으며. 모듈에서 해당 함수는 자신이 몰라야하는 여러 객체(디미터의 법칙)를 탐색할 필요가 없다.

## 자료 전달 객체
자료 구조체의 전형적인 형태는 **공개 변수만 있고 함수가 없는 클래스**다. 이런 자료 구조체를 때로는 자료 전달 객체(Data Transfer Object, DTO)라 한다. 

DTO는 굉장히 유용한 구조체다. 특히 데이터베이스와 통신하거나 소켓에서 받은 메시지의 구문을 분석할 때 유용하다. 흔히 DTO는 데이터베이스에 저장된 가공되지 않은 정보를 애플리케이션 코드에서 사용할 객체로 변환하는 일련의 단계에서 가장 처음으로 사용하는 구조체다.

좀 더 일반적인 형태는 'bean' 구조다. bean은 비공개(private) 변수를 조회/설정 함수로 조작한다. 이는 일종의 _사이비 캡슐화_ 로 간주되며, 별다른 이익을 제공하지 않는다.

```java
// 목록 6-7 address.java
public class Address {
    private String street;
    private String streetExtra;
    private String city; private String state; private String zip;
    
    public Address (String street, String streetExtra,
                   String city, String state, String zip) {
        this.street = street;
        this.streetExtra = streetExtra;
        this.city = city;
        this.state = state;
        this.zip = zip;
    }
    
    public String getStreet() {
        return street;
    }
    
    public String getStreetExtra () {
        return streetExtra;
    }
    
    public String getCity() {
        return city;
    }
    
    public String getState() {
        return state;
    }
    
    public String getZip() {
        return zip;
    }
}

```

### 활성 _레코드_

활성 레코드는 DTO의 특수한 형태다. 공개 변수가 있거나 비공개 변수에 조회/설정 함수가 있는 자료 구조지만, 대개 `save`나 `find`와 같은 탐색 함수도 제공한다. 활성 레코드는 데이터베이스 테이블이나 다른 소스에서 자료를 직접 변환한 결과다.

활성 레코드에 비즈니스 규칙 메서드를 추가해 자료 구조를 객체로 취급하는 것은 바람직하지 않다. 그러면 자료 구조도 아니고 객체도 아닌 잡종 구조가 나오기 때문이다.

해결책은 당연하다. 활성 레코드는 자료 구조로 취급한다. 비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성한다. (여기서 내부 자료는 활성 레코드의 인스턴스일 가능성이 높다.)

## 결론

**객체는 동작을 공개하고 자료를 숨긴다.** 그래서 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉬운 반면, 기존 객체에 새 동작(함수)을 추가하기는 어렵다. 

**자료 구조는 별다른 동작 없이 자료를 노출한다.** 그래서 기존 자료 구조에 새 동작을 추가하기는 쉬우나, 기존 함수에 새 자료 구조를 추가하기는 어렵다.

**시스템을 구현할 때, 새로운 자료 타입을 추가하는 유연성이 필요하면 객체가 더 적합하다. 다른 경우로 새로운 동작을 추가하는 유연성이 필요하면 자료 구조와 절차적인 코드가 더 적합하다.** 우수한 소프트웨어 개발자는 편견 없이 이 사실을 이해해 직면한 문제에 최적인 해결책을 선택한다.
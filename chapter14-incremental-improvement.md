# 14장 점진적인 개선

## Args 구현

`Args.class` : 명령행 인수를 정의된 형식(schema)에 따라 파싱하고, 해당 값을 타입별로 쉽게 조회할 수 있게 해주는 유틸리티 클래스

```java
public static void main(String[] args) {
	try {
		Args arg = new Args("l,p#,d*", args);
		boolean logging = arg.getBoolean('l');
		int port = arg.getInt('p');
		String directory = arg.getString('d');
		executeApplication(logging, port, directory);
	} catch (ArgsException e) {
		System.out.printf("Argument error: %s\n", e.errorMessage());
	}
}
```

매개변수 `"l,p#,d*"`: 명령행 인수의 **스키마.** 각각의 문자는 인수의 이름과 타입을 나타낸다.

- `l` → boolean 값 (`true/false`)
- `p#` → 정수형 인수
- `d*` → 문자열 인수

두 번째 매개변수: 실제 명령행에서 받은 인수 배열. 파싱해서 각 타입에 맞게 저장한다.

파싱이 성공적으로 끝나면 메서드(`getBoolean`, `getInt`, `getString`)를 통해 인수 값을 조회할 수 있다.

### Args 핵심

| 구성 요소 | 책임 |
| --- | --- |
| `Args` | 스키마 해석,  인수 파싱 |
| `ArgumentMarshaler` | 인수 유형 인터페이스 |
| <Type>`ArgumentMarshaler`  | 실제 파싱 로직 |
| `ArgsException` | 오류 처리 |

```java
public class Args {
  private Map<Character, ArgumentMarshaler> marshalers;
  private Set<Character> argsFound;
  private ListIterator<String> currentArgument;

  public Args(String schema, String[] args) throws ArgsException {
    marshalers = new HashMap<>();
    argsFound = new HashSet<>();
    parseSchema(schema);
    parseArgumentStrings(Arrays.asList(args));
  }

```

- `marshalers`: 각 인수 이름(`l`, `p`, `d`)에 대응하는 처리 객체를 저장하는 Map
- `argsFound`: 실제 입력에서 발견된 인수 목록을 추적하는 Set
- `currentArgument`: 인수를 순회하며 파싱할 때 사용

```java
	private void parseSchema(String schema) throws ArgsException {
	  for (String element : schema.split(","))
	    if (element.length() > 0)
	      parseSchemaElement(element.trim());
	}

  private void parseSchemaElement(String element) throws ArgsException {
    char elementId = element.charAt(0);
    String elementTail = element.substring(1); validateSchemaElementId(elementId);
    if (elementTail.length() == 0)
      marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (elementTail.equals("*"))
      marshalers.put(elementId, new StringArgumentMarshaler());
    else if (elementTail.equals("#"))
      marshalers.put(elementId, new IntegerArgumentMarshaler());
    else if (elementTail.equals("##"))
      marshalers.put(elementId, new DoubleArgumentMarshaler());
		else if (elementTail.equals("[*]"))
      marshalers.put(elementId, new StringArrayArgumentMarshaler());
    else
      throw new ArgsException(INVALID_ARGUMENT_FORMAT, elementId, elementTail);
	  }
	}

```

1. 형식 문자열은 쉼표로 구분된 각 인수 정의(`l`, `p#`, `d*` )로 나눈다.
2. `parseSchemaElement()`는 각 정의를 파싱하여 해당 타입에 맞는 `ArgumentMarshaler`를 `marshalers`에 등록한다.

```java
private void parseArgumentStrings(List<String> argsList) throws ArgsException {
  for (currentArgument = argsList.listIterator(); currentArgument.hasNext();) {
    String argString = currentArgument.next();
    if (argString.startsWith("-")) {
      parseArgumentCharacters(argString.substring(1));
    } else {
      currentArgument.previous();
      break;
    }
  }

```

실제 명령행 인수를 순회하면서 `-`로 시작하는 항목을 발견하면 해당 인수 문자를 분해해 파싱한다.

- `p 8080`이라면 `p`를 찾아서 `IntegerArgumentMarshaler`에 값을 설정한다.

**ArgumentMarshaler 인터페이스와 파생 클래스**

```java
public interface ArgumentMarshaler {
  void set(Iterator<String> currentArgument) throws ArgsException;
}
```

모든 인수 타입은 이 인터페이스를 구현한다.

각 타입별 구현 클래스는 다음과 같다:

- `BooleanArgumentMarshaler` : 단순히 `true`로 설정
- `StringArgumentMarshaler` : 다음 인수를 문자열로 저장
- `IntegerArgumentMarshaler` : 다음 인수를 정수로 변환
- `DoubleArgumentMarshaler` : 다음 인수를 실수로 변환
- `StringArrayArgumentMarshaler` : 문자열 배열로 저장 (`[*]` 형식)

각 클래스는 `getValue()`라는 static 메서드로 결과를 조회할 수 있도록 한다.

독자는 코드를 위에서 아래로 순차적으로 읽을 수 있다.

**ArgsException : 에러 처리**

```java
public class ArgsException extends Exception {
  // ...
  public enum ErrorCode {
    OK, INVALID_ARGUMENT_FORMAT, UNEXPECTED_ARGUMENT, INVALID_ARGUMENT_NAME,
    MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, MISSING_DOUBLE, INVALID_DOUBLE
  }

  public String errorMessage() {
    switch (errorCode) {
      case OK: return "TILT: Should not get here.";
      case UNEXPECTED_ARGUMENT: return String.format("Argument -%c unexpected.", errorArgumentId);
      // 생략...
    }
  }
}
```

에러 상황을 명확히 구분하기 위해 열거형 `ErrorCode`를 사용하며, 사용자에게 전달할 수 있는 에러 메시지

**설계 특징**

- **가독성이 좋다**: 위에서 아래로 순차적으로 읽히며 흐름이 자연스럽다.
- **확장에 유리하다**: 새 인수 유형을 추가하려면 `ArgumentMarshaler`를 확장하고 `parseSchemaElement()`에 한 줄만 추가하면 된다.
- **책임 분리**가 명확하다: 스키마 파싱, 인수 파싱, 타입별 처리, 예외 처리 등이 잘 나뉘어 있다.

### 어떻게 짰느냐고?

프로그래밍은 과학이라기보다는 공예에 가깝다.

처음부터 깔끔한 코드가 나오는 경우는 거의 없다. 일단 동작하는 지저분한 코드를 짠 뒤, 계속해서 다듬고 정리해서 최종 버전을 만든다.

대부분의 초보자는 코드가 "돌아가기만" 하면 만족하고 다음 단계로 넘어간다.

하지만 숙련된 개발자는 **돌아가는 코드가 아니라, 읽기 좋고 고치기 좋은 코드**가 진짜 코드라는 걸 안다.

## Args: 1차 초안

```java
import java.text.ParseException; 
import java.util.*;

public class Args {
  private String schema;
  private String[] args;
  private boolean valid = true;
  private Set<Character> unexpectedArguments = new TreeSet<Character>(); 
  private Map<Character, Boolean> booleanArgs = new HashMap<Character, Boolean>();
  private Map<Character, String> stringArgs = new HashMap<Character, String>(); 
  private Map<Character, Integer> intArgs = new HashMap<Character, Integer>(); 
  private Set<Character> argsFound = new HashSet<Character>();
  private int currentArgument;
  private char errorArgumentId = '\0';
  private String errorParameter = "TILT";
  private ErrorCode errorCode = ErrorCode.OK;
  
  private enum ErrorCode {
    OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT}
    
  public Args(String schema, String[] args) throws ParseException { 
    this.schema = schema;
    this.args = args;
    valid = parse();
  }
  
  private boolean parse() throws ParseException { 
    if (schema.length() == 0 && args.length == 0)
      return true; 
    parseSchema(); 
    try {
      parseArguments();
    } catch (ArgsException e) {
    }
    return valid;
  }
  
  private boolean parseSchema() throws ParseException { 
    for (String element : schema.split(",")) {
      if (element.length() > 0) {
        String trimmedElement = element.trim(); 
        parseSchemaElement(trimmedElement);
      } 
    }
    return true; 
  }
  
  private void parseSchemaElement(String element) throws ParseException { 
    char elementId = element.charAt(0);
    String elementTail = element.substring(1); 
    validateSchemaElementId(elementId);
    if (isBooleanSchemaElement(elementTail)) 
      parseBooleanSchemaElement(elementId);
    else if (isStringSchemaElement(elementTail)) 
      parseStringSchemaElement(elementId);
    else if (isIntegerSchemaElement(elementTail)) 
      parseIntegerSchemaElement(elementId);
    else
      throw new ParseException(String.format("Argument: %c has invalid format: %s.", 
        elementId, elementTail), 0);
    } 
  }
    
  private void validateSchemaElementId(char elementId) throws ParseException { 
    if (!Character.isLetter(elementId)) {
      throw new ParseException("Bad character:" + elementId + "in Args format: " + schema, 0);
    }
  }
  
  private void parseBooleanSchemaElement(char elementId) { 
    booleanArgs.put(elementId, false);
  }
  
  private void parseIntegerSchemaElement(char elementId) { 
    intArgs.put(elementId, 0);
  }
  
  private void parseStringSchemaElement(char elementId) { 
    stringArgs.put(elementId, "");
  }
  
  private boolean isStringSchemaElement(String elementTail) { 
    return elementTail.equals("*");
  }
  
  private boolean isBooleanSchemaElement(String elementTail) { 
    return elementTail.length() == 0;
  }
  
  private boolean isIntegerSchemaElement(String elementTail) { 
    return elementTail.equals("#");
  }
  
  private boolean parseArguments() throws ArgsException {
    for (currentArgument = 0; currentArgument < args.length; currentArgument++) {
      String arg = args[currentArgument];
      parseArgument(arg); 
    }
    return true; 
  }
  
  private void parseArgument(String arg) throws ArgsException { 
    if (arg.startsWith("-"))
      parseElements(arg); 
  }
  
  private void parseElements(String arg) throws ArgsException { 
    for (int i = 1; i < arg.length(); i++)
      parseElement(arg.charAt(i)); 
  }
  
  private void parseElement(char argChar) throws ArgsException { 
    if (setArgument(argChar))
      argsFound.add(argChar); 
    else 
      unexpectedArguments.add(argChar); 
      errorCode = ErrorCode.UNEXPECTED_ARGUMENT; 
      valid = false;
  }
  
  private boolean setArgument(char argChar) throws ArgsException { 
    if (isBooleanArg(argChar))
      setBooleanArg(argChar, true); 
    else if (isStringArg(argChar))
      setStringArg(argChar); 
    else if (isIntArg(argChar))
      setIntArg(argChar); 
    else
      return false;
    
    return true; 
  }
  
  private boolean isIntArg(char argChar) {
    return intArgs.containsKey(argChar);
  }
  
  private void setIntArg(char argChar) throws ArgsException { 
    currentArgument++;
    String parameter = null;
    try {
      parameter = args[currentArgument];
      intArgs.put(argChar, new Integer(parameter)); 
    } catch (ArrayIndexOutOfBoundsException e) {
      valid = false;
      errorArgumentId = argChar;
      errorCode = ErrorCode.MISSING_INTEGER;
      throw new ArgsException();
    } catch (NumberFormatException e) {
      valid = false;
      errorArgumentId = argChar; 
      errorParameter = parameter;
      errorCode = ErrorCode.INVALID_INTEGER; 
      throw new ArgsException();
    } 
  }
  
  private void setStringArg(char argChar) throws ArgsException { 
    currentArgument++;
    try {
      stringArgs.put(argChar, args[currentArgument]); 
    } catch (ArrayIndexOutOfBoundsException e) {
      valid = false;
      errorArgumentId = argChar;
      errorCode = ErrorCode.MISSING_STRING; 
      throw new ArgsException();
    } 
  }
  
  private boolean isStringArg(char argChar) { 
    return stringArgs.containsKey(argChar);
  }
  
  private void setBooleanArg(char argChar, boolean value) { 
    booleanArgs.put(argChar, value);
  }
  
  private boolean isBooleanArg(char argChar) { 
    return booleanArgs.containsKey(argChar);
  }
  
  public int cardinality() { 
    return argsFound.size();
  }
  
  public String usage() { 
    if (schema.length() > 0)
      return "-[" + schema + "]"; 
    else
      return ""; 
  }
  
  public String errorMessage() throws Exception { 
    switch (errorCode) {
      case OK:
        throw new Exception("TILT: Should not get here.");
      case UNEXPECTED_ARGUMENT:
        return unexpectedArgumentMessage();
      case MISSING_STRING:
        return String.format("Could not find string parameter for -%c.", errorArgumentId);
      case INVALID_INTEGER:
        return String.format("Argument -%c expects an integer but was '%s'.", errorArgumentId, errorParameter);
      case MISSING_INTEGER:
        return String.format("Could not find integer parameter for -%c.", errorArgumentId);
    }
    return ""; 
  }
  
  private String unexpectedArgumentMessage() {
    StringBuffer message = new StringBuffer("Argument(s) -"); 
    for (char c : unexpectedArguments) {
      message.append(c); 
    }
    message.append(" unexpected.");
    
    return message.toString(); 
  }
  
  private boolean falseIfNull(Boolean b) { 
    return b != null && b;
  }
  
  private int zeroIfNull(Integer i) { 
    return i == null ? 0 : i;
  }
  
  private String blankIfNull(String s) { 
    return s == null ? "" : s;
  }
  
  public String getString(char arg) { 
    return blankIfNull(stringArgs.get(arg));
  }
  
  public int getInt(char arg) {
    return zeroIfNull(intArgs.get(arg));
  }
  
  public boolean getBoolean(char arg) { 
    return falseIfNull(booleanArgs.get(arg));
  }
  
  public boolean has(char arg) { 
    return argsFound.contains(arg);
  }
  
  public boolean isValid() { 
    return valid;
  }
  
  private class ArgsException extends Exception {
  } 
}
```

### 그래서 멈췄다

- 타입별 처리 로직이 스키마 파싱, 인수 파싱, 값 조회의 세 곳에 중복되어 있음
- 새로운 타입을 추가하려면 세 군데 이상 수정해야 함
- `Map<Character, Type>`이 여러 개 존재해 응집도가 낮고 중복됨
- `ArgsException`이 실질적인 정보 전달에 활용되지 않음

인수 유형은 다양하지만 모두가 유사한 메서드를 제공. 👉 인터페이스스 **ArgumentMarshaler 탄생**

### 점진적으로 개선하다

프로그램을 망치는 가장 빠른 방법 중 하나는 '개선'이라는 이름 아래 구조를 무리하게 뒤엎는 것이다. 그렇게 구조를 크게 바꾼 프로그램은 대개 다시는 정상으로 돌아오지 못한다. 변경 후에도 시스템이 이전과 **정확히 같은 방식으로 동작하는 것**이 정말 어려워지기 때문이다.

👉 **테스트 주도 개발(TDD)**

TDD는 항상 시스템이 **정상적으로 동작해야 한다**는 원칙을 기반으로 한다. 즉, 어떤 변경을 하든 간에 프로그램은 여전히 잘 돌아가야 한다. 변경 이후에도 프로그램이 이전과 **동일한 결과를 보장**해야만 한다.

이걸 보장하려면 언제든 실행 가능한 자동화 테스트가 필수다. 

리팩토링을 할 때는 코드를 최소로 건드리는, 가장 단순한 변경 부터 시작한다. 구조 변경을 최소화하면서도 코드를 점점 더 좋은 방향으로 개선해 나가도 TDD 덕분에 언제든 테스트로 변경이 안전함을 검증할 수 있었고, 덕분에 과감한 구조 개선도 시도할 수 있다. 

1. `Map<Character, Boolean>`→ `Map<Character, ArgumentMarshaler>`로 변경 

```java
// 전: boolean 값을 직접 저장
private Map<Character, Boolean> booleanArgs = new HashMap<>();

// 후: ArgumentMarshaler 객체 저장
private Map<Character, ArgumentMarshaler> booleanArgs = new HashMap<>();
```

```java
// 전: 스키마 요소 파싱 시 true로 저장
private void parseBooleanSchemaElement(char elementId) {
  booleanArgs.put(elementId, false); // 기본값
}

// 후: BooleanArgumentMarshaler 객체 저장
private void parseBooleanSchemaElement(char elementId) {
  booleanArgs.put(elementId, new BooleanArgumentMarshaler());
}
```

```java
// 전: 값 설정
private void setBooleanArg(char argChar, boolean value) {
  booleanArgs.put(argChar, value);
}

// 후: BooleanArgumentMarshaler 내부에 값 설정
private void setBooleanArg(char argChar, boolean value) {
  booleanArgs.get(argChar).setBoolean(value);
}
```

```java
// 전: 값 조회
public boolean getBoolean(char arg) {
  Boolean value = booleanArgs.get(arg);
  return value != null && value;
}

// 후: BooleanArgumentMarshaler에서 값 꺼내기
public boolean getBoolean(char arg) {
  ArgumentMarshaler am = booleanArgs.get(arg);
  return am != null && am.getBoolean();
}
```

## String 인수

나머지 인수 추가 과정은 boolean 인수 처리 방식과 유사하다.

1. 우선 `HashMap`에 관련 정보를 저장하도록 하고, `parse`, `set`, `get` 함수를 수정
2. 모든 인수 유형에 대한 처리를 `ArgumentMarshaler`에 넣고,
3. **파생 클래스로 분리**

이렇게 하면 프로그램 구조를 점진적으로 개선하면서도 정상 동작을 유지할 수 있다.

**기존 예외 코드의 문제**

- 코드가 흉하고 읽기 어렵다.
- Args 클래스와 무관한 책임을 떠안고 있다.
- `ParseException`을 던지지만, 이는 `Args` 모듈에 소속되지 않는다.

 👉 모든 예외 처리를 `ArgsException`이라는 독립 클래스로

- `Args` 모듈과 예외/오류 처리 로직을 분리

```java
public class ArgsException extends Exception {
  private char errorArgumentId = '\0';
  private String errorParameter = "TILT";
  private ErrorCode errorCode = ErrorCode.OK;

  public ArgsException() {}

  public ArgsException(String message) {super(message);}

  public ArgsException(ErrorCode errorCode) {
    this.errorCode = errorCode;
  }

  public ArgsException(ErrorCode errorCode, String errorParameter) {
    this.errorCode = errorCode;
    this.errorParameter = errorParameter;
  }

  public ArgsException(ErrorCode errorCode, char errorArgumentId, String errorParameter) {
    this.errorCode = errorCode;
    this.errorParameter = errorParameter;
    this.errorArgumentId = errorArgumentId;
  }

  public char getErrorArgumentId() {
    return errorArgumentId;
  }

  public void setErrorArgumentId(char errorArgumentId) {
    this.errorArgumentId = errorArgumentId;
  }

  public String getErrorParameter() {
    return errorParameter;
  }

  public void setErrorParameter(String errorParameter) {
    this.errorParameter = errorParameter;
  }

  public ErrorCode getErrorCode() {
    return errorCode;
  }

  public void setErrorCode(ErrorCode errorCode) {
    this.errorCode = errorCode;
  }

  public String errorMessage() throws Exception {
    switch (errorCode) {
      case OK:
        throw new Exception("TILT: Should not get here.");
      case UNEXPECTED_ARGUMENT:
        return String.format("Argument -%c unexpected.", errorArgumentId);
      case MISSING_STRING:
        return String.format("Could not find string parameter for -%c.", errorArgumentId);
      case INVALID_INTEGER:
        return String.format("Argument -%c expects an integer but was '%s'.", errorArgumentId, errorParameter);
      case MISSING_INTEGER:
        return String.format("Could not find integer parameter for -%c.", errorArgumentId);
      case INVALID_DOUBLE:
        return String.format("Argument -%c expects a double but was '%s'.", errorArgumentId, errorParameter);
      case MISSING_DOUBLE:
        return String.format("Could not find double parameter for -%c.", errorArgumentId);
    }
    return "";
  }

  public enum ErrorCode {
    OK, INVALID_FORMAT, UNEXPECTED_ARGUMENT, INVALID_ARGUMENT_NAME, MISSING_STRING,
    MISSING_INTEGER, INVALID_INTEGER,
    MISSING_DOUBLE, INVALID_DOUBLE
  }
}
```

- 에러 코드(`ErrorCode`)를 열거형으로 정의.
- 인수 식별자, 파라미터 값을 저장.
- 오류 메시지를 생성하는 `errorMessage()` 메서드 포함.
- SRP(Single Responsibility Principle) 준수

## **결론**

그저 돌아가기만 하는 코드는 부족하다.

나쁜 코드보다 개발 프로젝트에 더 심각한 악영향을 끼친다.

- 나쁜 일정은 다시 계획하면 된다.
- 나쁜 요구사항은 다시 정의하면 된다.
- 나쁜 팀 역학은 복구할 수 있다.

시간이 지날수록 무게가 늘어나고 팀의 발목을 잡는다.

물론 나쁜 코드도 깨끗한 코드로 바꿀 수는 있지만 그 비용은 크다.

- 코드가 썩어가면서 뒤엉키는 모듈들
- 수없이 생기는 숨겨진 의존성
- 그 의존성을 찾아 깨는 데는 막대한 시간과 인내

**처음부터 깨끗하게 유지하는 것은 상대적으로 쉽다.**

아침에 엉망으로 만든 코드를 오후에 정리하는 건 어렵지 않다.
더 나아가, 5분 전에 만든 엉망진창 코드는 바로 지금 정리하는 게 가장 쉽다.

**코드는 언제나 최대한 깔끔하고 단순하게 유지해야 한다.**
절대로 썩어가도록 방치해서는 안 된다.
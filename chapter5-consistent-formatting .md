# 5장 형식 맞추기

## 형식을 맞추는 목적

**형식은 의사소통이다**

- 코드는 문서이며, **개발자 간의 의사소통 수단**이다.
- 오늘 구현한 코드는 내일 수정 될 수 있다.
    - → **가독성은 유지보수성과 직결**된다.

## 적절한 행 길이를 유지하라

행 길이는 반드시 지킬 엄격한 규칙은 아니지만 일반적으로 큰 파일보다 작은 파일이 이해하기 쉽다.

- 짧고 명확한 단위로 나눈다.
- 일반적으로 한 파일은 200줄 이내가 바람직하다.

### 신문 기사처럼 작성하라

신문의 표제는 최상단에서 기사 내용을 몇 마디로 요약한다. 소스 파일도 신문 기사와 비슷하게 작성하는 것이 좋다.

- **가장 중요한 내용 → 세부사항** 순서.
    - 상단에는 고차원 개념, 아래로 갈수록 세부 구현.
- **파일명**과 클래스명만 보고 역할을 파악할 수 있어야 한다.


### 개념은 빈 행으로 분리하라

각 행은 수식이나 절을 나타내고, 일련의 행 묶음은 완결된 생각 하나를 표현한다. 관련된 코드끼리는 **묶고**, 서로 다른 개념은 **빈 줄로 구분**한다. 

👍👍👍

```java
package fitnesse.wikitext.widgets;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class BoldWidget extends ParentWidget {
	public static final String REGEXP = "'''.+?'''";
	private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
		Pattern.MULTILINE + Pattern.DOTALL
	);

	public BoldWidget(ParentWidget parent, String text) throws Exception {
		super(parent);
		Matcher match = pattern.matcher(text);
		match.find();
		addChildWidgets(match.group(1));
	}
}
```

👎👎👎

```java
package fitnesse.wikitext.widgets;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
public class BoldWidget extends ParentWidget {
	public static final String REGEXP = "'''.+?'''";
	private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
		Pattern.MULTILINE + Pattern.DOTALL);
	public BoldWidget(ParentWidget parent, String text) throws Exception {
		super(parent);
		Matcher match = pattern.matcher(text);
		match.find();
		addChildWidgets(match.group(1));}
}
```

### 세로 밀집도(연관성)

서로 밀접한 코드는 세로로 가까이 배치한다.

👍👍👍

```java
public class ReporterConfig {
    private String m_className;
    private List<Property> m_properties = new ArrayList<Property>();

    public void addProperty(Property property) {
        m_properties.add(property);
}
```

👎👎👎

```java
public class ReporterConfig {
    /**
     * 리포터 리스너의 클래스 이름
    */
    private String m_className;

    /**
     * 리포터 리스너의 속성
    */
    private List<Property> m_properties = new ArrayList<Property>();
    public void addProperty(Property property) {
        m_properties.add(property);
}
```

### 수직 거리

관련 있는 코드끼리는 **물리적으로 가까워야** 이해하기 쉽다. 

- 무엇을 하는지 이해하기 위해 탐색할 때 조각 조각들이 어디에 있는지 찾기 쉽다.
    - protected, 상속 관계..👎

같은 파일에 속할 정도로 밀접한 두 개념은 세로 거리로 연관성을 표현한다.


1. **변수 선언**
    - 변수는 사용하는 위치에 최대한 가까이 선언한다.
    - 지역 변수는 각 함수 맨 처음에 선언한다.
        
        ```java
          private static void readPreferences() {
              InputStream is = null;
              try {
                  is = new FileInputStream(getPreferencesFile());
                  setPreferences(new Properties(getPreferences()));
                  getPreferences().load(is);
              } catch (IOException e) {
                  try {
                      if (is != null)
                          is.close();
                  } catch (IOException e1) {
                  }
              }
          }
        ```
        
    - loop문 제어 변수는 loop문 내부에 선언
        
        ```java
        public int countTestCases() {
        		int count = 0;
        		for (Test each : tests) 
        				count += each.countTestCases();
        		return count;
        }
        ```
        

1. **인스턴스 변수**
    - 인스턴스 변수를 선언하는 위치는 잘 알려진 위치여야 한다.
        - 클래스 맨 처음에 선언한다.
    - 메서드 중간에 숨겨두지 않는다.

1. **종속 함수**
    - 한 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다.
    - 호출하는 함수를 호출되는 함수보다 먼저 배치한다.
        - 이 규칙이 일관적으로 적용되면 독자는 방금 호출된 함수가 곧 정의될 거라 예측할 수 있고, 자연스럽게 코드를 읽을 수 있다.
        
        ```java
          public class WikiPageResponder implements SecureResponder { 
              protected WikiPage page;
              protected PageData pageData;
              protected String pageTitle;
              protected Request request; 
              protected PageCrawler crawler;
        
        			// 호출 → 정의 순서로 배치
              public Response makeResponse(FitNesseContext context, Request request) throws Exception {
                  String pageName = getPageNameOrDefault(request, "FrontPage");
                  loadPage(pageName, context); 
              }
        
              private String getPageNameOrDefault(Request request, String defaultPageName) {
                  String pageName = request.getResource(); 
                  if (StringUtil.isBlank(pageName))
                      pageName = defaultPageName;
        
                  return pageName; 
              }
        
              protected void loadPage(String resource, FitNesseContext context)
                  throws Exception {
                  WikiPagePath path = PathParser.parse(resource);
                  crawler = context.root.getPageCrawler();
                  crawler.setDeadEndStrategy(new VirtualEnabledPageCrawler()); 
                  page = crawler.getPage(context.root, path);
                  if (page != null)
                      pageData = page.getData();
              }
        
          ...
        ```
        
    
2. 개념적 유사성
    
    **비슷한 역할/이름/구조**를 가진 함수는 **근접 배치**한다.
    
    - 친화도가 높은 요인
        - 다른 함수를 호출해 생기는 직접적인 종속성
        - 변수와 그 변수를 사용하는 함수
        - 비슷한 동작을 수행하는 함수
            
            ```java
            class Assert {
            		static public void assertTrue(String message, boolean condition) {
            			if (!condition) 
            				fail(message);
            		}
            		
            		static public void assertTrue(boolean condition) { 
            			assertTrue(null, condition);
            		}
            		
            		static public void assertFalse(String message, boolean condition) { 
            			assertTrue(message, !condition);
            		}
            		
            		static public void assertFalse(boolean condition) { 
            			assertFalse(null, condition);
            		} 
            ...
            
            // 명명법이 같고 기본 기능이 유사하다.
            ```
            
        

| 개념 | 위치 기준 |
| --- | --- |
| 지역 변수 | 사용하는 코드와 최대한 가까이 |
| 루프 제어 변수 | 루프문 내부에서 선언 |
| 인스턴스 변수 | 클래스 맨 위에 모아서 선언 |
| 종속 함수 | 호출하는 함수 위에 배치 |

### 세로 순서

1. 함수 호출 종속성은 아래 방향으로 유지한다. (고차원 추상화→ 저차원 추상화)
2. 표제, 가장 중요한 개념을 가장 먼저 표현한다.
    1. 표제는 세세한 사항을 최대한 배제한다.

## 가로 형식 맞추기

가급적 한 줄 **80~120자 이내**로 제한한다.

→ 긴 줄은 읽기 어렵고, 리뷰/디버깅 시 불편함.

### 가로 공백과 밀집도

밀집도는 가로 공백을 사용해서 느슨함을 표현한다.

```java
private void measureLine(String line) { 
    lineCount++;

	// 공백으로 할당 연산자 강조 -> 왼쪽과 오른쪽 구분이 분명해짐
    int lineSize = line.length();
    totalChars += lineSize; 
	
	// 함수명과 괄호 사이에는 공백을 넣지 않는다. -> 함수와 인수의 밀접함을 표현
	// 공백으로 쉼표 강조 -> 별개의 인수 강조
    lineWidthHistogram.addLine(lineSize, lineCount);
    recordWidestLine(lineSize);
}
```

### 가로 정렬

과도한 정렬은 지양한다.

- 가독성을 해치고, **작은 수정에도 정렬 전체가 깨짐**.
- 타입/변수명/할당 연산자 기준으로 줄맞춤을 시도하지 말 것.

👎👎👎

```java
public class FitNesseExpediter implements ResponseSender {
		// 변수 유형은 무시하고 이름만 읽게 됨
		private     Socket         socket;
		private     InputStream    input;
		private     OutputStream   output;
		private     Reques         request;
		private     Response       response;
		private     FitNesseContex context;
		protected   long           requestParsingTimeLimit;
		private     long           requestProgress;
		private     long           requestParsingDeadline;
		private     boolean        hasError;
		
		// 할당 연산자는 보이지 않고 오른쪽 피연산자만 보임
		public FitNesseExpediter(Socket          s
														,FitNesseContext context) throws Exception 
		{
				this.context = context;
				socket =                  s;
				input =                   s.getInputStream();
				output =                  s.getOutputStream();
				requestParsingTimeLimit = 10000;
		}
```

위와 같은 정렬은 코드가 엉뚱한 부분을 강조해서 진짜 의도가 가려지기 때문에 유용하지 않다.

👍👍👍

```java
public class FitNesseExpediter implements ResponseSender {
		private Socket socket;
		private InputStream input;
		private OutputStream output;
		private Request request;
		private Response response;
		private FitNesseContex context;
		protected long requestParsingTimeLimit;
		private long requestProgress;
		private long requestParsingDeadline;
		private boolean hasError;
		
		public FitNesseExpediter(Socket s,FitNesseContext context) throws Exception {
				this.context = context;
				socket = s;
				input = s.getInputStream();
				output = s.getOutputStream();
				requestParsingTimeLimit = 10000;
		}
```

### 들여쓰기

소스파일은 윤곽도와 계층이 비슷하다.

- 파일 전체에 적용되는 정보
- 파일 내 개별 클래스에 적용되는 정보
- 클래스 내 각 메서드에 적용되는 정보
- 블록 내 블록에 재귀적으로 적용되는 정보

계층에서 각 수준은 이름을 선언하는 범위이자 선언문과 실행문을 해석하는 범위이다. 이 범위(scope)를 표현하기 위해 **들여쓰기**를 사용한다. 들여쓰기를 통해 왼쪽으로 코드를 맞춰 코드가 속하는 범위를 시각적으로 표현한다.


👎👎👎

```java
public class CommentWidget extends TextWidget {
    public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";

    public CommentWidget(ParentWidget parent, String text){super(parent, text);}
    public String render() throws Exception {return ""; } 
}
```

👍👍👍

```java
public class CommentWidget extends TextWidget {
    public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";

    public CommentWidget(ParentWidget parent, String text){
        super(parent, text);
    }

    public String render() throws Exception {
        return ""; 
    } 
}
```

### 가짜 범위

- `if`, `while`, `for` 뒤에 오는 **아무 동작도 하지 않는 빈 문장(세미콜론 `;`)이 오는 경우**

---

👎👎👎

```java
// while은 세미콜론 하나가 루프 본문처럼 동작 
//   → 루프는 실행되지만 아무 일도 안 함
while (inputStream.read() != -1);
    
// for 뒤에 ;가 있어 루프 내부가 없음 
//   → 실제 블록은 루프 외부이며 단 한 번만 실행
for (int i = 0; i < 10; i++);
{
    System.out.println("This only runs once!");
}
```

- `;` 하나가 본문으로 처리됨 → 읽기 어렵고, 실수로 작성된 것처럼 보임


### 👍👍👍

```java
while (inputStream.read() != -1)
    ; // intentionally empty
  
    
// 예: 딜레이 (busy wait) CPU를 소모하면서 일정 시간 대기
for (long wait = 0; wait < 1_000_000L; wait++)
    ; // intentional delay
```

- 주석으로 의도 명시
- 들여쓰기로 가독성 개선

---

| ❌ **삭제해야 할 경우** | ✅ **유지할 수 있는 경우** |
| --- | --- |
| 실수로 세미콜론이 들어간 경우 | 스트림을 소비하거나 busy-wait처럼 명확한 목적이 있는 경우 |
| 루프 내부에 실행 코드가 빠진 경우 | 동작이 없어야만 하는 루프일 때 (단, **주석 필수**) |


## 팀 규칙

각자 선호하는 규칙이 있겠지만 팀에 속해있다면 팀 규칙을 따라야 한다. 그래야 소프트웨어가 일관적인 스타일을 가질 수 있다. 스타일이 일관적이고 매끄러워야 독자에게 신뢰감을 줄 수 있다.
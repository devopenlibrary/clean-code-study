---
nav_order: 15
---

# 15장 JUnit

## JUnit 프레임워크


기존

```java
public class Comparisoncompactor{
    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String DELTA_START = "[";
    
    private int fContextLength;
    private String fExpected;
    private String fActual;
    private int fPrefix;
    private int fSuffix;
    
    public ComparisonCompactor(int contextLength, String expected, String actual){
        fContextLength = contextLength;
        fExpected = expected;
        fActual = actual;
    }
    
    public String compact(String message) {
        if(fExpected == null || fActual == null || areStringsEqual())
            return Assert.format(message, fExpected, fActual);
            
        findCommonPrefix();
        findCommonSuffix();
        String expected = comapactString(fExpected);
        String actual = comapactString(fActual);
        return Assert.format(message, expected, actual);
    }
    
    private String compactString(String source){
        String result = DELTA_START + source.substring(fPrefix, source.length() - fSuffix + 1) + DELTA_END;
        if(fPrefix > 0)
            result = computeCommonPrefix() + result;
        if(fSuffix > 0)
            result = result + computeCommonSuffix();
        return result;
    }
    
    private void findCommonPrefix(){
        fPrefix = 0;
        int end = Math.min(fExpected.length(), fActual.length());
        for(; fPrefix < end; fPrefix++){
            if(fExpected.charAt(fPrefix) != fActual.charAt(fPrefix))
                break;
        }
    }
    
    private void findCommonSuffix(){
        int expectedSuffix = fExpected.length()-1;
        int actualSuffix = fActual.length()-1;
        for(; actualSuffix >= fPrefix && expectedSuffix >= fPrefix; actualSuffix--, expectedSuffix--){
            if(fExpected.charAt(expectedSuffix) != fActual.charAt(actualSuffix))
                break;
        }
        fSuffix = fExpected.length() - expectedSuffix;
    }
    
    private String computeCommonPrefix(){
        return (fPrefix > fContextLength ? ELLIPSIS : "") + fExpected.substring(Math.max(0, fPrefix - fContextLegnth), fPrefix);
    }
    
    private String computeCommonSuffix(){
        int end = Math.min(fExepcted.legngth() - fSuffix - fContextLength, fExpected.length());
        return fExpected.substirng(fExpected.length() - fSuffix +1, end) + (fExpected.length() - fSuffix +1 < fExpected.length() - fContextLength ? ELLIPSIS :"")
    }
    
    private boolean areStringEqual(){
        return fExpected.equals(fActual);
    }
}
```

- 접두어 f 제거: 오늘날에는 접두어가 따로 필요하지 않다.
    - 지역변수와 접두어f를 제거한 변수명 구분을 위해 더욱 명확한 네이밍 
- 조건문 캡슐화: 의도를 명확하게 하기 위해서는 조건문을 캡슐화
- 조건문을 긍정문으로 변경
- 조건문을 명확한 이름으로 캡슐화
- 숨겨진 시간적인 결합을 외부에 노출: `findCommonPrefix()` 후 `findCommonSuffix()`를 호출해야함
    - 순서를 지정한 새로운 함수 정의: `findCommonPrefixAndSuffix()`
    - `findCommonPrefixAndSuffix()` 내에 시간적인 결합 내용을 정의
- index와 length의 차이: 0부터 시작하는가?
    - 만약 length를 구하기 위해 계속해서 +1 연산을 한다면, 차라리 index로 네이밍하자 

결과
```java
public class ComparisonCompactor{
    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String DELTA_START = "[";
    
    private int contextLength;
    private String expected;
    private String actual;
    private int prefixLength;
    
    public ComparisonCompactor(int contextLength, String expected, String actual){
        this.contextLength = contextLength;
        this.expected = expected;
        this.actual = actual;
    }
    
    public String formatCompactedComparison(String message){
        String compactExpected = expected;
        String compactActual = actual;
        
        if(shouldBeCompacted()){
            findCommonPrefixAndSuffix();
            compactExpected = comapactString(expected);
            compactActual = comapactString(actual);            
        }
        
        return Assert.format(message, compactExpected, compactActual);
    }
    
    private boolean shouldBeCompacted(){
        return !shouldNotBeCompacted();
    }
    
    private boolean shouldBeCompacted(){
        return expected == null || actual == null || expected.equals(actual);
    }

    private void findCommonPrefixAndSuffix(){
        findCommonPrefix();
        suffixLength = 0;
        for(; suffixOverlapsPrefix(suffixLength); suffixLength++){
            if(charFromEnd(expected, suffixLength) != charFromEnd(actual, suffixLength))
                break;
        }
    }
    
    private char charFromEnd(String s, int i){
        return s.charAt(s.length() -i -1);
    }
    
    private boolean suffixOverlapsPrefix(int suffixLength){
        return actual.length() - suffixLength <= prefixLength || expected.length() - suffixLength <= prefixLength;
    }
    
    private void findCommonPrefixAndSuffix(){
        prefixLength = 0;
        int end = Math.min(expected.length(), actual.length());
        for(; prefixLength < end; prefixLength ++)
            if(expected.chatAt(prefixLength) != actual.charAt(prefixLength))
                break;
    }
    
    private String compact(String s){
        return new StringBuilder()
            .append(startingEllipsis())
            .append(startingContext())
            .append(DELTA_START)
            .append(delta(s))
            .append(DELTA_END)
            .append(endingContext())
            .append(endingEllipsis())
            .toString();
    }
    
    private String startingEllipsis(){
        return prefixLength > contextLength ? ELLIPSIS : "";
    }
    
    private String startingContext(){
        int contextStart = Math.max(0, prefixLength = contextLength);
        int contextEnd = prefixLength;
        return expected.substring(contextStart, contextEnd);
    }
    
    private String delta(String s){
        int dletaStart = prefixLength;
        int deltaEnd = s.length() - suffixLenth;
        return s.substring(deltaStart, deltaEnd);
    }
    
    private String endContext(){
        int contextStart = expected.length() - suffixLength;
        int contextEnd = Math.min(contextStart + contextLength, expected.length());
        return expected.substring(contextStart, contextEnd);
    }
    
    private String endingEllipsis(){
        return (suffixLength > contextLenth ? ELLIPSIS :"");
    }
}
```

- 리팩토링을 진행하다보면 어느 수준에 이를 때까지 시행착오를 반복하는 작업이다.
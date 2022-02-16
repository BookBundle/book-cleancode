# 14장. 점진적인 개선 p.245
- 이 장은 점진적인 개선을 보여주는 사례 연구다. 
- 출발은 좋았으나 확장성이 부족했던 모듈 소개 후 모듈을 개선하고 정리하는 단계를 살펴본다.

## 목차
1. [Args 구현](#1)
    1. [어떻게 짰느냐고?](#2)
1. [Args: 1차 초안](#3)
    1. [그래서 멈췄다](#4)
    1. [점진적으로 개선하다](#5)
1. [String 인수](#6)
1. [결론](#7)

<br>

- 프로그램을 짜다보면 종종 명령행 인수의 구문을 분석할 필요가 생긴다. 
- 편리한 유틸리티가 없다면 main 함수로 넘어오는 문자열 배열을 직접 분석하게 된다. 
- 새로 짤 유틸리티를 Args라 부르겠다.

<br>

#### 목록 14-1 간단한 Args 사용법

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

- Args 클래스의 첫째 매개변수는 형식 또는 스키마를 지정한다. 이 문자열은 명령행 인수 세 개를 정의한다.
- 첫 번째 -l은 부울 인수, 두 번째 -p는 정수 인수, 세 번째 -d는 문자열 인수다. 
- Args 생성자로 넘긴 둘째 매개변수는 main으로 넘어온 명령행 인수 배열 자체다.

<br>

- 생성자에서 ArgsException이 발생하지 않으면 명령행 인수의 구문을 성공적으로 분석했으며 Args 인스턴스에 질의를 던져도 좋다는 말이다. - 인수 값을 가져오려면 getBoolean, getInteger, getString 등과 같은 메서드를 사용한다.
- 형식 문자열이나 명령행 인수 자체에 문제가 있다면 ArgsException이 발생하며, 구체적인 오류를 알아내려면 catch에서 errorMessage() 메서드를 사용한다.

## <a name = '1'> Args 구현 p.247 </a>

- 아래 목록 14-2는 Args 클래스다.

#### 목록 14-2 Args.java

```java
package com.objectmentor.utilities.args;

import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;
import java.util.*;

public class Args {
	private Map<Character, ArgumentMarshaler> marshalers;
	private Set<Character> argsFound;
	private ListIterator<String> currentArgument;

	public Args(String schema, String[] args) throws ArgsException {
		marshalers = new HashMap<Character, ArgumentMarshaler>();
		argsFound = new HashSet<Character>();
    
        parseSchema(schema);
        parseArgumentStrings(Arrays.asList(args)); 
    }
  
    private void parseSchema(String schema) throws ArgsException { 
        for (String element : schema.split(","))
            if (element.length() > 0) 
                parseSchemaElement(element.trim());
    }
  
    private void parseSchemaElement(String element) throws ArgsException { 
        char elementId = element.charAt(0);
        String elementTail = element.substring(1); 
        validateSchemaElementId(elementId);
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
  
    private void validateSchemaElementId(char elementId) throws ArgsException { 
        if (!Character.isLetter(elementId))
            throw new ArgsException(INVALID_ARGUMENT_NAME, elementId, null); 
    }
  
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
    }
  
    private void parseArgumentCharacters(String argChars) throws ArgsException { 
        for (int i = 0; i < argChars.length(); i++)
            parseArgumentCharacter(argChars.charAt(i)); 
    }
  
    private void parseArgumentCharacter(char argChar) throws ArgsException { 
        ArgumentMarshaler m = marshalers.get(argChar);
        if (m == null) {
            throw new ArgsException(UNEXPECTED_ARGUMENT, argChar, null); 
        } else {
            argsFound.add(argChar); 
            try {
                m.set(currentArgument); 
            } catch (ArgsException e) {
                e.setErrorArgumentId(argChar);
                throw e; 
            }
        } 
    }
  
    public boolean has(char arg) { 
        return argsFound.contains(arg);
    }
  
    public int nextArgument() {
        return currentArgument.nextIndex();
    }
  
    public boolean getBoolean(char arg) {
        return BooleanArgumentMarshaler.getValue(marshalers.get(arg));
    }
  
    public String getString(char arg) {
        return StringArgumentMarshaler.getValue(marshalers.get(arg));
    }
  
    public int getInt(char arg) {
        return IntegerArgumentMarshaler.getValue(marshalers.get(arg));
    }
  
    public double getDouble(char arg) {
        return DoubleArgumentMarshaler.getValue(marshalers.get(arg));
    }
  
    public String[] getStringArray(char arg) {
        return StringArrayArgumentMarshaler.getValue(marshalers.get(arg));
    } 
}
```

- 여기저기 뒤적일 필요 없이 위에서 아래로 코드가 읽힌다는 사실에 주목한다. 
- 한 가지 먼저 읽어볼 코드가 있다면 ArgumentMarshaler 정의인데, 아래 목록 14-3에서 목록 14-6까지는 ArgumentMarshaler 인터페이스와 파생 클래스다.

<details>
<summary> 목록 14-3 ~ 목록 14-6 </summary>

<div markdown = "1">

#### 목록 14-3 ArgumentMarshaler.java

```java
public interface ArgumentMarshaler {
    void set(Iterator<String> currentArgument) throws ArgsException;
}
```

#### 목록 14-4 BooleanArgumentMarshaler.java

```java
public class BooleanArgumentMarshaler implements ArgumentMarshaler { 
    private boolean booleanValue = false;
    
    public void set(Iterator<String> currentArgument) throws ArgsException { 
        booleanValue = true;
    }
    
    public static boolean getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof BooleanArgumentMarshaler)
            return ((BooleanArgumentMarshaler) am).booleanValue; 
        else
            return false; 
    }
}
```

#### 목록 14-5 StringArgumentMarshaler.java

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class StringArgumentMarshaler implements ArgumentMarshaler { 
    private String stringValue = "";
    
    public void set(Iterator<String> currentArgument) throws ArgsException { 
        try {
            stringValue = currentArgument.next(); 
        } catch (NoSuchElementException e) {
            throw new ArgsException(MISSING_STRING); 
        }
    }
  
    public static String getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof StringArgumentMarshaler)
            return ((StringArgumentMarshaler) am).stringValue; 
        else
            return ""; 
    }
}
```

#### 목록 14-6 IntegerArgumentMarshaler.java

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class IntegerArgumentMarshaler implements ArgumentMarshaler { 
    private int intValue = 0;
  
    public void set(Iterator<String> currentArgument) throws ArgsException { 
        String parameter = null;
        try {
            parameter = currentArgument.next();
            intValue = Integer.parseInt(parameter);
        } catch (NoSuchElementException e) {
            throw new ArgsException(MISSING_INTEGER);
        } catch (NumberFormatException e) {
            throw new ArgsException(INVALID_INTEGER, parameter); 
        }
    }
  
    public static int getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof IntegerArgumentMarshaler)
            return ((IntegerArgumentMarshaler) am).intValue; 
        else
            return 0; 
    }
}
```

- 나머지 DoubleArgumentMarshaler와 StringArrayArgumentMarshaler는 다른 파생 클래스와 똑같은 패턴이므로 코드를 생략한다.
- 한 가지가 눈에 거슬릴지 모르겠다. 바로 오류 코드 상수를 정의하는 부분이다. 
- 목록 14-7을 살펴본다.

#### 목록 14-7 ArgsException.java

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class ArgsException extends Exception { 
    private char errorArgumentId = '\0'; 
    private String errorParameter = null; 
    private ErrorCode errorCode = OK;
    
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
    
    public String errorMessage() { 
        switch (errorCode) {
        case OK:
            return "TILT: Should not get here.";
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
        case INVALID_ARGUMENT_NAME:
            return String.format("'%c' is not a valid argument name.", errorArgumentId);
        case INVALID_ARGUMENT_FORMAT:
            return String.format("'%s' is not a valid argument format.", errorParameter);
        }
        return ""; 
    }
  
    public enum ErrorCode {
        OK, INVALID_ARGUMENT_FORMAT, UNEXPECTED_ARGUMENT, INVALID_ARGUMENT_NAME, 
        MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, MISSING_DOUBLE, INVALID_DOUBLE
    }
}
```

</div>
</details>

- 이처럼 단순한 개념을 구현하는데 코드가 너무 많이 필요해 놀랄지도 모르겠다. 
- 한 가지 이유는 우리가 장황한 언어인 자바를 사용해서인데, 자바는 정적 타입 언어라서 타입 시스템을 만족하려면 많은 단어가 필요하다. [루비](https://github.com/unclebob/rubyargs/tree/master), 파이썬 등과 같은 언어를 사용하면 프로그램이 훨씬 작아졌을것이다.
- 이름을 붙인 방법, 함수 크기, 코드 형식에 각별히 주목해 읽어보길 바란다.
    - 예를 들어, 날짜 인수나 복소수 인수 등 새로운 인수 유형을 추가하는 방법이 명백하다. 고칠 코드도 별로 없다.
    - ArgumentMarshaler에서 새 클래스를 파생해 getXXX 함수를 추가한 후 parseSchemaElement 함수에 새 case 문만 추가하면 끝이다.

### <a name = '2'> 어떻게 짰느냐고? </a>

- 처음부터 저렇게 구현하지 않았다. 깨끗한 코드를 짜려면 먼저 **지저분한 코드를 짠 뒤에 정리해야 한다**는 의미다.
- 대다수 신참 프로그래머는 이 충고를 충실히 따르지 않는다. 
- 그들은 무조건 돌아가는 프로그램을 목표로 잡고, 일단 프로그램이 '돌아가면' 다음 업무로 넘어간다. '돌아가는' 프로그램은 상태가 어떻든 그대로 둔다.
- 경험이 풍부한 전문 프로그래머라면 이런 행동이 전문가로서 자살 행위라는 사실을 잘 안다.

## <a name = '3'> Args: 1차 초안 p.255 </a>

- 목록 14-8은 맨 처음 짰던 Args 클래스다. 코드는 '돌아가지만' 엉망이다.

#### 목록 14-8 Args.java(1차 초안)

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
        else if (isIntegerSchemaElement(elementTail)) {
            parseIntegerSchemaElement(elementId);
        } else {
            throw new ParseException(
                String.format("Argument: %c has invalid format: %s.", elementId, elementTail), 0);
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
        for (currentArgument = 0; currentArgument < args.length; currentArgument++) 
        {
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
    
    private boolean isIntArg(char argChar) {return intArgs.containsKey(argChar);}
    
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

- 이처럼 지저분한 코드를 보고 처음 든 생각이 "저자가 그냥 버려두지 않아서 진짜 다행이야!"이기 바란다.
- 만약 그렇다면 자신이 대충 짜서 남겨둔 코드를 남들이 어떻게 느낄지 생각하기 바란다.

<br>

- 목록 14-8은 명백히 미완성이다. 
- 많은 인스턴스 변수 개수, "TILT"와 같은 희한한 문자열, HashSets, TreeSets, try-catch-catch 블록 등 모두가 지저분한 코드에 기여하는 요인이다.
- 처음부터 지저분한 코드를 짜려는 생각은 없었다.
- 실제로도 코드를 어느 정도 손보려 애썼지만, 어느 순간 프로그램은 내 손을 벗어났다.
- 코드는 조금씩 엉망이 되어갔다. 첫 버전은 이만큼 엉망이지 않았다. 

<br>

- 목록 14-9는 Boolean 인수만 지원하던 초기 버전이다.

<details>
<summary> 목록 14-9 Args.java (Boolean만 지원하는 버전) </summary>

<div markdown = "1">

```java
package com.objectmentor.utilities.getopts;

import java.util.*;

public class Args {
    private String schema;
    private String[] args;
    private boolean valid;
    private Set<Character> unexpectedArguments = new TreeSet<Character>();
    private Map<Character, Boolean> booleanArgs = new HashMap<Character, Boolean>();
    private int numberOfArguments = 0;
    
    public Args(String schema, String[] args) {
        this.schema = schema;
        this.args = args;
        valid = parse();
    }
    
    public boolean isValid() {
        return valid;
    }
    
    private boolean parse() {
        if (schema.length() == 0 && args.length == 0)
            return true;
        parseSchema();
        parseArguments();
        return unexpectedArguments.size() == 0;
    }

    private boolean parseSchema() {
        for (String element : schema.split(",")) {
            parseSchemaElement(element);
        }
        return true;
    }
    
    private void parseSchemaElement(String element) {
        if (element.length() == 1) {
            parseBooleanSchemaElement(element);
        }
    }
    
    private void parseBooleanSchemaElement(String element) {
        char c = element.charAt(0);
        if (Character.isLetter(c)) {
            booleanArgs.put(c, false);
        }
    }
    
    private boolean parseArguments() {
        for (String arg : args) 
            parseArgument(arg);
        return true;
    }
    
    private void parseArgument(String arg) {
        if (arg.startsWith("-"))
            parseElements(arg);
    }
    
    private void parseElements(String arg) {
        for (int i = 1; i < arg.length(); i++) 
            parseElement(arg.charAt(i));
    }
    
    private void parseElement(char argChar) {
        if (isBoolean(argCahr)) {
            numberOfArguments++;
            setBooleanArg(argChar, true);
        } else
            unexpectedArguments.add(argChar);
    }
    
    private void setBooleanArg(char argChar, boolean value) {
        booleanArgs.put(argChar, value);
    }
    
    private boolean isBoolean(char argChar) {
        return booleanArgs.containsKey(argChar);
    }
    
    public int cardinality() {
        return numberOfArguments;
    }
    
    public String usage() {
        if (schema.length() > 0) 
            return "-["+schema+"]";
        else 
            return "";
    }
    
    public String errorMessage() {
        if (unexpectedArguments.size() > 0) {
            return unexpectedArgumentMessage();
        } else
            return "";
    }
    
    private String unexpectedArgumentMessage() {
        StringBuffer message = new StringBuffer("Arguments(s) -");
        for (char c : unexpectedArguments) {
            message.append(c);
        }
        message.append(" unexpected.");
        
        return message.toString();
    }
    
    public boolean getBoolean(char arg) {
        return booleanArgs.get(arg);
    }
}

```

</div>
</details>

- 위 코드도 불평 거리가 많겠지만 나름대로 괜찮은 코드다. 
- 간결하고 단순하며 이해하기도 쉽지만, 이후에 코드가 점차 지저분해진 이유가 분명히 드러난다.

<br>

- 다음 코드는 위 코드에 String과 Integer라는 인수 유형 두 개만 추가했을 뿐인데 유지 보수가 적당히 수월했던 코드가 버그와 결함이 숨어있을지도 모른다는 상당히 의심스러운 코드로 뒤바뀌어버렸다.
- 다음은 String 인수 유형을 추가한 코드다.

<details>
<summary> 목록 14-10 Args.java (Boolean과 String) </summary>

<div markdown = "1">

```java
package com.objectmentor.utilities.getopts;

import java.util.*;

public class Args {
    private String schema;
    private String[] args;
    private boolean valid;
    private Set<Character> unexpectedArguments = new TreeSet<Character>();
    private Map<Character, Boolean> booleanArgs = new HashMap<Character, Boolean>();
    private Map<Character, String> stringArgs = new HashMap<Character, String>();
    private Set<Character> argsFound = new HashSet<Character>();
    private int currentArgument;
    private char errorArgumentId = '\0';
    
    enum ErrorCode {
        OK, MISSING_STRING}

    private ErrorCode errorCode = ErrorCode.OK;
    
    public Args(String schema, String[] args) {
        this.schema = schema;
        this.args = args;
        valid = parse();
    }
    
    private boolean parse() {
        if (schema.length() == 0 && args.length == 0)
            return true;
        parseSchema();
        parseArguments();
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
    }

    private void validateSchemaElementId(char elementId) throws ParseException {
        if (!Character.isLetter(elementId)) {
            throw new ParseException("Bad character:" + elementId + "in Args format: " + schema, 0);
        }
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

    private void parseBooleanSchemaElement(char elementId) {
        booleanArgs.put(elementId, false);
    }

    private boolean parseArguments() {
        for (currentArgument = 0; currentArgument < args.length; currentArgument++)
        {
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
        boolean set = true;
        if (isBooleanArg(argChar))
            setBooleanArg(argChar, true);
        else if (isStringArg(argChar))
            setStringArg(argChar, "");
        else
            set = false;

        return set;
    }

    private void setIntArg(char argChar, String s) {
        currentArgument++;
        try {
            stringArgs.put(argChar, args[currentArgument]);
        } catch (ArrayIndexOutOfBoundsException e) {
            valid = false;
            errorArgument = argChar;
            errorCode = ErrorCode.MISSING_INTEGER;
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
        if (unexpectedArguments.size() > 0) {
            return unexpectedArgumentMessage();    
        } else 
        switch (errorCode) {
            case MISSING_STRING:
                return String.format("Could not find string parameter for -%c.", errorArgumentId);
            case OK:
                throw new Exception("TILT: Should not get here.");
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

    public boolean getBoolean(char arg) {
        return falseIfNull(booleanArgs.get(arg));
    }

    private boolean falseIfNull(Boolean b) {
        return b != null && b;
    }

    public String getString(char arg) {
        return blankIfNull(stringArgs.get(arg));
    }

    private String blankIfNull(String s) {
        return s == null ? "" : s;
    }

    public boolean has(char arg) {
        return argsFound.contains(arg);
    }

    public boolean isValid() {
        return valid;
    }
    
}

```

</div>
</details>

- 보다시피 코드는 통제를 벗어나기 시작했다. 
- 아직은 엉망이라 부르기 어렵지만, 여기다 Integer 인수 유형을 추가하니 코드는 완전히 엉망이 되어버렸다.

### <a name = '4'> 그래서 멈췄다 </a>

- 추가할 인수 유형이 적어도 두 개는 더 있었는데 그러면 코드가 훨씬 더 나빠지리라는 사실이 자명했다. 
- 계속 밀어붙이면 프로그램은 어떻게든 완성하겠지만 그랬다가는 너무 커서 손대기 어려운 골칫거리가 생겨날 참이었다. 
- 코드 구조를 유지보수하기 좋은 상태로 만들려면 지금이 적기라 판단했다.

<br>

- 그래서 나는 기능을 더 추가하지 않기로 결정하고 리팩토링을 시작했다. 
- String, Integer 인수 유형을 추가한 경험에서 새 인수 유형을 추가하려면 주요 지점 세 곳에다 코드를 추가해야 한다는 사실을 이미 깨달았다.

<br> 

- 인수 유형은 다양하지만 모두가 유사한 메서드를 제공하므로 클래스 하나가 적합하다 판단했고, ArgumentMarshaler라는 개념이 탄생했다.

### <a name = '5'> 점진적으로 개선하다 </a>

- 프로그램을 망치는 가장 좋은 방법 중 하나는 개선이라는 이름 아래 구조를 크게 뒤집는 행위다. 
- 어떤 프로그램은 그저 그런 '개선'에서 결코 회복하지 못하는데, '개선' 전과 똑같이 프로그램을 돌리기가 아주 어렵기 때문이다.

<br>

- 그래서 나는 테스트 주도 개발(`Test-Driven Development, TDD`)라는 기법을 사용했다. 
- 다시 말해, TDD는 시스템을 망가뜨리는 변경을 허용하지 않는다. 변경을 가한 후에도 시스템이 변경 전과 똑같이 돌아가야 한다는 말이다.

<br>

- 변경 전후 시스템이 똑같이 돌아간다는 사실을 확인하려면 언제든 실행이 가능한 자동화된 테스트 슈트가 필요하다. 
- 앞서 Args 클래스를 구현하는 동안에 나는 이미 단위 테스트 슈트와 인수 테스트를 만들어 놓았다. 
- 두 테스트 모두 언제든 실행이 가능했으며, 시스템이 두 테스트를 모두 통과하면 올바로 동작한다고 봐도 좋았다.

<br>

- 그래서 나는 시스템에 자잘한 변경을 가하기 시작했고, 코드를 변경할 때마다 시스템 구조는 조금씩 ArgumentMarshaler 개념에 가까워졌다.
- 또한 변경 후에도 시스템은 변경 전과 다름없이 돌아갔다. 

<br>

목록 14-11은 기존 코드 끝에 ArgumentMarshaler 클래스의 골격을 추가한 코드다.

<details>
<summary> 목록 14-11 Args.java 끝에 추가한 ArgumentMarshaler </summary>

<div markdown = "1">

```java
private class ArgumentMarshaler { 
    private boolean booleanValue = false;

    public void setBoolean(boolean value) { 
        booleanValue = value;
    }
    
    public boolean getBoolean() {return booleanValue;} 
}

private class BooleanArgumentMarshaler extends ArgumentMarshaler {
   
}

private class StringArgumentMarshaler extends ArgumentMarshaler {

}

private class IntegerArgumentMarshaler extends ArgumentMarshaler { 

}
```

</div>
</details>

- 당연히 위와 같은 변경은 아무 문제도 일으키지 않았다. 
- 다음으로 나는 코드를 최소로 건드리는, 가장 단순한 변경을 가했다. 
- 구체적으로 Boolean 인수를 저장하는 HashMap에서 Boolean 인수 유형을 ArgumentMarshaler 유형으로 바꿨다.

<details>
<summary> HashMap에서 Boolean 인수 유형을 ArgumentMarshaler 유형으로 </summary>

<div markdown = "1">

```java
private Map<Character, ArgumentMarshaler> boolean Args = new HashMap<Character, ArgumentMarshaler>();
```

</div>

- 그러면 코드 일부가 깨지기에 재빨리 고쳤다.

<div markdown = "1">

```java
...

private void parseBooleanSchemaElement(char elementId) {
    booleanArgs.put(elementId, new BooleanArgumentMarshaler());
}
  
...

private void setBooleanArg(char argChar, boolean value) {
    booleanArgs.get(argChar).setBoolean(value);
}
  
...

public boolean getBoolean(char arg) {
    return falseIfNull(booleanArgs.get(arg).getBoolean());
}
```

</div>

- 앞서 새 인수 유형을 추가하려면 세 곳(parse, get, set)을 변경해야 한다고 말했는데, 위에서 수정한 부분과 정확히 일치한다. 
- 불행히도, 사소한 변경이었으나 몇몇 테스트 케이스가 실패하기 시작했다.

<br>

- 다음 변경으로 넘어가기 전에 이것부터 고쳐야 했고, 실제로 그다지 어렵지는 않았다. null 점검 위치만 바꿔주면 충분했다.

<div markdown = "1">

```java
public boolean getBoolean(char arg) {
    return booleanArgs.get(arg).getBoolean();
}
```

</div>

- 다음으로 함수를 두 행으로 쪼갠 후 ArgumentMarshaler를 argumentMarshaler라는 독자적인 변수에 저장했다. 
- 유형 이름과 중복이 심했고, 함수가 길어 argumentMarshaler를 am으로 줄였다.

<div markdown = "1">

```java
public boolean getBoolean(char arg) {
    Args.ArgumentMarshaler am = booleanArgs.get(arg);
    return am.getBoolean();
}
```

</div>

- 그런 다음 null을 점검했다.

<div markdown = "1">

```java
public boolean getBoolean(char arg) {
    Args.ArgumentMarshaler am = booleanArgs.get(arg);
    return am != null && am.getBoolean();
}
```

</div>
</details>

## <a name = '6'> String 인수 p.272 </a>

- String 인수를 추가하는 과정은 boolean 인수와 매우 유사했다. 
- HashMap을 변경한 후 parse, set, get 함수를 고쳤다. 

<details>
<summary> String 인수 추가 </summary>

<div markdown = "1">

```java
private Map<Character, ArgumentMarshaler> stringArgs = new HashMap<Character, ArgumentMarshaler>();
...
private void parseStringSchemaElement(char elementId) {
    stringArgs.put(elementId, new StringArgumentMarshaler());
}
...
private void setStringArg(char argChar) throws ArgsException {
    currentArgument++;
    try {
        stringArgs.get(argChar).setString(args[currentArgument]);
    } catch (ArrayIndexOutOfBoundsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorCode = ErrorCode.MISSING_STRING;
        throw new ArgsException();
    }
}
...
public String getString(char arg) {
    Args.ArgumentMarshaler am = stringArgs.get(arg);
    return am == null ? "" : am.getString();
}
...
private class ArgumentMarshaler {
    private boolean booleanValue = false;
    private String stringValue;

    public void setBoolean(boolean value) {
        booleanValue = value;
    }

    public boolean getBoolean() {
        return booleanValue;
    }
    
    public void setString(String s) {
        stringValue = s;
    }
    
    public String getSring() {
        return stringValue == null ? "" : stringValue;
    }
}
```
</div>

</details>

- 앞서 구현과 마찬가지로 한 번에 하나씩 고치면서 테스트를 계속 돌렸고, 하나라로 실패하면 다음 변경으로 넘어가기 전에 오류를 수정했다.
- 일단 각 인수 유형을 처리하는 코드를 모두 ArgumentMarshaler 클래스에 넣고 나서 ArgumentMarshaler 파생 클래스를 만들어 코드를 분리할 작정이었다. 
- 그러면 프로그램 구조를 조금씩 변경하는 동안에도 시스템의 정상 동작을 유지하기 쉬워지기 때문이다.
- 다음으로 int 인수 기능을 ArgumentMarshaler로 옮겼다.

<details>
<summary> int 인수 기능 ArgumentMarshaler로 옮기기 </summary>

<div markdown = "1">

```java
private Map<Character, ArgumentMarshaler> intArgs = new HashMap<Character, ArgumentMarshaler>();
...
private void parseIntegerSchemaElement(char elementId) {
    intArgs.put(elementId, new IntegerArgumentMarshaler());
}
...
private void setIntArg(char argChar) throws ArgsException {
    currentArgument++;
    String parameter = null;
    try {
        parameter = args[currentArgument];
        intArgs.get(argChar).setInteger(Integer.parseInt(parameter));
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

...

public int getInt(char arg) {
    Args.ArgumentMarshaler am = new intArgs.get(arg);
    return am == null ? 0 : am.getInteger();
}

...

private class ArgumentMarshaler {
    private boolean booleanValue = false;
    private String stringValue;
    private int integerValue;

    public void setBoolean(boolean value) {
        booleanValue = value;
    }

    public boolean getBoolean() {
        return booleanValue;
    }

    public void setString(String s) {
        stringValue = s;
    }

    public String getSring() {
        return stringValue == null ? "" : stringValue;
    }

    public void setInteger(int i) {
        integerValue = i;
    }

    public int getInteger() {
        return integerValue;
    }

}
```
</div>

- 이제 모든 논리를 ArgumentMarshaler로 옮겼으니 파생 클래스를 만들어 기능을 분산한다. 

<div markdown = "1">

```java
private abstract class ArgumentMarshaler {
    protected boolean booleanValue = true;
    private String stringValue;
    private int integerValue;

    public void setBoolean(boolean value) {
        booleanValue = value;
    }

    public boolean getBoolean() {
        return booleanValue;
    }

    public void setString(String s) {
        stringValue = s;
    }

    public String getSring() {
        return stringValue == null ? "" : stringValue;
    }
    
    public void setInteger(int i) {
        integerValue = i;
    }
    
    public int getInteger() {
        return integerValue;
    }

    public abstract void set(String s);
}
```
</div>

- 그런 다음 BooleanArgumentMarshaler 클래스에 set 메서드를 구현했다.

<div markdown = "1">

```java
private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    public void set(String s) {
        booleanValue = true;
    }
}
```

</div>

- 마지막으로 setBoolean 호출을 set으로 바꿨다.

<div markdown = "1">

```java
private void setBooleanArg(char argChar, boolean value) {
    booleanArgs.get(argChar).set("true");
}
```

</div>
</details>

- 코드는 여전히 모든 테스트를 통과했다. 
- 이제 set 기능을 BooleanArgumentMarshaler로 옮겼으므로 ArgumentMarshaler에서 setBoolean 메서드를 제거했다.
- 다음으로 get 메서드를 booleanArgumentMarshaler로 옮겼다. 
- get 함수는 반환 객체 유형이 Object여야 하기때문에 옮기기 어렵다. 
- 여기서는 Boolean으로 형변환 되어야 한다.

<details>
<summary> get 메서드 booleanArgumentMarshaler로 옮기기 </summary>

<div markdown = "1">

```java
public boolean getBoolean(char arg) {
    Arg.Argumentmarshaler am = booleanArgs.get(arg);
    return am != null && (Boolean) am.get();
}
```

- 순전히 위 코드를 컴파일할 목적으로 ArgumentMarshaler에 get 함수를 추가했다.

```java
private abstract class ArgumentMarshaler {
    ...
    public Object get() {
        return null;
    }
}
```

- 코드는 컴파일되었으나 테스트는 실패했다. 
- 테스트를 통과하기 위해 ArgumentMarshaler에서 get을 추상 메서드로 만든 후 BooleanArgumentMarshaler에다 get을 구현했다.

```java
private abstract class ArgumentMarshaler {
    protected boolean booleanValue = false;
    ...

    public abstract Object get();
}

private class BoolenaArgumentMarshalser extends ArgumentMarshaler {
    public void set(String s) {
        booleanValue = true;
    }

    public Object get() {
        return booleanValue;
    }
}
```
</div>
</details>

- 코드는 모든 테스트를 통과했다! 
- ArgumentMarshaler에서 getBoolean 함수를 제거한 후 protected 변수인 booleanValue를 BoolenaArgumentMarshalser로 내려 private 변수로 선언한다.
- String 인수 유형도 동일한 방식으로 변경했다. 
- set과 get을 옮긴 후 사용하지 않는 함수를 제거하고 변수를 옮겼다.

<details>
<summary> String 인수 유형 변경 </summary>

<div markdonw = "1">

```java
private void setStringArg(char argChar) throws ArgsException {
    currentArgument++;
    try {
        stringArgs.get(argChar).set(args[currentArgument]);
    } catch (ArrayIndexOutOfBoundsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorCode = ErrorCode.MISSING_STRING;
        throw new ArgsException();
    }
}
...
public String getString(char arg) {
    Args.c am = stringArgs.get(arg);
    return am == null ? "" : (String) am.get();
}
...
private class ArgumentMarshaler {
    private int integerValue;

    public void setInteger(int i) {
        integerValue = i;
    }

    public int getInteger() {
        return integerValue;
    }
    
    public abstract void set(String s);
    
    public abstract Object get();
}

private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    private boolean booleanValue = false;
    
    public void set(String s) {
        booleanValue = true;
    }
    
    public Object get() {
        return booleanValue;
    }
}

private class StringArgumentMarshaler extends ArgumentMarshaler {
    private String stringValue = "";
    
    public void set(String s) {
        stringValue = s;
    }
    
    public Object get() {
        return stringValue;
    }
}

private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    public void set(String s) {
    }
    
    public Object get() {
        return null;
    }
}
```
</div>
</details>

- 마지막으로 integer 인수 유형에도 같은 과정을 반복했다. 
- integer 인수는 구문을 분석해야 하므로, 즉 parse에서 예외를 던질지도 모르므로, 앞서 구현보다 조금 더 복잡했다. 
- 하지만 NumberFormatException 이라는 개념 전체가 IntegerArgumentMarshaler에 숨겨지므로 결과는 더 좋았다. 

<details>
<summary> integer 인수 유형에도 같은 과정 반복 </summary>

<div markdown = "1">

```java
private boolean isIntArg(char argChar) {return intArgs.containsKey(argChar);}

private void setIntArg(char argChar) throws ArgsException {
    currentArgument++;
    String parameter = null;
    try {
        parameter = args[currentArgument];
        intArgs.get(argChar).set(parameter);
    } catch (ArrayIndexOutOfBoundsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorCode = ErrorCode.MISSING_INTEGER;
        throw new ArgsException();
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorParameter = parameter;
        errorCode = ErrorCode.INVALID_INTEGER;
        throw e;
    }
}
...
private void setBooleanArg(char argChar) {
    try {
        booleanArgs.get(argChar).set("true");
    } catch (ArgsException e) {
    }
}
...
public int getInt(cahr arg) {
    Args.Argumentmarshaler am = intArgs.get(arg);
    return am == null ? 0 : (Integer) am.get();
}
...
private abstract class ArgumentMarshaler {
    public abstract void set(String s) throws ArgsException;
    public abstract Object get();
}
...
private class IntegerArgumentmarshaler extends ArgumentMarshaler {
    private int intValue = 0;
    
    public void set(String s) throws ArgsException {
        try {
            intValue = Integer.parseInt(s);
        } catch (NumberFormatException e) {
            throw new ArgsException();
        }
    }
    
    public Object get() {
        return intValue;
    }
}
```

- 물론 코드는 테스트를 계속 통과했다.
- 다음으로 나는 알고리즘 처음에 나오는 (인수 유형마다 따로 만든) 맵 세 개를 없앴다. 
- 맵을 그냥 없애면 시스템이 깨지므로, 그 대신 나는 ArgumentMarshaler로 맵을 만들어 원래 맵을 교체하고 관련 메서드를 변경했다. 

```java
public class Args {
...
    private Map<Character, Boolean> booleanArgs = new HashMap<Character, Boolean>();
    private Map<Character, String> stringArgs = new HashMap<Character, String>();
    private Map<Character, Integer> intArgs = new HashMap<Character, Integer>();
    private Map<Character, Integer> marshalers = new HashMap<Character, Integer>();
...
    private void parseBooleanSchemaElement(char elementId) {
        ArgumentMarshaler m = new ArgumentMarshaler();
        booleanArgs.put(elementId, m);
        marshalers.put(elementId, m);
    }

    private void parseIntegerSchemaElement(char elementId) {
        ArgumentMarshaler m = new ArgumentMarshaler();
        intArgs.put(elementId, new IntegerArgumentMarshaler());
        marshalers.put(elementId, m);
    }

    private void parseStringSchemaElement(char elementId) {
        ArgumentMarshaler m = new ArgumentMarshaler();
        stringArgs.put(elementId, "");
        marshalers.put(elementId, m);
    }
}
```

- 물론 코드는 모든 테스트를 계속 통과했다. 다음으로 isBooleanArg를

```java
private boolean isBooleanArg(char argChar) {
    return booleanArgs.containsKey(argChar); 
}
```

에서

```java
private boolean isBooleanArg(char argChar) {
    ArgumentMarshaler m = marshalers.get(argChar);
    return m instanceof BooleanArgumentMarshaler;
}
```

로 변경했다.

- 코드는 테스트를 계속 통과했다. 그래서 isIntArg와 isStringArg도 똑같이 변경했다.

```java
private boolean isIntArg(char argChar) {
    ArgumentMarshaler m = marshalers.get(argChar);
    return m instanceof IntegerArgumentMarshaler;
}

private boolean isStringArg(char argChar) {
    ArgumentMarshaler m = marshalers.get(argChar);
    return m instanceof StringArgumentMarshaler;
}
```

- 코드는 테스트를 계속 통과했다. 
- 그래서 marshalers.get을 호출하는 코드를 모두 제거 했다.

```java
private boolean setArgument(char argChar) throws ArgsException { 
    ArgumetnMarshaler m = marshlers.get(argChar);
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

private boolean isIntArg(ArgumetnMarshaler m) {
    return m instanceof IntegerArgumentMarshaler;
}

private boolean isStringArg(ArgumetnMarshaler m) {
    return m instanceof StringArgumentMarshaler;
}

private boolean isBooleanArg(ArgumetnMarshaler m) {
    return m instanceof BooleanArgumentMarshaler;
}
```

- isxxxArg메서드도 별도로 선언할 이유가 없어졌다. 
- 그래서 인라인 코드로 만들었다.

```java
private boolean setArgument(char argChar) throws ArgsException { 
    if (m instanceof BooleanArgumentMarshaler)
        setBooleanArg(argChar); 
    else if (m instanceof StringArgumentMarshaler)
        setStringArg(argChar); 
    else if (m instanceof IntegerArgumentMarshaler)
        setIntArg(argChar); 
    else
        return false;
    
    return true; 
}
```
</div>
</details>

- 다음으로 set 함수에서 기존 Hashmap을 marshalers Hashmap으로 교체하기 시작했다. 
- 먼저 boolean 인수부터 시작했다.

<details>
<summary> Hashmap을 marshalers Hashmap 으로 교체</summary>

<div markdown = "1">

```java
private boolean setArgument(char argChar) throws ArgsException { 
    if (m instanceof BooleanArgumentMarshaler)
        setBooleanArg(m); 
    else if (m instanceof StringArgumentMarshaler)
        setStringArg(argChar); 
    else if (m instanceof IntegerArgumentMarshaler)
        setIntArg(argChar); 
    else
        return false;
    
    return true; 
}
...
private void setBooleanArg(ArgumentMarshaler m) {
    try {
        m.set("true"); // 이전 코드 : booleanArgs.get(argChar).set("true");
    } catch (ArgsException e) {
    }
}
```

- 코드는 테스트를 계속 통과했다. 
- 그래서 String 과 Integer도 똑같이 변경했다. 
- 이로써 일부 흉한 예외 관리 코드를 setArgument 함수에 넣을 수 있었다.

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumetnMarshaler m = marshlers.get(argChar);
    try {
        if (m instanceof BooleanArgumentMarshaler)
            setBooleanArg(m);
        else if (m instanceof StringArgumentMarshaler)
         setStringArg(argChar);
        else if (m instanceof IntegerArgumentMarshaler)
            setIntArg(argChar);
        else
            return false;
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
    return true;
}

private void setIntArg(ArgumetnMarshaler m) throws ArgsException {
    currentArgument++;
    String parameter = null;
    try {
        parameter = args[currentArgument];
        m.set(parameter);
    } catch (ArrayIndexOutOfBoundsException e) {
        errorCode = ErrorCode.MISSING_INTEGER;
        throw new ArgsException();
    } catch (ArgsException e) {
        errorParameter = parameter;
        errorCode = ErrorCode.INVALID_INTEGER;
        throw e;
    }
}

private void setStringArg(ArgumetnMarshaler m) throws ArgsException {
    currentArgument++;
    try {
        m.set(args[currentArgument]);
    } catch (ArrayIndexOutOfBoundsException e) {
        errorCode = ErrorCode.MISSING_STRING;
        throw new ArgsException();
    }
}   
```

- 이제 원래 맵 세 개를 제거해도 괜찮은 시점에 가까워졌다. 
- 먼저 아래 getBoolean 함수를

```java
public boolean getBoolena(char arg) {
    Args.ArgumentMarshaler am = booleanArgs.get(arg);
    return am != null && (Boolean) am.get();
}
```

아래와 같이 변경했다.

```java
public boolean getBoolean(char arg) {
    Args.Argumentmarshaler am = marshalers.get(arg);    
    boolean b = false;
    try {
        b = am != null && (Boolean) am.get();    
    } catch (ClassCastException e) {
        b = false;    
    }
    return b;
}
```
</div>

- ClassCastException을 사용한 이유는 인수 테스트 케이스를 FitNess에서 구현했기 때문이다. (단위 테스트 집합은 FitNess에서 구현하지 않았다.) 
- 직전 변경 덕택에 이전 boolean 맵을 사용하는 코드를 제거할 수 있게 되었다.

<pre>
private void parseBooleanSchemaElement(char elementId) {
    ArgumentMarshaler m = new ArgumentMarshaler();
    <del>booleanArgs.put(elementId, m);</del> 
    marshalers.put(elementId, m);
}
</pre>

- 이로써 boolean 맵도 제거가 가능하다.

<pre>
...
<del>private Map&lt;Character, Boolean&gt; booleanArgs = new HashMap&lt;Character, Boolean&gt;();</del>
private Map&lt;Character, String&gt; stringArgs = new HashMap&lt;Character, String&gt;();
private Map&lt;Character, Integer&gt; intArgs = new HashMap&lt;Character, Integer&gt;();
private Map&lt;Character, Integer&gt; marshalers = new HashMap&lt;Character, Integer&gt;();
...
</pre>

- 다음으로 String과 Integer 인수도 boolean과 똑같이 변경하고 이전 맵을 제거했다.

<pre>
private void parseBooleanSchemaElement(char elementId) {
    marshalers.put(elementId, <b>new BooleanArgumentMarshaler()</b>);
}

private void parseIntegerSchemaElement(char elementId) {
    marshalers.put(elementId, <b>new IntegerArgumentMarshaler()</b>);
}

private void parseStringSchemaElement(char elementId) {
    marshalers.put(elementId, <b>new StringArgumentMarshaler()</b>);
}
...
public String getString(char arg) {
    Args.ArgumentMarshaler am = <b>marshalers</b>.get(arg);
    <b>try {</b>
        return am == null ? "" : (String) am.get();    
    <b>} catch (ClassCastException e) {
        return "";    
    }</b>
}

public int getInt(char arg) {
    Args.ArgumentMarshaler am = <b>marshalers</b>.get(arg);
    <b>try {</b>
        return am == null ? 0 : (Integer) am.get();    
    <b>} catch (ClassCastException e) {
        return 0;    
    }</b>
}
...
public class {
...
<del>private Map&lt;Character, String&gt; stringArgs = new HashMap&lt;Character, String&gt;();</del>
<del>private Map&lt;Character, Integer&gt; intArgs = new HashMap&lt;Character, Integer&gt;();</del>
private Map&lt;Character, Integer&gt; marshalers = new HashMap&lt;Character, Integer&gt;();
...
}
</pre>

- 다음으로 이제 거의 사용하지 않는 parse 메서드 세 개를 인라인 코드로 만들었다.

```java
private void parseSchemaElement(String element) throws ParseException { 
    char elementId = element.charAt(0);
    String elementTail = element.substring(1); 
    validateSchemaElementId(elementId);
    if (isBooleanSchemaElement(elementTail)) 
        marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (isStringSchemaElement(elementTail)) 
        marshalers.put(elementId, new StringArgumentMarshaler());
    else if (isIntegerSchemaElement(elementTail)) {
        marshalers.put(elementId, new IntegerArgumentMarshaler());
    } else {
        throw new ParseException(String.format("Argument: %c has invalid format: %s.", elementId, elementTail), 0);
    } 
}
```

</details>

- 아래 목록 14-12는 현재 Args 클래스가 도달한 상태다.

### 목록 14-12 Args.java(첫 번째 리팩터링을 끝낸 버전)

```java
package com.objectmentor.utilities.args;

import java.text.ParseException;
import java.util.*;

public class Args {
    private String schema;
    private String[] args;
    private boolean valid = true;
    private Set<Character> unexpectedArguments = new TreeSet<Character>();
    private Map<Character, ArgumentMarshaler> marshalers = new HashMap<Character, ArgumentMarshaler>();
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
           marshalers.put(elementId, new BooleanArgumentMarshaler());
        else if (isStringSchemaElement(elementTail)) 
            marshalers.put(elementId, new StringArgumentMarshaler());
        else if (isIntegerSchemaElement(elementTail)) {
            marshalers.put(elementId, new IntegerArgumentMarshaler());
        } else {
            throw new ParseException(String.format("Argument: %c has invalid format: %s.", elementId, elementTail), 0);
        }
    }

    private void validateSchemaElementId(char elementId) throws ParseException {
        if (!Character.isLetter(elementId)) {
            throw new ParseException("Bad character:" + elementId + "in Args format: " + schema, 0);
        }
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
        for (currentArgument = 0; currentArgument < args.length; currentArgument++)
        {
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
        ArgumetnMarshaler m = marshlers.get(argChar);
        try {
            if (m instanceof BooleanArgumentMarshaler)
                setBooleanArg(m);
            else if (m instanceof StringArgumentMarshaler)
                setStringArg(argChar);
            else if (m instanceof IntegerArgumentMarshaler)
                setIntArg(argChar);
            else
                return false;
        } catch (ArgsException e) {
            valid = false;
            errorArgumentId = argChar;
            throw e;
        }
        return true;
    }

    private void setIntArg(ArgumetnMarshaler m) throws ArgsException {
        currentArgument++;
        String parameter = null;
        try {
            parameter = args[currentArgument];
            m.set(parameter);
        } catch (ArrayIndexOutOfBoundsException e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (ArgsException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw e;
        }
    }

    private void setStringArg(ArgumetnMarshaler m) throws ArgsException {
        currentArgument++;
        try {
            m.set(args[currentArgument]);
        } catch (ArrayIndexOutOfBoundsException e) {
            errorCode = ErrorCode.MISSING_STRING;
            throw new ArgsException();
        }
    }
    
    private void setBooleanArg(ArgumentMarshaler m) {
        try {
            m.set("true");
        } catch (ArgsException e) {
        }
    }

    public int cardinality() {
        return argsFound.size();
    }

    public String usage() {
        if (schema.length() > 0)
            return "-["+schema+"]";
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

    public boolean getBoolean(char arg) {
        Args.ArgumentMarshaler am = booleanArgs.get(arg);
        boolean b = false;
        try {
            b = am != null && (Boolean) am.get();
        } catch (ClassCastException e) {
            b = false;
        }
        return b;
    }
    
    public String getString(char arg) {
        Args.ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? "" : (String) am.get();
        } catch (ClassCastException e) {
            return "";
        }
    }
    
    public int getInt(char arg) {
        Args.ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? 0 : (Integer) am.get();
        } catch (ClassCastException e) {
            return 0;
        }
    }
    
    public boolean has(char arg) {
        return argsFound.contains(arg);
    }
    
    public boolean isValid() {
        return valid;
    }
    
    private class ArgsException extends Exception {
    }

    private abstract class ArgumentMarshaler {
        public abstract void set(String s) throws ArgsException;
        public abstract Object get();
    }

    private class BooleanArgumentMarshaler extends ArgumentMarshaler {
        private boolean booleanValue = true;
        
        public void set(String s) {
            booleanValue = true;
        }

        public Object get() {
            return booleanValue;
        }
    }

    private class StringArgumentMarshaler extends ArgumentMarshaler {
        private String stringValue = "";

        public void set(String s) {
            stringValue = s;
        }

        public Object get() {
            return stringValue;
        }
    }

    private class IntegerArgumentMarshaler extends ArgumentMarshaler {
        private int intValue = 0;

        public void set(String s) throws ArgsException {
            try {
                intValue = Integer.parseInt(s);
            } catch (NumberFormatException e) {
                throw new ArgsException();
            }
        }

        public Object get() {
            return intValue;
        }
    }
    
}
```

- 열심히 고쳤건만 결과는 다소 실망스럽다. 구조만 조금 나아졌을 뿐이다. 
- 유형을 일일이 확인하는 코드, 오류 처리 코드, 모든 set 함수 등이 그대로 남아있다. 
- 아직도 할 일은 아주 많다.
- 아래 코드는 10단계에 걸쳐 이뤄졌다. 물론 단계마다 코드는 테스트를 통과했다.

<details>
<summary> 10단계에 걸쳐 고친 코드 </summary>

<pre>
public class Args {
    private String schema;
    <del>private String[] args;</del>
    private boolean valid = true;
    private Set&lt;Character&gt; unexpectedArguments = new TreeSet&lt;Character&gt;();
    private Map&lt;Character, Integer> marshalers = new HashMap&lt;Character, Integer&gt;();
    private Set&lt;Character&gt; argsFound = new TreeSet&lt;Character&gt;();
    private <b>Iterator&lt;String&gt;</b> currentArgument;
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;
    <b>private List&lt;String&gt; argsList;</b>

    private enum ErrorCode {
        OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT}

    public Args(String schema, String[] args) throws ParseException {
        this.schema = schema;
        <b>argsList = Arrays.asList(args);</b>
        valid = parse();
    }
    
    private boolean parse() throws ParseException {
        if (schema.length() == 0 && <b>args.size()</b> == 0)
            return true;
        parseSchema();
        try {
            parseArguments();
        } catch (ArgsException e) {
        }
        return valid;
    }
---
    private boolean parseArguments() throws ArgsException {
        for (currentArgument = <b>argsList.iterator();</b> currentArgument.<b>hasNext();</b>) {
            String arg = currentArgument<b>.next();</b>
            parseArgument(arg);
        }

        return true;
    }
---
    private void setIntArg(ArgumetnMarshaler m) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument<b>.next();</b>
            m.set(parameter);
        } catch (<b>NoSuchElementException</b> e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (ArgsException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw e;
        }
    }

    private void setStringArg(ArgumentMarshaler m) throws ArgsException {
        try {
            m.set(currentArgument<b>.next();</b>
        } catch (<b>NoSuchElementException</b> e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        }
    }

</pre>

- 변경은 간단했으며 코드는 모든 테스트를 통과했다. 
- 이제 set 함수를 적절한 파생 클래스로 내려도 괜찮아졌다.

<pre>

private boolean setArgument(char argChar) throws ArgsException {
    ArgumetnMarshaler m = marshlers.get(argChar);
    <b>if (m == null)
        return false;</b>
    try {
        if (m instanceof BooleanArgumentMarshaler)
            setBooleanArg(m);
        else if (m instanceof StringArgumentMarshaler)
            setStringArg(argChar);
        else if (m instanceof IntegerArgumentMarshaler)
            setIntArg(argChar);
        <del>else
            return false;</del>
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
    return true;
}

</pre>
</details>

- 위 변경은 중요하다. 
- if-else가 연쇄적으로 이어지는 구문을 완전히 제거하기 위해서다. 
- 그래서 if-else 연쇄문에서 오류 코드를 꺼내고, set 함수를 옮겼다.

<details>
<summary> if-else 연쇄문에서 오류 코드 꺼내기 </summary>

<pre>

private boolean setArgument(char argChar) throws ArgsException {
    ArgumetnMarshaler m = marshlers.get(argChar);
    if (m == null)
        return false;
    try {
        if (m instanceof BooleanArgumentMarshaler)
            setBooleanArg(m, <b>currentArgument</b>);
        else if (m instanceof StringArgumentMarshaler)
            setStringArg(argChar);
        else if (m instanceof IntegerArgumentMarshaler)
            setIntArg(argChar);

    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
    return true;
}
---
private void setBooleanArg(ArgumentMarshaler m, <b>Iterator&lt;String&gt; currentArgument)</b> throws ArgsException {
    <del>try {</del>
        m.set("true");
    <del>catch (ArgsException e) {</del>
    <del>}</del>
}

</pre>

</details>

- 리팩터링을 하다보면 코드를 넣었다 뺐다 하는 사례가 아주 흔하다. - 큰 목표 하나를 이루기 위해 자잘한 단계를 수없이 거친다.
- 각 단계를 거쳐야 다음 단계가 가능하다.
- 이제 setBooleanArg 함수는 필요없다. 
- ArgumentMarshaler에 set 함수가 있다면 직접 호출할 수 있으므로 set 함수를 만든다. 

<details>
<summary> set함수 만들기 </summary>

```java
private abstract class ArgumentMarshaler {
    public abstract void set(Iterator<String> currentArgument) throws ArgsException;
    public abstract void set(String s) throws ArgsException;
    public abstract Object get();
}
```

- 물론 ArgumentMarshaler에 새로운 추상 메서드를 선언하면 모든 파생 클래스가 컴파일에 실패한다.
- 그래서 각 파생 클래스에도 set 메서드를 추가한다.

<pre>
private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    private boolean booleanValue = false;    
    
    <b>public abstract void set(Iterator&lt;String&gt; currentArgument) throws ArgsException {
        booleanValue = true;
    }</b>

    public void set(String s) {
        <del>booleanValue = true;</del>
    }
    
    public Object get() {
        return booleanValue;
    }
}

private class StringArgumentMarshaler extends ArgumentMarshaler {
    private String stringValue = "";
    
    <b>public abstract void set(Iterator&lt;String&gt; currentArgument) throws ArgsException {
    }</b>

    public void set(String s) {
        stringValue = s;
    }
    
    public Object get() {
        return stringValue;
    }
}

private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;

    <b>public abstract void set(Iterator&lt;String&gt; currentArgument) throws ArgsException {
        booleanValue = true;
    }</b>
    
    public void set(String s) throws ArgsException {
        try {
            intValue = Integer.parseInt(s);
        } catch (NumberFormatException e) {
            throw new ArgsException();
        }
    }
    
    public Object get() {
        return intValue;
    }
}
</pre>

- 이제 setBooleanArg를 제거해도 안전하다.

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumetnMarshaler m = marshlers.get(argChar);
    if (m == null)
        return false;
    try {
        if (m instanceof BooleanArgumentMarshaler)
            m.set(currentArgument);
        else if (m instanceof StringArgumentMarshaler)
            setStringArg(m);
        else if (m instanceof IntegerArgumentMarshaler)
            setIntArg(m);

    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
    return true;
}
```

</details>

- 코드는 모든 테스트를 통과한다. 
- 이제 String 과 Integer 인수도 똑같이 변경할 차례다.

<details>
<summary> String, Integer 인수에도 적용 </summary>

```java
private class StringArgumentMarshaler extends ArgumentMarshaler {
    private String stringValue = "";
    
    public abstract void set(Iterator<String> currentArgument) throws ArgsException {
        try {
            stringValue = currentArgument.next();
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_STRING;
            throw new ArgsException();
        }
    }

    public void set(String s) {
        stringValue = s;
    }
    
    public Object get() {
        return stringValue;
    }
}

private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;
    
    public abstract void set(Iterator<String> currentArgument) throws ArgsException {
        
        String parameter = null;
        try {
            parameter = currentArgument.next();
            set(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (ArgsException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw e;
        }
    }

    public void set(String s) throws ArgsException {
        try {
            intValue = Integer.parseInt(s);
        } catch (NumberFormatException e) {
            throw new ArgsException();
        }
    }
    
    public Object get() {
        return intValue;
    }
}
```

- 이제 마지막이다. 인수 유형을 일일이 확인하던 코드를 제거해도 괜찮다.

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumetnMarshaler m = marshlers.get(argChar);
    if (m == null)
        return false;
    try {
        m.set(currentArgument);
        return true;
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
}
```

- 다음으로 IntegerArgumentMarshaler에서 몇 가지 허술한 코드를 고쳐 정리한다.

```java
private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;
    
    public abstract void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            intValue = Integer.parseInt(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (NumberFormatException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw new ArgsException();
        }
    }
    
    public Object get() {
        return intValue;
    }
}
```

- 또한 ArgumentMarshaler를 인터페이스로 변환한다.

```java
private interface ArgumentMarshaler {
    void set(Iterator<String> currentArgument) throws ArgsException;
    Object get();
}
```

</details>

- 이제부터 우리 구조에 새로운 인수 유형을 추가하기 쉬워졌다.
- 변경할 코드는 아주 적으며 나머지 시스템에 영향을 미치지 않는다. 
- 우선 시스템이 double 인수 유형을 제대로 받아들이는지 확인할 테스트 케이스부터 추가한다.

<details>
<summary> double 인수 확인할 테스트 케이스 </summary>

<div markdown = "1">

```java
public void testSimpleDoublePresent() throws Exception {
    Args args = new Args("x##", new String[] {"-x","42.3"});
    assertTrue(args.isValid());
    assertEquals(1, args.cardinality());
    assertTrue(args.has('x'));
    assertEquals(42.3, args.getDouble('x'), .001);
}
```

</div>

- 이제 스키마 구문 분석 코드를 정리하고 ## 감지 코드를 추가한다. 
- 여기서 ##는 double 인수 유형을 뜻한다.

```java
private void parseSchemaElement(String element) throws ArgsException { 
    char elementId = element.charAt(0);
    String elementTail = element.substring(1); 
    validateSchemaElementId(elementId);
    if (elementTail.length() == 0)
        marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (elementTail.equals("*")) 
        marshalers.put(elementId, new StringArgumentMarshaler());
    else if (elementTail.equals("#"))
        marshalers.put(elementId, new IntegerArgumentMarshaler());
    else if (elementTail.equals("##")) 
        marshalers.put(elementId, new DoubleArgumentMarshaler());
    else
        throw new ArgsException(INVALID_ARGUMENT_FORMAT, elementId, elementTail);
}
```

- 다음으로 DoubleArgumentMarshaler 클래스를 작성한다.

```java
private class DoubleArgumentMarshaler extends ArgumentMarshaler {
    private double doubleValue = 0;
    
    public abstract void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            doubleValue = Double.parseDouble(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_DOUBLE;
            throw new ArgsException();
        } catch (NumberFormatException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_DOUBLE;
            throw new ArgsException();
        }
    }
    
    public Object get() {
        return doubleValue;
    }
}

```

- 여기서 새로운 ErrCode가 필요하다.

```java
public enum ErrorCode {
    OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, 
    UNEXPECTED_ARGUMENT, MISSING_DOUBLE, INVALID_DOUBLE
}
```

- 또한 getDouble 함수도 필요하다.

```java
public double getDouble(char arg) {
    Args.Argumentmarshaler am = marshalers.get(arg);
    try {
        return am == null ? 0 : (Double) am.get();
    } catch (Exception e) {
        return 0.0;
    }
}
```

- 코드는 모든 테스트를 통과하고, 어려움은 거의 없었다!
- 다음 테스트 케이스는 구문 분석이 불가능한 문자열을 ## 인수에 전달해 오류 처리 코드 동작을 확인한다.

```java
public void testSimpleDoublePresent() throws Exception {
    Args args = new Args("x##", new String[] {"-x","Forty two"});
    assertFalse(args.isValid());
    assertEquals(0, args.cardinality());
    assertFalse(args.has('x'));
    assertEquals(0, args.getInt('x'));
    assertEquals("Argument -x expects a double but was 'Forty two'.", args.errorMessage());
}
---
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
    case INVALID_DOUBLE:
        return String.format("Argument -%c expects a double but was '%s'.", errorArgumentId, errorParameter);
    case MISSING_DOUBLE:
        return String.format("Could not find double parameter for -%c.", errorArgumentId);
    }
    return ""; 
}
```

- 코드는 테스트를 통과한다.
- 다음 테스트 케이스는 double 인수를 빠뜨린 경우다.

```java
public void testMissingDouble() throws Exception {
    Args args = new Args("x##", new String[]{"-x"});
    assertFalse(args.isValid());
    assertEquals(0, args.cardinality());
    assertFalse(args.has("x"));
    assertEquals(0.0, args.getDouble("X"), 0.01);
    assertEquals("Could not find double parameter for -x.", args.errorMessage());
}
```

</details>

- 예상대로 코드는 테스트를 통과한다. 
- 예외 코드는 흉할뿐더러 Args 클래스에 속하지도 않는다. 
- 그러므로 모든 예외를 하나로 모아 ArgsException 클래스를 만든 후 독자 모듈로 옮긴다.

<details>
<summary> ArgsException 클래스에 예외 모으기 </summary>

```java
public class ArgsException extends Exception { 
    private char errorArgumentId = '\0'; 
    private String errorParameter = null; 
    private ErrorCode errorCode = OK;

    public ArgsException() {}
    
    public ArgsException(String message) {super(message);}

    public enum ErrorCode {
        OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, 
        UNEXPECTED_ARGUMENT, MISSING_DOUBLE, INVALID_DOUBLE
    }
}
---
public class Args {
    ...
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ArgsException.ErrorCode errorCode = ArgsException.ErrorCode.OK;
    private List<String> argsList;

    public Args(String schema, String[] args) throws ArgsException {
        this.schema = schema;
        argsList = Arrays.asList(args);
        valid = parse();
    }

    private boolean parse() throws ParseException {
        if (schema.length() == 0 && args.size() == 0)
            return true;
        parseSchema();
        try {
            parseArguments();
        } catch (ArgsException e) {
        }
        return valid;
    }

    private boolean parseArguments() throws ArgsException {
        ...
    }

    private void parseSchemaElement(String element) throws ArgsException { 
        ...
        else
            throw new ArgsException(
            String.format("Argument: %c has invalid format: %s.", elementId, elementTail));
    }

    private void validateSchemaElementId(char elementId) throws ArgsException { 
        if (!Character.isLetter(elementId)) {
            throw new ArgsException("Bad character:" + elementId + "in Args format: " + schema);
        }
    }
    ...
    private void parseElement(char argChar) throws ArgsException { 
        if (setArgument(argChar))
            argsFound.add(argChar); 
        else 
            unexpectedArguments.add(argChar); 
        errorCode = ArgsException.ErrorCode.UNEXPECTED_ARGUMENT; 
        valid = false;
    }
}
...
public class StringArgumentMarshaler implements ArgumentMarshaler { 
    private String stringValue = "";
    
    public void set(Iterator<String> currentArgument) throws ArgsException { 
        try {
            stringValue = currentArgument.next(); 
        } catch (NoSuchElementException e) {
            errorCode = ArgsException.ErrorCode.MISSING_STRING;
            throw new ArgsException();
        }
    }
  
    public Object get() {
        return stringValue;
    }
}

public class IntegerArgumentMarshaler implements ArgumentMarshaler { 
    private int intValue = 0;
  
    public void set(Iterator currentArgument) throws ArgsException { 
        String parameter = null;
        try {
            parameter = currentArgument.next();
            intValue = Integer.parseInt(parameter);
        } catch (NoSuchElementException e) {
            throw new ArgsException(MISSING_INTEGER);
        } catch (NumberFormatException e) {
            errorParameter = parameter;
            errorCode = ArgsException.ErrorCode.INVALID_INTEGER;
            throw new ArgsException(); 
        }
    }
  
    public Object get() {
        return intValue;
    }
}

private class DoubleArgumentMarshaler extends ArgumentMarshaler {
    private double doubleValue = 0;
    
    public abstract void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            doubleValue = Double.parseDouble(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ArgsException.ErrorCode.MISSING_DOUBLE;
            throw new ArgsException();
        } catch (NumberFormatException e) {
            errorParameter = parameter;
            errorCode = ArgsException.ErrorCode.INVALID_DOUBLE;
            throw new ArgsException();
        }
    }
    
    public Object get() {
        return doubleValue;
    }
}
```

</details>

- 멋지다. 이제 Args 클래스가 던지는 예외는 ArgsException 뿐이다. 
- ArgsException을 독자적인 모듈로 만들면 Args 모듈에서 잡다한 오류 지원 코드를 옮겨올 수 있다.
- Args 모듈에서 예외/오류 처리 코드를 완벽하게 분리했다. (목록 14-13 ~ 목록 14-16이 자세한 과정이다.)
- 물론 매 단계마다 코드는 테스트를 통과했다.

<details>
<summary> 목록 14-13 ArgsTest.java </summary>

<div markdown = "1">

```java
package com.objectmentor.utilities.args;

import junit.framework.TestCase;

public class ArgsTest extends TestCase {
    public void testCreateWithNoSchemaOrArguments() throws Exception {
        Args args = new Args("", new String[0]);
        assertEquals(0, args.cardinality());
    }
    
    public void testWithNoSchemaButWithOneArgument() throws Exception {
        try {
            new Args("", new String[]{"-x"});
            fail();
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, e.getErrorCode());
            assertEquals('x', e.getErrorArgumnetId());
        }
    }
    
    public void testWithNoSchemaButWithMultipleArgument() throws Exception {
        try {
            new Args("", new String[]{"-x", "-y"});
            fail();
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, e.getErrorCode());
            assertEquals('x', e.getErrorArgumnetId());
        }
    }
    
    public void testNonLetterSchema() throws Exception {
        try {
            new Args("*", new String[]{});
            fail("Args constructor should have thrown exception");
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_ARGUMENT_NAME, e.getErrorCode());
            assertEquals('*', e.getErrorArgumnetId());
        }
    }

    public void testInvalidArgumentFormat() throws Exception {
        try {
            new Args("f~", new String[]{});
            fail("Args constructor should have thrown exception");
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_FORMAT, e.getErrorCode());
            assertEquals('f', e.getErrorArgumnetId());
        }
    }
    
    public void testSimpleBooleanPresent() throws Exception {
        Args args = new Args("x", new String[]{"-x"});
        assertEquals(1, args.cardinality());
        assertEquals(true, args.getBoolean('x'));
    }

    public void testSimpleStringPresent() throws Exception {
        Args args = new Args("x*", new String[]{"-x", "param"});
        assertEquals(1, args.cardinality());
        assertTrue(args.has('x'));
        assertEquals("param", args.getString('x'));
    }

    public void testMissingStringPresent() throws Exception {
        try {
            new Args("x*", new String[]{"-x"});
            fail();
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.MISSING_STRING, e.getErrorCode());
            assertEquals('x', e.getErrorArgumnetId());
        }
    }
    
    public void testSpacesInFormat() throws Exception {
        Args args = new Args("x, y", new String[]{"-xy"});
        assertEquals(2, args.cardinality());
        assertTrue(args.has('x'));
        assertTrue(args.has('y'));
    }

    public void testSimpleIntPresent() throws Exception {
        Args args = new Args("x#", new String[]{"-x", "42"});
        assertEquals(1, args.cardinality());
        assertTrue(args.has('x'));
        assertEquals(42, args.getInt('x'));
    }
    
    public void testInvalidInteger() throws Exception {
        try {
            new Args("x#", new String[]{"-x", "Forty two"});
            fail();
        } catch(ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_INTEGER, e.getErrorCode());
            assertEquals('x', e.getErrorArgumnetId());
            assertEquals("Forty two", e.getErrorArgumnetId());
        }
    }
    
    public void testMissingInteger() throws Exception {
        try {
            new Args("x#", new String[]{"-x"});
            fail();
        } catch(ArgsException e) {
            assertEquals(ArgsException.ErrorCode.MISSING_INTEGER, e.getErrorCode());
            assertEquals('x', e.getErrorArgumnetId());
        }
    }

    public void testSimpleDoublePresent() throws Exception {
        Args args = new Args("x##", new String[] {"-x","42.3"});
        assertEquals(1, args.cardinality());
        assertTrue(args.has('x'));
        assertEquals(42.3, args.getDouble('x'), .001);
    }

    public void testInvalidDouble() throws Exception {
        try {
            new Args("x##", new String[]{"-x","Forty two"});
            fail();
        } catch(ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_DOUBLE, e.getErrorCode());
            assertEquals('x', e.getErrorArgumnetId());
            assertEquals("Forty two", e.getErrorArgumnetId());
        }
    }

    public void testMissingDouble() throws Exception {
        try {
            new Args("x##", new String[]{"-x"});
            fail();
        } catch(ArgsException e) {
            assertEquals(ArgsException.ErrorCode.MISSING_DOUBLE, e.getErrorCode());
            assertEquals('x', e.getErrorArgumnetId());
        }
    }    
}
```

</div>
</details>

<details>
<summary> 목록 14-14 ArgsExceptionTest.java </summary>

<div markdown = "1">

```java
public class ArgsExceptionTest extends TestCase {
    public void testUnexpectedMessage() throws Exception {
        ArgsException e = new ArgsException(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, 'x', null);
        assertEquals("Argument -x unexpected.", e.errorMessage());
    }

    public void testMissingMessage() throws Exception {
        ArgsException e = new ArgsException(ArgsException.ErrorCode.MISSING_STRING, 'x', null);
        assertEquals("Could not find string parameter for -x.", e.errorMessage());
    }

    public void testInvalidIntegerMessage() throws Exception {
        ArgsException e = new ArgsException(ArgsException.ErrorCode.INVALID_INTEGER, 'x', "Forty two");
        assertEquals("Argument -x expects an integer but was 'Forty two'.", e.errorMessage());
    }

    public void testMissingIntegerMessage() throws Exception {
        ArgsException e = new ArgsException(ArgsException.ErrorCode.MISSING_INTEGER, 'x', null);
        assertEquals("Could not find integer parameter for -x.", e.errorMessage());
    }

    public void testInvalidDoubleMessage() throws Exception {
        ArgsException e = new ArgsException(ArgsException.ErrorCode.INVALID_DOUBLE, 'x', "Forty two");
        assertEquals("Argument -x expects a double but was 'Forty two'.", e.errorMessage());
    }

    public void testMissingDoubleMessage() throws Exception {
        ArgsException e = new ArgsException(ArgsException.ErrorCode.MISSING_DOUBLE, 'x', null);
        assertEquals("Could not find double parameter for -x.", e.errorMessage());
    }
}
```

</div>
</details>

<details>
<summary> 목록 14-15 ArgsException.java </summary>

<div markdown = "1">

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

    public String errorMessage() {
        switch (errorCode) {
            case OK:
                return "TILT: Should not get here.";
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
        OK, INVALID_FORMAT, UNEXPECTED_ARGUMENT, INVALID_ARGUMENT_NAME,
        MISSING_STRING, 
        MISSING_INTEGER, INVALID_INTEGER, 
        MISSING_DOUBLE, INVALID_DOUBLE}
}
```

</div>
</details>

### 최종 코드 

목록 14-16 Args.java

```java
public class Args {
    private String schema;
    private Map<Character, ArgumentMarshaler> marshalers = new HashMap<Character, ArgumentMarshaler>();
    private Set<Character> argsFound = new HashSet<Character>();
    private Iterator<String> currentArgument;
    private List<String> argsList;

    public Args(String schema, String[] args) throws ArgsException {
        this.schema = schema;
        argsList = Arrays.asList(args);
        parse();
    }

    private void parse() throws ArgsException {
        parseSchema();
        parseArguments();
    }

    private boolean parseSchema() throws ArgsException {
        for (String element : schema.split(",")) {
            if (element.length() > 0) {
                parseSchemaElement(element.trim());
            }
        }
        return true;
    }

    private void parseSchemaElement(String element) throws ArgsException {
        char elementId = element.charAt(0);
        String elementTail = element.substring(1);
        validateSchemaElementId(elementId);
        if (elementTail.length() == 0)
            marshalers.put(elementId, new BooleanArgumentMarshaler());
        else if (elementTail.equals("*"))
            marshalers.put(elementId, new StringArgumentMarshaler());
        else if (elementTail.equals("#"))
            marshalers.put(elementId, new IntegerArgumentMarshaler());
        else if (elementTail.equals("##"))
            marshalers.put(elementId, new DoubleArgumentMarshaler());
        else
            throw new ArgsException(ArgsException.ErrorCode.INVALID_FORMAT, elementId, elementTail);
    }

    private void validateSchemaElementId(char elementId) throws ArgsException {
        if (!Character.isLetter(elementId)) {
            throw new ArgsException(ArgsException.ErrorCode.INVALID_ARGUMENT_NAME, elementId, null);
        }
    }

    private void parseArguments() throws ArgsException {
        for (currentArgument = argsList.iterator(); currentArgument.hasNext();) {
            String arg = currentArgument.next();
            parseArgument(arg);
        }
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
            throw new ArgsException(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, argChar, null);
    }

    private boolean setArgument(char argChar) throws ArgsException {
        ArgumentMarshaler m = marshalers.get(argChar);
        if (m == null)
            return false;
        try {
            m.set(currentArgument);
            return true;
        } catch (ArgsException e) {
            e.setErrorArgumentId(argChar);
            throw e;
        }
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

    public boolean getBoolean(char arg) {
        ArgumentMarshaler am = marshalers.get(arg);
        boolean b = false;
        try {
            b = am != null && (Boolean) am.get();
        } catch (ClassCastException e) {
            b = false;
        }
        return b;
    }

    public String getString(char arg) {
        ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? "" : (String) am.get();
        } catch (ClassCastException e) {
            return "";
        }
    }

    public int getInt(char arg) {
        ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? 0 : (Integer) am.get();
        } catch (Exception e) {
            return 0;
        }
    }

    public double getDouble(char arg) {
        ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? 0 : (Double) am.get();
        } catch (Exception e) {
            return 0.0;
        }
    }

    public boolean has(char arg) {
        return argsFound.contains(arg);
    }
}
```

- Args 클래스에서 코드 중복을 최소화하고, 상당한 코드를 Args 클래스에서 ArgsException 클래스로 옮겼다. 
- 또한 모든 ArgumentMarshaler 클래스도 각자 파일로 옮겼다. 
- 더욱 멋지다!

<br>

- 소프트웨어 설계는 분할만 잘해도 품질이 크게 높아진다. 
- 적절한 장소를 만들어 코드만 분리해도 설계가 좋아진다. 
- 관심사를 분리하면 코드를 이해하고 보수하기 훨씬 더 쉬워진다.

<br>

- 특별히 눈여겨 볼 코드는 ArgsException의 errorMessage 메서드다. 
- (Args 클래스에 속했던) 이 메서드는 Args 클래스가 오류 메시지 형식까지 책임졌기에 명백히 SRP 위반이었다.
- Args 클래스는 인수를 처리하는 클래스지 오류 메시지 형식을 처리하는 클래스가 아니기 때문이다. 
- 하지만 그렇다고 ArgsException 클래스가 오류 메시지 형식을 처리해야 옳을까?
- 솔직하게 말해, 이것은 절충안이다. 
- ArgsException에게 맡겨서는 안 된다고 생각하는 독자라면 새로운 클래스가 필요하다. 
- 하지만 미리 깔끔하게 만들어진 오류 메시지로 얻는 장점은 무시하기 어렵다.

## <a name = '7'> 결론 p.321 </a>

- 그저 돌아가는 코드만으로는 부족하다. 돌아가는 코드가 심하게 망가지는 사례는 흔하다.
- 나쁜 코드보다 더 오랫동안 더 심각하게 개발 프로젝트에 악영향을 미치는 요인도 없다.
- 나쁜 코드는 썩어 문드러지고, 점점 무게가 늘어나 팀의 발목을 잡는다.
- 나쁜 코드도 깨끗한 코드로 개선할 수 있지만, 비용이 엄청나게 많이 든다.
- 오래된 의존성을 찾아내 깨려면 상당한 시간과 인내심이 필요하다.
- 반면 처음부터 코드를 깨끗하게 유지하기란 상대적으로 쉽다.
- 그러므로 코드는 언제나 최대한 깔끔하고 단순하게 정리하자.
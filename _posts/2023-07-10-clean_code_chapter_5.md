---
title: Chapter 05 - 형식 맞추기
date: 2023-07-10 12:00:00 +09:00
categories: [Book, Clean Code]
tags:
  [
    clean code,
  ]
---

## 형식을 맞추는 목적
- 오늘 구현한 기능이 다음 버전에서 바뀔 확률은 아주 높다.
- 하지만 구현한 코드의 가독성은 앞으로 바뀔 코드의 품질에 지대한 영향을 미친다.

## 적절한 행 길이를 유지하라

### 신문 기사처럼 작성하라
- 이름은 간단하면서도 설명이 가능하게 짓는다.
- 소스 파일 첫 부분은 고차원 개념과 알고리즘을 설명한다.
- 아래로 내려갈수록 의도를 세세하게 묘사한다.
- 마지막에는 가장 저차원 함수와 세부 내역을 작성한다.

### 개념은 빈 행으로 분리하라
- 일련의 행 묶음은 완결된 생각 하나를 표현한다.
- 생각 사이는 빈 행을 넣어 분리해야 마땅하다.
- 예를 들어, 패키지 선언부, import 문, 각 함수 사이에 빈 행을 넣는다.

```java
// Bad
package fitnesse.wikitext.widgets;
import java.util.regex.*;
public class BoldWidget extends ParentWidget {
	public static final String REGEXP = "'''.+?'''";
	private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
		Pattern.MULTILINE + Pattern.DOTALL);
	public BoldWidget(ParentWidget parent, String text) throws Exception {
		super(parent);
		Matcher match = pattern.matcher(text); match.find(); 
		addChildWidgets(match.group(1));}
	public String render() throws Exception { 
		StringBuffer html = new StringBuffer("<b>"); 		
		html.append(childHtml()).append("</b>"); 
		return html.toString();
	} 
}
```

```java
// Good
package fitnesse.wikitext.widgets;

import java.util.regex.*;

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
	
	public String render() throws Exception { 
		StringBuffer html = new StringBuffer("<b>"); 
		html.append(childHtml()).append("</b>"); 
		return html.toString();
	} 
}
```

### 세로 밀집도
- 세로 밀집도는 연관성을 의미한다. 서로 밀접한 코드 행은 세로로 가까이 놓여야 한다.

```java
// Bad
// 의미없는 주석으로 변수를 떨어뜨려 놓아서 한눈에 파악이 잘 안된다.

public class ReporterConfig {
	/**
	* The class name of the reporter listener 
	*/
	private String m_className;
	
	/**
	* The properties of the reporter listener 
	*/
	private List<Property> m_properties = new ArrayList<Property>();
	public void addProperty(Property property) { 
		m_properties.add(property);
	}
```

```java
// Good
// 의미 없는 주석을 제거함으로써 코드가 한눈에 들어온다.
// 변수 2개에 메소드가 1개인 클래스라는 사실이 드러난다.

public class ReporterConfig {
	private String m_className;
	private List<Property> m_properties = new ArrayList<Property>();
	
	public void addProperty(Property property) { 
		m_properties.add(property);
	}
```


### 수직 거리
- 서로 밀접한 개념은 세로로 가까이 둬야 한다.
	- 다른 파일에 속할 타당한 근거가 없다면, 서로 밀접한 개념은 한 파일에 속해야 한다.
	- `protected` 변수를 피해야 하는 이유 중 하나다.
- 변수
	- 변수는 사용하는 위치에 최대한 가까이 선언한다.
	- 루프를 제어하는 변수는 루프 문 내부에 선언한다.
	- 드물지만, 다소 긴 함수에서 블록 상단이나 루프 직전에 변수를 선언하는 사례도 있다.
- 인스턴스 변수
	- 클래스 맨 처음에 선언한다. 변수 간에 세로로 거리를 두지 않는다.
- 종속 함수
	- 한 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다.
	- 호출하는 함수를 호출되는 함수보다 먼저 배치한다.
- 개념적 유사성
	- 개념적인 친화도가 높을수록 코드를 가까이 배치한다.
	- 개념적인 친화도 : 호출 관계, 변수 상용 관계, 기본 기능 유사성 등

### 세로 순서
- 호출되는 함수를 호출하는 함수보다 나중에 배치한다. (파스칼, C, C++와 반대)
- 그러면 소스 코드 모듈이 고차원에서 저차원으로 자연스럽게 내려간다.
- 가장 중요한 개념을 가장 먼저 표현한다. 이때는 세세한 사항을 최대한 배제한다.

## 가로 형식 맞추기
- 한 행에 가로로는 짧은 것이 바람직하다.
- (저자 개인적으로는) 120자 정도로 행 길이를 제한한다.

## 가로 공백과 밀집도
- 가로로는 공백을 사용해 밀접한 개념과 느슨한 개념을 표현한다.
- 예를 들어, 할당(`=`) 시에는 equal 앞뒤에 공백을 준다.
	- `int lineSize = line.length();`
- 반면, 함수 이름과 이어지는 괄호 사이에는 공백을 넣지 않는다.(+ 인자 사이의 쉼표 뒤에는 공백을 넣는다.)
	- `lineWidthHistogram.addLine(lineSize, lineCount);`
- 연산자 우선순위를 강조하기 위해서도 공백을 사용한다.
	- `return b*b - 4*a*c;`

## 가로 정렬

```java
// Bad
public class FitNesseExpediter implements ResponseSender {
	private		Socket		  socket;
	private 	InputStream 	  input;
	private 	OutputStream 	  output;
	private 	Reques		  request; 		
	private 	Response 	  response;	
	private 	FitNesseContex	  context; 
	protected 	long		  requestParsingTimeLimit;
	private 	long		  requestProgress;
	private 	long		  requestParsingDeadline;
	private 	boolean		  hasError;
	
	... 
```

## 들여쓰기
- 때로는 간단한 if 문, 짧은 while 문, 짧은 함수에서 들여쓰기 규칙을 무시하고 싶더라도, 반드시 들여쓰기를 하자

```java
// Bad
public class CommentWidget extends TextWidget {
	public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";
	
	public CommentWidget(ParentWidget parent, String text){super(parent, text);}
	public String render() throws Exception {return ""; } 
}
```

```java
// Good
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

## 가짜 범위
- 때로는 빈 while 문이나 for 문을 접한다.
- 가능한 한 이런 구조를 피하고, 어쩔 수 없다면 빈 블록을 올바로 들여쓰고 괄호로 감싼다.

```java
while (dis.read(buf, 0, readBufferSize) != -1)
;
```

## 팀 규칙
- 팀은 한 가지 규칙에 합의해야 한다. 그리고 모든 팀원은 그 규칙을 따라야 한다.
- 어디에 괄호를 넣을지, 들여쓰기는 몇 자로 할지, 클래스와 변수와 메서드 이름은 어떻게 지을지 등을 결정
- 좋은 소프트웨어 시스템은 읽기 쉬운 문서로 이뤄진다는 것을 기억하라

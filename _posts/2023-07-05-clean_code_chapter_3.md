---
title: Chapter 03 - 함수
date: 2023-07-05 12:00:00 +09:00
categories: [Book, Clean Code]
tags:
  [
    clean code,
  ]
---
### 작게 만들어라
- 함수를 만드는 첫째 규칙은 '작게'이다.
- if 문/else 문/while 문 등에 들어가는 블록은 한 줄이어야 한다.
  - 아마 대게 그 한줄은 함수 호출일 것이다.
  - 그러면 바깥을 감싸는 함수(enclosing function)가 작아질 뿐 아니라, 블록 안에서 호출하는 함수 이름을 적절히 짓는다면 코드 이해하기 쉬워진다.
- 중첩 구조가 생길만큼 함수가 커져서는 안된다.
- 함수에서 들여쓰기 수준은 1단이나 2단을 넘어서면 안된다.

### 한 가지만 해라
- 함수는 한 가지를 해야 한다. 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.
- `한 가지`의 정의란 무엇인가?
  - 추상화 수준이 하나인 단계만 수행한다면, 그 함수는 한 가지 작업만 한다고 할 수 있다.
  - 또는 함수 내부의 작업을 의미 있는 이름으로 다른 함수를 추출할 수 있다면, 그 함수는 여러 작업을 하는 셈이다.

### 함수 당 추상화 수준은 하나로
- 함수 내 모든 문장의 추상화 수준이 동일해야 한다.
- 예를 들어, `getHtml()`은 추상화 수준이 아주 높다. `PathParser.render(pagepath)`는 추상화 수준이 중간이다. `.append("\n")`는 추상화 수준이 아주 낮다.
- 한 함수 내에 추상화 수준을 섞으면, 읽는 사람은 특정 표현이 근본 개념인지 세부사항인지 구분하기 어렵다.
- 또한 깨어진 창문처럼 사람들이 함수에 세부사항을 점점 더 추가한다.

> 내려가기 규칙: 한 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 온다.

- 하지만 추상화 수준이 하나인 함수를 구현하기란 쉽지 않다.
- 핵심은 짧으면서도 한 가지만 하는 함수이다.

### Switch 문
- switch 문(또는 연속된 if-else문)은 작게 만들기 어렵다.
- 본질적으로 switch 문은 N가지를 처리하지만, 이를 완전히 피할 방법은 없다.
- 다형성을 이용하여 switch 문을 abstract factory에 숨기고, switch 문의 반복을 피할 수 있다.

```java
// Before
public Money calculatePay(Employee e) throws InvalidEmployeeType {
	switch (e.type) { 
		case COMMISSIONED:
			return calculateCommissionedPay(e); 
		case HOURLY:
			return calculateHourlyPay(e); 
		case SALARIED:
			return calculateSalariedPay(e); 
		default:
			throw new InvalidEmployeeType(e.type); 
	}
}
```
```java
// After
public abstract class Employee {
	public abstract boolean isPayday();
	public abstract Money calculatePay();
	public abstract void deliverPay(Money pay);
}
-----------------
public interface EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType; 
}
-----------------
public class EmployeeFactoryImpl implements EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
		switch (r.type) {
			case COMMISSIONED:
				return new CommissionedEmployee(r);
			case HOURLY:
				return new HourlyEmployee(r);
			case SALARIED:
				return new SalariedEmploye(r);
			default:
				throw new InvalidEmployeeType(r.type);
		} 
	}
}
```

### 서술적인 이름을 사용하라
- 함수명으로 서술적인 이름을 사용하는 것이, 함수가 하는 일을 더 잘 표현하므로 훨씬 좋은 이름이다.
- 이름이 길어도 괜찮다. 길고 서술적인 이름이 짧고 어려운 이름 또는 주석보다 좋다.
- 함수 이름을 정할 때는 여러 단어가 쉽게 읽히는 명명법을 사용한다.

### 함수 인수
- 함수에서 이상적인 인수 개수는 0개(무항)다. 다음은 1개, 다음은 2개다.
- 3개(삼항)는 가능한 피하는 편이 좋다. 4개 이상은 특별한 이유가 있더라도 사용하면 안된다.

**많이 쓰는 단항 형식**
- 인수에 질문을 던지는 경우
  `boolean fileExists(“MyFile”);`
- 인수를 뭔가로 변환해 결과를 반환하는 경우
  `InputStream fileOpen(“MyFile”);`
- 이벤트 함수
  `passwordAttemptFailedNtimes(int attempts)`
- 그 외의 경우, 가급적 단항 함수는 피한다.
  - 예를 들어, 변환 함수에서 출력 인수를 사용하면 혼란을 일으킨다. 함수라면 변환 결과는 반환값으로 돌려줘라.

**플래그 인수**
- 함수로 bool 값을 넘기지 마라
  - 플래그가 `true`면 이것을 하고, `false`면 저것을 처리한다고 대놓고 공표하는 셈

**이항 함수**
- 일반적으로 인수가 1개인 함수보다 이해하기 어렵다.
- 이항 함수가 적절한 경우
  `Point p = new Point(0, 0)`
- 이항 함수는 가능하면 단항 함수로 바꾸도록 애써야 한다.

**삼항 함수**
- 삼항 함수를 만들 때는 신중히 고려하라

**인수 객체**
- 인수가 2-3개 필요하다면 독자적인 클래스 변수로 선언할지 고려하라
  `Circle makeCircle(double x, double y, double radius);`
  `Circle makeCircle(Point center, double radius);`

**인수 목록**
- 때로는 인수 개수가 가변적인 함수도 필요하다.
- 대표적인 예시가 `String.format` 메서드이다.
  - `public String format(String format, Object... args)`

**동사와 키워드**
- 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다.
  - 예를 들어, `writeField(name)` 함수는 name이 field이고, 이를 write 한다는 것을 추측할 수 있다.
- 다항 함수에서는 함수 이름에 인수 이름을 순서대로 넣으면, 인수 순서를 기억할 필요가 없어진다.
  - `assertExpectedEqualsActual(expected, actual)`

### 부수 효과(Side Effect)를 일으키지 마라
- 예상치 못하게, 남몰래 다른 짓을 하지 마라
- 출력 인수를 사용하지 마라.
  - 대신 객체 지향 언어에서는 `this`를 사용하자
  - `void appendFooter(StringBuffer report)`를 호출해서 출력 인자를 사용하는것 보다는
  - `report.appendFooter()` 을 호출하는 방식이 더 좋다.

### 명령과 조회를 분리하라
- 예를 들어, `public boolean set(String attribute, String value);` 는 속성값을 value로 설정 후 성공하면 `true`를 리턴한다.
- 이 함수를 사용하면 `if(set("name", "bob"))` 과 같은 괴상한 코드가 나온다.
- 해결 방법은, 명령과 조회를 분리하는 방법이다.
  ```java
  if(attributeExists("name")){
    setAttribute("name", "bob");
  }
  ```

### 오류 코드보다 예외를 사용하라
```java
if (deletePage(page) == E_OK) {
	if (registry.deleteReference(page.name) == E_OK) {
		if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
			logger.log("page deleted");
		} else {
			logger.log("configKey not deleted");
		}
	} else {
		logger.log("deleteReference from registry failed"); 
	} 
} else {
	logger.log("delete failed"); return E_ERROR;
}
```

- try/catch 블록은 정상 동작과 오류 처리 동작을 뒤섞으므로, 별도 함수로 뽑아내는 편이 좋다.

```java
public void delete(Page page) {
	try {
		deletePageAndAllReferences(page);
  } catch (Exception e) {
  		logError(e);
  }
}

private void deletePageAndAllReferences(Page page) throws Exception { 
	deletePage(page);
	registry.deleteReference(page.name); 
	configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) { 
	logger.log(e.getMessage());
}
```

**오류 처리도 한 가지 작업이다.**
- 오류를 처리하는 함수는 오류만 처리해야 마땅하다.
- 함수에 `try`가 있다면, `catch/finally` 문으로 끝나야 한다.

### 반복하지 마라
- 중복은 코드 길이가 늘어날 뿐만 아니라, 동시 변경점이 여러곳에 생긴다.

### 구조적 프로그래밍
- 함수는 `return` 문이 하나여야 한다.
- 루프 안에서 `break`나 `continue`를 사용해선 안되며, `goto`는 절대로 안된다.
- 함수가 작다면 오히려 여러 개의 `return`, `break`, `continue`가 의도를 더 잘 표현하기도 한다.

### 함수를 어떻게 짜죠?
- 논문이나 기사의 초안은 대개 서투르고 어수선하다. 원하는 대로 읽힐 때까지 다듬고 문장을 고친다.
- 함수를 짤 때도 마찬가지다. 처음에는 길고 복잡하고 중복 코드가 많다.
- 이후 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거한다. 때로는 전체 클래스를 쪼갠다.

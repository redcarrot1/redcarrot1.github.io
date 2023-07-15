---
title: Chapter 10 - 클래스
date: 2023-07-14 12:00:00 +09:00
categories: [Book, Clean Code]
tags:
  [
    clean code,
  ]
---

## 클래스 체계
- 표준 자바 관례에 따른 내부 체계 순서
  1. static public 상수
  2. static private 변수
  3. private instance 변수
  4. public 함수
    - private 함수는 자신을 호출하는 public 함수 직후에 선언

**캡슐화**
- 변수와 유틸리티 함수는 가능한 숨기는 것이 좋지만, 반드시 그래야 하는 것은 아님
- 테스트를 위해 protected로 선언하기도 한다.
- 하지만 캡슐화를 풀어주는 결정은 언제나 최후의 수단

## 클래스는 작아야 한다!
- 클래스를 설계할 때도 (함수와 마찬가지로) '작게'가 기본 규칙이다.
- '작다'의 기준은 클래스가 맡은 책임의 수이다.
- 클래스 이름은 해당 클래스 책임을 기술해야 한다.
  - 클래스 이름이 모호하거나 간결한 이름이 떠오르지 않는다면, 클래스의 책임이 많아서 그렇다.
  - 예를 들어, Processor, Manager, Super 등과 같은 모호한 단어가 있는 경우
- 클래스 설명은 if, and, or, but을 사용하지 않고 25단어 내외로 가능해야 한다.
  - 예를 들어, "~는 컴포넌트에 접근하는 방법을 제공**하며**, 버전을 추적하는 메커니즘을 제공한다"
  - 위의 경우 클래스의 책임이 너무 많다는 증거이다.

**단일 책임 원칙(SRP)**
> 클래스나 모듈을 변경할 이유가 단 하나뿐이어야 한다.

- SRP는 OOP에서 중요하고 지키기 수월한 개념이지만, 많은 설계자가 무시한다.
  - '깨끗하고 체계적인 소프트웨어'보다 '돌아가는 소프트웨어'에 초점이 맞춰서 그렇다.
  - 자잘한 단일 책임 클래스가 많아지면 큰 그림을 이해하기 어려워진다고 우려한다.
    - 규모가 어느 수준에 이르는 시스템은 논리가 많고 복잡하므로 체계적인 정리가 필수다.

**응집도**
- 클래스는 인스턴스 변수 수가 작아야 한다.
- 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 한다.
  - 일반적으로 메서드가 변수를 많이 사용할수록 메서드와 클래스는 응집도가 더 높다.
- 일반적으로 응집도가 높은 클래스를 선호한다.
  - 클래스가 논리적인 단위로 묶인다는 의미이기 때문이다.
- 함수를 작게, 매개변수 목록을 짧게 유지하다 보면, 응집도가 낮아지게 된다.
  - 새로운 클래스로 쪼개야 한다는 신호이다.

```java
// Stack을 구현한 코드
// 응집도가 높다. (size()를 제외하곤 모든 메서드가 모든 변수를 사용한다.)

public class Stack {
    private int topOfStack = 0;
	List<Integer> elements = new LinkedList<Integer>();

	public int size() { 
		return topOfStack;
	}

	public void push(int element) { 
		topOfStack++; 
		elements.add(element);
	}

	public int pop() throws PoppedWhenEmpty { 
		if (topOfStack == 0)
			throw new PoppedWhenEmpty();
		int element = elements.get(--topOfStack); 
		elements.remove(topOfStack);
		return element;
	}
}
```

**응집도를 유지하면 작은 클래스 여럿이 나온다.**
- 큰 함수를 작은 함수 여럿으로 나누기만 해도 클래스 수가 많아진다.
- 예를 들어,
  - 큰 함수 일부를 작은 함수 하나로 빼고 싶다.
  - 근데 빼내려는 코드가 큰 함수에 정의된 변수 4개를 사용한다.
  - 그럼 작은 함수의 인수를 4개로 넣어줘야 할까?
  - 클래스로 분리하고, 4개의 변수를 인스턴스 변수로 승격하면 된다!

```java
// Bad
public class PrintPrimes {
	public static void main(String[] args) {
		final int M = 1000; 
		final int RR = 50;
		final int CC = 4;
		final int WW = 10;
		final int ORDMAX = 30; 
		int P[] = new int[M + 1]; 
		int PAGENUMBER;
		int PAGEOFFSET; 
		int ROWOFFSET; 
		int C;
		int J;
		int K;
		boolean JPRIME;
		int ORD;
		int SQUARE;
		int N;
		int MULT[] = new int[ORDMAX + 1];
		
		J = 1;
		K = 1; 
		P[1] = 2; 
		ORD = 2; 
		SQUARE = 9;
	
		while (K < M) { 
			do {
				J = J + 2;
				if (J == SQUARE) {
					ORD = ORD + 1;
					SQUARE = P[ORD] * P[ORD]; 
					MULT[ORD - 1] = J;
				}
				N = 2;
				JPRIME = true;
				while (N < ORD && JPRIME) {
					while (MULT[N] < J)
						MULT[N] = MULT[N] + P[N] + P[N];
					if (MULT[N] == J) 
						JPRIME = false;
					N = N + 1; 
				}
			} while (!JPRIME); 
			K = K + 1;
			P[K] = J;
		} 
		{
			PAGENUMBER = 1; 
			PAGEOFFSET = 1;
			while (PAGEOFFSET <= M) {
				System.out.println("The First " + M + " Prime Numbers --- Page " + PAGENUMBER);
				System.out.println("");
				for (ROWOFFSET = PAGEOFFSET; ROWOFFSET < PAGEOFFSET + RR; ROWOFFSET++) {
					for (C = 0; C < CC;C++)
						if (ROWOFFSET + C * RR <= M)
							System.out.format("%10d", P[ROWOFFSET + C * RR]); 
					System.out.println("");
				}
				System.out.println("\f"); PAGENUMBER = PAGENUMBER + 1; PAGEOFFSET = PAGEOFFSET + RR * CC;
			}
		}
	}
}
```

위 코드를 리팩토링하자.

```java
package literatePrimes;

public class PrimePrinter {
	public static void main(String[] args) {
		final int NUMBER_OF_PRIMES = 1000;
		int[] primes = PrimeGenerator.generate(NUMBER_OF_PRIMES);
		
		final int ROWS_PER_PAGE = 50; 
		final int COLUMNS_PER_PAGE = 4; 
		RowColumnPagePrinter tablePrinter = 
			new RowColumnPagePrinter(ROWS_PER_PAGE, 
						COLUMNS_PER_PAGE, 
						"The First " + NUMBER_OF_PRIMES + " Prime Numbers");
		tablePrinter.print(primes); 
	}
}
```

```java
package literatePrimes;

import java.io.PrintStream;

public class RowColumnPagePrinter { 
	private int rowsPerPage;
	private int columnsPerPage; 
	private int numbersPerPage; 
	private String pageHeader; 
	private PrintStream printStream;
	
	public RowColumnPagePrinter(int rowsPerPage, int columnsPerPage, String pageHeader) { 
		this.rowsPerPage = rowsPerPage;
		this.columnsPerPage = columnsPerPage; 
		this.pageHeader = pageHeader;
		numbersPerPage = rowsPerPage * columnsPerPage; 
		printStream = System.out;
	}
	
	public void print(int data[]) { 
		int pageNumber = 1;
		for (int firstIndexOnPage = 0 ; 
			firstIndexOnPage < data.length ; 
			firstIndexOnPage += numbersPerPage) { 
			int lastIndexOnPage =  Math.min(firstIndexOnPage + numbersPerPage - 1, data.length - 1);
			printPageHeader(pageHeader, pageNumber); 
			printPage(firstIndexOnPage, lastIndexOnPage, data); 
			printStream.println("\f");
			pageNumber++;
		} 
	}
	
	private void printPage(int firstIndexOnPage, int lastIndexOnPage, int[] data) { 
		int firstIndexOfLastRowOnPage =
		firstIndexOnPage + rowsPerPage - 1;
		for (int firstIndexInRow = firstIndexOnPage ; 
			firstIndexInRow <= firstIndexOfLastRowOnPage ;
			firstIndexInRow++) { 
			printRow(firstIndexInRow, lastIndexOnPage, data); 
			printStream.println("");
		} 
	}
	
	private void printRow(int firstIndexInRow, int lastIndexOnPage, int[] data) {
		for (int column = 0; column < columnsPerPage; column++) {
			int index = firstIndexInRow + column * rowsPerPage; 
			if (index <= lastIndexOnPage)
				printStream.format("%10d", data[index]); 
		}
	}

	private void printPageHeader(String pageHeader, int pageNumber) {
		printStream.println(pageHeader + " --- Page " + pageNumber);
		printStream.println(""); 
	}
		
	public void setOutput(PrintStream printStream) { 
		this.printStream = printStream;
	} 
}
```

```java
// 인스턴스화 하는 크래스가 아님을 주목
package literatePrimes;

import java.util.ArrayList;

public class PrimeGenerator {
	private static int[] primes;
	private static ArrayList<Integer> multiplesOfPrimeFactors;

	protected static int[] generate(int n) {
		primes = new int[n];
		multiplesOfPrimeFactors = new ArrayList<Integer>(); 
		set2AsFirstPrime(); 
		checkOddNumbersForSubsequentPrimes();
		return primes; 
	}

	private static void set2AsFirstPrime() { 
		primes[0] = 2; 
		multiplesOfPrimeFactors.add(2);
	}
	
	private static void checkOddNumbersForSubsequentPrimes() { 
		int primeIndex = 1;
		for (int candidate = 3 ; primeIndex < primes.length ; candidate += 2) { 
			if (isPrime(candidate))
				primes[primeIndex++] = candidate; 
		}
	}

	private static boolean isPrime(int candidate) {
		if (isLeastRelevantMultipleOfNextLargerPrimeFactor(candidate)) {
			multiplesOfPrimeFactors.add(candidate);
			return false; 
		}
		return isNotMultipleOfAnyPreviousPrimeFactor(candidate); 
	}

	private static boolean isLeastRelevantMultipleOfNextLargerPrimeFactor(int candidate) {
		int nextLargerPrimeFactor = primes[multiplesOfPrimeFactors.size()];
		int leastRelevantMultiple = nextLargerPrimeFactor * nextLargerPrimeFactor; 
		return candidate == leastRelevantMultiple;
	}
	
	private static boolean isNotMultipleOfAnyPreviousPrimeFactor(int candidate) {
		for (int n = 1; n < multiplesOfPrimeFactors.size(); n++) {
			if (isMultipleOfNthPrimeFactor(candidate, n)) 
				return false;
		}
		return true; 
	}
	
	private static boolean isMultipleOfNthPrimeFactor(int candidate, int n) {
		return candidate == smallestOddNthMultipleNotLessThanCandidate(candidate, n);
	}
	
	private static int smallestOddNthMultipleNotLessThanCandidate(int candidate, int n) {
		int multiple = multiplesOfPrimeFactors.get(n); 
		while (multiple < candidate)
			multiple += 2 * primes[n]; 
		multiplesOfPrimeFactors.set(n, multiple); 
		return multiple;
	} 
}
```

## 변경하기 쉬운 클래스
- 대다수 시스템은 지속적인 변경이 가해진다.
- 깨끗한 시스템은 클래스를 체계적으로 관리해 변경에 따르는 위험을 최대한 낮춘다.
- 새 기능을 추가할 때 시스템을 확장할 뿐 기존 코드를 변경하면 안된다.(OCP)

**변경으로부터 격리**
- 상세한 구현 클래스에 의존하는 클라이언트는 구현이 바뀌면 위험에 빠진다.
  - 인터페이스와 추상 클래스를 사용해 구현이 미치는 영향을 격리한다.
- 상세한 구현에 의존하는 코드는 테스트하기 어렵다.
  - 예를 들어, 5분마다 값이 달라지는 API를 직접 의존하는 클래스는 테스트하기 어렵다.
  - 대신 인터페이스를 만들어서 여기에 의존하자.
- 결합도를 줄이면 DIP 원칙을 따르게 된다.(구현이 아니라, 인터페이스에 의존해야 한다는 원칙)

  ```java
  // 인터페이스 구현 클래스에 외부 API를 호출
  public interface StockExchange { 
    Money currentPrice(String symbol);
  }
  ```

  ```java
  // 외부 API를 의존하던 것을 인터페이스에 의존하도록 변경
  public Portfolio {
    private StockExchange exchange;
    public Portfolio(StockExchange exchange) {
      this.exchange = exchange; 
    }
    // ... 
  }
  ```

  ```java
  // 테스트 코드를 작성하기 쉬워졌다. (외부 API를 Mocking 가능하므로)

  public class PortfolioTest {
    private FixedStockExchangeStub exchange;
    private Portfolio portfolio;
    
    @Before
    protected void setUp() throws Exception {
      // Mocking을 만들어서 portfolio에 주입
      exchange = new FixedStockExchangeStub(); 
      exchange.fix("MSFT", 100);
      portfolio = new Portfolio(exchange);
    }

    @Test
    public void GivenFiveMSFTTotalShouldBe500() throws Exception {
      portfolio.add(5, "MSFT");
      Assert.assertEquals(500, portfolio.value()); 
    }
  }
  ```

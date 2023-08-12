## 개요

### 11장에서 다루는 내용

- 비공개 메서드 단위 테스트
- 단위 테스트를 하기 위한 비공개 메소드 노출
- 테스트로 유출된 도메인 지식
- 구체 클래스 목 처리

## 11.1 비공개 메서드 단위 테스트

비공개 메서드를 노출하면 테스트가 구현 세부 사항과 결합되고 결과적으로 리팩터링 내성이 떨어진다.

직접 테스트하는 대신, 간접적으로 테스트하자

### 11.1.2 비공개 메서드와 불필요한 커버리지

비공개 메서드가 너무 복잡해서 테스트 하기에 충분한 커버리지를 얻지 못한 경우

사용이 되지 않는 관계없는 코드일 수 있다.

너무 복잡하면 별도의 클래스로 도출해야 하는 추상화가 누락된 코드이다.

```java
public class MyOrder {

    private MyCustomer customer;
    private List<MyProduct> productList;
    
    
    public String generateDescription() {
        return String.format(
                "Customer Name : %s \n" +
                        "Total number of Products : %d \n" +
                        "Total Price : %d",
                customer.getName(),
                productList.size(),
                getPrice()
        );
    }
    
    private double getPrice() {
        double basePrice = 0;
        double discounts = 0;
        double taxes = 0;
        return basePrice - discounts + taxe
    }
    
}
```

```java
public class MyOrderRefactor {

    private MyCustomer customer;
    private List<MyProduct> productList;
    private Calculator calculator;

    public String generateDescription() {
        return String.format(
                "Customer Name : %s \n" +
                        "Total number of Products : %d \n" +
                        "Total Price : %d",
                customer.getName(),
                productList.size(),
                calculator.calculate(customer, productList));
    }
    
    class Calculator{
        public double calculate(MyCustomer myCustomer, List<MyProduct> productList) {
            double basePrice = 0;
            double discounts = 0;
            double taxes = 0;
            return basePrice - discounts + taxes;
        }
    }
```

### 11.1.3 비공개 메서드 테스트가 타당한 경우

|  | 식별할 수 있는 동작 | 구현 세부 사항 |
| --- | --- | --- |
| 공개 | 좋음 | 나쁨 |
| 비공개 | 해당 없음 | 좋음 |

캡슐화를 지키는데 필요한 전제 조건이 모두 포함돼 있는지 확인해라.

## 11.2 비공개 상태 노출

단위 테스트를 목적으로만 비공개 상태를 노출하는 것

비공개 상태는 '구현 세부 사항'이기 때문에 단위 테스트에서 이것을 검증할 필요는 없다.

오히려 리팩토링 내성을 떨어뜨려 나쁜 테스트를 만들게 된다.

## 11.3 테스트로 유출된 도메인 지식

도메인 지식이 테스트로 유출되는 것은 또 하나의 안티패턴이다. 

이것은 도메인 지식이 바뀔 때 마다 코드를 수정해야하며, 이는 리팩토링 내성의 저하를 불러일으킨다.

```java
public class MyCalculator {
	public int calculate(int n1, int n2) {
	    return n1 + n2;
	}
	
	@Test
	void test() {
	    // given
	    MyCalculator myCalculator = new MyCalculator();
	    int n1 = 1;
	    int n2 = 2;
	    int expected = n1 + n2; // 도메인 지식 유출
	
	    // when
	    int calculate = myCalculator.calculate(n1, n2);
	
	    // then
	    assertThat(calculate).isEqualTo(expected);
	}
}
```

## 11.4 코드 오염

코드오염이란?

> 테스트에서만 사용하는 코드가 제품 코드에 포함되는 것
> 

```java
public void some_test()
{
	var logger = new Logger(true); // test 환경에만 true
	var sut = new Controller();
	
	sut.SomeMethod(logger);
}
```

## 11.5 구체 클래스를 목으로 처리하기

외부 통신하는 인터페이스를 Mock으로 처리하는 것을 배웠지만 구체 클래스를 목으로 바꿔 일부 기능만 사용하는 것은 SRP에 위배된다.

Mock으로 바꾼 구체 클래스의 기본 기능을 살린다는 것은 Mock의 비즈니스 로직이 필요한 경우다. 즉, Mock으로 사용된 객체는 비즈니스 로직 + 외부 프로세스 통신에 대한 책임을 모두 가진다는 것을 의미한다.

이를 험블 객체 패턴을 통해 나누어 줄 수 있다.

## **11.6 시간 처리하기**

많은 어플리케이션에는 시간에 따라 달라지는 기능이 존재할 수 있다. 시간에 따라 달라지는 기능을 테스트하면 거짓 양성이 발생할 가능성이 크다. 따라서 이 부분을 테스트 하기 위해서는 프로덕트 코드 역시 수정되어야 하고, 테스트 코드 역시 수정되어야 한다.

****명시적 의존성 형태로 시간 주입하기****

```java
public class TimeController {

    private MyTimer timer;

    // 시간을 값 객체로 주입받는다.
    public TimeController(MyTimer timer) {
        this.timer = timer;
    }

    public void toInquery() {
        LocalDateTime time = timer.getTime();

    }
}
```

## 요약

- 단위 테스트를 가능하게 하고자 비공개 메서드를 노출하게 되면 테스트가 세부 구현 사항에 결합되게 된다. 그 결과 리팩토링 내성이 떨어진다. 비공개 메서드를 직접 테스트하는 대신, 식별할 수 있는 동작으로서 간접적으로 테스트하라.
- 비공개 메서드가 너무 복잡해서 공개 API로 테스트를 할 수 없다면, 추상화가 누락되었다는 뜻이다. 비공개 메서드를 공개로 하지 말고, 복잡한 녀석을 별도의 클래스로 추출하라.
- 비공개 상태를 단위 테스트만을 위해 노출하지 마라. 테스트 코드는 제품 코드와 같은 코드가 사용되는 방식을 이용해서 검증해야한다. 이것은 또한 코드 오염에 해당되기도 한다.
- 테스트를 작성할 때 특정 구현을 암시하지 마라. 이것은 도메인 로직이 테스트 코드에 유출된 것이고, 리팩토링 내성을 떨어뜨린다.
- 코드 오염은 테스트에만 필요한 코드를 제품 코드에 포함하는 것을 의미한다. 제품 코드 내에 다양한 코드가 산재되기 때문에 유지보수 비용이 증가한다. 만약 테스트를 위한 코드가 반드시 포함되어야하면 인터페이스를 하나 추가하고, 그 인터페이스의 구현체를 생성하는 것으로 코드 오염을 최소화한다.
- 기능을 지키기 위해 구체 클래스를 Mock으로 처리해야 한다면, 이는 단일책임원칙(SRP)를 위반한 결과다. Mock으로 처리한다는 것은 외부 프로세스와 통신하는 역할을 하기 때문이다. 만약 이런 문제가 있다면, 외부 프로세스 통신과 도메인 로직이 있는 클래스를 분리하라.